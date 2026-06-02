---
name: hardening
description: Harden a self-hosted Linux server. Detects current state, runs a full interview to understand what the user wants, warns about edge cases (no SSH keys, password-only auth, provider firewalls), then implements only the selected steps in a safe order.
argument-hint: [server IP or hostname, optional]
---

# Hardening

You are hardening a Linux VPS. **Do not implement anything until the interview is complete and the user has confirmed their selections.** The order matters: detect state → interview → confirm → implement.

**Target:** {{args}}

If no target was supplied above (empty `{{args}}`), ask the user for the server IP or hostname before any SSH-dependent phase, and use that value (call it `HOST`) wherever a target is needed below.


## Privacy Rule — Redact Sensitive Values by Default

Never print the following in chat, even if you just ran a command that returned them:

- Server IP addresses or hostnames
- The chosen SSH port (custom/randomized port) — see "Redact by omission" below
- API keys, personal access tokens, or bearer tokens
- SSH private keys, fingerprints, or TOTP secret keys / QR code seeds
- Passwords or passphrases
- Cloud provider credentials or account IDs
- Any value the user passed as a secret placeholder (e.g. `<your-do-api-token>`)

When one of these values would naturally appear in your response, replace it with a placeholder and offer to share on request. Examples:

> SSH key generated and installed. *(The key fingerprint is not shown in chat — ask if you need it.)*

> Cloud firewall updated successfully. *(The API token is not shown in chat.)*

> 2FA configured. *(The TOTP secret and QR code are not shown in chat — scan the QR code from your terminal directly.)*

If the user explicitly asks — "show me the IP", "what's the token?", "give me the full command with the real values" — then output the real value in that one response only. Do not repeat it in follow-up messages unless asked again.

### Use the `vibe-target` SSH alias — never the raw IP

The single biggest leak is embedding the literal IP into every `ssh user@IP` command and every verification-subagent prompt. Avoid it entirely: **write the IP into a local SSH alias once, then reference the alias `vibe-target` everywhere.** Do this on your **local machine** right after you have the `HOST`, `USER`, and current port:

```bash
# Append a Host block to ~/.ssh/config. The IP is written to the file once
# and never echoed again — every later command uses the alias, not the IP.
mkdir -p ~/.ssh && chmod 700 ~/.ssh
cat >> ~/.ssh/config << EOF

Host vibe-target
    HostName $HOST
    User $USER_ON_SERVER
    Port 22
    StrictHostKeyChecking accept-new
EOF
chmod 600 ~/.ssh/config
```

From here on, **all** SSH/scp commands and **all** subagent prompts use `ssh vibe-target …` / `scp … vibe-target:…` — never `user@<IP>`. After the SSH-port change is confirmed (Phase 7), update the alias's `Port` line to the new port (`sed -i 's/^    Port .*/    Port NEW_PORT/' ~/.ssh/config`) so subsequent connections keep working without ever printing the port.

### Redact by omission — never the "value (kept private)" anti-pattern

Do **not** write things like `New SSH port will be 56224 (kept private — ask me if you need it)`. That announces the exact value you claim to be hiding. Redact by omission instead:

> A new random SSH port was chosen and applied. *(The port is not shown in chat — ask if you need it.)*

SSH **public** keys are the one exception — they are public by design and are needed for UI registration, so they may be shown when a step requires pasting them somewhere.


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
curl -s --max-time 2 http://169.254.169.254/metadata/v1/id 2>/dev/null && echo "IS_DIGITALOCEAN" || true
which fail2ban-client ufw clamav lynis auditd 2>/dev/null
aa-status 2>/dev/null | head -3 || echo "AppArmor not active"
systemctl is-active auditd 2>/dev/null && (auditctl -l 2>/dev/null | grep -q . && echo "AUDIT_RULES_PRESENT" || echo "AUDIT_NO_CUSTOM_RULES") || echo "AUDITD_NOT_RUNNING"
which docker 2>/dev/null && docker ps --format '{{.Names}}: {{.Ports}}' 2>/dev/null && echo "DOCKER_PRESENT" || echo "NO_DOCKER"
grep -rE "NOPASSWD" /etc/sudoers /etc/sudoers.d/ 2>/dev/null && echo "NOPASSWD_PRESENT" || echo "NO_NOPASSWD"
ls /etc/ssh/sshd_config.d/ 2>/dev/null
# Is sshd started via systemd socket activation? (Ubuntu 22.10+/24.04 default.)
# If so, the `Port` directive in sshd_config is IGNORED and `systemctl restart
# sshd` does NOT change the listening port — see Phase 7.
systemctl is-active ssh.socket 2>/dev/null | grep -q '^active$' && echo "SSH_SOCKET_ACTIVE" || echo "SSH_SOCKET_INACTIVE"
```

Note: SSH port, password auth state, SSH key presence, cloud provider, open ports, Docker, auditd rules, NOPASSWD, **and whether sshd uses socket activation** (`SSH_SOCKET_ACTIVE`).


## Phase 2: Full Interview

First, show the server summary as a single text message (do not use AskUserQuestion for this):

> **Server hardening setup — I've scanned your server. Here's what I found:**
> - **SSH port:** [detected port]
> - **Password auth:** [on / off]
> - **SSH keys:** [present / not found]
> - **Running as:** [root / user]
> - **Provider:** [AWS Lightsail / EC2 / Hetzner / DigitalOcean / OVH / unknown]
> - **Open ports:** [list]
> - **Docker:** [present with ports: X,Y / not installed]
> - **NOPASSWD sudo:** [found / none]
> - **sshd socket activation:** [active — port set via ssh.socket / inactive]
>
> I'll ask 5 quick questions, then plan and implement everything in one pass.

Then use **AskUserQuestion** for each of the 5 groups below. Each question uses `multiSelect: true`. Present them one at a time, in order — do not combine them into one call. Show or hide individual options based on the Phase 1 findings as noted.

---

**Question 1 — Account & Access**

Options (include only those that apply based on Phase 1):
- "Set up SSH keys (generate + install)" — *only if NO_KEYS detected*
- "Create non-root sudo user" — *only if running as root; follow up for username if selected*
- "Disable password authentication" — *only if KEYS_PRESENT; omit if no keys (would cause lockout)*
- "Disable root login via SSH"

---

**Question 2 — SSH Hardening**

Options (always show all 4):
- "Randomize SSH port (I'll auto-generate a secure random port)"
- "Restrict SSH to specific users (AllowUsers)" — *follow up for usernames if selected*
- "Disable TCP/agent forwarding + drop idle sessions (~10 min)"
- "Modern ciphers only (curve25519, aes256-gcm + chacha20-poly1305, ETM MACs)"

---

**Question 3 — Firewall & Intrusion Prevention**

Options (always show all 4):
- "Enable UFW (deny all inbound, rate-limit SSH)"
- "Configure [PROVIDER] firewall via CLI" — *replace [PROVIDER] with detected name; if unknown show "Configure provider firewall (tell me which one)"*
- "fail2ban (auto-ban brute-force SSH IPs)"
- "Unattended security updates (auto-patch, no reboot)"

---

**Question 4 — System Hardening**

Options (always show all 4; if NOPASSWD_PRESENT was detected, replace the 4th with "Review/tighten sudo NOPASSWD policy"):
- "Kernel hardening (sysctl: SYN-flood, anti-spoof, ASLR, ptrace/kptr restrict)"
- "Core dumps disabled + filesystem protections (hardlinks, symlinks, fifos)"
- "AppArmor enforcing + disable unused services (avahi, cups, bluetooth)"
- "auditd custom ruleset + lock root password + password policy (SHA512, UMASK 022)"

---

**Question 5 — Extras & Monitoring**

Options (always show all 4):
- "Login banner (standard legal warning — or provide custom text)"
- "Lynis security audit (runs after hardening, targets score ≥70)"
- "ClamAV antivirus daemon (daily scans at 3am, ~50 MB RAM overhead)"
- "SSH 2FA with TOTP ⚠️ must verify in a fresh session before closing this one"

---

After collecting answers, summarize the full plan in plain text and ask: **"Ready to proceed?"** Do not begin any implementation until the user confirms.


## Phase 2.5: Plan

Call `EnterPlanMode` (or switch to the strongest available model with `/model opus` if unavailable).

Draft the implementation plan:
1. List every selected step in safe execution order, respecting all dependencies (provider FW before UFW enable, UFW before sshd/socket restart, etc.)
2. Note every safety gate (subagent SSH+sudo verification checkpoints, backup steps, port 22 transition windows)
3. Record the randomly generated SSH port that will be used (generate it now: `shuf -i 49152-65535 -n 1` — store it as `NEW_PORT` for the rest of the run). **Do not print `NEW_PORT` in the plan or anywhere in chat** — redact by omission (see the Privacy Rule). Refer to it as "a new random SSH port".
4. **Lockout cross-check.** If the selections include "disable root login" and/or "lock root password", confirm the plan creates a non-root admin user with a verified escalation path (Phase 5 `SUDO_OK`) **before** those steps. If "tighten/remove NOPASSWD" (Q4) is also selected, confirm the admin user has a password set — otherwise removing NOPASSWD strips its only escalation path. Surface any such dependency in the plan text.
5. **Socket-activation note.** If `SSH_SOCKET_ACTIVE` was detected and a port change is selected, the plan must drive the port via the `ssh.socket` drop-in (Phase 7), not via a plain `Port` directive + `restart sshd`.

Present the plan. Do not proceed until the user approves. Call `ExitPlanMode` after approval.


## Phase 3: Pre-flight Safety Checks

Before writing any config file, run these checks based on selections:

**If "disable password auth" selected:**
Spawn a subagent to verify key auth works before proceeding (this is a separate shell — the truth source):

```
Agent({
  description: "Verify SSH key auth before disabling password auth",
  prompt: "Run: ssh -o StrictHostKeyChecking=no -o PasswordAuthentication=no -o ConnectTimeout=15 vibe-target echo 'KEY_AUTH_OK'. (vibe-target is the alias from the Privacy Rule — it hides the IP.) Report SUCCESS if output contains KEY_AUTH_OK, otherwise FAILURE with the exact error."
})
```

Do not disable password auth until the subagent reports SUCCESS.

**If "change SSH port" selected:**

> ❌ **LOCKOUT RULE — read this before touching anything.**
>
> The only safe sequence when changing the SSH port is:
> 1. Open NEW_PORT in UFW — **while keeping port 22 open in UFW too**
> 2. Open NEW_PORT in the provider firewall — **while keeping port 22 open there too**
> 3. Restart sshd
> 4. Ask the user to verify in a **NEW terminal**: `ssh -p NEW_PORT vibe-target echo "ok"` (the alias hides the IP)
> 5. Only after step 4 succeeds: remove port 22 from UFW
> 6. Only after step 5: remove port 22 from the provider firewall
>
> **Never enable UFW or restart sshd without port 22 still open at both layers.** Closing port 22 before the new port is confirmed in a live terminal = lockout. If this happens, rescue mode is required — see [references/rescue-mode.md](references/rescue-mode.md).
>
> This sequence is enforced in Phases 7 and 8. Do not deviate from it.

- Lightsail: port 22 must stay open in the Lightsail firewall until new port is confirmed.
- EC2: port 22 must stay open in the Security Group until new port is confirmed.
- DigitalOcean: port 22 must stay open in the Cloud Firewall until new port is confirmed.

**If "disable root login" and/or "lock root password" selected:**

> ❌ **ROOT LOCKOUT RULE — read before disabling root access.**
>
> Disabling root SSH login (Phase 6) and locking the root password (Phase 11)
> are only safe once a **separate, non-root admin session is verified to BOTH
> (a) SSH in AND (b) run `sudo`.** The Phase 5 subagent check (`SUDO_OK`) is that
> gate. Until it passes:
> 1. Do **not** add `PermitRootLogin no` to the SSH drop-in.
> 2. Do **not** run `passwd -l root`.
>
> The trap: a user created with `--disabled-password` and no NOPASSWD entry can
> log in but cannot escalate — disabling root then leaves no admin path and
> requires the provider's web console to recover.
>
> **Lockout combo to flag in the plan:** "disable root login" + "lock root
> password" + a non-root user that lacks working sudo = total admin lockout.
> If both are selected, confirm the admin user's `SUDO_OK` first and surface the
> dependency explicitly.

**If no keys and user skipped key setup:**
> ❌ Cannot safely disable password auth. Either help them set up keys first or skip that step.


## Phase 4: SSH Key Setup (if selected: A)

Run on the **local machine** (use the captured `HOST` value, or `{{args}}` if it was supplied):

> Set up the `vibe-target` SSH alias first (see the Privacy Rule) so none of the
> commands below print the raw IP. They all reference `vibe-target`.

```bash
ssh-keygen -t ed25519 -C "vps-hardening" -f ~/.ssh/id_ed25519_vps
ssh-copy-id -i ~/.ssh/id_ed25519_vps.pub vibe-target
```

On Windows, run these via Git Bash or WSL. Stock Windows OpenSSH has no `ssh-copy-id` — append the key manually instead:

```bash
cat ~/.ssh/id_ed25519_vps.pub | ssh vibe-target "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

Then spawn a subagent to verify key auth from a clean shell:

```
Agent({
  description: "Verify new SSH key works",
  prompt: "Run: ssh -o StrictHostKeyChecking=no -o PasswordAuthentication=no -o ConnectTimeout=15 -i ~/.ssh/id_ed25519_vps vibe-target echo 'KEY_OK'. (vibe-target is the alias from the Privacy Rule — it hides the IP.) Report SUCCESS if output contains KEY_OK, otherwise FAILURE with the exact error."
})
```

Do not continue until the subagent reports SUCCESS.


## Phase 5: Create Non-Root Sudo User (if selected: B)

> ❌ **The #1 cause of "I hardened my box and got locked out of admin":** a non-root
> user is created **without a working escalation path**, then root login is
> disabled (Phase 6) and/or the root password is locked (Phase 11) — leaving
> nobody who can run `sudo`. Recovery then needs the provider's web console.
> **A non-root user is not "ready" until it can both SSH in AND run `sudo`.**

`adduser` without `--disabled-password` prompts interactively for a password and
will hang in a non-interactive run. Create the user non-interactively, and give it
a working sudo path. On a **key-only** server (the usual case — no password to
type) use passwordless sudo:

```bash
adduser --disabled-password --gecos "" USERNAME
usermod -aG sudo USERNAME
mkdir -p /home/USERNAME/.ssh
cp /root/.ssh/authorized_keys /home/USERNAME/.ssh/
chown -R USERNAME:USERNAME /home/USERNAME/.ssh
chmod 700 /home/USERNAME/.ssh && chmod 600 /home/USERNAME/.ssh/authorized_keys

# Working escalation path. Key-only server → passwordless sudo (there is no
# password to enter). Mirrors the cloud-init `ubuntu ALL=(ALL) NOPASSWD:ALL`
# default. visudo -c validates before it can take effect.
echo "USERNAME ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/90-USERNAME-vibe
chmod 440 /etc/sudoers.d/90-USERNAME-vibe
visudo -c
```

> **Want password-protected sudo instead of NOPASSWD?** Set a real password with
> `passwd USERNAME` and skip the sudoers drop-in. Do **not** leave a
> `--disabled-password` user with no NOPASSWD entry — it cannot escalate at all.
> Note the interplay with Question 4's "tighten NOPASSWD" option: if the user
> wants NOPASSWD removed server-wide, they **must** set a password here, or they
> lose the only escalation path.

Spawn a subagent to verify the new user can SSH in **and escalate** (separate
shell, clean state — this is the truth source, not just connectivity):

```
Agent({
  description: "Verify SSH + sudo as new non-root user",
  prompt: "Run: ssh -o StrictHostKeyChecking=no -o ConnectTimeout=15 -l USERNAME vibe-target 'whoami && sudo -n true && echo SUDO_OK'. (vibe-target is the alias to the server; -l USERNAME logs in as the new user. Usernames are not secret — only the IP is, and the alias hides it.) Report SUCCESS only if the output contains SUDO_OK. Otherwise report FAILURE with the exact error — FAILURE means the user exists but cannot run sudo (e.g. created with --disabled-password and no NOPASSWD entry, or a password is required)."
})
```

Do not continue until the subagent reports SUCCESS (`SUDO_OK`). A user that can
SSH in but cannot `sudo` is **not** a safe replacement for root — treat it as a
blocker and fix the escalation path before any Phase 6 / Phase 11 step.

Once `SUDO_OK` is confirmed, point the alias at the new admin user so every later
step connects as it (not root):

```bash
sed -i 's/^    User .*/    User USERNAME/' ~/.ssh/config
```


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

> ⚠️ **Socket activation (`SSH_SOCKET_ACTIVE` from Phase 1):** if you are changing
> the port, the `Port` directive in the drop-in is **not enough** — on Ubuntu
> 22.10+/24.04 the listening port is owned by `ssh.socket`, and `Port` is ignored.
> Phase 7 drives the port through a `ssh.socket` drop-in instead. Still include
> `Port NEW_PORT` in the sshd drop-in (it is correct if socket activation is ever
> disabled, and harmless otherwise).

**Do not restart sshd here — do it after UFW is configured.**


## Phase 7: UFW Firewall (if selected: D)

```bash
apt-get install -y ufw
ufw --force reset
ufw default deny incoming
ufw default allow outgoing
```

⚠️ If Docker is installed: Docker writes its own iptables rules into the `DOCKER` / `DOCKER-USER` chains, evaluated **before** UFW's FORWARD rules. Containers with `-p PORT:PORT` are exposed even when UFW says DENY. **Do not edit `/etc/ufw/before.rules` for Docker — use `DOCKER-USER` chain rules or rebind containers to `127.0.0.1`.**

Auto-detect Docker-published ports and decide automatically which ones to keep open based on what the containers are (web servers → allow 80/443, databases bound to 127.0.0.1 → no rule needed, management UIs → allow with restriction):

```bash
docker ps --format 'table {{.Names}}\t{{.Ports}}' 2>/dev/null
```

Only ask the user if a port's purpose is ambiguous. For clearly internal ports (5432, 3306, 6379, 27017, etc.), skip the UFW allow — do not open them.

Add all rules — **always keeping both port 22 and NEW_PORT open if a port change was selected:**

```bash
# If changing SSH port: keep BOTH open until new port is confirmed
ufw limit NEW_PORT/tcp comment "SSH (new)"
ufw allow 22/tcp    comment "SSH (old — remove ONLY after new port confirmed)"

# Web/app ports identified above:
# ufw allow <PORT>/tcp comment "<label>"

ufw --force enable
ufw status verbose
```

**⛔ STOP HERE if provider firewall selected.** Run Phase 8 Part 1 (open NEW_PORT there, keep port 22 open) before restarting sshd. Return here after Phase 8 Part 1 is done.

Now apply the new port. **The command depends on whether sshd uses socket activation** (`SSH_SOCKET_ACTIVE` from Phase 1):

**Case A — socket activation active (`SSH_SOCKET_ACTIVE`, e.g. Ubuntu 22.10+/24.04):**
`systemctl restart sshd` will **not** change the port — `ssh.socket` owns it. Drive the port through a socket drop-in that listens on **both** the old and new port during the transition (this is the keep-22-open safety rule, enforced at the socket layer):

```bash
mkdir -p /etc/systemd/system/ssh.socket.d
cat > /etc/systemd/system/ssh.socket.d/listen.conf << EOF
[Socket]
ListenStream=
ListenStream=22
ListenStream=NEW_PORT
EOF
systemctl daemon-reload
systemctl restart ssh.socket
ss -tlnp | grep ssh    # should now show BOTH 22 and NEW_PORT listening
```

**Case B — no socket activation (`SSH_SOCKET_INACTIVE`):** the `Port` directive in the drop-in is authoritative. **It must list BOTH ports** — a lone `Port NEW_PORT` makes sshd listen on the new port *only* and drops 22, which is exactly how you get locked out. The `references/ssh-config.md` port block already writes both `Port NEW_PORT` and `Port 22`; confirm both lines are present, then restart:

```bash
grep -E '^Port (22|NEW_PORT)$' /etc/ssh/sshd_config.d/hardened.conf   # expect BOTH lines
# If `Port 22` is missing, add it before restarting:
grep -q '^Port 22$' /etc/ssh/sshd_config.d/hardened.conf || echo "Port 22" >> /etc/ssh/sshd_config.d/hardened.conf
sshd -t && { systemctl restart ssh 2>/dev/null || systemctl restart sshd; }
ss -tlnp | grep ssh    # should show BOTH 22 and NEW_PORT listening
```

Spawn a subagent to verify the new SSH port from a completely separate shell (this is the source of truth — not just checking if sshd is listening, but actually connecting through all firewall layers):

```
Agent({
  description: "Verify SSH on new port through all firewall layers",
  prompt: "Run: ssh -o StrictHostKeyChecking=no -o ConnectTimeout=15 -p NEW_PORT vibe-target echo 'NEW_PORT_OK'. (vibe-target is the alias from the Privacy Rule — it hides the IP.) Report SUCCESS if output contains NEW_PORT_OK, otherwise FAILURE with the exact error. This test goes through UFW and the provider firewall — a failure means one of those layers is still blocking the port, or (on Ubuntu 24.04) ssh.socket was not updated."
})
```

If the subagent reports FAILURE: **do NOT remove port 22**. Diagnose — check `systemctl status ssh ssh.socket sshd`, `ss -tlnp | grep ssh` (is NEW_PORT actually listening? if only 22 shows, the socket drop-in did not apply), `ufw status verbose`, and whether the provider firewall has the new port open.

**Only after the subagent reports SUCCESS**, point the alias at the new port and close port 22.

First, on your **local machine**, update the alias so later connections use the new port (no IP/port printed in chat):

```bash
# LOCAL machine — edits your local ~/.ssh/config, not the server's:
sed -i 's/^    Port .*/    Port NEW_PORT/' ~/.ssh/config
```

Then, on the **server**, stop listening on 22 and remove it from UFW. Run the branch for your case:

```bash
#  - Case A (socket activation): drop the ListenStream=22 line and reload the socket:
if systemctl is-active ssh.socket 2>/dev/null | grep -q '^active$'; then
  sed -i '/^ListenStream=22$/d' /etc/systemd/system/ssh.socket.d/listen.conf
  systemctl daemon-reload && systemctl restart ssh.socket
else
  #  - Case B (no socket activation): drop the `Port 22` line and restart the service:
  sed -i '/^Port 22$/d' /etc/ssh/sshd_config.d/hardened.conf
  sshd -t && { systemctl restart ssh 2>/dev/null || systemctl restart sshd; }
fi
ss -tlnp | grep ssh    # confirm 22 is gone and NEW_PORT remains

ufw delete allow 22/tcp
ufw status verbose
```

Then return to Phase 8 Part 2 to remove port 22 from the provider firewall.


## Phase 8: Provider Firewall (if selected: E)

> ⚠️ **If changing SSH port:** this phase runs in **two parts** — one before sshd restarts and one after the new port is confirmed. If you were sent here from the Phase 7 STOP gate, you are in Part 1.

**Part 1 — before sshd restart:** Open NEW_PORT at the provider level. Keep port 22 open. Then return to Phase 7 to restart sshd and verify.

**Part 2 — after new port confirmed in a fresh terminal:** Remove port 22 from the provider firewall.

For CLI commands to configure AWS Lightsail, EC2 Security Groups, Hetzner, DigitalOcean, and OVH firewalls (both the open and the revoke commands), see [references/provider-firewall.md](references/provider-firewall.md).

If the user chose **"Other provider"**: ask which provider/control panel they use, then research the correct CLI or API commands on the spot and walk them through it. Do not skip this step — every provider with an edge-level firewall must be updated.

If the worst happens and you get locked out despite these precautions, see [references/rescue-mode.md](references/rescue-mode.md) for provider-by-provider recovery steps.


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

# Apply the config. With socket activation, per-connection sshd instances read the
# config fresh, so a service reload may not exist — fall back gracefully.
sshd -t && { systemctl reload ssh 2>/dev/null || systemctl reload sshd 2>/dev/null \
  || systemctl restart ssh.socket 2>/dev/null || true; }
```


## Phase 13: Optional Extras (if selected: J)

For full installation commands for Lynis, ClamAV, and SSH 2FA (TOTP), see [references/extras.md](references/extras.md).

- **Lynis**: full security audit; aim for hardening index 70+
- **ClamAV**: antivirus daemon with daily cron scan at 3am
- **SSH 2FA**: `libpam-google-authenticator` + TOTP ⚠️ test in a new terminal before closing your current session


## Phase 14: Final Verification & Completion

For the full verification command block and the per-selection completion checklist, see [references/completion-checklist.md](references/completion-checklist.md).

---

## Recovery: If You Get Locked Out

If SSH access is lost despite these precautions, do not attempt further SSH changes — use rescue mode to access the server out-of-band and fix the config. See [references/rescue-mode.md](references/rescue-mode.md) for step-by-step instructions for Hetzner, DigitalOcean, AWS, OVH, and other providers.
