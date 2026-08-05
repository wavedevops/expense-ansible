# 1. Using Only Ansible Push

    If we use the **Ansible Push** model:

* This playbook follows a **declarative approach** because we predefine which roles should be applied to which hosts.
* The control node pushes the configuration to the target servers.
* We must maintain an **inventory file** that defines the host groups (`mysql`, `backend`, `frontend`).
* The playbook explicitly maps each host group to its corresponding role.

### Example Playbook

```yaml
- name: mysql
  hosts: mysql
  become: true
  tags:
    - mysql
  roles:
    - mysql

- name: backend
  hosts: backend
  become: true
  tags:
    - backend
  roles:
    - backend

- name: frontend
  hosts: frontend
  become: true
  tags:
    - frontend
  roles:
    - frontend
```

### Commands

Run all components:

```bash
ansible-playbook -i inventory.ini expense.yaml
```

Run only the frontend component:

```bash
ansible-playbook -i inventory.ini expense.yaml --limit frontend --tags frontend
```

### Advantages

* Easy to understand and manage.
* Centralized management from a single control node.
* Suitable for small and medium-sized environments.
* No Git installation is required on the managed nodes.
* Easy to execute ad hoc commands and one-time tasks.
* Individual components can be deployed using `--limit` and `--tags`.

### Limitations

* Requires maintaining an inventory file.
* The control node must have SSH connectivity to all managed servers.
* Scaling becomes more difficult as the number of servers increases.
* Adding a new server requires updating the inventory before it can be managed.

---

# 2. Using Ansible Pull

> **This playbook supports both Ansible Push and Ansible Pull.**

    If we use the **Ansible Pull** model:

* This playbook is **dynamic** because the role to execute is selected at runtime using the `component` variable.
* Each server pulls the latest playbook and roles from the Git repository and configures itself.
* It does **not require a centralized inventory** of all servers. The playbook runs only on the local machine (`localhost`).
* The same playbook can be used to configure multiple components (`frontend`, `backend`, `db`) by passing the appropriate `component` variable.
* This approach is commonly used with **Terraform**, where Terraform provisions the server and then triggers `ansible-pull` to configure it.

### Example Playbook

```yaml
- hosts: localhost
  connection: local
  become: true
  roles:
    - common
    - "{{ component }}"
```

### Commands

```bash
# Frontend
ansible-pull -i localhost, -U https://github.com/wavedevops/expense-ansible.git -e env=dev -e component=frontend expense.yaml

# Backend
ansible-pull -i localhost, -U https://github.com/wavedevops/expense-ansible.git -e env=dev -e component=backend expense.yaml

# Database
ansible-pull -i localhost, -U https://github.com/wavedevops/expense-ansible.git -e env=dev -e component=mysql expense.yaml
```

### Advantages

* A single playbook can configure all components.
* No centralized inventory management is required.
* Each server configures itself by pulling the latest code from Git.
* Well suited for cloud environments, auto-scaling groups, and immutable infrastructure.
* Easy to integrate with Terraform, EC2 User Data, systemd, cron jobs, and CI/CD pipelines.
* Reduces the need for direct SSH access from a central control node to every managed server.

### Limitations

* Each server must have access to the Git repository.
* Troubleshooting is performed on the target server instead of a central control node.
* Configuration changes are applied only when `ansible-pull` is executed (manually, through cron, a systemd timer, Terraform, or another automation).
* Secrets and repository access must be managed securely on each server.
