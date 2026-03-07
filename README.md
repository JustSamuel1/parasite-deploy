# Parasite VPN — Production Deployment

Censorship-resistant VPN server deployment via Docker Hub.

## Quick Start

### 1. Prerequisites

- Docker & Docker Compose installed
- Domain name pointing to your server
- Ports 443 and 9090 (optional) open in firewall

### 2. Setup TLS Certificates

**Option A: Let's Encrypt (Recommended)**
```bash
# Install certbot
sudo apt install certbot

# Get certificate
sudo certbot certonly --standalone -d your-domain.com

# Copy certificates
mkdir -p certs
sudo cp /etc/letsencrypt/live/your-domain.com/fullchain.pem certs/
sudo cp /etc/letsencrypt/live/your-domain.com/privkey.pem certs/
sudo chmod 644 certs/*.pem
```

**Option B: Self-signed (Testing only)**
```bash
mkdir -p certs
openssl req -x509 -newkey rsa:4096 -keyout certs/privkey.pem -out certs/fullchain.pem -days 365 -nodes
```

### 3. Generate Server Keys

```bash
# Set your Docker Hub username
export DOCKER_USERNAME=your-dockerhub-username

# Generate keypair
docker run --rm $DOCKER_USERNAME/parasite-vpn:latest --keygen
```

Copy the **private key** (not public!) to `config/config.yaml`.

### 4. Configure

Edit `config/config.yaml`:
- Set `crypto.private_key` to generated private key
- Adjust `mask.site` if needed (default: Hacker News)
- Enable metrics if needed: `metrics.enabled: true`

### 5. Deploy

```bash
# Set Docker Hub username in environment
export DOCKER_USERNAME=your-dockerhub-username

# Start server
docker-compose up -d

# Check logs
docker-compose logs -f
```

### 6. Verify

```bash
# Server should be running
docker-compose ps

# Check public key in logs
docker-compose logs | grep "Server public key"
```

## Client Connection

Share this URI with clients (replace with your values):

```
parasite://your-domain.com:443?pk=SERVER_PUBLIC_KEY&dns=1.1.1.1#MyVPN
```

- `SERVER_PUBLIC_KEY`: From server logs (base64)
- `dns`: Optional DNS server for clients
- `#MyVPN`: Optional display name

## Maintenance

### Update Server

```bash
docker-compose pull
docker-compose up -d
```

### View Logs

```bash
docker-compose logs -f
```

### Renew Certificates

```bash
# Let's Encrypt auto-renews, just copy new certs
sudo cp /etc/letsencrypt/live/your-domain.com/fullchain.pem certs/
sudo cp /etc/letsencrypt/live/your-domain.com/privkey.pem certs/
sudo chmod 644 certs/*.pem

# Restart server
docker-compose restart
```

### Backup

```bash
# Backup config and keys
tar -czf parasite-backup-$(date +%Y%m%d).tar.gz config/ certs/
```

## Monitoring

If metrics enabled (`metrics.enabled: true`):

```bash
# Prometheus metrics
curl http://localhost:9090/metrics
```

## Troubleshooting

### Container won't start

```bash
# Check logs
docker-compose logs

# Common issues:
# - Invalid private key
# - Certificate files not found
# - Ports already in use
```

### No internet access through VPN

```bash
# Check IP forwarding on host
sysctl net.ipv4.ip_forward

# Enable if needed
sudo sysctl -w net.ipv4.ip_forward=1
```

### Clients can't connect

- Verify domain points to server IP
- Check firewall allows port 443
- Verify server public key matches client config
- Check server logs for auth failures

## Security

- **Never commit** `config/config.yaml` or `certs/` to git (already in `.gitignore`)
- Rotate server keys periodically
- Monitor `docker-compose logs` for suspicious activity
- Use strong, unique private keys
- Keep Docker images updated

## Directory Structure

```
parasite-deploy/
├── docker-compose.yml    # Production deployment config
├── .gitignore           # Prevents committing secrets
├── config/
│   └── config.yaml      # Server configuration (SECRET)
└── certs/               # TLS certificates (SECRET)
    ├── fullchain.pem
    └── privkey.pem
```

## Links

- Server source: [parasite-server](https://github.com/JustSamuel1/parasite-server)
- Client source: [parasite-client](https://github.com/JustSamuel1/parasite-client)
- Docker Hub: `docker pull <your-username>/parasite-vpn`