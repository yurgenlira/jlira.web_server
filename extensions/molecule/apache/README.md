# Apache Role Molecule Testing Scenario

This Molecule scenario comprehensively tests the `jlira.web_server.apache` role, including installation, configuration, virtual hosts, PHP-FPM integration, status pages, and authentication.

## Test Flow

The scenario uses Molecule's test sequence to validate different aspects of the Apache role:

### 1. **converge.yml** - Initial Installation & Configuration
- Installs Apache web server
- Configures main Apache settings (security, limits, modules)
- Sets up custom configuration files
- Configures virtual hosts with HTTP and HTTPS
- Creates virtual hosts from templates
- Integrates with PHP-FPM
- Enables Apache status page
- Enables PHP-FPM status proxy
- Configures HTTP authentication

### 2. **idempotence** - Idempotency Check
- Re-runs converge.yml to ensure no changes occur
- Validates that the role is truly idempotent

### 3. **verify.yml** - Comprehensive Verification
Tests are organized by task file:

#### **install.yml tests**
- ✅ Apache2 package is installed
- ✅ Apache service is running and enabled
- ✅ Required Apache modules are enabled
- ✅ Apache is listening on configured ports

#### **php_integration.yml tests**
- ✅ PHP-FPM integration modules loaded (proxy_fcgi, setenvif)
- ✅ PHP-FPM configuration file exists
- ✅ PHP-FPM is set as default handler
- ✅ PHP-FPM service is running

#### **configure.yml tests**
- ✅ Main Apache configuration settings applied
- ✅ Security settings configured (ServerTokens, ServerSignature, TraceEnable)
- ✅ Environment variables set in envvars file
- ✅ HTTP/HTTPS ports configured
- ✅ Custom log directory created

#### **add_custom_configs.yml tests**
- ✅ Custom configuration files deployed
- ✅ Configuration syntax is valid
- ✅ Security headers are present
- ✅ Configurations are properly included

#### **virtual_hosts.yml tests**
- ✅ Virtual host configuration files created
- ✅ Sites are enabled and linked
- ✅ Document roots exist
- ✅ HTTP and HTTPS configurations present
- ✅ Custom directives applied
- ✅ Authentication configured

#### **virtual_hosts_from_templates.yml tests**
- ✅ Template-based virtual hosts created
- ✅ Variables substituted correctly
- ✅ HTTP and HTTPS configurations present
- ✅ Environment variables resolved

#### **status_page.yml tests**
- ✅ Apache status module enabled
- ✅ Status page accessible from allowed hosts
- ✅ Status page restricted from unauthorized hosts
- ✅ PHP-FPM status proxy configured (if enabled)
- ✅ PHP-FPM status accessible

#### **authentication.yml tests**
- ✅ htpasswd files created
- ✅ Users configured correctly
- ✅ Password authentication works
- ✅ File permissions are secure

## Running the Tests

### Run Full Test Suite
```bash
cd extensions
molecule test -s apache
```

### Run Specific Test Phases

**Syntax check only:**
```bash
molecule syntax -s apache
```

**Create and converge:**
```bash
molecule converge -s apache
```

**Run verification tests:**
```bash
molecule verify -s apache
```

**Run idempotence check:**
```bash
molecule idempotence -s apache
```

**Clean up and destroy:**
```bash
molecule destroy -s apache
```

### Development Workflow

**During role development:**
```bash
# Create instance
molecule create -s apache

# Apply changes and test
molecule converge -s apache
molecule verify -s apache

# Destroy when done
molecule destroy -s apache
```

**Quick iteration:**
```bash
# Skip destroy and re-use container
molecule converge -s apache && molecule verify -s apache
```

## Test Variables

The test uses custom variables defined in `inventory/hosts.yml`:

```yaml
# PHP-FPM Integration
apache_php_fpm_integration: true
apache_php_fpm_version: "8.3"
apache_php_fpm_set_default: true
apache_php_fpm_proxy_timeout: 300

# Main Apache Settings
apache_main_settings:
  - name: LimitRequestLine
    value: 8190
  - name: LimitRequestFieldSize
    value: 8190

# Environment Variables
apache_envvars:
  - name: CUSTOM_LOG_DIR
    value: /var/log/apache2/custom
  - name: DEFAULT_DOMAIN
    value: mydomain.local

# Ports Configuration
apache_http_ports:
  - 80
  - 8080
apache_https_ports:
  - 443
  - 8443

# Modules
apache_modules_enabled:
  - rewrite
  - headers
  - status
  - http2

# Security Settings
apache_security_settings:
  - name: ServerTokens
    value: Prod
  - name: ServerSignature
    value: "Off"

# Virtual Hosts
apache_virtual_hosts:
  - name: secure.example.local
    server_name: secure.example.local
    http:
      enabled: true
    https:
      enabled: true

# Status Page
apache_status_page:
  enabled: true
  allowed_hosts:
    - 127.0.0.1
    - 172.17.0.1

# Authentication
apache_htpasswd_files:
  - path: /etc/apache2/.htpasswd
    users:
      - name: admin
        password: adminpass123
```

## Test Coverage

### Features Tested
- ✅ Fresh Apache installation
- ✅ Module management (enable/disable)
- ✅ Main configuration settings
- ✅ Security hardening
- ✅ Environment variables
- ✅ Port configuration
- ✅ Custom configuration files
- ✅ Virtual host creation (direct)
- ✅ Virtual host creation (from templates)
- ✅ PHP-FPM integration
- ✅ Apache status page
- ✅ PHP-FPM status proxy
- ✅ HTTP Basic authentication
- ✅ htpasswd file management
- ✅ Service management (start, enable, restart)
- ✅ Idempotency
- ✅ Configuration syntax validation
- ✅ Directory creation

### Scenarios Covered
1. **Fresh Installation** - First time Apache installation
2. **Configuration Changes** - Modifying Apache settings
3. **Virtual Host Management** - Creating and managing vhosts
4. **PHP Integration** - Setting up PHP-FPM with Apache
5. **Security Configuration** - Hardening and authentication

## Platform

Tests run on Ubuntu 24.04 using Docker with systemd support.

**Container Image:** `geerlingguy/docker-ubuntu2404-ansible`

**Requirements:**
- Docker with cgroups v2 support
- Systemd-compatible container
- Privileged mode enabled (for systemd)

## Extending Tests

### Adding New Test Files

1. Create test file in `tests/` directory matching the task file name:
   ```
   roles/apache/tasks/my_feature.yml → tests/my_feature.yml
   ```

2. Add test inclusion to `verify.yml`:
   ```yaml
   - name: Include my_feature tests
     ansible.builtin.include_tasks:
       file: tests/my_feature.yml
   ```

3. Run tests:
   ```bash
   molecule verify -s apache
   ```

### Testing Custom Scenarios

Modify `inventory/hosts.yml` to test different configurations:

```yaml
# Test with different modules
apache_modules_enabled:
  - ssl
  - deflate
  - expires

# Test with different security settings
apache_security_settings:
  - name: ServerTokens
    value: Full
```

## Troubleshooting

### Container Issues
```bash
# Rebuild container from scratch
molecule destroy -s apache
molecule create -s apache
```

### Test Failures
```bash
# Run with verbose output
molecule verify -s apache -- -vvv

# Login to container for debugging
molecule login -s apache

# Check specific service
molecule login -s apache
$ systemctl status apache2
$ apachectl -t
$ apache2ctl -S
```

### Linting Issues
```bash
# Run ansible-lint
ansible-lint roles/apache/

# Run yamllint
yamllint roles/apache/
```

## CI/CD Integration

This scenario is designed to run in CI/CD pipelines:

```yaml
# GitHub Actions example
- name: Run Molecule tests
  run: |
    cd extensions
    molecule test -s apache
```

See `.github/workflows/` for complete CI configuration.

## Additional Resources

- [Molecule Documentation](https://molecule.readthedocs.io/)
- [Apache Role Documentation](../../../roles/apache/README.md)
- [Testing Best Practices](https://docs.ansible.com/ansible/latest/dev_guide/testing.html)