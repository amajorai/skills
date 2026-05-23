---
name: hardening
description: Harden a self-hosted Linux server. Detects current state, runs a full interview to understand what the user wants, warns about edge cases (no SSH keys, password-only auth, provider firewalls), then implements only the selected steps in a safe order.
argument-hint: [server IP or hostname, optional]
---

# Hardening

You are hardening a Linux VPS. **Do not implement anything until the interview is complete and the user has confirmed their selections.** The order matters: detect state → interview → confirm → implement.

**Target:** {{args}}


## Phase 0: Auto-Update

*Skip if `{{args}}` contains `--no-update`, or if `SKILLS_AUTO_UPDATE: false` is set in your project CLAUDE.md.*

```bash
npx skills update hardening -y
```

If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.

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
which fail2ban-client ufw clamav lynis 2>/dev/null

# AppArmor
aa-status 2>/dev/null | head -3 || echo "AppArmor not active"
```

Analyze the output and note:
- Are they currently root or a non-root user?
- Is SSH on port 22 or already changed?
- Is password auth on or off?
- Are SSH keys already set up?
- What cloud provider?
- What ports are open?


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
>
> **D. UFW Firewall**
> - [ ] Enable UFW with rate-limited SSH + deny everything else
>   → I'll show you what's listening and ask which ports to keep open
>
> **E. Provider-level Firewall**
> *(Only shown for AWS/Hetzner/OVH)*
> [For AWS Lightsail]: ⚠️ Lightsail has its own firewall that overrides UFW: if we change the SSH port, we MUST update it here too or you'll lose access.
> [For EC2]: Security Groups also need to be updated.
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
> - [ ] Kernel sysctl hardening (block redirects, enable SYN cookies, prevent spoofing)
> - [ ] Disable core dumps (prevents secrets leaking from memory dumps)
> - [ ] Verify AppArmor is enforcing
> - [ ] Disable unused system services (avahi, cups, bluetooth)
> - [ ] Lock root password
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

```bash
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

Apply only the options the user selected:

```bash
# If changing port:
sed -i "s/^#*Port .*/Port NEW_PORT/" /etc/ssh/sshd_config

# If disabling root login:
sed -i "s/^#*PermitRootLogin .*/PermitRootLogin no/" /etc/ssh/sshd_config

# If disabling password auth:
sed -i "s/^#*PasswordAuthentication .*/PasswordAuthentication no/" /etc/ssh/sshd_config

# Always add these if not present:
grep -q "^PubkeyAuthentication" /etc/ssh/sshd_config \
  || echo "PubkeyAuthentication yes" >> /etc/ssh/sshd_config
grep -q "^X11Forwarding" /etc/ssh/sshd_config \
  || echo "X11Forwarding no" >> /etc/ssh/sshd_config
grep -q "^MaxAuthTries" /etc/ssh/sshd_config \
  || echo "MaxAuthTries 3" >> /etc/ssh/sshd_config
grep -q "^LoginGraceTime" /etc/ssh/sshd_config \
  || echo "LoginGraceTime 20" >> /etc/ssh/sshd_config

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

⚠️ Lightsail's firewall is separate from UFW and must be updated: otherwise the new SSH port is blocked at the provider level.

```bash
# If no AWS CLI, install it
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip && sudo ./aws/install
aws configure
```

If user wants to use the console instead:
> Lightsail Console → your instance → Networking → Firewall → Add rule → Custom TCP, port NEW_PORT

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
maxretry = 3
bantime  = 24h
EOF

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
cat >> /etc/sysctl.d/99-hardening.conf << 'EOF'
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.log_martians = 1
EOF
sysctl -p /etc/sysctl.d/99-hardening.conf

# Disable core dumps
echo "* hard core 0" >> /etc/security/limits.conf
echo "fs.suid_dumpable = 0" >> /etc/sysctl.d/99-hardening.conf
sysctl -p /etc/sysctl.d/99-hardening.conf

# Lock down shared memory
echo "tmpfs /run/shm tmpfs defaults,noexec,nosuid 0 0" >> /etc/fstab

# Lock root password
passwd -l root

# Restrict su to sudo group
dpkg-statoverride --update --add root sudo 4750 /bin/su

# AppArmor: ensure enforcing
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
- [ ] UFW enabled with rate-limited SSH, all unused ports closed *(if selected)*
- [ ] Provider-level firewall updated *(if selected)*
- [ ] fail2ban installed and protecting SSH *(if selected)*
- [ ] Unattended security updates enabled *(if selected)*
- [ ] Kernel sysctl hardening applied *(if selected)*
- [ ] Core dumps disabled *(if selected)*
- [ ] AppArmor enforcing *(if selected)*
- [ ] Root password locked *(if selected)*
- [ ] Login banner configured *(if selected)*
- [ ] Lynis audit completed *(if selected)*
- [ ] ClamAV installed and scanning *(if selected)*
- [ ] SSH 2FA tested and working in new terminal *(if selected)*
- [ ] **Verified: can SSH in on correct port as non-root user from a fresh terminal**
