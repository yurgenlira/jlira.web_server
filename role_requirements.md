Role Requirements
This document outlines the requirements that the certificates role was designed to fulfill, based on an analysis of its configuration and logic.

1. General Requirements
- Unified Management: The role must provide a single interface for managing different types of SSL/TLS certificates (Self-Signed, Let's Encrypt, and Imported).
- Auto-Detection: The role should automatically determine the appropriate certificate mode based on the domain name (TLD) if not explicitly specified.
Local TLDs (.test, .example, .local, etc.) -> selfsigned
Public TLDs -> letsencrypt
- Idempotency: The role must be idempotent, ensuring that certificates are only generated or renewed when necessary.
- Platform Support: The role must support the following Linux distributions:
Ubuntu (Focal, Jammy)
Debian (Bullseye, Bookworm)
- Dependencies: The role relies on the community.crypto Ansible collection.
- Ansible Version: Minimum Ansible version required is 2.9.

2. Certificate Modes
2.1 Self-Signed Certificates
- Customization: Users must be able to customize certificate details:
Common Name (CN)
Organization (O)
Organizational Unit (OU)
Country (C)
State (ST)
Locality (L)
Email
- Technical Parameters:
Configurable key size (default: 4096 bits).
Configurable validity period (default: 365 days).
Support for Subject Alternative Names (SANs), including DNS and IP entries.

2.2 Let's Encrypt (ACME)
- Challenge Types: Support for both http-01 and dns-01 validation methods.
- Plugins: Support for various Certbot plugins when challenge type is http-01:
webroot
apache
nginx
standalone
- DNS Providers: Native support for common DNS providers for dns-01 challenges like:
Cloudflare
Route53
DigitalOcean
Google
Custom (via hooks)
etc
- Environment Selection: Ability to switch between production, staging, and dry-run environments for testing and validation. Allow to change acme URL to test (for use pebble).
- Credentials Management: Secure handling of DNS provider credentials via environment variables.
- Hooks: Support for pre/post-hooks (auth, cleanup, deploy) to handle complex validation scenarios and service reloads.
- Renewal: The role must support automatic certificate renewal (e.g., via systemd timers or cron) without requiring manual intervention.

2.3 Imported Certificates
- External Files: Ability to import existing certificate files (cert, key, chain) from the control node to the target machine.

3. Configuration & Defaults
- Global Defaults: Provide global default values for common settings (e.g., organization info, email) to reduce repetition in configuration.
- Per-Certificate Overrides: Allow overriding global defaults on a per-certificate basis.
- Directory Management: Configurable output directories for certificates and keys.

4. Validation & Safety
- Input Validation: The role must validate the configuration before execution to catch errors early (e.g., missing required fields for a specific mode).
- Safe Defaults: Default to safe settings (e.g., high key size, staging mode if not specified - though code defaults to production, the requirement implies safety controls).

5. Output & Reporting
- Summary: Provide a summary of actions taken (number of certificates managed by type) at the end of execution.
