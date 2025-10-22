# PHP Playbook Upgrade Testing Scenario

This Molecule scenario tests the `playbooks/php-upgrade.yml` playbook, validating that it correctly upgrades PHP from one version to another while properly removing the old version and configuring the new one.

## Test Flow

The scenario uses Molecule's test sequence to validate the PHP upgrade process:

### 1. **prepare.yml** - Install Old PHP Version
- Installs logrotate (required for testing)
- Installs PHP 8.3 as the "old" version using `playbooks/php-install.yml`
- Configures PHP-FPM
- Sets up initial environment

### 2. **converge.yml** - Execute Upgrade Playbook
- Displays upgrade information (upgrading from 8.3 to 8.4)
- Executes `playbooks/php-upgrade.yml`
- Handles:
  - Uninstallation of old PHP 8.3 packages
  - Removal of old PHP-FPM service
  - Cleanup of old configuration files
  - Installation of new PHP 8.4 packages
  - Configuration of new PHP-FPM service
  - Update of alternatives system
  - Logrotate configuration

### 3. **idempotence** - Idempotency Check
- Re-runs converge.yml to ensure no changes occur
- Validates that the upgrade playbook is idempotent

### 4. **verify.yml** - Comprehensive Verification
Tests validate the complete upgrade process:

#### **Version Verification**
- ✅ New PHP version (8.4) is installed and working
- ✅ `php --version` returns PHP 8.4
- ✅ New version is set as default CLI version

#### **Package Cleanup**
- ✅ No old PHP 8.3 packages remain on the system
- ✅ All php8.3-* packages have been removed
- ✅ Package database is clean

#### **Service Management**
- ✅ New PHP-FPM service (php8.4-fpm) is running
- ✅ New PHP-FPM service is enabled at boot
- ✅ Old PHP-FPM service (php8.3-fpm) is removed or disabled
- ✅ No conflicting services running

#### **Alternatives System**
- ✅ PHP 8.4 is set as default version
- ✅ Alternatives system properly configured
- ✅ All CLI tools point to new version

## Running the Tests

### Run Full Test Suite
```bash
cd extensions
molecule test -s php-playbook-upgrade
```

### Run Specific Test Phases

**Syntax check only:**
```bash
molecule syntax -s php-playbook-upgrade
```

**Prepare environment (install old version):**
```bash
molecule prepare -s php-playbook-upgrade
```

**Run upgrade playbook:**
```bash
molecule converge -s php-playbook-upgrade
```

**Run verification tests:**
```bash
molecule verify -s php-playbook-upgrade
```

**Run idempotence check:**
```bash
molecule idempotence -s php-playbook-upgrade
```

**Clean up and destroy:**
```bash
molecule destroy -s php-playbook-upgrade
```

### Development Workflow

**During playbook development:**
```bash
# Create instance and prepare (install old version)
molecule create -s php-playbook-upgrade
molecule prepare -s php-playbook-upgrade

# Test upgrade playbook
molecule converge -s php-playbook-upgrade
molecule verify -s php-playbook-upgrade

# Destroy when done
molecule destroy -s php-playbook-upgrade
```

**Quick iteration:**
```bash
# Re-run upgrade and verify
molecule converge -s php-playbook-upgrade && molecule verify -s php-playbook-upgrade
```

## Test Variables

The test uses custom variables defined in `inventory/hosts.yml`:

```yaml
# Target PHP version (upgrade to)
php_version: "8.4"

# Old PHP version (upgrade from)
php_old_version: "8.3"

# Features
php_fpm_enabled: true
php_cli_set_as_default_version: true
php_cli_logrotate_enabled: true
php_fpm_logrotate_enabled: true

# Packages to install
php_packages:
  - php8.4-cli
  - php8.4-bcmath
  - php8.4-curl
  - php8.4-dev
  - php8.4-gd
  - php8.4-imagick
  - php8.4-imap
  - php8.4-intl
  - php8.4-mbstring
  - php8.4-mysql
  - php8.4-opcache
  - php8.4-xml
  - php8.4-zip
```

## Test Coverage

### Upgrade Process Tested
1. **Pre-upgrade State**
   - ✅ Old PHP version (8.3) installed and working
   - ✅ Old PHP-FPM service running

2. **Upgrade Execution**
   - ✅ Old packages uninstalled
   - ✅ Old services stopped and disabled
   - ✅ Old configuration removed
   - ✅ New packages installed
   - ✅ New services started and enabled
   - ✅ New configuration applied

3. **Post-upgrade State**
   - ✅ New PHP version (8.4) working
   - ✅ New PHP-FPM service running
   - ✅ No old packages remain
   - ✅ Alternatives updated
   - ✅ Default version set correctly

### Scenarios Covered
- **Major Version Upgrade** - PHP 8.3 → PHP 8.4
- **Service Migration** - Old FPM service to new FPM service
- **Clean Removal** - Complete cleanup of old version
- **Configuration Migration** - Settings applied to new version
- **Idempotency** - Upgrade can be run multiple times safely

## What This Test Validates

This scenario specifically tests the **playbook-based upgrade workflow**, ensuring that:

1. The `playbooks/php-upgrade.yml` playbook works correctly
2. Users can upgrade PHP versions using the provided playbook
3. The upgrade process is clean and complete
4. No remnants of the old version remain
5. The new version is properly configured
6. The process is idempotent

This is different from the `php` scenario (role-based testing) which tests the individual role tasks.

## Platform

Tests run on Ubuntu 24.04 using Docker with systemd support.

**Container Image:** `geerlingguy/docker-ubuntu2404-ansible`

**Requirements:**
- Docker with cgroups v2 support
- Systemd-compatible container
- Privileged mode enabled (for systemd)

## Customizing the Test

### Testing Different Version Upgrades

Modify `inventory/hosts.yml` to test different upgrade paths:

```yaml
# Test upgrading from 8.2 to 8.3
php_old_version: "8.2"
php_version: "8.3"
```

Modify `prepare.yml` to install the corresponding old version:

```yaml
- name: Install old PHP version
  import_playbook: ../../../playbooks/php-install.yml
  vars:
    php_version: "8.2"  # Match php_old_version
```

### Testing Without FPM

```yaml
php_fpm_enabled: false
```

### Testing With Different Packages

```yaml
php_packages:
  - php{{ php_version }}-cli
  - php{{ php_version }}-common
  - php{{ php_version }}-json
```

## Troubleshooting

### Container Issues
```bash
# Rebuild container from scratch
molecule destroy -s php-playbook-upgrade
molecule create -s php-playbook-upgrade
```

### Upgrade Failures
```bash
# Run with verbose output
molecule converge -s php-playbook-upgrade -- -vvv

# Login to container for debugging
molecule login -s php-playbook-upgrade

# Check versions and packages
molecule login -s php-playbook-upgrade
$ php --version
$ dpkg -l | grep php8.3
$ dpkg -l | grep php8.4
$ systemctl status php8.4-fpm
```

### Verification Failures
```bash
# Run verify with verbose output
molecule verify -s php-playbook-upgrade -- -vvv

# Check specific assertions
molecule login -s php-playbook-upgrade
$ php --version
$ update-alternatives --list php
```

### Linting Issues
```bash
# Run ansible-lint on playbook
ansible-lint playbooks/php-upgrade.yml

# Run yamllint
yamllint playbooks/php-upgrade.yml
```

## CI/CD Integration

This scenario is designed to run in CI/CD pipelines to validate the upgrade playbook:

```yaml
# GitHub Actions example
- name: Test PHP Upgrade Playbook
  run: |
    cd extensions
    molecule test -s php-playbook-upgrade
```

See `.github/workflows/` for complete CI configuration.

## Related Scenarios

- **`php`** - Tests the PHP role directly (not the playbook)
- **`php-playbook-uninstall`** - Tests the uninstall playbook

## Additional Resources

- [Molecule Documentation](https://molecule.readthedocs.io/)
- [PHP Upgrade Playbook](../../../playbooks/php-upgrade.yml)
- [PHP Role Documentation](../../../roles/php/README.md)
- [Testing Best Practices](https://docs.ansible.com/ansible/latest/dev_guide/testing.html)