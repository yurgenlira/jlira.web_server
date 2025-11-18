# Molecule Test Scenario: Certificates Role

This comprehensive Molecule scenario tests all features of the `jlira.web_server.certificates` role including self-signed certificates, import mode, and Let's Encrypt certificate acquisition using Pebble ACME test server.

## Test Coverage

### Installation Tests (`tests/install.yml`)
- ✅ python3-cryptography package installation (selfsigned/import modes)
- ✅ certbot and python3-certbot packages installation (letsencrypt mode)
- ✅ Certbot plugin packages installation (python3-certbot-apache, python3-certbot-dns-*)
- ✅ Certificate directory creation and permissions (0755)
- ✅ Private key directory creation and permissions (0700)
- ✅ Directory ownership (root:root)

### Self-Signed Certificate Tests (`tests/selfsigned.yml`)
- ✅ Basic certificate generation
- ✅ Advanced certificate with full X.509 fields (O, OU, C, ST, L, email)
- ✅ Subject Alternative Names (SAN) support (DNS and IP)
- ✅ Multi-domain wildcard certificates
- ✅ Certificate file permissions (0644)
- ✅ Private key file permissions (0600)
- ✅ Certificate validity period
- ✅ Private key/certificate matching
- ✅ CSR cleanup and idempotence

### Import Mode Tests (`tests/import.yml` and `side_effect.yml`)
- ✅ Certificate import from source files
- ✅ Private key import from source files
- ✅ Certificate chain import (when provided)
- ✅ Imported file permissions (cert: 0644, key: 0600, chain: 0644)
- ✅ File ownership verification (root:root)
- ✅ Certificate validation after import (subject, validity dates)
- ✅ Certificate information extraction
- ✅ Dynamic chain file testing (only when chain_src is defined)

### Let's Encrypt Tests (`tests/letsencrypt.yml` and `side_effect.yml`)
- ✅ Certbot installation verification
- ✅ Certbot plugin installation validation (apache, nginx, dns-*)
- ✅ Certbot configuration directories (/etc/letsencrypt/*)
- ✅ HTTP-01 challenge tests:
  - ✅ Webroot plugin with directory validation
  - ✅ Apache plugin with Apache installation
- ✅ DNS-01 challenge tests:
  - ✅ Custom hooks with challtestsrv integration
  - ✅ Deploy hook validation
- ✅ ACME server configuration (Pebble)
- ✅ Certificate file structure validation
- ✅ Certificate directory existence (/etc/letsencrypt/live/\<domain\>/)
- ✅ Required certificate files (cert.pem, chain.pem, fullchain.pem, privkey.pem)
- ✅ Private key permissions (0600/0640)
- ✅ Certificate information extraction and validation
- ✅ Domain verification in Subject Alternative Names

## Test Infrastructure

### Pebble ACME Test Server
The test environment includes a [Pebble](https://github.com/letsencrypt/pebble) ACME server for realistic Let's Encrypt testing:
- Runs in separate Docker container
- Listens on port 14000 (ACME API) and 15000 (Management API)
- Pre-configured to accept all validations (`PEBBLE_VA_ALWAYS_VALID=1`)
- No sleep mode for faster testing (`PEBBLE_VA_NOSLEEP=1`)
- Self-signed CA certificate trusted by test instance

### challtestsrv (DNS Challenge Server)
For DNS-01 challenge testing:
- Provides DNS challenge validation via API
- Listens on port 8055 (Management API)
- Used with custom authentication and cleanup hooks
- Supports wildcard certificates

### Test Network
All containers run on the `molecule` Docker network:
- `pebble` - ACME server
- `challtestsrv` - DNS challenge server
- `certificates-instance` - Test target (Ubuntu 24.04)

## Test Scenarios

### Prepare Phase
Prepares test environment and generates test certificates:

1. **Create test files directory** on control node
2. **Generate test certificate** with proper CSR and subject fields:
   - Private key (2048-bit RSA)
   - CSR with subject: `test-import.local`, organization, country, state, locality
   - Self-signed certificate (valid 365 days)
3. **Generate intermediate CA certificate** for chain testing:
   - CA private key (2048-bit RSA)
   - CA CSR with CA extensions (`CA:TRUE`, `keyCertSign`, `cRLSign`)
   - CA certificate (valid 730 days)
4. **Cleanup temporary files** (CSR files, CA private key)
5. **Configure Pebble ACME server trust**:
   - Install ca-certificates package
   - Download Pebble root CA certificate
   - Update CA trust store
   - Wait for Pebble to be ready
6. **Create certbot hook scripts**:
   - Deploy hook script (`/usr/local/bin/deploy.sh`)
   - DNS authentication hook (`/data/certbot/authenticator.sh`)
   - DNS cleanup hook (`/data/certbot/cleanup.sh`)
   - All hooks integrate with challtestsrv API

### Converge Phase
Tests self-signed certificate generation with 4 different certificate types:
1. **Basic** - Minimal configuration (`test-basic.local`)
2. **Advanced** - Full X.509 fields (`test-advanced.local`)
3. **SAN** - Multiple domains and IPs (`test-san.local`)
4. **Multi-domain** - Wildcard support (`test-multidomain.local`)

### Side Effect Phase
Tests multiple operational modes sequentially:

1. **Import Mode** - Imports pre-generated certificates (with chain) and validates:
   - Imports `test-import` certificate from files
   - Imports private key and certificate chain
   - Validates imported certificates exist and have correct permissions
   - Verifies certificate information and validity

2. **Let's Encrypt Mode** - Obtains real certificates from Pebble ACME server:
   - **Apache installation and setup** for HTTP-01 challenges
   - **Webroot directory creation** (`/var/www/html`)
   - **ACME challenge directory** (`/var/www/html/.well-known/acme-challenge`)
   - **HTTP-01 with webroot plugin** (`test-webroot-letsencrypt.example.com` + www subdomain)
   - **HTTP-01 with apache plugin** (`test-apache-letsencrypt.example.com`)
   - **DNS-01 with custom hooks** (`test-dns-letsencrypt.example.com` + wildcard `*.test-dns-letsencrypt.example.com`)
   - Obtains actual certificates from Pebble in `staging` mode

### Verify Phase
Comprehensive verification of all three modes:

1. **Self-Signed Mode Verification**:
   - Verifies `python3-cryptography` installation
   - Verifies certificate and key directories exist with correct permissions
   - Tests all 4 self-signed certificates created in converge phase
   - Validates certificate properties, permissions, and matching keys

2. **Import Mode Verification**:
   - Verifies imported certificate files exist
   - Validates file permissions (cert: 0644, key: 0600, chain: 0644)
   - Verifies certificate ownership (root:root)
   - Validates certificate information and validity dates

3. **Let's Encrypt Configuration Verification** (in `dry-run` mode):
   - Verifies certbot installation
   - Verifies certbot plugin packages (apache, dns-cloudflare)
   - Validates configuration variables (email, ACME server, mode)
   - Verifies certificate list configuration for all 3 certificates
   - Validates webroot directories exist
   - Checks Let's Encrypt directory structure
   - **Note**: Uses `dry-run` mode to validate configuration without checking actual certificate files

## Running Tests

### Full Test Suite
```bash
cd extensions
molecule test -s certificates
```

### Individual Test Phases
```bash
# Create test environment (Pebble + challtestsrv + test instance)
molecule create -s certificates

# Prepare (install dependencies, create test files, configure Pebble)
molecule prepare -s certificates

# Apply role (converge - selfsigned mode)
molecule converge -s certificates

# Run verification tests
molecule verify -s certificates

# Test import mode and Let's Encrypt with Pebble
molecule side-effect -s certificates

# Check idempotency
molecule idempotence -s certificates

# Clean up
molecule destroy -s certificates
```

### Quick Development Iteration
```bash
cd extensions
molecule converge -s certificates && molecule verify -s certificates
```

### Login to Test Container
```bash
molecule login -s certificates
```

## Test Variables

### Converge Phase Variables
Defined in `inventory/hosts.yml` for the converge phase:

```yaml
certificates_mode: selfsigned
certificates_cert_dir: /etc/ssl/certs
certificates_key_dir: /etc/ssl/private
certificates_list:
  - name: test-basic.local
    common_name: test-basic.local
    organization: "Test Organization"
    days: 365
  - name: test-advanced.local
    common_name: test-advanced.local
    organization: "Advanced Test Org"
    organizational_unit: "IT Department"
    country: "US"
    state: "California"
    locality: "San Francisco"
    email: "test@example.com"
    days: 730
    key_size: 4096
  - name: test-san.local
    common_name: test-san.local
    organization: "SAN Test Org"
    days: 365
    san:
      - DNS:www.test-san.local
      - DNS:api.test-san.local
      - DNS:mail.test-san.local
      - IP:192.168.1.100
      - IP:10.0.0.1
  - name: test-multidomain.local
    common_name: test-multidomain.local
    organization: "Multi-Domain Test"
    days: 365
    san:
      - DNS:test-multidomain.local
      - DNS:www.test-multidomain.local
      - DNS:*.test-multidomain.local
```

### Side Effect Phase Variables
Defined inline in `side_effect.yml`:

**Import Mode:**
```yaml
certificates_mode: import
certificates_list:
  - name: test-import
    cert_src: "{{ playbook_dir }}/files/test-import.crt"
    key_src: "{{ playbook_dir }}/files/test-import.key"
    chain_src: "{{ playbook_dir }}/files/test-import-chain.crt"
```

**Let's Encrypt Mode:**
```yaml
certificates_mode: letsencrypt
certificates_certbot_email: "test@example.com"
certificates_certbot_mode: staging
certificates_acme_server: "https://pebble:14000/dir"
certificates_acme_verify_ssl: false
certificates_certbot_dns_provider: cloudflare
certificates_list:
  - name: test-webroot-letsencrypt.example.com
    domains:
      - test-webroot-letsencrypt.example.com
      - www.test-webroot-letsencrypt.example.com
    challenge_type: http-01
    plugin: webroot
    webroot: /var/www/html
  - name: test-apache-letsencrypt.example.com
    domains:
      - test-apache-letsencrypt.example.com
    challenge_type: http-01
    plugin: apache
  - name: test-dns-letsencrypt.example.com
    domains:
      - test-dns-letsencrypt.example.com
      - "*.test-dns-letsencrypt.example.com"
    challenge_type: dns-01
    provider: custom
    auth_hook: "/data/certbot/authenticator.sh"
    cleanup_hook: "/data/certbot/cleanup.sh"
    deploy_hook: "/usr/local/bin/deploy.sh"
```

### Verify Phase Variables
**Important:** The verify phase defines its own variables inline for each test mode to ensure accurate verification against the actual state created by converge and side_effect phases. It tests all three modes sequentially:

1. **Self-Signed Mode**: Uses the same certificate list from converge phase
2. **Import Mode**: Uses the import certificate configuration from side_effect phase
3. **Let's Encrypt Mode**: Uses `dry-run` mode to validate configuration without checking actual certificate files

## Expected Results

### Selfsigned Mode (Converge Phase)
- 4 certificates generated in `/etc/ssl/certs/`
- 4 private keys generated in `/etc/ssl/private/`
- All files have correct permissions and ownership
- All certificates are valid and properly formatted
- CSR files are cleaned up after generation
- Idempotent (no changes on second run)

### Import Mode (Side Effect Phase)
- Certificate copied to `/etc/ssl/certs/test-import.crt`
- Private key copied to `/etc/ssl/private/test-import.key`
- Chain certificate copied to `/etc/ssl/certs/test-import-chain.crt`
- All files have correct permissions (cert: 0644, key: 0600, chain: 0644)
- All files have correct ownership (root:root)
- Certificate is valid with proper subject information (commonName, organization, etc.)
- Certificate validity dates are correct (not expired)

### Let's Encrypt Mode (Side Effect Phase)
- Apache2 installed and running
- Webroot directory created with proper permissions
- Certbot and required plugins installed
- ACME account registered with Pebble
- 3 certificates obtained from Pebble ACME server:
  - `test-webroot-letsencrypt.example.com` (HTTP-01 webroot)
  - `test-apache-letsencrypt.example.com` (HTTP-01 apache)
  - `test-dns-letsencrypt.example.com` (DNS-01 custom hooks + wildcard)
- Certificate directories created in `/etc/letsencrypt/live/<domain>/`
- All required certificate files present (cert.pem, chain.pem, fullchain.pem, privkey.pem)
- Private keys have secure permissions (0600)
- Certificates contain expected domains in SAN

## Troubleshooting

### View Test Output
```bash
molecule --debug converge -s certificates
```

### Check Certificate Details
```bash
molecule login -s certificates
openssl x509 -in /etc/ssl/certs/test-basic.local.crt -text -noout
```

### Verify Private Key
```bash
molecule login -s certificates
openssl rsa -in /etc/ssl/private/test-basic.local.key -check
```

### Check Imported Certificate Chain
```bash
molecule login -s certificates
# View imported certificate
openssl x509 -in /etc/ssl/certs/test-import.crt -text -noout

# View imported chain certificate
openssl x509 -in /etc/ssl/certs/test-import-chain.crt -text -noout

# Verify chain certificate is a CA
openssl x509 -in /etc/ssl/certs/test-import-chain.crt -text -noout | grep "CA:TRUE"
```

### Check Let's Encrypt Certificates
```bash
molecule login -s certificates
# List certificates
certbot certificates

# View certificate details
openssl x509 -in /etc/letsencrypt/live/test-webroot-letsencrypt.example.com/cert.pem -text -noout

# Check certificate files
ls -la /etc/letsencrypt/live/test-webroot-letsencrypt.example.com/
```

### Check Pebble ACME Server
```bash
# Check if Pebble is running
docker ps | grep pebble

# Check Pebble logs
docker logs pebble

# Test Pebble API
curl -k https://localhost:14000/dir
```

### Check challtestsrv
```bash
# Check if challtestsrv is running
docker ps | grep challtestsrv

# Check challtestsrv logs
docker logs challtestsrv
```

## Test Files

### Main Playbooks
- `molecule.yml` - Molecule configuration with Pebble, challtestsrv, and test instance platforms
- `prepare.yml` - Prepares test environment, generates test certificates, configures Pebble trust, creates hooks
- `converge.yml` - Applies the certificates role in selfsigned mode (4 test certificates)
- `side_effect.yml` - Tests import mode and Let's Encrypt mode with Pebble ACME server
- `verify.yml` - Orchestrates all verification tests for all three modes (selfsigned, import, letsencrypt) with inline variables

### Test Suite Files
- `tests/install.yml` - Installation and setup verification (packages, directories, permissions)
- `tests/selfsigned.yml` - Self-signed certificate generation validation (files, permissions, certificate properties, key matching)
- `tests/import.yml` - Certificate import mode validation (file copying, permissions, chain files)
- `tests/letsencrypt.yml` - Let's Encrypt/certbot configuration validation (plugins, directories, configuration)

## Platform

- **OS**: Ubuntu 24.04 (geerlingguy/docker-ubuntu2404-ansible)
- **Driver**: Docker
- **Python**: 3.x
- **Required Collections**: community.crypto
- **Test Servers**:
  - Pebble ACME server (ghcr.io/letsencrypt/pebble:latest)
  - challtestsrv (ghcr.io/letsencrypt/pebble-challtestsrv:latest)

## Notes

### Let's Encrypt Testing with Pebble
- Uses Pebble ACME test server (not production Let's Encrypt)
- Obtains real certificates from Pebble
- Pebble configured to accept all validations automatically
- SSL verification disabled for Pebble's self-signed CA
- Tests all challenge types (HTTP-01 webroot, HTTP-01 apache, DNS-01 custom)
- Suitable for CI/CD pipelines without rate limits
- No risk of hitting Let's Encrypt rate limits

### Idempotence
- Self-signed certificate generation is idempotent
- CSRs only generated when certificates don't exist
- CSR cleanup doesn't report as changed
- Import mode is idempotent (files only copied if changed)
- Let's Encrypt mode uses certbot's built-in idempotence (--keep-until-expiring)

## CI/CD Integration

This test scenario is suitable for CI/CD pipelines:

```yaml
# GitHub Actions example
- name: Run Molecule tests
  run: |
    cd extensions
    molecule test -s certificates
```

All tests run in containerized environments with Pebble ACME server for realistic Let's Encrypt testing without external dependencies or rate limits.
