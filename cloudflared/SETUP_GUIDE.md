# Cloudflare Tunnel Setup Guide

This guide explains how to set up a free Cloudflare Tunnel to self-host your site securely without exposing your IP address.

## What is Cloudflare Tunnel?

Cloudflare Tunnel creates a secure connection from your local machine to Cloudflare's network, allowing you to expose your local services (like your Flask app) to the internet without port forwarding or firewall changes.

## Prerequisites

- A Cloudflare account (free tier works)
- A domain registered and pointed to Cloudflare nameservers
- `cloudflared` installed on your Raspberry Pi

## Installation

### 1. Install cloudflared

**On Raspberry Pi (Debian/Ubuntu)**:
```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm.tgz
tar -xzf cloudflared-linux-arm.tgz
sudo cp cloudflared /usr/local/bin/
sudo chmod +x /usr/local/bin/cloudflared
cloudflared --version
```

**On Arch Linux (Steam Deck)**:
```bash
sudo pacman -S cloudflare-warp
```

### 2. Authenticate cloudflared

Log in to Cloudflare:
```bash
cloudflared tunnel login
```

This will open a browser window. Select your domain and authorize. It creates `~/.cloudflared/cert.pem`.

### 3. Create a tunnel

```bash
cloudflared tunnel create my-tunnel
```

This generates:
- Tunnel credentials in `~/.cloudflared/<TUNNEL_ID>.json`
- Tunnel name and ID

### 4. Configure the tunnel

Create a config file at `~/.cloudflared/config.yml`:

```yaml
tunnel: my-tunnel
credentials-file: /home/deck/.cloudflared/<TUNNEL_ID>.json

ingress:
  - hostname: myapp.yourdomain.com
    service: http://localhost:8085
  - hostname: astro.yourdomain.com
    service: http://localhost:3000
  - service: http_status:404
```

Replace:
- `my-tunnel` with your tunnel name
- `<TUNNEL_ID>` with your tunnel ID
- `yourdomain.com` with your actual domain
- Port numbers with your app ports (8085 for Flask, 3000 for Astro, etc.)

### 5. Create DNS records in Cloudflare

In the Cloudflare dashboard:
1. Go to **DNS** → **Records**
2. Add a CNAME record:
   - Name: `myapp`
   - Content: `<TUNNEL_ID>.cfargotunnel.com`
   - Proxy status: Proxied ✓

Repeat for each subdomain in your config.

### 6. Run the tunnel

**One-time test**:
```bash
cloudflared tunnel run my-tunnel
```

**Run as a service** (recommended):
```bash
sudo cloudflared service install
sudo systemctl start cloudflared
sudo systemctl enable cloudflared
```

Check status:
```bash
sudo systemctl status cloudflared
cloudflared tunnel info my-tunnel
```

## Docker Setup (Alternative)

Add this to your `docker-compose.yml`:

```yaml
cloudflared:
  image: cloudflare/cloudflared:latest
  container_name: cloudflared
  volumes:
    - ./cloudflared/config.yml:/etc/cloudflared/config.yml
    - ~/.cloudflared/credentials.json:/root/.cloudflared/credentials.json
  command: tunnel run
  restart: unless-stopped
```

Then run:
```bash
docker-compose up -d cloudflared
```

## Troubleshooting

**Check tunnel status**:
```bash
cloudflared tunnel list
cloudflared tunnel info my-tunnel
```

**View logs**:
```bash
sudo journalctl -u cloudflared -f
```

**Test local connection**:
```bash
curl http://localhost:8085
```

**Verify DNS resolution**:
```bash
nslookup myapp.yourdomain.com
```

## Security Tips

- Keep credentials file secure: `chmod 600 ~/.cloudflared/cert.pem`
- Enable Cloudflare security features (firewall rules, rate limiting)
- Use strong passwords and 2FA on Cloudflare account
- Monitor tunnel activity in Cloudflare dashboard

## References

- [Cloudflare Tunnel Docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)
- [cloudflared GitHub](https://github.com/cloudflare/cloudflared)
