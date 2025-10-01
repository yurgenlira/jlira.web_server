
jlira.web_server.apache
=======================

Install and manage the Apache HTTP server on Debian/Ubuntu hosts.

This role installs the `apache2` package, ensures the service is enabled and
running, and provides a simple, minimal foundation you can extend with
configuration templates, virtual hosts, and modules.

Requirements
------------

- Target systems: Debian/Ubuntu family (the role uses the apt/package and
  service modules).

Role variables
--------------

This role intentionally ships with no opinionated defaults. The `defaults`
directory is empty; tune variables in your playbook or group/host vars as
required for your deployment. The role currently performs two main actions:

- Install packages: `apache2` and `apache2-utils`.
- Start and enable the `apache2` service.

If you need to install extra packages or enable modules, set variables in the
playbook (or extend the role tasks) to meet your needs.

Behavior / Tasks
----------------

- `tasks/main.yml` includes `install.yml` which:
  - Installs `apache2` and `apache2-utils` with `state: present` and
    `update_cache: true`.
  - Starts and enables the `apache2` service.

Handlers
--------

The role includes an empty `handlers/main.yml`. Add handlers if you extend the
role to manage configuration files (for example, `notify: restart apache`).

Dependencies
------------

No external role dependencies are declared in `meta/main.yml`.

Example playbook
----------------

Simple usage:

    - hosts: web
      become: true
      roles:
        - role: jlira.web_server.apache

Extending the role (example):

    - hosts: web
      become: true
      vars:
        extra_apache_packages:
          - libapache2-mod-php
      tasks:
        - name: install extra Apache packages
          apt:
            name: "{{ extra_apache_packages }}"
            state: present

Testing
-------

A simple test playbook is provided at `tests/test.yml`. It runs the role
against `localhost` and can be used as a starting point for Molecule or CI
test scenarios.

License and author
------------------

License: GPL-2.0-or-later

Author: Julio Lira

Where to look next
------------------

- `tasks/install.yml` — package installation and service management.
- `defaults/main.yml` — role defaults (currently empty).
- `meta/main.yml` — role metadata (author, license, minimum Ansible version).
