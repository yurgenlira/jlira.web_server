# jlira.web_server.apache

A comprehensive Ansible role for installing, configuring, and managing the Apache HTTP Server on Debian/Ubuntu systems.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Role Variables](#role-variables)
  - [Basic Configuration](#basic-configuration)
  - [PHP-FPM Integration](#php-fpm-integration)
  - [Port Configuration](#port-configuration)
  - [Module Management](#module-management)
  - [Security Configuration](#security-configuration)
  - [Virtual Hosts](#virtual-hosts)
  - [Virtual Host Templates](#virtual-host-templates)
  - [Status Pages](#status-pages)
  - [HTTP Authentication](#http-authentication)
  - [Custom Configuration Files](#custom-configuration-files)
- [Dependencies](#dependencies)
- [Example Playbooks](#example-playbooks)
- [Tags](#tags)
- [Handlers](#handlers)
- [Testing](#testing)
- [License](#license)
- [Author](#author)

## Overview

This role provides a complete solution for managing Apache HTTP Server, including:

- Installation and service management
- SSL/TLS configuration
- PHP-FPM integration with support for multiple PHP versions
- Virtual host management (direct and template-based)
- Apache and PHP-FPM status pages
- HTTP Basic Authentication
- Security hardening
- Custom configuration management
- Comprehensive validation

## Features

- **Flexible Installation**: Installs Apache with configurable modules and settings
- **SSL/TLS Support**: Built-in SSL configuration with customizable certificates
- **PHP-FPM Integration**: Seamless integration with PHP-FPM, supporting multiple PHP versions
- **Virtual Host Management**: Create and manage virtual hosts directly or using reusable templates
- **Status Monitoring**: Apache and PHP-FPM status pages with access control
- **Security Hardening**: Secure defaults including ServerTokens, ServerSignature, and TraceEnable
- **HTTP Authentication**: Manage htpasswd files and users
- **Idempotent**: All tasks are idempotent and support check mode
- **Validation**: Comprehensive variable validation and configuration syntax checking
- **Tag Support**: Fine-grained control using Ansible tags

## Requirements

- **Target Systems**: Debian 11+ or Ubuntu 20.04+
- **Ansible**: 2.9+
- **Privileges**: Tasks require `become: true` (root/sudo access)
- **Dependencies**: None (PHP role is automatically included when PHP-FPM integration is enabled)

## Role Variables

All variables are defined in `defaults/main.yml` with comprehensive documentation and examples.

### Basic Configuration

#### `apache_server_name`
- **Type**: String
- **Default**: `{{ ansible_hostname }}`
- **Description**: Server hostname for Apache ServerName directive

```yaml
apache_server_name: "example.com"
```

#### `apache_ssl_enabled`
- **Type**: Boolean
- **Default**: `true`
- **Description**: Enable or disable SSL support

```yaml
apache_ssl_enabled: true
```

### SSL Certificate Configuration

#### `apache_default_ssl_certificate_file`
- **Type**: String
- **Default**: `/etc/ssl/certs/ssl-cert-snakeoil.pem`
- **Description**: Path to the SSL certificate file for the default SSL virtual host

```yaml
apache_default_ssl_certificate_file: /etc/ssl/certs/example.com.crt
```

#### `apache_default_ssl_certificate_key_file`
- **Type**: String
- **Default**: `/etc/ssl/private/ssl-cert-snakeoil.key`
- **Description**: Path to the SSL certificate key file for the default SSL virtual host

```yaml
apache_default_ssl_certificate_key_file: /etc/ssl/private/example.com.key
```

#### `apache_ssl_certificate_fallback`
- **Type**: String
- **Default**: `fail`
- **Options**: `fail`, `disable`, `generate`
- **Description**: Behavior when SSL is enabled but certificates don't exist

**Options:**
- `fail`: Fail the playbook with clear error message (recommended for production)
- `disable`: Skip SSL configuration, run HTTP-only (fallback mode)
- `generate`: Auto-generate self-signed certificates (good for development)

```yaml
# For production - fail if certificates missing
apache_ssl_certificate_fallback: fail

# For development - auto-generate self-signed
apache_ssl_certificate_fallback: generate

# For graceful degradation - disable SSL if certificates missing
apache_ssl_certificate_fallback: disable
```

#### `apache_ssl_selfsigned_organization`
- **Type**: String
- **Default**: `{{ inventory_hostname }}`
- **Description**: Organization name for auto-generated self-signed certificates (only used when `apache_ssl_certificate_fallback` is `generate`)

```yaml
apache_ssl_selfsigned_organization: "My Company"
```

#### `apache_ssl_selfsigned_days`
- **Type**: Integer
- **Default**: `365`
- **Description**: Number of days the auto-generated self-signed certificate is valid

```yaml
apache_ssl_selfsigned_days: 730  # 2 years
```

#### `apache_main_settings`
- **Type**: List of Dictionaries
- **Default**: `[]`
- **Description**: Custom Apache directives for apache2.conf

```yaml
apache_main_settings:
  - name: LimitRequestLine
    value: 8190
    comment: |
      Maximum size of the request line in bytes.
      Helps prevent buffer overflow attacks.
  - name: LimitRequestFieldSize
    value: 8190
```

#### `apache_envvars`
- **Type**: List of Dictionaries
- **Default**: `[]`
- **Description**: Custom environment variables for Apache

```yaml
apache_envvars:
  - name: APACHE_CUSTOM_LOG_DIR
    value: /var/log/apache2/custom
    comment: Custom log directory
```

### PHP-FPM Integration

#### `apache_php_fpm_integration`
- **Type**: Boolean
- **Default**: `false`
- **Description**: Enable PHP-FPM integration. Automatically includes the PHP role.

```yaml
apache_php_fpm_integration: true
```

#### `apache_php_fpm_version`
- **Type**: String
- **Default**: `{{ php_version | default('8.4') }}`
- **Description**: PHP version for PHP-FPM integration

```yaml
apache_php_fpm_version: "8.4"
```

#### `apache_php_fpm_set_default`
- **Type**: Boolean
- **Default**: `true`
- **Description**: Set this PHP version as the default handler for PHP files

```yaml
apache_php_fpm_set_default: true
```

#### `apache_php_fpm_proxy_timeout`
- **Type**: Integer
- **Default**: `300`
- **Description**: Maximum time in seconds Apache waits for PHP-FPM to respond

```yaml
apache_php_fpm_proxy_timeout: 600
```

#### `apache_php_status_page`
- **Type**: Dictionary
- **Default**: See defaults/main.yml
- **Description**: Configuration for PHP-FPM status page

```yaml
apache_php_status_page:
  enabled: true
  status_path: /fpm-status
  ping_path: /fpm-ping
  ping_response: pong
  realtime_endpoint: /fpm-realtime
  allowed_hosts:
    - 127.0.0.1
    - 10.0.0.0/24
```

### Port Configuration

#### `apache_http_ports`
- **Type**: List of Integers
- **Default**: `[80]`
- **Description**: HTTP ports to listen on

```yaml
apache_http_ports:
  - 80
  - 8080
```

#### `apache_https_ports`
- **Type**: List of Integers
- **Default**: `[443]`
- **Description**: HTTPS ports to listen on (when SSL is enabled)

```yaml
apache_https_ports:
  - 443
  - 8443
```

### Module Management

#### `apache_modules_enabled`
- **Type**: List of Strings
- **Default**: `[]`
- **Description**: Apache modules to enable

```yaml
apache_modules_enabled:
  - rewrite
  - headers
  - http2
  - expires
```

### Security Configuration

#### `apache_security_settings`
- **Type**: List of Dictionaries
- **Default**: Secure defaults (ServerTokens: Prod, ServerSignature: Off, TraceEnable: Off)
- **Description**: Security directives for security.conf

```yaml
apache_security_settings:
  - name: ServerTokens
    value: Prod
  - name: ServerSignature
    value: "Off"
  - name: TraceEnable
    value: "Off"
```

### Virtual Hosts

#### `apache_virtual_hosts`
- **Type**: List of Dictionaries
- **Default**: `[]`
- **Description**: Virtual hosts to configure

**Simple HTTP virtual host:**
```yaml
apache_virtual_hosts:
  - name: example.com
    server_name: example.com
    server_alias:
      - www.example.com
    server_admin: admin@example.com
    document_root: /var/www/example.com
    http:
      enabled: true
      custom_directives: |
        DirectoryIndex index.html index.php
```

**HTTP and HTTPS virtual host:**
```yaml
apache_virtual_hosts:
  - name: secure.example.com
    server_name: secure.example.com
    document_root: /var/www/secure.example.com
    http:
      enabled: true
      custom_directives: |
        Redirect permanent / https://secure.example.com/
    https:
      enabled: true
      certificate_file: /etc/ssl/certs/example.com.crt
      certificate_key_file: /etc/ssl/private/example.com.key
      certificate_chain_file: /etc/ssl/certs/example.com-chain.crt
      custom_directives: |
        <Directory /var/www/secure.example.com>
            Options -Indexes +FollowSymLinks
            AllowOverride All
            Require all granted
        </Directory>
```

### Virtual Host Templates

#### `apache_vhost_templates_dir`
- **Type**: String
- **Default**: `/etc/apache2/templates`
- **Description**: Directory for virtual host templates

#### `apache_vhost_templates`
- **Type**: List of Dictionaries
- **Default**: `[]`
- **Description**: Reusable virtual host templates

```yaml
apache_vhost_templates:
  - name: wordpress_site
    http:
      content: |
        <VirtualHost *:80>
            ServerName {{ server_name }}
            DocumentRoot {{ document_root }}
            <Directory {{ document_root }}>
                AllowOverride All
                Require all granted
            </Directory>
        </VirtualHost>
```

#### `apache_vhosts_from_template`
- **Type**: List of Dictionaries
- **Default**: `[]`
- **Description**: Virtual hosts to create from templates

```yaml
apache_vhosts_from_template:
  - name: myblog.com
    template: wordpress_site
    enabled: true
    vars:
      server_name: myblog.com
      document_root: /var/www/myblog
```

### Status Pages

#### `apache_status_page`
- **Type**: Dictionary
- **Default**: See defaults/main.yml
- **Description**: Apache server status page configuration

```yaml
apache_status_page:
  enabled: true
  endpoint: /server-status
  allowed_hosts:
    - 127.0.0.1
    - 10.0.0.0/24
    - monitoring.example.com
```

### HTTP Authentication

#### `apache_htpasswd_files`
- **Type**: List of Dictionaries
- **Default**: `[]`
- **Description**: htpasswd files to create and manage

```yaml
apache_htpasswd_files:
  - path: /etc/apache2/.htpasswd
    users:
      - name: admin
        password: secure_password
      - name: user1
        password: another_password
  - path: /etc/apache2/.htpasswd-api
    users:
      - name: api_user
        password: api_secret
```

### Custom Configuration Files

#### `apache_custom_configs`
- **Type**: List of Dictionaries
- **Default**: `[]`
- **Description**: Custom Apache configuration files

```yaml
apache_custom_configs:
  - name: Custom security headers
    path: /etc/apache2/conf-available/security-headers.conf
    content: |
      Header always set X-Frame-Options "SAMEORIGIN"
      Header always set X-Content-Type-Options "nosniff"
      Header always set X-XSS-Protection "1; mode=block"
```

## Dependencies

- **jlira.web_server.php**: Automatically included when `apache_php_fpm_integration: true`

## Example Playbooks

### Basic Apache Installation

```yaml
---
- name: Install Apache
  hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.apache
```

### Apache with PHP-FPM

```yaml
---
- name: Install Apache with PHP-FPM
  hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.apache
      vars:
        apache_php_fpm_integration: true
        apache_php_fpm_version: "8.4"
```

### Apache with Custom SSL Certificates

```yaml
---
- name: Apache with custom SSL certificates
  hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.apache
      vars:
        apache_ssl_enabled: true
        apache_default_ssl_certificate_file: /etc/ssl/certs/mysite.crt
        apache_default_ssl_certificate_key_file: /etc/ssl/private/mysite.key
        apache_ssl_certificate_fallback: fail  # Fail if certs missing
```

### Apache with Auto-Generated Self-Signed Certificate (Development)

```yaml
---
- name: Apache with auto-generated self-signed certificate
  hosts: devservers
  become: true
  roles:
    - role: jlira.web_server.apache
      vars:
        apache_ssl_enabled: true
        apache_ssl_certificate_fallback: generate  # Auto-generate if missing
        apache_ssl_selfsigned_organization: "Development Test"
        apache_ssl_selfsigned_days: 365
```

### Apache with SSL Fallback to HTTP

```yaml
---
- name: Apache with SSL fallback to HTTP
  hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.apache
      vars:
        apache_ssl_enabled: true
        apache_ssl_certificate_fallback: disable  # Skip SSL if certs missing
```

### Apache with Certificate Role Integration

```yaml
---
- name: Apache with jlira.web_server.certificates role
  hosts: webservers
  become: true
  tasks:
    - name: Generate self-signed certificates
      ansible.builtin.include_role:
        name: jlira.web_server.certificates
      vars:
        certificates_mode: selfsigned
        certificates_list:
          - name: "{{ inventory_hostname }}"
            common_name: "{{ inventory_hostname }}"
            organization: "My Organization"
            days: 365

    - name: Configure Apache with generated certificates
      ansible.builtin.include_role:
        name: jlira.web_server.apache
      vars:
        apache_ssl_enabled: true
        apache_default_ssl_certificate_file: "/etc/ssl/certs/{{ inventory_hostname }}.crt"
        apache_default_ssl_certificate_key_file: "/etc/ssl/private/{{ inventory_hostname }}.key"
```

### Complete Configuration

```yaml
---
- name: Configure Apache with virtual hosts
  hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.apache
      vars:
        apache_server_name: "webserver.example.com"
        apache_ssl_enabled: true
        apache_http_ports:
          - 80
        apache_https_ports:
          - 443
        apache_modules_enabled:
          - rewrite
          - headers
          - http2
        apache_php_fpm_integration: true
        apache_php_fpm_version: "8.4"
        apache_virtual_hosts:
          - name: example.com
            server_name: example.com
            server_alias:
              - www.example.com
            document_root: /var/www/example.com
            http:
              enabled: true
            https:
              enabled: true
              certificate_file: /etc/ssl/certs/example.com.crt
              certificate_key_file: /etc/ssl/private/example.com.key
        apache_status_page:
          enabled: true
          endpoint: /server-status
          allowed_hosts:
            - 127.0.0.1
```

## Tags

The role supports the following tags for selective execution:

- `apache_install`: Installation tasks only
- `apache_configure`: Configuration tasks only
- `apache_virtual_hosts`: Virtual host configuration only
- `apache_php_integration`: PHP-FPM integration only
- `apache_status_page`: Status page configuration only
- `apache_authentication`: HTTP authentication tasks only
- `always`: Validation tasks (always run)

**Examples:**

```bash
# Only install Apache
ansible-playbook playbook.yml --tags apache_install

# Only configure virtual hosts
ansible-playbook playbook.yml --tags apache_virtual_hosts

# Configure everything except PHP integration
ansible-playbook playbook.yml --skip-tags apache_php_integration
```

## Handlers

The role includes the following handlers:

- **Validate Apache Configuration**: Validates Apache configuration syntax
- **Reload Apache**: Gracefully reloads Apache (non-disruptive)
- **Restart Apache**: Restarts Apache service (disruptive)

## Testing

The role includes comprehensive Molecule tests:

```bash
# Run full test suite (from collection root)
cd extensions
molecule test -s apache

# Create test environment
molecule create -s apache

# Apply role
molecule converge -s apache

# Run verification tests
molecule verify -s apache

# Check idempotency
molecule idempotence -s apache

# Clean up
molecule destroy -s apache
```

### Development Testing

```bash
# Quick iteration (skip destroy)
cd extensions
molecule converge -s apache && molecule verify -s apache

# Login to test container for debugging
molecule login -s apache
```

## License

GPL-2.0-or-later

## Author

Julio Lira (yurgenlira@hotmail.com)

---

For more information and examples, see:
- [Collection README](../../README.md)
- [Playbook Examples](../../playbooks/README.md)
- [defaults/main.yml](defaults/main.yml) - Complete variable documentation