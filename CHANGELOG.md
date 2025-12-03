# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - 2025-12-03
### Fixed
- **Apache Role**:
  - Fixed missing `error_log` and `custom_log` attributes causing failures when not defined in virtual host configurations
  - Added default values for log paths to prevent errors
  - Improved status page module management using `community.general.apache2_module`
  - Fixed status page configuration idempotence with proper state checking
- **Playbooks**:
  - Renamed `apache-setup.yml` to `apache_setup.yml` for consistency with Ansible naming conventions
  - Added `target` variable to define target hosts with default value `apache`
  - Added support for running playbook via collection namespace: `jlira.web_server.apache_setup`

### Changed
- Updated all documentation to reflect `apache_setup` playbook name
- Improved playbook documentation with collection namespace usage examples

## [1.0.0] - 2025-12-02
### Added
- New `certificates` role for automated SSL/TLS certificate management
  - Support for Self-Signed certificates
  - Support for Let's Encrypt (Certbot) with HTTP-01 and DNS-01 challenges
  - Support for importing existing certificates
- New `apache-setup` playbook for orchestrated, multi-phase web server setup with idempotence
- CI workflow for `apache-setup` playbook testing

### Changed
- Refactored Apache virtual host configuration to support dynamic certificate generation
- Updated documentation to reflect new orchestration capabilities
- **Apache Role**:
  - Removed redundant tasks for disabling virtual hosts (logic moved to playbook orchestration)
  - Updated log configuration to use `custom_log` instead of `access_log` for consistency
  - Improved template handling for virtual hosts
  - Added dynamic creation of log directories based on environment variable expansion
  - **Breaking Change**: Removed automatic PHP role inclusion - users must now explicitly include the PHP role before the Apache role if PHP-FPM integration is needed

### Removed
- Legacy `web-server-setup` playbook (replaced by `apache-setup`)
- **Apache Role**:
  - Removed `vhost_template.j2` and `vhost-ssl_template.j2` (consolidated into main templates)
  - Removed automatic PHP role dependency (improves role separation and flexibility)

## [0.0.2] - 2025-10-28
### Added
- Validation for virtual host `listen` ports to ensure they match configured `apache_http_ports` or `apache_https_ports`
- Default values for virtual host `listen` attribute (`*:80` for HTTP, `*:443` for HTTPS)

### Fixed
- Virtual host templates now use default listen values when not explicitly defined
- Fixed variable name typo: `apache_vhosts_from_templates` → `apache_vhosts_from_template`
- Fixed Apache handler to properly guard against accessing undefined attributes in check mode

## [0.0.1] - 2025-10-23
### Added
- Apache role for managing Apache web server installation, configuration, and virtual hosts
- PHP role for managing PHP installation and configuration
- Molecule test scenarios for apache and php roles
- GitHub Actions CI/CD workflows for linting and testing

## [Unreleased]
### Added
- Initial release of the collection
