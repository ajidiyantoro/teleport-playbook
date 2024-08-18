# Teleport Agent Deployment with Ansible

Teleport is a platform to provides connectivity, authentication, access controls and audit for infrastructure. This project provide how to deploy new Teleport Agent to new instance and update existing deployment using Ansible.

- Make sure you have a teleport server running, to run the teleport server you can refer to [Teleport Documentation](https://goteleport.com/docs/installation/)
- Teleport Client (`tsh` and `tctl`) has installed on your control node
- Control Node (the machine you are running `ansible` commands) must have Ansible 2.11+

## Getting Started

Before running the playbook, edit the following configuration: 

### Default Variables

```yaml
# → teleport/default/main.yml

temp_dir: /tmp
mode: "install"
teleport_proxy_addr: teleport.example.com:443
teleport_ca_pin: sha256:xxx
repository_url: https://cdn.teleport.dev
teleport_pkg: teleport
teleport_version: 16.1.4
system_arch: amd64
```
Notes: 
- `temp_dir`: path to store downloaded archives
- `mode`: set to `"install"` for new deployment or `update` for update/upgrade from existing version
- `teleport_proxy_addr`: address of your teleport server `example.teleport.com:443`
- `teleport_ca_pin`: (optional) If you're running Teleport Server using self-signed certificate, you must provide this variable, run `tctl status` to get ca_pin.
- `repository_url`: default `https://cdn.teleport.dev`
- `teleport_pkg`: default `teleport`
- `teleport_version`: target version of teleport you want to install/update
- `system_arch`: systecm architecture of your instance `(amd64,arm64,arm,386)`

### Inventory

Modify the `inventory.yml` file and according to the number of instances used. for example:

```yaml
# → inventory.yml

all:
  hosts:
    vm-demo-1:
      ansible_host: 192.168.1.1
    vm-demo-2:
      ansible_host: 192.168.1.2

  children:
    teleport-instance:
      hosts:
        vm-demo-1:
          teleport_join_token: ""
        vm-demo-2:
          teleport_join_token: ""
      vars:
        ansible_user: root
        ansible_ssh_private_key_file: ~/.ssh/id_rsa
        ansible_host_key_cheching: false

```
Notes: 
- `teleport_join_token` variable must be set on every instance if you want to new deployment / `mode: "install" `
- to generate join token use this command: `tctl tokens add --type=node`
- For the authentication section, change the value in the vars segment

### Run Playbook

```bash
ansible-playbook -i inventory.yml teleport.yml
```