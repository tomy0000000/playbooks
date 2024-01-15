# Disable local DNS

Disable the `systemd-resolved` service, backup the original configuration file and replace it with a new one that uses the Cloudflare DNS servers.

## Usage

### Run with playbook

Use [`disable-local-dns.yml`](../../setup/disable-local-dns.yml) playbook.

### Run as task

```yml
...
  tasks:
    ...
    - name: Disable local DNS
      become: true
      ansible.builtin.import_tasks: ../../utils/disable-local-dns/tasks.yml
    ...
  handlers:
    ...
    - name: Handlers for disabling local DNS
      become: true
      ansible.builtin.import_tasks: ../../utils/disable-local-dns/handlers.yml
    ...
```
