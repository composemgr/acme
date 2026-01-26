## 👋 Welcome to acme 🚀

ACME server implementation for issuing SSL/TLS certificates

## 📋 Description

ACME server implementation for issuing SSL/TLS certificates

## 🚀 Services

- **server**: knrdl/acme-ca-server:latest

### Infrastructure Components

- **db**: Postgres database


## 📦 Installation

### Option 1: Quick Install
```bash
curl -q -LSsf "https://raw.githubusercontent.com/composemgr/acme/main/docker-compose.yaml" -o compose.yml
```

### Option 2: Git Clone
```bash
git clone "https://github.com/composemgr/acme" ~/.local/srv/docker/acme
cd ~/.local/srv/docker/acme
docker compose up -d
```

### Option 3: Using composemgr
```bash
composemgr install acme
```

## 🔧 Configuration

### Environment Variables

```shell
TZ=America/New_York
DB_ADMIN_PASS=changeme_admin_password
ENCRYPTION_KEY=changeme_ca_encryption_key
EMAIL_SERVER_HOST=172.17.0.1
EMAIL_SERVER_PORT=587
EMAIL_SERVER_MAIL_FROM=no-reply@${BASE_DOMAIN_NAME:-${BASE_HOST_NAME
APP_ORG_NAME=ACME CA Server
```

See `docker-compose.yaml` for complete list of configurable options.

## 🌐 Access

- **Web Interface**: http://172.17.0.1:59082

## 📂 Volumes

- `./rootfs/data/db/postgres/acme` - Data storage

## 🔐 Security

- Change all default passwords before deploying to production
- Use strong secrets for all authentication tokens
- Configure HTTPS using a reverse proxy (nginx, traefik, caddy)
- Regularly update Docker images for security patches
- Backup your data regularly

## 🔍 Logging

```shell
docker compose logs -f server
```

## 🛠️ Management

```bash
# Start services
docker compose up -d

# Stop services
docker compose down

# Update to latest images
docker compose pull && docker compose up -d

# View logs
docker compose logs -f

# Restart services
docker compose restart
```

## 📋 Requirements

- Docker Engine 20.10+
- Docker Compose V2+

## 🤝 Author

🤖 casjay: [Github](https://github.com/casjay) 🤖  
🦄 composemgr: [Github](https://github.com/composemgr) 🦄
