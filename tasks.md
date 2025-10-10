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

### ✅ 3.1 Main Apache Configuration
- ✅ **ServerName Configuration:**
  - ✅ Configure server hostname
  - ✅ Support system hostname as default
  - ✅ Allow custom server names

- ✅ **Apache Main Settings:**
  - ✅ Support custom Apache directives like LimitRequestLine, LimitRequestFieldSize, etc
  - ✅ Support documentation via comments
  - ✅ Support arbitrary user-defined environment variables
  - Allow to use a custom configuration directory path
  - Allow to set custom log directory path if you want

- ✅ **SSL Configuration:**
  - ✅ Enable SSL site by default
  - ✅ Manage site enablement

- ✅ **Port Configuration:**
  - ✅ Configure multiple HTTP ports (default: 80)
  - ✅ Configure multiple HTTPS ports (default: 443, when SSL enabled)
  - ✅ Manage port configuration

- ✅ **Firewall Configuration:**
  - ✅ Detect firewall presence
  - ✅ Configure HTTP port access
  - ✅ Configure HTTPS port access (when SSL enabled)
  - ✅ Configure custom port access
  - ✅ Manage firewall state

- ✅ **Module Management:**
  - ✅ Enable additional modules
  - ✅ Activate modules idempotently

- ✅ **Default Index Page:**
  - ✅ Create default welcome page
  - ✅ Set appropriate file ownership and permissions

- ✅ **Default Virtual Host Configuration:**
  - ✅ Organize default virtual host files
  - ✅ Integrate PHP-FPM when PHP is enabled
  - ✅ Configure SSL virtual hosts
  - ✅ Support custom ports in SSL virtual hosts

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

## ✅ 8. Idempotency Requirements

### ✅ 8.1 Change Detection
- ✅ Use appropriate automation modules to detect changes
- ✅ Avoid unnecessary service disruptions
- ✅ Check current state before making changes
- ✅ Support dry-run validation mode
- ✅ Report changes accurately

### ✅ 8.2 File Management
- ✅ Mark managed configuration sections
- ✅ Preserve existing configuration outside managed blocks
- ✅ Create files conditionally when needed
- ✅ Support idempotent command execution

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


## 16. PHP Role Functional Requirements

### 16.1 PHP Installation
- **Repository Management:**
  - ✅ Add third-party PHP repository for accessing specific PHP versions

- **Package Installation:**
  - ✅ Install PHP core and extensions from configurable package list
  - ✅ Support PHP-FPM installation when enabled
  - ✅ Support Composer installation when enabled
  - Install packages idempotently
  - ✅ Ensure FPM service is started and enabled
### 16.2 PHP Configuration Management
#### 16.2.1 CLI Configuration
- **Configuration File Management:**
  - ✅ Backup original php.ini before modifications
  - ✅ Update PHP settings using regex-based replacements
  - ✅ Support comprehensive configuration options:
    - ✅ Execution limits (max_execution_time, max_input_time, max_input_vars)
    - ✅ Memory management (memory_limit)
    - ✅ Error handling (error_reporting, display_errors, error_log)
    - ✅ Upload limits (post_max_size, upload_max_filesize, max_file_uploads)
    - ✅ Behavior settings (short_open_tag, mail headers)
  - ✅ Create custom log directories

#### 16.2.2 FPM Configuration
- **PHP-FPM Settings:**
  - ✅ Backup original FPM php.ini before modifications
  - ✅ Configure FPM-specific PHP settings independently from CLI
  - ✅ Support same configuration options as CLI with different defaults
  - ✅ Restart FPM service only when configuration changes

- **FPM Pool Configuration:**
  - ✅ Backup original pool configuration
  - ✅ Configure FPM pool settings:
    - ✅ Process manager type (static, dynamic, ondemand)
    - ✅ Process limits (pm_max_children, pm_start_servers, pm_min_spare_servers, pm_max_spare_servers)
  - ✅ Create pool configuration from templates

### 16.3 PHP Version Management
- **CLI Version Management:**
  - ✅ Set default PHP CLI version using system alternatives
  - ✅ Update php, phar, phar.phar, phpize, and php-config alternatives
  - ✅ Create symlinks for default version tools

- **FPM Version Management:**
  - Configure PHP-FPM as default version when enabled


- **Version Upgrade/Downgrade:**
  - Remove old PHP versions when specified
  - Stop and disable old PHP-FPM services
  - Clean up old packages, configuration files, and logs
  - Remove old extension files and directories
  - Support clean migration between versions

### 16.4 PHP Extensions
#### 16.4.1 Microsoft SQL Server Extension
- **MSSQL Driver Installation:**
  - ✅ Add Microsoft repository and GPG key
  - ✅ Install Microsoft ODBC driver with EULA acceptance
  - ✅ Install mssql-tools with command-line utilities
  - ✅ Add mssql-tools to system PATH

- **PHP MSSQL Extension Compilation:**
  - ✅ Install sqlsrv and pdo_sqlsrv extensions using PECL
  - ✅ Create module configuration files with correct priority
  - ✅ Enable extensions using phpenmod
  - ✅ Skip compilation if extensions already exist

### 16.5 Status Page Configuration
- **FPM Status Page:**
  - Configure proxy timeout settings for FPM connections
  - Add file existence checks for PHP file execution
  - Configure status page endpoint access
  - Create version-specific status page HTML files
  - Update status page URL references for version-specific endpoints

### 16.6 Log Management

#### 16.6.1 Log Rotation
- **CLI Log Rotation:**
  - Configure logrotate for PHP CLI error logs
  - Support custom log paths
  - Create logrotate configuration from templates

- **FPM Log Rotation:**
  - Configure logrotate for PHP-FPM logs
  - Support custom FPM log paths
  - Create version-specific logrotate configuration



### 16.10 Variables and Configuration
### 16.11 Testing Requirements
- **Molecule Tests:**
  - Support create, converge, verify, and destroy lifecycle

### 16.12 Integration Requirements
- **Web Server Integration:**
  - Integrate with web servers via environment variables
  - Support conditional web server restarts
  - Configure FPM status page endpoint access
  - Add file existence checks for secure execution

- **System Integration:**
  - Gather system facts for version-specific decisions
  - Manage system alternatives for version switching
  - Integrate with system service manager (systemd)
  - Configure system PATH for command-line tools



- **Tag Support:**
  - Support role-wide tags (php_*)
  - Support task-specific tags for granular execution
  - Allow selective execution of specific functionality
  - Support conditional task execution based on variables

### 16.9 Idempotency Requirements
- **Change Detection:**
  - Check if files exist before creating backups
  - Use 'creates' parameter for idempotent installations
  - Register and check task results before triggering handlers
  - Only restart services when configuration actually changes

- **Service State Management:**
  - Conditional service restarts/reloads based on registered changes


### 16.12 playbooks
- Create playbook for intstall multiple php versions (use role serveral times)

- Create test for install_mssql_extension.yml

## 10. Apache PHP-FPM Integration

### 10.1 PHP-FPM Integration
- **Apache Configuration:**
  - Create a new task `php_integration.yml` in the `apache` role.
  - Add logic to enable the `proxy_fcgi` Apache module.
  - Add logic to execute `a2enconf php{{ apache_php_fpm_version }}-fpm`.
- **Variables:**
  - Add `apache_php_fpm_integration: false` to `apache` role defaults.
  - Add `apache_php_fpm_version` to `apache` role defaults.
- **Role Integration:**
  - Update `apache/tasks/main.yml` to include `php_integration.yml` conditionally.
  - Support multiple PHP versions on same system if you don't set a default version (mejor agregar el arcgivo .conf por defecto)
