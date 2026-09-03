# 09 · Automation — Ansible

> **TODO** — fill in the blanks marked below and drop playbooks into this folder.

**Task objective.** Replace repeated manual configuration and checking with
playbooks whose scope is explicit, whose results are traceable, and which are
safe to run twice.

Control node: srv-auto01 (192.168.50.10) · SSH key auth to managed hosts

## Inventory

```ini
[web]
srv-web01 ansible_host=192.168.60.10

[infra]
srv-dns01 ansible_host=192.168.40.10

[all:vars]
ansible_user=ops01
ansible_python_interpreter=/usr/bin/python3
```

```bash
ansible-inventory -i inventory.ini --graph
ansible all -i inventory.ini -m ping
```

## Playbooks

| Playbook | Purpose | Target group | Idempotent |
|---|---|---|---|
| `baseline_check.yml` | Report failed units and disk usage | all | ✅ (read-only) |
| `TODO` | | | |

Example — the read-only health check. `changed_when: false` on the command tasks
matters: without it every run reports "changed", and once a playbook always
reports changes you stop reading its output.

```yaml
---
- name: Baseline status check
  hosts: all
  gather_facts: true
  become: true
  tasks:
    - name: Check failed services
      command: systemctl --failed --no-legend
      register: failed_services
      changed_when: false
      failed_when: false

    - name: Show disk usage
      command: df -hT
      register: disk_status
      changed_when: false
```

## Execution discipline

```bash
ansible-playbook -i inventory.ini <play>.yml --syntax-check
ansible-playbook -i inventory.ini <play>.yml --check --limit <one host>
ansible-playbook -i inventory.ini <play>.yml --limit <group>
```

Syntax check, then dry run against one host, then the target group. Never a bare
run against `all` on first execution.

Sensitive variables go in Ansible Vault. No plaintext credentials in the
inventory or in this repository.

## Idempotency

The acceptance criterion: run the same playbook twice; the second run reports no
changes.

| Playbook | Run 1 (changed) | Run 2 (changed) | Idempotent |
|---|---|---|---|
| `baseline_check.yml` | | | |
| `TODO` | | | |

`TODO: if a playbook was not idempotent on the first attempt, say what caused it
and how you fixed it — a task using command instead of a module is the usual
culprit, and describing that is a real demonstration of understanding.`

## Known limitations

`TODO — e.g. no roles, only flat playbooks? no CI check? network devices not
managed by Ansible? no Vault usage yet?`
