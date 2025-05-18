# Ansible Setup and Configuration Guide

## Table of Contents

* [Introduction to Ansible](#introduction-to-ansible)
* [Key Concepts](#key-concepts)
* [Infrastructure Setup](#infrastructure-setup)
* [Installing Ansible on the Master Server](#installing-ansible-on-the-master-server)
* [Configuring the Inventory File](#configuring-the-inventory-file)
* [Transferring the PEM Key](#transferring-the-pem-key)
* [Verifying Connectivity and run commands](#verifying-connectivity-and-run-commands)
* [Running Ansible Playbooks](#running-ansible-playbooks)
* [Example Playbooks](#example-playbooks)
* [Conclusion](#conclusion)
* [Author](#author)

## Introduction to Ansible 

Ansible is an open-source, Python-based automation tool used for configuration management, application deployment, and orchestration. It is known for its simplicity and agentless architecture, using SSH for secure communication with target systems. Key features include:

* Follows declarative approach, where you define the desired state of the system rather than scripting specific commands.
* Ansible uses SSH for communication with target systems, making it agentless and easy to set up.
* Multi-environment (Dev, Prod) and multi-cloud (AWS, GCP) management.

## Key Concepts

1. **Inventory** - The inventory file lists the target hosts on which Ansible will run tasks. It can be a static file or generated dynamically.
2. **Playbooks** - YAML files that define a set of tasks and configurations to be executed on target hosts.
3. **Tasks** - Individual units of work within playbooks. They represent actions to be performed on target hosts.
4. **Modules** - Predefined actions like package installation, file manipulation, etc. Ansible provides a wide range of modules for various tasks, such as package installation, file manipulation, service management, etc.
5. **Roles** - Reusable playbooks with organized structure. They encapsulate related tasks, handlers, variables, and files into a directory structure.
6. **Ad-Hoc Commands vs Modules** - Quick commands (`-a`) vs structured modules (`-m`)
   - ad hoc commands are great for tasks you repeat rarely
   - -a is used for adhoc commands:
     - ansible all -a "df -h" -u ubuntu
     - ansible servers -a "uptime" -u ubuntu
   - Ansible modules are units of code that can control system resources or execute system commands
   - -m is used for modules:
     - ansible all -m ping -u ubuntu

8. **ansible.cfg** - Main configuration file for Ansible settings.
9. **Ansible Facts** - Information about a managed node that is automatically discovered by Ansible during the execution of playbooks or tasks.


## Infrastructure Setup

1. **Create EC2 Instances**

   * Create an `ansible-master` instance with a PEM key.
   * Create 3 instances named `server-1`, `server-2`, and `server-3` using the same PEM key.

## Installing Ansible on the Master Server

```bash
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
ansible --version
```

## Configuring the Inventory File

Edit the Ansible hosts file to include your server IPs:

```bash
sudo vim /etc/ansible/hosts
```

Add the following:

```ini
[devservers]
server_1 ansible_host=<Public_IP_1>
server_2 ansible_host=<Public_IP_2>

[prdservers]
server_3 ansible_host=<Public_IP_3>

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/keys/pem_key_name.pem
```
Note: Instead of [all:vars], you can specify any grp like [devservers:vars] or [prdservers:vars].

## Transferring the PEM Key

1. Create a directory on the master server to store the PEM key:

```bash
mkdir keys
cd keys
pwd
```

2. Copy ssh command of master instance for future use. (Ex. ssh -i "pem_key_name.pem" ubuntu@ec2-54-152-198-35.compute-1.amazonaws.com)

3. Give required permission to pem file in local system.

4. Copy the PEM file from your local machine to the master server byrunning following command in cmd where PEM file is downloaded:

```bash
scp -i "pem_key_name.pem" pem_key_name.pem ubuntu@ec2-54-152-198-35.compute-1.amazonaws.com:/home/ubuntu/keys
```

5. Give permission to newly copied pem_key_name.pem inside master server:

```bash
chmod 400 pem_key_name.pem
```

## Verifying Connectivity and run commands

```bash
ansible devservers -m ping
ansible prdservers -m ping
ansible all -m ping

ansible devservers -a "free -h"
ansible all -a "sudo apt update"
ansible prdservers -a "sudo uptime"
```

## Running Ansible Playbooks

To list the inventory:

```bash
ansible-inventory --list
```

To run a playbook:

```bash
ansible-playbook -v playbook_name.yml
ansible-playbook -v playbook_name.yml --limit devservers
```

## Example Playbooks
```bash
mkdir playbooks
cd playbooks
vim date_play.yml
```
### Date Playbook (date_play.yml)

```yaml
-
  name: Date Playbook
  hosts: devservers
  tasks:
    - name: Show the current date
      command: date
    - name: Show the system uptime
      command: uptime
```

### Install Nginx Playbook (install_nginx_play.yml)

```yaml
-
  name: Install Nginx and Start Service
  hosts: devservers
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: latest
    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

### Conditional Statements (conditional_statement_play.yml)

```yaml
-
  name: Install Packages Based on OS
  hosts: devservers
  become: yes
  tasks:
    - name: Install Docker
      apt:
        name: docker.io
        state: latest
    - name: Install AWS CLI
      apt:
        name: awscli
        state: latest
      when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'
```

### Deploy Static Webpage (deploy_static_page_play.yml)

```yaml
-
  name: Deploy Static Webpage
  hosts: prdservers
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: latest
    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
    - name: Deploy webpage
      copy:
        src: index.html
        dest: /var/www/html
```

## Ansible VS Chef

- Ansible (Push Based)- 1 server (master) have Ansible installed, it pushes updates on all other servers.
- Chef (Pull Based)- Bring Configuration from all servers and than updates them.

## Conclusion

This guide covers the basics of Ansible setup, from installing the tool to running playbooks. For more advanced use cases, consider exploring roles, Ansible Galaxy, and dynamic inventories.

## Author 

Abhishek Rajput
