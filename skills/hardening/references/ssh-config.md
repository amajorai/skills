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

```bash
# Change SSH port
echo "Port NEW_PORT" >> /etc/ssh/sshd_config.d/hardened.conf

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
