# sothanav-ansible-playbook

Ansible playbook to collect and report server hardware/OS information (CPU, memory, disk, OS) from remote hosts.

## Project Structure

```
sothanav-ansible-playbook/
├── inventory.ini              # Host inventory
├── report-servers/
│   └── server_report.yml      # Main playbook
└── reports/                   # Output reports (auto-created)
```

## Requirements

- Ansible 2.9+
- SSH access to target hosts
- Python 3 on target hosts

## Setup

1. Edit `inventory.ini` and replace the example IPs/users with your actual hosts:

```ini
[webservers]
web01 ansible_host=<YOUR_IP> ansible_user=<YOUR_USER>
```

2. Ensure your SSH key is in place (default: `~/.ssh/id_rsa`), or set `ansible_ssh_private_key_file` in the inventory.

## Usage

Run the server report playbook against all hosts:

```bash
ansible-playbook -i inventory.ini report-servers/server_report.yml
```

Run against a specific group only:

```bash
ansible-playbook -i inventory.ini report-servers/server_report.yml --limit webservers
```

Test connectivity first:

```bash
ansible -i inventory.ini all -m ping
```

## Output

- Each host writes a report to `/tmp/report.txt` on the remote machine.
- Reports are fetched back to `./reports/<hostname>_report.txt` on the control node.
