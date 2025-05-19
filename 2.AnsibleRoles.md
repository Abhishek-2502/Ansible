#  Ansible Role

In Ansible, a role is a reusable and modular unit of configuration that organizes tasks, variables, files, templates, handlers, etc. Roles make playbooks cleaner and more maintainable.
This Ansible role installs Apache and renders a custom HTML page using a Jinja2 template.

## 📁 Directory Structure

```
project-root/
├── apache/                     # Custom Ansible role
│   ├── defaults/
│   │   └── main.yml            # Default variables
│   ├── handlers/
│   │   └── main.yml            # Notification handlers
│   ├── meta/
│   │   └── main.yml            # Role metadata
│   ├── tasks/
│   │   └── main.yml            # Main tasks
│   ├── templates/
│   │   └── index.html.j2       # Jinja2 HTML template
│   ├── tests/                  # (Optional) Test cases
│   ├── vars/
│   │   └── main.yml            # Static role variables
│   └── README.md               # Role-specific info (Galaxy)
├── inventory/
│   └── hosts                   # Inventory file
├── cleanup.yml                 # Optional cleanup playbook
├── role-playbook.yml           # Main playbook using the role
└── README.md                   # This README
```

---

## 📦 Role Variables

### `defaults/main.yml`

These are overrideable defaults:

```yaml
---
apache_port: 80
apache_doc_root: /var/www/html
```

### `vars/main.yml`

These are internal/static values used by the role:

```yaml
---
apache_package: apache2
apache_service: apache2
custom_message: "Hello from Apache Role!"
```

---

## 🧰 Tasks

### `tasks/main.yml`

```yaml
---
- name: Install Apache
  apt:
    name: "{{ apache_package }}"
    state: present
    update_cache: yes
  become: yes

- name: Start and enable Apache
  service:
    name: "{{ apache_service }}"
    state: started
    enabled: yes
  become: yes

- name: Deploy custom index.html
  template:
    src: index.html.j2
    dest: "{{ apache_doc_root }}/index.html"
  notify: Restart Apache
  become: yes
```

---

## 🔁 Handlers

### `handlers/main.yml`

```yaml
---
- name: Restart Apache
  service:
    name: "{{ apache_service }}"
    state: restarted
```

---

## 🧩 Template

### `templates/index.html.j2`

```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h1>{{ custom_message }}</h1>
    <p>This page is rendered by Ansible using a Jinja2 template.</p>
</body>
</html>
```

---

## 📇 Metadata

### `meta/main.yml`

```yaml
---
galaxy_info:
  role_name: apache
  author: your_name
  description: Install and configure Apache to serve a custom HTML page
  license: MIT
  min_ansible_version: 2.9
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy

dependencies: []
```

---

## 📜 Inventory

### `inventory/hosts`

```ini
[webservers]
server_1 ansible_host=<Public_IP_1>

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/keys/pem_key_name.pem
```

Replace `<Public_IP_1>` and `.pem` path with actual values.

---

## ▶️ Playbook

### `role-playbook.yml`

```yaml
---
- name: Apply Apache Role
  hosts: webservers
  become: yes
  roles:
    - apache
```

---

## 🧼 Cleanup (Optional)

### `cleanup.yml`

```yaml
---
- name: Remove Apache and Custom Page
  hosts: webservers
  become: yes
  tasks:
    - name: Stop Apache
      service:
        name: apache2
        state: stopped

    - name: Uninstall Apache
      apt:
        name: apache2
        state: absent
        purge: yes

    - name: Remove index.html
      file:
        path: /var/www/html/index.html
        state: absent
```

---

## How to Use

1. Initialize your role (already done):

```bash
   ansible-galaxy init apache
```

2. Replace the generated files with the contents above.

3. Run the playbook:

```bash
   ansible-playbook -i inventory/hosts role-playbook.yml
```
   This will use the custom hosts file instead of global one which makes it more cleaner.

## Author 

Abhishek Rajput

