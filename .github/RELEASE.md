# Release Process Guide

This document describes the automated release workflows for the `jlira.web_server` Ansible collection.

## Prerequisites

Before using either release workflow, you need to set up your Ansible Galaxy API token:

1. Log in to [Ansible Galaxy](https://galaxy.ansible.com/)
2. Go to your profile → Preferences → API Key
3. Copy your API token
4. Add it to your GitHub repository secrets:
   - Go to Settings → Secrets and variables → Actions
   - Create a new secret named `GALAXY_API_KEY`
   - Paste your API token

## Release Methods

### Option 1: Semi-Automated Release (Tag-Triggered)

**Best for**: Planned releases where you want full control over versioning and changelog.

#### Process:

1. **Manually update CHANGELOG.md**:
   ```bash
   # Edit CHANGELOG.md to move items from [Unreleased] to a new version section
   ## [0.0.2] - 2025-10-23
   ### Added
   - New feature X
   ### Fixed
   - Bug Y
   ```

2. **Commit the changelog**:
   ```bash
   git add CHANGELOG.md
   git commit -m "docs: update changelog for v0.0.2"
   git push
   ```

3. **Create and push a tag**:
   ```bash
   git tag v0.0.2
   git push origin v0.0.2
   ```

4. **Automatic steps** (handled by `.github/workflows/release.yml`):
   - ✅ Updates `galaxy.yml` version
   - ✅ Builds the collection tarball
   - ✅ Publishes to Ansible Galaxy
   - ✅ Creates GitHub Release with changelog notes
   - ✅ Attaches collection tarball to release

#### Workflow File:
`.github/workflows/release.yml`

---

## Version Numbering (Semantic Versioning)

This project follows [Semantic Versioning](https://semver.org/) (MAJOR.MINOR.PATCH):

- **MAJOR** (1.0.0): Breaking changes, incompatible API changes
- **MINOR** (0.1.0): New features, backward-compatible
- **PATCH** (0.0.1): Bug fixes, backward-compatible

---

## Troubleshooting

### Release workflow failed to publish to Galaxy

**Issue**: `ansible-galaxy collection publish` failed

**Solution**:
1. Check that `GALAXY_API_KEY` secret is set correctly
2. Verify the API key is valid on Ansible Galaxy
3. Ensure the collection version doesn't already exist on Galaxy

---

## Testing Releases

Before publishing to Galaxy, you can test the build process:

```bash
# Build collection locally
ansible-galaxy collection build

# Test install locally
ansible-galaxy collection install jlira-web_server-0.0.2.tar.gz --force

# Test the collection
ansible-playbook test-playbook.yml
```

---

## Rollback a Release

If you need to rollback a release:

1. **Delete the tag locally and remotely**:
   ```bash
   git tag -d v0.0.2
   git push origin :refs/tags/v0.0.2
   ```

2. **Delete the GitHub Release**:
   - Go to Releases → Click on the release → Delete

3. **Note**: You cannot delete a version from Ansible Galaxy once published. You'll need to publish a new patch version with fixes.

---

## Additional Resources

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Semantic Versioning](https://semver.org/)
- [Keep a Changelog](https://keepachangelog.com/)
- [Ansible Galaxy Publishing Guide](https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_distributing.html)
