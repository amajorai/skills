---
name: hardening
description: Harden a self-hosted Linux server. Detects current state, runs a full interview to understand what the user wants, warns about edge cases (no SSH keys, password-only auth, provider firewalls), then implements only the selected steps in a safe order.
argument-hint: [server IP or hostname, optional]
---

# Hardening

You are hardening a Linux VPS. **Do not implement anything until the interview is complete and the user has confirmed their selections.** The order matters: detect state → interview → confirm → implement.

**Target:** {{args}}


## Phase 0: Auto-Update

*Skip unless `{{args}}` contains `--update`, or `SKILLS_AUTO_UPDATE: true` is set in your project CLAUDE.md.*

```bash
npx --yes skills update hardening -y 2>/dev/null || true
```

This is best-effort: if the update fails (offline, network error), continue silently. If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.

## Phase 1: Detect Current State

Run silently before asking anything:

```bash
# OS
uname -a && cat /etc/os-release

# Who we are
whoami && id

# Current SSH config
grep -E "^Port|^PermitRootLogin|^PasswordAuthentication|^PubkeyAuthentication|^Banner" /etc/ssh/sshd_config 2>/dev/null
ss -tlnp | grep sshd

# Existing users with sudo
grep -E '^sudo' /etc/group

# SSH keys present?
ls ~/.ssh/authorized_keys 2>/dev/null && echo "KEYS_PRESENT" || echo "NO_KEYS"

# What's listening
ss -tlnp

# Cloud provider detection
curl -s --max-time 2 http://169.254.169.254/latest/meta-data/instance-id 2>/dev/null && echo "IS_AWS" || true
curl -s --max-time 2 http://169.254.169.254/latest/meta-data/services/domain 2>/dev/null | grep -qi lightsail && echo "IS_LIGHTSAIL" || true
curl -s --max-time 2 http://169.254.0.1/metadata 2>/dev/null | grep -qi hetzner && echo "IS_HETZNER" || true

# Installed tools
which fail2ban-client ufw clamav lynis auditd 2>/dev/null

# AppArmor
aa-status 2>/dev/null | head -3 || echo "AppArmor not active"

# auditd present / running?
systemctl is-active auditd 2>/dev/null && (auditctl -l 2>/dev/null | grep -q . && echo "AUDIT_RULES_PRESENT" || echo "AUDIT_NO_CUSTOM_RULES") || echo "AUDITD_NOT_RUNNING"

# Docker present? (Docker bypasses UFW with its own iptables rules)
which docker 2>/dev/null && docker ps --format '{{.Names}}: {{.Ports}}' 2>/dev/null && echo "DOCKER_PRESENT" || echo "NO_DOCKER"

# Current sudoers policy (NOPASSWD risk?)
grep -rE "NOPASSWD" /etc/sudoers /etc/sudoers.d/ 2>/dev/null && echo "NOPASSWD_PRESENT" || echo "NO_NOPASSWD"

# Existing SSH drop-in hardening config?
ls /etc/ssh/sshd_config.d/ 2>/dev/null
```

Analyze the output and note:
- Are they currently root or a non-root user?
- Is SSH on port 22 or already changed?
- Is password auth on or off?
- Are SSH keys already set up?
- What cloud provider?
- What ports are open?
- Is Docker installed? (it manages its own iptables and bypasses UFW for published ports)
- Is auditd running, and does it have custom rules or only the default empty ruleset?
- Does sudo allow NOPASSWD (passwordless privilege escalation)?


## Phase 2: Full Interview

Present everything in one message. Tailor the warnings based on what you detected in Phase 1.


> **Server hardening setup: tell me what you want and I'll implement it all in one pass.**
>
> I've scanned your server. Here's what I found:
> - **Current SSH port:** [detected port]
> - **Password auth:** [on/off]
> - **SSH keys:** [present / not found]
> - **Running as:** [root / user]
> - **Provider detected:** [AWS / Hetzner / OVH / unknown]
> - **Open ports:** [list]
>
> ---
>
> **Select what you want to set up:**
>
> **A. SSH Key Setup**
> *(Only shown if no keys detected)*
> ⚠️ You have no SSH keys configured. If we disable password auth without setting up keys first, you will be **permanently locked out**.
> - [ ] Yes: generate a key pair and add it before anything else (required if you want to disable password auth)
> - [ ] Skip: I'll add my own key manually before we proceed
>
> **B. Non-root sudo user**
> *(Only shown if currently running as root)*
> Running as root is dangerous. We'll create a regular user with sudo access.
> - [ ] Yes: create a non-root sudo user (recommended)
> - [ ] Skip
> → If yes: **What username?** (e.g. `deploy`)
>
> **C. Harden SSH**
> - [ ] Change SSH port away from 22 (reduces automated attack noise)
>   → If yes: **What port?** (default: 2222)
> - [ ] Disable password auth (require SSH key only)
>   ⚠️ Only safe if keys are already set up and confirmed working
> - [ ] Disable root login via SSH
> - [ ] Restrict SSH to specific users only (`AllowUsers` allowlist)
>   → If yes: **Which usernames?** (e.g. `ubuntu deploy`)
> - [ ] Disable TCP/agent forwarding (`AllowTcpForwarding no`) — recommended unless you tunnel through this host
> - [ ] Drop idle sessions (`ClientAliveInterval 300` / `ClientAliveCountMax 2` = disconnect after ~10 min idle)
> - [ ] Restrict to modern crypto only (strong Ciphers / MACs / KexAlgorithms)
>   *(applies aes256-gcm + chacha20-poly1305 ciphers, ETM-mode MACs, curve25519 key exchange — drops legacy/weak algorithms)*
>
> **D. UFW Firewall**
> - [ ] Enable UFW with rate-limited SSH + deny everything else
>   → I'll show you what's listening and ask which ports to keep open
> *(If Docker was detected)* ⚠️ Docker publishes ports by writing its **own** iptables rules that bypass UFW. Containers with `-p 80:80` will be reachable from the internet **even though UFW says DENY**. I'll flag every Docker-published port and explain how to handle this.
>
> **E. Provider-level Firewall**
> *(Only shown for AWS/Hetzner/OVH)*
> [For AWS Lightsail]: ⚠️ Lightsail has its **own** firewall that is **completely separate** from UFW and host iptables. It has THREE consequences you must handle:
>   1. If we change the SSH port, you MUST add the new port in the **Lightsail console** (Networking → Firewall) too, or you are locked out — UFW allowing it is not enough.
>   2. Public ports 80/443 must be open in the Lightsail firewall as well, not just UFW/Docker.
>   3. Closing/limiting the old port 22 in UFW does nothing at the provider edge until you also remove it from the Lightsail firewall.
> [For EC2]: Security Groups also need to be updated (same SSH-port gotcha applies).
> - [ ] Yes: install + authenticate the provider CLI and configure the firewall from here
> - [ ] No: I'll update the provider firewall manually
>
> **F. fail2ban**
> Bans IPs that fail SSH login too many times.
> - [ ] Install and configure fail2ban (recommended)
>
> **G. Unattended security updates**
> Auto-applies security patches silently, no reboot.
> - [ ] Enable unattended-upgrades (recommended)
>
> **H. System hardening**
> - [ ] Kernel sysctl hardening — network (block redirects + source routing, SYN-flood protection, anti-spoof rp_filter, ignore ICMP broadcast/bogus, IPv6 redirect protection) **and** kernel (full ASLR, ptrace scope, kptr/dmesg restrict)
>   *(If Docker was detected)* I'll keep `net.ipv4.ip_forward=1` so Docker networking keeps working.
> - [ ] Disable core dumps (prevents secrets leaking from memory dumps — sets both `limits.conf` hard+soft and `fs.suid_dumpable=0`)
> - [ ] Filesystem protections (`fs.protected_hardlinks/symlinks/fifos/regular` — blocks classic /tmp symlink & FIFO attacks)
> - [ ] Verify AppArmor is enforcing
> - [ ] Disable unused system services (avahi, cups, bluetooth)
> - [ ] Lock root password
> - [ ] Password / encryption policy in `/etc/login.defs` (`ENCRYPT_METHOD SHA512`, `UMASK 022`, `HOME_MODE 0750` so home dirs aren't world-readable)
> - [ ] auditd — install + enable, and add a meaningful custom ruleset (default install runs with an **empty** ruleset and records nothing useful)
> - [ ] Review sudo policy
>   *(If NOPASSWD detected)* ⚠️ The current user has **NOPASSWD:ALL** (cloud-init default). Anyone who gets a shell as this user has instant root with no password. On a key-only server this is a documented, accepted risk — but I can switch sudo to require a password if you want.
>
> **I. Login banner**
> Shows a warning message to anyone who connects via SSH.
> - [ ] Yes: set up a login banner
>   → **What text?** (or leave blank and I'll generate a standard legal warning)
>
> **J. Optional extras**
> - [ ] Lynis: run a full security audit after hardening (shows a score + recommendations)
> - [ ] ClamAV: install antivirus with daily scans
> - [ ] SSH 2FA: require TOTP authenticator app on top of SSH key

Wait for the user's answers. Once confirmed, summarize the plan and ask: **"Ready to proceed?"**


## Phase 3: Pre-flight Safety Checks

Before writing a single config file, run these checks based on what was selected:

**If "disable password auth" was selected:**
```bash
# Verify key auth actually works right now
# Instruct user: open a NEW terminal and run:
# ssh -i <your-key> USER@{{args}} echo "key works"
# Only proceed after they confirm it worked
```
> ⚠️ Do not disable password auth until the user has confirmed key login works in a separate terminal.

**If "change SSH port" was selected:**
- Remind user: the new port must be opened in UFW AND in the provider firewall BEFORE sshd restarts.
- On AWS Lightsail: **must** update the Lightsail firewall or access is lost.
- On EC2: **must** update the Security Group.

**If no SSH key and they skipped key setup:**
> ❌ Cannot disable password auth safely. Either help them set up keys first or skip that step.


## Phase 4: SSH Key Setup (if selected: A)

Run on the **local machine**:

```bash
ssh-keygen -t ed25519 -C "vps-hardening" -f ~/.ssh/id_ed25519_vps
# Add public key to server (still using password at this point)
ssh-copy-id -i ~/.ssh/id_ed25519_vps.pub USER@{{args}}
# Verify key login works before proceeding
ssh -i ~/.ssh/id_ed25519_vps USER@{{args}} echo "Key auth confirmed"
```

**Do not continue until the user confirms key login works.**


## Phase 5: Create Non-Root Sudo User (if selected: B)

```bash
adduser --gecos "" USERNAME
usermod -aG sudo USERNAME
mkdir -p /home/USERNAME/.ssh
cp /root/.ssh/authorized_keys /home/USERNAME/.ssh/
chown -R USERNAME:USERNAME /home/USERNAME/.ssh
chmod 700 /home/USERNAME/.ssh && chmod 600 /home/USERNAME/.ssh/authorized_keys
```

Instruct user: **Open a new terminal and confirm you can SSH as USERNAME before continuing.**


## Phase 6: Harden SSH (if selected: C)

Write all SSH hardening to a single drop-in file so the stock `sshd_config` stays untouched and the changes are easy to review/revert as one unit (this matches how production servers keep `/etc/ssh/sshd_config.d/hardened.conf`):

```bash
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
# Confirm the main config actually includes the drop-in dir (Ubuntu 20.04+ does by default):
grep -q "^Include /etc/ssh/sshd_config.d/\*.conf" /etc/ssh/sshd_config \
  || echo "Include /etc/ssh/sshd_config.d/*.conf" >> /etc/ssh/sshd_config
mkdir -p /etc/ssh/sshd_config.d
```

Build the drop-in, including **only** the lines for what the user selected. Start with the always-on baseline:

```bash
cat > /etc/ssh/sshd_config.d/hardened.conf << 'EOF'
# Baseline (always applied when hardening SSH)
PubkeyAuthentication yes
X11Forwarding no
MaxAuthTries 3
LoginGraceTime 30
EOF
```

Then append the selected options:

```bash
# If changing port:
echo "Port NEW_PORT" >> /etc/ssh/sshd_config.d/hardened.conf

# If disabling root login:
echo "PermitRootLogin no" >> /etc/ssh/sshd_config.d/hardened.conf

# If disabling password auth:
echo "PasswordAuthentication no" >> /etc/ssh/sshd_config.d/hardened.conf

# If restricting to specific users (AllowUsers allowlist) — space-separated:
echo "AllowUsers USER1 USER2" >> /etc/ssh/sshd_config.d/hardened.conf

# If disabling TCP forwarding:
echo "AllowTcpForwarding no" >> /etc/ssh/sshd_config.d/hardened.conf

# If dropping idle sessions (~10 min):
cat >> /etc/ssh/sshd_config.d/hardened.conf << 'EOF'
ClientAliveInterval 300
ClientAliveCountMax 2
EOF

# If restricting to modern crypto only:
cat >> /etc/ssh/sshd_config.d/hardened.conf << 'EOF'
Ciphers aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group16-sha512
EOF
```

⚠️ Before applying the crypto restrictions, confirm your SSH **client** supports these algorithms (any OpenSSH from the last several years does). An old/embedded client that only speaks legacy ciphers would be locked out.

```bash
# Validate before any restart
sshd -t && echo "Config OK"
```

**Do not restart sshd here: do it after UFW is configured.**


## Phase 7: UFW Firewall (if selected: D)

```bash
apt-get install -y ufw
ufw --force reset
ufw default deny incoming
ufw default allow outgoing

# Rate-limited SSH (blocks IPs with >6 connections in 30s, before fail2ban even fires)
ufw limit NEW_PORT/tcp comment "SSH"
```

Show the user what's listening:

```bash
ss -tlnp
```

**⚠️ If Docker is installed, UFW does not protect Docker-published ports.** Docker writes its own rules into the iptables `DOCKER` / `DOCKER-USER` chains, which are evaluated **before** UFW's `FORWARD` rules. A container started with `-p 80:80` (or a Swarm/compose published port) is reachable from the internet even when `ufw status` shows `deny incoming`. To handle this:

```bash
# List what Docker has actually published to the host (these bypass UFW):
docker ps --format 'table {{.Names}}\t{{.Ports}}' 2>/dev/null

# Option 1 (recommended): bind sensitive containers to localhost only,
#   e.g. publish as 127.0.0.1:PORT:PORT instead of PORT:PORT, and front them
#   with a reverse proxy / Cloudflare Tunnel rather than a public host port.
# Option 2: enforce rules in the DOCKER-USER chain (UFW cannot, Docker won't touch it), e.g.:
#   iptables -I DOCKER-USER -p tcp --dport DOCKER_PORT ! -s TRUSTED_CIDR -j DROP
```

> Audit every Docker-published port the same way you audit a UFW `allow`. Intentionally-public ports (80/443 behind a proxy) are fine; a stray `docker-proxy` port open to `0.0.0.0` is an exposure UFW will silently fail to block.

Ask: "Which of these ports need to stay open? I'll allow only those." Apply their answers:

```bash
# Example: ufw allow 443/tcp comment "HTTPS"
ufw --force enable
ufw status verbose
```

Now restart sshd:

```bash
systemctl restart sshd
ss -tlnp | grep ssh
```

Instruct user: **Open a new terminal and SSH on the new port: `ssh -p NEW_PORT USERNAME@{{args}}`**
Do not close the current session until confirmed.


## Phase 8: Provider Firewall (if selected: E)

### AWS Lightsail

⚠️ Lightsail's firewall is a **separate layer at the provider edge** — independent of UFW and host iptables. Traffic is filtered there *before* it ever reaches UFW. Critical consequences:
- **SSH port change:** the new port MUST be added to the Lightsail firewall, or you are locked out the moment sshd moves — UFW allowing it is irrelevant. **Do this BEFORE restarting sshd**, and keep port 22 open in Lightsail until you've confirmed the new port works.
- **Public web ports:** 80/443 must be opened in the Lightsail firewall too (not just UFW / Docker), or the site is unreachable from outside.
- Removing a port from UFW does not close it at the Lightsail edge, and vice versa — you must keep both layers in sync.

```bash
# If no AWS CLI, install it
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip && sudo ./aws/install
aws configure
```

If user wants to use the console instead:
> Lightsail Console → your instance → Networking → Firewall → Add rule → Custom TCP, port NEW_PORT (add the new SSH port and 80/443 as needed). Remove port 22 only after the new port is confirmed working.

### AWS EC2: Security Groups

```bash
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
SG_ID=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID \
  --query 'Reservations[0].Instances[0].SecurityGroups[0].GroupId' --output text)

aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID --protocol tcp --port NEW_PORT --cidr 0.0.0.0/0

# After confirming new port works:
aws ec2 revoke-security-group-ingress \
  --group-id $SG_ID --protocol tcp --port 22 --cidr 0.0.0.0/0
```

### Hetzner

```bash
curl -fsSL https://github.com/hetznercloud/cli/releases/latest/download/hcloud-linux-amd64.tar.gz \
  | tar xz -C /usr/local/bin/
hcloud context create hardening
# Paste API token from cloud.hetzner.com → Security → API Tokens

SERVER_NAME=$(hostname)
hcloud firewall create --name "$SERVER_NAME-fw"
hcloud firewall add-rule "$SERVER_NAME-fw" \
  --direction in --protocol tcp --port NEW_PORT \
  --source-ips 0.0.0.0/0 --source-ips ::/0
hcloud firewall apply-to-resource "$SERVER_NAME-fw" --type server --server "$SERVER_NAME"
```

### OVH

```bash
curl -fsSL https://raw.githubusercontent.com/ovh/ovhcloud-cli/main/install.sh | sh
ovhcloud login
```

> OVH firewall CLI varies by product. If CLI doesn't work for your product type, go to:
> OVH Manager → Bare Metal Cloud → IP → Firewall → add rule for NEW_PORT


## Phase 9: fail2ban (if selected: F)

```bash
apt-get install -y fail2ban

cat > /etc/fail2ban/jail.local << EOF
[DEFAULT]
bantime  = 1h
findtime = 10m
maxretry = 5
backend  = systemd

[sshd]
enabled  = true
port     = NEW_PORT
logpath  = %(sshd_log)s
findtime = 10m
maxretry = 3
bantime  = 24h
EOF

# Note: `port = NEW_PORT` MUST match the hardened SSH port, or fail2ban watches the
# wrong port and never bans attackers. On distros that log SSH to a plain file rather
# than the journal, set `backend = auto` and `logpath = /var/log/auth.log`.

systemctl enable fail2ban && systemctl restart fail2ban
fail2ban-client status sshd
```


## Phase 10: Unattended Security Updates (if selected: G)

```bash
apt-get install -y unattended-upgrades apt-listchanges

cat > /etc/apt/apt.conf.d/50unattended-upgrades << 'EOF'
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};
Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::MinimalSteps "true";
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "false";
EOF

cat > /etc/apt/apt.conf.d/20auto-upgrades << 'EOF'
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
EOF

unattended-upgrade --dry-run --debug
systemctl enable unattended-upgrades
```


## Phase 11: System Hardening (if selected: H)

Apply only the sub-options the user selected:

```bash
# Disable unused services
systemctl disable --now avahi-daemon cups bluetooth 2>/dev/null || true

# Kernel sysctl hardening
cat > /etc/sysctl.d/99-hardening.conf << 'EOF'
# --- Network: redirects, source routing, SYN-flood, spoofing ---
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.log_martians = 1
# --- ICMP ---
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1
# --- IPv6 redirect protection ---
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
# --- Kernel hardening ---
kernel.randomize_va_space = 2
kernel.yama.ptrace_scope = 1
kernel.kptr_restrict = 2
kernel.dmesg_restrict = 1
EOF

# If Docker (or any container/router role) is present, IP forwarding must stay on,
# otherwise container networking breaks. Add this ONLY when needed:
# echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.d/99-hardening.conf

sysctl -p /etc/sysctl.d/99-hardening.conf

# Filesystem protections (if selected)
cat >> /etc/sysctl.d/99-hardening.conf << 'EOF'
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
fs.protected_fifos = 1
fs.protected_regular = 2
EOF
sysctl -p /etc/sysctl.d/99-hardening.conf

# Disable core dumps — set BOTH hard and soft limits, plus the kernel flag,
# so neither a user nor a setuid binary can produce a dump that leaks secrets.
cat >> /etc/security/limits.conf << 'EOF'
* hard core 0
* soft core 0
EOF
echo "fs.suid_dumpable = 0" >> /etc/sysctl.d/99-hardening.conf
sysctl -p /etc/sysctl.d/99-hardening.conf

# Password / encryption policy in /etc/login.defs (if selected)
sed -i 's/^#*ENCRYPT_METHOD.*/ENCRYPT_METHOD SHA512/' /etc/login.defs
grep -q '^ENCRYPT_METHOD' /etc/login.defs || echo 'ENCRYPT_METHOD SHA512' >> /etc/login.defs
sed -i 's/^#*UMASK.*/UMASK 022/' /etc/login.defs
sed -i 's/^#*HOME_MODE.*/HOME_MODE 0750/' /etc/login.defs
grep -q '^HOME_MODE' /etc/login.defs || echo 'HOME_MODE 0750' >> /etc/login.defs
# Apply 0750 to existing home dirs too (login.defs only affects newly-created users):
for d in /home/*; do [ -d "$d" ] && chmod 0750 "$d"; done

# Lock down shared memory
echo "tmpfs /run/shm tmpfs defaults,noexec,nosuid 0 0" >> /etc/fstab

# Lock root password
passwd -l root

# Restrict su to sudo group
dpkg-statoverride --update --add root sudo 4750 /bin/su

# Sudo policy (if user chose to require a password instead of NOPASSWD):
#   The cloud-init default is `ubuntu ALL=(ALL) NOPASSWD:ALL` in /etc/sudoers.d/90-cloud-init-users.
#   If the user wants password-protected sudo, replace NOPASSWD with PASSWD for that user, e.g.:
# sed -i 's/NOPASSWD:ALL/ALL/' /etc/sudoers.d/90-cloud-init-users && visudo -c
#   If keeping NOPASSWD (accepted on a key-only server), record it as a known, documented risk
#   in the completion summary so it isn't forgotten.

# AppArmor: ensure enforcing
systemctl enable --now apparmor
aa-enforce /etc/apparmor.d/* 2>/dev/null || true
aa-status | head -5
# On a Docker host, confirm the docker-default profile is loaded — it confines every
# container that doesn't ship its own profile:
aa-status 2>/dev/null | grep -q docker-default && echo "docker-default profile active" || true
```


## Phase 11b: auditd (if selected: H → auditd)

A bare `apt install auditd` leaves you with the daemon running but an **empty ruleset** — it records essentially nothing actionable. The value is in the rules. Install, then drop in a meaningful baseline ruleset:

```bash
apt-get install -y auditd audispd-plugins

cat > /etc/audit/rules.d/99-hardening.rules << 'EOF'
# Make config immutable last; flush existing first
-D
-b 8192
-f 1

# Identity / auth changes
-w /etc/passwd -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/gshadow -p wa -k identity
-w /etc/sudoers -p wa -k scope
-w /etc/sudoers.d/ -p wa -k scope

# SSH config
-w /etc/ssh/sshd_config -p wa -k sshd
-w /etc/ssh/sshd_config.d/ -p wa -k sshd

# Login records
-w /var/log/auth.log -p wa -k auth
-w /var/log/faillog -p wa -k logins
-w /var/log/lastlog -p wa -k logins

# Privilege escalation
-w /bin/su -p x -k priv_esc
-w /usr/bin/sudo -p x -k priv_esc

# Kernel module load/unload
-w /sbin/insmod -p x -k modules
-w /sbin/rmmod -p x -k modules
-w /sbin/modprobe -p x -k modules
-a always,exit -F arch=b64 -S init_module -S delete_module -k modules

# Time changes
-a always,exit -F arch=b64 -S adjtimex -S settimeofday -k time-change
-w /etc/localtime -p wa -k time-change

# Make the ruleset immutable until next reboot (must be last)
-e 2
EOF

augenrules --load 2>/dev/null || service auditd reload
systemctl enable --now auditd
auditctl -l | head    # confirm rules are loaded (not empty)
auditctl -s | grep enabled
```

> If auditd was already running with only the default (empty) ruleset — a common gap on stock servers — this is exactly the fix: the daemon being "active" is meaningless without rules.


## Phase 12: Login Banner (if selected: I)

Use the user's provided text, or this default:

```bash
cat > /etc/issue.net << 'EOF'
*************************************************************
*  Authorized access only. All activity is monitored and   *
*  logged. Unauthorized access will be prosecuted.         *
*************************************************************
EOF

grep -q "^Banner" /etc/ssh/sshd_config \
  && sed -i "s|^Banner.*|Banner /etc/issue.net|" /etc/ssh/sshd_config \
  || echo "Banner /etc/issue.net" >> /etc/ssh/sshd_config

sshd -t && systemctl reload sshd
```


## Phase 13: Optional Extras (if selected: J)

### Lynis

```bash
apt-get install -y lynis
lynis audit system
# Aim for hardening index 70+. Review suggestions at the bottom of the report.
```

### ClamAV

```bash
apt-get install -y clamav clamav-daemon
systemctl stop clamav-freshclam
freshclam
systemctl start clamav-freshclam
systemctl enable --now clamav-daemon

# Initial scan
clamscan --recursive --infected /home /var/www 2>/dev/null

# Daily cron scan at 3am
echo "0 3 * * * root clamscan --recursive --infected --quiet /home /var/www >> /var/log/clamav/daily-scan.log 2>&1" \
  > /etc/cron.d/clamav-daily
```

### SSH 2FA (TOTP)

```bash
apt-get install -y libpam-google-authenticator

# Run as the non-root user: each user sets up their own TOTP
google-authenticator
# Scan the QR code with Google Authenticator / Authy, then answer y to all prompts

# Enable in PAM
sed -i 's/@include common-auth/auth required pam_google_authenticator.so\n@include common-auth/' \
  /etc/pam.d/sshd

# Enable challenge-response auth
sed -i 's/^#*KbdInteractiveAuthentication.*/KbdInteractiveAuthentication yes/' /etc/ssh/sshd_config
grep -q "^AuthenticationMethods" /etc/ssh/sshd_config \
  || echo "AuthenticationMethods publickey,keyboard-interactive" >> /etc/ssh/sshd_config

sshd -t && systemctl restart sshd
```

> ⚠️ Test 2FA in a new terminal before closing your current session.


## Phase 14: Final Verification

Run only the checks relevant to what was installed:

```bash
echo "=== SSH Port ===" && ss -tlnp | grep ssh
echo "=== UFW Status ===" && ufw status verbose
echo "=== fail2ban ===" && fail2ban-client status sshd 2>/dev/null || echo "not installed"
echo "=== Unattended upgrades ===" && systemctl is-active unattended-upgrades 2>/dev/null || echo "not installed"
echo "=== Listening ports ===" && ss -tlnp
echo "=== Sudo users ===" && grep -E '^sudo' /etc/group
echo "=== PermitRootLogin ===" && grep PermitRootLogin /etc/ssh/sshd_config
echo "=== PasswordAuthentication ===" && grep PasswordAuthentication /etc/ssh/sshd_config
echo "=== AppArmor ===" && aa-status 2>/dev/null | head -3 || echo "not active"
echo "=== SSH crypto/limits ===" && sshd -T 2>/dev/null | grep -E '^(ciphers|macs|kexalgorithms|allowusers|allowtcpforwarding|clientaliveinterval|clientalivecountmax) '
echo "=== sysctl hardening ===" && sysctl kernel.randomize_va_space kernel.kptr_restrict kernel.dmesg_restrict net.ipv4.icmp_echo_ignore_broadcasts fs.protected_symlinks fs.suid_dumpable 2>/dev/null
echo "=== Core dump limits ===" && grep -E 'core 0' /etc/security/limits.conf
echo "=== login.defs ===" && grep -E '^(ENCRYPT_METHOD|UMASK|HOME_MODE)' /etc/login.defs
echo "=== auditd ===" && (systemctl is-active auditd 2>/dev/null && (auditctl -l 2>/dev/null | grep -q . && echo "rules loaded" || echo "WARNING: running but NO rules")) || echo "not installed"
echo "=== Docker-published ports (bypass UFW!) ===" && docker ps --format '{{.Names}}: {{.Ports}}' 2>/dev/null || echo "no docker"
echo "=== Sudo NOPASSWD ===" && grep -rE NOPASSWD /etc/sudoers /etc/sudoers.d/ 2>/dev/null || echo "none (sudo requires password)"
echo "=== ClamAV ===" && systemctl is-active clamav-daemon 2>/dev/null || echo "not installed"
echo "=== 2FA ===" && grep AuthenticationMethods /etc/ssh/sshd_config 2>/dev/null || echo "not configured"
```


## Completion Checklist

Generate this checklist based only on what the user selected:

- [ ] SSH key auth confirmed working before disabling password login
- [ ] Non-root sudo user created and tested *(if selected)*
- [ ] SSH port changed from 22 → NEW_PORT *(if selected)*
- [ ] Password auth disabled *(if selected)*
- [ ] Root login disabled *(if selected)*
- [ ] SSH restricted to AllowUsers allowlist *(if selected)*
- [ ] TCP forwarding disabled *(if selected)*
- [ ] Idle sessions dropped (ClientAlive*) *(if selected)*
- [ ] Modern-only Ciphers/MACs/KexAlgorithms applied, client verified compatible *(if selected)*
- [ ] UFW enabled with rate-limited SSH, all unused ports closed *(if selected)*
- [ ] Docker-published ports audited (they bypass UFW) *(if Docker present)*
- [ ] Provider-level firewall updated — incl. new SSH port + 80/443 on Lightsail/EC2 *(if selected)*
- [ ] fail2ban installed and protecting SSH (port matches new SSH port) *(if selected)*
- [ ] Unattended security updates enabled *(if selected)*
- [ ] Kernel sysctl hardening applied (network + ICMP + IPv6 + kernel ASLR/ptrace/kptr/dmesg) *(if selected)*
- [ ] Filesystem protections applied (protected_hardlinks/symlinks/fifos/regular) *(if selected)*
- [ ] Core dumps disabled (hard + soft limits + suid_dumpable=0) *(if selected)*
- [ ] login.defs policy set (SHA512, UMASK 022, HOME_MODE 0750) + existing home dirs chmod'd *(if selected)*
- [ ] auditd installed with a real custom ruleset (not just running empty) *(if selected)*
- [ ] AppArmor enforcing (docker-default profile active on Docker hosts) *(if selected)*
- [ ] Root password locked *(if selected)*
- [ ] Sudo NOPASSWD reviewed — changed to require password, or documented as accepted risk *(if applicable)*
- [ ] Login banner configured *(if selected)*
- [ ] Lynis audit completed *(if selected)*
- [ ] ClamAV installed and scanning *(if selected)*
- [ ] SSH 2FA tested and working in new terminal *(if selected)*
- [ ] **Verified: can SSH in on correct port as non-root user from a fresh terminal**
