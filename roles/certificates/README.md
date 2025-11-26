# Ansible Role: Certificates

An Ansible role for managing SSL/TLS certificates on Linux systems. Supports self-signed certificates, Let's Encrypt (via Certbot), and importing existing certificates.

## Features

- **Self-Signed Certificates**: Generate self-signed certificates with customizable parameters
- **Let's Encrypt Integration**: Automated certificate issuance via Certbot
  - HTTP-01 challenge support
  - DNS-01 challenge support (with provider plugins or custom hooks)
  - Automatic renewal via systemd timer
- **Certificate Import**: Import existing certificates, keys, and chain files
- **Auto-Detection**: Automatically selects appropriate mode based on domain TLD
- **Idempotent**: Safe to run multiple times without changes
- **Comprehensive Testing**: Full Molecule test suite with Pebble ACME server

## Requirements

- **Ansible**: 2.9 or higher
- **Collections**:
  - `community.crypto` (for certificate generation and validation)
  - `community.general` (optional, for snap support)
- **Target System**: Ubuntu 20.04+ or Debian 10+
- **Python Packages** (on target): `python3-cryptography`

## Role Variables

### Global Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `certificates_items` | `[]` | List of certificates to manage |
| `certificates_cert_dir` | `/etc/ssl/certs` | Directory for certificate files |
| `certificates_key_dir` | `/etc/ssl/private` | Directory for private key files |
| `certificates_certbot_email` | `admin@example.com` | Email for Let's Encrypt registration |
| `certificates_certbot_environment` | `production` | Certbot environment (`production` or `test`) |

### Certificate Item Variables

Each item in `certificates_items` can have the following variables:

#### Common Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `name` | Yes | - | Certificate name (used for filenames) |
| `mode` | No | `auto` | Certificate mode: `auto`, `selfsigned`, `letsencrypt`, or `imported` |

#### Self-Signed Certificate Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `key_size` | `4096` | RSA key size in bits |
| `validity_days` | `365` | Certificate validity period in days |
| `country` | `""` | Country name (C) |
| `state` | `""` | State or province name (ST) |
| `locality` | `""` | Locality name (L) |
| `organization` | `""` | Organization name (O) |
| `organizational_unit` | `""` | Organizational unit name (OU) |
| `email` | `""` | Email address |
| `san` | `[]` | Subject Alternative Names (e.g., `["DNS:*.example.com", "IP:192.168.1.1"]`) |

#### Let's Encrypt Certificate Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `challenge_type` | No | `http-01` | Challenge type: `http-01` or `dns-01` |
| `webroot` | For HTTP-01 | `/var/www/html` | Webroot path for HTTP-01 challenges |
| `dns_provider` | For DNS-01 | - | DNS provider name (e.g., `cloudflare`, `route53`, `custom`) |
| `dns_credentials` | For DNS-01 | - | Path to DNS provider credentials file |
| `auth_hook` | For custom DNS | - | Path to authentication hook script |
| `cleanup_hook` | For custom DNS | - | Path to cleanup hook script |
| `deploy_hook` | No | `""` | Path to deployment hook script |
| `san` | No | `[]` | Additional domain names for the certificate |

#### Imported Certificate Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `src_cert` | Yes | Path to source certificate file |
| `src_key` | Yes | Path to source private key file |
| `src_chain` | No | Path to source certificate chain file |

## Mode Auto-Detection

When `mode: auto` (or mode is not specified), the role automatically selects the appropriate mode based on the domain TLD:

- **Self-Signed**: `.test`, `.local`, `.localhost`, `.invalid`, `.example`
- **Let's Encrypt**: All other TLDs (`.com`, `.org`, `.net`, etc.)

## Dependencies

None. The role will install required packages (certbot, python3-cryptography) automatically.

## Example Playbooks

### Basic Self-Signed Certificate

```yaml
---
- hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.certificates
      certificates_items:
        - name: myapp.local
          # Mode auto-detected as 'selfsigned' due to .local TLD
```

### Self-Signed with Custom Configuration

```yaml
---
- hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.certificates
      certificates_items:
        - name: custom.local
          mode: selfsigned
          key_size: 2048
          validity_days: 730
          country: US
          state: Texas
          locality: Austin
          organization: My Company
          organizational_unit: IT
          email: admin@mycompany.com
          san:
            - DNS:custom.local
            - DNS:*.custom.local
            - IP:127.0.0.1
```

### Let's Encrypt with HTTP-01 Challenge

```yaml
---
- hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.certificates
      certificates_certbot_email: admin@example.com
      certificates_items:
        - name: example.com
          mode: letsencrypt
          challenge_type: http-01
          webroot: /var/www/html
          san:
            - www.example.com
            - api.example.com
```

### Let's Encrypt with DNS-01 Challenge (Cloudflare)

```yaml
---
- hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.certificates
      certificates_certbot_email: admin@example.com
      certificates_items:
        - name: example.com
          mode: letsencrypt
          challenge_type: dns-01
          dns_provider: cloudflare
          dns_credentials: /root/.secrets/cloudflare.ini
          san:
            - "*.example.com"  # Wildcard certificate
```

### Let's Encrypt with Custom DNS Hooks

```yaml
---
- hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.certificates
      certificates_certbot_email: admin@example.com
      certificates_items:
        - name: example.com
          mode: letsencrypt
          challenge_type: dns-01
          dns_provider: custom
          auth_hook: /usr/local/bin/dns-auth.sh
          cleanup_hook: /usr/local/bin/dns-cleanup.sh
          san:
            - "*.example.com"
```

### Import Existing Certificate

```yaml
---
- hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.certificates
      certificates_items:
        - name: imported-cert
          mode: imported
          src_cert: /path/to/certificate.crt
          src_key: /path/to/private.key
          src_chain: /path/to/chain.crt  # Optional
```

### Mixed Certificate Types

```yaml
---
- hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.certificates
      certificates_certbot_email: admin@example.com
      certificates_items:
        # Self-signed for development
        - name: dev.local
          mode: selfsigned

        # Let's Encrypt for production
        - name: example.com
          mode: letsencrypt
          challenge_type: http-01
          webroot: /var/www/html
          san:
            - www.example.com

        # Imported certificate from external CA
        - name: corporate-cert
          mode: imported
          src_cert: /tmp/certs/corporate.crt
          src_key: /tmp/certs/corporate.key
```

## File Locations

### Self-Signed and Imported Certificates

- **Certificate**: `/etc/ssl/certs/<name>.crt`
- **Private Key**: `/etc/ssl/private/<name>.key` (mode 0600)
- **Chain** (imported only): `/etc/ssl/certs/<name>.chain.crt`

### Let's Encrypt Certificates

Certbot manages these in its own directory structure:

- **Certificate**: `/etc/letsencrypt/live/<name>/cert.pem`
- **Private Key**: `/etc/letsencrypt/live/<name>/privkey.pem`
- **Full Chain**: `/etc/letsencrypt/live/<name>/fullchain.pem`
- **Chain**: `/etc/letsencrypt/live/<name>/chain.pem`

## Certificate Renewal

Let's Encrypt certificates are automatically renewed via the `certbot.timer` systemd unit, which runs twice daily.

To manually renew:

```bash
certbot renew
```

To test renewal:

```bash
certbot renew --dry-run
```

## Testing

The role includes comprehensive Molecule tests. See `extensions/molecule/certificates/README.md` for details.

### Running Tests

```bash
cd extensions
molecule test -s certificates
```

### Test Coverage

- Self-signed certificate generation with various configurations
- Let's Encrypt HTTP-01 challenges
- Let's Encrypt DNS-01 challenges with custom hooks
- Certificate import functionality
- Idempotency verification
- Input validation

## Troubleshooting

### Certbot HTTP-01 Challenge Fails

**Problem**: "Failed authorization procedure"

**Solutions**:
- Ensure port 80 is accessible from the internet
- Verify the webroot directory exists and is writable
- Check that your domain's DNS points to the server

### Certbot DNS-01 Challenge Fails

**Problem**: "DNS problem: query timed out"

**Solutions**:
- Verify DNS provider credentials are correct
- Check that the DNS plugin is installed
- For custom hooks, ensure scripts are executable and working

### Certificate Not Renewed Automatically

**Problem**: Certificate expired

**Solutions**:
- Check certbot.timer status: `systemctl status certbot.timer`
- View renewal logs: `journalctl -u certbot.service`
- Test renewal: `certbot renew --dry-run`

### Permission Denied on Private Key

**Problem**: Applications can't read private key

**Solutions**:
- Private keys are created with mode 0600 (owner-only)
- Add application user to appropriate group or adjust permissions carefully
- Consider using ACLs for fine-grained access control

## Security Considerations

1. **Private Key Protection**: Private keys are created with mode 0600 and should never be world-readable
2. **Certbot Credentials**: Store DNS provider credentials securely with restricted permissions
3. **Email Privacy**: The email used for Let's Encrypt registration may be publicly visible
4. **Rate Limits**: Let's Encrypt has rate limits; use staging environment for testing
5. **Backup**: Regularly backup `/etc/letsencrypt/` and private keys

## License

MIT

## Author Information

This role was created as part of the jlira.web_server Ansible collection.

## Contributing

Issues and pull requests are welcome. Please ensure all changes include appropriate tests.
