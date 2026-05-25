# System Hardening Reference

## Kernel sysctl hardening

```bash
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

# If Docker (or any container/router role) is present, keep IP forwarding on:
# echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.d/99-hardening.conf

sysctl -p /etc/sysctl.d/99-hardening.conf
```

## Filesystem protections (if selected)

```bash
cat >> /etc/sysctl.d/99-hardening.conf << 'EOF'
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
fs.protected_fifos = 1
fs.protected_regular = 2
EOF
sysctl -p /etc/sysctl.d/99-hardening.conf
```

## Disable core dumps (if selected)

Sets both hard and soft limits so neither a user nor a setuid binary can produce a dump that leaks secrets.

```bash
cat >> /etc/security/limits.conf << 'EOF'
* hard core 0
* soft core 0
EOF
echo "fs.suid_dumpable = 0" >> /etc/sysctl.d/99-hardening.conf
sysctl -p /etc/sysctl.d/99-hardening.conf
```

## Password / encryption policy in /etc/login.defs (if selected)

```bash
sed -i 's/^#*ENCRYPT_METHOD.*/ENCRYPT_METHOD SHA512/' /etc/login.defs
grep -q '^ENCRYPT_METHOD' /etc/login.defs || echo 'ENCRYPT_METHOD SHA512' >> /etc/login.defs
sed -i 's/^#*UMASK.*/UMASK 022/' /etc/login.defs
sed -i 's/^#*HOME_MODE.*/HOME_MODE 0750/' /etc/login.defs
grep -q '^HOME_MODE' /etc/login.defs || echo 'HOME_MODE 0750' >> /etc/login.defs

# Apply 0750 to existing home dirs (login.defs only affects newly-created users):
for d in /home/*; do [ -d "$d" ] && chmod 0750 "$d"; done
```

## Lock down shared memory

```bash
echo "tmpfs /run/shm tmpfs defaults,noexec,nosuid 0 0" >> /etc/fstab
```

## Sudo NOPASSWD policy (if applicable)

The cloud-init default is `ubuntu ALL=(ALL) NOPASSWD:ALL`. On a key-only server this is an accepted, documented risk. To require a password instead:

```bash
# Replace NOPASSWD with PASSWD in the cloud-init sudoers file:
sed -i 's/NOPASSWD:ALL/ALL/' /etc/sudoers.d/90-cloud-init-users && visudo -c
```

If keeping NOPASSWD: record it as a known, documented risk in the completion summary.

## auditd — install and configure with a real ruleset

A bare `apt install auditd` leaves the daemon running with an **empty ruleset** — it records nothing actionable. The value is in the rules.

```bash
apt-get install -y auditd audispd-plugins

cat > /etc/audit/rules.d/99-hardening.rules << 'EOF'
# Flush existing rules and set buffer/failure mode
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

> If auditd was already running with the default empty ruleset, this is the fix. The daemon being "active" means nothing without rules.
