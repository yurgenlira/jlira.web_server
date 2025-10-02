jlira.web_server.php
=====================

Install and configure PHP (and common extensions) on Debian/Ubuntu systems.

This role adds the Ondrej PHP PPA, installs the requested PHP packages, optionally
installs PHP-FPM, and can install Composer globally. The role is intentionally
minimal and splits additional configuration (FPM pool tuning, php.ini tweaks,
logrotate, status page, extensions like mssql) into optional tasks that are
commented out in the role for easy enablement.

Requirements
------------

- Target systems: Debian/Ubuntu family (the role uses apt and adds an APT PPA).
- The role uses the `apt-add-repository` command when adding the PPA. Ensure
  the `software-properties-common` package is available on older images if
  necessary.

Role variables
--------------

The most important configurable variables live in `defaults/main.yml`. Key
variables (with their default values) are:

- `php_version`: 8.4
  - The PHP major/minor version installed (used in package names).
- `php_packages`:
  - A list of package names installed by apt. By default it contains:
    - `php{{ php_version }}` and `php{{ php_version }}-cli`
- `php_composer_enabled`: false
  - When true, the role downloads and installs Composer to `/usr/local/bin`.
- `php_fpm_enabled`: false
  - When true, the role installs the `php{{ php_version }}-fpm` package.
- `php_ppa_repository`: "ppa:ondrej/php"
  - The PPA used to install modern PHP packages.
- `php_ppa_file`: ondrej-ubuntu-php-noble.sources
  - File name used to track the created APT source file.

See `defaults/main.yml` for additional commented variables and detailed
examples of php.ini and php-fpm configuration snippets that can be enabled.

Behavior / Tasks
----------------

- `tasks/main.yml` includes `install.yml` which performs these actions:
  - Adds the Ondrej PHP PPA (creates `/etc/apt/sources.list.d/{{ php_ppa_file }}`).
  - Installs packages listed in `php_packages`.
  - Installs `php{{ php_version }}-fpm` when `php_fpm_enabled` is true.
  - Installs Composer when `php_composer_enabled` is true.

Optional tasks (present but commented) include configuring CLI/FPM php.ini
settings, logrotate, status pages, and Microsoft SQL Server PHP extension
installation. Enable them as required by your environment.

Dependencies
------------

This role has no internal role dependencies (see `meta/main.yml`). You may want
to use it together with your webserver role (Apache or nginx) to wire PHP-FPM
to the web server.

Example playbook
----------------

Install the default PHP packages (no FPM, no Composer):

    - hosts: webservers
      roles:
        - role: jlira.web_server.php

Install PHP 8.4 with FPM and Composer:

    - hosts: webservers
      vars:
        php_version: 8.4
        php_fpm_enabled: true
        php_composer_enabled: true
      roles:
        - role: jlira.web_server.php

Testing
-------

A simple Molecule / test playbook is provided at `tests/test.yml`. It runs the
role against `localhost` as `root` and is intended as a starting point for
adding CI tests.

Handlers
--------

The role ships an empty `handlers/main.yml`. If you add services that require
notification (for example restarting php-fpm), define handlers and call them
from your tasks.

License and author
------------------

License: GPL-2.0-or-later

Author: Julio Lira

Where to look next
------------------

- `defaults/main.yml` — all default variables and commented examples for php.ini/
  php-fpm configuration.
- `tasks/install.yml` — concrete package and Composer installation steps.
- `meta/main.yml` — role metadata (author, license, minimum Ansible version).

Security notes
--------------

- Installing Composer from the upstream installer downloads a PHP script from
  getcomposer.org. Ensure you trust the network and sources when enabling
  `php_composer_enabled` on production systems.
- Adding PPAs changes package sources for the host. Confirm the PPA choice is
  appropriate for your distribution and security policy.
