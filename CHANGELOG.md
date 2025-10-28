# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]
### Added
- Initial release of the collection

## [0.0.1] - 2025-10-23
### Added
- Apache role for managing Apache web server installation, configuration, and virtual hosts
- PHP role for managing PHP installation and configuration
- Molecule test scenarios for apache and php roles
- GitHub Actions CI/CD workflows for linting and testing

## [0.0.2] - 2025-10-28
### Added
- Validation for virtual host `listen` ports to ensure they match configured `apache_http_ports` or `apache_https_ports`
- Default values for virtual host `listen` attribute (`*:80` for HTTP, `*:443` for HTTPS)

### Fixed
- Virtual host templates now use default listen values when not explicitly defined
- Fixed variable name typo: `apache_vhosts_from_templates` → `apache_vhosts_from_template`
- Fixed Apache handler to properly guard against accessing undefined attributes in check mode
