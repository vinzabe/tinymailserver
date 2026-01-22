# TinyMailServer

A simplified, Portainer-ready mail server based on [Mailu](https://github.com/Mailu/Mailu).

## Features

- **SMTP/IMAP/POP3** - Full email server with secure protocols
- **Webmail** - Snappymail or Roundcube web interface
- **Anti-spam** - Rspamd spam filtering with DKIM signing
- **Admin Panel** - Web-based administration
- **Let's Encrypt** - Automatic SSL certificate management
- **Portainer Ready** - Deploy directly from Portainer with stack.env

## Quick Start

### Using Portainer

1. In Portainer, go to **Stacks** > **Add stack**
2. Select **Repository** and enter:
   - Repository URL: `https://github.com/vinzabe/tinymailserver`
   - Compose path: `docker-compose.yml`
   - Env file: `stack.env`
3. Configure the environment variables
4. Deploy the stack

### Using Docker Compose

```bash
# Clone the repository
git clone https://github.com/vinzabe/tinymailserver.git
cd tinymailserver

# Copy and edit the environment file
cp stack.env .env
nano .env

# Start the server
docker compose up -d

# Enable optional services (webmail, antivirus, etc.)
docker compose --profile webmail up -d
```

## Configuration

Edit `stack.env` (or `.env` locally) before deployment:

### Required Settings

| Variable | Description | Example |
|----------|-------------|---------|
| `SECRET_KEY` | Random 16+ character string | `MySecretKey12345` |
| `DOMAIN` | Your mail domain | `example.com` |
| `HOSTNAMES` | Mail server hostname(s) | `mail.example.com` |
| `POSTMASTER` | Admin email local part | `admin` |

### Optional Features

| Variable | Options | Description |
|----------|---------|-------------|
| `WEBMAIL` | `snappymail`, `roundcube`, `none` | Webmail client |
| `ANTIVIRUS` | `clamav`, `none` | ClamAV (uses ~1GB RAM) |
| `WEBDAV` | `radicale`, `none` | CalDAV/CardDAV |
| `TLS_FLAVOR` | `letsencrypt`, `cert`, `notls` | SSL certificate mode |

## Ports

| Port | Protocol | Description |
|------|----------|-------------|
| 25 | SMTP | Mail transfer |
| 465 | SMTPS | Secure SMTP |
| 587 | Submission | Mail submission |
| 110 | POP3 | Mail retrieval |
| 995 | POP3S | Secure POP3 |
| 143 | IMAP | Mail access |
| 993 | IMAPS | Secure IMAP |
| 80 | HTTP | Web interface |
| 443 | HTTPS | Secure web interface |

## DNS Configuration

Add these DNS records for your domain:

```
# MX Record
@       MX      10 mail.example.com.

# A/AAAA Records
mail    A       YOUR_SERVER_IP
mail    AAAA    YOUR_SERVER_IPV6

# SPF Record
@       TXT     "v=spf1 mx a:mail.example.com -all"

# DMARC Record
_dmarc  TXT     "v=DMARC1; p=reject; rua=mailto:admin@example.com"

# DKIM Record (get from admin panel after setup)
dkim._domainkey TXT "v=DKIM1; k=rsa; p=YOUR_DKIM_PUBLIC_KEY"
```

## First Login

After deployment:

1. Access admin panel: `https://mail.example.com/admin`
2. Initial admin account: `admin@YOUR_DOMAIN`
3. Set password via CLI:

```bash
docker exec -it mail_admin flask mailu admin admin YOUR_DOMAIN PASSWORD
```

## Profiles

Optional services are enabled via Docker Compose profiles:

```bash
# Enable webmail
docker compose --profile webmail up -d

# Enable antivirus (requires ~1GB RAM)
docker compose --profile antivirus up -d

# Enable CalDAV/CardDAV
docker compose --profile webdav up -d

# Enable fetchmail
docker compose --profile fetchmail up -d

# Enable all optional services
docker compose --profile webmail --profile antivirus --profile webdav up -d
```

## Volumes

| Volume | Purpose |
|--------|---------|
| `mail_certs` | SSL certificates |
| `mail_mail` | User mailboxes |
| `mail_queue` | Mail queue |
| `mail_filter` | Spam filter data |
| `mail_dkim` | DKIM keys |
| `mail_admin` | Admin database |
| `mail_webmail` | Webmail data |
| `mail_redis` | Cache data |

## Troubleshooting

### Check logs
```bash
docker compose logs -f front
docker compose logs -f smtp
docker compose logs -f imap
```

### Test SMTP
```bash
telnet mail.example.com 25
```

### Rebuild after config changes
```bash
docker compose down
docker compose up -d
```

## License

Based on Mailu, licensed under MIT. See [LICENSE](LICENSE).

## Credits

- [Mailu Project](https://mailu.io/) - The upstream mail server
- [Portainer](https://www.portainer.io/) - Container management
