# SSH Drop-in Configuration

Write each block to `/etc/ssh/sshd_config.d/hardened.conf`. Include **only** the lines for what the user selected.

## Baseline (always applied when hardening SSH)

```bash
cat > /etc/ssh/sshd_config.d/hardened.conf << 'EOF'
PubkeyAuthentication yes
X11Forwarding no
MaxAuthTries 3
LoginGraceTime 30
EOF
```

## Per-selection options (append as selected)

> ⚠️ **Socket activation (Ubuntu 22.10+/24.04):** a `Port` directive here is IGNORED —
> `ssh.socket` owns the listening port and `systemctl restart sshd` does not change
> it. When `SSH_SOCKET_ACTIVE` was detected, the port is driven via a `ssh.socket`
> drop-in instead (SKILL.md Phase 7). Still write `Port NEW_PORT` below: it is
> correct if socket activation is later disabled, and harmless otherwise. Keep a
> `Port 22` line too until the new port is confirmed, so 22 stays reachable.

```bash
# Change SSH port (see socket-activation caveat above).
# ⚠️ sshd listens ONLY on the ports it is told once ANY Port is set — so a lone
# `Port NEW_PORT` DROPS port 22 on the next restart and can lock you out. During
# the transition (Case B / non-socket hosts) write BOTH ports; remove the 22 line
# only after NEW_PORT is confirmed (SKILL.md Phase 7).
{ echo "Port NEW_PORT"; echo "Port 22"; } >> /etc/ssh/sshd_config.d/hardened.conf

# Disable root login
echo "PermitRootLogin no" >> /etc/ssh/sshd_config.d/hardened.conf

# Disable password auth
echo "PasswordAuthentication no" >> /etc/ssh/sshd_config.d/hardened.conf

# AllowUsers allowlist (space-separated usernames)
echo "AllowUsers USER1 USER2" >> /etc/ssh/sshd_config.d/hardened.conf

# Disable TCP/agent forwarding
echo "AllowTcpForwarding no" >> /etc/ssh/sshd_config.d/hardened.conf

# Drop idle sessions (~10 min)
cat >> /etc/ssh/sshd_config.d/hardened.conf << 'EOF'
ClientAliveInterval 300
ClientAliveCountMax 2
EOF

# Modern crypto only (drops legacy/weak algorithms)
cat >> /etc/ssh/sshd_config.d/hardened.conf << 'EOF'
Ciphers aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group16-sha512
EOF
```

⚠️ Before applying the crypto restrictions, confirm the SSH **client** supports these algorithms. Any OpenSSH from the last several years does. An old or embedded client speaking only legacy ciphers would be locked out.
