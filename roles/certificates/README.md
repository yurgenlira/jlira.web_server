# jlira.web_server.certificates

A comprehensive Ansible role for managing SSL/TLS certificates on Debian/Ubuntu systems. Supports self-signed certificate generation, importing existing certificates, and Let's Encrypt integration with certbot.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Role Variables](#role-variables)
- [Dependencies](#dependencies)
- [Example Playbooks](#example-playbooks)
- [Tags](#tags)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Future Enhancements](#future-enhancements)
- [License](#license)
- [Author](#author)

## Overview

This role provides a comprehensive, flexible solution for SSL/TLS certificate management that works across all deployment scenarios. It follows the philosophy of keeping the role simple and generic, letting playbooks handle complex orchestration.

**Supported modes:**
- **selfsigned**: Generate self-signed certificates using community.crypto modules
- **import**: Import existing certificates from control machine to managed hosts
- **letsencrypt**: Obtain trusted certificates from Let's Encrypt using certbot

## Features

- **Simple and Generic**: Works with single servers, multiple servers, and load balancer setups
- **Three Operation Modes**: Generate self-signed certificates, import existing ones, or obtain from Let's Encrypt
- **Let's Encrypt Integration with Certbot**:
  - HTTP-01 challenge support (webroot, standalone, Apache, Nginx plugins)
  - DNS-01 challenge support for wildcard certificates (Cloudflare, Route53, DigitalOcean, Google, custom hooks)
  - Automatic certificate renewal via certbot's built-in systemd timer
  - Support for production, staging, and dry-run modes
  - Per-certificate deploy hooks for service reloads
  - Per-certificate configuration (challenge type, plugin, webroot, DNS provider)
  - Automatic domain expansion when adding domains to existing certificates
- **Comprehensive Validation**: Validates all inputs before execution
- **Subject Alternative Names (SAN)**: Full support for multi-domain certificates
- **Secure Defaults**: Proper file permissions and ownership
- **Idempotent**: All tasks are idempotent and support check mode
- **Tag Support**: Fine-grained control using Ansible tags

## Requirements

- **Target Systems**: Debian 11+ or Ubuntu 20.04+
- **Ansible**: 2.9+
- **Collections**: community.crypto (automatically installed)
- **Privileges**: Tasks require `become: true` (root/sudo access)

## Role Variables

All variables are defined in `defaults/main.yml` with comprehensive documentation.

### Operation Mode

#### `certificates_mode`
- **Type**: String
- **Default**: `selfsigned`
- **Options**: `selfsigned`, `import`, `letsencrypt`
- **Description**: Certificate operation mode

```yaml
certificates_mode: selfsigned
# Or for Let's Encrypt:
# certificates_mode: letsencrypt
```

### Directory Configuration

#### `certificates_cert_dir`
- **Type**: String
- **Default**: `/etc/ssl/certs`
- **Description**: Output directory for certificate files (selfsigned and import modes)
- **Note**: For letsencrypt mode, certificates are managed in `/etc/letsencrypt/live/`

```yaml
certificates_cert_dir: /etc/ssl/certs
```

#### `certificates_key_dir`
- **Type**: String
- **Default**: `/etc/ssl/private`
- **Description**: Output directory for private key files (selfsigned and import modes)
- **Note**: For letsencrypt mode, keys are managed in `/etc/letsencrypt/live/`

```yaml
certificates_key_dir: /etc/ssl/private
```

### Certificate List

#### `certificates_list`
- **Type**: List of Dictionaries
- **Default**: `[]`
- **Description**: List of certificates to manage. Structure varies by mode.

**Common fields (all modes):**
- `name`: Unique identifier for the certificate (required)

**Fields for selfsigned mode:**
- `common_name`: Common Name (CN) for the certificate (required)
- `organization`: Organization Name (O) (optional)
- `organizational_unit`: Organizational Unit (OU) (optional)
- `country`: Country Code (C) - 2 letter code (optional)
- `state`: State or Province (ST) (optional)
- `locality`: Locality or City (L) (optional)
- `email`: Email Address (optional)
- `days`: Certificate validity in days (default: 365)
- `key_size`: RSA key size in bits (default: 4096)
- `san`: Subject Alternative Names list (optional)

**Example for selfsigned mode:**
```yaml
certificates_list:
  - name: example.com
    common_name: example.com
    organization: "My Company"
    organizational_unit: "IT Department"
    country: "US"
    state: "California"
    locality: "San Francisco"
    email: "admin@example.com"
    days: 730
    key_size: 4096
    san:
      - DNS:www.example.com
      - DNS:api.example.com
      - IP:192.168.1.1
```

**Fields for import mode:**
- `cert_src`: Source path to certificate file on control machine (required)
- `key_src`: Source path to private key file on control machine (required)
- `chain_src`: Source path to certificate chain file (optional)

**Example for import mode:**
```yaml
certificates_list:
  - name: example.com
    cert_src: files/example.com.crt
    key_src: files/example.com.key
    chain_src: files/example.com-chain.crt
```

**Fields for letsencrypt mode:**
- `domains`: List of domain names for the certificate (required, first domain is primary)
- `challenge_type`: ACME challenge type (required: `http-01` or `dns-01`)

**For HTTP-01 challenges:**
- `plugin`: Certbot plugin (required: `webroot`, `standalone`, `apache`, `nginx`)
- `webroot`: Document root path (required for webroot plugin, must be absolute path)

**For DNS-01 challenges:**
- `provider`: DNS provider override (optional, uses global `certificates_certbot_dns_provider` if not set)
  - Options: `cloudflare`, `route53`, `digitalocean`, `google`, `custom`
- `dns_credentials`: DNS credentials override (optional, uses global `certificates_certbot_dns_credentials` if not set)
- `auth_hook`: Authentication script path for custom provider (required when provider is `custom`)
- `cleanup_hook`: Cleanup script path for custom provider (required when provider is `custom`)

**Optional for all letsencrypt challenges:**
- `deploy_hook`: Command to run after successful obtain/renew (e.g., service reload)

**Example for letsencrypt mode (HTTP-01 with webroot):**
```yaml
certificates_list:
  - name: example.com
    domains:
      - example.com
      - www.example.com
    challenge_type: http-01
    plugin: webroot
    webroot: /var/www/html
    deploy_hook: "systemctl reload apache2"
```

**Example for letsencrypt mode (HTTP-01 with Apache plugin):**
```yaml
certificates_list:
  - name: apache-site.com
    domains:
      - apache-site.com
      - www.apache-site.com
    challenge_type: http-01
    plugin: apache
```

**Example for letsencrypt mode (DNS-01 wildcard with Cloudflare):**
```yaml
certificates_list:
  - name: wildcard.example.com
    domains:
      - example.com
      - "*.example.com"
    challenge_type: dns-01
    # Uses global certificates_certbot_dns_provider and credentials
```

**Example for letsencrypt mode (DNS-01 with custom hooks):**
```yaml
certificates_list:
  - name: custom-dns.example.com
    domains:
      - custom-dns.example.com
      - "*.custom-dns.example.com"
    challenge_type: dns-01
    provider: custom
    auth_hook: "/usr/local/bin/dns-auth.sh"
    cleanup_hook: "/usr/local/bin/dns-cleanup.sh"
    deploy_hook: "/usr/local/bin/reload-services.sh"
```

### Let's Encrypt Configuration

These variables apply globally to all Let's Encrypt certificates, but can be overridden per-certificate.

#### `certificates_certbot_email`
- **Type**: String
- **Default**: `""`
- **Required**: Yes (for letsencrypt mode)
- **Description**: Email address for ACME account registration and notifications

```yaml
certificates_certbot_email: "admin@example.com"
```

#### `certificates_certbot_mode`
- **Type**: String
- **Default**: `production`
- **Options**: `production`, `staging`, `dry-run`
- **Description**: Certbot execution mode
  - `production`: Use production ACME directory, obtain real certificates
  - `staging`: Use staging ACME directory for testing (avoids rate limits)
  - `dry-run`: Validate configuration only, no actual certificates obtained

```yaml
# For production use
certificates_certbot_mode: production

# For testing without rate limits
certificates_certbot_mode: staging

# For validation only
certificates_certbot_mode: dry-run
```

#### `certificates_certbot_dns_provider`
- **Type**: String
- **Default**: `""`
- **Options**: `cloudflare`, `route53`, `digitalocean`, `google`, `custom`
- **Description**: Global DNS provider for DNS-01 challenges (can be overridden per-certificate)
- **Note**: Required DNS provider plugins are automatically installed based on certificate configurations

```yaml
# Global DNS provider for all DNS-01 certificates
certificates_certbot_dns_provider: cloudflare
```

#### `certificates_certbot_dns_credentials`
- **Type**: Dictionary
- **Default**: `{}`
- **Description**: Global DNS provider credentials (environment variables passed to certbot, can be overridden per-certificate)

```yaml
# For Cloudflare
certificates_certbot_dns_credentials:
  CLOUDFLARE_API_TOKEN: "your-token-here"

# For AWS Route53
certificates_certbot_dns_credentials:
  AWS_ACCESS_KEY_ID: "your-key-id"
  AWS_SECRET_ACCESS_KEY: "your-secret-key"
```

#### `certificates_acme_verify_ssl`
- **Type**: Boolean
- **Default**: `true`
- **Description**: Enable ACME server SSL certificate verification
- **Warning**: Only set to `false` for testing with self-signed ACME servers (e.g., Pebble). Always use `true` in production.

```yaml
# Production (always use true)
certificates_acme_verify_ssl: true

# Testing with Pebble ACME server
certificates_acme_verify_ssl: false
```

### File Permissions

Certificate and private key files are managed with secure defaults:
- Certificates: `0644` (root:root)
- Private keys: `0600` (root:root)
- Directories: `0755` for certificates, `0700` for private keys

For Let's Encrypt certificates, certbot manages its own permissions in `/etc/letsencrypt/`.

## Dependencies

- **Collections**: community.crypto (specified in meta/main.yml)

## Example Playbooks

### Single Server Development (Self-Signed)

```yaml
---
- name: Setup web server with self-signed certificate
  hosts: web01
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: selfsigned
        certificates_list:
          - name: web01.local
            common_name: web01.local
            organization: "Development Lab"
            days: 365

    - role: jlira.web_server.apache
      vars:
        apache_ssl_enabled: true
        apache_default_ssl_certificate_file: /etc/ssl/certs/web01.local.crt
        apache_default_ssl_certificate_key_file: /etc/ssl/private/web01.local.key
```

### Multiple Servers (Same Self-Signed Certificate)

```yaml
---
- name: Setup web servers with shared self-signed certificate
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: selfsigned
        certificates_list:
          - name: example.com
            common_name: example.com
            organization: "My Company"
            days: 730
            san:
              - DNS:www.example.com
              - DNS:api.example.com

    - role: jlira.web_server.apache
      vars:
        apache_ssl_enabled: true
        apache_default_ssl_certificate_file: /etc/ssl/certs/example.com.crt
        apache_default_ssl_certificate_key_file: /etc/ssl/private/example.com.key
```

### Import Production Certificates

```yaml
---
- name: Deploy production certificates
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: import
        certificates_list:
          - name: example.com
            cert_src: files/production/example.com.crt
            key_src: files/production/example.com.key
            chain_src: files/production/example.com-chain.crt

    - role: jlira.web_server.apache
      vars:
        apache_ssl_enabled: true
        apache_default_ssl_certificate_file: /etc/ssl/certs/example.com.crt
        apache_default_ssl_certificate_key_file: /etc/ssl/private/example.com.key
```

### Load Balancer Setup (User Orchestrates)

In this scenario, certificates are managed only on the load balancer, and web servers run HTTP-only:

```yaml
---
- name: Configure load balancer with SSL
  hosts: load_balancers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: selfsigned
        certificates_list:
          - name: example.com
            common_name: example.com
            organization: "My Company"
            san:
              - DNS:www.example.com
              - DNS:api.example.com

    # Configure your load balancer role here
    # (e.g., HAProxy, Nginx, etc.)

- name: Configure web servers without SSL
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.apache
      vars:
        apache_ssl_enabled: false  # Load balancer handles SSL
```

### Let's Encrypt with HTTP-01 (Webroot Plugin)

```yaml
---
- name: Setup Let's Encrypt with HTTP-01 webroot
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: letsencrypt
        certificates_certbot_email: "admin@example.com"
        certificates_certbot_mode: production
        certificates_list:
          - name: example.com
            domains:
              - example.com
              - www.example.com
            challenge_type: http-01
            plugin: webroot
            webroot: /var/www/html
            deploy_hook: "systemctl reload apache2"

    - role: jlira.web_server.apache
      vars:
        apache_ssl_enabled: true
        # Certbot stores certificates in /etc/letsencrypt/live/
        apache_default_ssl_certificate_file: /etc/letsencrypt/live/example.com/fullchain.pem
        apache_default_ssl_certificate_key_file: /etc/letsencrypt/live/example.com/privkey.pem
```

### Let's Encrypt with HTTP-01 (Apache Plugin)

The Apache plugin automatically configures Apache virtual hosts:

```yaml
---
- name: Setup Let's Encrypt with Apache plugin
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.apache
      # Apache must be configured first for the Apache plugin

    - role: jlira.web_server.certificates
      vars:
        certificates_mode: letsencrypt
        certificates_certbot_email: "admin@example.com"
        certificates_certbot_mode: production
        certificates_list:
          - name: example.com
            domains:
              - example.com
              - www.example.com
            challenge_type: http-01
            plugin: apache
            deploy_hook: "systemctl reload apache2"
```

### Let's Encrypt with DNS-01 (Cloudflare - Wildcard)

```yaml
---
- name: Setup Let's Encrypt wildcard certificate with DNS-01
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: letsencrypt
        certificates_certbot_email: "admin@example.com"
        certificates_certbot_mode: production
        certificates_certbot_dns_provider: cloudflare
        certificates_certbot_dns_credentials:
          CLOUDFLARE_API_TOKEN: "your-api-token-here"
        certificates_list:
          - name: wildcard.example.com
            domains:
              - example.com
              - "*.example.com"
            challenge_type: dns-01
            deploy_hook: "systemctl reload apache2"
```

### Let's Encrypt with DNS-01 (Custom Hooks)

```yaml
---
- name: Setup Let's Encrypt with custom DNS hooks
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: letsencrypt
        certificates_certbot_email: "admin@example.com"
        certificates_certbot_mode: production
        certificates_list:
          - name: custom.example.com
            domains:
              - custom.example.com
              - "*.custom.example.com"
            challenge_type: dns-01
            provider: custom
            auth_hook: "/usr/local/bin/dns-auth.sh"
            cleanup_hook: "/usr/local/bin/dns-cleanup.sh"
            deploy_hook: "systemctl reload apache2"
```

### Let's Encrypt Staging (Testing)

For testing, use staging mode to avoid rate limits:

```yaml
---
- name: Test Let's Encrypt with staging
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: letsencrypt
        certificates_certbot_email: "admin@example.com"
        certificates_certbot_mode: staging  # Use staging ACME server
        certificates_list:
          - name: test.example.com
            domains:
              - test.example.com
            challenge_type: http-01
            plugin: webroot
            webroot: /var/www/html
```

### Multiple Certificates (Mixed HTTP-01 and DNS-01)

```yaml
---
- name: Setup multiple Let's Encrypt certificates
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: letsencrypt
        certificates_certbot_email: "admin@example.com"
        certificates_certbot_mode: production
        certificates_certbot_dns_provider: cloudflare
        certificates_certbot_dns_credentials:
          CLOUDFLARE_API_TOKEN: "your-token"
        certificates_list:
          # HTTP-01 with webroot
          - name: site1.example.com
            domains:
              - site1.example.com
              - www.site1.example.com
            challenge_type: http-01
            plugin: webroot
            webroot: /var/www/site1

          # HTTP-01 with Apache plugin
          - name: site2.example.com
            domains:
              - site2.example.com
            challenge_type: http-01
            plugin: apache

          # DNS-01 wildcard with global Cloudflare credentials
          - name: wildcard.example.com
            domains:
              - "*.example.com"
            challenge_type: dns-01
```

## Tags

The role supports the following tags for selective execution:

- `certificates_validate`: Validation tasks only
- `certificates_install`: Installation and preparation tasks
- `certificates_selfsigned`: Self-signed certificate generation
- `certificates_import`: Certificate import tasks
- `certificates_letsencrypt`: Let's Encrypt certificate obtainment
- `certificates`: All certificate management tasks (excludes validation)
- `always`: Validation tasks (always run)

**Examples:**

```bash
# Only generate self-signed certificates (skip installation)
ansible-playbook playbook.yml --tags certificates_selfsigned

# Only Let's Encrypt tasks
ansible-playbook playbook.yml --tags certificates_letsencrypt

# Run everything except validation
ansible-playbook playbook.yml --skip-tags always
```

## Testing

### Manual Testing

Test certificate generation:

```bash
# Create a test playbook
cat > test-certificates.yml <<'EOF'
---
- hosts: localhost
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: selfsigned
        certificates_list:
          - name: test.local
            common_name: test.local
            organization: "Test Org"
EOF

# Run the playbook
ansible-playbook test-certificates.yml

# Verify certificate was created
openssl x509 -in /etc/ssl/certs/test.local.crt -text -noout
```

### Molecule Testing

The role includes comprehensive Molecule test scenarios:

**Main certificates scenario** - Tests all three modes (selfsigned, import, letsencrypt):
```bash
cd extensions
molecule test -s certificates

# Individual test phases
molecule create -s certificates
molecule converge -s certificates     # Tests selfsigned mode
molecule side-effect -s certificates  # Tests import and letsencrypt modes
molecule verify -s certificates       # Verifies all three modes
molecule idempotence -s certificates
molecule destroy -s certificates
```

**certificates_standalone scenario** - Tests HTTP-01 standalone and nginx plugins:
```bash
cd extensions
molecule test -s certificates_standalone
```

For detailed documentation on each test scenario:
- [extensions/molecule/certificates/README.md](../../extensions/molecule/certificates/README.md) - Main scenario
- [extensions/molecule/certificates_standalone/README.md](../../extensions/molecule/certificates_standalone/README.md) - Standalone/nginx plugins

## Troubleshooting

### Let's Encrypt Issues

#### HTTP-01 Challenge Fails

**Symptoms**: Challenge fails with connection errors

**Solutions**:
1. Ensure port 80 is accessible from the internet
2. Verify webroot directory exists and is writable: `ls -la /var/www/html/.well-known/acme-challenge`
3. Check that your domain resolves to the server's IP: `dig example.com`
4. Verify no firewall is blocking port 80
5. Check Apache/Nginx is serving the challenge directory

#### DNS-01 Challenge Fails

**Symptoms**: Challenge fails with DNS verification errors

**Solutions**:
1. Verify DNS provider credentials are correct in `certificates_certbot_dns_credentials`
2. Ensure DNS API access is enabled in your DNS provider
3. Check DNS propagation: `dig TXT _acme-challenge.example.com`
4. Wait for DNS propagation (can take up to 10 minutes)
5. Verify DNS provider is supported (cloudflare, route53, digitalocean, google)
6. For custom hooks, check that `auth_hook` and `cleanup_hook` scripts exist and are executable

#### Rate Limiting

**Symptoms**: "too many certificates" or "too many registrations" errors

**Solutions**:
1. Use staging mode for testing: `certificates_certbot_mode: staging`
2. Review [Let's Encrypt rate limits](https://letsencrypt.org/docs/rate-limits/)
3. Wait for rate limit reset (varies by limit type)

### Certificate Renewal

Certbot automatically sets up renewal via its systemd timer (`certbot.timer`). Certificates are renewed automatically when they have 30 days or less remaining.

**Check renewal status:**
```bash
# Check certbot timer status
systemctl status certbot.timer

# List certificates and expiration dates
certbot certificates

# Test renewal (dry run)
certbot renew --dry-run

# Check renewal logs
journalctl -u certbot.service
```

### Adding Domains to Existing Certificates

The role automatically handles domain expansion when you add new domains to an existing certificate. Simply update your `certificates_list` with the additional domains and re-run the playbook:

```yaml
certificates_list:
  - name: example.com
    domains:
      - example.com
      - www.example.com
      - api.example.com  # New domain added
    challenge_type: http-01
    plugin: webroot
    webroot: /var/www/html
```

The role uses certbot's `--expand` flag automatically, which:
- ✅ Expands the certificate to include the new domains
- ✅ Replaces the old certificate with the expanded one
- ✅ Maintains the same certificate name
- ✅ Works idempotently (safe to run multiple times)
- ✅ No manual intervention required

**Note**: You cannot remove domains from an existing certificate. To remove domains, you must delete the old certificate and create a new one with the desired domain list.

### Permission Issues

**Symptoms**: Permission denied errors

**Solutions**:
1. Ensure role is run with root/sudo privileges (`become: true`)
2. Check directory permissions: `ls -la /etc/ssl/certs /etc/ssl/private`
3. For Let's Encrypt: `ls -la /etc/letsencrypt`
4. Check SELinux contexts if using SELinux

### Certificate Not Trusted in Browser

**Symptoms**: Browser shows "Not Secure" or certificate warnings

**Solutions**:
1. **Self-signed certificates**: This is expected behavior. Add exception in browser or import CA certificate
2. **Let's Encrypt staging**: Staging certificates are not trusted. Use `certificates_certbot_mode: production`
3. **Let's Encrypt production**: Verify certificate chain: `openssl s_client -connect example.com:443 -showcerts`
4. Check Apache/Nginx is configured to use fullchain certificate: `/etc/letsencrypt/live/example.com/fullchain.pem`

### Debug Mode

Run playbook with verbose output:

```bash
ansible-playbook playbook.yml -vvv
```

Check certbot logs:
```bash
# Recent certbot operations
tail -f /var/log/letsencrypt/letsencrypt.log

# Specific certificate
grep "example.com" /var/log/letsencrypt/letsencrypt.log
```

## Certificate Management Best Practices

### Production Deployments

1. **Start with Staging**: Always test with `certificates_certbot_mode: staging` before switching to production
2. **Monitor Expiration**: Use `certbot certificates` to monitor expiration dates
3. **Test Renewal**: Run `certbot renew --dry-run` to verify renewal will work
4. **Backup Certificates**: Backup `/etc/letsencrypt/` regularly (especially before major changes)
5. **Deploy Hooks**: Use `deploy_hook` to reload services after certificate renewal

### Domain Changes

**Adding domains:**
```yaml
# Safe - uses --expand flag automatically
certificates_list:
  - name: example.com
    domains:
      - example.com
      - www.example.com
      - api.example.com  # Adding new domain
```

**Removing domains:**
```bash
# Manual process required
1. Delete the certificate: certbot delete --cert-name example.com
2. Update your certificates_list to remove unwanted domains
3. Re-run the playbook to obtain a new certificate
```

### Multiple Certificates

For better organization, use separate certificates for different services:

```yaml
certificates_list:
  # Web application
  - name: web.example.com
    domains:
      - example.com
      - www.example.com

  # API service
  - name: api.example.com
    domains:
      - api.example.com
      - api-v2.example.com

  # Admin panel
  - name: admin.example.com
    domains:
      - admin.example.com
```

This approach provides:
- ✅ Easier certificate management
- ✅ Independent renewal cycles
- ✅ Service-specific deploy hooks
- ✅ Better troubleshooting

## Future Enhancements

These features may be added in future versions without breaking existing functionality:

### Advanced Features
- Integration with HashiCorp Vault
- Integration with cloud secret managers (AWS Secrets Manager, GCP Secret Manager, Azure Key Vault)
- Certificate distribution helpers (for multi-server deployments)
- Load balancer certificate formatting (HAProxy combined PEM, etc.)
- Additional DNS provider support for DNS-01 challenge
- Certificate backup and restore functionality
- Prometheus metrics export for monitoring
- Automatic certificate revocation support

## License

GPL-2.0-or-later

## Author

Julio Lira (yurgenlira@hotmail.com)

---

For more information and examples, see:
- [Collection README](../../README.md)
- [Apache Role README](../apache/README.md)
- [Molecule Tests README](../../extensions/molecule/certificates/README.md)
- [defaults/main.yml](defaults/main.yml) - Complete variable documentation