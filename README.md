# CML Proxy Deployment

Deploy the CML Proxy to connect [LabProof](https://labproof.app) to your Cisco Modeling Labs instance.

## Prerequisites

- Docker and Docker Compose installed
- Access to a CML instance (direct or via VPN)

## Quick Start

1. **Sign in to LabProof**
   
   Go to [labproof.app](https://labproof.app) and sign in with your GitHub account.

2. **Generate your API key**
   
   In the LabProof setup page, click **Generate** to create your Proxy API Key. Keep this page open - you'll need the key in step 5.

3. **Clone this repo**
   ```bash
   git clone https://github.com/mediocretriumph/cml-proxy-deploy.git
   cd cml-proxy-deploy
   ```

4. **Create your environment file**
   ```bash
   cp .env.example .env
   ```

5. **Add your API key**
   
   Edit `.env` and replace `your-api-key-here` with the key you generated in LabProof.

6. **Start the containers**
   ```bash
   docker-compose up -d
   ```

7. **Get your tunnel URL**
   ```bash
   docker-compose logs cloudflared | grep trycloudflare.com
   ```
   
   Copy the `https://something-something.trycloudflare.com` URL.

8. **Complete LabProof setup**
   
   Back in the LabProof portal, enter your tunnel URL and CML credentials, then click **Connect**.

## Verify It's Working

```bash
curl https://YOUR-TUNNEL-URL/health
```

Should return: `{"status":"healthy","service":"cml-proxy"}`

## Stopping

```bash
docker-compose down
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Tunnel URL changed after restart | Quick tunnels get new URLs on restart. Run the logs command again for the new URL. |
| Container won't start | Check `docker-compose logs cml-proxy` for errors. |
| Health check fails | Ensure port 8080 isn't in use by another application. |

### CML on VPN (if needed)

Most VPN setups work without modification. If your containers can't reach CML, try adding `network_mode: host` to the cml-proxy service in `docker-compose.yml`:

```yaml
services:
  cml-proxy:
    image: mediocretriumph/cml-proxy:latest
    container_name: cml-proxy
    network_mode: host
    environment:
      - CML_PROXY_API_KEY=${CML_PROXY_API_KEY}
    restart: unless-stopped
```

And update cloudflared to point to `host.docker.internal:8080` instead of `cml-proxy:8080`.

## Support

This is a proof of concept and support is not currently offered.
For issues with LabProof, contact labadmin@labproof.app
>>>>>>> refs/remotes/origin/main
