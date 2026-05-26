# Rescue Mode Recovery

Use this when SSH is inaccessible — UFW blocked the old port before the new one was confirmed, sshd is misconfigured, or similar. Each provider has an out-of-band access method that bypasses the server's network firewall entirely.

**Common fix once inside rescue/console:**

```bash
# Option A — disable UFW so port 22 is open again, then reboot back to normal OS:
ufw disable

# Option B — if sshd is misconfigured, restore the backup:
# (Hetzner/DO: if /etc/ssh/sshd_config.bak exists from Phase 6)
cp /etc/ssh/sshd_config.bak /etc/ssh/sshd_config
systemctl restart sshd

# Option C — if iptables rules are broken, flush them and let UFW rebuild:
iptables -F
iptables -X
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT
ip6tables -F && ip6tables -X
ip6tables -P INPUT ACCEPT
ip6tables -P FORWARD ACCEPT
ip6tables -P OUTPUT ACCEPT
ufw disable   # prevent UFW from re-adding rules on next boot until you're back in
reboot
```

After rebooting back to normal OS and regaining SSH access on port 22, redo the hardening steps in the correct order from Phase 3.

---

## Hetzner

Hetzner has a full Linux rescue environment that boots instead of your OS.

```bash
# From your LOCAL machine (hcloud CLI required, authenticated):

# 1. Enable rescue mode (boots a minimal Linux on next reset)
hcloud server enable-rescue <server-name> --type linux64 --ssh-key <key-name>

# 2. Hard reset the server into rescue
hcloud server reset <server-name>

# 3. Wait ~30s, then SSH into rescue as root (the server's normal OS is not running)
SERVER_IP=$(hcloud server ip <server-name>)
ssh-keygen -R "$SERVER_IP" 2>/dev/null   # clear stale host key from previous session
ssh root@"$SERVER_IP"
```

Inside rescue (your normal OS disk is at `/dev/sda` or similar — check with `lsblk`):

```bash
# Mount your OS partition:
mount /dev/sda1 /mnt   # adjust partition if needed — check with lsblk
# chroot in to run commands as if you were on the normal OS:
chroot /mnt

# Fix UFW / sshd (see "Common fix" block above), then exit chroot and reboot:
exit
umount /mnt
reboot
```

After reboot the server returns to its normal OS. Rescue mode is automatically disabled.

---

## DigitalOcean

DigitalOcean provides a **browser-based console** directly in the control panel — it is not affected by the droplet's network firewall or SSH config.

1. Go to cloud.digitalocean.com → Droplets → select your droplet → **Access** tab → **Launch Droplet Console**
2. Log in as root (uses password auth — the root password was set at droplet creation, or reset via the console)
3. Fix UFW / sshd (see "Common fix" block above)
4. Close the console — SSH should now be accessible again

**If you don't know the root password:**
- In the same **Access** tab → **Reset Root Password** → a new password is emailed to your account and injected on next reboot
- After the reboot, use the Droplet Console to log in with the new password and fix the config

**Recovery Mode (if the OS is unbootable):**
- Power off the droplet → go to **Backups & Snapshots** → restore a snapshot from before the change
- Or contact DigitalOcean support for assisted recovery

---

## AWS EC2

**Option A — EC2 Instance Connect (quickest, if the instance launched with it enabled):**

```bash
aws ec2-instance-connect send-ssh-public-key \
  --region <region> \
  --instance-id <instance-id> \
  --os-user ubuntu \
  --ssh-public-key file://~/.ssh/id_ed25519.pub

ssh -o "IdentitiesOnly=yes" -i ~/.ssh/id_ed25519 ubuntu@<PUBLIC_IP>
```

**Option B — AWS Systems Manager Session Manager (no open ports needed):**

1. AWS Console → EC2 → Instances → select instance → **Connect** → **Session Manager** tab → **Connect**
2. This opens a terminal in the browser. Fix UFW / sshd (see "Common fix" block above).

*SSM Agent must be installed and running on the instance. It is pre-installed on Amazon Linux and official Ubuntu AMIs from AWS.*

**Option C — Detach and repair EBS root volume (nuclear option):**

1. Stop (not terminate) the locked-out instance
2. Detach its root EBS volume
3. Attach the volume to a healthy instance as `/dev/xvdf` (or similar)
4. SSH into the healthy instance, mount the volume, fix the config:
   ```bash
   sudo mount /dev/xvdf1 /mnt
   sudo chroot /mnt
   # fix UFW / sshd here
   exit && sudo umount /mnt
   ```
5. Detach from the healthy instance, reattach to the original as `/dev/xvda`, start it

---

## OVH

OVH provides a rescue mode for both VPS and dedicated servers.

**VPS:**

1. OVH Manager → Bare Metal Cloud → VPS → select VPS → **Reboot my VPS** → choose **Rescue mode**
2. OVH sends the rescue root password to your account email
3. SSH into the VPS with that password — it boots into a live Debian environment
4. Mount your OS partition and fix the config:
   ```bash
   lsblk   # identify your OS partition, usually /dev/sda1
   mount /dev/sda1 /mnt
   chroot /mnt
   # fix UFW / sshd here
   exit && umount /mnt
   ```
5. OVH Manager → reboot back to **normal mode**

**Dedicated Server:**

1. OVH Manager → Bare Metal Cloud → Dedicated Servers → select server → **Boot** → change netboot to **Rescue-pro**
2. Hard reboot the server from the same panel
3. Follow the same mount + chroot steps as VPS above

---

## Other Providers

Most providers offer at least one of:

- **Browser-based console / VNC / KVM-over-IP** — bypasses SSH entirely; look for "Console", "KVM", "VNC", or "Emergency console" in the control panel
- **Rescue / recovery mode** — boots a minimal live OS; look for "Rescue mode", "Recovery mode", or "Boot into recovery"
- **Serial console** — terminal over a serial port, not affected by iptables/UFW

**What to ask if you can't find it:**

> "My SSH is locked out. Does this provider offer a browser console, rescue mode, or out-of-band access?"

Once inside via any of these methods, apply the "Common fix" commands at the top of this file, then reboot into the normal OS.
