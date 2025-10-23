# Release Process Guide

This document describes the automated release workflows for the `jlira.web_server` Ansible collection.

## Prerequisites

### 1. Ansible Galaxy API Token

Set up your Ansible Galaxy API token for publishing:

1. Log in to [Ansible Galaxy](https://galaxy.ansible.com/)
2. Go to your profile → Preferences → API Key
3. Copy your API token
4. Add it to your GitHub repository secrets:
   - Go to Settings → Secrets and variables → Actions
   - Create a new secret named `GALAXY_API_KEY`
   - Paste your API token

### 2. Branch Protection Configuration (for automated workflow)

If you want to use the automated release workflow with a **protected main branch**, you need to configure GitHub Actions to bypass branch protection:

**Option A: Allow GitHub Actions to bypass (Recommended)**
1. Go to: Settings → Branches → Branch protection rules for `main`
2. Scroll to: "Rules applied to everyone including administrators"
3. Enable: **"Allow specified actors to bypass required pull requests"**
4. Add: `github-actions[bot]` to the allowed actors list

**Option B: Use only the semi-automated workflow**
- If you don't want to bypass branch protection at all
- Use **Option 1: Semi-Automated Release** only
- This creates tags from any branch (doesn't need main branch push access)

**Option C: Use PAT (Personal Access Token) - Advanced**
1. Create a GitHub Personal Access Token with `repo` scope
2. Add it as a repository secret: `RELEASE_TOKEN`
3. Update workflow to use `token: ${{ secrets.RELEASE_TOKEN }}` instead of `GITHUB_TOKEN`

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

### Option 2: Automated Release (Manual Trigger)

**Best for**: Protected main branches where direct pushes are not allowed.

**Note**: This workflow is configured for **manual triggering only** because most production repositories have protected main branches that require pull requests. If your main branch is NOT protected, you can uncomment the `push` trigger in `.github/workflows/auto-release.yml`.

#### Process (for protected branches):

1. **Merge your PR to main** using conventional commit messages:
   ```bash
   # Create feature branch
   git checkout -b feature/virtual-hosts

   # Make changes and commit using conventional commits
   git commit -m "feat: add virtual host configuration support"

   # Push and create PR
   git push origin feature/virtual-hosts
   # Create PR via GitHub UI and merge it
   ```

2. **After PR is merged, manually trigger the release workflow**:
   - Go to: **Actions → "Automated Release (Conventional Commits)"**
   - Click **"Run workflow"**
   - Select release type:
     - `auto` - Detects version bump from recent commits
     - `major` - Breaking changes (X.0.0)
     - `minor` - New features (0.X.0)
     - `patch` - Bug fixes (0.0.X)
   - Click **"Run workflow"**

3. **Automatic steps** (handled by `.github/workflows/auto-release.yml`):
   - ✅ Analyzes recent commits since last tag
   - ✅ Determines version bump
   - ✅ Generates changelog entry from commits
   - ✅ Updates `CHANGELOG.md` and `galaxy.yml`
   - ✅ Commits changes directly to main (bypasses PR requirement)
   - ✅ Creates and pushes version tag
   - ✅ Builds collection
   - ✅ Publishes to Ansible Galaxy
   - ✅ Creates GitHub Release

#### For unprotected branches (optional):

If your main branch is **NOT** protected, you can enable automatic releases on push:

1. Edit `.github/workflows/auto-release.yml`
2. Uncomment lines 6-8:
   ```yaml
   on:
     push:  # Uncomment these lines
       branches:
         - main
   ```
3. Now pushes with conventional commits will trigger releases automatically

#### Workflow File:
`.github/workflows/auto-release.yml`

---

## Conventional Commit Format

The automated release workflow uses [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Commit Types:

- `feat:` - A new feature (triggers MINOR version bump)
- `fix:` - A bug fix (triggers PATCH version bump)
- `feat!:` or `BREAKING CHANGE:` - Breaking change (triggers MAJOR version bump)
- `docs:` - Documentation changes (no release)
- `chore:` - Maintenance tasks (no release)
- `test:` - Test changes (no release)
- `refactor:` - Code refactoring (no release)

### Examples:

```bash
# Add new feature (0.0.1 → 0.1.0)
git commit -m "feat: add SSL certificate management for Apache"

# Fix bug (0.1.0 → 0.1.1)
git commit -m "fix: resolve PHP-FPM service restart issue"

# Breaking change (0.1.1 → 1.0.0)
git commit -m "feat!: rename apache_port to apache_http_port

BREAKING CHANGE: The variable apache_port has been renamed to apache_http_port"

# Multiple changes
git commit -m "feat(apache): add virtual host support
feat(php): add PHP 8.3 support
fix(apache): correct service handler"
```

---

## Comparison: Which Method to Use?

| Feature | Semi-Automated | Fully Automated |
|---------|----------------|-----------------|
| **Trigger** | Manual tag creation | Commit to main branch |
| **Changelog** | Manually written | Auto-generated from commits |
| **Version bump** | You decide | Based on commit type |
| **Control** | Full control | Convention-based |
| **Best for** | Major releases, curated changelogs | Frequent releases, CI/CD |
| **Effort** | More manual work | Less manual work |

### Recommendations:

- **Use Semi-Automated** when:
  - You want to craft detailed, user-friendly changelogs
  - You're making major releases
  - You want full control over when releases happen
  - You prefer explicit versioning

- **Use Fully Automated** when:
  - You follow conventional commits already
  - You want faster release cycles
  - You trust automated changelog generation
  - You want less manual intervention

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

### Changelog not updating correctly

**Issue**: Automated changelog generation is incorrect

**Solution**:
1. Ensure you're using conventional commit format
2. Check commit messages follow the pattern: `type: description`
3. Manually edit CHANGELOG.md if needed and use semi-automated release

### Version bump is incorrect

**Issue**: Wrong version number after automated release

**Solution**:
1. Verify commit message type (feat, fix, feat!)
2. Use manual trigger to specify exact bump type
3. Or use semi-automated release for full control

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
