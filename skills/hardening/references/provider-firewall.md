# Provider Firewall Configuration

## AWS Lightsail

⚠️ Lightsail's firewall is a **separate layer at the provider edge** — independent of UFW and host iptables. Traffic is filtered there *before* it reaches UFW. Key consequences:
- **SSH port change:** new port MUST be added to the Lightsail firewall BEFORE restarting sshd — UFW allowing it is not enough.
- **Public ports:** 80/443 must be open in the Lightsail firewall too, not just UFW/Docker.
- Closing a port in UFW does not close it at the Lightsail edge, and vice versa — keep both layers in sync.

```bash
# Install AWS CLI if needed
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip && sudo ./aws/install
aws configure
```

If the user prefers the console:
> Lightsail Console → instance → Networking → Firewall → Add rule → Custom TCP, port NEW_PORT. Remove port 22 only after new port is confirmed working.

## AWS EC2: Security Groups

```bash
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
SG_ID=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID \
  --query 'Reservations[0].Instances[0].SecurityGroups[0].GroupId' --output text)

aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID --protocol tcp --port NEW_PORT --cidr 0.0.0.0/0

# After confirming the new port works:
aws ec2 revoke-security-group-ingress \
  --group-id $SG_ID --protocol tcp --port 22 --cidr 0.0.0.0/0
```

## Hetzner

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

## DigitalOcean

⚠️ DigitalOcean Cloud Firewalls are enforced at the **network edge**, before traffic reaches the droplet. UFW and the DO firewall are independent layers — both must be updated when changing the SSH port.

```bash
# Install doctl if needed
DOCTL_VERSION=$(curl -s https://api.github.com/repos/digitalocean/doctl/releases/latest \
  | grep '"tag_name"' | cut -d'"' -f4 | tr -d 'v')
curl -fsSL "https://github.com/digitalocean/doctl/releases/latest/download/doctl-${DOCTL_VERSION}-linux-amd64.tar.gz" \
  | tar xz -C /usr/local/bin/

# Authenticate non-interactively with an API token
# Get it from cloud.digitalocean.com → API → Tokens → Generate New Token (read+write)
export DIGITALOCEAN_ACCESS_TOKEN=<your-do-api-token>
doctl auth init --access-token "$DIGITALOCEAN_ACCESS_TOKEN"

# Get this droplet's ID
DROPLET_ID=$(curl -s http://169.254.169.254/metadata/v1/id)

# Check if a firewall already exists for this droplet
EXISTING_FW=$(doctl compute firewall list --format ID,Name --no-header \
  | while read id name; do
      doctl compute firewall get "$id" --format DropletIDs --no-header \
        | grep -q "$DROPLET_ID" && echo "$id $name" && break
    done)

if [ -n "$EXISTING_FW" ]; then
  FW_ID=$(echo "$EXISTING_FW" | awk '{print $1}')
  echo "Found existing firewall: $EXISTING_FW"
  # Add new SSH port rule to the existing firewall
  doctl compute firewall add-rules "$FW_ID" \
    --inbound-rules "protocol:tcp,ports:NEW_PORT,address:0.0.0.0/0,address:::/0"
  # After confirming the new port works, remove the old port 22 rule:
  # doctl compute firewall remove-rules "$FW_ID" \
  #   --inbound-rules "protocol:tcp,ports:22,address:0.0.0.0/0,address:::/0"
else
  # No existing firewall — create one. Adjust port list to match what's needed.
  doctl compute firewall create \
    --name "$(hostname)-fw" \
    --inbound-rules "protocol:tcp,ports:NEW_PORT,address:0.0.0.0/0,address:::/0 protocol:tcp,ports:80,address:0.0.0.0/0,address:::/0 protocol:tcp,ports:443,address:0.0.0.0/0,address:::/0" \
    --outbound-rules "protocol:tcp,ports:all,address:0.0.0.0/0,address:::/0 protocol:udp,ports:all,address:0.0.0.0/0,address:::/0"
  FW_ID=$(doctl compute firewall list --format ID,Name --no-header | grep "$(hostname)-fw" | awk '{print $1}')
  doctl compute firewall add-droplets "$FW_ID" --droplet-ids "$DROPLET_ID"
fi

echo "Firewall updated. Verify in DO Console → Networking → Firewalls before restarting sshd."
```

If the user prefers the console:
> DigitalOcean Console → Networking → Firewalls → select the firewall attached to this droplet → Inbound Rules → add TCP NEW_PORT. Remove port 22 only after the new port is confirmed working.

## OVH

```bash
curl -fsSL https://raw.githubusercontent.com/ovh/ovhcloud-cli/main/install.sh | sh
ovhcloud login
```

> OVH firewall CLI varies by product type. If the CLI doesn't work for your product, go to:
> OVH Manager → Bare Metal Cloud → IP → Firewall → add rule for NEW_PORT.

## Other Provider

If the user's provider is not listed above, ask: **"Which provider or control panel are you using?"** Then:

1. Search for that provider's CLI or API firewall documentation.
2. Walk the user through opening `NEW_PORT` and (after confirmation) closing port 22 using the provider's native tooling.
3. If the provider has no CLI (control-panel only), guide them through the console steps instead.

Key questions to answer for any provider:
- Is the firewall enforced at the **network edge** (before the VM) or only at the **host** level?
- Does changing the SSH port require updating the provider firewall, or is UFW alone sufficient?
- Are rules stateful (most cloud firewalls) or stateless (some bare-metal providers)?

Always confirm the new port is reachable from outside before removing the old port 22 rule.
