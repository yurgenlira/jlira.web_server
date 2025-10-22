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
- **Integration**: Seamless Apache-PHP integration with automatic PHP-FPM configuration
- **Ready-to-Use Playbooks**: Pre-built playbooks for common deployment scenarios
- **Production-Ready**: Security-hardened defaults, comprehensive validation, and idempotent operations

## Features

### Apache Role Features

- ✅ **Installation & Service Management**: Automated Apache installation and service lifecycle management
- ✅ **SSL/TLS Support**: Built-in SSL configuration with support for custom certificates
- ✅ **Virtual Host Management**:
  - Direct virtual host configuration
  - Template-based virtual hosts for reusability
  - HTTP and HTTPS support
  - Custom directives and includes
- ✅ **PHP-FPM Integration**:
  - Automatic PHP-FPM configuration
  - Multiple PHP version support
  - Version-specific configuration
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

## Requirements

- **Ansible**: >= 2.18.7
- **Target OS**: Debian 11+ or Ubuntu 20.04+
- **Privileges**: Root or sudo access (`become: true`)
- **Python**: Python 3.6+ on target hosts
- **Internet Access**: Required for package installation (or configured local mirrors)

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy collection install jlira.web_server
```

### From Source

```bash
git clone https://github.com/yurgenlira/jlira.web_server.git
cd jlira.web_server
ansible-galaxy collection build
ansible-galaxy collection install jlira-web_server-*.tar.gz
```

### Using requirements.yml

Create a `requirements.yml` file:

```yaml
---
collections:
  - name: jlira.web_server
    version: ">=0.0.1"
```

Install the collection:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Quick Start

### Basic Web Server Setup

Create a playbook (`webserver.yml`):

```yaml
---
- name: Setup web server
  hosts: webservers
  become: true

  roles:
    - role: jlira.web_server.apache
      vars:
        apache_server_name: "example.com"
        apache_ssl_enabled: true
        apache_modules_enabled:
          - rewrite
          - headers

    - role: jlira.web_server.php
      vars:
        php_version: "8.4"
        php_fpm_enabled: true
        php_composer_enabled: true
```

Run the playbook:

```bash
ansible-playbook -i inventory webserver.yml
```

### Using Pre-built Playbooks

The collection includes ready-to-use playbooks:

```bash
# Basic web server (Apache + PHP)
ansible-playbook -i inventory jlira.web_server.playbooks.web-server-setup

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
| `apache_http_ports` | list | `[80]` | HTTP ports |
| `apache_https_ports` | list | `[443]` | HTTPS ports |
| `apache_php_fpm_integration` | boolean | `false` | Enable PHP-FPM |
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

## Playbooks

The collection includes several pre-built playbooks for common scenarios:

### Web Server Setup Playbooks

| Playbook | Description | Use Case |
|----------|-------------|----------|
| `web-server-setup` | Basic Apache + PHP setup | Single PHP version web server |
| `web-server-multi-php` | Apache + Multiple PHP versions | Multi-version PHP environment |
| `php-install` | PHP installation only | Add PHP to existing server |
| `php-upgrade` | PHP version upgrade | Migrate between PHP versions |
| `php-uninstall` | PHP removal | Clean up PHP installation |

**Documentation:** [Playbooks README](playbooks/README.md)

## Common Use Cases

### 1. Basic LAMP Stack

```yaml
---
- name: Deploy LAMP stack
  hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.apache
      vars:
        apache_server_name: "lamp.example.com"
        apache_ssl_enabled: true
        apache_php_fpm_integration: true

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
```

### 2. Virtual Host with SSL

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
              enabled: true
              custom_directives: |
                Redirect permanent / https://myapp.com/
            https:
              enabled: true
              certificate_file: /etc/ssl/certs/myapp.crt
              certificate_key_file: /etc/ssl/private/myapp.key
```

### 3. Multiple PHP Versions

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

    # Configure Apache with PHP-FPM
    - ansible.builtin.include_role:
        name: jlira.web_server.apache
      vars:
        apache_php_fpm_integration: true
        apache_php_fpm_version: "8.4"
```

### 4. Development Environment

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

### 5. Production Hardened Setup

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

## Testing

The collection uses [Molecule](https://molecule.readthedocs.io/) for role testing.

### Run All Tests

```bash
# From collection root
cd extensions

# Test Apache role
molecule test -s apache

# Test PHP role
molecule test -s php
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

## Contributing

Contributions are welcome! Please follow these guidelines:

1. **Fork the Repository**: Create your own fork on GitHub
2. **Create a Feature Branch**: `git checkout -b feature/your-feature`
3. **Follow Ansible Best Practices**: See [CLAUDE.md](CLAUDE.md) for project conventions
4. **Write Tests**: Add Molecule tests for new features
5. **Run Linters**: Ensure `yamllint` and `ansible-lint` pass
6. **Update Documentation**: Document new features in role READMEs
7. **Submit Pull Request**: Provide a clear description of changes

### Development Workflow

```bash
# Clone repository
git clone https://github.com/yurgenlira/jlira.web_server.git
cd jlira.web_server

# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Make changes and test
molecule test -s apache

# Run linters
yamllint .
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
- [Playbook Examples](playbooks/README.md)
- [Development Guide](CLAUDE.md)