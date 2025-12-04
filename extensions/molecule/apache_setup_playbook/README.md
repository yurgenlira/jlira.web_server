# Apache Setup Playbook Molecule Testing Scenario

This Molecule scenario tests the `apache_setup.yml` playbook, which demonstrates a multi-phase execution pattern for setting up a complete web server with Apache, PHP-FPM, and SSL certificates.

## What This Tests

The playbook solves the "chicken-and-egg" problem of Let's Encrypt HTTP-01 challenges by using a multi-phase execution strategy with built-in idempotence:

1. **Phase 0 (PHP)**: Install and configure PHP-FPM
2. **Phase 1 (HTTP)**: Configure Apache with HTTP-only virtual hosts (Skipped if certificates already exist)
3. **Phase 2 (Certificates)**: Generate SSL certificates (self-signed in tests)
4. **Phase 3 (HTTPS)**: Re-configure Apache with full HTTP/HTTPS virtual hosts using the new certs

## Test Flow

### inventory/hosts.yml
Runs the playbook with:
- PHP 8.3 with FPM enabled
- Apache with execution phase support
- Self-signed certificate generation (instead of Let's Encrypt)
- Test virtual host: `secure.example.local`

### verify.yml
Validates each phase:

#### Phase 0: PHP Installation
- ✅ PHP-FPM service is running
- ✅ PHP 8.3 is installed

#### Phase 1: HTTP Configuration
- ✅ HTTP virtual host configuration exists
- ✅ HTTP site is enabled
- ✅ Apache is listening on port 80

#### Phase 2: Certificate Generation
- ✅ Certificate file exists
- ✅ Private key file exists
- ✅ Certificate is valid for the domain

#### Phase 3: HTTPS Configuration
- ✅ HTTPS virtual host configuration exists
- ✅ HTTPS site is enabled
- ✅ Configuration references correct certificates
- ✅ Apache is listening on port 443

#### Integration
- ✅ Apache service is running
- ✅ All services are healthy

## Running the Tests

### Full Test Suite
```bash
cd extensions
molecule test -s apache_setup_playbook
```

### Development Workflow
```bash
# Create instance
molecule create -s apache_setup_playbook

# Run playbook
molecule converge -s apache_setup_playbook

# Verify all phases
molecule verify -s apache_setup_playbook

# Clean up
molecule destroy -s apache_setup_playbook
```

### Quick Iteration
```bash
molecule converge -s apache_setup_playbook && molecule verify -s apache_setup_playbook
```

## CI/CD Integration

The scenario is integrated into the CI/CD pipeline.

**Triggers**:
- Pull requests that modify:
  - `playbooks/apache_setup.yml`
  - Any role used in orchestration (apache, php, certificates)
  - The scenario itself
- Pushes to `main` branch
- Manual workflow dispatch

## Differences from Production

### Certificates
- **Test**: Uses self-signed certificates
- **Production**: Would use Let's Encrypt with HTTP-01 or DNS-01 challenge

### Variables
The test uses simplified configuration:
```yaml
certificate_generation: true
certificates_selfsigned_validity_days: 3650
```

In production, you would use:
```yaml
certificates_certbot_environment: "production"
certificates_certbot_email: "admin@example.com"
```

## Platform

Tests run on Ubuntu 24.04 using Docker with systemd support.

**Container Image**: `geerlingguy/docker-ubuntu2404-ansible`

**Requirements**:
- Docker with cgroups v2 support
- Systemd-compatible container
- Privileged mode enabled

## Troubleshooting

### Test Failures

**Check logs**:
```bash
molecule verify -s apache_setup_playbook -- -vvv
```

**Login to container**:
```bash
molecule login -s apache_setup_playbook
```

**Check services**:
```bash
molecule login -s apache_setup_playbook
$ systemctl status apache2
$ systemctl status php8.3-fpm
$ ls -la /etc/ssl/certs/secure.example.local.crt
$ ls -la /etc/apache2/sites-enabled/
```

### Common Issues

**Certificate not found**:
- Check if certificates role ran successfully
- Verify certificate paths in inventory match verify.yml

**HTTPS site not enabled**:
- Check if Phase 3 (https_only) executed
- Verify `apache_execution_phase` is set correctly

**Port conflicts**:
- Ensure no other containers are using ports 80/443
- Run `molecule destroy -s apache_setup_playbook` to clean up

## Extending Tests

### Test Different Certificate Types

Modify `inventory/hosts.yml` to test with different certificate configurations:

```yaml
# Test with imported certificates
certificates_items:
  - name: test.example.local
    mode: imported
    src_cert: /path/to/cert.crt
    src_key: /path/to/key.key
```

### Test Multiple Virtual Hosts

Add more virtual hosts to `apache_virtual_hosts` in `inventory/hosts.yml`:

```yaml
apache_virtual_hosts:
  - name: site1.example.local
    # ...
  - name: site2.example.local
    # ...
```

### Test Different PHP Versions

Change `php_version` in `inventory/hosts.yml`:

```yaml
php_version: "8.3"
```

## Related Documentation

- [Apache Setup Playbook](../../../playbooks/apache_setup.yml)
- [Apache Role README](../../../roles/apache/README.md)
- [PHP Role README](../../../roles/php/README.md)
- [Certificates Role README](../../../roles/certificates/README.md)
- [Molecule Documentation](https://molecule.readthedocs.io/)
