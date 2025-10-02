# Ansible Collection - jlira.web_server

A collection of Ansible roles and playbooks for managing web servers, including Apache and PHP.

## Requirements

- Ansible version >= 2.18.7
- Supported OS: Debian/Ubuntu

## Installation

To install this collection, use the `ansible-galaxy` command:

```bash
ansible-galaxy collection install jlira.web_server
```

## Usage

Here is an example of how to use the roles in this collection in your playbook:

```yaml
- hosts: webservers
  become: true
  roles:
    - role: jlira.web_server.apache
    - role: jlira.web_server.php
      vars:
        php_version: 8.4
        php_fpm_enabled: true
        php_composer_enabled: true
```

## Roles

This collection includes the following roles:

### `apache`

-   **Description:** Installs and manages the Apache HTTP server on Debian/Ubuntu hosts.
-   **Key Features:**
    -   Installs the `apache2` and `apache2-utils` packages.
    -   Ensures the `apache2` service is started and enabled.
-   **Variables:** This role is intentionally minimal and does not come with predefined variables. You can extend it by defining your own variables in your playbook.

For more details, see the [apache role README](./roles/apache/README.md).

### `php`

-   **Description:** Installs and configures PHP, PHP-FPM, and Composer on Debian/Ubuntu systems.
-   **Key Features:**
    -   Adds the Ondrej PHP PPA for up-to-date PHP versions.
    -   Installs specified PHP versions and extensions.
    -   Optionally installs PHP-FPM and Composer.
-   **Variables:**
    -   `php_version`: The PHP version to install (default: `8.4`).
    -   `php_packages`: A list of PHP packages to install.
    -   `php_fpm_enabled`: Set to `true` to install and enable PHP-FPM (default: `false`).
    -   `php_composer_enabled`: Set to `true` to install Composer (default: `false`).

For more details, see the [php role README](./roles/php/README.md).

## Contributing

Contributions are welcome! Please feel free to open an issue or submit a pull request on the [GitHub repository](https://github.com/yurgenlira/jlira.web_server).

## License

This collection is licensed under the [GPL-2.0-or-later](https://spdx.org/licenses/GPL-2.0-or-later.html) license.
