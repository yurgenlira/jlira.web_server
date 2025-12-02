# jlira.web_server.apache

A comprehensive Ansible role for installing, configuring, and managing the Apache HTTP Server on Debian/Ubuntu systems.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Role Variables](#role-variables)
  - [Basic Configuration](#basic-configuration)
  - [Execution Phase Configuration](#execution-phase-configuration)
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

### Execution Phase Configuration

#### `apache_execution_phase`
- **Type**: String
- **Default**: `all`
- **Options**: `all`, `http_only`, `https_only`
- **Description**: Control which parts of the role are executed. Useful for split execution workflows (e.g., configure HTTP → generate certificates → configure HTTPS).

```yaml
# Execute everything (default)
apache_execution_phase: "all"

# Configure only HTTP virtual hosts and basic settings
apache_execution_phase: "http_only"

# Configure only HTTPS virtual hosts (requires HTTP to be configured first)
apache_execution_phase: "https_only"
```

**Use Case Example:**
```yaml
# Step 1: Configure HTTP and request certificates
- hosts: webservers
  roles:
    - role: jlira.web_server.apache
      vars:
        apache_execution_phase: "http_only"

# Step 2: Generate Let's Encrypt certificates (requires HTTP for verification)
- hosts: webservers
  roles:
    - role: jlira.web_server.certificates

# Step 3: Configure HTTPS with the generated certificates
- hosts: webservers
  roles:
    - role: jlira.web_server.apache
      vars:
        apache_execution_phase: "https_only"
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
- **Description**: Enable PHP-FPM integration with Apache. When enabled, Apache will be configured to proxy PHP requests to PHP-FPM.

**Important:** You must include the PHP role before the Apache role in your playbook.

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
      custom_directives: |
        Redirect permanent / https://secure.example.com/
    https:
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
- **Description**: Reusable virtual host templates with structured configuration

**Note:** Templates use the same structure as `apache_virtual_hosts` but with `${VARIABLE}` syntax for variable substitution.

```yaml
apache_vhost_templates:
  - name: wordpress_site
    server_name: ${SERVER_NAME}
    document_root: ${DOCUMENT_ROOT}
    http:
      custom_directives: |
        <Directory ${DOCUMENT_ROOT}>
            AllowOverride All
            Require all granted
        </Directory>
    https:
      certificate_file: /etc/ssl/certs/${SERVER_NAME}.crt
      certificate_key_file: /etc/ssl/private/${SERVER_NAME}.key
      custom_directives: |
        <Directory ${DOCUMENT_ROOT}>
            Options -Indexes +FollowSymLinks
            AllowOverride All
            Require all granted
        </Directory>
```

#### `apache_vhosts_from_template`
- **Type**: List of Dictionaries
- **Default**: `[]`
- **Description**: Virtual hosts to create from templates

```yaml
apache_vhosts_from_template:
  - name: myblog.com
    template: wordpress_site
    vars:
      - name: SERVER_NAME
        value: myblog.com
      - name: DOCUMENT_ROOT
        value: /var/www/myblog
```

**Variable Substitution:**
- Variables defined in `vars` are exported as shell environment variables during virtual host creation
- Template variables use `${VARIABLE}` syntax and are substituted using `envsubst`
- Variables can reference other environment variables (e.g., `${DEFAULT_DOMAIN}` can reference an Apache environment variable)

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

- **jlira.web_server.php**: Must be explicitly included in your playbook before the Apache role when `apache_php_fpm_integration: true`

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

**Note:** You must explicitly include the PHP role before the Apache role.

```yaml
---
- name: Install Apache with PHP-FPM
  hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.php
      vars:
        php_version: "8.4"
        php_fpm_enabled: true
    - role: jlira.web_server.apache
      vars:
        apache_php_fpm_integration: true
        apache_php_fpm_version: "8.4"
```

### Complete Configuration with PHP-FPM

```yaml
---
- name: Configure Apache with virtual hosts and PHP-FPM
  hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.php
      vars:
        php_version: "8.4"
        php_fpm_enabled: true
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
              custom_directives: |
                DirectoryIndex index.html index.php
            https:
              certificate_file: /etc/ssl/certs/example.com.crt
              certificate_key_file: /etc/ssl/private/example.com.key
        apache_status_page:
          enabled: true
          endpoint: /server-status
          allowed_hosts:
            - 127.0.0.1
```

### Split Execution with Certificate Generation

```yaml
---
- name: Configure Apache with Let's Encrypt certificates
  hosts: webservers
  become: true
  tasks:
    # Step 1: Configure Apache HTTP-only
    - name: Configure Apache HTTP
      ansible.builtin.include_role:
        name: jlira.web_server.apache
      vars:
        apache_execution_phase: "http_only"
        apache_virtual_hosts:
          - name: example.com
            server_name: example.com
            document_root: /var/www/example.com
            http: {}
            https:
              certificate_file: /etc/letsencrypt/live/example.com/fullchain.pem
              certificate_key_file: /etc/letsencrypt/live/example.com/privkey.pem

    # Step 2: Generate certificates (requires HTTP for ACME validation)
    - name: Generate Let's Encrypt certificates
      ansible.builtin.include_role:
        name: jlira.web_server.certificates
      vars:
        certificates:
          - domain: example.com
            method: acme

    # Step 3: Configure Apache HTTPS with generated certificates
    - name: Configure Apache HTTPS
      ansible.builtin.include_role:
        name: jlira.web_server.apache
      vars:
        apache_execution_phase: "https_only"
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
