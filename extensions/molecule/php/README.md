# PHP Role Molecule Testing Scenario

This Molecule scenario comprehensively tests the `jlira.web_server.php` role, including installation, configuration, and upgrade scenarios.

## Test Flow

The scenario uses Molecule's test sequence to validate different aspects of the PHP role.

### Test Sequence Overview

```
create → prepare → converge → idempotence → side_effect → verify → destroy
```

**What gets tested:**
- **converge.yml**: Installs PHP 8.2
- **side_effect.yml**: Upgrades from PHP 8.2 to PHP 8.4
- **verify.yml**: Verifies PHP 8.4 exists and PHP 8.2 is removed

### 1. **converge.yml** - Initial Installation (PHP 8.2)
- Installs PHP 8.2 with all features enabled
- Configures PHP-FPM
- Installs Composer
- Installs MSSQL extensions
- Sets up logrotate for CLI and FPM

### 2. **idempotence** - Idempotency Check
- Re-runs converge.yml to ensure no changes occur
- Validates that the role is truly idempotent

### 3. **side_effect.yml** - Upgrade Scenario (PHP 8.2 → 8.4)
- **Tests the `uninstall` functionality**
- Upgrades from PHP 8.2 (installed in converge.yml) to PHP 8.4
- Validates that:
  - Old PHP 8.2 packages are removed
  - Old PHP-FPM service is stopped and disabled
  - Old configuration files are cleaned up
  - Alternatives system is updated
  - New PHP 8.4 is installed and configured

### 4. **verify.yml** - Comprehensive Verification (PHP 8.4)
⚠️ **Runs AFTER side_effect.yml** - Tests that PHP 8.4 is installed and PHP 8.2 is removed.

Tests are organized by task file:

#### **uninstall.yml tests**
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

⚠️ **Important:** The test sequence must be followed in order: `create → converge → side-effect → verify`

**1. Syntax check only:**
```bash
molecule syntax -s php
```

**2. Create container:**
```bash
molecule create -s php
```

**3. Converge (install PHP 8.2):**
```bash
molecule converge -s php
```

**4. Test idempotence:**
```bash
molecule idempotence -s php
```

**5. Side effect (upgrade PHP 8.2 → 8.4):**
```bash
molecule side-effect -s php
```
⚠️ **This MUST run before verify!** It upgrades from 8.2 to 8.4.

**6. Run verification tests (verify PHP 8.4):**
```bash
molecule verify -s php
```
This tests that PHP 8.4 is installed and 8.2 is removed.

**7. Clean up and destroy:**
```bash
molecule destroy -s php
```

### Development Workflow

**During role development (full sequence):**
```bash
# Create instance
molecule create -s php

# Install PHP 8.2
molecule converge -s php

# Test idempotency of 8.2 installation
molecule idempotence -s php

# Upgrade from 8.2 to 8.4
molecule side-effect -s php

# Verify 8.4 is installed and 8.2 removed
molecule verify -s php

# Destroy when done
molecule destroy -s php
```

**Quick iteration (testing changes to converge.yml):**
```bash
# Re-run installation and upgrade
molecule destroy -s php
molecule create -s php
molecule converge -s php
molecule side-effect -s php
molecule verify -s php
```

**Testing only the upgrade process:**
```bash
# Assumes container exists with PHP 8.2 already installed
molecule side-effect -s php
molecule verify -s php
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