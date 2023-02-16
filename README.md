# Playbooks

My ansible playbooks for setting up servers and services.

## Usage

1. Setup SSH server with [dotfiles](https://github.com/tomy0000000/dotfiles) on the remote host
2. Setup server connection in the inventory file
   ```ini
   [$GROUP_NAME]
   $SERVER_NAME   ansible_host=$SERVER_IP
   ```
3. Run the `ansible-setup.yml` playbook
4. Run the rest of the playbooks needed

## Catalog

### Basic setup

- Non-Root User
  - Use the public key from 1Password
  - Use the public key from GitHub
- Setup Firewall

### Docker

- Setup Docker Engine

## Playbook Rules

- Only setting `become: true` when needed
- Only setting `gather_facts: true` when needed
- Use `docker compose` over `docker`
- Don't use `community.docker.docker_compose` at the moment [^1]
- Don't add the service prefix to compose service name (e.g. just use `app` or `portainer` instead of `portainer_app`)
- Always set the `container_name` for each service
- Don't add the service prefix to compose volume name [^2]
- Inject network configs to compose file, instead of attach container with ansible task

[^1]: Most server have compose v2 installed, which is not supported
[^2]: Prefix will be added automatically
