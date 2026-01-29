# Ansible Core Concepts – High Retrieval & Teaching Cheat Sheet

This document is designed for **quick revision, recall, and teaching**. It assumes you already know Ansible concepts but need fast retrieval for interviews, KT sessions, or explaining to juniors. Each section follows a fixed pattern with minimal examples, key syntax, and ready-made speaking lines.

---

## Table of Contents

1. [Left-Side Topic Map (Overview)](#left-side-topic-map-overview)
2. [Ansible Architecture](#ansible-architecture)
3. [Inventory (INI & YAML)](#inventory-ini--yaml)
4. [Static vs Dynamic Inventory](#static-vs-dynamic-inventory)
5. [ansible.cfg](#ansiblecfg)
6. [Ad-hoc Commands](#ad-hoc-commands)
7. [Playbooks](#playbooks)
8. [Plays vs Tasks](#plays-vs-tasks)
9. [Modules](#modules)
10. [Variables](#variables)
11. [Variable Precedence](#variable-precedence)
12. [Facts & Gathering Facts](#facts--gathering-facts)
13. [Conditionals (when)](#conditionals-when)
14. [Loops](#loops)
15. [Handlers](#handlers)
16. [Templates (Jinja2)](#templates-jinja2)
17. [Files & Copy](#files--copy)
18. [Roles](#roles)
19. [Role Directory Structure](#role-directory-structure)
20. [Ansible Galaxy](#ansible-galaxy)
21. [Vault](#vault)
22. [Tags](#tags)
23. [Error Handling (ignore_errors, failed_when)](#error-handling-ignore_errors-failed_when)
24. [Async & Poll](#async--poll)
25. [Delegation (delegate_to, local_action)](#delegation-delegate_to-local_action)
26. [Privilege Escalation (become)](#privilege-escalation-become)
27. [Ansible Best Practices](#ansible-best-practices)
28. [Closing Notes](#closing-notes)

---

## Left-Side Topic Map (Overview)

| Topic | Trigger Idea (1 line) | Most Used Command / Keyword |
|-------|----------------------|----------------------------|
| Ansible Architecture | Control node pushes config to managed nodes over SSH | `ansible --version` |
| Inventory (INI & YAML) | List of target hosts grouped logically | `ansible-inventory --list` |
| Static vs Dynamic Inventory | Static files vs scripts fetching hosts dynamically | `dynamic inventory plugin` |
| ansible.cfg | Configuration file controlling Ansible behavior | `ansible-config view` |
| Ad-hoc Commands | One-liner commands without playbooks | `ansible all -m ping` |
| Playbooks | YAML files defining automation tasks declaratively | `ansible-playbook site.yml` |
| Plays vs Tasks | Play targets hosts; tasks are individual actions | `hosts:` and `tasks:` |
| Modules | Reusable units performing specific actions | `yum`, `copy`, `service` |
| Variables | Dynamic values passed to playbooks | `vars:` and `{{ variable }}` |
| Variable Precedence | Order determining which variable value wins | Extra vars > Play vars > Inventory |
| Facts & Gathering Facts | System info auto-collected from managed nodes | `gather_facts: true` |
| Conditionals (when) | Execute tasks based on conditions | `when: ansible_os_family == "RedHat"` |
| Loops | Repeat tasks with different values | `loop:` or `with_items:` |
| Handlers | Tasks triggered by notifications from other tasks | `notify:` and `handlers:` |
| Templates (Jinja2) | Dynamic file generation using Jinja2 syntax | `template:` module |
| Files & Copy | Transfer files from control to managed nodes | `copy:` and `file:` modules |
| Roles | Reusable, organized structure for playbooks | `roles/` directory |
| Role Directory Structure | Standard layout: tasks, handlers, vars, templates | `tasks/main.yml` |
| Ansible Galaxy | Community hub for sharing roles | `ansible-galaxy install` |
| Vault | Encrypt sensitive data like passwords | `ansible-vault encrypt` |
| Tags | Selectively run parts of playbooks | `--tags` and `tags:` |
| Error Handling | Control task failure behavior | `ignore_errors`, `failed_when` |
| Async & Poll | Run long tasks asynchronously without blocking | `async:` and `poll:` |
| Delegation | Run tasks on different hosts | `delegate_to:` |
| Privilege Escalation (become) | Execute tasks with elevated privileges | `become: true` |
| Ansible Best Practices | Idempotency, roles, version control, testing | Use roles, limit scope |

---

## Ansible Architecture
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Agentless automation – control node pushes configuration to managed nodes over SSH."

### 1. What it is (Simple Definition)

- Ansible is an agentless automation tool using SSH for communication
- Control node runs playbooks and connects to managed nodes
- No agent installation required on target systems

### 2. Why / When to Use

- Automate repetitive IT tasks (provisioning, configuration, deployment)
- Manage infrastructure as code
- Simple setup compared to agent-based tools
- Ideal for configuration management, orchestration, and application deployment

### 3. Key Syntax / Keywords

- Control node
- Managed nodes
- Inventory
- Modules
- Playbooks
- SSH / WinRM

### 4. Minimal Playbook Example

```yaml
---
- name: Basic architecture demonstration
  hosts: all
  tasks:
    - name: Ping all managed nodes
      ping:
```

### 5. Example Scenario

- Control node (laptop/server) has Ansible installed
- Inventory file lists 50 web servers
- Run playbook from control node
- Ansible SSHs to all 50 servers simultaneously and executes tasks

### 6. Common Interview / KT Line

"Ansible's agentless architecture means we only need SSH access – no agents to maintain on thousands of servers."

### 7. Closing Line Trigger

"In short, Ansible's architecture keeps things simple with SSH-based, push-mode automation."

---

## Inventory (INI & YAML)
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Inventory is the list of managed hosts, organized into groups."

### 1. What it is (Simple Definition)

- Inventory defines target hosts for Ansible operations
- Can be written in INI or YAML format
- Supports grouping hosts for logical organization

### 2. Why / When to Use

- Organize servers by function (web, db, app)
- Apply playbooks to specific host groups
- Define host-specific or group-specific variables
- Centralize infrastructure host management

### 3. Key Syntax / Keywords

- `[groupname]`
- `ansible_host`
- `ansible_user`
- `ansible_port`
- `children` (for nested groups)

### 4. Minimal Playbook Example

**INI Format (inventory.ini):**
```ini
[web]
web1.example.com
web2.example.com

[db]
db1.example.com ansible_host=192.168.1.10 ansible_user=admin

[prod:children]
web
db
```

**YAML Format (inventory.yml):**
```yaml
all:
  children:
    web:
      hosts:
        web1.example.com:
        web2.example.com:
    db:
      hosts:
        db1.example.com:
          ansible_host: 192.168.1.10
          ansible_user: admin
```

### 5. Example Scenario

- Three web servers need nginx installed
- Two database servers need mysql configured
- Create groups `[web]` and `[db]` in inventory
- Run playbook targeting specific groups: `ansible-playbook site.yml --limit web`

### 6. Common Interview / KT Line

"Inventory files let us organize hundreds of servers into logical groups, making it easy to target specific environments or tiers."

### 7. Closing Line Trigger

"In short, inventory is your infrastructure map in Ansible."

---

## Static vs Dynamic Inventory
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Static inventory is a fixed file; dynamic inventory fetches hosts programmatically."

### 1. What it is (Simple Definition)

- **Static inventory**: Manually maintained INI or YAML files
- **Dynamic inventory**: Scripts or plugins querying cloud APIs, CMDBs, etc.
- Dynamic inventory adapts to changing infrastructure automatically

### 2. Why / When to Use

- Static: Small, stable environments
- Dynamic: Cloud environments (AWS, Azure, GCP) with auto-scaling
- Dynamic: Large infrastructures where manual updates are impractical
- Dynamic: Integration with external systems (ServiceNow, Terraform)

### 3. Key Syntax / Keywords

- Static: `hosts` file
- Dynamic: `inventory plugin`, `script`
- AWS EC2, Azure RM, GCP plugins
- `ansible-inventory --list`

### 4. Minimal Playbook Example

**Static Inventory Usage:**
```bash
ansible-playbook -i inventory.ini site.yml
```

**Dynamic Inventory Usage (AWS EC2):**
```yaml
# inventory_aws_ec2.yml
plugin: aws_ec2
regions:
  - us-east-1
keyed_groups:
  - key: tags.Environment
    prefix: env
```

```bash
ansible-playbook -i inventory_aws_ec2.yml site.yml
```

### 5. Example Scenario

- AWS environment with auto-scaling groups
- Servers created/destroyed dynamically
- Use AWS EC2 dynamic inventory plugin
- Ansible automatically discovers running instances based on tags
- No manual inventory updates needed

### 6. Common Interview / KT Line

"Dynamic inventory is essential for cloud environments where servers come and go – it queries the cloud provider API in real-time."

### 7. Closing Line Trigger

"In short, static inventory works for fixed setups, while dynamic inventory adapts to cloud-scale infrastructure."

---

## ansible.cfg
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Configuration file controlling Ansible behavior and defaults."

### 1. What it is (Simple Definition)

- `ansible.cfg` sets default behavior for Ansible operations
- Defines inventory location, SSH settings, privilege escalation, etc.
- Multiple locations checked in precedence order

### 2. Why / When to Use

- Customize default inventory path
- Configure SSH connection parameters (timeout, retries)
- Set default privilege escalation method
- Control logging, callback plugins, and output formatting

### 3. Key Syntax / Keywords

- `[defaults]`
- `inventory`
- `remote_user`
- `host_key_checking`
- `roles_path`
- `[privilege_escalation]`

### 4. Minimal Playbook Example

**ansible.cfg:**
```ini
[defaults]
inventory = ./inventory.ini
remote_user = ansible
host_key_checking = False
retry_files_enabled = False
roles_path = ./roles:/usr/share/ansible/roles

[privilege_escalation]
become = True
become_method = sudo
become_user = root
```

### 5. Example Scenario

- Working on a project with custom inventory path
- Disable SSH host key checking for lab environment
- Set default remote user to `ansible`
- Place `ansible.cfg` in project directory
- Ansible automatically picks up these settings

### 6. Common Interview / KT Line

"The ansible.cfg file lets us standardize configuration across teams without passing command-line options every time."

### 7. Closing Line Trigger

"In short, ansible.cfg centralizes configuration, making playbook execution consistent and predictable."

---

## Ad-hoc Commands
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "One-liner commands for quick tasks without writing playbooks."

### 1. What it is (Simple Definition)

- Ad-hoc commands execute single tasks from command line
- Useful for quick operations, testing, or troubleshooting
- Not reusable like playbooks but fast for immediate needs

### 2. Why / When to Use

- Quick checks (ping all servers, check disk space)
- One-time operations (restart service, copy file)
- Testing inventory or module behavior
- Emergency fixes without writing full playbooks

### 3. Key Syntax / Keywords

- `ansible`
- `-m` (module)
- `-a` (arguments)
- `-i` (inventory)
- `--become`
- `-b` (shorthand for become)

### 4. Minimal Playbook Example

```bash
# Ping all hosts
ansible all -m ping

# Check disk space
ansible web -m shell -a "df -h"

# Install package
ansible db -m yum -a "name=mysql-server state=present" --become

# Restart service
ansible web -m service -a "name=nginx state=restarted" --become

# Copy file
ansible all -m copy -a "src=/tmp/file.txt dest=/tmp/file.txt"

# Gather facts
ansible web -m setup
```

### 5. Example Scenario

- Web servers suddenly slow
- Quick check disk space: `ansible web -m shell -a "df -h"`
- See high usage on `/var/log`
- Clean logs immediately: `ansible web -m shell -a "rm -f /var/log/old.log" --become`
- No playbook needed for this emergency fix

### 6. Common Interview / KT Line

"Ad-hoc commands are perfect for quick checks and one-time fixes when you don't want to write a full playbook."

### 7. Closing Line Trigger

"In short, ad-hoc commands are Ansible's quick-action tool for immediate tasks."

---

## Playbooks
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Playbooks are YAML files defining declarative automation tasks."

### 1. What it is (Simple Definition)

- Playbooks contain one or more plays written in YAML
- Each play targets hosts and defines tasks to execute
- Reusable, version-controlled automation scripts

### 2. Why / When to Use

- Automate complex, multi-step processes
- Ensure consistency across environments
- Document infrastructure as code
- Create repeatable, auditable deployments

### 3. Key Syntax / Keywords

- `---` (YAML start)
- `name`
- `hosts`
- `tasks`
- `become`
- `vars`
- `handlers`

### 4. Minimal Playbook Example

```yaml
---
- name: Install and configure web server
  hosts: web
  become: true
  vars:
    http_port: 80
  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present

    - name: Start nginx service
      service:
        name: nginx
        state: started
        enabled: true

    - name: Copy index.html
      copy:
        src: files/index.html
        dest: /usr/share/nginx/html/index.html
```

### 5. Example Scenario

- Need to deploy web application to 20 servers
- Write playbook with tasks: install packages, copy files, start services
- Run once: `ansible-playbook deploy.yml`
- All 20 servers configured identically
- Playbook version-controlled in Git for future use

### 6. Common Interview / KT Line

"Playbooks are the heart of Ansible – they let us define infrastructure state declaratively and execute it repeatedly."

### 7. Closing Line Trigger

"In short, playbooks turn manual runbooks into automated, consistent, repeatable workflows."

---

## Plays vs Tasks
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Play targets hosts and contains tasks; tasks are individual actions."

### 1. What it is (Simple Definition)

- A **play** maps hosts to tasks with specific settings
- A **task** is a single action (install package, copy file, etc.)
- One playbook can contain multiple plays

### 2. Why / When to Use

- Organize automation logically (web servers get play 1, db servers get play 2)
- Apply different settings per play (different users, privilege levels)
- Sequence operations across different host groups
- Separate concerns within one playbook

### 3. Key Syntax / Keywords

- **Play level**: `name`, `hosts`, `become`, `vars`, `tasks`
- **Task level**: `name`, module name, module parameters

### 4. Minimal Playbook Example

```yaml
---
# Play 1: Configure web servers
- name: Setup web servers
  hosts: web
  become: true
  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present

# Play 2: Configure database servers
- name: Setup database servers
  hosts: db
  become: true
  tasks:
    - name: Install mysql
      yum:
        name: mysql-server
        state: present
```

### 5. Example Scenario

- Deploying a three-tier application
- Play 1 targets load balancers: install HAProxy
- Play 2 targets app servers: install application code
- Play 3 targets database servers: configure MySQL
- Single playbook orchestrates entire deployment with three plays

### 6. Common Interview / KT Line

"Think of plays as chapters targeting different host groups, and tasks as the individual steps within each chapter."

### 7. Closing Line Trigger

"In short, plays organize tasks for specific host groups, creating structured multi-tier automation."

---

## Modules
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Modules are reusable units performing specific actions like installing packages or copying files."

### 1. What it is (Simple Definition)

- Modules are Ansible's building blocks for automation
- Each module performs a specific function (yum, copy, service, etc.)
- Written in Python, executed on managed nodes
- Idempotent by design (safe to run multiple times)

### 2. Why / When to Use

- Abstract complexity of system commands
- Provide idempotent operations
- Support all major platforms (Linux, Windows, network devices)
- Over 3000+ modules available covering diverse needs

### 3. Key Syntax / Keywords

- `yum` / `apt` / `package` (package management)
- `copy` / `file` (file operations)
- `service` / `systemd` (service management)
- `user` / `group` (user management)
- `shell` / `command` (run commands)
- `template` (Jinja2 templates)

### 4. Minimal Playbook Example

```yaml
---
- name: Module examples
  hosts: web
  become: true
  tasks:
    - name: Install package (yum module)
      yum:
        name: httpd
        state: present

    - name: Start service (service module)
      service:
        name: httpd
        state: started
        enabled: true

    - name: Copy file (copy module)
      copy:
        src: /tmp/file.txt
        dest: /var/www/html/file.txt
        mode: '0644'

    - name: Create user (user module)
      user:
        name: webadmin
        state: present
        shell: /bin/bash
```

### 5. Example Scenario

- Need to install nginx on CentOS servers
- Use `yum` module instead of `shell: yum install nginx -y`
- Module checks if nginx already installed (idempotent)
- Module handles errors gracefully
- Cleaner, safer, more reliable than shell commands

### 6. Common Interview / KT Line

"Modules are Ansible's abstraction layer – they handle platform differences and ensure idempotency automatically."

### 7. Closing Line Trigger

"In short, modules are purpose-built tools making automation safe, clean, and idempotent."

---

## Variables
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Variables store dynamic values used throughout playbooks."

### 1. What it is (Simple Definition)

- Variables hold values that can change across environments
- Referenced using Jinja2 syntax: `{{ variable_name }}`
- Defined in multiple places: playbooks, inventory, files, command line

### 2. Why / When to Use

- Avoid hardcoding values (ports, paths, usernames)
- Reuse playbooks across environments (dev, staging, prod)
- Parameterize automation for flexibility
- Store environment-specific configuration

### 3. Key Syntax / Keywords

- `vars:`
- `vars_files:`
- `{{ variable_name }}`
- `register:`
- `set_fact:`
- `extra_vars` (`-e`)

### 4. Minimal Playbook Example

```yaml
---
- name: Variable usage examples
  hosts: web
  become: true
  vars:
    http_port: 80
    app_name: myapp
  tasks:
    - name: Install {{ app_name }}
      yum:
        name: "{{ app_name }}"
        state: present

    - name: Configure port
      lineinfile:
        path: /etc/myapp/config.conf
        line: "port={{ http_port }}"

    - name: Register command output
      command: hostname
      register: server_hostname

    - name: Display hostname
      debug:
        msg: "Server hostname is {{ server_hostname.stdout }}"
```

**External variable file (vars.yml):**
```yaml
http_port: 8080
app_name: webapp
```

**Using external file:**
```yaml
- name: Use external vars
  hosts: web
  vars_files:
    - vars.yml
  tasks:
    - name: Show port
      debug:
        msg: "Port is {{ http_port }}"
```

### 5. Example Scenario

- Same playbook for dev (port 8080) and prod (port 80)
- Define `http_port` variable differently per environment
- Dev: `ansible-playbook site.yml -e "http_port=8080"`
- Prod: `ansible-playbook site.yml -e "http_port=80"`
- No playbook changes needed

### 6. Common Interview / KT Line

"Variables make playbooks flexible and reusable – same code works across environments by just changing variable values."

### 7. Closing Line Trigger

"In short, variables parameterize playbooks, enabling environment-agnostic automation."

---

## Variable Precedence
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Variable precedence determines which value wins when defined in multiple places."

### 1. What it is (Simple Definition)

- Multiple locations can define the same variable
- Ansible follows precedence rules to determine final value
- Higher precedence sources override lower ones

### 2. Why / When to Use

- Understand which variable value will be used
- Debug unexpected behavior when variables conflict
- Design playbooks with proper variable layering
- Override defaults safely

### 3. Key Syntax / Keywords

**Precedence order (low to high):**
1. Role defaults
2. Inventory file/script group vars
3. Inventory group_vars/all
4. Playbook group_vars/all
5. Inventory group_vars/*
6. Playbook group_vars/*
7. Inventory file/script host vars
8. Inventory host_vars/*
9. Playbook host_vars/*
10. Host facts
11. Registered vars
12. Set_facts
13. Play vars
14. Play vars_files
15. Role vars
16. Block vars
17. Task vars
18. Extra vars (`-e`)

### 4. Minimal Playbook Example

```yaml
---
- name: Variable precedence demo
  hosts: web
  vars:
    app_port: 8080  # Play vars
  tasks:
    - name: Show port (play vars)
      debug:
        msg: "Port is {{ app_port }}"

    - name: Override with task vars
      debug:
        msg: "Port is {{ app_port }}"
      vars:
        app_port: 9090  # Task vars (higher precedence)

# Running with extra vars (highest precedence):
# ansible-playbook demo.yml -e "app_port=7070"
```

### 5. Example Scenario

- Role defines `app_port: 80` in defaults
- Inventory defines `app_port: 8080` for staging group
- Playbook defines `app_port: 8888`
- At runtime, pass `-e app_port=9090`
- Final value used: `9090` (extra vars have highest precedence)

### 6. Common Interview / KT Line

"Extra vars always win – that's the key to remember for variable precedence in Ansible."

### 7. Closing Line Trigger

"In short, understanding precedence prevents surprises when the same variable appears in multiple places."

---

## Facts & Gathering Facts
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Facts are system information auto-collected from managed nodes."

### 1. What it is (Simple Definition)

- Facts are variables containing system info (OS, IP, memory, etc.)
- Automatically gathered at play start by default
- Accessed using `ansible_facts` or `ansible_*` variables

### 2. Why / When to Use

- Make decisions based on system properties (OS type, version)
- Use system info in templates (hostname, IP address)
- Conditional task execution based on facts
- Disable gathering for faster playbook execution when not needed

### 3. Key Syntax / Keywords

- `gather_facts: true/false`
- `ansible_facts`
- `ansible_os_family`
- `ansible_distribution`
- `ansible_hostname`
- `ansible_default_ipv4.address`
- `setup` module

### 4. Minimal Playbook Example

```yaml
---
- name: Using facts
  hosts: all
  gather_facts: true
  tasks:
    - name: Show OS family
      debug:
        msg: "OS is {{ ansible_facts['os_family'] }}"

    - name: Install package based on OS
      yum:
        name: httpd
        state: present
      when: ansible_facts['os_family'] == "RedHat"

    - name: Install package on Debian
      apt:
        name: apache2
        state: present
      when: ansible_facts['os_family'] == "Debian"

    - name: Display IP address
      debug:
        msg: "Server IP is {{ ansible_default_ipv4.address }}"
```

**Disable fact gathering:**
```yaml
- name: Fast playbook without facts
  hosts: all
  gather_facts: false
  tasks:
    - name: Simple task
      ping:
```

### 5. Example Scenario

- Playbook needs to work on both CentOS and Ubuntu
- Gather facts to detect OS family
- Use `when: ansible_os_family == "RedHat"` for yum
- Use `when: ansible_os_family == "Debian"` for apt
- Single playbook works across different Linux distributions

### 6. Common Interview / KT Line

"Facts give us automatic system discovery – we can write OS-agnostic playbooks using fact-based conditionals."

### 7. Closing Line Trigger

"In short, facts provide runtime intelligence about managed systems, enabling smart automation decisions."

---

## Conditionals (when)
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Execute tasks conditionally using the 'when' statement."

### 1. What it is (Simple Definition)

- `when` clause controls whether a task runs
- Evaluates expressions to true/false
- Based on variables, facts, or registered results

### 2. Why / When to Use

- Skip tasks on certain OS types or versions
- Run tasks only when specific conditions met
- Handle environment-specific logic
- React to previous task results

### 3. Key Syntax / Keywords

- `when:`
- `ansible_facts`
- `is defined`
- `in`
- `==`, `!=`, `>`, `<`
- `and`, `or`, `not`

### 4. Minimal Playbook Example

```yaml
---
- name: Conditional execution
  hosts: all
  become: true
  tasks:
    - name: Install nginx on RedHat
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat"

    - name: Install nginx on Debian
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"

    - name: Only on production servers
      service:
        name: firewalld
        state: started
      when: inventory_hostname in groups['production']

    - name: Check if file exists
      stat:
        path: /etc/app/config.conf
      register: config_file

    - name: Create config if missing
      file:
        path: /etc/app/config.conf
        state: touch
      when: not config_file.stat.exists

    - name: Multiple conditions
      debug:
        msg: "This is production RedHat server"
      when:
        - ansible_os_family == "RedHat"
        - inventory_hostname in groups['production']
```

### 5. Example Scenario

- Playbook runs on mixed environment (CentOS and Ubuntu)
- Some tasks only relevant for RedHat family
- Use `when: ansible_os_family == "RedHat"` to skip Ubuntu
- Another task only for servers with >16GB RAM
- Use `when: ansible_memtotal_mb > 16000`

### 6. Common Interview / KT Line

"The when statement makes playbooks intelligent – tasks only run when conditions are met, reducing errors and wasted effort."

### 7. Closing Line Trigger

"In short, conditionals enable smart, context-aware automation that adapts to different scenarios."

---

## Loops
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Loops repeat tasks with different values using 'loop' or 'with_items'."

### 1. What it is (Simple Definition)

- Loops iterate over lists of items
- Execute same task multiple times with different parameters
- Modern syntax uses `loop`, older syntax uses `with_*`

### 2. Why / When to Use

- Install multiple packages in one task
- Create multiple users or files
- Iterate over configuration items
- Reduce playbook repetition

### 3. Key Syntax / Keywords

- `loop:`
- `{{ item }}`
- `with_items:` (legacy)
- `with_dict:`
- `with_fileglob:`
- `loop_control:`

### 4. Minimal Playbook Example

```yaml
---
- name: Loop examples
  hosts: web
  become: true
  tasks:
    - name: Install multiple packages
      yum:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - git
        - vim

    - name: Create multiple users
      user:
        name: "{{ item }}"
        state: present
      loop:
        - alice
        - bob
        - charlie

    - name: Loop over dictionary
      debug:
        msg: "User {{ item.key }} has ID {{ item.value }}"
      loop:
        - { key: 'alice', value: 1001 }
        - { key: 'bob', value: 1002 }

    - name: Loop with index
      debug:
        msg: "Item {{ item }} at index {{ idx }}"
      loop:
        - apple
        - banana
        - cherry
      loop_control:
        index_var: idx

    - name: Copy multiple files
      copy:
        src: "{{ item }}"
        dest: "/tmp/{{ item }}"
      with_fileglob:
        - "files/*.conf"
```

### 5. Example Scenario

- Need to install 10 packages on web servers
- Without loop: 10 separate tasks
- With loop: Single task with list of packages
- Cleaner playbook, easier to maintain
- Add/remove packages by updating list

### 6. Common Interview / KT Line

"Loops eliminate repetitive tasks – instead of writing 10 tasks to install 10 packages, write one task with a loop."

### 7. Closing Line Trigger

"In short, loops make playbooks DRY (Don't Repeat Yourself) by iterating efficiently."

---

## Handlers
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Handlers are tasks triggered by notifications, typically for service restarts."

### 1. What it is (Simple Definition)

- Handlers are special tasks that run only when notified
- Execute at end of play, not immediately
- Run only once even if notified multiple times
- Typically used for service restarts after config changes

### 2. Why / When to Use

- Restart services only when configuration changes
- Avoid unnecessary service restarts
- Ensure services reload after all config updates
- Clean separation of actions and reactions

### 3. Key Syntax / Keywords

- `handlers:`
- `notify:`
- `listen:`
- `meta: flush_handlers` (force immediate execution)

### 4. Minimal Playbook Example

```yaml
---
- name: Handler demonstration
  hosts: web
  become: true
  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present

    - name: Copy nginx config
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx

    - name: Copy site config
      copy:
        src: files/site.conf
        dest: /etc/nginx/conf.d/site.conf
      notify: Restart nginx

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

**Using listen:**
```yaml
tasks:
  - name: Update config
    copy:
      src: app.conf
      dest: /etc/app.conf
    notify: Reload app

handlers:
  - name: Restart application
    service:
      name: myapp
      state: restarted
    listen: Reload app

  - name: Clear cache
    file:
      path: /var/cache/app
      state: absent
    listen: Reload app
```

### 5. Example Scenario

- Updating 5 nginx configuration files
- Each task notifies "Restart nginx" handler
- Handler runs only once at play end
- If no configs changed, handler doesn't run
- Efficient – avoids multiple unnecessary restarts

### 6. Common Interview / KT Line

"Handlers ensure services restart only when needed and only once, even if multiple tasks make changes."

### 7. Closing Line Trigger

"In short, handlers provide efficient, conditional service management triggered by configuration changes."

---

## Templates (Jinja2)
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Templates generate dynamic configuration files using Jinja2 syntax."

### 1. What it is (Simple Definition)

- Templates are files with placeholders for variables
- Use Jinja2 templating language
- Processed by `template` module to create customized files
- Enables dynamic, environment-specific configuration

### 2. Why / When to Use

- Generate config files with environment-specific values
- Avoid maintaining separate configs per environment
- Use conditionals and loops in configuration files
- Single template serves dev, staging, and prod

### 3. Key Syntax / Keywords

- `template:` module
- `{{ variable }}`
- `{% if condition %}`
- `{% for item in list %}`
- `{% endif %}`, `{% endfor %}`
- `.j2` file extension

### 4. Minimal Playbook Example

**Template file (nginx.conf.j2):**
```jinja2
server {
    listen {{ http_port }};
    server_name {{ server_name }};

    location / {
        proxy_pass http://{{ backend_host }}:{{ backend_port }};
    }

    {% if enable_ssl %}
    ssl_certificate {{ ssl_cert_path }};
    ssl_certificate_key {{ ssl_key_path }};
    {% endif %}
}
```

**Playbook:**
```yaml
---
- name: Deploy nginx config from template
  hosts: web
  become: true
  vars:
    http_port: 80
    server_name: example.com
    backend_host: localhost
    backend_port: 8080
    enable_ssl: false
  tasks:
    - name: Generate nginx config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/mysite.conf
      notify: Restart nginx

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

### 5. Example Scenario

- Application config needs different database hosts per environment
- Create template: `db_host: {{ database_host }}`
- Dev vars: `database_host: dev-db.local`
- Prod vars: `database_host: prod-db.local`
- Same template generates environment-specific configs

### 6. Common Interview / KT Line

"Templates let us maintain one source file that generates customized configs for different environments using variables."

### 7. Closing Line Trigger

"In short, templates eliminate config duplication and enable truly dynamic infrastructure as code."

---

## Files & Copy
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Copy module transfers files; file module manages file properties."

### 1. What it is (Simple Definition)

- `copy` module transfers files from control node to managed nodes
- `file` module manages file/directory properties (permissions, ownership, state)
- Both modules are idempotent and commonly used

### 2. Why / When to Use

- Deploy application files, scripts, or configs
- Ensure files exist with correct permissions
- Create directories or symbolic links
- Manage file ownership and modes

### 3. Key Syntax / Keywords

- `copy:` - `src`, `dest`, `mode`, `owner`, `group`
- `file:` - `path`, `state`, `mode`, `owner`, `recurse`
- `state: file/directory/link/absent/touch`

### 4. Minimal Playbook Example

```yaml
---
- name: File and copy operations
  hosts: web
  become: true
  tasks:
    - name: Copy file from control node
      copy:
        src: files/app.conf
        dest: /etc/app/app.conf
        owner: root
        group: root
        mode: '0644'

    - name: Copy with content inline
      copy:
        content: |
          # Auto-generated config
          port=8080
          host=0.0.0.0
        dest: /etc/myapp/config.ini
        mode: '0640'

    - name: Create directory
      file:
        path: /var/www/myapp
        state: directory
        owner: nginx
        group: nginx
        mode: '0755'

    - name: Create symbolic link
      file:
        src: /etc/nginx/sites-available/mysite
        dest: /etc/nginx/sites-enabled/mysite
        state: link

    - name: Ensure file exists (touch)
      file:
        path: /var/log/myapp.log
        state: touch
        owner: appuser
        mode: '0644'

    - name: Remove file
      file:
        path: /tmp/old-file.txt
        state: absent

    - name: Set permissions recursively
      file:
        path: /var/www/html
        state: directory
        owner: nginx
        recurse: yes
```

### 5. Example Scenario

- Deploy web application to servers
- Copy HTML files: `copy` module
- Create document root: `file` module with `state: directory`
- Set ownership to `nginx` user
- Set permissions `755` for directories, `644` for files

### 6. Common Interview / KT Line

"Copy moves files from control node to targets, while file manages properties like permissions and ownership on remote systems."

### 7. Closing Line Trigger

"In short, copy and file modules handle all basic file management needs in Ansible."

---

## Roles
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Roles are reusable, organized structures for grouping related automation."

### 1. What it is (Simple Definition)

- Roles organize playbooks into reusable components
- Standard directory structure: tasks, handlers, vars, templates, files
- Enable modular, maintainable automation
- Can be shared via Ansible Galaxy

### 2. Why / When to Use

- Create reusable automation units (nginx role, mysql role)
- Organize complex playbooks logically
- Share common functionality across projects
- Simplify playbook structure and readability

### 3. Key Syntax / Keywords

- `roles:`
- `tasks/main.yml`
- `handlers/main.yml`
- `vars/main.yml`
- `defaults/main.yml`
- `templates/`
- `files/`
- `meta/main.yml`

### 4. Minimal Playbook Example

**Directory structure:**
```
roles/
  nginx/
    tasks/main.yml
    handlers/main.yml
    templates/nginx.conf.j2
    files/index.html
    vars/main.yml
    defaults/main.yml
```

**roles/nginx/tasks/main.yml:**
```yaml
---
- name: Install nginx
  yum:
    name: nginx
    state: present

- name: Copy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart nginx

- name: Start nginx
  service:
    name: nginx
    state: started
    enabled: true
```

**roles/nginx/handlers/main.yml:**
```yaml
---
- name: Restart nginx
  service:
    name: nginx
    state: restarted
```

**Playbook using role:**
```yaml
---
- name: Setup web servers
  hosts: web
  become: true
  roles:
    - nginx
```

### 5. Example Scenario

- Multiple projects need nginx configured
- Create `nginx` role with all nginx tasks
- Include role in any playbook: `roles: [nginx]`
- Role handles installation, configuration, service management
- Reuse across projects without duplication

### 6. Common Interview / KT Line

"Roles let us package related tasks, handlers, and files into reusable modules that work like building blocks."

### 7. Closing Line Trigger

"In short, roles transform playbooks from scripts into modular, maintainable infrastructure components."

---

## Role Directory Structure
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Standard layout organizes role components: tasks, handlers, vars, templates, files."

### 1. What it is (Simple Definition)

- Ansible expects specific directory structure for roles
- Each directory has specific purpose and naming convention
- `main.yml` files automatically loaded from each directory
- Structure enables automatic inclusion and organization

### 2. Why / When to Use

- Follow Ansible conventions for portability
- Organize related files logically
- Enable automatic file discovery by Ansible
- Share roles via Ansible Galaxy

### 3. Key Syntax / Keywords

```
role_name/
├── tasks/main.yml       # Main task list
├── handlers/main.yml    # Handlers
├── templates/           # Jinja2 templates
├── files/               # Static files
├── vars/main.yml        # Role variables (high precedence)
├── defaults/main.yml    # Default variables (low precedence)
├── meta/main.yml        # Role metadata (dependencies)
├── library/             # Custom modules
└── README.md            # Role documentation
```

### 4. Minimal Playbook Example

**Complete role structure:**
```
roles/webserver/
  tasks/
    main.yml
  handlers/
    main.yml
  templates/
    site.conf.j2
  files/
    index.html
  vars/
    main.yml
  defaults/
    main.yml
  meta/
    main.yml
```

**roles/webserver/defaults/main.yml:**
```yaml
---
http_port: 80
server_name: localhost
```

**roles/webserver/tasks/main.yml:**
```yaml
---
- name: Install web server
  yum:
    name: nginx
    state: present

- include_tasks: configure.yml
```

**roles/webserver/meta/main.yml:**
```yaml
---
dependencies:
  - role: common
  - role: firewall
```

### 5. Example Scenario

- Creating nginx role for standardization
- Put installation tasks in `tasks/main.yml`
- Put restart handler in `handlers/main.yml`
- Put config template in `templates/nginx.conf.j2`
- Put default variables in `defaults/main.yml`
- Structure recognized automatically by Ansible

### 6. Common Interview / KT Line

"Following the standard role structure makes roles portable, shareable, and automatically understood by Ansible."

### 7. Closing Line Trigger

"In short, role structure is Ansible's convention for organizing reusable automation logically."

---

## Ansible Galaxy
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Community hub for discovering, sharing, and installing Ansible roles."

### 1. What it is (Simple Definition)

- Ansible Galaxy is a repository of community-contributed roles
- Command-line tool for installing and managing roles
- Thousands of pre-built roles for common tasks
- Hosted at galaxy.ansible.com

### 2. Why / When to Use

- Avoid reinventing the wheel for common automation
- Leverage community-tested roles
- Quick-start projects with proven components
- Share your own roles with the community

### 3. Key Syntax / Keywords

- `ansible-galaxy install`
- `ansible-galaxy init`
- `requirements.yml`
- `ansible-galaxy list`
- `ansible-galaxy remove`

### 4. Minimal Playbook Example

```bash
# Install role from Galaxy
ansible-galaxy install geerlingguy.nginx

# Install specific version
ansible-galaxy install geerlingguy.nginx,3.1.4

# Install from requirements file
ansible-galaxy install -r requirements.yml

# Initialize new role structure
ansible-galaxy init my_new_role

# List installed roles
ansible-galaxy list

# Remove role
ansible-galaxy remove geerlingguy.nginx
```

**requirements.yml:**
```yaml
---
roles:
  - name: geerlingguy.nginx
    version: 3.1.4
  - name: geerlingguy.mysql
  - src: https://github.com/user/custom-role.git
    name: custom_role
```

**Using Galaxy role in playbook:**
```yaml
---
- name: Deploy web server using Galaxy role
  hosts: web
  become: true
  roles:
    - geerlingguy.nginx
```

### 5. Example Scenario

- Need to set up PostgreSQL on servers
- Search Galaxy: find `geerlingguy.postgresql` role
- Install: `ansible-galaxy install geerlingguy.postgresql`
- Include in playbook: `roles: [geerlingguy.postgresql]`
- Saves hours of writing and testing custom role

### 6. Common Interview / KT Line

"Ansible Galaxy is like npm or pip for Ansible – a central repository where we can find and install pre-built roles."

### 7. Closing Line Trigger

"In short, Galaxy accelerates automation by providing battle-tested, community-maintained roles."

---

## Vault
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Vault encrypts sensitive data like passwords and keys in playbooks."

### 1. What it is (Simple Definition)

- Ansible Vault encrypts sensitive files or variables
- Uses AES256 encryption
- Integrates seamlessly with playbooks and roles
- Protects secrets in version control

### 2. Why / When to Use

- Store passwords, API keys, certificates securely
- Commit encrypted secrets to Git safely
- Comply with security policies
- Share playbooks without exposing credentials

### 3. Key Syntax / Keywords

- `ansible-vault create`
- `ansible-vault encrypt`
- `ansible-vault decrypt`
- `ansible-vault edit`
- `ansible-vault view`
- `--ask-vault-pass`
- `--vault-password-file`

### 4. Minimal Playbook Example

```bash
# Create encrypted file
ansible-vault create secrets.yml

# Encrypt existing file
ansible-vault encrypt vars/passwords.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# View encrypted file
ansible-vault view secrets.yml

# Decrypt file
ansible-vault decrypt secrets.yml

# Change vault password
ansible-vault rekey secrets.yml

# Run playbook with vault
ansible-playbook site.yml --ask-vault-pass

# Use password file
ansible-playbook site.yml --vault-password-file ~/.vault_pass
```

**secrets.yml (before encryption):**
```yaml
---
db_password: SuperSecret123
api_key: abc123xyz789
```

**Using vault-encrypted variables:**
```yaml
---
- name: Deploy with secrets
  hosts: db
  become: true
  vars_files:
    - secrets.yml
  tasks:
    - name: Configure database
      mysql_user:
        name: dbuser
        password: "{{ db_password }}"
        state: present
```

**Inline vault encryption:**
```yaml
---
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          66386439653236396662343061...
```

### 5. Example Scenario

- Playbook needs database passwords
- Create `secrets.yml` with sensitive data
- Encrypt: `ansible-vault encrypt secrets.yml`
- Commit encrypted file to Git safely
- Team members decrypt with shared vault password
- Secrets protected in version control

### 6. Common Interview / KT Line

"Ansible Vault lets us safely commit encrypted secrets to Git, so automation can be version-controlled without security risks."

### 7. Closing Line Trigger

"In short, Vault secures sensitive data while maintaining infrastructure-as-code best practices."

---

## Tags
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Tags enable selective execution of specific tasks or roles."

### 1. What it is (Simple Definition)

- Tags are labels applied to tasks, plays, or roles
- Filter playbook execution using `--tags` or `--skip-tags`
- Run only tagged subsets without modifying playbooks
- Useful for partial deployments or testing

### 2. Why / When to Use

- Run only configuration tasks, skip installation
- Execute specific sections during troubleshooting
- Separate fast tasks from slow tasks
- Test individual components independently

### 3. Key Syntax / Keywords

- `tags:`
- `--tags`
- `--skip-tags`
- `--list-tags`
- `always` (special tag)
- `never` (special tag)
- `tagged`
- `untagged`
- `all`

### 4. Minimal Playbook Example

```yaml
---
- name: Web server deployment
  hosts: web
  become: true
  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present
      tags:
        - install
        - packages

    - name: Copy nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      tags:
        - config
        - nginx

    - name: Start nginx
      service:
        name: nginx
        state: started
      tags:
        - service
        - nginx

    - name: Deploy application code
      copy:
        src: app/
        dest: /var/www/html/
      tags:
        - deploy
        - app

    - name: Always run health check
      uri:
        url: http://localhost
        status_code: 200
      tags:
        - always
```

**Running with tags:**
```bash
# Run only config tasks
ansible-playbook site.yml --tags config

# Run multiple tags
ansible-playbook site.yml --tags "install,config"

# Skip certain tags
ansible-playbook site.yml --skip-tags deploy

# List all available tags
ansible-playbook site.yml --list-tags

# Run only untagged tasks
ansible-playbook site.yml --tags untagged
```

### 5. Example Scenario

- Full deployment takes 30 minutes
- Only need to update application code
- Run: `ansible-playbook deploy.yml --tags app`
- Skips installation, configuration, just deploys code
- Completes in 2 minutes instead of 30

### 6. Common Interview / KT Line

"Tags let us run specific parts of playbooks on demand without modifying the playbook itself – perfect for selective deployments."

### 7. Closing Line Trigger

"In short, tags provide surgical control over playbook execution for efficiency and flexibility."

---

## Error Handling (ignore_errors, failed_when)
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Control how Ansible responds to task failures."

### 1. What it is (Simple Definition)

- By default, Ansible stops execution on task failure
- `ignore_errors` continues execution despite failures
- `failed_when` customizes failure conditions
- `block/rescue/always` provides try-catch-finally logic

### 2. Why / When to Use

- Handle expected failures gracefully
- Continue playbook on non-critical errors
- Define custom failure conditions
- Implement error recovery workflows

### 3. Key Syntax / Keywords

- `ignore_errors: true`
- `failed_when:`
- `changed_when:`
- `block:`, `rescue:`, `always:`
- `any_errors_fatal: true`

### 4. Minimal Playbook Example

```yaml
---
- name: Error handling examples
  hosts: web
  become: true
  tasks:
    # Ignore errors
    - name: Try to stop service (may not exist)
      service:
        name: optional-service
        state: stopped
      ignore_errors: true

    # Custom failure condition
    - name: Check disk space
      shell: df -h / | tail -1 | awk '{print $5}' | sed 's/%//'
      register: disk_usage
      failed_when: disk_usage.stdout|int > 90

    # Never fail
    - name: Cleanup logs (always succeeds)
      shell: rm -f /tmp/*.log
      register: cleanup
      failed_when: false

    # Block/rescue/always (try-catch-finally)
    - block:
        - name: Attempt risky operation
          shell: /opt/scripts/risky.sh
        - name: Continue if above succeeds
          debug:
            msg: "Risky operation completed"
      rescue:
        - name: Handle failure
          debug:
            msg: "Risky operation failed, running recovery"
        - name: Recovery task
          shell: /opt/scripts/recover.sh
      always:
        - name: Always cleanup
          file:
            path: /tmp/risky.lock
            state: absent

    # Register and check result
    - name: Run command
      command: /opt/check.sh
      register: result
      ignore_errors: true

    - name: Fail if specific condition
      fail:
        msg: "Check failed with unexpected error"
      when: 
        - result.rc != 0
        - "'expected error' not in result.stderr"
```

### 5. Example Scenario

- Playbook stops optional service before proceeding
- Service might not be installed on all servers
- Use `ignore_errors: true` to continue if service missing
- Critical tasks still fail normally
- Playbook completes successfully on all servers

### 6. Common Interview / KT Line

"Error handling lets us build resilient playbooks that gracefully handle expected failures while still catching real problems."

### 7. Closing Line Trigger

"In short, Ansible provides flexible error handling for both continuing through issues and catching failures precisely."

---

## Async & Poll
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Run long-running tasks asynchronously without blocking playbook execution."

### 1. What it is (Simple Definition)

- `async` runs tasks in background without waiting
- `poll` controls how often Ansible checks task status
- Enables parallel execution of time-consuming operations
- Prevents SSH timeout on long-running tasks

### 2. Why / When to Use

- Run tasks taking >30 minutes (backups, updates)
- Execute tasks in parallel across hosts
- Avoid SSH connection timeouts
- Fire-and-forget operations

### 3. Key Syntax / Keywords

- `async:` (timeout in seconds)
- `poll:` (check interval, 0 = fire-and-forget)
- `async_status` module
- `register:` (capture job ID)

### 4. Minimal Playbook Example

```yaml
---
- name: Async task examples
  hosts: web
  tasks:
    # Long-running task with polling
    - name: Long software update
      yum:
        name: "*"
        state: latest
      async: 3600  # Allow up to 1 hour
      poll: 10     # Check every 10 seconds

    # Fire-and-forget (poll=0)
    - name: Start background backup
      shell: /opt/scripts/backup.sh
      async: 7200  # 2 hours max
      poll: 0      # Don't wait
      register: backup_job

    # Check async task status later
    - name: Do other tasks while backup runs
      debug:
        msg: "Performing other work..."

    - name: Check backup status
      async_status:
        jid: "{{ backup_job.ansible_job_id }}"
      register: job_result
      until: job_result.finished
      retries: 60
      delay: 10

    # Parallel long operations
    - name: Deploy large application files
      copy:
        src: "/source/large-app-{{ item }}.tar.gz"
        dest: "/opt/apps/"
      async: 1800
      poll: 0
      loop:
        - web
        - api
        - worker
      register: copy_jobs

    - name: Wait for all copies to complete
      async_status:
        jid: "{{ item.ansible_job_id }}"
      register: copy_results
      until: copy_results.finished
      retries: 180
      delay: 10
      loop: "{{ copy_jobs.results }}"
```

### 5. Example Scenario

- Need to update all packages on 100 servers
- Updates take 20 minutes per server
- Normal execution: 2000 minutes total (serial)
- With `async`: All servers update simultaneously
- Completes in 20 minutes instead of 2000
- Ansible polls periodically to track completion

### 6. Common Interview / KT Line

"Async execution lets long-running tasks complete in parallel without blocking the playbook or timing out SSH connections."

### 7. Closing Line Trigger

"In short, async and poll enable efficient handling of time-consuming operations at scale."

---

## Delegation (delegate_to, local_action)
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Execute tasks on different hosts than the play target."

### 1. What it is (Simple Definition)

- `delegate_to` runs task on specified host instead of target
- `local_action` is shorthand for delegating to control node
- Enables running tasks on load balancers, databases, or localhost
- Useful for orchestration and external integrations

### 2. Why / When to Use

- Update load balancer during rolling deployments
- Run API calls from control node
- Perform database operations on db server
- Execute local scripts before remote deployment

### 3. Key Syntax / Keywords

- `delegate_to:`
- `local_action:`
- `run_once: true`
- `delegate_facts: true`

### 4. Minimal Playbook Example

```yaml
---
- name: Delegation examples
  hosts: web
  tasks:
    # Delegate to specific host
    - name: Remove server from load balancer
      haproxy:
        state: disabled
        host: "{{ inventory_hostname }}"
      delegate_to: loadbalancer.example.com

    - name: Deploy application
      copy:
        src: app.tar.gz
        dest: /opt/app/

    - name: Add server back to load balancer
      haproxy:
        state: enabled
        host: "{{ inventory_hostname }}"
      delegate_to: loadbalancer.example.com

    # Local action (run on control node)
    - name: Generate deployment report
      local_action:
        module: copy
        content: "Deployed to {{ inventory_hostname }} at {{ ansible_date_time.iso8601 }}"
        dest: "/tmp/deployment-{{ inventory_hostname }}.txt"

    # Alternative local_action syntax
    - name: Check external API
      uri:
        url: "https://api.example.com/status"
        method: GET
      delegate_to: localhost

    # Run once for all hosts
    - name: Send deployment notification
      slack:
        token: "{{ slack_token }}"
        msg: "Deployment to {{ ansible_play_hosts | length }} servers complete"
      delegate_to: localhost
      run_once: true

    # Delegate with facts
    - name: Get database status
      command: mysql -e "SHOW STATUS"
      delegate_to: db.example.com
      delegate_facts: true
      register: db_status
```

### 5. Example Scenario

- Rolling deployment of web servers
- Before updating each server, remove from load balancer
- Task delegates to load balancer host
- Deploy application on web server
- Add server back to load balancer via delegation
- Load balancer tasks run on LB, not web servers

### 6. Common Interview / KT Line

"Delegation lets us orchestrate multi-tier operations by running tasks on different hosts than the play targets."

### 7. Closing Line Trigger

"In short, delegation enables complex orchestration by running tasks exactly where they need to execute."

---

## Privilege Escalation (become)
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Execute tasks with elevated privileges using become."

### 1. What it is (Simple Definition)

- `become` enables privilege escalation (sudo, su, etc.)
- Required for tasks needing root permissions
- Can be set at play, task, or command-line level
- Supports multiple escalation methods

### 2. Why / When to Use

- Install packages (requires root)
- Modify system files or services
- Create users or change permissions
- Any operation requiring elevated privileges

### 3. Key Syntax / Keywords

- `become: true/false`
- `become_user:`
- `become_method:` (sudo, su, pbrun, etc.)
- `become_flags:`
- `--become` (`-b`)
- `--ask-become-pass` (`-K`)

### 4. Minimal Playbook Example

```yaml
---
- name: Privilege escalation examples
  hosts: web
  become: true  # All tasks run as root
  tasks:
    - name: Install package (needs root)
      yum:
        name: nginx
        state: present

    - name: Start service (needs root)
      service:
        name: nginx
        state: started

    - name: Run as specific user
      command: whoami
      become_user: appuser
      register: current_user

    - name: Display current user
      debug:
        msg: "Task ran as {{ current_user.stdout }}"

# Play-level become disabled, task-level enabled
- name: Selective privilege escalation
  hosts: web
  become: false
  tasks:
    - name: Non-privileged task
      command: whoami

    - name: Privileged task
      yum:
        name: vim
        state: present
      become: true

# Using different become method
- name: Use su instead of sudo
  hosts: legacy
  become: true
  become_method: su
  tasks:
    - name: Install package
      yum:
        name: httpd
        state: present
```

**Command-line become:**
```bash
# Run playbook with become
ansible-playbook site.yml --become

# Prompt for sudo password
ansible-playbook site.yml --become --ask-become-pass

# Become specific user
ansible-playbook site.yml --become --become-user=admin
```

**ansible.cfg configuration:**
```ini
[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

### 5. Example Scenario

- Playbook needs to install packages and restart services
- Normal user doesn't have root privileges
- Add `become: true` at play level
- Ansible uses sudo to execute tasks as root
- If sudo requires password, use `--ask-become-pass`
- All system modifications complete successfully

### 6. Common Interview / KT Line

"Become is Ansible's way of handling sudo – it elevates privileges transparently so automation can perform administrative tasks."

### 7. Closing Line Trigger

"In short, become enables Ansible to perform privileged operations securely and flexibly."

---

## Ansible Best Practices
[↑ Back to Overview](#left-side-topic-map-overview)

🔑 **Primary Trigger:** "Guidelines for writing maintainable, secure, and efficient Ansible automation."

### 1. What it is (Simple Definition)

- Collection of proven patterns and conventions
- Improves code quality, security, and maintainability
- Reduces errors and increases reusability
- Industry-standard recommendations

### 2. Why / When to Use

- Start projects with solid foundation
- Ensure team consistency
- Prepare for scale and complexity
- Pass audits and security reviews

### 3. Key Syntax / Keywords

- Idempotency
- Use roles
- Name all tasks
- Version control
- Test in CI/CD
- Use tags
- Encrypt with Vault
- Limit scope
- Use `--check` mode

### 4. Minimal Playbook Example

**Good Practices:**
```yaml
---
# ✅ Good: Named play and tasks
- name: Deploy web application
  hosts: web
  become: true
  
  # ✅ Good: Variables in separate file
  vars_files:
    - vars/web.yml
  
  # ✅ Good: Using roles
  roles:
    - common
    - nginx
    - app
  
  tasks:
    # ✅ Good: Descriptive task names
    - name: Ensure application directory exists
      file:
        path: /opt/myapp
        state: directory
        mode: '0755'
      
    # ✅ Good: Using modules instead of shell
    - name: Install required packages
      package:
        name: "{{ item }}"
        state: present
      loop:
        - git
        - python3
      
    # ✅ Good: Idempotent operations
    - name: Copy application config
      template:
        src: app.conf.j2
        dest: /etc/myapp/app.conf
      notify: Restart application
  
  handlers:
    - name: Restart application
      service:
        name: myapp
        state: restarted
```

**Bad Practices (Avoid):**
```yaml
---
# ❌ Bad: No play name, no task names
- hosts: all
  tasks:
    # ❌ Bad: Using shell instead of modules
    - shell: yum install -y nginx
    
    # ❌ Bad: Not idempotent
    - shell: echo "test" >> /etc/config
    
    # ❌ Bad: Hardcoded values
    - copy:
        src: /tmp/file.txt
        dest: /opt/app/
        owner: user1234
```

**Key Best Practices List:**

1. **Idempotency**: Always use idempotent modules
2. **Name everything**: Every play and task should have descriptive names
3. **Use modules**: Prefer modules over shell/command
4. **Version control**: Keep playbooks in Git
5. **Use roles**: Organize complex playbooks with roles
6. **Variables**: Externalize configuration
7. **Vault**: Encrypt sensitive data
8. **Testing**: Use `--check` and `--diff` modes
9. **Tagging**: Tag tasks for selective execution
10. **Documentation**: Comment complex logic, maintain README
11. **Limit scope**: Use `--limit` for testing
12. **Error handling**: Use `failed_when` and `changed_when`

### 5. Example Scenario

- Team starting new Ansible project
- Follow best practices from day one:
  - Create role structure with `ansible-galaxy init`
  - Use separate `group_vars` and `host_vars`
  - Name all tasks descriptively
  - Encrypt secrets with Vault
  - Test in CI pipeline before production
- Project scales smoothly with minimal technical debt

### 6. Common Interview / KT Line

"Following Ansible best practices means writing idempotent, modular, well-named automation that's easy to maintain and scale."

### 7. Closing Line Trigger

"In short, best practices turn Ansible from a scripting tool into a professional infrastructure-as-code platform."

---

## Closing Notes

This cheat sheet covers **core Ansible concepts** for fast retrieval, speaking fluency, and knowledge transfer. Each section follows a consistent pattern to help you recall and explain concepts confidently.

**How to Use This Document:**

- **Before interviews**: Read primary triggers to activate memory
- **During KT sessions**: Use examples and speaking lines verbatim
- **Quick lookups**: Reference commands and YAML syntax
- **Teaching juniors**: Follow the structure for clear explanations

**Practice Tips:**

- Read each section's primary trigger and try explaining without notes
- Run the example commands and playbooks in lab environment
- Speak sections aloud to practice verbal fluency
- Combine multiple topics for comprehensive explanations

**Remember:**
- Ansible is **agentless** (SSH-based)
- Playbooks are **declarative** (desired state)
- Modules are **idempotent** (safe to run multiple times)
- Roles make automation **reusable** and **maintainable**

---

**End of Ansible Core Concepts – High Retrieval & Teaching Cheat Sheet**
