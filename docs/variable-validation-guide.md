# Variable Validation Guide for jlira.web_server Collection

## Table of Contents
1. [Introduction](#introduction)
2. [Why Variable Validation Matters](#why-variable-validation-matters)
3. [Validation Strategies](#validation-strategies)
4. [Validation Techniques](#validation-techniques)
5. [Common Validation Patterns](#common-validation-patterns)
6. [Best Practices](#best-practices)
7. [Testing Validation Logic](#testing-validation-logic)
8. [Examples](#examples)

---

## Introduction

Variable validation is a defensive programming practice that ensures all inputs to an Ansible role meet expected requirements before any system modifications occur. This guide provides comprehensive information on implementing robust validation in the `jlira.web_server` collection.

### What is Variable Validation?

Variable validation is the process of checking that:
- Required variables are defined
- Variables have the correct data type
- Values fall within acceptable ranges
- Dependencies between variables are satisfied
- Inputs meet security and functional requirements

### Benefits

- **Fail Fast**: Catch errors before they cause issues deep in task execution
- **Clear Error Messages**: Provide actionable feedback instead of cryptic failures
- **Self-Documenting**: Validation logic serves as executable documentation
- **Security**: Prevent injection attacks and path traversal vulnerabilities
- **Reliability**: Ensure consistent behavior across different environments

---

## Why Variable Validation Matters

### The Problem Without Validation

Consider this scenario without validation:

```yaml
# User provides invalid PHP version
php_version: "8.3.1"  # Should be "8.3" but user included patch version

# Role attempts to install
- name: Install PHP packages
  ansible.builtin.apt:
    name: "php{{ php_version }}-cli"  # Becomes "php8.3.1-cli" - doesn't exist!
    state: present
```

**Result**: Task fails with unclear error like "Package php8.3.1-cli not found"

### The Solution With Validation

```yaml
# Validation catches the error early
- name: Validate php_version format (must be X.Y)
  ansible.builtin.assert:
    that:
      - php_version is match('^[0-9]+\.[0-9]+$')
    fail_msg: >-
      php_version must be in format 'X.Y' (e.g., '8.3', '8.4').
      Current value: '{{ php_version }}'
```

**Result**: Clear, actionable error message before any system changes occur

---

## Validation Strategies

### When to Fail vs. When to Warn

#### Fail Hard (Use `assert` with `that`)

Use when the condition **must** be met for the role to function:

- Required variables are missing
- Values are outside valid ranges (e.g., port 70000)
- Security violations (path traversal, injection risks)
- Type mismatches that break functionality
- Logical inconsistencies (e.g., max < min)

**Example:**
```yaml
- name: Validate critical port number
  ansible.builtin.assert:
    that:
      - apache_port | int > 0
      - apache_port | int < 65536
    fail_msg: "apache_port must be between 1 and 65535"
```

#### Warn (Use `ansible.builtin.debug` with `msg`)

Use when the configuration is suboptimal but functional:

- Performance-related values that are unusually low/high
- Deprecated options that still work
- Non-critical best practice violations

**Example:**
```yaml
- name: Warn about low PHP memory limit
  ansible.builtin.debug:
    msg: >-
      WARNING: php_memory_limit is set to {{ php_memory_limit }}.
      This is unusually low for modern PHP applications.
      Consider increasing to at least 128M.
  when:
    - php_memory_limit is match('^[0-9]+M$')
    - php_memory_limit | regex_replace('M$', '') | int < 64
```

### Trade-offs

| Approach | Advantages | Disadvantages | When to Use |
|----------|-----------|---------------|-------------|
| **Fail Hard** | Prevents broken configurations; Clear contract | May be too strict; Blocks automation | Critical requirements, security issues |
| **Warn** | Flexible; Doesn't block execution | Users may ignore warnings | Performance tuning, best practices |
| **Auto-Fix** | User-friendly; Continues execution | May hide underlying issues | Simple defaults, known safe values |

---

## Validation Techniques

### 1. Using `ansible.builtin.assert`

The primary tool for validation. The `assert` module checks conditions and fails if they're not met.

**Basic Syntax:**
```yaml
- name: Validate variable
  ansible.builtin.assert:
    that:
      - condition1
      - condition2
    fail_msg: "Error message with {{ variable_name }}"
    success_msg: "Success message"
    quiet: true  # Suppress success messages in output
```

**Key Parameters:**
- `that`: List of conditions (all must be true)
- `fail_msg`: Message displayed when validation fails
- `success_msg`: Message displayed on success (optional)
- `quiet`: If `true`, suppresses success messages (recommended)

### 2. Type Checking

Ansible provides several type-checking tests:

```yaml
# Check if variable is boolean
- variable_name is boolean

# Check if variable is number
- variable_name is number

# Check if variable is string
- variable_name is string

# Check if variable is list/array
- variable_name is iterable
- variable_name is not string
- variable_name is not mapping

# Check if variable is dictionary
- variable_name is mapping
```

**Example:**
```yaml
- name: Validate php_fpm_enabled is boolean
  ansible.builtin.assert:
    that:
      - php_fpm_enabled is defined
      - php_fpm_enabled is boolean
    fail_msg: "php_fpm_enabled must be a boolean (true/false)"
```

### 3. Regular Expression Validation

Use `match()` filter for pattern validation:

```yaml
# Exact match from start to end
- variable_name is match('^pattern$')

# Match anywhere in string
- variable_name is search('pattern')

# Case-insensitive match
- variable_name is match('(?i)pattern')
```

**Examples:**

```yaml
# Validate version format (X.Y)
- php_version is match('^[0-9]+\.[0-9]+$')

# Validate size format (128M, 2G, etc.)
- memory_limit is match('^[0-9]+[KMG]$')

# Validate absolute path
- error_log is match('^/')

# Validate alphanumeric with hyphens/underscores
- pool_name is match('^[a-zA-Z0-9_-]+$')

# Validate URL format
- repo_url is match('^https?://')
```

### 4. Range Validation

Check if values are within acceptable ranges:

```yaml
# Numeric range
- name: Validate port number
  ansible.builtin.assert:
    that:
      - apache_port | int >= 1
      - apache_port | int <= 65535
    fail_msg: "Port must be between 1 and 65535"

# String length
- name: Validate server name length
  ansible.builtin.assert:
    that:
      - server_name | length > 0
      - server_name | length <= 255
    fail_msg: "Server name must be 1-255 characters"

# List/array size
- name: Validate packages list is not empty
  ansible.builtin.assert:
    that:
      - php_packages | length > 0
    fail_msg: "php_packages list cannot be empty"
```

### 5. Enum/Choice Validation

Ensure values are from a predefined set:

```yaml
- name: Validate process manager type
  ansible.builtin.assert:
    that:
      - php_pool_settings.pm in ['static', 'dynamic', 'ondemand']
    fail_msg: >-
      php_pool_settings.pm must be one of: 'static', 'dynamic', 'ondemand'.
      Current value: '{{ php_pool_settings.pm }}'
```

### 6. Dependency Validation

Check relationships between variables:

```yaml
- name: Validate FPM settings when FPM is enabled
  ansible.builtin.assert:
    that:
      - php_fpm_settings is defined
      - php_fpm_settings is mapping
    fail_msg: "php_fpm_settings must be defined when php_fpm_enabled is true"
  when: php_fpm_enabled | bool

- name: Validate max > min relationship
  ansible.builtin.assert:
    that:
      - php_pool_settings.pm_max_spare_servers | int >= php_pool_settings.pm_min_spare_servers | int
    fail_msg: "pm_max_spare_servers must be >= pm_min_spare_servers"
```

### 7. Security Validation

Prevent common security issues:

```yaml
- name: Validate path for security issues
  ansible.builtin.assert:
    that:
      # Must be absolute path
      - error_log is match('^/')
      # No path traversal
      - "'..' not in error_log"
      # No null bytes
      - "'\\x00' not in error_log"
      # Not just a directory
      - error_log is not match('/$')
    fail_msg: "error_log path has security issues: {{ error_log }}"
```

---

## Common Validation Patterns

### Pattern 1: Required Variable

```yaml
- name: Validate required variable is defined and non-empty
  ansible.builtin.assert:
    that:
      - variable_name is defined
      - variable_name | length > 0
    fail_msg: "variable_name must be defined and non-empty"
    quiet: true
```

### Pattern 2: File Path Validation

```yaml
- name: Validate file path
  ansible.builtin.assert:
    that:
      - path_variable is string
      - path_variable | length > 0
      - path_variable is match('^/')           # Absolute path
      - "'..' not in path_variable"            # No traversal
      - path_variable is not match('/$')       # Not a directory
    fail_msg: "Invalid file path: {{ path_variable }}"
    quiet: true
```

### Pattern 3: Directory Path Validation

```yaml
- name: Validate directory path
  ansible.builtin.assert:
    that:
      - dir_variable is string
      - dir_variable | length > 0
      - dir_variable is match('^/')
      - "'..' not in dir_variable"
    fail_msg: "Invalid directory path: {{ dir_variable }}"
    quiet: true
```

### Pattern 4: URL Validation

```yaml
- name: Validate URL format
  ansible.builtin.assert:
    that:
      - url_variable is string
      - url_variable | length > 0
      - url_variable is match('^https?://[a-zA-Z0-9.-]+')
    fail_msg: "Invalid URL format: {{ url_variable }}"
    quiet: true
```

### Pattern 5: Port Number Validation

```yaml
- name: Validate port number
  ansible.builtin.assert:
    that:
      - port_variable is number
      - port_variable | int > 0
      - port_variable | int < 65536
    fail_msg: >-
      Port must be between 1 and 65535.
      Current value: {{ port_variable }}
    quiet: true
```

### Pattern 6: Email Validation

```yaml
- name: Validate email format
  ansible.builtin.assert:
    that:
      - email_variable is string
      - email_variable is match('^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')
    fail_msg: "Invalid email format: {{ email_variable }}"
    quiet: true
```

### Pattern 7: PHP Size Format Validation

```yaml
- name: Validate PHP size format (memory_limit, post_max_size, etc.)
  ansible.builtin.assert:
    that:
      - size_variable is string
      - >-
        size_variable is match('^-1$') or
        size_variable is match('^[0-9]+[KMG]?$')
    fail_msg: >-
      Invalid size format: {{ size_variable }}.
      Valid formats: '-1' (unlimited), '128M', '2G', '512K', '1024' (bytes)
    quiet: true
```

### Pattern 8: Boolean Validation

```yaml
- name: Validate boolean variable
  ansible.builtin.assert:
    that:
      - bool_variable is defined
      - bool_variable is boolean
    fail_msg: >-
      variable_name must be a boolean (true/false).
      Current value: '{{ bool_variable }}' (type: {{ bool_variable | type_debug }})
    quiet: true
```

### Pattern 9: Dictionary Structure Validation

```yaml
- name: Validate dictionary structure
  ansible.builtin.assert:
    that:
      - settings_dict is defined
      - settings_dict is mapping
      - settings_dict.required_key is defined
    fail_msg: "settings_dict must be a dictionary with required_key"
    quiet: true
```

### Pattern 10: List Validation

```yaml
- name: Validate list variable
  ansible.builtin.assert:
    that:
      - list_variable is defined
      - list_variable is iterable
      - list_variable is not string
      - list_variable is not mapping
      - list_variable | length > 0
    fail_msg: "list_variable must be a non-empty list"
    quiet: true
```

---

## Best Practices

### 1. Validation Task File Organization

**Recommended Structure:**
```yaml
# roles/rolename/tasks/validate.yml

# 1. Required Variable Existence Checks
- name: Check all required variables are defined
  # ...

# 2. Type Validations
- name: Validate variable types
  # ...

# 3. Format Validations
- name: Validate value formats
  # ...

# 4. Range Validations
- name: Validate values are in acceptable ranges
  # ...

# 5. Logical Consistency Checks
- name: Validate relationships between variables
  # ...

# 6. Dependency Validations
- name: Validate conditional dependencies
  # ...

# 7. Summary
- name: Display validation summary
  # ...
```

### 2. Error Message Best Practices

**Good Error Messages:**
```yaml
fail_msg: >-
  php_version must be in format 'X.Y' (e.g., '8.3', '8.4').
  Current value: '{{ php_version }}'
```

**What makes it good:**
- Explains what's wrong
- Shows expected format
- Provides examples
- Displays current (invalid) value

**Bad Error Messages:**
```yaml
fail_msg: "Invalid version"  # Too vague
fail_msg: "Error"            # No information
```

### 3. Use `quiet: true`

Suppress success messages to keep output clean:

```yaml
- name: Validate variable
  ansible.builtin.assert:
    that:
      - condition
    fail_msg: "Error message"
    quiet: true  # Don't show success message
```

### 4. Validation Placement

**Always run validation first:**
```yaml
# roles/php/tasks/main.yml
---
- name: Include validation tasks
  ansible.builtin.include_tasks:
    file: validate.yml
    apply:
      tags:
        - always  # Always run validation
  tags:
    - always

# Then other tasks...
```

**Why `tags: always`?**
- Ensures validation runs even when using `--tags`
- Example: `ansible-playbook site.yml --tags install` will still validate

### 5. Conditional Validation

Only validate variables that are relevant:

```yaml
- name: Validate FPM settings
  # ... validation logic ...
  when: php_fpm_enabled | bool
```

### 6. Loop Validation for Dictionaries

When validating complex structures:

```yaml
- name: Validate each PHP setting
  ansible.builtin.assert:
    that:
      - item.value.type is defined
      - item.value.value is defined
    fail_msg: "Invalid structure for setting: {{ item.key }}"
    quiet: true
  loop: "{{ php_cli_settings | dict2items }}"
  loop_control:
    label: "{{ item.key }}"  # Only show key in output, not full item
```

### 7. Security Validation Priority

**Always validate for security first:**
```yaml
# 1. Security checks (prevent injection, traversal)
- name: Security validation
  # ...

# 2. Format checks (ensure correct format)
- name: Format validation
  # ...

# 3. Business logic checks (ensure values make sense)
- name: Business logic validation
  # ...
```

### 8. Document Validation Requirements

In `defaults/main.yml`, document what's validated:

```yaml
# Path to the error log file.
# Requirements:
#   - Must be an absolute path (start with /)
#   - Must not contain '..' (security)
#   - Must point to a file, not directory
# Type: String (Path)
php_error_log: /var/log/php/php_errors.log
```

---

## Testing Validation Logic

### Molecule Test Structure

Create dedicated test scenarios to verify validation works:

```yaml
# extensions/molecule/php/tests/validate_variables.yml
---
- name: Test validation with invalid php_version
  block:
    - name: Set invalid php_version
      ansible.builtin.set_fact:
        php_version: "8.3.1"  # Invalid format

    - name: Run role (should fail)
      ansible.builtin.include_role:
        name: jlira.web_server.php
      ignore_errors: true
      register: role_result

    - name: Verify role failed
      ansible.builtin.assert:
        that:
          - role_result is failed
          - "'format' in role_result.msg | lower"
        fail_msg: "Role should have failed due to invalid php_version"

  rescue:
    - name: Expected failure occurred
      ansible.builtin.debug:
        msg: "Validation correctly caught invalid php_version"
```

### Test Cases to Cover

1. **Missing Required Variables**
   ```yaml
   - Undefined variable
   - Empty string
   - Null value
   ```

2. **Type Mismatches**
   ```yaml
   - String instead of boolean
   - Integer instead of string
   - List instead of dictionary
   ```

3. **Format Violations**
   ```yaml
   - Invalid version format
   - Malformed paths
   - Invalid size format
   ```

4. **Range Violations**
   ```yaml
   - Negative numbers where positive required
   - Out-of-range port numbers
   - Zero where positive required
   ```

5. **Security Issues**
   ```yaml
   - Path traversal attempts (..)
   - Null byte injection
   - Suspicious patterns
   ```

6. **Logical Inconsistencies**
   ```yaml
   - Max < Min
   - Dependencies not met
   - Conflicting options
   ```

---

## Examples

### Example 1: Complete Variable Validation Set

```yaml
# roles/example/tasks/validate.yml
---
# Required string variable
- name: Validate server_name is defined
  ansible.builtin.assert:
    that:
      - server_name is defined
      - server_name is string
      - server_name | length > 0
    fail_msg: "server_name must be defined as a non-empty string"
    quiet: true

# Port number
- name: Validate server_port
  ansible.builtin.assert:
    that:
      - server_port is defined
      - server_port is number
      - server_port | int > 0
      - server_port | int < 65536
    fail_msg: >-
      server_port must be between 1 and 65535.
      Current value: {{ server_port }}
    quiet: true

# File path
- name: Validate log_file path
  ansible.builtin.assert:
    that:
      - log_file is defined
      - log_file is string
      - log_file is match('^/')
      - "'..' not in log_file"
      - log_file is not match('/$')
    fail_msg: "log_file must be a valid absolute file path"
    quiet: true

# Boolean
- name: Validate feature_enabled
  ansible.builtin.assert:
    that:
      - feature_enabled is defined
      - feature_enabled is boolean
    fail_msg: "feature_enabled must be true or false"
    quiet: true

# Enum
- name: Validate log_level
  ansible.builtin.assert:
    that:
      - log_level is defined
      - log_level in ['debug', 'info', 'warning', 'error', 'critical']
    fail_msg: >-
      log_level must be one of: debug, info, warning, error, critical.
      Current value: {{ log_level }}
    quiet: true

# List
- name: Validate allowed_hosts
  ansible.builtin.assert:
    that:
      - allowed_hosts is defined
      - allowed_hosts is iterable
      - allowed_hosts is not string
      - allowed_hosts | length > 0
    fail_msg: "allowed_hosts must be a non-empty list"
    quiet: true

# Dictionary
- name: Validate settings dictionary
  ansible.builtin.assert:
    that:
      - settings is defined
      - settings is mapping
      - settings.timeout is defined
    fail_msg: "settings must be a dictionary with 'timeout' key"
    quiet: true

# Conditional dependency
- name: Validate SSL settings when SSL is enabled
  ansible.builtin.assert:
    that:
      - ssl_certificate is defined
      - ssl_certificate_key is defined
    fail_msg: "SSL certificate and key must be defined when ssl_enabled is true"
  when: ssl_enabled | bool

# Logical consistency
- name: Validate min/max relationship
  ansible.builtin.assert:
    that:
      - max_connections | int >= min_connections | int
    fail_msg: >-
      max_connections must be >= min_connections.
      Current: max={{ max_connections }}, min={{ min_connections }}
    quiet: true
```

### Example 2: PHP-Specific Validations

```yaml
# Validate PHP memory limit format and value
- name: Validate memory_limit format
  ansible.builtin.assert:
    that:
      - memory_limit is match('^-1$') or memory_limit is match('^[0-9]+[KMG]?$')
    fail_msg: >-
      memory_limit must be '-1' or a size like '128M', '2G', '512K'.
      Current: {{ memory_limit }}
    quiet: true

# Warn if memory limit is low
- name: Check if memory limit is low
  ansible.builtin.debug:
    msg: >-
      WARNING: memory_limit is {{ memory_limit }}.
      Consider increasing to at least 128M for modern PHP applications.
  when:
    - memory_limit is match('^[0-9]+M$')
    - memory_limit | regex_replace('M$', '') | int < 128

# Validate PHP version is supported
- name: Validate PHP version is supported
  ansible.builtin.assert:
    that:
      - php_version is match('^[78]\.[0-9]+$')
    fail_msg: >-
      Only PHP 7.x and 8.x versions are supported.
      Current: {{ php_version }}
    quiet: true

# Validate upload size is less than post size
- name: Validate upload_max_filesize <= post_max_size
  ansible.builtin.assert:
    that:
      - upload_max_filesize_bytes | int <= post_max_size_bytes | int
    fail_msg: >-
      upload_max_filesize must be <= post_max_size.
      Current: upload={{ upload_max_filesize }}, post={{ post_max_size }}
  vars:
    upload_max_filesize_bytes: "{{ upload_max_filesize | regex_replace('[KMG]', '') | int * multiplier }}"
    post_max_size_bytes: "{{ post_max_size | regex_replace('[KMG]', '') | int * multiplier }}"
    multiplier: >-
      {{
        1024 if 'K' in upload_max_filesize else
        1024 * 1024 if 'M' in upload_max_filesize else
        1024 * 1024 * 1024 if 'G' in upload_max_filesize else
        1
      }}
```

### Example 3: Advanced Dictionary Validation

```yaml
# Validate complex nested dictionary
- name: Validate PHP pool settings structure
  ansible.builtin.assert:
    that:
      # Required keys exist
      - php_pool_settings.pm is defined
      - php_pool_settings.pm_max_children is defined

      # Types are correct
      - php_pool_settings.pm is string
      - php_pool_settings.pm_max_children is number

      # Values are valid
      - php_pool_settings.pm in ['static', 'dynamic', 'ondemand']
      - php_pool_settings.pm_max_children | int > 0
    fail_msg: "Invalid php_pool_settings structure"
    quiet: true

# Validate each item in dictionary using loop
- name: Validate each PHP-FPM pool setting
  ansible.builtin.assert:
    that:
      - item.key is match('^[a-z_]+$')
      - item.value is defined
    fail_msg: "Invalid pool setting: {{ item.key }}"
    quiet: true
  loop: "{{ php_pool_settings | dict2items }}"
  loop_control:
    label: "{{ item.key }}"
```

---

## Summary

Variable validation is essential for creating robust, maintainable Ansible roles. By following the patterns and best practices in this guide, you can:

1. **Catch errors early** before they cause system issues
2. **Provide clear feedback** to users when something is wrong
3. **Document requirements** in an executable format
4. **Improve security** by preventing injection and traversal attacks
5. **Ensure consistency** across different environments

### Key Takeaways

- Always validate at the start of role execution (use `tags: always`)
- Use descriptive error messages that explain what's wrong and how to fix it
- Validate security issues first (paths, injections, etc.)
- Use `quiet: true` to keep output clean
- Test your validation logic with Molecule
- Document validation requirements in variable defaults

### Additional Resources

- [Ansible assert module documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/assert_module.html)
- [Ansible tests (type checking)](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tests.html)
- [Jinja2 filters (regex, match, etc.)](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html)
- [jlira.web_server Collection README](/README.md)