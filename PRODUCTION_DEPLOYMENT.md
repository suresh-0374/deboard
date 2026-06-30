# Production Deployment Guide

## Prerequisites
- Docker & Docker Compose 2.0+
- SSL/TLS certificates (Let's Encrypt or your CA)
- At least 4GB RAM, 2 CPU cores
- Linux server (Ubuntu 22.04 LTS recommended)

## Setup Steps

### 1. Prepare Server
```bash
# Clone repository
git clone <repo-url>
cd Dev-Board

# Create directories
mkdir -p nginx/conf.d certs/ssl monitoring/grafana/provisioning backups logs

# Copy production environment
cp .env.production .env
# EDIT .env with your actual values
nano .env
```

### 2. SSL Certificates
```bash
# Option A: Let's Encrypt (Recommended)
sudo certbot certonly --standalone -d your-domain.com
sudo cp /etc/letsencrypt/live/your-domain.com/fullchain.pem certs/ssl/cert.pem
sudo cp /etc/letsencrypt/live/your-domain.com/privkey.pem certs/ssl/key.pem
sudo chown $USER:$USER certs/ssl/*

# Option B: Self-signed (Testing only)
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout certs/ssl/key.pem -out certs/ssl/cert.pem
```

### 3. Configure nginx
```bash
# Edit nginx configuration
nano nginx/nginx.conf
# Update server_name, ssl certificate paths
```

### 4. Pull Latest Images
```bash
docker compose -f docker-compose.prod.yml pull
```

### 5. Start Services
```bash
# Start with production compose file
docker compose -f docker-compose.prod.yml up -d

# Verify all services
docker compose -f docker-compose.prod.yml ps

# Check logs
docker compose -f docker-compose.prod.yml logs -f
```

### 6. Initialize Database
```bash
# Apply migrations (if any)
docker compose -f docker-compose.prod.yml exec backend go run ./cmd/migrate
```

### 7. Access Dashboards
- **Application**: https://your-domain.com
- **Grafana**: https://your-domain.com:3000 (add proxy in nginx if needed)
- **Prometheus**: https://your-domain.com:9090 (add proxy in nginx if needed)

## Monitoring & Maintenance

### Health Checks
```bash
# Check all services health
docker compose -f docker-compose.prod.yml ps
docker compose -f docker-compose.prod.yml logs backend | tail -20
```

### Database Backups
```bash
# Manual backup
docker compose -f docker-compose.prod.yml exec postgres \
  pg_dump -U $POSTGRES_USER $POSTGRES_DB > backups/backup-$(date +%Y%m%d-%H%M%S).sql

# Automated backup (cron job)
0 2 * * * cd /path/to/Dev-Board && docker compose -f docker-compose.prod.yml exec -T postgres pg_dump -U devboard_prod_user devboard_production > backups/backup-$(date +\%Y\%m\%d-\%H\%M\%S).sql
```

### Log Rotation (already configured)
- Logs limited to 10MB per file, 3 files max
- Located in container, follow via `docker compose logs`

### Certificate Renewal (Let's Encrypt)
```bash
# Add to crontab (runs monthly)
0 3 1 * * certbot renew --quiet && \
  cp /etc/letsencrypt/live/your-domain.com/fullchain.pem /path/to/certs/ssl/cert.pem && \
  cp /etc/letsencrypt/live/your-domain.com/privkey.pem /path/to/certs/ssl/key.pem && \
  docker compose -f docker-compose.prod.yml restart nginx
```

## Scaling to Multiple Hosts

For multi-host deployment, use **Docker Swarm** or **Kubernetes**:

### Docker Swarm
```bash
# Initialize swarm
docker swarm init

# Deploy stack
docker stack deploy -c docker-compose.prod.yml devboard
```

### Kubernetes (helm chart recommended)
```bash
helm install devboard ./chart/devboard -f values-prod.yaml
```

## Security Checklist
- ✅ SSL/TLS enabled
- ✅ Environment variables not in code
- ✅ Database SSL mode enabled
- ✅ Rate limiting configured
- ✅ Security headers set
- ✅ Resource limits enforced
- ✅ Logging enabled
- ✅ Monitoring configured
- ✅ Backups automated
- ✅ Firewall rules: only 80/443 public

## Support & Troubleshooting
- Check logs: `docker compose -f docker-compose.prod.yml logs -f [service]`
- View metrics: http://your-domain.com:9090
- Grafana dashboards: http://your-domain.com:3000
