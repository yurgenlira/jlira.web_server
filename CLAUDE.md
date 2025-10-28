# Project Overview
This project, `jlira.web_server`, is an Ansible collection designed to automate the setup, configuration, and management of web servers on Linux systems. Its primary goal is to provide idempotent, reusable, and well-documented roles for provisioning Apache web servers and PHP runtimes. The intended audience is system administrators, DevOps engineers, and developers who need a standardized and automated way to deploy and maintain web infrastructure.

## Technology Stack
- **Ansible**: The core automation engine used for configuration management and application deployment.
- **YAML**: The language used for writing Ansible playbooks and role definitions.
- **Jinja2**: The templating engine used for creating dynamic configuration files.
- **Molecule**: The testing framework used for developing and verifying the functionality of Ansible roles across different environments.
- **Docker**: Used by Molecule to create containerized environments for testing roles.
- **GitHub Actions**: The CI/CD platform used for automating linting, testing, and validation of the collection.

## Key Features
The collection provides the following Ansible roles and functionalities:
- **`jlira.web_server.apache`**:
    - Installs the Apache web server (`apache2`).
    - Manages the Apache service (start, stop, restart, enable).
    - Deploys custom web pages.
    - Configures virtual hosts.
- **`jlira.web_server.php`**:
    - Installs specific versions of PHP.
    - Configures PHP for both CLI and FPM (FastCGI Process Manager) usage.
    - Manages PHP-FPM service.
    - Adjusts `php.ini` settings for performance and security.

## Folder Structure
A high-level overview of the project's directory layout:
```
/
├── roles/ - Contains the core Ansible roles.
│   ├── apache/ - Role for managing the Apache web server.
│   └── php/ - Role for managing PHP.
├── extensions/ - Houses extensions and testing configurations.
│   └── molecule/ - Contains Molecule testing scenarios for the roles.
│       ├── apache/ - Tests for the `apache` role.
│       └── php/ - Tests for the `php` role.
├── .github/ - CI/CD workflows.
│   └── workflows/ - GitHub Actions workflows for automated testing.
├── galaxy.yml - Ansible collection metadata.
├── README.md - The main README file for the project.
└── ... (other configuration and documentation files)
```

## Architecture and Design Principles
The project is structured as an Ansible collection, which is the standard way to distribute and reuse Ansible content. It follows the official Ansible role directory structure.

- **Idempotency**: All roles and tasks are designed to be idempotent. Running them multiple times will result in the same system state, preventing unintended changes.
- **Modularity**: Each role has a single, clear purpose (e.g., managing Apache, managing PHP). This separation of concerns makes the roles easier to maintain, test, and reuse.
- **Default Variables**: Roles provide sensible default variables in `defaults/main.yml`, which can be easily overridden by the user for customization.
- **Testing**: Every role is accompanied by a Molecule test suite to ensure its correctness and reliability. Tests cover installation, configuration, and idempotency.
- **CI/CD**: The repository is configured with GitHub Actions workflows to automatically run `ansible-lint`, `yamllint`, and Molecule tests on every pull request and merge to the main branch.

## Technical Constraints and Guidelines
- **Linting**: All YAML and Ansible files must pass `yamllint` and `ansible-lint` checks before being committed. Configuration for these linters is provided in `.yamllint.yml` and `.ansible-lint`.
- **Variable Naming**: Role variables should be prefixed with the role name (e.g., `apache_port`, `php_version`) to avoid naming conflicts.
- **Commits**: Commit messages should follow a conventional format, clearly describing the change.
- **Documentation**: New features or changes to existing roles must be documented in the role's `README.md` file.
- **Testing**: Any new functionality must be accompanied by corresponding Molecule tests to validate its behavior.

## Best Practices and Tips
-   **Generic Best Practices:**
    - Prefer YAML instead of JSON: Although Ansible allows JSON syntax, using YAML is preferred and improves the readability of files and projects.
    - Use consistent whitespaces: To separate things nicely and improve readability, consider leaving a blank line between blocks, tasks, or other components.
    - Use a consistent tagging strategy: Tagging is a powerful concept in Ansible since it allows us to group and manage tasks more granularly.  Tags provide us with the option to add fine-grained controls to the execution of tasks.
    - Use a consistent naming strategy: Before starting to set up your Ansible projects, consider applying a consistent naming convention for your tasks (always name them), plays, variables, roles, and modules.
    - Define a style guide: Following a style guide might be helpful to be consistent with the above suggestions. Example: openshift-ansible Style Guide
    - Keep it simple: Ansible provides many options and advanced features, but that doesn’t mean we have to use all of them. Find the Ansible parts and mechanics that fit your use case and keep your Ansible projects as simple as possible. For example, begin with a simple playbook and static inventory and add more complex structures or refactor later according to your needs.
    - Store your projects in a Version Control System (VCS): Keep your Ansible files in a code repository and commit any new changes regularly.
    - Don’t store sensitive values in plain text: For secrets and sensitive values, use Ansible Vault to encrypt variables and files and protect any sensitive information.
    - Test your ansible projects: Use linting tools like Ansible Lint and add testing steps in your CI/CD pipelines for your Ansible repositories. For testing Ansible roles, have a look at Molecule. To test inputs or verify custom expressions, you can use the assert module.
-   **Inventory Best Practices:**
    - Use inventory groups: Group hosts based on common attributes they might share (geography, purpose, roles, environment).
    - Separate Inventory per Environment: Define a separate inventory file per environment (production, staging, testing, etc.) to isolate them from each other and avoid mistakes by targeting the wrong environments.
    - Dynamic Inventory: When working with cloud providers and ephemeral or fast-changing environments, maintaining static inventories might quickly become complex. Instead, set up a mechanism to synchronize the inventory dynamically with your cloud providers.
    - Leverage Dynamic Grouping at runtime: We can create dynamic groups using the `group_by` module based on a specific attribute. For example, group hosts dynamically based on their operating system and run different tasks on each without defining such groups in the inventory.
    ```
    - name: Gather facts from all hosts
      hosts: all
      tasks:
        - name: Classify hosts depending on their OS distribution
          group_by:
            key: OS_{{ ansible_facts['distribution'] }}

    # Only for the Ubuntu hosts
    - hosts: OS_Ubuntu
      tasks:
        - # tasks that only happen on Ubuntu go here

    # Only for the CentOS hosts
    - hosts: OS_CentOS
      tasks:
        - # tasks that only happen on CentOS go here
    ```
-   **Plays & Playbooks Best Practices**
    - Keep it as simple as possible: Try to keep your tasks simple. There are many options and nested structures in Ansible, and by combining lots of features, you can end up with fairly complex setups. Spending some time simplifying your Ansible artifacts pays off in the long term.
    - Always give descriptive names to your tasks, plays, and playbooks: Choose names that help you and others quickly understand the artifact’s functionality and purpose.
    - Strive for readability: Use 2-space indentation and add blank lines between tasks to increase readability.
    - Use comments when necessary: There will be times when the task definition won’t be enough to explain the whole situation, so feel free to use comments for more complex parts of playbooks.
    - Always mention the state of tasks explicitly: Many modules have a default state that allows us to skip the state parameter. It’s always better to be explicit in these cases to avoid confusion.
    - Place every task argument in its own separate line:  This point is in line with the general approach of striving for readability in our Ansible files. Check the examples below.
    This works but isn’t readable enough:
    ```
    - name: Add the user {{ username }}
      ansible.builtin.user: name={{ username }} state=present uid=999999 generate_ssh_key=yes
      become: yes
    ```
    Use this syntax instead, which improves a lot the readability and understandability of the tasks and their arguments:
    ```
    - name: Add the user {{ username }}
      ansible.builtin.user:
        name: "{{ username }}"
        state: present
        uid: 999999
        generate_ssh_key: yes
      become: yes
    ```
    - Use top-level playbooks to orchestrate other lower-level playbooks: You can logically group tasks, plays, and roles into low-level playbooks and use other top-level playbooks to import them and set up an orchestration layer according to your needs.
    - Use block syntax to group tasks: Tasks that relate to each other and share common attributes or tags can be grouped using the block option. Another advantage of this option is easier rollbacks for tasks under the same block.
    ```
    - name: Install, configure, and start an Nginx web server
      block:
        - name: Update and upgrade apt
          ansible.builtin.apt:
            update_cache: yes
            cache_valid_time: 3600
            upgrade: yes

        - name: "Install Nginx"
          ansible.builtin.apt:
            name: nginx
            state: present

        - name: Copy the Nginx configuration file to the host
          template:
            src: templates/nginx.conf.j2
            dest: /etc/nginx/sites-available/default

        - name: Create link to the new config to enable it
          file:
            dest: /etc/nginx/sites-enabled/default
            src: /etc/nginx/sites-available/default
            state: link

        - name: Create Nginx directory
          ansible.builtin.file:
            path: /home/ubuntu/nginx
            state: directory

        - name: Copy index.html to the Nginx directory
          copy:
            src: files/index.html
            dest: /home/ubuntu/nginx/index.html
          notify: Restart the Nginx service
      when: ansible_facts['distribution'] == 'Ubuntu'
      tags: nginx
      become: true
      become_user: root
    ```
    - Use handlers for tasks that should be triggered: Handlers allow a task to be executed after something has changed. This handler will be triggered when there are changes to index.html from the above example.
    ```
    handlers:
      - name: Restart the Nginx service
        service:
          name: nginx
          state: restarted
        become: true
        become_user: root
    ```
-   **Variables Best Practices**
    -   Use `_` separators for variable names (e.g., `my_var_name`).
    -   All strings containing special characters should be enclosed in double quotes.
    - Always provide sane defaults for your variables: Set default values for all groups under group_vars/all. For every role, set default role variables in roles/<role_name>/defaults/main.yml.
    - Use groups_vars and host_vars directories: To keep your inventory file clean, prefer setting group and hosts variables in the groups_vars and host_vars directories.
    - Add the role’s name as a prefix to variables: Try to be explicit when defining variable names for your roles by adding a prefix with the role name.
    ```
    nginx_port: 80
    apache_port: 8080
    ```
    - Keep your variable’s setup simple: Many options are available for setting variables in Ansible. Using all of them isn’t probably the way to go unless you have specific needs. Pick the ones more appropriate for your use case and keep it as simple as possible.
    - Use vault to save sensitive data: vault files ensure sensitive information remains protected while keeping your setup straightforward and easy to manage with tools like grep.
    Use a group_vars/{group_name} to define all variables in the vars file, including placeholders for sensitive ones, and store actual sensitive data in the vault file, prefixed with vault_. Use Jinja2 syntax in the vars file to reference these encrypted variables
    ```
    group_vars/webservers/vars:
    db_host: "localhost"
    db_name: "webapp"
    db_user: "{{ vault_db_user }}"
    db_password: "{{ vault_db_password }}"

    group_vars/webservers/vault:
    vault_db_user: "secure_user"
    vault_db_password: "super_secret_password"

    # Encrypt the Vault File:
    ansible-vault encrypt group_vars/webservers/vault
    ```
    - Documents all variables, for example:
   ```
    # List of main Apache settings to be configured in apache.conf.
    # Each setting is defined as a dictionary with the following keys:
    # - `name`: The name of the Apache directive to configure (e.g.`LimitRequestLine`,`LimitRequestFieldSize`,).
    # - `value`: The value to assign to the Apache directive.
    # - `comment` (optional): A comment to include above the directive in the configuration file. Useful for documenting the purpose of the setting.
    # The name `ServerName` is reserved for use with the `apache_server_name` setting,
    # so it will be ignored to prevent conflicts.
    #
    # Example:
    # apache_main_settings:
    #   - name: LimitRequestLine
    #     value: 8190
    #     comment: |
    #       Maximum size of the request line in bytes.
    #       Helps prevent buffer overflow attacks.
    #   - name: LimitRequestFieldSize
    #     value: 8190
    #     comment: Maximum size of each HTTP request header field in bytes.
    # Type: List of Dictionaries
    apache_main_settings: []
    ```

-   **Modules Best Practices**
    - Keep local modules close to playbooks: Use each Ansible project’s ./library directory to store relevant custom modules. Playbooks that have a ./library directory relative to their path can directly reference any modules inside it.
    - Avoid command and shell modules: It’s considered a best practice to limit the usage of command and shell modules only when there isn’t another option. Instead, prefer specialized modules that provide idempotency and proper error handling. Read more about the Ansible shell module.
    - Specify module arguments when it makes sense: Default values can be omitted in many module arguments. To be more transparent and explicit, we can opt to specify some of these arguments, like the state in our playbook definitions.
    - Prefer multi-tasks in a module over loops: The most efficient way of defining a list of similar tasks, like installing packages, is to use multiple tasks in a single module.
    ```
    - name: Install Docker dependencies
      ansible.builtin.apt:
        name:
          - curl
          - ca-certificates
          - gnupg
          - lsb-release
        state: latest
    ```
    - Document and test your custom modules: Every custom module should include examples, explicitly document dependencies, and describe return responses. New modules should be tested thoroughly before releasing. You can create testing roles and playbooks to test your custom modules and validate different test cases.
-   **Roles Best Practices**
    - Follow the Ansible Galaxy Role Directory structure: Leverage the ansible-galaxy init <role_name> command to generate a default role directory layout according to Ansible Galaxy’s standards.
    - Keep your roles single-purposed: Each role should have a separate responsibility and distinct functionality to conform with the separation of concerns design principle. Separate your roles based on different functionalities or technical domains.
    - Try to limit role dependencies: By avoiding many dependencies in your roles, you can keep them loosely coupled, develop them independently, and use them without managing complex dependencies between them.
    - Prefer import_role or include_role: To better control the execution order of roles and tasks, prefer using import_role or include_role over the classic roles option.
    - Do your due diligence for Ansible Galaxy Roles: When downloading and using content and roles from Galaxy, do your due diligence, validate their content, and pick roles from trustworthy contributors.
    - Store Galaxy roles used locally: To avoid depending on the upstream of Ansible Galaxy, you can store any roles from Galaxy to your code repositories and manage them as part of your project.
-   **Execution and Deployments Best Practices and Tricks**
    - Test changes in staging first: Having a staging or testing environment to test your tasks before production is a great way to validate that your changes have the expected outcome.
    - Limit task execution to specific hosts: If you want to run a playbook against specific hosts, you can use the --limit flag.
    - Limit tasks execution to specific tasks based on tags: In case you need to run only specific tasks from a playbook based on tags, you can define which tags to be executed with the --tags flag.
    - Validate which tasks will run before executing: You can use the --list-tasks flag to confirm which tasks would be run without actually running them.
    - Validate against which hosts the playbook will run: You can use the --list-hosts flag to confirm which hosts will be affected by the playbook without running it.
    - Validate which changes will happen without making them: Leverage the --check flag to predict any changes that may occur. Combine it with --diff flag to show differences in changed files.
    - Start at a specific task: Use the --start-at-task flag to start executing your playbook at a particular task.
    - Use Rolling Updates to control the number of target machines: By default, Ansible attempts to run the play against all hosts in parallel. To achieve a rolling update setup, you can leverage the serial keyword. Using this keyword, you can define how many to hosts the changes can be performed in parallel.
    - Control playbook execution strategy: By default, Ansible finishes the execution of each task on all hosts before moving to the next task. If you wish to select another execution strategy


## General Workflow

1.  The user starts a task from `tasks.md`.
2.  The AI agent analyzes the task and workspace context.
3.  The AI agent explains the planned actions based on the task flow and context.
4.  The AI agent shows a side-by-side diff in vscode GUI if it's possible.
5.  The user approves the plan, and the changes are implemented using the appropriate tools.
6.  Real-time feedback and validation are provided.

## Ansible Development Workflow

1.  Initialize the repository if needed.
2.  Create a Collection if needed:
    -   Edit `galaxy.yml`.
    -   Add `CHANGELOG.md`.
    -   Edit `meta/runtime.yml`.
3.  Create a branch for the role if needed.
4.  Create a role if needed.
5.  Develop Content (Playbooks/Tasks/Templates):
    -   Create the tasks first, then the variables.
    -   Write tasks in YAML, adhering to style guidelines.
    -   Ensure tasks are idempotent.
    -   Use appropriate variables, templates, handlers, files, etc.
    -   Implement documentation and comments.
    -   For Jinja2 templates (.j2 files), always use `## {mark} ANSIBLE MANAGED FILE ##` header instead of `# {{ ansible_managed }}` to clearly mark files as managed by Ansible.
    -   **Check Mode Compatibility**: Ensure all tasks work properly in check mode (`--check`). Tasks that manage services or require installed packages should include `when: not ansible_check_mode` to prevent failures during dry-run validation.
    -   Update the role's `meta/main.yml` and `README.md`.
6.  Create Tests (Mandatory):
    -   Create a scenario for each role (molecule scenario should match the role name).
    -   Design and implement Molecule tests for the new role or feature.
    -   Test file organization: The test file name in `extensions/molecule/<role>/tests/` must match the task file name in `roles/<role>/tasks/`. For example:
        -   Task file: `roles/apache/tasks/configure.yml` → Test file: `extensions/molecule/apache/tests/configure.yml`
        -   Task file: `roles/apache/tasks/install.yml` → Test file: `extensions/molecule/apache/tests/install.yml`
    -   Define scenarios, platforms, verifiers, etc.
7.  Local Validation:
    -   Molecule should be executed from the extensions directory.
    -   Run a syntax check.
    -   Run the linter.
    -   **Test in Check Mode and Diff Mode (Mandatory)**: Always run playbooks with `--check --diff` flags to validate that the playbook works correctly in dry-run mode and shows accurate change predictions. This ensures:
        -   Tasks are properly guarded with `when: not ansible_check_mode` where needed (e.g., service management before package installation).
        -   The playbook provides accurate change previews without failing.
        -   Example: `ansible-playbook playbooks/webserver/web-server-setup.yml --check --diff`
8.  Review and Submit:
    -   Organize and review the changes.
    -   Present the solution and test results to the user.

Develop one role at time using serial and requirement-driven as Development strategy. Start with the highest priority requirement and complete the following four-step cycle for it. Only upon full completion of this cycle should you proceed to the next requirement:

Plan: Define the specific implementation tasks.
Code & Test: Write the code and its corresponding tests.
Verify: Provide local validation steps.
Next: Move to the next requirement.


