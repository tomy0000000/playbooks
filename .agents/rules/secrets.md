---
paths:
  - "service/**"
  - "setup/**"
  - "utils/**"
---

# Secrets on target hosts

- Don't leave a plaintext secret on the target host: no `.env`, config file, or compose file holding a key, token, or password. A host compromise or a careless backup shouldn't hand out credentials.
- One-time credentials (e.g. Tailscale auth keys):
  - Detect first install from the service's own state (e.g. `tailscale status` reporting `NeedsLogin`), not from whether a file exists. Then the playbook re-joins correctly after a device is removed.
  - Prompt for the credential only on first install, with `ansible.builtin.pause` and `echo: false`. Don't use `vars_prompt`, which asks on every run.
  - Pass it to the process through the command's `environment:` with `no_log: true`, read in compose as `${VAR:-}`. Never write it to a file.
  - Once the service has used it, recreate the container without it, so it also leaves the container's environment and `docker inspect`.
- How long-lived secrets (database passwords, API tokens) reach a host isn't decided yet. Ask the user instead of writing them to a file.
