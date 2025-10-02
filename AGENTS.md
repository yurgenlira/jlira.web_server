# GEMINI.md for Ansible Collection

## Project Workflow
1. User requests changes or assistance
2. Gemini analyzes request and workspace context
3. Gemini explains planned actions before execution
4. VS Code shows side-by-side diff (if required)
5. I approve and changes are implemented using appropriate tools
6. Real-time feedback and validation

## Project and Style Guidelines
-   **Purpose:** This ansible collection is designed to automate the setup and management of Linux systems. It must be portable, idempotent, and well-documented.
-   **Generic Best Practices:**
    - Prefer YAML instead of JSON: Although Ansible allows JSON syntax, using YAML is preferred and improves the readability of files and projects.
    - Use consistent whitespaces: To separate things nicely and improve readability, consider leaving a blank line between blocks, tasks, or other components.
    - Use a consistent tagging strategy: Tagging is a powerful concept in Ansible since it allows us to group and manage tasks more granularly.  Tags provide us with the option to add fine-grained controls to the execution of tasks.
    - Add comments: When you think a further explanation is needed, feel free to add a comment explaining the purpose and the reason behind plays, tasks, variables, etc.
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


## Technology Stack & Architecture Decisions
### **Ansible Automation Platform**
- **Ansible Core**: Latest stable version for playbook execution
- **Ansible Galaxy**: Role and collection management
- **Ansible Vault**: Secrets and sensitive data encryption
- **Ansible Lint**: Code quality and best practices enforcement
- **Molecule**: Role testing and validation framework

### Architecture Design Principles
Role structure
```
roles
├── common			# this hierarchy represents a "role"
│   ├── defaults		#
│   │   └── main.yml	#  default lower priority variables for this role
│   ├── files		#
│   │   ├── bar.txt	#  files for use with the copy resource
│   │   └── foo.sh	#  script files for use with the script resource
│   ├── handlers		#
│   │   └── main.yml	#  handlers file
│   ├── meta		#
│   │   └── main.yml	#  role dependencies
│   ├── tasks		#
│   │   └── main.yml	#  tasks file can include smaller files if warranted
│   ├── templates		#
│   │   └── ntp.conf.j2	#  templates end in .j2
│   ├── vars		#
│   │   └── main.yml	#  variables associated with this role
│   ├── module_utils	# roles can also include custom module_utils
│   ├── library		# roles can also include custom modules
│   └── lookup_plugins 	# or other types of plugins, like lookup in this case
│
├── monitoring		# another role with the same structure
└── webtier			# another role with the same structure
```

Collection structure
```
my_collection
├── galaxy.yml
├── meta
│   └── runtime.yml
├── plugins
│   └── README.md
├── README.md
├── playbooks
└── roles
    ├── role1
    └── role2
```

## **Gated Execution Protocols**

### <PROTOCOL:EXPLAIN>

This protocol is for when I ask you to explain a concept, module, or code snippet. Your goal is to provide a clear, educational response, follow these guidelines:

-   **Start with a High-Level Summary:** Begin with a single sentence that concisely explains the main purpose. For example, "This module manages user accounts on a system."
-   **Use a Structured Format:** Use a bulleted list to break down the explanation.
    -   **Purpose:** Detail what the component does and why it's used in the project.
    -   **Key Parameters:** List the most important parameters or options and briefly describe their function.
    -   **Example:** Provide a small, commented code snippet that shows how to use the component.
-   **Link to Documentation:** If applicable, mention where to find the official Ansible documentation for the module or plugin.
-   **Stay in Context:** All explanations must be relevant to the current project and its established conventions. Do not provide generic, unrelated information.

---

### <PROTOCOL:PLAN>

This protocol is for when I ask you to propose a solution or create a new component. Your response must be a structured plan, not code.

1.  **High-Level Overview:** Begin with a brief summary of the proposed solution.
2.  **Actionable Steps:** Use a numbered list for the step-by-step plan. Each step should be clear and distinct.
    -   Example steps: "Create directory structure," "Write module code," "Add documentation," "Create test playbook."
3.  **No Code:** Do not include any code snippets in this protocol. Focus solely on the strategy.

---

### <PROTOCOL:IMPLEMENT>

When implementing code, strictly follow these guidelines.

-   **Ansible Best Practices:** Follow the best practices from 'Project and Style Guidelines' section
-   **Documentation:** Every new role, module, or plugin must have a `README.md` file that explains its purpose, parameters, and examples of usage.
-   **Testing:** Create a `tests/` directory with `test.yml` playbooks to validate the functionality of new components.

---

### <PROTOCOL:IMPLEMENT>

This protocol is for when I have approved a plan and ask you to write or modify code. Your output should be the requested code, following all project rules.

1.  **Strict Adherence:** All code must strictly follow the style, security, and idempotence rules defined in the "Project and Style Guidelines" section.
2.  **Documentation:** Any new module or role must include a `README.md` file.
3.  **Idempotence Check:** Before generating the code, verify that the proposed solution is idempotent.

---
