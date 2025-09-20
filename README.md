# Ansible Setup and Configuration Guide

## Table of Contents

* [Introduction to Ansible](#introduction-to-ansible)
* [Key Concepts](#key-concepts)
* [Infrastructure Setup](#infrastructure-setup)
* [Installing Ansible on the Master Server](#installing-ansible-on-the-master-server)
* [Transferring the Key so that master can access host on AWS](#transferring-the-key-so-that-master-can-access-host-on-aws)
* [Transferring the Key so that master can access host on GCP](#transferring-the-key-so-that-master-can-access-host-on-gcp)
* [Configuring the Inventory File](#configuring-the-inventory-file)
* [Verifying Connectivity and run commands](#verifying-connectivity-and-run-commands)
* [Playbooks](#playbooks)
* [Running Ansible Playbooks](#running-ansible-playbooks)
* [Ansible VS Chef](#ansible-vs-chef)
* [Conclusion](#conclusion)
* [Author](#author)

---

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
5. **Roles** - Reusable playbooks with organized structure. They encapsulate related Tasks, Variables, Files, Templates and Handlers into a directory structure.
6. **Ad-Hoc Commands vs Modules** - Quick commands (`-a`) vs structured modules (`-m`)
   - ad hoc commands are great for tasks you repeat rarely
   - -a is used for adhoc commands:
     - ansible all -a "df -h" -u ubuntu
     - ansible servers -a "uptime" -u ubuntu
   - Ansible modules are units of code that can control system resources or execute system commands
   - -m is used for modules:
     - ansible all -m ping -u ubuntu

8. **ansible.cfg** - Main configuration file for Ansible settings.

---

## Infrastructure Setup

1. **Create Instances**

   * Create an `ansible-master` instance with a PEM key.
   * Create 3 instances named `server-1`, `server-2`, and `server-3` using the same PEM key.

## Installing Ansible on the Master Server

```bash
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
ansible --version
```

## Transferring the Key so that master can access host on AWS

1. Create a directory on the master server to store the PEM key:

```bash
mkdir keys
cd keys
pwd
```

2. Copy ssh command of master instance for future use. (Ex. ssh -i "pem_key_name.pem" ubuntu@ec2-54-152-198-35.compute-1.amazonaws.com)

3. Give required permission to pem file in local system.

4. Copy the PEM file from your local machine to the master server by running following command in cmd where PEM file is downloaded:

```bash
scp -i "pem_key_name.pem" pem_key_name.pem ubuntu@ec2-54-152-198-35.compute-1.amazonaws.com:/home/ubuntu/keys
```

5. Give permission to newly copied pem_key_name.pem inside master server:

```bash
chmod 400 pem_key_name.pem
```

## Transferring the Key so that master can access host on GCP

1. Generate key pair on your master VM:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
```

2. Copy the public key for future use:

```bash
cat ~/.ssh/id_rsa.pub
```

3. Copy the public key to each VM and set permissions:

```bash
mkdir -p ~/.ssh
echo "<YOUR_PUBLIC_KEY_CONTENT_FROM_ABOVE_STEP>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

4. Testing from master instance for each VM:
```bash
ssh -i ~/.ssh/id_rsa username@<HOST_PUBLIC_IP>
```
ex: ssh -i ~/.ssh/id_rsa abhishek25022004@35.193.55.228

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

## Verifying Connectivity and run commands

```bash
ansible devservers -m ping
ansible prdservers -m ping
ansible all -m ping

ansible devservers -a "free -h"
ansible all -a "sudo apt update"
ansible prdservers -a "sudo uptime"
```

---

## Playbooks
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
```
### Deploy Static Webpage (deploy_static_page_play.yml)

```yaml
-
  name: Deploy Static Webpage
  hosts: all
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

---

## Ansible VS Chef
Ansible and Chef are popular configuration management tools, each with its own approach to automating IT infrastructure. Understanding their differences can help you choose the right tool for your environment.

### **Key Differences:**

| **Aspect**              | **Ansible (Push-Based)**                                                                  | **Chef (Pull-Based)**                                                                |
| ----------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| **Architecture**        | Single server (master) with Ansible installed pushes configurations to all managed nodes. | Managed nodes (clients) pull configurations from a central Chef server.              |
| **Communication**       | Uses agentless SSH for communication, reducing overhead and simplifying setup.            | Requires Chef clients on each node, which periodically pull updates from the server. |
| **Ease of Use**         | Simple YAML-based playbooks make it easy to get started.                                  | More complex, using Ruby-based recipes and cookbooks.                                |
| **Configuration Model** | Declarative, focusing on defining the desired state of the infrastructure.                | Primarily declarative, but supports procedural logic through Ruby.                   |
| **Scalability**         | Ideal for small to medium-sized infrastructures.                                          | Designed for large-scale, complex environments.                                      |
| **Setup Time**          | Faster initial setup due to its agentless design.                                         | Slower setup, requiring client installation and configuration.                       |

* Use **Ansible** if you prefer a simpler, agentless approach with faster setup.
* Use **Chef** for larger infrastructures where complex workflows and scalability are critical.

## Conclusion

This guide covers the basics of Ansible setup, from installing the tool to running playbooks.

## Author 

Abhishek Rajput
