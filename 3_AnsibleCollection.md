# Ansible Collections

**Ansible Collections** are a standardized way to package and distribute all types of Ansible content — such as roles, playbooks, modules, plugins, and documentation — under a **single, versioned namespace**.

Collections make it easier to:

* Organize content by domain or use-case
* Share and reuse automation assets
* Version and publish updates consistently

---

# Ansible Galaxy

**Ansible Galaxy** is the **community-driven platform** for finding, sharing, and downloading Ansible Collections and Roles. It's hosted at [galaxy.ansible.com](https://galaxy.ansible.com).

Key features:

* Free and open to everyone
* Community and vendor-maintained collections
* Can be used with `ansible-galaxy` CLI

Install a collection from Galaxy:

```bash
ansible-galaxy collection install community.general
```

---

# Automation Hub

**Red Hat Automation Hub** is the **enterprise-grade alternative to Galaxy**, available with **Red Hat Ansible Automation Platform (AAP)**. It provides:

* Official, fully-supported collections from Red Hat and partners
* Verified content for production use
* Private content hosting for organizations
* Integration with automation controller and content signer

Organizations use **Automation Hub** to ensure security, stability, and compliance of automation content.

Use it by configuring an Ansible config or automation controller to point to:

```ini
https://<your-org>/automation-hub/api/
```

---

## 🔗 Summary Table

| Feature      | Ansible Galaxy                                   | Automation Hub                                          |
| ------------ | ------------------------------------------------ | ------------------------------------------------------- |
| Source       | Community                                        | Enterprise (Red Hat)                                    |
| Content Type | Community Roles/Collections                      | Certified/Verified Collections                          |
| Use Case     | Open source, non-critical environments           | Production, secure and audited environments             |
| URL          | [galaxy.ansible.com](https://galaxy.ansible.com) | Hosted by your Red Hat AAP instance or cloud.redhat.com |

---

This collection provides a modular and reusable Ansible role to **install and configure Apache** with a customizable homepage using a Jinja2 template.

## 📦 Structure

This role lives inside an Ansible **Collection** named `webops.apache`.

```
collections/
└── ansible_collections/
    └── webops/
        └── apache/
            ├── roles/
            │   └── apache/
            │       ├── tasks/
            │       ├── templates/
            │       ├── defaults/
            │       ├── vars/
            │       ├── handlers/
            │       └── meta/
            ├── README.md
            └── galaxy.yml
```

---

## Getting Started

### 1. Initialize the Collection

```bash
ansible-galaxy collection init webops.apache --init-path ./collections/ansible_collections
```

### 2. Copy the Existing Role into the Collection

```bash
cp -r apache/ collections/ansible_collections/webops/apache/roles/
```

---

## Sample Playbook

### `role-playbook.yml`

```yaml
---
- name: Apply Apache Collection
  hosts: webservers
  become: yes
  roles:
    - webops.apache.apache
```

---

## Using External Collections

To include additional functionality from external collections (e.g. from **Automation Hub** or **Ansible Galaxy**), define them in a `requirements.yml` file:

### `requirements.yml`

```yaml
collections:
  - name: snagoor.aap_common       # From private Automation Hub or Galaxy
  - name: community.general        # Popular collection from Ansible Galaxy
```

Then install with:

```bash
ansible-galaxy collection install -r requirements.yml
```

---

## Run the Playbook

```bash
ansible-playbook -i inventory/hosts role-playbook.yml
```

---

## Publishing or Sharing the Collection

To publish the collection to **Ansible Galaxy** or **Automation Hub**:

1. Populate `galaxy.yml` with metadata (name, version, author, dependencies, etc.).
2. Build and optionally publish:

```bash
ansible-galaxy collection build
ansible-galaxy collection publish webops-apache-*.tar.gz
```

## Author

Abhishek Rajput

