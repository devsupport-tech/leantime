# Deploying Leantime on Coolify

This guide covers deploying Leantime on Coolify using Docker Compose.

## Prerequisites

- Coolify instance running and accessible
- Domain name configured (optional but recommended)
- SSL certificate (Coolify can manage this with Let's Encrypt)

## Deployment Steps

### 1. Create New Project in Coolify

1. Log into your Coolify dashboard
2. Click "New Project" or select existing project
3. Choose "Docker Compose" as the deployment type

### 2. Configure Docker Compose

1. **Source**: Choose your deployment source:
   - **Git Repository** (recommended): Point to your Leantime fork/repo
   - **Direct Docker Compose**: Paste the docker-compose.yml content directly

2. **Docker Compose File**: Use `docker-compose.coolify.yml` or paste its contents

3. **Build Pack**: Select "Docker Compose"

### 3. Environment Variables

In Coolify's environment variables section, add these required variables:

```bash
# Required
APP_URL=https://your-domain.com
MYSQL_ROOT_PASSWORD=<generate-strong-password>
MYSQL_PASSWORD=<generate-strong-password>
LEAN_SESSION_PASSWORD=<generate-32-char-string>

# Optional but recommended
LEAN_EMAIL_RETURN=admin@your-domain.com
LEAN_DEFAULT_TIMEZONE=America/New_York
```

### 4. Configure Networking

1. **Domain Settings**:
   - Add your domain in Coolify's domain settings
   - Enable "Force HTTPS" for security
   - Coolify will automatically handle Let's Encrypt SSL

2. **Port Configuration**:
   - Expose port 80 (or custom port via APP_PORT)
   - Coolify will proxy this through its reverse proxy

### 5. Storage Configuration

Coolify automatically handles Docker volumes. The compose file defines:
- `mysql_data`: Database persistence
- `leantime_public_userfiles`: Uploaded files
- `leantime_userfiles`: User files
- `leantime_plugins`: Plugin storage

### 6. Deploy

1. Click "Deploy" in Coolify
2. Monitor deployment logs
3. Wait for containers to be healthy

### 7. Initial Setup

1. Navigate to `https://your-domain.com/install`
2. Follow the installation wizard
3. Create admin account
4. Configure initial settings

## Advanced Configuration

### Email Setup (SMTP)

Add these environment variables in Coolify:

```bash
LEAN_EMAIL_USE_SMTP=true
LEAN_EMAIL_SMTP_HOSTS=smtp.gmail.com
LEAN_EMAIL_SMTP_USERNAME=your-email@gmail.com
LEAN_EMAIL_SMTP_PASSWORD=your-app-password
LEAN_EMAIL_SMTP_PORT=587
LEAN_EMAIL_SMTP_SECURE=tls
```

### S3 Storage

For file storage in S3:

```bash
LEAN_USE_S3=true
LEAN_S3_BUCKET=your-bucket-name
LEAN_S3_REGION=us-east-1
LEAN_S3_KEY=your-access-key
LEAN_S3_SECRET=your-secret-key
```

### OIDC/SSO

For single sign-on:

```bash
LEAN_OIDC_ENABLE=true
LEAN_OIDC_CLIENT_ID=your-client-id
LEAN_OIDC_CLIENT_SECRET=your-client-secret
LEAN_OIDC_PROVIDER_URL=https://your-provider.com
```

## Updating Leantime

### Automatic Updates
1. Enable "Auto Deploy" in Coolify for Git-based deployments
2. Coolify will redeploy when you push updates

### Manual Updates
1. Change image tag in docker-compose.yml (e.g., `leantime/leantime:v3.2.0`)
2. Click "Redeploy" in Coolify
3. Run database migrations if needed

## Backup Strategy

### Database Backups
Add a backup service to docker-compose.yml:

```yaml
backup:
  image: mysql:8.0
  volumes:
    - ./backups:/backups
    - mysql_data:/var/lib/mysql
  command: >
    bash -c "while true; do
      mysqldump -h mysql -u root -p$${MYSQL_ROOT_PASSWORD} 
      leantime > /backups/leantime_$$(date +%Y%m%d_%H%M%S).sql;
      sleep 86400;
    done"
  depends_on:
    - mysql
```

### File Backups
Configure Coolify's backup feature or use external backup solution for volumes.

## Monitoring

### Health Checks
The docker-compose file includes health checks. Monitor in Coolify's dashboard.

### Logs
Access logs through Coolify:
1. Go to your application
2. Click "Logs" tab
3. Select container (leantime or mysql)

## Troubleshooting

### Container Won't Start
- Check environment variables are set correctly
- Verify MySQL password matches in both services
- Check Coolify logs for errors

### Can't Access Application
- Verify domain DNS points to Coolify server
- Check SSL certificate status in Coolify
- Ensure port 80/443 are open in firewall

### Database Connection Issues
- Verify LEAN_DB_HOST is set to 'mysql'
- Check MySQL container is running
- Ensure passwords match in environment variables

### Permission Issues
- Volumes are managed by Coolify
- Check container user permissions
- May need to adjust file ownership in volumes

## Performance Optimization

### Redis Cache (Optional)
Add Redis for better performance:

```yaml
redis:
  image: redis:7-alpine
  restart: unless-stopped
  networks:
    - leantime-network
```

Then add to leantime environment:
```bash
LEAN_CACHE_DRIVER=redis
LEAN_REDIS_HOST=redis
```

### Resource Limits
Configure in Coolify's resource settings:
- Leantime: 1-2 CPU, 1-2GB RAM
- MySQL: 1 CPU, 1-2GB RAM
- Redis (if used): 0.5 CPU, 512MB RAM

## Security Recommendations

1. **Strong Passwords**: Use generated passwords for all services
2. **HTTPS Only**: Always force HTTPS in Coolify
3. **Firewall**: Only expose necessary ports
4. **Updates**: Keep Leantime and dependencies updated
5. **Backups**: Regular automated backups
6. **Monitoring**: Set up uptime monitoring

## Support

- Leantime Documentation: https://docs.leantime.io
- Coolify Documentation: https://coolify.io/docs
- Leantime Community: https://discord.gg/4zMzJtAq9z