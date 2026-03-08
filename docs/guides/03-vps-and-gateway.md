# 03. VPS & Gateway Setup

This step requires a cloud server. I use a DigitalOcean Droplet, but any Linux VPS (Linode, AWS EC2, Hetzner) will work. A basic $6/month machine with 1GB of RAM is sufficient.

## 1. Initial Server Hardening

Log into your new server as `root` and immediately create a non-root user. Running node services as root is a security liability.

```bash
# SSH into your new box
ssh root@<your-server-ip>

# Create a new user (replace 'username' with your preferred name)
useradd -m -s /bin/bash -G sudo username
passwd username

# Set up SSH keys for the new user
mkdir -p /home/username/.ssh
chmod 700 /home/username/.ssh
cp ~/.ssh/authorized_keys /home/username/.ssh/
chown -R username:username /home/username/.ssh
```

Open a *new* terminal window, verify you can SSH in as `username`, and then disable root login.

```bash
# Inside the server, edit the SSH config
sudo nano /etc/ssh/sshd_config

# Set these values:
PermitRootLogin no
PasswordAuthentication no

# Restart SSH
sudo systemctl restart ssh
```

## 2. Install Dependencies

You need Node.js (via `nvm`) and the `Caddy` web server.

```bash
# Install nvm and Node.js 22
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.nvm/nvm.sh
nvm install 22
nvm alias default 22

# Install the TypingMind MCP Gateway globally
npm install -g @typingmind/mcp

# Install Caddy (Ubuntu/Debian)
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

## 3. Configure the MCP Multiplexer

The Gateway needs a JSON configuration file to know which MCP tools to spawn. 

First, create a directory for your configuration:
```bash
mkdir -p ~/.config/mcp
```

Create the configuration file:
```bash
nano ~/.config/mcp/mcp-servers.json
```

Paste the following configuration, replacing the `<placeholders>` with your actual API keys. 
*Note:* The Memory server uses the OpenRouter API for embeddings (`text-embedding-3-small` by default), but you can supply a standard OpenAI key if you prefer.

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
    }
  }
}
```

## 4. Secure the Gateway

Generate a strong, cryptographically secure bearer token. **Save this token.** You will need it to connect TypingMind to the server.

```bash
openssl rand -hex 32
# Output: e.g., 6e2941d48af8f957b2b14071...
```

Create a systemd service file to keep the Gateway running in the background.

```bash
sudo nano /etc/systemd/system/mcp-connector.service
```

Paste the following (replace `username` and `<your-generated-token>`):

```ini
[Unit]
Description=TypingMind MCP Connector
After=network.target

[Service]
Type=simple
User=username
# Ensure nvm is loaded so `npx` works
Environment=PATH=/home/username/.nvm/versions/node/v22.14.0/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# Pass the auth token as an environment variable
Environment=MCP_AUTH_TOKEN=<your-generated-token>
ExecStart=/home/username/.nvm/versions/node/v22.14.0/bin/mcp-server --config /home/username/.config/mcp/mcp-servers.json --port 50880
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable mcp-connector
sudo systemctl start mcp-connector
sudo systemctl status mcp-connector # Verify it is running
```

## 5. Configure Caddy (HTTPS)

TypingMind runs on HTTPS and will block connections to an insecure `http://` endpoint. We use Caddy to proxy traffic to our local port `50880` and automatically provision an SSL certificate.

You must point a domain name (e.g., `mcp.yourdomain.com`) at your server's IP address. If you do not own a domain, you can use a free dynamic DNS provider like DuckDNS.

Edit the Caddyfile:
```bash
sudo nano /etc/caddy/Caddyfile
```

Replace the contents with:
```caddyfile
mcp.yourdomain.com {
    reverse_proxy localhost:50880
}
```

Restart Caddy to apply the changes and fetch the SSL certificate:
```bash
sudo systemctl restart caddy
```

## 6. Verification

From your local computer (not the VPS), run this `curl` command to verify the connection is secure, the token works, and the servers are loaded:

```bash
curl -H "Authorization: Bearer <your-generated-token>" https://mcp.yourdomain.com/clients
```

If successful, you will receive a JSON payload listing the active MCP clients (memory, brave-search, github). You are ready to connect TypingMind.
