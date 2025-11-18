# Molecule Test Scenario: Certificates Role - Standalone/Nginx Plugins

This Molecule scenario tests the `jlira.web_server.certificates` role with HTTP-01 challenge plugins (standalone and nginx) using the Pebble ACME test server.

## Test Coverage

### HTTP-01 Standalone Plugin Tests
- ✅ Certbot standalone plugin installation
- ✅ Certificate obtainment with standalone plugin (temporary web server)
- ✅ Multi-domain certificate support (primary domain + www subdomain)
- ✅ Certificate file structure in /etc/letsencrypt/live/
- ✅ Certificate file existence (fullchain.pem, privkey.pem, chain.pem)
- ✅ Private key permissions verification (0600/0640)
- ✅ Certificate information extraction

### HTTP-01 Nginx Plugin Tests
- ✅ Certbot nginx plugin installation (python3-certbot-nginx)
- ✅ Nginx installation and configuration
- ✅ Certificate obtainment with nginx plugin
- ✅ Multi-domain certificate support (primary domain + www subdomain)
- ✅ Certificate file structure in /etc/letsencrypt/live/
- ✅ Certificate file existence (fullchain.pem, privkey.pem, chain.pem)
- ✅ Private key permissions verification (0600/0640)
- ✅ Certificate information extraction

## Test Infrastructure

### Pebble ACME Test Server
The test environment includes a [Pebble](https://github.com/letsencrypt/pebble) ACME server for realistic Let's Encrypt testing:
- Runs in separate Docker container
- Listens on port 14000 (ACME API) and 15000 (Management API)
- Pre-configured to accept all validations (`PEBBLE_VA_ALWAYS_VALID=1`)
- No sleep mode for faster testing (`PEBBLE_VA_NOSLEEP=1`)
- Self-signed CA certificate trusted by test instance

### Test Network
All containers run on the `molecule` Docker network:
- `pebble` - ACME server
- `certificates_standalone-instance` - Test target (Ubuntu 24.04)

## Test Scenarios

### Prepare Phase
Prepares test environment:

1. **Update package cache** for apt
2. **Configure Pebble ACME Server Trust**:
   - Install ca-certificates package
   - Download Pebble root CA certificate
   - Update CA trust store
   - Wait for Pebble to be ready
3. **Install nginx** for nginx plugin testing
4. **Stop nginx** (standalone plugin needs port 80 available)
5. **Create dummy hook scripts** for testing deploy hooks

### Converge Phase
Tests Let's Encrypt certificate obtainment with two HTTP-01 plugins:

1. **Standalone Plugin** (`test-standalone-letsencrypt.example.com`):
   - Domains: `test-standalone-letsencrypt.example.com`, `www.test-standalone-letsencrypt.example.com`
   - Plugin: standalone (certbot runs temporary web server)
   - Challenge: HTTP-01

2. **Nginx Plugin** (`test-nginx-letsencrypt.example.com`):
   - Domains: `test-nginx-letsencrypt.example.com`, `www.test-nginx-letsencrypt.example.com`
   - Plugin: nginx (certbot integrates with nginx)
   - Challenge: HTTP-01

### Verify Phase
Comprehensive verification of both plugins:

1. **Standalone Certificate Verification**:
   - Verifies certificate files exist (fullchain.pem, privkey.pem, chain.pem)
   - Validates private key permissions (0600 or 0640)
   - Extracts and displays certificate information

2. **Nginx Certificate Verification**:
   - Verifies certificate files exist (fullchain.pem, privkey.pem, chain.pem)
   - Validates private key permissions (0600 or 0640)
   - Extracts and displays certificate information

## Running Tests

### Full Test Suite
```bash
cd extensions
molecule test -s certificates_standalone
```

### Individual Test Phases
```bash
# Create test environment (Pebble + test instance)
molecule create -s certificates_standalone

# Prepare (install dependencies, configure Pebble)
molecule prepare -s certificates_standalone

# Apply role (obtain certificates)
molecule converge -s certificates_standalone

# Run verification tests
molecule verify -s certificates_standalone

# Check idempotency
molecule idempotence -s certificates_standalone

# Clean up
molecule destroy -s certificates_standalone
```

### Quick Development Iteration
```bash
cd extensions
molecule converge -s certificates_standalone && molecule verify -s certificates_standalone
```

### Login to Test Container
```bash
molecule login -s certificates_standalone
```

## Test Variables

All test variables are defined in `inventory/hosts.yml`:

```yaml
certificates_mode: letsencrypt
certificates_certbot_email: "test@example.com"
certificates_certbot_mode: staging
certificates_acme_server: "https://pebble:14000/dir"
certificates_acme_verify_ssl: false

certificates_list:
  # HTTP-01 with standalone plugin
  - name: test-standalone-letsencrypt.example.com
    domains:
      - test-standalone-letsencrypt.example.com
      - www.test-standalone-letsencrypt.example.com
    challenge_type: http-01
    plugin: standalone

  # HTTP-01 with nginx plugin
  - name: test-nginx-letsencrypt.example.com
    domains:
      - test-nginx-letsencrypt.example.com
      - www.test-nginx-letsencrypt.example.com
    challenge_type: http-01
    plugin: nginx
```

## Expected Results

### After Converge Phase

**Standalone Certificate:**
- Certificate directory created: `/etc/letsencrypt/live/test-standalone-letsencrypt.example.com/`
- Certificate files present:
  - `fullchain.pem` - Full certificate chain
  - `privkey.pem` - Private key (0600 permissions)
  - `chain.pem` - Certificate chain only
  - `cert.pem` - Certificate only
- Certificate valid for both domains:
  - `test-standalone-letsencrypt.example.com`
  - `www.test-standalone-letsencrypt.example.com`

**Nginx Certificate:**
- Certificate directory created: `/etc/letsencrypt/live/test-nginx-letsencrypt.example.com/`
- Certificate files present:
  - `fullchain.pem` - Full certificate chain
  - `privkey.pem` - Private key (0600 permissions)
  - `chain.pem` - Certificate chain only
  - `cert.pem` - Certificate only
- Certificate valid for both domains:
  - `test-nginx-letsencrypt.example.com`
  - `www.test-nginx-letsencrypt.example.com`
- Nginx plugin package installed: `python3-certbot-nginx`

## Troubleshooting

### View Test Output
```bash
molecule --debug converge -s certificates_standalone
```

### Check Standalone Certificate Details
```bash
molecule login -s certificates_standalone
# View certificate
openssl x509 -in /etc/letsencrypt/live/test-standalone-letsencrypt.example.com/cert.pem -text -noout

# List all certificates
certbot certificates
```

### Check Nginx Certificate Details
```bash
molecule login -s certificates_standalone
# View certificate
openssl x509 -in /etc/letsencrypt/live/test-nginx-letsencrypt.example.com/cert.pem -text -noout

# Check certificate files
ls -la /etc/letsencrypt/live/test-nginx-letsencrypt.example.com/
```

### Verify Certificate Domains
```bash
molecule login -s certificates_standalone
# Check Subject Alternative Names
openssl x509 -in /etc/letsencrypt/live/test-standalone-letsencrypt.example.com/cert.pem -noout -text | grep -A1 "Subject Alternative Name"
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

### Check Certbot Logs
```bash
molecule login -s certificates_standalone
# View recent certbot operations
tail -f /var/log/letsencrypt/letsencrypt.log

# Search for specific domain
grep "test-standalone" /var/log/letsencrypt/letsencrypt.log
```

## Test Files

### Main Playbooks
- `molecule.yml` - Molecule configuration with Pebble ACME server and test instance
- `prepare.yml` - Prepares test environment, configures Pebble trust, installs nginx
- `converge.yml` - Applies the certificates role with standalone and nginx plugins
- `verify.yml` - Verifies both certificates were obtained correctly with proper permissions

### Inventory
- `inventory/hosts.yml` - Test variables defining the two certificates to obtain

## Platform

- **OS**: Ubuntu 24.04 (geerlingguy/docker-ubuntu2404-ansible)
- **Driver**: Docker
- **Python**: 3.x
- **Required Collections**: community.crypto
- **Test Servers**: Pebble ACME server (ghcr.io/letsencrypt/pebble:latest)

## Notes

### Plugin Behavior

**Standalone Plugin:**
- Requires port 80 to be available (no web server running)
- Certbot starts a temporary web server to respond to challenges
- Ideal for servers without a permanent web server
- Cannot be used while a web server is running on port 80

**Nginx Plugin:**
- Requires nginx to be installed
- Certbot automatically configures nginx for the challenge
- Can work with a running nginx server
- Automatically installs `python3-certbot-nginx` package

### Certificate Expansion

The role automatically uses the `--expand` flag when adding domains to existing certificates. This allows:
- Adding new domains to an existing certificate
- Modifying the domain list without interactive prompts
- Idempotent behavior when running multiple times

### Let's Encrypt Testing with Pebble

- Uses Pebble ACME test server (not production Let's Encrypt)
- Obtains real certificates from Pebble
- Pebble configured to accept all validations automatically
- SSL verification disabled for Pebble's self-signed CA
- Tests both standalone and nginx plugins
- Suitable for CI/CD pipelines without rate limits
- No risk of hitting Let's Encrypt rate limits

### Idempotence

- Certificate obtainment is idempotent (uses `--keep-until-expiring`)
- Running converge multiple times won't re-obtain valid certificates
- Certificate expansion is automatic with `--expand` flag
- No manual intervention required

## CI/CD Integration

This test scenario is suitable for CI/CD pipelines:

```yaml
# GitHub Actions example
- name: Run Molecule tests for standalone/nginx plugins
  run: |
    cd extensions
    molecule test -s certificates_standalone
```

All tests run in containerized environments with Pebble ACME server for realistic Let's Encrypt testing without external dependencies or rate limits.

## Comparison with Other Scenarios

### certificates_standalone vs certificates
- **certificates_standalone**: Tests HTTP-01 standalone and nginx plugins only
- **certificates**: Comprehensive testing of all modes (selfsigned, import, letsencrypt with webroot, apache, DNS-01)

### When to Use This Scenario
- Testing standalone plugin functionality
- Testing nginx plugin functionality
- Quick HTTP-01 validation without full infrastructure
- CI/CD pipelines focused on standalone/nginx deployments

### When to Use Main certificates Scenario
- Testing self-signed certificates
- Testing certificate import
- Testing Let's Encrypt with Apache
- Testing DNS-01 challenges
- Comprehensive end-to-end testing