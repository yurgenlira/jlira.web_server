# jlira.web_server.certificates

A comprehensive Ansible role for managing SSL/TLS certificates on Debian/Ubuntu systems. Supports self-signed certificate generation, importing existing certificates, Let's Encrypt integration, certificate expiration monitoring, and automatic renewal.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Role Variables](#role-variables)
- [Dependencies](#dependencies)
- [Example Playbooks](#example-playbooks)
- [Let's Encrypt Integration](#lets-encrypt-integration)
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
- **letsencrypt**: Obtain trusted certificates from Let's Encrypt with HTTP-01 or DNS-01 challenges

## Features

- **Simple and Generic**: Works with single servers, multiple servers, and load balancer setups
- **Three Operation Modes**: Generate self-signed certificates, import existing ones, or obtain from Let's Encrypt
- **Let's Encrypt Integration with Certbot**:
  - HTTP-01 challenge support for domain verification (webroot, standalone, Apache, Nginx plugins)
  - DNS-01 challenge support for wildcard certificates
  - Automatic certificate renewal via certbot's built-in systemd timer
  - Support for production and staging ACME directories
  - Deploy hooks for service reloads after renewal
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
- **Description**: Output directory for certificate files

```yaml
certificates_cert_dir: /etc/ssl/certs
```

#### `certificates_key_dir`
- **Type**: String
- **Default**: `/etc/ssl/private`
- **Description**: Output directory for private key files

```yaml
certificates_key_dir: /etc/ssl/private
```

### Certificate List

#### `certificates_list`
- **Type**: List of Dictionaries
- **Default**: `[]`
- **Description**: List of certificates to manage

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

**Fields for import mode:**
- `cert_src`: Source path to certificate file on control machine (required)
- `key_src`: Source path to private key file on control machine (required)
- `chain_src`: Source path to certificate chain file (optional)

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
- `provider`: DNS provider override (optional, uses global if not set)
- `dns_credentials`: DNS credentials override (optional, uses global if not set)
- `auth_hook`: Authentication script for custom provider (required when `certificates_certbot_dns_provider: custom`)
- `cleanup_hook`: Cleanup script for custom provider (required when `certificates_certbot_dns_provider: custom`)

**Optional for all challenges:**
- `deploy_hook`: Command to run after successful obtain/renew

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
  - name: example.com
    domains:
      - example.com
      - www.example.com
    challenge_type: http-01
    plugin: apache
```

**Example for letsencrypt mode (DNS-01 wildcard):**
```yaml
certificates_list:
  - name: wildcard.example.com
    domains:
      - example.com
      - "*.example.com"
    challenge_type: dns-01
```

### Let's Encrypt Configuration

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
- **Description**: DNS provider for DNS-01 challenges (applies globally unless overridden per-certificate)
- **Note**: Required DNS provider plugins are automatically installed based on certificate configurations

```yaml
certificates_certbot_dns_provider: cloudflare
```

#### `certificates_certbot_dns_credentials`
- **Type**: Dictionary
- **Default**: `{}`
- **Description**: Environment variables for DNS provider authentication (can be overridden per-certificate)

```yaml
# For Cloudflare
certificates_certbot_dns_credentials:
  CLOUDFLARE_API_TOKEN: "your-token-here"

# For AWS Route53
certificates_certbot_dns_credentials:
  AWS_ACCESS_KEY_ID: "your-key-id"
  AWS_SECRET_ACCESS_KEY: "your-secret-key"
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

### Multiple Certificates

```yaml
---
- name: Generate multiple certificates
  hosts: webserver
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: selfsigned
        certificates_list:
          - name: site1.example.com
            common_name: site1.example.com
            organization: "Company A"
            days: 365

          - name: site2.example.com
            common_name: site2.example.com
            organization: "Company B"
            days: 730
            san:
              - DNS:www.site2.example.com
              - DNS:api.site2.example.com

          - name: site3.example.com
            common_name: site3.example.com
            organization: "Company C"
            days: 365
```

### Advanced Self-Signed Certificate

```yaml
---
- name: Generate advanced self-signed certificate
  hosts: webserver
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: selfsigned
        certificates_list:
          - name: secure.example.com
            common_name: secure.example.com
            organization: "Acme Corporation"
            organizational_unit: "Security Department"
            country: "US"
            state: "California"
            locality: "San Francisco"
            email: "security@example.com"
            days: 730
            key_size: 4096
            san:
              - DNS:secure.example.com
              - DNS:www.secure.example.com
              - DNS:api.secure.example.com
              - IP:192.168.1.100
              - IP:10.0.0.100
```

## Let's Encrypt Integration

This role uses **certbot** (the official Let's Encrypt client) for obtaining and managing Let's Encrypt certificates. Certbot provides:

- Automatic certificate obtainment with various challenge methods
- Built-in automatic renewal via systemd timer
- Deploy hooks for service reloads
- Support for multiple plugins (webroot, standalone, apache, nginx, DNS providers)

### Certbot Configuration Variables

#### `certificates_certbot_plugin`
- **Type**: String
- **Default**: `webroot`
- **Options**: `webroot`, `standalone`, `apache`, `nginx`
- **Description**: Certbot plugin to use for HTTP-01 challenges
- **Note**: When set to `apache` or `nginx`, the corresponding plugin package (`python3-certbot-apache` or `python3-certbot-nginx`) is automatically installed

#### `certificates_certbot_plugins`
- **Type**: List
- **Default**: `[]`
- **Description**: Additional certbot plugins to install (e.g., DNS provider plugins)
- **Note**: Only needed for DNS providers or other additional plugins. Apache and Nginx plugins are installed automatically based on `certificates_certbot_plugin`.

```yaml
# Apache/Nginx plugins are installed automatically, no need to specify
certificates_certbot_plugin: apache  # Automatically installs python3-certbot-apache

# Only specify additional plugins like DNS providers
certificates_certbot_plugins:
  - python3-certbot-dns-cloudflare
  - python3-certbot-dns-route53
```

#### `certificates_certbot_copy_certs`
- **Type**: Boolean
- **Default**: `true`
- **Description**: Copy certificates from /etc/letsencrypt/live/ to custom locations for backward compatibility

#### `certificates_certbot_deploy_hook`
- **Type**: String
- **Default**: `""`
- **Description**: Command to run after successful renewal (e.g., service reload)

```yaml
# Reload Apache after renewal
certificates_certbot_deploy_hook: "systemctl reload apache2"

# Reload multiple services
certificates_certbot_deploy_hook: "systemctl reload apache2 && systemctl reload nginx"
```

#### `certificates_acme_webroot_owner`
- **Type**: String
- **Default**: `www-data`
- **Description**: Owner of the webroot directory for HTTP-01 challenge (varies by web server)

#### `certificates_acme_webroot_group`
- **Type**: String
- **Default**: `www-data`
- **Description**: Group of the webroot directory for HTTP-01 challenge (varies by web server)

```yaml
# For Nginx on Debian/Ubuntu
certificates_acme_webroot_owner: www-data
certificates_acme_webroot_group: www-data

# For Nginx on RHEL/CentOS
certificates_acme_webroot_owner: nginx
certificates_acme_webroot_group: nginx

# For Apache on RHEL/CentOS
certificates_acme_webroot_owner: apache
certificates_acme_webroot_group: apache
```

### HTTP-01 Challenge Example (Webroot Plugin)

```yaml
---
- name: Setup Let's Encrypt with HTTP-01 challenge (webroot)
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: letsencrypt
        certificates_acme_email: "admin@example.com"
        certificates_acme_agree_tos: true
        certificates_acme_challenge_type: http-01
        certificates_certbot_plugin: webroot
        certificates_acme_webroot: /var/www/html
        certificates_certbot_deploy_hook: "systemctl reload apache2"
        certificates_list:
          - name: example.com
            domains:
              - example.com
              - www.example.com

    - role: jlira.web_server.apache
      vars:
        apache_ssl_enabled: true
        apache_virtual_hosts:
          - name: example.com
            server_name: example.com
            document_root: /var/www/html
            https:
              certificate_file: /etc/ssl/certs/example.com.crt
              certificate_key_file: /etc/ssl/private/example.com.key
```

### HTTP-01 Challenge Example (Nginx with Webroot)

For Nginx servers, configure the webroot owner to match your web server:

```yaml
---
- name: Setup Let's Encrypt with Nginx (webroot)
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: letsencrypt
        certificates_acme_email: "admin@example.com"
        certificates_acme_agree_tos: true
        certificates_acme_challenge_type: http-01
        certificates_certbot_plugin: webroot
        certificates_acme_webroot: /var/www/html
        certificates_acme_webroot_owner: www-data  # Nginx user on Debian/Ubuntu
        certificates_acme_webroot_group: www-data
        certificates_certbot_deploy_hook: "systemctl reload nginx"
        certificates_list:
          - name: example.com
            domains:
              - example.com
              - www.example.com
```

### HTTP-01 Challenge Example (Apache Plugin)

The Apache plugin automatically configures Apache virtual hosts:

```yaml
---
- name: Setup Let's Encrypt with Apache plugin
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: letsencrypt
        certificates_acme_email: "admin@example.com"
        certificates_acme_agree_tos: true
        certificates_acme_challenge_type: http-01
        certificates_certbot_plugin: apache  # Plugin automatically installed
        certificates_certbot_deploy_hook: "systemctl reload apache2"
        certificates_list:
          - name: example.com
            domains:
              - example.com
              - www.example.com
```

### DNS-01 Challenge Example (Wildcard Certificates)

For wildcard certificates, use DNS-01 challenge with a DNS provider plugin:

```yaml
---
- name: Setup Let's Encrypt with DNS-01 challenge for wildcard
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: letsencrypt
        certificates_acme_email: "admin@example.com"
        certificates_acme_agree_tos: true
        certificates_acme_challenge_type: dns-01
        certificates_acme_dns_provider: "cloudflare"
        certificates_certbot_plugins:
          - python3-certbot-dns-cloudflare
        certificates_acme_dns_credentials:
          CLOUDFLARE_API_TOKEN: "your-api-token-here"
        certificates_certbot_deploy_hook: "systemctl reload apache2"
        certificates_list:
          - name: example.com
            domains:
              - example.com
              - "*.example.com"  # Wildcard supported with DNS-01
```

### Using Let's Encrypt Staging Environment

For testing, use the staging environment to avoid rate limits:

```yaml
---
- name: Test Let's Encrypt with staging
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: letsencrypt
        certificates_acme_directory: https://acme-staging-v02.api.letsencrypt.org/directory
        certificates_acme_email: "admin@example.com"
        certificates_acme_agree_tos: true
        certificates_list:
          - name: test.example.com
            domains:
              - test.example.com
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
# Only generate certificates (skip installation)
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

The role can be tested using Molecule:

```bash
# Create and test
cd extensions
molecule create -s certificates
molecule converge -s certificates
molecule verify -s certificates

# Check idempotency
molecule idempotence -s certificates

# Clean up
molecule destroy -s certificates
```

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
1. Verify DNS provider credentials are correct
2. Ensure DNS API access is enabled in your DNS provider
3. Check DNS propagation: `dig TXT _acme-challenge.example.com`
4. Wait for DNS propagation (can take up to 10 minutes)
5. Verify DNS provider is supported

#### Rate Limiting

**Symptoms**: "too many certificates" or "too many registrations" errors

**Solutions**:
1. Use staging environment for testing: `certificates_acme_directory: https://acme-staging-v02.api.letsencrypt.org/directory`
2. Review [Let's Encrypt rate limits](https://letsencrypt.org/docs/rate-limits/)
3. Wait for rate limit reset (varies by limit type)

### Certificate Renewal Issues

#### Timer Not Running

**Symptoms**: Certificates not renewing automatically

**Solutions**:
1. Check timer status: `systemctl status cert-renew.timer`
2. Check service logs: `journalctl -u cert-renew.service`
3. Verify systemd timer is enabled: `systemctl is-enabled cert-renew.timer`
4. Manually start timer: `systemctl start cert-renew.timer`

#### Renewal Fails

**Symptoms**: Renewal service fails with errors

**Solutions**:
1. Check playbook and inventory paths in `/etc/systemd/system/cert-renew.service`
2. Test manual renewal: `systemctl start cert-renew.service`
3. Verify Ansible is installed and accessible
4. Check file permissions on certificate directories

### Permission Issues

**Symptoms**: Permission denied errors

**Solutions**:
1. Ensure role is run with root/sudo privileges
2. Check directory permissions: `ls -la /etc/ssl/certs /etc/ssl/private`
3. Verify ownership matches `certificates_owner` and `certificates_group`
4. Check SELinux contexts if using SELinux

### Certificate Not Trusted in Browser

**Symptoms**: Browser shows "Not Secure" or certificate warnings

**Solutions**:
1. **Self-signed certificates**: This is expected behavior. Add exception in browser or import CA certificate
2. **Let's Encrypt**: Ensure you're using production ACME directory, not staging
3. Verify certificate chain is correct: `openssl s_client -connect example.com:443 -showcerts`
4. Check Apache/Nginx is configured to use full chain certificate

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

## License

GPL-2.0-or-later

## Author

Julio Lira (yurgenlira@hotmail.com)

---

For more information and examples, see:
- [Collection README](../../README.md)
- [Apache Role README](../apache/README.md)
- [defaults/main.yml](defaults/main.yml) - Complete variable documentation
