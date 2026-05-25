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

## OVH

```bash
curl -fsSL https://raw.githubusercontent.com/ovh/ovhcloud-cli/main/install.sh | sh
ovhcloud login
```

> OVH firewall CLI varies by product type. If the CLI doesn't work for your product, go to:
> OVH Manager → Bare Metal Cloud → IP → Firewall → add rule for NEW_PORT.
