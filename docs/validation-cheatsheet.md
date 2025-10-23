# Ansible Variable Validation Cheatsheet

Quick reference for common validation patterns in Ansible roles.

---

## Basic Assert Structure

```yaml
- name: Validate variable
  ansible.builtin.assert:
    that:
      - condition1
      - condition2
    fail_msg: "Clear error message with {{ variable }}"
    success_msg: "Optional success message"
    quiet: true  # Recommended: suppress success output
```

---

## Type Validation

```yaml
# Boolean
- variable is boolean

# Number/Integer
- variable is number
- variable | int == variable  # Ensures it's actually an integer

# String
- variable is string

# List/Array
- variable is iterable
- variable is not string
- variable is not mapping

# Dictionary/Map
- variable is mapping
```

---

## Existence & Emptiness

```yaml
# Variable is defined
- variable is defined

# Variable is defined and not empty (string)
- variable is defined
- variable | length > 0

# Variable is defined and not empty (list)
- variable is defined
- variable | length > 0

# Variable is not none/null
- variable is not none
```

---

## String Patterns (Regex)

```yaml
# Exact match (anchored)
- variable is match('^pattern$')

# Contains pattern
- variable is search('pattern')

# Examples:
- php_version is match('^[0-9]+\.[0-9]+$')        # "8.3"
- email is match('^[^@]+@[^@]+\.[a-zA-Z]{2,}$')   # "user@example.com"
- url is match('^https?://')                       # "http://" or "https://"
- size is match('^[0-9]+[KMG]$')                   # "128M", "2G"
- path is match('^/')                              # Absolute path
- name is match('^[a-zA-Z0-9_-]+$')               # Alphanumeric + _ -
```

---

## Numeric Ranges

```yaml
# Basic range
- variable | int >= 1
- variable | int <= 100

# Port number
- variable | int > 0
- variable | int < 65536

# Positive number
- variable | int > 0

# Non-negative number
- variable | int >= 0
```

---

## Enum/Choice Validation

```yaml
# Single variable
- variable in ['option1', 'option2', 'option3']

# Example
- log_level in ['debug', 'info', 'warning', 'error']
- php_pool_pm in ['static', 'dynamic', 'ondemand']
```

---

## File & Directory Paths

### File Path
```yaml
- path is string
- path | length > 0
- path is match('^/')              # Absolute path
- "'..' not in path"               # No path traversal
- "'\\x00' not in path"            # No null bytes
- path is not match('/$')          # Not a directory
```

### Directory Path
```yaml
- path is string
- path | length > 0
- path is match('^/')
- "'..' not in path"
```

### URL
```yaml
- url is string
- url | length > 0
- url is match('^https?://[a-zA-Z0-9.-]+')
```

---

## PHP-Specific Patterns

### PHP Version
```yaml
- php_version is match('^[0-9]+\.[0-9]+$')  # "8.3", "8.4"
```

### PHP Size Format
```yaml
# Accepts: -1, 128M, 2G, 512K, 1024
- size is match('^-1$') or size is match('^[0-9]+[KMG]?$')
```

### PHP Boolean in php.ini
```yaml
- value is boolean  # true/false in YAML
```

### PHP Error Reporting
```yaml
- error_level is string
- error_level | length > 0
# Examples: "E_ALL", "E_ALL & ~E_DEPRECATED"
```

---

## Relationship Validation

### Greater Than / Less Than
```yaml
- max_value | int > min_value | int
- max_value | int >= min_value | int
```

### Dependency Checks
```yaml
# If A is enabled, B must be defined
- name: Validate dependency
  ansible.builtin.assert:
    that:
      - dependent_variable is defined
    fail_msg: "dependent_variable required when feature_enabled is true"
  when: feature_enabled | bool
```

---

## List Validation

### Non-Empty List
```yaml
- list_var is defined
- list_var is iterable
- list_var is not string
- list_var is not mapping
- list_var | length > 0
```

### List Contains Value
```yaml
- "'required_item' in list_var"
```

### Validate Each List Item
```yaml
- name: Validate list items
  ansible.builtin.assert:
    that:
      - item is string
      - item | length > 0
    fail_msg: "Invalid item in list: {{ item }}"
  loop: "{{ list_var }}"
```

---

## Dictionary Validation

### Dictionary Structure
```yaml
- dict_var is defined
- dict_var is mapping
- dict_var.required_key is defined
```

### Validate Each Key-Value Pair
```yaml
- name: Validate dictionary items
  ansible.builtin.assert:
    that:
      - item.key is match('^[a-z_]+$')
      - item.value is defined
    fail_msg: "Invalid entry: {{ item.key }}"
  loop: "{{ dict_var | dict2items }}"
  loop_control:
    label: "{{ item.key }}"  # Only show key in output
```

---

## Complex Validation Examples

### PHP Pool Settings (Dynamic PM)
```yaml
# Validate relationships between pool settings
- name: Validate dynamic PM consistency
  ansible.builtin.assert:
    that:
      - pm_start_servers | int >= pm_min_spare_servers | int
      - pm_start_servers | int <= pm_max_spare_servers | int
      - pm_max_spare_servers | int >= pm_min_spare_servers | int
      - pm_max_spare_servers | int <= pm_max_children | int
    fail_msg: "Dynamic PM settings are inconsistent"
```

### Nested Dictionary with Type Checking
```yaml
- name: Validate PHP settings structure
  ansible.builtin.assert:
    that:
      - item.value.type is defined
      - item.value.value is defined
      - item.value.type in ['bool', 'int', 'string', 'size', 'path']
    fail_msg: "Invalid structure for {{ item.key }}"
  loop: "{{ php_settings | dict2items }}"
  loop_control:
    label: "{{ item.key }}"
```

---

## Error Message Best Practices

### Good Error Messages
```yaml
fail_msg: >-
  variable_name must be in format 'X.Y' (e.g., '8.3', '8.4').
  Current value: '{{ variable_name }}'
```

**Elements of good error messages:**
1. What's wrong
2. What's expected
3. Example of correct format
4. Current (invalid) value

### Bad Error Messages
```yaml
fail_msg: "Invalid"           # Too vague
fail_msg: "Error"             # No context
fail_msg: "Wrong format"      # No details
```

---

## Conditional Validation

```yaml
# Only validate when feature is enabled
- name: Validate feature settings
  ansible.builtin.assert:
    that:
      - feature_settings is defined
    fail_msg: "feature_settings required when feature_enabled is true"
  when: feature_enabled | bool

# Validate different requirements based on type
- name: Validate based on type
  ansible.builtin.assert:
    that:
      - condition_for_type_a
    fail_msg: "Error for type A"
  when: type == 'A'
```

---

## Security Validations

```yaml
# Path traversal prevention
- "'..' not in path_variable"

# Null byte injection prevention
- "'\\x00' not in string_variable"

# Ensure absolute paths (not relative)
- path_variable is match('^/')

# Alphanumeric only (prevent injection)
- variable is match('^[a-zA-Z0-9_-]+$')
```

---

## Common Filters & Tests

### Type Checks
- `is boolean`
- `is number`
- `is string`
- `is iterable`
- `is mapping`

### Existence
- `is defined`
- `is not defined`
- `is none`

### Pattern Matching
- `is match('pattern')`
- `is search('pattern')`

### Conversions
- `| int`
- `| string`
- `| bool`
- `| length`

---

## Testing Validation

### Test Invalid Input
```yaml
- name: Test validation catches invalid input
  block:
    - name: Set invalid variable
      ansible.builtin.set_fact:
        test_var: "invalid_value"

    - name: Run role (should fail)
      ansible.builtin.include_role:
        name: role_name
      ignore_errors: true
      register: result

    - name: Verify validation caught the error
      ansible.builtin.assert:
        that:
          - result is failed
        fail_msg: "Validation should have caught invalid input"
```

---

## Quick Tips

1. **Always use `quiet: true`** to keep output clean
2. **Validate early** - first task in main.yml with `tags: always`
3. **Use descriptive names** - "Validate X is Y" format
4. **Include current value** in fail_msg for debugging
5. **Group related validations** in blocks
6. **Test validation logic** with Molecule
7. **Document requirements** in defaults/main.yml

---

## Template for New Validation File

```yaml
# roles/ROLE_NAME/tasks/validate.yml
---
# 1. Required variable checks
- name: Validate required_var is defined
  ansible.builtin.assert:
    that:
      - required_var is defined
      - required_var | length > 0
    fail_msg: "required_var must be defined and non-empty"
    quiet: true

# 2. Type validations
- name: Validate variable types
  ansible.builtin.assert:
    that:
      - bool_var is boolean
      - int_var is number
      - string_var is string
    fail_msg: "Variable type mismatch"
    quiet: true

# 3. Format validations
- name: Validate formats
  ansible.builtin.assert:
    that:
      - version is match('^[0-9]+\.[0-9]+$')
    fail_msg: "Invalid format"
    quiet: true

# 4. Range validations
- name: Validate ranges
  ansible.builtin.assert:
    that:
      - port | int > 0
      - port | int < 65536
    fail_msg: "Value out of range"
    quiet: true

# 5. Dependency validations
- name: Validate dependencies
  ansible.builtin.assert:
    that:
      - dependent_var is defined
    fail_msg: "Dependency not met"
  when: feature_enabled | bool

# 6. Summary
- name: Display validation summary
  ansible.builtin.debug:
    msg: "All validations passed!"
```

---

## Including Validation in main.yml

```yaml
# roles/ROLE_NAME/tasks/main.yml
---
- name: Include validation tasks
  ansible.builtin.include_tasks:
    file: validate.yml
    apply:
      tags:
        - always
  tags:
    - always

# Rest of your tasks...
```

---

## References

- Full Guide: [variable-validation-guide.md](./variable-validation-guide.md)
- Ansible assert module: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/assert_module.html
- Ansible tests: https://docs.ansible.com/ansible/latest/user_guide/playbooks_tests.html
- Jinja2 filters: https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html