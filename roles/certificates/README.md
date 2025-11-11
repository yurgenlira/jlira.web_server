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
- [Certificate Expiration Monitoring](#certificate-expiration-monitoring)
- [Automatic Renewal](#automatic-renewal)
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
- **Let's Encrypt Integration**:
  - HTTP-01 challenge support for domain verification
  - DNS-01 challenge support for wildcard certificates
  - Automatic certificate renewal via systemd timer
  - Support for production and staging ACME directories
- **Certificate Expiration Monitoring**: Track certificate expiration dates with configurable thresholds
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
- `domains`: List of domain names for the certificate (required)
- `challenge_type`: Override global challenge type for this cert (optional)
- `webroot`: Override global webroot for HTTP-01 challenge (optional)

**Example for letsencrypt mode:**
```yaml
certificates_list:
  - name: example.com
    domains:
      - example.com
      - www.example.com
    challenge_type: http-01
    webroot: /var/www/example.com
```

### Let's Encrypt Configuration

#### `certificates_acme_directory`
- **Type**: String
- **Default**: `https://acme-v02.api.letsencrypt.org/directory`
- **Description**: Let's Encrypt ACME directory URL (use staging for testing)

#### `certificates_acme_email`
- **Type**: String
- **Default**: `""`
- **Required**: Yes (for letsencrypt mode)
- **Description**: Email address for ACME account registration

#### `certificates_acme_agree_tos`
- **Type**: Boolean
- **Default**: `false`
- **Required**: Yes (must be true for letsencrypt mode)
- **Description**: Agreement to Let's Encrypt Terms of Service

#### `certificates_acme_challenge_type`
- **Type**: String
- **Default**: `http-01`
- **Options**: `http-01`, `dns-01`
- **Description**: ACME challenge type for domain verification

#### `certificates_acme_webroot`
- **Type**: String
- **Default**: `/var/www/html`
- **Description**: Web server document root for HTTP-01 challenge

#### `certificates_acme_auto_renew`
- **Type**: Boolean
- **Default**: `true`
- **Description**: Enable automatic certificate renewal via systemd timer

#### `certificates_acme_renew_days_before_expiry`
- **Type**: Integer
- **Default**: `30`
- **Description**: Days before expiry to attempt renewal

### Certificate Expiration Monitoring

#### `certificates_monitor_expiration`
- **Type**: Boolean
- **Default**: `false`
- **Description**: Enable certificate expiration monitoring

#### `certificates_expiration_warning_days`
- **Type**: Integer
- **Default**: `30`
- **Description**: Warning threshold in days before expiry

#### `certificates_expiration_critical_days`
- **Type**: Integer
- **Default**: `7`
- **Description**: Critical threshold in days before expiry

#### `certificates_expiration_action`
- **Type**: String
- **Default**: `log`
- **Options**: `log`, `fail`, `notify`
- **Description**: Action to take when certificate is expiring

### Default Values

#### `certificates_default_days`
- **Type**: Integer
- **Default**: `365`
- **Description**: Default certificate validity in days (for selfsigned mode)

#### `certificates_default_key_size`
- **Type**: Integer
- **Default**: `4096`
- **Description**: Default RSA key size in bits (for selfsigned mode)

### File Permissions

#### `certificates_cert_mode`
- **Type**: String (octal)
- **Default**: `"0644"`
- **Description**: Certificate file permissions

#### `certificates_key_mode`
- **Type**: String (octal)
- **Default**: `"0600"`
- **Description**: Private key file permissions

#### `certificates_owner` / `certificates_group`
- **Type**: String
- **Default**: `root`
- **Description**: Owner and group for certificate files

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

### HTTP-01 Challenge Example

```yaml
---
- name: Setup Let's Encrypt with HTTP-01 challenge
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: letsencrypt
        certificates_acme_email: "admin@example.com"
        certificates_acme_agree_tos: true
        certificates_acme_challenge_type: http-01
        certificates_acme_webroot: /var/www/html
        certificates_acme_auto_renew: true
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

### DNS-01 Challenge Example (Wildcard Certificates)

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
        certificates_acme_auto_renew: true
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

## Certificate Expiration Monitoring

Monitor certificate expiration and take action when certificates are about to expire:

```yaml
---
- name: Setup certificate expiration monitoring
  hosts: web_servers
  become: true
  roles:
    - role: jlira.web_server.certificates
      vars:
        certificates_mode: selfsigned
        certificates_monitor_expiration: true
        certificates_expiration_warning_days: 30
        certificates_expiration_critical_days: 7
        certificates_expiration_action: fail  # Fail playbook if certificate is critical
        certificates_list:
          - name: example.com
            common_name: example.com
            days: 90
```

## Automatic Renewal

When using Let's Encrypt with `certificates_acme_auto_renew: true`, a systemd timer is automatically configured to check for and renew certificates twice daily.

### Check Renewal Timer Status

```bash
sudo systemctl status cert-renew.timer
sudo systemctl list-timers cert-renew.timer
```

### Manually Trigger Renewal

```bash
sudo systemctl start cert-renew.service
```

### View Renewal Logs

```bash
sudo journalctl -u cert-renew.service
```

### Disable Automatic Renewal

```yaml
certificates_acme_auto_renew: false
```

## Tags

The role supports the following tags for selective execution:

- `certificates_validate`: Validation tasks only
- `certificates_install`: Installation and preparation tasks
- `certificates_selfsigned`: Self-signed certificate generation
- `certificates_import`: Certificate import tasks
- `certificates_letsencrypt`: Let's Encrypt certificate obtainment
- `certificates_renewal`: Automatic renewal setup
- `certificates_monitor`: Certificate expiration monitoring
- `certificates`: All certificate management tasks (excludes validation)
- `always`: Validation tasks (always run)

**Examples:**

```bash
# Only generate certificates (skip installation)
ansible-playbook playbook.yml --tags certificates_selfsigned

# Only Let's Encrypt tasks
ansible-playbook playbook.yml --tags certificates_letsencrypt

# Setup renewal only
ansible-playbook playbook.yml --tags certificates_renewal

# Monitor expiration only
ansible-playbook playbook.yml --tags certificates_monitor

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