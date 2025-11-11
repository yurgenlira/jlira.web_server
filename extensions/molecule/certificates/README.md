# Molecule Test Scenario: Certificates Role

This comprehensive Molecule scenario tests all features of the `jlira.web_server.certificates` role including self-signed certificates, import mode, Let's Encrypt configuration validation, certificate expiration monitoring, and automatic renewal setup.

## Test Coverage

### Installation Tests (`tests/install.yml`)
- ✅ python3-cryptography package installation
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
- ✅ CSR cleanup

### Import Mode Tests (`tests/import.yml` and `side_effect.yml`)
- ✅ Certificate import from source files
- ✅ Private key import from source files
- ✅ Imported file permissions
- ✅ Certificate validation after import
- ✅ Certificate information extraction

### Let's Encrypt Tests (`tests/letsencrypt.yml` and `side_effect.yml`)
- ✅ ACME account directory creation (0700 permissions)
- ✅ ACME account key generation (4096-bit RSA, 0600 permissions)
- ✅ ACME account key validation
- ✅ HTTP-01 webroot directory structure (.well-known/acme-challenge)
- ✅ ACME configuration validation (email, TOS agreement, challenge type)
- ✅ Domains list validation
- ✅ DNS-01 DNS provider configuration check
- ✅ Certificate file structure validation
- Note: Does not connect to actual Let's Encrypt servers (validation only)

### Certificate Expiration Monitoring Tests (`tests/monitoring.yml` and `side_effect.yml`)
- ✅ Certificate expiration date retrieval
- ✅ Days-until-expiry calculation
- ✅ Certificate validity status checking
- ✅ Multiple certificate monitoring
- ✅ Warning and critical threshold configuration
- ✅ Expired certificate detection

### Automatic Renewal Tests (`tests/renewal.yml`)
- ✅ Systemd availability check
- ✅ cert-renew.service file creation (0644 permissions)
- ✅ cert-renew.timer file creation (0644 permissions)
- ✅ Service file content validation (Unit, Service, Install sections)
- ✅ Timer file content validation (OnCalendar, Persistent settings)
- ✅ Timer enabled state verification
- ✅ Timer status and next run display

## Test Scenarios

### Converge Phase
Tests self-signed certificate generation with 4 different certificate types:
1. **Basic** - Minimal configuration
2. **Advanced** - Full X.509 fields
3. **SAN** - Multiple domains and IPs
4. **Multi-domain** - Wildcard support

### Side Effect Phase
Tests multiple operational modes:
1. **Import Mode** - Imports pre-generated certificates and validates
2. **Certificate Expiration Monitoring** - Enables monitoring on existing certificates
3. **Let's Encrypt Validation** - Validates Let's Encrypt configuration (no actual ACME connection)

## Running Tests

### Full Test Suite
```bash
cd extensions
molecule test -s certificates
```

### Individual Test Phases
```bash
# Create test environment
molecule create -s certificates

# Prepare (install dependencies, create test files)
molecule prepare -s certificates

# Apply role (converge)
molecule converge -s certificates

# Run verification tests
molecule verify -s certificates

# Test import mode
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

All test variables are defined in `inventory/hosts.yml`:

```yaml
certificates_mode: selfsigned
certificates_list:
  - name: test-basic.local
    common_name: test-basic.local
    organization: "Test Organization"
    days: 365
  # ... more certificates
```

## Expected Results

### Selfsigned Mode (Converge Phase)
- 4 certificates generated in `/etc/ssl/certs/`
- 4 private keys generated in `/etc/ssl/private/`
- All files have correct permissions and ownership
- All certificates are valid and properly formatted
- CSR files are cleaned up after generation

### Import Mode (Side Effect Phase)
- Certificate copied to `/etc/ssl/certs/test-import.crt`
- Private key copied to `/etc/ssl/private/test-import.key`
- Files have correct permissions
- Certificate is valid and readable

### Certificate Expiration Monitoring (Side Effect Phase)
- Certificate expiration dates retrieved successfully
- Days-until-expiry calculated correctly
- All test certificates reported as valid
- Monitoring summary displayed

### Let's Encrypt Validation (Side Effect Phase)
- ACME account directory created: `/etc/letsencrypt/accounts/`
- ACME account key generated with correct permissions
- Configuration validated (email, TOS, challenge type)
- No actual connection to Let's Encrypt servers

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

## Test Files

- `tests/install.yml` - Installation and setup tests
- `tests/selfsigned.yml` - Self-signed certificate generation tests
- `tests/import.yml` - Certificate import mode tests
- `tests/letsencrypt.yml` - Let's Encrypt configuration validation tests
- `tests/monitoring.yml` - Certificate expiration monitoring tests
- `tests/renewal.yml` - Automatic renewal timer tests

## Platform

- **OS**: Ubuntu 24.04 (geerlingguy/docker-ubuntu2404-ansible)
- **Driver**: Docker
- **Python**: 3.x
- **Required Collections**: community.crypto
- **Systemd**: Enabled (required for renewal timer tests)

## Notes

### Let's Encrypt Testing Limitations
- Tests validate configuration only
- Does not connect to actual Let's Encrypt ACME servers
- Does not obtain real certificates
- Suitable for CI/CD pipelines without external dependencies

### Renewal Testing Limitations
- Tests verify systemd timer creation and configuration
- Does not actually trigger renewal (would require real Let's Encrypt certificates)
- Timer schedule can be verified but not executed in test environment

## CI/CD Integration

This test scenario is suitable for CI/CD pipelines:

```yaml
# GitHub Actions example
- name: Run Molecule tests
  run: |
    cd extensions
    molecule test -s certificates
```

All tests can run in containerized environments without external dependencies.