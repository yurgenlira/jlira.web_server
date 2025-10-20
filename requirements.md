# Requirements for jlira.web_server Ansible Collection

## Functional Requirements

The collection must provide the following capabilities through its roles:

### `apache` Role

*   **Installation:** Ensures that Apache and its required modules are installed on the target system.
*   **Core Configuration:** Manages the main `apache.conf` file, allowing for global directives like `ServerName`, `ServerTokens`, and `LimitRequestLine`. Supports custom environment variables and configuration directories.
*   **Port Configuration:** Allows configuration of multiple HTTP and HTTPS ports (default: 80 for HTTP, 443 for HTTPS when SSL is enabled).
*   **Virtual Host Management:** Provides a flexible way to define and manage virtual hosts, supporting both static configuration files and templates for dynamic VHost creation. It also manages custom configuration files.
*   **PHP Integration:** Orchestrates the installation and configuration of PHP, ensuring that Apache is correctly set up to work with PHP-FPM.
*   **Security:** Manages security-related aspects, such as enabling SSL/TLS, setting security headers, configuring basic authentication (`.htpasswd`)
*   **Maintenance:** Handles the creation of custom log directories and validates configuration changes before reloading the Apache service to prevent downtime.
*   **Monitoring:** Sets up a server status page with controlled access.

### `php` Role

*   **Installation:** Handles the installation of a specific PHP version and its extensions (using PPA). It can also remove and purge old versions if specified.
*   **Configuration:**
    *   Configures `php.ini` files for both CLI and PHP-FPM.
    *   Sets the default system-wide PHP version for CLI and related tools.
    *   Configures FPM connection pools (`www.conf`).
*   **Integration:** Sets up PHP-FPM to work with a web server (like Apache) and can set the default PHP version for the system.
*   **Extension Management:** Supports the installation of additional extensions, with specific support for the Microsoft SQL Server driver (`mssql`).
*   **Tooling:** Installs and configures related tools like Composer.
*   **Maintenance:** Sets up log rotation for PHP logs.
*   **Monitoring:** Configures a PHP-FPM status page for monitoring.
