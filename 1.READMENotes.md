# Ansible Notes

## Table of Contents
* [Ansible Variables](#ansible-variables)
* [Ansible Facts](#ansible-facts)
* [Ansible Constructs](#ansible-constructs)
* [Ansible Templates](#ansible-templates)
* [Author](#author)


## Ansible Variables
Ansible variables allow you to simplify your playbooks, improve readability, and make your automation scripts more flexible. They enable you to define reusable parameters, reducing duplication and centralizing configuration management.

### Date Playbook (date_play.yml)

```yaml
---
- name: Date Playbook
  hosts: devservers
  vars:
    date_command: date
    uptime_command: uptime
  tasks:
    - name: Show the current date
      command: "{{ date_command }}"

    - name: Show the system uptime
      command: "{{ uptime_command }}"
```

---

## Ansible Facts
Ansible facts are automatically collected information about your managed nodes, including system architecture, network interfaces, disk space, and more. They provide real-time data that can be used in your playbooks to make context-aware automation decisions.

### Date Playbook (date_play.yml)

```yaml
---
- name: Date and Uptime Playbook
  hosts: devservers
  gather_facts: yes
  tasks:
    - name: Show the current date
      debug:
        msg: "Current Date: {{ ansible_date_time.date }}"

    - name: Show the system uptime
      debug:
        msg: "System Uptime: {{ ansible_uptime_seconds }} seconds"
```

---

## **Ansible Constructs**

Ansible provides a range of constructs to enhance the flexibility and power of your playbooks. These include **Conditionals**, **Handlers**, and **Loops**, which allow you to control the flow of tasks and respond to changes in your infrastructure.

### **1. Conditionals**

Conditionals allow you to run tasks based on the state of a variable, output of a command, or facts collected from the managed nodes.

**Example:**

```yaml
- name: Install Apache if not already installed
  apt:
    name: apache2
    state: present
  when: ansible_os_family == "Debian"
```

### **2. Handlers**

Handlers are special tasks that are triggered when a related task reports a change, ensuring efficient resource usage.

**Example:**

```yaml
- name: Copy the Apache configuration file
  copy:
    src: apache.conf
    dest: /etc/apache2/apache2.conf
  notify: Restart Apache

handlers:
  - name: Restart Apache
    service:
      name: apache2
      state: restarted
```

### **3. Loops**

Loops enable you to repeat a task for multiple items, reducing redundancy and simplifying your playbooks.

**Example:**

```yaml
- name: Install multiple packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - curl
    - nginx
```

Using these constructs effectively can make your playbooks more efficient, reusable, and easier to maintain.

---

## **Ansible Templates**

Ansible templates provide a flexible way to dynamically generate configuration files using the **Jinja2** templating engine. This approach simplifies the management of complex files, ensuring consistency and reusability.

### **Why Use Templates?**

* **Dynamic Configuration:** Generate files based on variables, facts, and logic.
* **Reusability:** Create a single template for multiple hosts or environments.
* **Consistency:** Ensure uniform formatting and structure.

### **Directory Structure:**

```
.
├── templates/
│   ├── welcome_message.j2
│   └── nginx.conf.j2
└── site.yml
```

---

### **Welcome Message Example:**

#### **Template File (`templates/welcome_message.j2`):**

```jinja
######################################
Welcome to {{ ansible_hostname }}!
Managed by Ansible.
######################################
```

#### **Playbook File (`site.yml`):**

```yaml
---
- name: Configure Welcome Message and Nginx
  hosts: all
  vars:
    welcome_file: /etc/motd
    server_name: example.com
    backend_server: app-server
    backend_port: 8080
  tasks:
    - name: Deploy Welcome Message
      template:
        src: templates/welcome_message.j2
        dest: "{{ welcome_file }}"
      notify: Display Welcome Message

    - name: Deploy Nginx Configuration
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx

handlers:
  - name: Display Welcome Message
    command:
      cmd: cat "{{ welcome_file }}"

  - name: Restart Nginx
    service:
      name: nginx
      state: restarted
```

### **Nginx Configuration Example:**

#### **Template File (`templates/nginx.conf.j2`):**

```jinja
server {
    listen 80;
    server_name {{ server_name }};
    location / {
        proxy_pass http://{{ backend_server }}:{{ backend_port }};
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

### **Expected Outputs:**

**Welcome Message:**
After running the playbook, logging into a node should display a message like:

```
######################################
Welcome to server01!
Managed by Ansible.
######################################
```

**Nginx Configuration:**
Your Nginx server will be configured to proxy requests to a backend server, as specified in the playbook variables.

---

## Author 

Abhishek Rajput
