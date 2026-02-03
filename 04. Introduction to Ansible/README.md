# ANSIBLE - CONCISE NOTES

**Presenter:** Akini Karunarathne, Systems Engineer  
**Date:** July 03, 2025

---

## What is Ansible?

A software tool for automating infrastructure provisioning, configuration management, application deployment, orchestration, and other IT processes.

### Why Ansible?
- Eliminates repetition and simplifies workflows
- Agentless (uses SSH)
- Simple and human-readable (YAML)
- Idempotent
- Cross-platform
- Modular and reusable

---

## Ansible vs Terraform

| Aspect | Terraform | Ansible |
|--------|-----------|---------|
| **Focus** | Infrastructure provisioning | Configuration management & deployment |
| **Analogy** | Building the house | Furnishing the house |
| **Use** | Create cloud resources, networking, databases | Install software, deploy apps, manage services |

**Together:** Terraform provisions infrastructure, Ansible configures it.

---

## Architecture

```
Control Node (Management Node)
    ↓ (SSH)
Inventory File → Managed Nodes
```

### Components:
- **Control Node:** Where Ansible is installed and runs
- **Managed Nodes:** Target systems to be automated
- **Inventory:** List of hosts organized in groups
- **SSH:** Communication method (agentless)

---

## Core Concepts

| Concept | Definition |
|---------|-----------|
| **Playbooks** | YAML files containing plays and tasks |
| **Plays** | Maps hosts to tasks |
| **Tasks** | Individual actions/operations |
| **Modules** | Code that executes tasks on managed nodes |
| **Roles** | Reusable, organized bundle of tasks, variables, templates |
| **Handlers** | Tasks triggered only when notified |
| **Variables** | Values that can change between runs |
| **Inventory** | List of hosts and groups |

---

## Installation

### Prerequisites:
- **Control Node:** Python + UNIX-like OS (Linux, macOS, WSL)
- **Managed Nodes:** Python + SSH access

### Install Ansible:

```bash
# Using pip
python3 -m pip install --user ansible

# Using apt (Ubuntu/Debian)
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

---

## Ansible Configuration File

Location priority:
1. Command line
2. ANSIBLE_CONFIG environment variable
3. ansible.cfg (current directory)
4. ~/.ansible.cfg (home directory)
5. /etc/ansible/ansible.cfg (system-wide)

### Check config being used:
```bash
ansible --version
ansible-config dump
```

---

## Inventory File

Defines hosts and groups to manage.

### INI Format:
```ini
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com

[webservers:vars]
http_port = 80
```

### YAML Format:
```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
      vars:
        http_port: 80
```

### Using Inventory:
```bash
ansible-playbook -i inventory.ini playbook.yml
```

---

## Modules (Common)

```bash
# Get help
ansible-doc <module_name>
```

| Module | Purpose |
|--------|---------|
| `ping` | Test connectivity |
| `command` | Run command (no shell features) |
| `shell` | Run command with shell features |
| `copy` | Copy file to remote host |
| `file` | Manage file properties |
| `package` | Install/remove packages |
| `service` | Manage services |
| `user` | Create/delete users |
| `apt/yum` | Install packages |
| `template` | Deploy template file |

---

## Playbooks

YAML file with plays and tasks.

### Basic Structure:
```yaml
---
- name: Playbook name
  hosts: webservers
  become: true
  
  vars:
    var1: value1
  
  tasks:
    - name: Task 1
      module_name:
        param1: value1
    
    - name: Task 2
      module_name:
        param1: value1
      notify: Handler name
  
  handlers:
    - name: Handler name
      service:
        name: nginx
        state: restarted
```

### Run Playbook:
```bash
ansible-playbook playbook.yml -i inventory.ini
ansible-playbook playbook.yml --check          # Dry run
ansible-playbook playbook.yml --diff           # Show changes
```

---

## Variables

### Types:

1. **Playbook Variables** - Defined in playbook with `vars:`
2. **Task Variables** - Specific to individual task
3. **Host Variables** - In inventory for specific host
4. **Group Variables** - In inventory for group
5. **Fact Variables** - Auto-gathered system info
6. **Role Variables** - Defined in role (defaults/vars)
7. **Extra Variables** - Passed at runtime with `-e`
8. **Environment Variables** - System environment vars

### Usage:
```yaml
vars:
  app_name: myapp
  
tasks:
  - name: Display variable
    debug:
      msg: "App: {{ app_name }}"
```

### Passing Extra Variables:
```bash
ansible-playbook playbook.yml -e "env=prod version=2.0"
```

---

## Conditionals

Use `when` clause for conditional execution.

```yaml
tasks:
  - name: Install on Debian
    apt:
      name: nginx
    when: ansible_os_family == "Debian"
  
  - name: Check if file exists
    stat:
      path: /etc/config.yml
    register: config_file
  
  - name: Create if not exists
    copy:
      src: config.yml
      dest: /etc/config.yml
    when: not config_file.stat.exists
```

---

## Loops

Automate repeated tasks.

```yaml
tasks:
  - name: Install packages
    package:
      name: "{{ item }}"
      state: present
    loop:
      - nginx
      - curl
      - git
  
  - name: Create users
    user:
      name: "{{ item.name }}"
      uid: "{{ item.uid }}"
    loop:
      - { name: john, uid: 1001 }
      - { name: jane, uid: 1002 }
```

---

## Templates (Jinja2)

Generate dynamic files using variables.

### Jinja2 Syntax:
```jinja2
{{ variable }}                    # Output variable
{% if condition %} ... {% endif %} # Logic
{# Comment #}                      # Comment
{{ var | upper }}                  # Filters
```

### Example Template (nginx.conf.j2):
```jinja2
server {
    listen {{ server_port }};
    server_name {{ server_name }};
    
    {% if enable_ssl %}
    ssl on;
    {% endif %}
}
```

### Deploy Template:
```yaml
- name: Deploy config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    mode: '0644'
```

---

## Roles

Organized, reusable collection of tasks, variables, templates.

### Create Role:
```bash
ansible-galaxy init myrole
```

### Directory Structure:
```
myrole/
├── tasks/
│   └── main.yml
├── handlers/
│   └── main.yml
├── templates/
├── files/
├── vars/
│   └── main.yml
├── defaults/
│   └── main.yml
└── meta/
    └── main.yml
```

### Using Roles:
```yaml
---
- name: Deploy
  hosts: webservers
  
  roles:
    - nginx
    - nodejs
    - docker
```

---

## Ansible Galaxy

Public repository of community-contributed roles and collections.

### Commands:
```bash
ansible-galaxy init <role_name>           # Create role
ansible-galaxy install <role_name>        # Install role
ansible-galaxy list                       # List installed roles
ansible-galaxy remove <role_name>         # Remove role
ansible-galaxy info <role_name>           # Get info
```

### Popular Roles:
- geerlingguy.nginx
- geerlingguy.docker
- geerlingguy.nodejs
- geerlingguy.mysql

---

## Ansible Facts

Auto-gathered system information.

### Gathering Facts:
```bash
# Automatically gathered before playbook runs
# Can be disabled with: gather_facts: false
```

### Using Facts:
```yaml
- name: Display OS
  debug:
    msg: "OS: {{ ansible_os_family }}"

- name: Display IP
  debug:
    msg: "IP: {{ ansible_default_ipv4.address }}"
```

### Common Facts:
- `ansible_os_family` - OS family (Debian, RedHat)
- `ansible_distribution` - Distribution (Ubuntu, CentOS)
- `ansible_hostname` - Hostname
- `ansible_default_ipv4.address` - Primary IP
- `ansible_processor_cores` - CPU cores
- `ansible_memtotal_mb` - Total memory

---

## Best Practices

1. **Organize Directory Structure**
   ```
   my-project/
   ├── inventories/
   ├── roles/
   ├── playbooks/
   ├── group_vars/
   ├── host_vars/
   └── ansible.cfg
   ```

2. **Use Descriptive Names**
   - Playbooks: `deploy-app.yml`, `configure-firewall.yml`
   - Tasks: `Install nginx`, `Create user`
   - Roles: `webserver`, `database`

3. **Use Roles for Organization**
   - Organize related tasks
   - Enable reusability

4. **Use Variables for Flexibility**
   - Different values for different environments
   - Makes playbooks more reusable

5. **Use Version Control**
   - Track changes in Git
   - Maintain history

6. **Use Handlers for Services**
   - Use handlers to restart services
   - Execute only once

7. **Test with Check Mode**
   ```bash
   ansible-playbook playbook.yml --check        # Dry run
   ansible-playbook playbook.yml --diff --check # Show changes
   ```

8. **Lint Code**
   ```bash
   pip install ansible-lint
   ansible-lint playbook.yml
   ```

---

## Quick Reference Commands

```bash
# List all hosts in inventory
ansible-inventory -i inventory.ini --list

# Test connectivity
ansible all -i inventory.ini -m ping

# Run ad-hoc command
ansible webservers -i inventory.ini -m shell -a "ls -la"

# Run playbook
ansible-playbook -i inventory.ini playbook.yml

# Run with extra variables
ansible-playbook playbook.yml -e "var1=value1 var2=value2"

# Run specific tasks with tags
ansible-playbook playbook.yml --tags "webserver"

# Verbose output
ansible-playbook playbook.yml -vv
```

---

## Summary

Ansible automates IT operations through:
- **Agentless** architecture (SSH-based)
- **YAML** playbooks (human-readable)
- **Idempotent** operations (safe to repeat)
- **Roles** for organization and reusability
- **Variables** for flexibility
- **Templates** for dynamic configs
- **Modules** for various tasks

**Key files:**
- `playbook.yml` - Define automation tasks
- `inventory.ini` - List of hosts
- `ansible.cfg` - Configuration
- `roles/` - Reusable content

---

**Resources:**
- https://docs.ansible.com
- https://galaxy.ansible.com
