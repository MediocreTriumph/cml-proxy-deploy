# CML Proxy Deployment

Deploy the CML Proxy to connect [LabProof](https://labproof.com) to your Cisco Modeling Labs instance.

## Prerequisites

- Docker and Docker Compose installed
- Access to a CML instance (direct or via VPN)
- A LabProof account

## Quick Start

1. **Clone this repo**
   ```bash
   git clone https://github.com/mediocretriumph/cml-proxy-deploy.git
   cd cml-proxy-deploy
   ```

2. **Create your environment file**
   ```bash
   cp .env.example .env
   ```

3. **Add your API key**
   
   Edit `.env` and set `CML_PROXY_API_KEY` to the key you generated in the LabProof portal (or generate one with `openssl rand -hex 32`).

4. **Start the containers**
   ```bash
   docker-compose up -d
   ```

5. **Get your tunnel URL**
   ```bash
   docker-compose logs cloudflared | grep trycloudflare.com
   ```
   
   Copy the `https://something-something.trycloudflare.com` URL.

6. **Configure LabProof**
   
   Enter the tunnel URL and API key in the LabProof portal setup page.

## Verify It's Working

```bash
curl https://YOUR-TUNNEL-URL/health
```

Should return: `{"status":"healthy","service":"cml-proxy"}`

## Stopping

```bash
docker-compose down
```

## CML on VPN

If your CML instance is only accessible via VPN, connect to the VPN on your host machine first. If containers can't reach CML, modify `docker-compose.yml`:

```yaml
services:
  cml-proxy:
    image: mediocretriumph/cml-proxy:latest
    container_name: cml-proxy
    network_mode: host  # Add this line
    # ports:            # Remove or comment out ports section
    #   - "8080:8080"
    environment:
      - CML_PROXY_API_KEY=${CML_PROXY_API_KEY}
    restart: unless-stopped
```

Then update cloudflared to use `host.docker.internal`:

```yaml
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    command: tunnel --no-autoupdate --url http://host.docker.internal:8080
    restart: unless-stopped
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Tunnel URL changed after restart | Quick tunnels get new URLs on restart. Check logs for the new URL. |
| Can't reach CML | Verify VPN connection. Try `network_mode: host` (see above). |
| Container won't start | Check `docker-compose logs cml-proxy` for errors. |
| Health check fails | Ensure port 8080 isn't in use by another application. |

## Support

For issues with LabProof, contact labadmin@labproof.app
