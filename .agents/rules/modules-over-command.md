---
paths:
  - "service/**"
  - "setup/**"
  - "utils/**"
---

# Modules over command

- Use a dedicated module instead of `ansible.builtin.command` or `ansible.builtin.shell` whenever one exists. Modules report real change status and check mode, so playbooks don't need `changed_when` guesses to stay idempotent (see `first-principle.md`).
- Docker equivalents:
  - `docker compose up`, `pull`: `community.docker.docker_compose_v2` (with `pull: always` to upgrade in the same task)
  - `docker exec`: `community.docker.docker_container_exec`
  - `docker network create`: `community.docker.docker_network`
- Search `community.docker`, `community.general` and `ansible.posix` before falling back to `command`. If no module fits, use `command` and set `changed_when` to reflect what actually happened.
- `community.docker.docker_compose` (v1) is a different module and doesn't work with Compose v2. Don't confuse it with `docker_compose_v2`.
