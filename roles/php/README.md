# jlira.web_server.php

A comprehensive Ansible role for installing, configuring, and managing PHP on Debian/Ubuntu systems, with support for multiple PHP versions and PHP-FPM.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Role Variables](#role-variables)
  - [Version Configuration](#version-configuration)
  - [Package Installation](#package-installation)
  - [CLI Configuration](#cli-configuration)
  - [PHP-FPM Configuration](#php-fpm-configuration)
  - [Status Page Configuration](#status-page-configuration)
  - [Log Rotation](#log-rotation)
  - [Microsoft SQL Server Extension](#microsoft-sql-server-extension)
  - [Version Management](#version-management)
- [Dependencies](#dependencies)
- [Example Playbooks](#example-playbooks)
- [Tags](#tags)
- [Handlers](#handlers)
- [Testing](#testing)
- [License](#license)
- [Author](#author)

## Overview

This role provides a complete solution for managing PHP installations, including:

- Installation from Ondrej's PHP PPA repository
- Support for multiple PHP versions on the same system
- PHP CLI and PHP-FPM configuration
- Version management using Debian alternatives
- PHP-FPM pool configuration
- PHP-FPM status page
- Log rotation for PHP logs
- Composer installation
- Microsoft SQL Server PHP extension support
- Upgrade/downgrade capabilities

## Features

- **Flexible Installation**: Install any PHP version from Ondrej's PPA
- **Multi-Version Support**: Run multiple PHP versions simultaneously
- **Comprehensive Configuration**: Fine-tune php.ini for CLI and FPM independently
- **PHP-FPM Integration**: Complete FPM pool management
- **Status Monitoring**: PHP-FPM status and ping endpoints
- **Version Management**: Set default PHP CLI version using alternatives
- **Composer Support**: Optional Composer installation
- **MSSQL Extension**: Optional Microsoft SQL Server driver installation
- **Log Management**: Configurable log rotation
- **Upgrade Support**: Clean migration between PHP versions
- **Idempotent**: All tasks are idempotent and support check mode
- **Validation**: Comprehensive variable validation
- **Tag Support**: Fine-grained control using Ansible tags

## Requirements

- **Target Systems**: Debian 11+ or Ubuntu 20.04+
- **Ansible**: 2.9+
- **Privileges**: Tasks require `become: true` (root/sudo access)
- **Dependencies**: None

## Role Variables

All variables are defined in `defaults/main.yml` with comprehensive documentation and examples.

### Version Configuration

#### `php_version`
- **Type**: String
- **Default**: `"8.4"`
- **Description**: PHP version to install and configure

```yaml
php_version: "8.4"
```

#### `php_cli_set_as_default_version`
- **Type**: Boolean
- **Default**: `true`
- **Description**: Set this PHP version as the default CLI version using alternatives

```yaml
php_cli_set_as_default_version: true
```

#### `php_old_version`
- **Type**: String
- **Default**: `""`
- **Description**: Previous PHP version to remove during upgrade/migration

```yaml
php_old_version: "8.3"
```

### Package Installation

#### `php_packages`
- **Type**: List of Strings
- **Default**: `["php{{ php_version }}-cli"]`
- **Description**: PHP packages to install

```yaml
php_packages:
  - "php{{ php_version }}-cli"
  - "php{{ php_version }}-fpm"
  - "php{{ php_version }}-mysql"
  - "php{{ php_version }}-mbstring"
  - "php{{ php_version }}-xml"
  - "php{{ php_version }}-curl"
  - "php{{ php_version }}-gd"
  - "php{{ php_version }}-zip"
  - "php{{ php_version }}-intl"
```

#### `php_composer_enabled`
- **Type**: Boolean
- **Default**: `false`
- **Description**: Install Composer globally

```yaml
php_composer_enabled: true
```

### CLI Configuration

#### `php_cli_settings`
- **Type**: Dictionary
- **Default**: See defaults/main.yml
- **Description**: php.ini settings for PHP CLI

```yaml
php_cli_settings:
  short_open_tag:
    value: false
    type: bool
  max_input_time:
    value: 60
    type: int
  max_input_vars:
    value: 1000
    type: int
  memory_limit:
    value: "-1"
    type: string
  error_reporting:
    value: E_ALL & ~E_DEPRECATED
    type: string
  display_errors:
    value: false
    type: bool
  error_log:
    value: /var/log/php/php_errors.log
    type: path
  post_max_size:
    value: 8M
    type: size
  upload_max_filesize:
    value: 2M
    type: size
  max_file_uploads:
    value: 20
    type: int
```

### PHP-FPM Configuration

#### `php_fpm_enabled`
- **Type**: Boolean
- **Default**: `false`
- **Description**: Install and configure PHP-FPM

```yaml
php_fpm_enabled: true
```

#### `php_fpm_settings`
- **Type**: Dictionary
- **Default**: See defaults/main.yml
- **Description**: php.ini settings for PHP-FPM

```yaml
php_fpm_settings:
  short_open_tag:
    value: false
    type: bool
  max_execution_time:
    value: 30
    type: int
  max_input_time:
    value: 60
    type: int
  memory_limit:
    value: 128M
    type: string
  error_reporting:
    value: E_ALL & ~E_DEPRECATED
    type: string
  display_errors:
    value: false
    type: bool
  post_max_size:
    value: 8M
    type: size
  upload_max_filesize:
    value: 2M
    type: size
```

#### `php_pool_settings`
- **Type**: Dictionary
- **Default**: See defaults/main.yml
- **Description**: PHP-FPM pool configuration

```yaml
php_pool_settings:
  pool_name: www
  pm: dynamic
  pm_max_children: 5
  pm_start_servers: 2
  pm_min_spare_servers: 1
  pm_max_spare_servers: 3
```

#### `php_pool_additional_settings`
- **Type**: Dictionary
- **Default**: `{}`
- **Description**: Additional FPM pool settings

```yaml
php_pool_additional_settings:
  slowlog: /var/log/php/php{{ php_version }}-fpm-slow.log
  request_slowlog_timeout: 5s
  request_terminate_timeout: 30s
  php_admin_value[memory_limit]: 256M
  catch_workers_output: yes
```

### Status Page Configuration

#### `php_status_page`
- **Type**: Dictionary
- **Default**: See defaults/main.yml
- **Description**: PHP-FPM status page configuration

```yaml
php_status_page:
  enabled: true
  status_path: /fpm-status
  ping_path: /fpm-ping
  ping_response: pong
```

### Log Rotation

#### `php_cli_logrotate_enabled`
- **Type**: Boolean
- **Default**: `false`
- **Description**: Enable log rotation for PHP CLI logs

```yaml
php_cli_logrotate_enabled: true
```

#### `php_cli_logrotate_options`
- **Type**: List of Strings
- **Default**: See defaults/main.yml
- **Description**: Logrotate options for CLI logs

```yaml
php_cli_logrotate_options:
  - su root root
  - daily
  - rotate 7
  - missingok
  - notifempty
  - compress
  - delaycompress
```

#### `php_fpm_logrotate_enabled`
- **Type**: Boolean
- **Default**: `false`
- **Description**: Enable log rotation for PHP-FPM logs

```yaml
php_fpm_logrotate_enabled: true
```

### Microsoft SQL Server Extension

#### `php_mssql_extension_enabled`
- **Type**: Boolean
- **Default**: `false`
- **Description**: Install Microsoft SQL Server PHP extension

```yaml
php_mssql_extension_enabled: true
```

#### `php_mssql_package_version`
- **Type**: Integer
- **Default**: `18`
- **Description**: MSSQL ODBC driver version

```yaml
php_mssql_package_version: 18
```

### Version Management

The role uses Debian alternatives to manage multiple PHP versions. When `php_cli_set_as_default_version: true`, the specified version becomes the default for:
- `php`
- `phar`
- `phar.phar`
- `phpize`
- `php-config`

## Dependencies

None

## Example Playbooks

### Basic PHP Installation

```yaml
---
- name: Install PHP
  hosts: php_servers
  become: true
  roles:
    - role: jlira.web_server.php
```

### PHP with FPM

```yaml
---
- name: Install PHP with FPM
  hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.php
      vars:
        php_version: "8.4"
        php_fpm_enabled: true
        php_cli_set_as_default_version: true
```

### PHP with Composer and Extensions

```yaml
---
- name: Install PHP with extensions
  hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.php
      vars:
        php_version: "8.4"
        php_fpm_enabled: true
        php_composer_enabled: true
        php_packages:
          - "php{{ php_version }}-cli"
          - "php{{ php_version }}-fpm"
          - "php{{ php_version }}-mysql"
          - "php{{ php_version }}-mbstring"
          - "php{{ php_version }}-xml"
          - "php{{ php_version }}-curl"
          - "php{{ php_version }}-gd"
          - "php{{ php_version }}-zip"
```

### Complete Configuration

```yaml
---
- name: Configure PHP with custom settings
  hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.php
      vars:
        php_version: "8.4"
        php_fpm_enabled: true
        php_composer_enabled: true
        php_cli_set_as_default_version: true
        php_packages:
          - "php{{ php_version }}-cli"
          - "php{{ php_version }}-fpm"
          - "php{{ php_version }}-mysql"
          - "php{{ php_version }}-xml"
        php_cli_settings:
          memory_limit:
            value: "-1"
            type: string
          error_log:
            value: /var/log/php/cli_errors.log
            type: path
        php_fpm_settings:
          max_execution_time:
            value: 60
            type: int
          memory_limit:
            value: 256M
            type: string
          post_max_size:
            value: 20M
            type: size
          upload_max_filesize:
            value: 10M
            type: size
        php_pool_settings:
          pool_name: www
          pm: dynamic
          pm_max_children: 10
          pm_start_servers: 4
          pm_min_spare_servers: 2
          pm_max_spare_servers: 6
        php_status_page:
          enabled: true
          status_path: /fpm-status
          ping_path: /fpm-ping
        php_cli_logrotate_enabled: true
        php_fpm_logrotate_enabled: true
```

### Upgrade PHP Version

```yaml
---
- name: Upgrade from PHP 8.3 to 8.4
  hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.php
      vars:
        php_version: "8.4"
        php_old_version: "8.3"
        php_fpm_enabled: true
        php_cli_set_as_default_version: true
```

### Multiple PHP Versions

To install multiple PHP versions, run the role multiple times with different versions:

```yaml
---
- name: Install multiple PHP versions
  hosts: webservers
  become: true
  tasks:
    - name: Install PHP 8.3
      ansible.builtin.include_role:
        name: jlira.web_server.php
      vars:
        php_version: "8.3"
        php_fpm_enabled: true
        php_cli_set_as_default_version: false

    - name: Install PHP 8.4 (default)
      ansible.builtin.include_role:
        name: jlira.web_server.php
      vars:
        php_version: "8.4"
        php_fpm_enabled: true
        php_cli_set_as_default_version: true
```

## Tags

The role supports the following tags for selective execution:

- `php_install`: Installation tasks only
- `php_configure_cli`: CLI configuration only
- `php_configure_fpm`: FPM configuration only
- `php_set_default_cli`: Set default CLI version only
- `php_logrotate_cli`: CLI log rotation configuration
- `php_logrotate_fpm`: FPM log rotation configuration
- `php_mssql_extension`: MSSQL extension installation
- `php_uninstall`: Uninstall old PHP version
- `always`: Validation tasks (always run)

**Examples:**

```bash
# Only install PHP packages
ansible-playbook playbook.yml --tags php_install

# Only configure PHP-FPM
ansible-playbook playbook.yml --tags php_configure_fpm

# Configure everything except MSSQL extension
ansible-playbook playbook.yml --skip-tags php_mssql_extension
```

## Handlers

The role includes the following handlers:

- **Restart PHP-FPM**: Restarts PHP-FPM service when configuration changes

## Testing

The role includes comprehensive Molecule tests:

```bash
# Run full test suite (from collection root)
cd extensions
molecule test -s php

# Create test environment
molecule create -s php

# Apply role
molecule converge -s php

# Run verification tests
molecule verify -s php

# Test upgrade scenario
molecule side-effect -s php

# Check idempotency
molecule idempotence -s php

# Clean up
molecule destroy -s php
```

### Development Testing

```bash
# Quick iteration (skip destroy)
cd extensions
molecule converge -s php && molecule verify -s php

# Login to test container for debugging
molecule login -s php
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

## Security Notes

- **Composer Installation**: Downloads PHP script from getcomposer.org. Ensure you trust the network and source.
- **PPA Repository**: Adding Ondrej's PPA changes package sources. Confirm this is appropriate for your security policy.
- **MSSQL Extension**: Downloads and installs Microsoft packages. Review Microsoft's license terms.
- **Passwords in Variables**: Never commit plain-text passwords. Use Ansible Vault for sensitive data.