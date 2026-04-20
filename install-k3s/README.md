# Install K3s (HA Multi-Master + Multi-Worker)

Ansible playbook to install a highly available K3s cluster using embedded etcd, with multiple master and worker nodes.

## Requirements

- Ansible 2.9+
- Target nodes running Ubuntu/Debian or RHEL/CentOS
- SSH access to all nodes
- Internet access on all nodes (to download k3s installer)

## Inventory

Ensure your inventory has `[masters]` and `[workers]` groups. The **first host** in `[masters]` is used as the bootstrap node.

```ini
[masters]
tlnw-prd-insm01 ansible_host=10.16.2.13
tlnw-prd-insm02 ansible_host=10.16.2.14
tlnw-prd-insm03 ansible_host=10.16.2.15

[workers]
tlnw-prd-insw01 ansible_host=10.16.2.16
tlnw-prd-insw02 ansible_host=10.16.2.17
tlnw-prd-insw03 ansible_host=10.16.2.18

[all:vars]
ansible_user=root
ansible_ssh_private_key_file=~/Desktop/ssh/dsm/privatekey
```

## Usage

```bash
# Run with a strong cluster token (required)
ansible-playbook -i inventory.ini install-k3s/install-k3s.yml \
  -e k3s_token="your-strong-secret-token"
```

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `k3s_version` | `stable` | K3s release channel (`stable`, `latest`, or specific version e.g. `v1.29.3+k3s1`) |
| `k3s_token` | `changeme-set-a-strong-token` | Shared secret for nodes to join the cluster — **always override this** |

## Architecture

```
                    ┌─────────────────────────────┐
                    │        K3s HA Cluster        │
                    │                             │
                    │  ┌─────────┐  ┌─────────┐  │
                    │  │ master1 │  │ master2 │  │
                    │  │(bootstrap)│ │         │  │
                    │  └────┬────┘  └────┬────┘  │
                    │       │  etcd      │        │
                    │  ┌────┴────────────┴────┐   │
                    │  │       master3        │   │
                    │  └─────────────────────┘   │
                    │                             │
                    │  ┌───────┐ ┌───────┐ ┌───────┐ │
                    │  │worker1│ │worker2│ │worker3│ │
                    │  └───────┘ └───────┘ └───────┘ │
                    └─────────────────────────────┘
```

## Plays

| # | Play | Targets | Description |
|---|------|---------|-------------|
| 1 | Prepare all nodes | masters + workers | Disable swap, load kernel modules, set sysctl, install prerequisites |
| 2 | Bootstrap first master | `masters[0]` | Install k3s with `--cluster-init` (embedded etcd), save node token |
| 3 | Join additional masters | `masters[1:]` | Join cluster as HA server nodes |
| 4 | Join workers | workers | Join cluster as agent nodes |
| 5 | Verify cluster | `masters[0]` | Run `kubectl get nodes -o wide` and display output |

## Access kubeconfig

After installation, kubeconfig is available on each master at `/etc/rancher/k3s/k3s.yaml`.

```bash
# Copy to local machine
scp stoadmin1@10.16.2.13:/etc/rancher/k3s/k3s.yaml ~/.kube/config

# Update the server address to the master IP
sed -i 's/127.0.0.1/10.16.2.13/' ~/.kube/config
```

## Uninstall

```bash
# On master nodes
/usr/local/bin/k3s-uninstall.sh

# On worker nodes
/usr/local/bin/k3s-agent-uninstall.sh
```
