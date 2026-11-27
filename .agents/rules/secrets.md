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
- Long-lived secrets that a systemd unit uses (e.g. the R2 key of a backup job):
  - Keep the secret in 1Password, vault `Develop`, and read it on the machine running Ansible with the `community.general.onepassword` lookup. Document how to create the item in `service/secrets.md`.
  - Store it on the host with `systemd-creds encrypt` under `/etc/credstore.encrypted/`, and load it in the unit with `LoadCredentialEncrypted=`. The process reads it from `$CREDENTIALS_DIRECTORY`, so the plaintext never touches the disk.
  - Decrypt and compare before encrypting, with `no_log: true`. The ciphertext differs on every run, so a blind re-encrypt always reports a change. `utils/backup/tasks.yml` shows the pattern.
- How long-lived secrets reach a container (e.g. a database password in compose) isn't decided yet. Ask the user instead of writing them to a file.
