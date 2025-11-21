# Ansible Development Rules & Best Practices

## General Guidelines
- **Keep it simple**: Avoid over-engineering.
- **Commit Practices**: Commit every iteration using best practices (clear messages, logical chunks, 50/72 rule, type(scope): description structure).
- **Documentation**: Document all variables (purpose, example).

## Role Structure
- **Generation**: Use `ansible-galaxy init <role_name>` to generate new roles.
- **Single Purpose**: Keep roles focused on a single purpose.
- **Dependencies**: Limit role dependencies. Keep them loosely coupled.
- **Orchestration**: `tasks/main.yml` must only contain `include_tasks` or `import_tasks` for orchestration.

## Variables & Naming
- **Prefixing**: Always add the role’s name as a prefix to variables (e.g., `certificates_variable`).
- **FQCN**: Use Fully Qualified Collection Names for builtin module actions (e.g., `ansible.builtin.copy`).

## Execution
- **Tagging**: Use a consistent tagging strategy. Use `apply` tags in `main.yml` instead of defining tags on each task in included files.

## Code Style & Structure
- **Line Length**: Maximum line length is 120 characters.
- **Orchestration**: Do not orchestrate in sub-task files (e.g., `process_certificate.yml`). Orchestration logic (includes) should primarily reside in `tasks/main.yml`.
