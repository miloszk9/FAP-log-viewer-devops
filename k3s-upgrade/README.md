# K3s & Ubuntu Upgrade Ansible Playbook

This Ansible playbook automates the minor/patch upgrade process for a K3s cluster (1 Control Plane + 1 Agent) and its underlying Ubuntu OS. Designed for zero-downtime workload migration using standard Kubernetes graceful degradation patterns.

## Prerequisites

- Ansible installed on the controller node.
- SSH access to the cluster nodes with sudo privileges.

## Configuration

Set your target K3s version in `group_vars/all.yml`. 
*Always verify available minor/patch releases at [K3s GitHub Releases](https://github.com/k3s-io/k3s/releases).*

## Usage

Run the playbook to upgrade the cluster:

```bash
ansible-playbook -i inventory.yml upgrade-cluster.yml -v
```

To upgrade only K3s:

```bash
ansible-playbook -i inventory.yml upgrade-cluster.yml --skip-tags "os_upgrade" -v
```

To upgrade only OS:

```bash
ansible-playbook -i inventory.yml upgrade-cluster.yml --skip-tags "k3s_upgrade" -v
```
