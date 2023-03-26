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

```shell
ansible-playbook -l some-host service/.../playbook.yml
```

## Catalog

### Basic setup

- Non-Root User
  - Use the public key from 1Password
  - Use the public key from GitHub
- Setup Firewall

### Docker

- Setup Docker Engine

## Playbook Rules

- Keep a link to the documentation at the top of the playbook
- Only setting `become: true` when needed
- Only setting `gather_facts: true` when needed
- Use `docker compose` over `docker`
- Create an actual `docker-compose.yml` instead of inline-create them in playbooks
- Don't use `community.docker.docker_compose` at the moment [^1]
- Inject network configs to compose file, instead of attaching container with ansible task

  ```yaml
  vars_prompt:
    - name: "docker_network"
      prompt: "Enter the name of the docker network to use"
      default: "a-default-network-name-goes-here"

  tasks:
    ...
    - name: Add networks block if docker_network is set
      when: docker_network is defined
      blockinfile:
        path: "{{ service_dir }}/docker-compose.yml"
        block: |
          networks:
            default:
              external: true
              name: "{{ docker_network }}"
  ```

- Always offer to add DNS entry when applicable

  ```yaml
  tasks:
    ...
    - name: Setup DNS record
      import_tasks: ../utils/cloudflare_dns.yml
  ```

## Compose File Rules

- Don't add the service prefix to compose service name (e.g. just use `app` or `portainer` instead of `portainer_app`)
- Don't add the service prefix to compose volume name [^2]
- Always set the `container_name` for each service
- Pin image version at minor level (e.g. `image: nginx:1.19`) if not specified in the documentation

[^1]: Most server have compose v2 installed, which is not supported
[^2]: Prefix will be added automatically
