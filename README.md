# Ansible Collection - jlira.web_server

[![License](https://img.shields.io/badge/license-GPL--2.0--or--later-blue.svg)](https://spdx.org/licenses/GPL-2.0-or-later.html)
[![Ansible](https://img.shields.io/badge/ansible-%3E%3D2.18.7-blue.svg)](https://www.ansible.com/)

A comprehensive Ansible collection for deploying, configuring, and managing web server infrastructure on Debian/Ubuntu systems. This collection provides production-ready roles for Apache HTTP Server and PHP with extensive configuration options, security hardening, and monitoring capabilities.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Roles](#roles)
- [Playbooks](#playbooks)
- [Common Use Cases](#common-use-cases)
- [Testing](#testing)
- [Contributing](#contributing)
- [License](#license)
- [Author](#author)
- [Support](#support)

## Overview

The `jlira.web_server` collection simplifies web server deployment and management by providing:

- **Apache HTTP Server Management**: Complete lifecycle management including installation, configuration, virtual hosts, SSL/TLS, security hardening, and monitoring
- **PHP Management**: Multi-version PHP support with PHP-FPM, extensive configuration options, and optional components like Composer and MSSQL drivers
- **Certificate Management**: Automated SSL/TLS certificate management with support for self-signed certificates, Let's Encrypt (via Certbot), and certificate import
- **Decoupled Architecture**: Roles follow single-purpose design - Apache and PHP are independent, allowing flexible deployment patterns
- **Integration Support**: Seamless Apache-PHP integration when both roles are used together
- **Ready-to-Use Playbooks**: Pre-built playbooks for common deployment scenarios
- **Production-Ready**: Security-hardened defaults, comprehensive validation, and idempotent operations

## Features

### Apache Role Features

- ✅ **Installation & Service Management**: Automated Apache installation and service lifecycle management
- ✅ **SSL/TLS Support**: Built-in SSL configuration with support for custom certificates
- ✅ **Execution Phases**: Split execution for certificate generation workflows (http_only, https_only, all)
- ✅ **Virtual Host Management**:
  - Direct virtual host configuration
  - Template-based virtual hosts for reusability
  - HTTP and HTTPS support with phase-aware configuration
  - Custom directives and includes
  - Environment variable substitution in paths
- ✅ **PHP-FPM Integration** (when used with PHP role):
  - Apache-side proxy configuration
  - Multiple PHP version support
  - Version-specific configuration
  - **Note**: PHP installation handled by separate PHP role
- ✅ **Security Hardening**:
  - Secure defaults (ServerTokens, ServerSignature, TraceEnable)
  - Custom security configurations
  - HTTP method restrictions
- ✅ **Monitoring**: Apache and PHP-FPM status pages with access control
- ✅ **HTTP Authentication**: htpasswd file and user management
- ✅ **Custom Configuration**: Support for arbitrary Apache configuration files
- ✅ **Port Management**: Configure multiple HTTP/HTTPS ports
- ✅ **Module Management**: Enable/disable Apache modules
- ✅ **Environment Variables**: Custom Apache environment variables

### PHP Role Features

- ✅ **Multi-Version Support**: Install and manage multiple PHP versions simultaneously
- ✅ **Version Management**: Set default PHP CLI version using Debian alternatives
- ✅ **PHP-FPM**: Complete FPM pool configuration and management
- ✅ **Comprehensive Configuration**:
  - Independent CLI and FPM php.ini settings
  - FPM pool settings (pm, max_children, etc.)
  - Custom php.ini directives
- ✅ **Extensions & Packages**: Flexible package selection from Ondrej's PPA
- ✅ **Composer Support**: Optional global Composer installation
- ✅ **MSSQL Driver**: Optional Microsoft SQL Server PHP extension
- ✅ **Log Management**: Configurable logrotate for PHP logs
- ✅ **Status Monitoring**: PHP-FPM status and ping endpoints
- ✅ **Upgrade Support**: Clean migration between PHP versions
- ✅ **Idempotent Operations**: All tasks support check mode and are fully idempotent

### Certificates Role Features

- ✅ **Self-Signed Certificates**: Generate self-signed certificates with customizable parameters (key size, validity, SANs, subject fields)
- ✅ **Let's Encrypt Integration**: Automated certificate issuance and renewal via Certbot
  - HTTP-01 challenge support for standard domains
  - DNS-01 challenge support for wildcard certificates
  - Support for multiple DNS providers (Cloudflare, Route53, etc.)
  - Custom DNS authentication hooks
  - Automatic renewal via systemd timer
- ✅ **Certificate Import**: Import existing certificates, private keys, and chain files
- ✅ **Auto-Detection**: Automatically selects appropriate mode based on domain TLD
- ✅ **Flexible Configuration**: Per-certificate customization of all parameters
- ✅ **Security**: Proper file permissions and ownership for certificates and keys
- ✅ **Idempotent**: Safe to run multiple times without unnecessary changes
- ✅ **Comprehensive Testing**: Full Molecule test suite with Pebble ACME server

## Requirements

- **Ansible**: >= 2.18.7
- **Target OS**: Debian 11+ or Ubuntu 20.04+
- **Privileges**: Root or sudo access (`become: true`)
- **Python**: Python 3.6+ on target hosts
- **Internet Access**: Required for package installation (or configured local mirrors)

## Installation

### From Ansible Galaxy (Recommended)

Install the collection from Ansible Galaxy. Dependencies will be installed automatically:

```bash
ansible-galaxy collection install jlira.web_server
```

This will automatically install the required dependencies:
- `community.crypto` (>= 2.0.0)
- `community.general` (>= 3.0.0)

### From Source

Clone the repository and build the collection:

```bash
git clone https://github.com/yurgenlira/jlira.web_server.git
cd jlira.web_server
ansible-galaxy collection build
```

Install the built collection and its dependencies:

```bash
# Install the collection
ansible-galaxy collection install jlira-web_server-*.tar.gz

# Install dependencies
ansible-galaxy collection install -r galaxy.yml
```

**Note**: When installing from a local tarball, dependencies are not installed automatically. You must manually install them using the second command.

### Using requirements.yml

Create a `requirements.yml` file:

```yaml
---
collections:
  - name: jlira.web_server
    version: ">=1.0.0"
```

Install the collection:

```bash
ansible-galaxy collection install -r requirements.yml
```

Dependencies (`community.crypto` and `community.general`) will be installed automatically when installing from Ansible Galaxy.

## Quick Start

### Basic Web Server Setup

Create a playbook (`webserver.yml`):

```yaml
---
- name: Setup web server
  hosts: webservers
  become: true

  roles:
    # Install PHP first
    - role: jlira.web_server.php
      vars:
        php_version: "8.4"
        php_fpm_enabled: true
        php_composer_enabled: true

    # Then configure Apache with PHP-FPM integration
    - role: jlira.web_server.apache
      vars:
        apache_server_name: "example.com"
        apache_ssl_enabled: true
        apache_php_fpm_integration: true
        apache_php_fpm_version: "8.4"
        apache_modules_enabled:
          - rewrite
          - headers
```

Run the playbook:

```bash
ansible-playbook -i inventory webserver.yml
```

### Using Pre-built Playbooks

The collection includes ready-to-use playbooks:

```bash
# Basic web server (Apache + PHP)
ansible-playbook jlira.web_server.apache_setup -i inventory

# Multi-PHP version setup
ansible-playbook -i inventory jlira.web_server.playbooks.web-server-multi-php
```

## Roles

### Apache Role (`jlira.web_server.apache`)

Comprehensive Apache HTTP Server management.

**Key Variables:**

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `apache_server_name` | string | `{{ ansible_hostname }}` | Server hostname |
| `apache_ssl_enabled` | boolean | `true` | Enable SSL support |
| `apache_execution_phase` | string | `"all"` | Execution phase: `all`, `http_only`, `https_only` |
| `apache_http_ports` | list | `[80]` | HTTP ports |
| `apache_https_ports` | list | `[443]` | HTTPS ports |
| `apache_php_fpm_integration` | boolean | `false` | Configure Apache for PHP-FPM |
| `apache_virtual_hosts` | list | `[]` | Virtual hosts config |
| `apache_modules_enabled` | list | `[]` | Modules to enable |

**Documentation:** [Apache Role README](roles/apache/README.md)

### PHP Role (`jlira.web_server.php`)

Modern PHP installation and configuration with multi-version support.

**Key Variables:**

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `php_version` | string | `"8.4"` | PHP version |
| `php_fpm_enabled` | boolean | `false` | Enable PHP-FPM |
| `php_composer_enabled` | boolean | `false` | Install Composer |
| `php_cli_set_as_default_version` | boolean | `true` | Set as default CLI |
| `php_packages` | list | See defaults | Packages to install |
| `php_pool_settings` | dict | See defaults | FPM pool settings |

**Documentation:** [PHP Role README](roles/php/README.md)

### Certificates Role (`jlira.web_server.certificates`)

Automated SSL/TLS certificate management for web servers.

**Key Variables:**

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `certificates_items` | list | `[]` | List of certificates to manage |
| `certificates_cert_dir` | string | `/etc/ssl/certs` | Certificate directory |
| `certificates_key_dir` | string | `/etc/ssl/private` | Private key directory |
| `certificates_certbot_email` | string | `admin@example.com` | Let's Encrypt email |
| `certificates_certbot_environment` | string | `production` | Certbot environment |

**Certificate Item Variables:**

| Variable | Required | Description |
|----------|----------|-------------|
| `name` | Yes | Certificate name |
| `mode` | No | Mode: `auto`, `selfsigned`, `letsencrypt`, or `imported` |
| `challenge_type` | For LE | Challenge type: `http-01` or `dns-01` |
| `webroot` | For HTTP-01 | Webroot path |
| `dns_provider` | For DNS-01 | DNS provider name |
| `src_cert` | For import | Source certificate file |
| `src_key` | For import | Source private key file |

**Documentation:** [Certificates Role README](roles/certificates/README.md)

## Playbooks

The collection includes several pre-built playbooks for common scenarios:

### Web Server Setup Playbooks

| Playbook | Description | Use Case |
|----------|-------------|----------|
| `apache_setup` | Orchestrated Apache + PHP + SSL | Complete web server setup with multi-phase execution for SSL challenges |
| `web-server-multi-php` | Apache + Multiple PHP versions | Multi-version PHP environment |
| `php-install` | PHP installation only | Add PHP to existing server |
| `php-upgrade` | PHP version upgrade | Migrate between PHP versions |
| `php-uninstall` | PHP removal | Clean up PHP installation |

**Documentation:** [Playbooks README](playbooks/README.md)

## Common Use Cases

### 1. Orchestrated Web Server Setup (Recommended)

The `apache_setup.yml` playbook provides a robust, multi-phase setup that handles the "chicken-and-egg" problem of generating SSL certificates for new domains.

```yaml
# inventory/hosts.yml
all:
  children:
    webservers:
      hosts:
        myserver:
      vars:
        # Enable orchestration
        certificate_generation: true

        # PHP Configuration
        php_version: "8.3"
        php_fpm_enabled: true

        # Apache Configuration
        apache_server_name: "example.com"
        apache_ssl_enabled: true
        apache_php_fpm_integration: true

        # Virtual Hosts
        apache_virtual_hosts:
          - name: example.com
            server_name: example.com
            document_root: /var/www/example
            http:
              custom_directives: "Redirect permanent / https://example.com/"
            https:
              certificate_file: /etc/letsencrypt/live/example.com/fullchain.pem
              certificate_key_file: /etc/letsencrypt/live/example.com/privkey.pem
```

Run the playbook:
```bash
ansible-playbook -i inventory playbooks/apache_setup.yml
```

### 2. Basic LAMP Stack

```yaml
---
- name: Deploy LAMP stack
  hosts: webservers
  become: true
  roles:
    # Install PHP first
    - role: jlira.web_server.php
      vars:
        php_version: "8.4"
        php_fpm_enabled: true
        php_packages:
          - "php8.4-cli"
          - "php8.4-fpm"
          - "php8.4-mysql"
          - "php8.4-mbstring"
          - "php8.4-xml"

    # Then configure Apache with PHP integration
    - role: jlira.web_server.apache
      vars:
        apache_server_name: "lamp.example.com"
        apache_ssl_enabled: true
        apache_php_fpm_integration: true
        apache_php_fpm_version: "{{ php_version }}"
```

### 3. Virtual Host with SSL

```yaml
---
- name: Configure virtual host with SSL
  hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.apache
      vars:
        apache_virtual_hosts:
          - name: myapp.com
            server_name: myapp.com
            server_alias:
              - www.myapp.com
            document_root: /var/www/myapp
            http:
              custom_directives: |
                Redirect permanent / https://myapp.com/
            https:
              certificate_file: /etc/ssl/certs/myapp.crt
              certificate_key_file: /etc/ssl/private/myapp.key
              custom_directives: |
                <Directory /var/www/myapp>
                    Options -Indexes +FollowSymLinks
                    AllowOverride All
                    Require all granted
                </Directory>
```

### 4. Multiple PHP Versions

```yaml
---
- name: Setup multiple PHP versions
  hosts: webservers
  become: true
  tasks:
    # Install PHP 8.3
    - ansible.builtin.include_role:
        name: jlira.web_server.php
      vars:
        php_version: "8.3"
        php_fpm_enabled: true
        php_cli_set_as_default_version: false

    # Install PHP 8.4 (default)
    - ansible.builtin.include_role:
        name: jlira.web_server.php
      vars:
        php_version: "8.4"
        php_fpm_enabled: true
        php_cli_set_as_default_version: true

    # Configure Apache with default PHP version
    - ansible.builtin.include_role:
        name: jlira.web_server.apache
      vars:
        apache_php_fpm_integration: true
        apache_php_fpm_version: "8.4"
        apache_php_fpm_set_default: true
```

### 5. Development Environment

```yaml
---
- name: Setup development web server
  hosts: dev_servers
  become: true
  roles:
    - role: jlira.web_server.apache
      vars:
        apache_server_name: "dev.local"
        apache_ssl_enabled: false
        apache_php_fpm_integration: true
        apache_status_page:
          enabled: true
          endpoint: /server-status
          allowed_hosts:
            - 127.0.0.1
            - 10.0.0.0/8

    - role: jlira.web_server.php
      vars:
        php_version: "8.4"
        php_fpm_enabled: true
        php_composer_enabled: true
        php_fpm_settings:
          display_errors:
            value: true
            type: bool
          error_reporting:
            value: E_ALL
            type: string
        php_status_page:
          enabled: true
```

### 6. Production Hardened Setup

```yaml
---
- name: Production web server with security hardening
  hosts: production
  become: true
  roles:
    - role: jlira.web_server.apache
      vars:
        apache_server_name: "prod.example.com"
        apache_ssl_enabled: true
        apache_security_settings:
          - name: ServerTokens
            value: Prod
          - name: ServerSignature
            value: "Off"
          - name: TraceEnable
            value: "Off"
        apache_custom_configs:
          - name: Security headers
            path: /etc/apache2/conf-available/security-headers.conf
            content: |
              Header always set X-Frame-Options "SAMEORIGIN"
              Header always set X-Content-Type-Options "nosniff"
              Header always set X-XSS-Protection "1; mode=block"
              Header always set Referrer-Policy "strict-origin-when-cross-origin"

    - role: jlira.web_server.php
      vars:
        php_version: "8.4"
        php_fpm_enabled: true
        php_fpm_settings:
          display_errors:
            value: false
            type: bool
          expose_php:
            value: false
            type: bool
```

### 7. Automated Let's Encrypt Certificates

```yaml
---
- name: Web server with automated Let's Encrypt certificates
  hosts: webservers
  become: true
  roles:
    # First, generate Let's Encrypt certificates
    - role: jlira.web_server.certificates
      vars:
        certificates_certbot_email: admin@example.com
        certificates_items:
          - name: example.com
            mode: letsencrypt
            challenge_type: http-01
            webroot: /var/www/html
            san:
              - www.example.com

    # Then configure Apache with the certificates
    - role: jlira.web_server.apache
      vars:
        apache_server_name: example.com
        apache_ssl_enabled: true
        apache_virtual_hosts:
          - name: example.com
            server_name: example.com
            document_root: /var/www/example
            http:
              custom_directives: |
                Redirect permanent / https://example.com/
            https:
              certificate_file: /etc/letsencrypt/live/example.com/fullchain.pem
              certificate_key_file: /etc/letsencrypt/live/example.com/privkey.pem
              custom_directives: |
                <Directory /var/www/example>
                    Options -Indexes +FollowSymLinks
                    AllowOverride All
                    Require all granted
                </Directory>
```

## Testing

The collection uses [Molecule](https://molecule.readthedocs.io/) for role testing.

### Run All Tests

```bash
# From collection root
cd extensions

# Test Apache Setup Playbook (Orchestrated)
molecule test -s apache-setup-playbook

# Test Apache role
molecule test -s apache

# Test PHP role
molecule test -s php

# Test Certificates role
molecule test -s certificates
```

### Development Testing

```bash
# From collection root
cd extensions

# Apache role
molecule create -s apache
molecule converge -s apache
molecule verify -s apache
molecule destroy -s apache

# PHP role (includes upgrade tests)
molecule create -s php
molecule converge -s php
molecule side-effect -s php  # Test upgrade scenario
molecule verify -s php
molecule destroy -s php

# Quick iteration
molecule converge -s apache && molecule verify -s apache
```

### CI/CD Integration

The collection includes GitHub Actions workflows for automated testing:

- **Linting**: `yamllint` and `ansible-lint`
- **Syntax Check**: Ansible syntax validation
- **Molecule Tests**: Full role testing in containers

## Releasing

This collection supports automated releases to Ansible Galaxy. See [.github/RELEASE.md](.github/RELEASE.md) for detailed release documentation.

### Quick Release Guide

**Semi-Automated Release (Recommended for planned releases):**
```bash
# 1. Update CHANGELOG.md with your changes
# 2. Commit the changelog
git add CHANGELOG.md
git commit -m "docs: update changelog for v0.0.2"

# 3. Create and push tag
git tag v0.0.2
git push origin v0.0.2
```

**Fully Automated Release (Conventional commits):**
```bash
# Use conventional commit messages
git commit -m "feat: add new Apache virtual host template"
git push origin main
# Release happens automatically!
```

See [Release Documentation](.github/RELEASE.md) for full details on both methods.

## Contributing

Contributions are welcome! Please follow these guidelines:

1. **Fork the Repository**: Create your own fork on GitHub
2. **Create a Feature Branch**: `git checkout -b feature/your-feature`
3. **Follow Ansible Best Practices**: See [CLAUDE.md](CLAUDE.md) for project conventions
4. **Use Conventional Commits**: For automated releases (optional but recommended)
5. **Write Tests**: Add Molecule tests for new features
6. **Run Linters**: Ensure `yamllint` and `ansible-lint` pass
7. **Update Documentation**: Document new features in role READMEs
8. **Submit Pull Request**: Provide a clear description of changes

### Development Workflow

```bash
# Clone repository
git clone https://github.com/yurgenlira/jlira.web_server.git
cd jlira.web_server

# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install test dependencies
pip install -r extensions/requirements-test.txt

# Install lint dependencies
pip install -r extensions/requirements-lint.txt

# Make changes and test
cd extensions
molecule test -s apache

# Run linters
ansible-lint
```

## License

This collection is licensed under the [GPL-2.0-or-later](https://spdx.org/licenses/GPL-2.0-or-later.html) license.

## Author

**Julio Lira**
- Email: yurgenlira@hotmail.com
- GitHub: [@yurgenlira](https://github.com/yurgenlira)

## Support

- **Issues**: [GitHub Issues](https://github.com/yurgenlira/jlira.web_server/issues)
- **Documentation**: [Collection Docs](https://github.com/yurgenlira/jlira.web_server)
- **Discussions**: [GitHub Discussions](https://github.com/yurgenlira/jlira.web_server/discussions)

---

**Project Links:**
- [Apache Role Documentation](roles/apache/README.md)
- [PHP Role Documentation](roles/php/README.md)
- [Certificates Role Documentation](roles/certificates/README.md)
- [Playbook Examples](playbooks/README.md)
- [Development Guide](CLAUDE.md)
