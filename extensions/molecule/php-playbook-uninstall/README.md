# PHP Playbook Uninstall Testing Scenario

This Molecule scenario tests the `playbooks/php-uninstall.yml` playbook, validating that it completely removes a PHP installation including all packages, services, configuration files, and related components.

## Test Flow

The scenario uses Molecule's test sequence to validate the PHP uninstall process:

### 1. **prepare.yml** - Install PHP Version to be Removed
- Displays preparation information
- Installs logrotate (required for testing)
- Installs PHP 8.2 with:
  - Common extensions (curl, mbstring, xml, zip, mysql, gd)
  - PHP-FPM enabled
  - CLI tools configured
  - Logrotate for both CLI and FPM
- Verifies PHP is installed and running
- Confirms PHP-FPM service is active

### 2. **converge.yml** - Execute Uninstall Playbook
- Displays uninstall information
- Executes `playbooks/php-uninstall.yml` with:
  - Target version: PHP 8.2 (from `php_version_to_remove`)
  - Confirmation skip enabled (`php_skip_confirm: true`)
- Handles complete removal:
  - Stops and disables PHP-FPM service
  - Removes all PHP packages
  - Deletes configuration directories
  - Removes logrotate configurations
  - Cleans up alternatives system
  - Removes PHP binaries

### 3. **verify.yml** - Comprehensive Verification
Tests validate the complete removal process:

#### **Package Removal**
- ✅ All php8.2-* packages are removed from the system
- ✅ No installed packages match the removed version
- ✅ Package database shows no remnants

#### **Service Removal**
- ✅ PHP-FPM service (php8.2-fpm) is not running
- ✅ Service unit is not-found or inactive
- ✅ No systemd units remain for removed version

#### **Configuration Cleanup**
- ✅ PHP configuration directory (`/etc/php/8.2`) is removed
- ✅ CLI configuration files deleted
- ✅ FPM configuration files deleted
- ✅ Pool configuration removed

#### **Logrotate Cleanup**
- ✅ Logrotate configurations removed
- ✅ No `/etc/logrotate.d/php8.2-*` files remain

#### **Binary Removal**
- ✅ PHP binary (`php8.2`) not found in PATH
- ✅ CLI tools removed (phar, phpize, php-config)
- ✅ Alternatives system cleaned up

## Running the Tests

### Run Full Test Suite
```bash
cd extensions
molecule test -s php-playbook-uninstall
```

### Run Specific Test Phases

**Syntax check only:**
```bash
molecule syntax -s php-playbook-uninstall
```

**Prepare environment (install PHP to be removed):**
```bash
molecule prepare -s php-playbook-uninstall
```

**Run uninstall playbook:**
```bash
molecule converge -s php-playbook-uninstall
```

**Run verification tests:**
```bash
molecule verify -s php-playbook-uninstall
```

**Clean up and destroy:**
```bash
molecule destroy -s php-playbook-uninstall
```

### Development Workflow

**During playbook development:**
```bash
# Create instance and prepare (install PHP)
molecule create -s php-playbook-uninstall
molecule prepare -s php-playbook-uninstall

# Test uninstall playbook
molecule converge -s php-playbook-uninstall
molecule verify -s php-playbook-uninstall

# Destroy when done
molecule destroy -s php-playbook-uninstall
```

**Quick iteration:**
```bash
# Re-run uninstall and verify
molecule converge -s php-playbook-uninstall && molecule verify -s php-playbook-uninstall
```

## Test Variables

The test uses variables configured in `molecule.yml`:

```yaml
# Version to uninstall
php_version_to_remove: "8.2"

# Version to install (for prepare phase)
php_version: "8.4"

# Features
php_fpm_enabled: true

# Skip confirmation prompt (for automated testing)
php_skip_confirm: true
```

Variables used during prepare phase:

```yaml
# Packages installed (to be removed later)
php_packages:
  - php8.2-curl
  - php8.2-mbstring
  - php8.2-xml
  - php8.2-zip
  - php8.2-mysql
  - php8.2-gd

# Configuration
php_cli_set_as_default_version: true
php_cli_logrotate_enabled: true
php_fpm_logrotate_enabled: true
```

## Test Coverage

### Uninstall Process Tested
1. **Pre-uninstall State**
   - ✅ PHP version (8.2) installed and working
   - ✅ PHP-FPM service running and enabled
   - ✅ Configuration files present
   - ✅ Logrotate configs present

2. **Uninstall Execution**
   - ✅ Services stopped and disabled
   - ✅ Packages removed
   - ✅ Configuration directories deleted
   - ✅ Logrotate configs removed
   - ✅ Binaries removed
   - ✅ Alternatives cleaned up

3. **Post-uninstall State**
   - ✅ No PHP packages remain
   - ✅ No services running
   - ✅ No configuration files exist
   - ✅ No binaries in PATH
   - ✅ System is clean

### Scenarios Covered
- **Complete Removal** - All components removed
- **Service Cleanup** - FPM service properly stopped and removed
- **Configuration Cleanup** - All config files deleted
- **Package Cleanup** - All packages uninstalled
- **Binary Cleanup** - All executables removed

## What This Test Validates

This scenario specifically tests the **playbook-based uninstall workflow**, ensuring that:

1. The `playbooks/php-uninstall.yml` playbook works correctly
2. Users can completely remove PHP installations using the provided playbook
3. The uninstall process is thorough and complete
4. No remnants of the removed version remain
5. The system is left in a clean state
6. Automated uninstall works (with `php_skip_confirm`)

This is different from:
- **`php` scenario** - Tests the role's uninstall tasks as part of an upgrade
- **`php-playbook-upgrade`** - Tests upgrading between versions (which includes uninstall)

## Platform

Tests run on Ubuntu 24.04 using Docker with systemd support.

**Container Image:** `geerlingguy/docker-ubuntu2404-ansible`

**Requirements:**
- Docker with cgroups v2 support
- Systemd-compatible container
- Privileged mode enabled (for systemd)

## Customizing the Test

### Testing Different PHP Versions

Modify variables in `molecule.yml`:

```yaml
provisioner:
  inventory:
    host_vars:
      ${MOLECULE_SCENARIO_NAME}-instance:
        php_version_to_remove: "8.3"  # Change version to test
        php_version: "8.4"
        php_fpm_enabled: true
        php_skip_confirm: true
```

Update `prepare.yml` to install the matching version:

```yaml
- name: Install PHP version to be removed
  ansible.builtin.include_role:
    name: php
  vars:
    php_version: "8.3"  # Match php_version_to_remove
```

### Testing Without FPM

```yaml
php_fpm_enabled: false
```

### Testing With Different Packages

Modify `prepare.yml`:

```yaml
php_packages:
  - "php{{ php_version_to_remove }}-cli"
  - "php{{ php_version_to_remove }}-common"
  - "php{{ php_version_to_remove }}-json"
```

### Testing Manual Confirmation

For testing the interactive confirmation prompt:

```yaml
php_skip_confirm: false  # Will require manual confirmation
```

**Note:** This won't work in automated testing but can be tested manually.

## Troubleshooting

### Container Issues
```bash
# Rebuild container from scratch
molecule destroy -s php-playbook-uninstall
molecule create -s php-playbook-uninstall
```

### Preparation Failures
```bash
# Run prepare with verbose output
molecule prepare -s php-playbook-uninstall -- -vvv

# Login to container for debugging
molecule login -s php-playbook-uninstall

# Verify installation
molecule login -s php-playbook-uninstall
$ php --version
$ systemctl status php8.2-fpm
```

### Uninstall Failures
```bash
# Run with verbose output
molecule converge -s php-playbook-uninstall -- -vvv

# Check what remains
molecule login -s php-playbook-uninstall
$ dpkg -l | grep php8.2
$ systemctl list-units | grep php
$ ls -la /etc/php/
```

### Verification Failures
```bash
# Run verify with verbose output
molecule verify -s php-playbook-uninstall -- -vvv

# Check specific components
molecule login -s php-playbook-uninstall
$ dpkg -l | grep php8.2
$ which php8.2
$ ls -la /etc/php/8.2
$ ls -la /etc/logrotate.d/php*
```

### Linting Issues
```bash
# Run ansible-lint on playbook
ansible-lint playbooks/php-uninstall.yml

# Run yamllint
yamllint playbooks/php-uninstall.yml
```

## CI/CD Integration

This scenario is designed to run in CI/CD pipelines to validate the uninstall playbook:

```yaml
# GitHub Actions example
- name: Test PHP Uninstall Playbook
  run: |
    cd extensions
    molecule test -s php-playbook-uninstall
```

See `.github/workflows/` for complete CI configuration.

## Use Cases

This test validates the playbook for these real-world scenarios:

1. **Complete Removal** - Removing PHP entirely from a server
2. **Version Cleanup** - Removing old versions after upgrade
3. **Fresh Start** - Cleaning up before reinstalling
4. **Automated Cleanup** - Scripted removal in pipelines

## Related Scenarios

- **`php`** - Tests the PHP role directly (includes uninstall as part of upgrade)
- **`php-playbook-upgrade`** - Tests the upgrade playbook (which uses uninstall)

## Additional Resources

- [Molecule Documentation](https://molecule.readthedocs.io/)
- [PHP Uninstall Playbook](../../../playbooks/php-uninstall.yml)
- [PHP Role Documentation](../../../roles/php/README.md)
- [Testing Best Practices](https://docs.ansible.com/ansible/latest/dev_guide/testing.html)