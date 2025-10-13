# PHP Role Molecule Testing Scenario

This Molecule scenario comprehensively tests the `jlira.web_server.php` role, including installation, configuration, and upgrade scenarios.

## Test Flow

The scenario uses Molecule's test sequence to validate different aspects of the PHP role:

### 1. **converge.yml** - Initial Installation
- Installs PHP 8.2 with all features enabled
- Configures PHP-FPM
- Installs Composer
- Installs MSSQL extensions
- Sets up logrotate for CLI and FPM

### 2. **idempotence** - Idempotency Check
- Re-runs converge.yml to ensure no changes occur
- Validates that the role is truly idempotent

### 3. **side_effect.yml** - Upgrade Scenario
- **Tests the `remove_old_version` functionality**
- Upgrades from PHP 8.2 (installed in converge.yml) to PHP 8.4
- Validates that:
  - Old PHP 8.2 packages are removed
  - Old PHP-FPM service is stopped and disabled
  - Old configuration files are cleaned up
  - Alternatives system is updated
  - New PHP 8.4 is installed and configured

### 4. **verify.yml** - Comprehensive Verification
Tests are organized by task file:

#### **remove_old_version.yml tests**
- ✅ No old PHP 8.2 packages remain
- ✅ Old configuration directory removed
- ✅ Old logrotate configs removed
- ✅ Old PHP-FPM service not running
- ✅ Old version not in alternatives system
- ✅ Current PHP version is not the old version

#### **install.yml tests**
- ✅ PHP 8.4 is installed
- ✅ PHP extensions are loaded
- ✅ PHP-FPM is installed (if enabled)
- ✅ Composer is installed (if enabled)

#### **set_default_cli_version.yml tests**
- ✅ PHP binary exists and is executable
- ✅ Default PHP version is correct
- ✅ Alternatives system configured correctly
- ✅ All CLI tools (phar, phpize, php-config) work

#### **configure_cli.yml tests**
- ✅ PHP CLI configuration file exists
- ✅ Settings are correctly applied
- ✅ Log directory exists

#### **logrotate_cli.yml tests**
- ✅ Logrotate config exists with correct permissions
- ✅ Configuration syntax is valid
- ✅ Log path is correct

#### **configure_fpm.yml tests**
- ✅ PHP-FPM service is running and enabled
- ✅ PHP-FPM configuration is correct
- ✅ Pool configuration is correct
- ✅ Socket/port is listening

#### **logrotate_fpm.yml tests**
- ✅ FPM logrotate config exists
- ✅ Configuration is valid

## Running the Tests

### Run Full Test Suite
```bash
cd extensions
molecule test -s php
```

### Run Specific Test Phases

**Syntax check only:**
```bash
molecule syntax -s php
```

**Create and converge:**
```bash
molecule converge -s php
```

**Run verification tests:**
```bash
molecule verify -s php
```

**Test upgrade scenario:**
```bash
molecule side-effect -s php
```

**Run idempotence check:**
```bash
molecule idempotence -s php
```

**Clean up and destroy:**
```bash
molecule destroy -s php
```

### Development Workflow

**During role development:**
```bash
# Create instance
molecule create -s php

# Apply changes and test
molecule converge -s php
molecule verify -s php

# Test upgrade scenario
molecule side-effect -s php
molecule verify -s php

# Destroy when done
molecule destroy -s php
```

**Quick iteration:**
```bash
# Skip destroy and re-use container
molecule converge -s php && molecule verify -s php
```

## Test Variables

The test uses custom variables defined in `vars.yml`:

```yaml
php_version: "8.2"                    # Initial version for converge.yml
php_fpm_enabled: true                 # Test PHP-FPM functionality
php_composer_enabled: true            # Test Composer installation
php_mssql_extension_enabled: true     # Test MSSQL extension
php_packages:                         # Test custom packages
  - php8.2-bcmath
  - php8.2-curl
php_cli_settings:                     # Test custom CLI settings
  memory_limit:
    value: "512M"
php_pool_settings:                    # Test custom pool settings
  pm_max_children: 10
```

## Test Coverage

### Features Tested
- ✅ Fresh PHP installation
- ✅ PHP version upgrade (8.2 → 8.4)
- ✅ Old version removal
- ✅ PHP-FPM installation and configuration
- ✅ Composer installation
- ✅ MSSQL extension installation
- ✅ Custom php.ini settings (CLI and FPM)
- ✅ Pool configuration
- ✅ Logrotate configuration
- ✅ Alternatives system management
- ✅ Service management (start, enable, restart)
- ✅ Idempotency
- ✅ Configuration file permissions
- ✅ Directory creation

### Scenarios Covered
1. **Fresh Installation** - First time PHP installation
2. **Upgrade** - Migrating from one PHP version to another
3. **Configuration Changes** - Modifying php.ini settings
4. **Service Management** - Start, stop, enable, disable
5. **Clean Removal** - Complete cleanup of old versions

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
   roles/php/tasks/my_feature.yml → tests/my_feature.yml
   ```

2. Add test inclusion to `verify.yml`:
   ```yaml
   - name: Include my_feature.yml tests
     ansible.builtin.include_tasks:
       file: tests/my_feature.yml
     when: php_my_feature_enabled
   ```

3. Run tests:
   ```bash
   molecule verify -s php
   ```

### Testing Custom Scenarios

Modify `vars.yml` to test different configurations:

```yaml
# Test with different PHP version
php_version: "8.3"

# Test without FPM
php_fpm_enabled: false

# Test with different pool settings
php_pool_settings:
  pm: static
  pm_max_children: 20
```

## Troubleshooting

### Container Issues
```bash
# Rebuild container from scratch
molecule destroy -s php
molecule create -s php
```

### Test Failures
```bash
# Run with verbose output
molecule verify -s php -- -vvv

# Login to container for debugging
molecule login -s php

# Check specific service
molecule login -s php
$ systemctl status php8.4-fpm
$ php --version
```

### Linting Issues
```bash
# Run ansible-lint
ansible-lint roles/php/

# Run yamllint
yamllint roles/php/
```

## CI/CD Integration

This scenario is designed to run in CI/CD pipelines:

```yaml
# GitHub Actions example
- name: Run Molecule tests
  run: |
    cd extensions
    molecule test -s php
```

See `.github/workflows/` for complete CI configuration.

## Additional Resources

- [Molecule Documentation](https://molecule.readthedocs.io/)
- [PHP Role Documentation](../../../roles/php/README.md)
- [Testing Best Practices](https://docs.ansible.com/ansible/latest/dev_guide/testing.html)