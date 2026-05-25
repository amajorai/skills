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

If updated, stop and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.


## Phase 1: Detect Current State

Run silently before asking anything:

```bash
uname -a && cat /etc/os-release
whoami && id
grep -E "^Port|^PermitRootLogin|^PasswordAuthentication|^PubkeyAuthentication|^Banner" /etc/ssh/sshd_config 2>/dev/null
ss -tlnp | grep sshd
grep -E '^sudo' /etc/group
ls ~/.ssh/authorized_keys 2>/dev/null && echo "KEYS_PRESENT" || echo "NO_KEYS"
ss -tlnp
curl -s --max-time 2 http://169.254.169.254/latest/meta-data/instance-id 2>/dev/null && echo "IS_AWS" || true
curl -s --max-time 2 http://169.254.169.254/latest/meta-data/services/domain 2>/dev/null | grep -qi lightsail && echo "IS_LIGHTSAIL" || true
curl -s --max-time 2 http://169.254.0.1/metadata 2>/dev/null | grep -qi hetzner && echo "IS_HETZNER" || true
which fail2ban-client ufw clamav lynis auditd 2>/dev/null
aa-status 2>/dev/null | head -3 || echo "AppArmor not active"
systemctl is-active auditd 2>/dev/null && (auditctl -l 2>/dev/null | grep -q . && echo "AUDIT_RULES_PRESENT" || echo "AUDIT_NO_CUSTOM_RULES") || echo "AUDITD_NOT_RUNNING"
which docker 2>/dev/null && docker ps --format '{{.Names}}: {{.Ports}}' 2>/dev/null && echo "DOCKER_PRESENT" || echo "NO_DOCKER"
grep -rE "NOPASSWD" /etc/sudoers /etc/sudoers.d/ 2>/dev/null && echo "NOPASSWD_PRESENT" || echo "NO_NOPASSWD"
ls /etc/ssh/sshd_config.d/ 2>/dev/null
```

Note: SSH port, password auth state, SSH key presence, cloud provider, open ports, Docker, auditd rules, NOPASSWD.


## Phase 2: Full Interview

Present everything in one message. Tailor warnings based on Phase 1 findings.

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
> **A. SSH Key Setup** *(only shown if no keys detected)*
> ⚠️ No SSH keys configured. Disabling password auth without keys = **permanent lockout**.
> - [ ] Yes: generate a key pair and add it first (required to safely disable password auth)
> - [ ] Skip: I'll add my key manually before we proceed
>
> **B. Non-root sudo user** *(only shown if running as root)*
> - [ ] Yes: create a non-root sudo user → **Username?**
> - [ ] Skip
>
> **C. Harden SSH**
> - [ ] Change SSH port away from 22 → **What port?** (default: 2222)
> - [ ] Disable password auth ⚠️ Only safe if keys confirmed working
> - [ ] Disable root login via SSH
> - [ ] Restrict SSH to specific users (`AllowUsers`) → **Which usernames?**
> - [ ] Disable TCP/agent forwarding
> - [ ] Drop idle sessions (~10 min idle)
> - [ ] Restrict to modern crypto only (aes256-gcm + chacha20-poly1305, ETM MACs, curve25519)
>
> **D. UFW Firewall**
> - [ ] Enable UFW with rate-limited SSH + deny everything else
> *(If Docker detected)* ⚠️ Docker publishes ports via its own iptables rules that bypass UFW — I'll audit every Docker-published port.
>
> **E. Provider-level Firewall** *(only shown for AWS/Hetzner/OVH)*
> [For AWS Lightsail]: ⚠️ Lightsail has its **own** firewall separate from UFW. SSH port change MUST be added there too or you're locked out. Same for 80/443.
> [For EC2]: Security Groups need updating too.
> - [ ] Yes: configure via CLI
> - [ ] No: I'll update the provider firewall manually
>
> **F. fail2ban** — bans IPs that fail SSH login too many times
> - [ ] Install and configure
>
> **G. Unattended security updates** — auto-applies patches, no reboot
> - [ ] Enable
>
> **H. System hardening**
> - [ ] Kernel sysctl (network: redirects, SYN-flood, anti-spoof, ICMP; kernel: ASLR, ptrace, kptr/dmesg restrict)
> - [ ] Disable core dumps
> - [ ] Filesystem protections (protected_hardlinks/symlinks/fifos/regular)
> - [ ] Verify AppArmor enforcing
> - [ ] Disable unused services (avahi, cups, bluetooth)
> - [ ] Lock root password
> - [ ] Password policy in `/etc/login.defs` (SHA512, UMASK 022, HOME_MODE 0750)
> - [ ] auditd — install + meaningful custom ruleset (default install records nothing useful)
> - [ ] Review sudo NOPASSWD policy *(only shown if NOPASSWD detected)*
>
> **I. Login banner**
> - [ ] Set up login banner → **Text?** (blank = standard legal warning)
>
> **J. Optional extras**
> - [ ] Lynis: full security audit after hardening
> - [ ] ClamAV: antivirus with daily scans
> - [ ] SSH 2FA: TOTP on top of SSH key

Wait for answers. Summarize the plan, then ask: **"Ready to proceed?"**


## Phase 3: Pre-flight Safety Checks

Before writing any config file, run these checks based on selections:

**If "disable password auth" selected:**
> ⚠️ Ask user to open a NEW terminal and confirm key login works: `ssh -i <key> USER@{{args}} echo "key works"`. Do not disable password auth until they confirm.

**If "change SSH port" selected:**
- Remind: new port must be opened in UFW AND provider firewall BEFORE sshd restarts.
- Lightsail: must update Lightsail firewall or access is lost when sshd moves.
- EC2: must update the Security Group.

**If no keys and user skipped key setup:**
> ❌ Cannot safely disable password auth. Either help them set up keys first or skip that step.


## Phase 4: SSH Key Setup (if selected: A)

Run on the **local machine**:

```bash
ssh-keygen -t ed25519 -C "vps-hardening" -f ~/.ssh/id_ed25519_vps
ssh-copy-id -i ~/.ssh/id_ed25519_vps.pub USER@{{args}}
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

Instruct user: **Open a new terminal and confirm SSH works as USERNAME before continuing.**


## Phase 6: Harden SSH (if selected: C)

Write all changes to a single drop-in file so the stock `sshd_config` stays untouched:

```bash
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
grep -q "^Include /etc/ssh/sshd_config.d/\*.conf" /etc/ssh/sshd_config \
  || echo "Include /etc/ssh/sshd_config.d/*.conf" >> /etc/ssh/sshd_config
mkdir -p /etc/ssh/sshd_config.d
```

Build the drop-in including **only** the lines for what the user selected. See [references/ssh-config.md](references/ssh-config.md) for all option blocks (baseline, port, root login, password auth, AllowUsers, TCP forwarding, idle timeout, modern crypto).

```bash
sshd -t && echo "Config OK"
```

**Do not restart sshd here — do it after UFW is configured.**


## Phase 7: UFW Firewall (if selected: D)

```bash
apt-get install -y ufw
ufw --force reset
ufw default deny incoming
ufw default allow outgoing
ufw limit NEW_PORT/tcp comment "SSH"
ss -tlnp
```

⚠️ If Docker is installed: Docker writes its own iptables rules into the `DOCKER` / `DOCKER-USER` chains, which are evaluated **before** UFW's FORWARD rules. Containers with `-p PORT:PORT` are reachable from the internet even when UFW says DENY.

```bash
docker ps --format 'table {{.Names}}\t{{.Ports}}' 2>/dev/null
```

Audit every Docker-published port the same way you audit a UFW `allow`. To protect sensitive ports: bind containers to `127.0.0.1:PORT:PORT` and use a reverse proxy, or add rules to the `DOCKER-USER` chain (UFW cannot manage this chain).

Ask: "Which of these ports need to stay open?" Then apply their answers and enable UFW:

```bash
# ufw allow <PORT>/tcp comment "<label>"
ufw --force enable
ufw status verbose
```

Restart sshd. Instruct user: **Open a new terminal and SSH on the new port before closing this session.**

```bash
systemctl restart sshd
ss -tlnp | grep ssh
```


## Phase 8: Provider Firewall (if selected: E)

For CLI commands to configure AWS Lightsail, EC2 Security Groups, Hetzner, and OVH firewalls, see [references/provider-firewall.md](references/provider-firewall.md).

⚠️ Always add the new SSH port to the provider firewall **before** restarting sshd. Keep the old port open until the new one is confirmed working.


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

systemctl enable fail2ban && systemctl restart fail2ban
fail2ban-client status sshd
```

Note: `port = NEW_PORT` must match the hardened SSH port. On distros logging SSH to a file (not journal): set `backend = auto` and `logpath = /var/log/auth.log`.


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

Apply only the sub-options the user selected. For the complete sysctl config, filesystem protections, core dump settings, login.defs changes, NOPASSWD handling, and auditd ruleset, see [references/system-hardening.md](references/system-hardening.md).

```bash
# Disable unused services (if selected)
systemctl disable --now avahi-daemon cups bluetooth 2>/dev/null || true

# Write and apply sysctl config (see references/system-hardening.md for file content)
sysctl -p /etc/sysctl.d/99-hardening.conf

# Lock root password (if selected)
passwd -l root

# Restrict su to sudo group (if selected)
dpkg-statoverride --update --add root sudo 4750 /bin/su

# AppArmor (if selected)
systemctl enable --now apparmor
aa-enforce /etc/apparmor.d/* 2>/dev/null || true
aa-status | head -5
```


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

For full installation commands for Lynis, ClamAV, and SSH 2FA (TOTP), see [references/extras.md](references/extras.md).

- **Lynis**: full security audit; aim for hardening index 70+
- **ClamAV**: antivirus daemon with daily cron scan at 3am
- **SSH 2FA**: `libpam-google-authenticator` + TOTP ⚠️ test in a new terminal before closing your current session


## Phase 14: Final Verification & Completion

For the full verification command block and the per-selection completion checklist, see [references/completion-checklist.md](references/completion-checklist.md).
