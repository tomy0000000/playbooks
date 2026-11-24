---
paths:
  - "service/**"
---

# First Principle

- Write each service playbook as if it's running on the host for the first time. After a run, the host must be in the state the playbook describes, whatever state it started in: a fresh host ends with the service installed and running, and an existing install ends upgraded to the pinned version.
- Describe the end state, not the steps from an assumed starting point. Create directories, networks and files with modules that reach the same result on every run (`file`, `copy`, `docker_network`), then run `docker compose pull` and `docker compose up --detach`, which do nothing when the service is already current.
- Don't split a playbook into separate install and upgrade paths, and don't skip tasks because a file or container already exists. Those checks can disagree with the host's real state and leave a half-installed host stuck.
- Read existing state only for a step that can't be repeated, such as joining a tailnet with a single-use key. Check the service's own state, not a marker file (see `secrets.md`).
- A second run right after a successful one must change nothing.
