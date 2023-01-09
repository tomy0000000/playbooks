# Playbooks

My ansible playbooks for setting up servers.

## Usage

1. Setup SSH with [dotfiles](https://github.com/tomy0000000/dotfiles)
2. Setup server connection in the inventory file
   ```ini
   [$GROUP_NAME]
   $SERVER_NAME   ansible_host=$SERVER_IP
   ```
3. Run the `ansible-setup.yml` playbook
4. Run the playbook you want

## Catalog

## Basic setup

- Setup Firewall

## Docker

- Setup Docker Engine
