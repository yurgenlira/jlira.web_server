# Molecule Tests for Certificates Role

This directory contains comprehensive Molecule tests for the `certificates` role, including self-signed certificates, Let's Encrypt (via Pebble ACME server), and certificate import functionality.

## Structure

- `molecule.yml` - Molecule configuration with Pebble ACME server and challtestsrv
- `converge.yml` - Playbook that applies the certificates role
- `prepare.yml` - Sets up test environment (Pebble CA trust, DNS hooks, import test certificates)
- `verify.yml` - Main verification playbook that includes all test suites
- `inventory/hosts.yml` - Inventory configuration with test certificate definitions
- `tests/` - Individual test files for each role component:
  - `setup.yml` - Tests for basic setup (certbot installation, timer)
  - `selfsigned.yml` - Tests for self-signed certificate generation
  - `import.yml` - Tests for certificate import functionality
  - `letsencrypt.yml` - Tests for Let's Encrypt certificate issuance
  - `validation.yml` - Tests for input validation

## Test Infrastructure

### Containers

1. **certificates-instance** (geerlingguy/docker-ubuntu2404-ansible)
   - Main test target where the role is applied
   - Runs systemd for certbot timer testing
   - Connected to molecule-certificates network

2. **pebble** (ghcr.io/letsencrypt/pebble:latest)
   - Let's Encrypt-compatible ACME test server
   - ACME API: https://pebble:14000/dir
   - Management API: https://pebble:15000
   - Environment: `PEBBLE_VA_ALWAYS_VALID=1` (accepts all challenges)

3. **challtestsrv** (ghcr.io/letsencrypt/pebble-challtestsrv:latest)
   - DNS and HTTP challenge validation server
   - DNS Server: Port 8053
   - Management API: http://challtestsrv:8055
   - Used for DNS-01 challenge testing

### Network
- **Name**: molecule-certificates
- All containers connected via Docker bridge network
- DNS resolution handled by challtestsrv

## Use Cases Covered

### 1. Self-Signed Certificates

- **Use Case 1.1**: Auto-detection for local TLDs (`.test`, `.local`)
  - Certificate: `autodetect.test`
  - Mode auto-detected as `selfsigned`

- **Use Case 1.2**: Explicit self-signed with custom configuration
  - Certificate: `custom.local`
  - Custom: 2048-bit key, 730 days validity, custom SANs
  - Fields: Country, State, Locality, Organization, OU, Email

- **Use Case 1.3**: Global defaults application
  - Certificate: `defaults.test`
  - Uses role default values

- **Use Case 1.4**: Subject Alternative Names (SANs)
  - Certificate: `sans.test`
  - Multiple DNS and IP SANs

### 2. Let's Encrypt Certificates (via Pebble)

- **Use Case 2.1**: Auto-detection for public TLDs
  - Certificate: `autodetect.com`
  - Mode auto-detected as `letsencrypt`
  - Challenge: HTTP-01

- **Use Case 2.2**: HTTP-01 challenge with webroot
  - Certificate: `http01.example.com`
  - Webroot: `/var/www/html`
  - SAN: `www.http01.example.com`

- **Use Case 2.3**: DNS-01 challenge with custom hooks
  - Certificate: `dns01.example.com`
  - Custom authentication and cleanup hooks
  - Wildcard SAN: `*.dns01.example.com`

### 3. Certificate Import

- **Use Case 1.5**: Import existing certificates
  - Certificate: `imported.test`
  - Imports: Certificate, private key, and chain file
  - Source: `/tmp/import-certs/` (created in prepare phase)
  - Destination: `/etc/ssl/certs/` and `/etc/ssl/private/`
  - Validates: File permissions, ownership, and certificate/key matching

## Running Tests

### Full Test Suite
```bash
# Run complete test sequence (destroy, create, prepare, converge, idempotence, verify, destroy)
cd extensions
molecule test -s certificates
```

### Individual Stages
```bash
# Create test infrastructure
molecule create -s certificates

# Prepare environment (install Pebble CA, create hooks, generate import test certs)
molecule prepare -s certificates

# Apply the role
molecule converge -s certificates

# Run verification tests
molecule verify -s certificates

# Clean up
molecule destroy -s certificates
```

### Development Workflow
```bash
# Quick iteration during development
molecule converge -s certificates  # Apply changes
molecule verify -s certificates    # Run tests

# Check idempotency
molecule converge -s certificates  # First run
molecule converge -s certificates  # Second run - should show 0 changes

# Or use the dedicated command
molecule idempotence -s certificates
```

### Debugging
```bash
# Run with verbose output
molecule --debug converge -s certificates

# Login to test container
molecule login -s certificates --host certificates-instance

# View logs
docker logs pebble
docker logs challtestsrv
docker logs certificates-instance
```

## Test Validation

### What Gets Tested

1. **Setup Tests** (`tests/setup.yml`)
   - Certbot is installed
   - Certbot timer is enabled and running

2. **Self-Signed Tests** (`tests/selfsigned.yml`)
   - Certificates exist in `/etc/ssl/certs/`
   - Private keys exist in `/etc/ssl/private/` with mode 0600
   - Certificate details match configuration (CN, O, validity, key size)
   - SANs are correctly included
   - Certificates are valid and not expired

3. **Import Tests** (`tests/import.yml`)
   - Certificate file imported with correct permissions (0644)
   - Private key imported with correct permissions (0600)
   - Chain file imported with correct permissions (0644)
   - Certificate contains expected subject details
   - Certificate and private key match (modulus comparison)

4. **Let's Encrypt Tests** (`tests/letsencrypt.yml`)
   - Certificates exist in `/etc/letsencrypt/live/`
   - Certificates contain correct SANs
   - Certificates are issued by Pebble
   - Wildcard certificates work with DNS-01 challenge

## Prepare Phase Details

The prepare playbook sets up the test environment:

1. **Pebble CA Trust**
   - Downloads Pebble root CA certificate
   - Adds to system trust store
   - Waits for Pebble to be ready

2. **DNS Hook Scripts**
   - Creates `/usr/local/bin/authenticator.sh` - Adds TXT records via challtestsrv API
   - Creates `/usr/local/bin/cleanup.sh` - Removes TXT records via challtestsrv API
   - Creates `/usr/local/bin/deploy.sh` - Dummy deploy hook for testing

3. **Import Test Certificates**
   - Creates `/tmp/import-certs/` directory
   - Generates test private key (2048-bit RSA)
   - Generates CSR with proper subject
   - Creates self-signed certificate
   - Creates dummy chain file

## Troubleshooting

### Pebble Connection Issues
```bash
# Check if Pebble is accessible
docker exec certificates-instance curl -k https://pebble:14000/dir

# View Pebble logs
docker logs pebble

# Check Pebble health
docker exec certificates-instance wget -q -O - https://pebble:15000/dir
```

### DNS Challenge Issues
```bash
# Check challtestsrv is running
docker logs challtestsrv

# Test DNS resolution
docker exec certificates-instance dig @challtestsrv _acme-challenge.example.com TXT

# Manually test DNS hook
docker exec certificates-instance /usr/local/bin/authenticator.sh
```

### Certificate Verification
```bash
# View self-signed certificate
docker exec certificates-instance openssl x509 -in /etc/ssl/certs/autodetect.test.crt -noout -text

# View Let's Encrypt certificate
docker exec certificates-instance openssl x509 -in /etc/letsencrypt/live/autodetect.com/cert.pem -noout -text

# Check certificate issuer
docker exec certificates-instance openssl x509 -in /etc/letsencrypt/live/autodetect.com/cert.pem -noout -issuer

# Verify imported certificate and key match
docker exec certificates-instance bash -c "openssl x509 -noout -modulus -in /etc/ssl/certs/imported.test.crt | openssl md5"
docker exec certificates-instance bash -c "openssl rsa -noout -modulus -in /etc/ssl/private/imported.test.key | openssl md5"
```

### Common Issues

1. **"Failed to create temporary directory" errors on pebble/challtestsrv**
   - These containers don't have shells - this is expected
   - Ensure `converge.yml` and `verify.yml` target only `certificates-instance`

2. **Pebble certificate validation failures**
   - Ensure Pebble CA is trusted (check prepare phase)
   - Verify `--no-verify-ssl` flag is used in test environment

3. **DNS-01 challenges timing out**
   - Check challtestsrv is running and accessible
   - Verify DNS hooks are executable
   - Check hook scripts can reach challtestsrv API

4. **Import tests failing**
   - Ensure prepare phase completed successfully
   - Check `/tmp/import-certs/` directory exists with test certificates
   - Verify file permissions on source certificates

## Linting

All test files pass ansible-lint with production profile:

```bash
# Lint the role
ansible-lint roles/certificates/

# Lint the molecule tests
ansible-lint extensions/molecule/certificates/
```

## Idempotency

The role is designed to be idempotent. Running `molecule converge` twice should result in 0 changes on the second run:

```bash
molecule converge -s certificates  # First run - makes changes
molecule converge -s certificates  # Second run - 0 changes (idempotent)
```

Or use the dedicated idempotence test:
```bash
molecule idempotence -s certificates
```

## CI/CD Integration

This test suite can be integrated into CI/CD pipelines:

```yaml
# Example GitHub Actions workflow
- name: Run Molecule tests
  run: |
    cd extensions
    molecule test -s certificates
```

## References

- [Pebble ACME Server](https://github.com/letsencrypt/pebble)
- [challtestsrv Documentation](https://github.com/letsencrypt/pebble/tree/main/cmd/pebble-challtestsrv)
- [Molecule Documentation](https://molecule.readthedocs.io/)
- [Let's Encrypt ACME Protocol](https://letsencrypt.org/docs/client-options/)
