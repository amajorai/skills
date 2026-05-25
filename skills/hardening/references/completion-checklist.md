# Final Verification & Completion Checklist

## Verification commands

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

## Completion checklist

Generate based only on what the user selected:

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
- [ ] Provider-level firewall updated — incl. new SSH port + 80/443 *(if selected)*
- [ ] fail2ban installed and protecting SSH (port matches new SSH port) *(if selected)*
- [ ] Unattended security updates enabled *(if selected)*
- [ ] Kernel sysctl hardening applied *(if selected)*
- [ ] Filesystem protections applied *(if selected)*
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
