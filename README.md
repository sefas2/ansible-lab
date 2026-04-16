Ansible Network Automation Lab
A hands-on network automation lab using FRRouting containers as virtual routers,
built to demonstrate Ansible best practices for NetDevOps roles.
The lab covers the full configuration lifecycle — from structured inventory and
encrypted credentials through to template-driven config deployment, post-change
validation, and idempotency testing.
---
Lab Topology
```
                    ┌─────────────────┐
                    │  Ansible Control │
                    │      Node        │
                    └────────┬────────┘
                             │ SSH (via localhost ports)
          ┌──────────────────┼──────────────────┐
          │                  │                  │
    ┌─────▼─────┐     ┌──────▼────┐     ┌──────▼────┐     ┌───────────┐
    │router-east│     │router-west│     │router-north│    │router-south│
    │ 10.0.0.1  │     │ 10.0.0.2  │     │ 10.0.0.3  │    │ 10.0.0.4  │
    │ port:2211 │     │ port:2212 │     │ port:2221 │    │ port:2222  │
    └───────────┘     └───────────┘     └───────────┘    └────────────┘
         FRRouting containers running via Docker
```
All routers run FRRouting 8.4 in Docker containers, accessible via SSH on
different localhost ports.
---
📋 Playbooks
Playbook	Purpose
`loopbacks.yaml`	Configures loopback interfaces on all routers using `host_vars`
`validate_loopbacks.yaml`	Validates loopback IPs are correctly applied using `register`, `debug`, and `assert`
`deploy_config.yaml`	Deploys Jinja2 template config with handler-triggered FRR reload on change
`site.yaml`	Master playbook — runs all roles
`backup.yaml`	Backs up running configuration from all routers
---
Project Structure
```
ansible-lab/
├── ansible.cfg                  ← project-level Ansible config
├── site.yaml                    ← master playbook
├── loopbacks.yaml               ← loopback interface configuration
├── validate_loopbacks.yaml      ← post-change validation playbook
├── deploy_config.yaml           ← template-driven config deploy with handlers
├── backup.yaml                  ← config backup playbook
├── inventory/
│   ├── hosts.yaml               ← YAML inventory with all 4 routers
│   ├── group_vars/
│   │   └── routers.yaml         ← shared vars (connection, user, vault-encrypted password)
│   └── host_vars/
│       ├── router-east.yaml     ← loopback IP, router ID
│       ├── router-west.yaml
│       ├── router-north.yaml
│       └── router-south.yaml
├── templates/
│   └── frr_loopback.j2          ← Jinja2 FRR config template
├── roles/
│   └── ...                      ← Ansible roles
└── docker_files/                ← FRRouting Docker setup
```
---
⚙️ Setup
1. Prerequisites
```bash
# Install Ansible
pip install ansible

# Install required collections
ansible-galaxy collection install ansible.netcommon
ansible-galaxy collection install community.network
```
2. Start the lab routers
```bash
cd docker_files
docker-compose up -d
```
3. Configure Ansible Vault password
```bash
echo 'your_vault_password' > ~/.vault_pass
chmod 600 ~/.vault_pass
```
Make sure `ansible.cfg` points to `~/.vault_pass`:
```ini
vault_password_file = ~/.vault_pass
```
4. Verify inventory
```bash
ansible-inventory -i inventory/hosts.yaml --list
```
---
Running Playbooks
Configure loopback interfaces:
```bash
ansible-playbook loopbacks.yaml
```
Validate loopbacks are correctly applied:
```bash
ansible-playbook validate_loopbacks.yaml
```
Deploy config from Jinja2 template:
```bash
ansible-playbook deploy_config.yaml
```
---
Key Concepts Demonstrated
Variable Management
YAML inventory (`hosts.yaml`) replacing flat INI inventory
`group_vars` for shared router settings
`host_vars` for per-device values (loopback IPs, router IDs)
Ansible Vault for encrypted credential storage
Validation
`register` — capturing device command output
`debug` — inspecting output during playbook runs
`assert` — explicit pass/fail checks against device state
Template-driven Config
Jinja2 templates (`.j2`) for device configuration
`lookup('template', ...)` to render and push configs
Handlers triggered only on actual config changes
Idempotency — second run shows `changed=0`, handler silent
---
Tech Stack
Tool	Purpose
Ansible	Automation framework
FRRouting	Virtual router (BGP, OSPF)
Docker	Lab containerisation
Jinja2	Config templating
Ansible Vault	Credential encryption
ansible.netcommon	Network device modules
