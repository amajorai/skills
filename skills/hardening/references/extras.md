# Optional Extras

## Lynis — full security audit

```bash
apt-get install -y lynis
lynis audit system
# Aim for hardening index 70+. Review suggestions at the bottom of the report.
```

## ClamAV — antivirus with daily scans

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

## SSH 2FA — TOTP authenticator

```bash
apt-get install -y libpam-google-authenticator

# Run as the non-root user — each user sets up their own TOTP
google-authenticator
# Scan the QR code with Google Authenticator / Authy, then answer y to all prompts

# Enable in PAM
sed -i 's/@include common-auth/auth required pam_google_authenticator.so\n@include common-auth/' \
  /etc/pam.d/sshd

# Enable challenge-response in sshd
sed -i 's/^#*KbdInteractiveAuthentication.*/KbdInteractiveAuthentication yes/' /etc/ssh/sshd_config
grep -q "^AuthenticationMethods" /etc/ssh/sshd_config \
  || echo "AuthenticationMethods publickey,keyboard-interactive" >> /etc/ssh/sshd_config

sshd -t && systemctl restart sshd
```

> ⚠️ Test 2FA in a **new terminal** before closing your current session. If TOTP is misconfigured and you close your session, you are locked out.
