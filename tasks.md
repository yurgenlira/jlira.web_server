## ✅ 1. Install apache

- ✅ **Installation:**
  - ✅ Install packages
- ✅ **Task Organization:**
  - ✅ Support selective execution via tags

### ✅ 2.1 PHP Setup
- ✅ **Conditional PHP Installation:**
  - ✅ Support optional PHP integration (use a php role for install and setup php)
- ✅ **Task Organization:**
  - ✅ Support selective execution via tags
  - ✅ Allow granular control of PHP features

**Technical Debt:**
- ⚠️ PHP role `configure_cli.yml` requires `php_cli_ini_regex_settings` variable to be defined in defaults
- ⚠️ PHP integration tests temporarily disabled in Apache molecule tests
- ⚠️ Need to complete PHP role implementation before re-enabling integration tests

## 3. Configuration Requirements

### 3.1 Main Apache Configuration
- **ServerName Configuration:**
  - Configure server hostname
  - Support system hostname as default
  - Allow custom server names

- **Apache Main Settings:**
  - Support custom Apache directives like LimitRequestLine, LimitRequestFieldSize, etc
  - Support documentation via comments
  - Support arbitrary user-defined environment variables
  - Allow to use a custom configuration directory path
  - Allow to set custom log directory path if you want

- **SSL Configuration:**
  - Enable SSL site by default
  - Manage site enablement

- **Port Configuration:**
  - ✅ Configure multiple HTTP ports (default: 80)
  - ✅ Configure multiple HTTPS ports (default: 443, when SSL enabled)
  - ✅ Manage port configuration

- **Firewall Configuration:**
  - Detect firewall presence
  - Configure HTTP port access
  - Configure HTTPS port access (when SSL enabled)
  - Configure custom port access
  - Manage firewall state

- **Module Management:**
  - Enable additional modules
  - Activate modules idempotently

- **Default Index Page:**
  - Create default welcome page
  - Set appropriate file ownership and permissions

- **Default Virtual Host Configuration:**
  - Organize default virtual host files
  - Integrate PHP-FPM when PHP is enabled
  - Configure SSL virtual hosts
  - Support custom ports in SSL virtual hosts

- **Security Configuration:**
  - ✅ Configure server information disclosure settings
  - ✅ Configure server signature settings
  - ✅ Configure HTTP method restrictions
  - ✅ Apply security hardening
  - ✅ Activate security configurations

### ✅ 3.2 Configuration File Management
- ✅ Support creation of custom Apache configuration files
- ✅ Each configuration file includes:
  - ✅ Descriptive name
  - ✅ Relative file path
  - ✅ Configuration body content
- ✅ Create necessary directory structure
- ✅ Set appropriate file ownership and permissions
- ✅ Mark configuration as managed by automation
- ✅ Trigger validation and reload after changes

## ✅ 4. Virtual Host Requirements

### ✅ 4.1 Virtual Host Management
- ✅ **Direct Virtual Host Creation:**
  - ✅ Create virtual host configuration files from specifications
  - ✅ Support HTTP virtual hosts with:
    - ✅ Listen addresses and ports
    - ✅ Server identity (name and aliases)
    - ✅ Administrative contact
    - ✅ Document root directory
    - ✅ Logging configuration
    - ✅ Custom configuration directives
  - ✅ Support HTTPS virtual hosts with:
    - ✅ Secure listen addresses and ports
    - ✅ SSL certificate configuration
    - ✅ HTTPS-specific logging
    - ✅ HTTPS-specific configuration directives
  - ✅ Create directory structure for virtual host files
  - ✅ Set appropriate file permissions

- ✅ **Template-Based Virtual Host Creation:**
  - ✅ Support reusable virtual host templates
  - ✅ Templates can define variables for substitution
  - ✅ Create virtual hosts from templates with specific variable values
  - ✅ Support template inclusion mechanism
  - ✅ Allow multiple virtual hosts to share common configurations

- ✅ **Virtual Host Features:**
  - ✅ Support flexible listen directives
  - ✅ Support variable substitution in configurations
  - ✅ Automatic validation and reload after changes
  - ✅ Support for modular configuration via includes

## ✅ 5. Status Page Requirements

### ✅ 5.1 Apache Status Page
- ✅ Configure server status monitoring endpoint
- ✅ Support access control via allowed hosts list
- ✅ Trigger validation and reload after changes

## ✅ 6. Authentication Requirements

### ✅ 6.1 HTTP Authentication
- ✅ Create and manage password files
- ✅ Support multiple authentication files
- ✅ Support multiple users per authentication file
- ✅ Automatically create password files if they don't exist
- ✅ Update existing users or add new users idempotently

## 7. Notification and Validation Requirements

### 7.1 Configuration Validation
- ✅ Trigger configuration validation after changes
- ✅ Validate configuration syntax before applying
- ✅ Only apply changes if validation succeeds

### ✅ 7.2 Service Management
- ✅ Reload service for non-disruptive changes
- ✅ Restart service when required for disruptive changes
- ✅ Manage service via system service manager

## 8. Idempotency Requirements

### 8.1 Change Detection
- Use appropriate automation modules to detect changes
- Avoid unnecessary service disruptions
- Check current state before making changes
- Support dry-run validation mode
- Report changes accurately

### 8.2 File Management
- Mark managed configuration sections
- Preserve existing configuration outside managed blocks
- Create files conditionally when needed
- Support idempotent command execution

## 9. Dependency Requirements

### 14.1 Ansible Collections
- ansible.builtin (core modules)
- community.general (ufw, htpasswd modules)

### 14.2 System Requirements
- Systemd for service management
- UFW firewall (optional, auto-detected)
- APT package manager (Debian/Ubuntu systems)

### 14.3 Role Dependencies
- PHP role (when `apache_install_php` is true)

## 15. Security Requirements

### 15.1 Secure Defaults
- ✅ ServerTokens set to Prod (minimal information disclosure)
- ✅ ServerSignature set to Off (hide server signature)
- ✅ TraceEnable set to Off (prevent HTTP TRACE attacks)
- ✅ Proper file permissions (0644 for config, 0640 for .htpasswd)
- ✅ Proper ownership (root:root for config, root:www-data for .htpasswd)


