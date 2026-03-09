# 03. VPS & Gateway Setup

Provision a cloud server for this setup. A Linux VPS with 1GB of RAM (e.g., DigitalOcean, Hetzner, AWS) is sufficient.

## 1. Initial Server Hardening

Create a non-root user immediately. Run node services as this user to avoid security risks associated with the `root` account.

```bash
# SSH as root
ssh root@<your-server-ip>

# Create a new user
useradd -m -s /bin/bash -G sudo username
passwd username

# Configure SSH for the new user
mkdir -p /home/username/.ssh
chmod 700 /home/username/.ssh
cp ~/.ssh/authorized_keys /home/username/.ssh/
chown -R username:username /home/username/.ssh
```

Verify the SSH connection as `username`, then disable root login:

```bash
sudo nano /etc/ssh/sshd_config

# Set these values:
PermitRootLogin no
PasswordAuthentication no

# Restart SSH
sudo systemctl restart ssh
```

## 2. Install Dependencies

Install Node.js (via `nvm`) and the `Caddy` web server.

```bash
# Install Node.js 22
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.nvm/nvm.sh
nvm install 22
nvm alias default 22

# Install the TypingMind MCP Gateway
npm install -g @typingmind/mcp

# Install Caddy
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

## 3. Configure the MCP Multiplexer

Create the configuration directory and file:

```bash
mkdir -p ~/.config/mcp
nano ~/.config/mcp/mcp-servers.json
```

Paste the configuration below. Replace `<placeholders>` with your keys. 

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@kaspnilsson/mcp-memory-supabase"],
      "env": {
        "SUPABASE_URL": "https://<your-project>.supabase.co",
        "SUPABASE_KEY": "<your-service-role-key>",
        "EMBEDDING_API_KEY": "<openrouter-or-openai-key>"
      }
    },
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@brave/brave-search-mcp-server"],
      "env": {
        "BRAVE_API_KEY": "<your-brave-search-api-key>"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<your-github-pat>"
      }
    },
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    }
  }
}
```

## 4. Secure the Gateway

Generate a secure bearer token for TypingMind:

```bash
openssl rand -hex 32
```

Create a systemd service to manage the Gateway process:

```bash
sudo nano /etc/systemd/system/mcp-connector.service
```

Paste the following, replacing `username` and `<your-token>`:

```ini
[Unit]
Description=TypingMind MCP Connector
After=network.target

[Service]
Type=simple
User=username
Environment=PATH=/home/username/.nvm/versions/node/v22.14.0/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
Environment=MCP_AUTH_TOKEN=<your-token>
ExecStart=/home/username/.nvm/versions/node/v22.14.0/bin/mcp-server --config /home/username/.config/mcp/mcp-servers.json --port 50880
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now mcp-connector
```

## 5. Configure HTTPS via Caddy

Point your domain name (e.g., `mcp.domain.com`) to the server IP. Caddy will manage the SSL certificate automatically.

Edit `/etc/caddy/Caddyfile`:

```caddyfile
mcp.yourdomain.com {
    reverse_proxy localhost:50880
}
```

Apply changes:

```bash
sudo systemctl restart caddy
```

## 6. Verification

Run this 'curl' command from your local machine to verify the setup:

```bash
curl -H "Authorization: Bearer <your-token>" https://mcp.yourdomain.com/clients
```

A JSON payload listing the active clients confirms success.

## 7. Maintenance & Troubleshooting

Use these commands to manage the Gateway and inspect logs.

### Check Service Status
```bash
sudo systemctl status mcp-connector
```

### Inspect Logs
View the last 50 lines of logs or follow them in real-time:
```bash
# View recent logs
journalctl -u mcp-connector -n 50

# Follow logs live
journalctl -u mcp-connector -f
```

### Restart after Config Changes
If you update `mcp-servers.json`, restart the service to apply changes:
```bash
sudo systemctl restart mcp-connector
```
