# Web Server Playbooks

This directory contains example playbooks for managing web server infrastructure using the `jlira.web_server` collection.

## Available Playbooks

### Web Server Setup

#### 1. web-server-setup.yml
Complete web server setup with Apache and PHP integration.

**Use when:**
- Setting up a new web server
- Installing Apache with PHP-FPM support
- Creating a standard LAMP-like environment

**Features:**
- Apache web server installation and configuration
- PHP installation with FPM support
- Automatic PHP-FPM integration with Apache
- SSL support (enabled by default)
- Customizable ports and server name

**Example:**
```bash
# Basic installation (Apache + PHP 8.4 with FPM)
ansible-playbook -i inventory playbooks/web-server-setup.yml

# With custom PHP version
ansible-playbook -i inventory playbooks/web-server-setup.yml \
  -e "php_version=8.3"

# With custom Apache server name
ansible-playbook -i inventory playbooks/web-server-setup.yml \
  -e "apache_server_name=example.com"

# Without PHP-FPM integration
ansible-playbook -i inventory playbooks/web-server-setup.yml \
  -e "apache_php_fpm_integration=false" \
  -e "php_fpm_enabled=false"

# Skip confirmation prompts
ansible-playbook -i inventory playbooks/web-server-setup.yml \
  -e "skip_confirm=true"
```

---

#### 2. web-server-multi-php.yml
Advanced web server setup with multiple PHP versions.

**Use when:**
- Running applications requiring different PHP versions
- Maintaining legacy applications alongside modern ones
- Testing applications across PHP versions
- Gradual migration between PHP versions

**Features:**
- Apache web server installation
- Multiple PHP versions with separate FPM pools
- Independent configuration per PHP version
- Select default CLI and Apache integration versions
- Per-version composer installation

**Example:**
```bash
# Basic installation with default config (PHP 8.3 and 8.4)
ansible-playbook -i inventory playbooks/web-server-multi-php.yml

# With custom configuration
ansible-playbook -i inventory playbooks/web-server-multi-php.yml \
  -e '@playbooks/vars/multi-php-example.yml'

# Skip confirmation prompts
ansible-playbook -i inventory playbooks/web-server-multi-php.yml \
  -e "skip_confirm=true"
```

**Configuration:**
See `playbooks/vars/multi-php-example.yml` for a complete configuration example with three PHP versions (8.2, 8.3, 8.4).

**Important:**
- Only one PHP version can be set as default CLI
- Only one PHP version can be set as default in Apache
- Each PHP version has its own FPM pool
- Virtual hosts can be configured to use specific PHP versions

---

### PHP Management

#### 3. php-install.yml
Fresh installation of PHP on servers.

**Use when:**
- Installing PHP for the first time
- Adding PHP to new servers
- Setting up a clean PHP environment

**Example:**
```bash
# Install PHP 8.4 (default)
ansible-playbook -i inventory playbooks/php-install.yml

# Install specific PHP version
ansible-playbook -i inventory playbooks/php-install.yml -e "php_version=8.3"

# Install with PHP-FPM
ansible-playbook -i inventory playbooks/php-install.yml -e "php_fpm_enabled=true"
```

---

#### 4. php-upgrade.yml
Upgrade PHP from one version to another.

**Use when:**
- Migrating from PHP 8.3 to 8.4
- Upgrading to newer PHP versions
- Replacing an old PHP installation

**Features:**
- Automatically removes old PHP version
- Installs new PHP version
- Updates alternatives system
- Maintains configuration consistency

**Example:**
```bash
# Upgrade from PHP 8.3 to 8.4
ansible-playbook -i inventory playbooks/php-upgrade.yml \
  -e "php_old_version=8.3" \
  -e "php_version=8.4"

# Upgrade with PHP-FPM
ansible-playbook -i inventory playbooks/php-upgrade.yml \
  -e "php_old_version=8.3" \
  -e "php_version=8.4" \
  -e "php_fpm_enabled=true"

# Dry run (check mode)
ansible-playbook -i inventory playbooks/php-upgrade.yml \
  -e "php_old_version=8.3" \
  -e "php_version=8.4" \
  --check
```

**Important:**
- Always specify both `php_old_version` and `php_version`
- Test in staging environment first
- Backup databases and configurations
- Verify application compatibility with new PHP version

---

#### 5. php-uninstall.yml
Complete removal of PHP from servers.

**Use when:**
- Decommissioning PHP applications
- Cleaning up test environments
- Preparing for fresh installation

**Features:**
- Interactive confirmation prompt
- Removes all PHP packages
- Stops and disables services
- Purges configuration files
- Cleans up alternatives system

**Example:**
```bash
# Uninstall PHP 8.4
ansible-playbook -i inventory playbooks/php-uninstall.yml \
  -e "php_version_to_remove=8.4"

# Skip confirmation (use with caution!)
ansible-playbook -i inventory playbooks/php-uninstall.yml \
  -e "php_version_to_remove=8.4" \
  --extra-vars "{'ansible_no_pause': true}"

# Dry run
ansible-playbook -i inventory playbooks/php-uninstall.yml \
  -e "php_version_to_remove=8.4" \
  --check
```

**Warning:**
- This completely removes PHP from the system
- Applications depending on PHP will stop working
- Consider using `php-upgrade.yml` instead for version changes

---

## Common Options

### Target Specific Hosts
```bash
# Target single host
ansible-playbook -i inventory playbooks/php-install.yml --limit webserver01

# Target specific group
ansible-playbook -i inventory playbooks/php-install.yml --limit production
```

### Using Tags
```bash
# Only install PHP (skip configuration)
ansible-playbook -i inventory playbooks/php-install.yml --tags install

# Only configure PHP-FPM
ansible-playbook -i inventory playbooks/php-install.yml --tags php_configure_fpm

# Remove old version only
ansible-playbook -i inventory playbooks/php-upgrade.yml --tags php_uninstall
```

### Verbose Output
```bash
# Standard verbosity
ansible-playbook -i inventory playbooks/php-install.yml -v

# More details
ansible-playbook -i inventory playbooks/php-install.yml -vv

# Debug level
ansible-playbook -i inventory playbooks/php-install.yml -vvv
```

---

## Best Practices

### Before Running Playbooks

1. **Test in Staging**
   ```bash
   ansible-playbook -i staging playbooks/php-upgrade.yml \
     -e "php_old_version=8.3" \
     -e "php_version=8.4" \
     --check
   ```

2. **Backup Important Data**
   - Database dumps
   - Configuration files
   - Custom PHP extensions
   - Application code

3. **Check PHP Version Compatibility**
   - Review PHP changelog
   - Test applications with new version
   - Update dependencies if needed

4. **Plan Maintenance Window**
   - Schedule during low-traffic periods
   - Notify users of downtime
   - Have rollback plan ready

### During Execution

1. **Monitor Progress**
   ```bash
   # Use verbose mode
   ansible-playbook -i inventory playbooks/php-upgrade.yml -v
   ```

2. **Use Check Mode First**
   ```bash
   # Dry run to see what would change
   ansible-playbook -i inventory playbooks/php-upgrade.yml --check --diff
   ```

### After Execution

1. **Verify Installation**
   ```bash
   ansible all -i inventory -m command -a "php --version"
   ansible all -i inventory -m command -a "php-fpm8.4 --version"
   ```

2. **Test Applications**
   - Check application functionality
   - Review error logs
   - Monitor performance

3. **Update Documentation**
   - Record PHP version changes
   - Update deployment guides
   - Document any issues encountered

---

## Customization

All playbooks can be customized by:

1. **Editing Variables**
   - Modify playbook vars directly
   - Use extra vars on command line
   - Create inventory group_vars

2. **Adding Custom Tasks**
   - Use `pre_tasks` and `post_tasks`
   - Create custom roles
   - Extend existing playbooks

3. **Integration**
   - Combine with other roles
   - Add to CI/CD pipelines
   - Use with orchestration tools

---

## Troubleshooting

### Common Issues

**PHP version conflicts:**
```bash
# Check installed versions
dpkg -l | grep php

# Manually remove conflicts
sudo apt-get purge 'php8.3*'
```

**Service not starting:**
```bash
# Check service status
sudo systemctl status php8.4-fpm

# View logs
sudo journalctl -u php8.4-fpm -n 50
```

**Alternatives not updating:**
```bash
# Check alternatives
update-alternatives --display php

# Manually set
sudo update-alternatives --set php /usr/bin/php8.4
```

---

## Additional Resources

- [PHP Role Documentation](../roles/php/README.md)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [PHP Migration Guides](https://www.php.net/manual/en/migration84.php)
- [Collection README](../README.md)
