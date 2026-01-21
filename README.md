## 👋 Welcome to acme 🚀

ACME CA Server - Self-hosted ACME protocol server with built-in Certificate Authority

## 📋 Description

ACME CA Server is a self-hosted implementation of the ACME protocol (the same protocol used by Let's Encrypt) with a built-in Certificate Authority. Perfect for internal PKI infrastructure, development environments, or any scenario where you need to issue and manage TLS certificates without relying on external CAs.

### Features

- ✅ **ACME Server** implementation supporting http-01 challenge
- 🔐 **Built-in CA** to sign/revoke certificates (replaceable with external CA)
- 📧 **Email notifications** for account creation, expiring and expired certificates
- 🌐 **Web UI** showing certificate and domain logs
- 🔄 **CA rollover** support for seamless certificate authority transitions
- 🔧 Compatible with Certbot, Traefik, Caddy, uacme, acme.sh

## 🚀 Services

This deployment includes the following services:

- **server**: ACME CA Server application (`knrdl/acme-ca-server`)
- **db**: PostgreSQL 15 database for persistent storage

### Infrastructure Components

- **Database**: PostgreSQL for storing certificates, accounts, and CA data

## 📦 Installation

### Using curl

```shell
curl -q -LSsf "https://raw.githubusercontent.com/composemgr/acme/main/docker-compose.yaml" | docker compose -f - up -d
```

### Using git

```shell
git clone "https://github.com/composemgr/acme" ~/.local/srv/docker/acme
cd ~/.local/srv/docker/acme
docker compose up -d
```

### Using composemgr

```shell
composemgr install acme
```

## 🔧 Configuration

### Prerequisites

Before starting the container, you must generate a CA root certificate:

```shell
# Generate CA private key
openssl genrsa -out ca.key 4096

# Generate CA certificate (valid for 10 years)
openssl req -new -x509 -nodes -days 3650 -subj "/C=US/O=MyOrg/CN=My CA" -key ca.key -out ca.pem

# Set proper permissions
chmod 600 ca.key ca.pem
chown 1000:1000 ca.key ca.pem
```

Update the volume paths in docker-compose.yaml to point to your CA files.

### Directory Structure

```
.
├── docker-compose.yaml
├── ca.key              # CA private key (generated above)
├── ca.pem              # CA certificate (generated above)
└── rootfs/
    ├── config/         # Application configuration (future use)
    ├── data/           # Application data and logs
    └── db/
        └── postgres/   # PostgreSQL database files
```

### Environment Variables

#### Core ACME Settings

```shell
TZ=America/New_York                       # Timezone
BASE_HOST_NAME=${HOSTNAME}                # Hostname for the service
BASE_DOMAIN_NAME=                         # Domain name (optional)
EXTERNAL_URL=https://${BASE_HOST_NAME}    # Public HTTPS URL for ACME clients

# Database connection
DB_ADMIN_PASS=changeme_admin_password     # PostgreSQL admin password
```

#### ACME Protocol Configuration

```shell
# Client requirements
ACME_MAIL_REQUIRED=true                   # Require email from ACME clients
ACME_MAIL_TARGET_REGEX=                   # Restrict email format (e.g., ".*@example\.com")
ACME_TARGET_DOMAIN_REGEX=                 # Restrict domain names (e.g., ".*\.example\.com")
ACME_TERMS_OF_SERVICE_URL=                # Optional ToS URL for clients
```

#### Certificate Authority Settings

```shell
# CA Configuration
CA_ENABLED=true                           # Enable built-in CA
ENCRYPTION_KEY=changeme_ca_encryption     # Key to encrypt CA private keys at rest
CA_IMPORT_DIR=/import                     # Directory for CA cert/key import
CA_CERT_LIFETIME=60d                      # Certificate validity period
CA_CERT_CDP_ENABLED=true                  # Include CRL distribution point
CA_CRL_LIFETIME=7d                        # Certificate revocation list rebuild interval
```

#### Email Notification Settings

```shell
# SMTP Configuration
MAIL_ENABLED=true                         # Enable email notifications
EMAIL_SERVER_HOST=172.17.0.1              # SMTP server address
EMAIL_SERVER_PORT=587                     # SMTP port
EMAIL_SERVER_USER=                        # SMTP auth username (optional)
EMAIL_SERVER_PASS=                        # SMTP auth password (optional)
MAIL_ENCRYPTION=tls                       # Encryption: tls, starttls, or plain
EMAIL_SERVER_MAIL_FROM=acme@example.com   # From address

# Notification triggers
MAIL_NOTIFY_ON_ACCOUNT_CREATION=true      # Email when new account created
MAIL_WARN_BEFORE_CERT_EXPIRES=20d         # Warning before expiration
MAIL_NOTIFY_WHEN_CERT_EXPIRED=true        # Email when certificate expires
```

#### Web UI Settings

```shell
WEB_ENABLED=true                          # Enable web interface
WEB_ENABLE_PUBLIC_LOG=false               # Show public certificate transparency log
APP_ORG_NAME=ACME CA Server               # Application title
WEB_APP_DESCRIPTION=Self-hosted ACME CA   # Application description
```

## 🌐 Access

- **Web Interface**: http://172.17.0.1:59082
- **ACME Directory**: https://your-domain.com/acme/directory
- **Production**: Configure reverse proxy with TLS termination

## 📂 Volumes

Data persistence locations:

- `./ca.key` - CA private key (imported once, then stored in database)
- `./ca.pem` - CA certificate (imported once, then stored in database)
- `./rootfs/db/postgres/acme` - PostgreSQL database files

## 🔐 Security

- All secrets use secure defaults with `changeme_*` prefix
- CA private keys encrypted at rest using `ENCRYPTION_KEY`
- Certificate revocation support via CRL
- SMTP configured for Docker gateway (172.17.0.1:587)

## 🎯 Usage Examples

### Test with Certbot

```shell
docker run -it --rm certbot/certbot certonly \
  --server https://acme.example.com/acme/directory \
  --standalone \
  --email admin@example.com \
  --agree-tos \
  --domains test.example.com
```

## 🔍 Logging

View logs:
```shell
docker compose logs -f
docker compose logs -f server
docker compose logs -f db
```

## 🛠️ Management

### Start services
```shell
docker compose up -d
```

### Stop services
```shell
docker compose down
```

### Update images
```shell
docker compose pull && docker compose up -d
```

## 🔄 Backup & Restore

### Backup
```shell
docker compose exec db pg_dump -U postgres postgres > backup-$(date +%Y%m%d).sql
tar -czf acme-backup-$(date +%Y%m%d).tar.gz rootfs/
```

### Restore
```shell
cat backup-YYYYMMDD.sql | docker compose exec -T db psql -U postgres postgres
tar -xzf acme-backup-YYYYMMDD.tar.gz && docker compose up -d
```

## 📋 Requirements

- Docker Engine 20.10+
- Docker Compose V2+
- OpenSSL (for CA generation)
- Reverse proxy for production TLS

## 🆘 Troubleshooting

### CA Import Issues
```shell
# Check file permissions
ls -la ca.key ca.pem
# Should show: -rw------- 1 1000 1000

# Fix if needed
chmod 600 ca.key ca.pem && chown 1000:1000 ca.key ca.pem
```

### Database Issues
```shell
docker compose logs db
docker compose exec db psql -U postgres -c "SELECT version();"
```

## 🤝 Author

🤖 casjay: [Github](https://github.com/casjay) 🤖  
🦄 composemgr: [Github](https://github.com/composemgr) 🦄

## 📚 Resources

- [ACME CA Server GitHub](https://github.com/knrdl/acme-ca-server)
- [ACME Protocol](https://datatracker.ietf.org/doc/html/rfc8555)
- [Certbot Docs](https://eff-certbot.readthedocs.io/)
