# Service Secrets

Instructions for creating the credentials the service playbooks fetch from
1Password.

Every one of them is an item in the `Develop` vault. A playbook reads it on
the machine running Ansible, with the `community.general.onepassword` lookup,
so the `op` CLI there must be signed in. Nothing here is committed, and no
secret is left on a host in plaintext (see `.agents/rules/secrets.md`).

Items are named `<host>-<vendor>-<purpose>`: dashes separate the three parts,
underscores stand in for spaces inside one, so `lumos-cloudflare-nocodb_backup`
reads as host `lumos`, vendor `cloudflare`, purpose `nocodb backup`. `<host>`
is the inventory name of the host. Each host gets its own items, so two hosts
never share a token. Items use the **API Credential** type, so the token sits
in the `credential` field.

Rotation is editing the item and running the playbook that reads it again.

## `<host>-cloudflare-nocodb_backup`

R2 S3 key the nightly NocoDB backup uploads its archives with.
`utils/backup/tasks.yml` reads it when `service/nocodb/backup.yml` runs, and
stores it on the host as an encrypted systemd credential,
`/etc/credstore.encrypted/nocodb-backup.rclone.conf`. Only
`nocodb-backup.service` decrypts it.

1. Go to [Cloudflare dashboard](https://dash.cloudflare.com) → **R2** →
   **Manage API tokens** → **Create Account API token**
2. Name it (e.g. `<Host Name> NocoDB Backup`), pick **Object Read & Write**,
   and under **Specify bucket(s)** apply it to `tomy-backup-nocodb` only. The
   infrastructure repo declares that bucket as `buckets.nocodb.name`
3. **Create API Token**, then copy the **Access Key ID**, the **Secret Access
   Key** and the S3 endpoint. Cloudflare shows the secret only once
4. In 1Password, in `Develop`: **New Item** → **API Credential**, title
   `<host>-cloudflare-nocodb_backup`
5. Set **username** to the Access Key ID, and paste the Secret Access Key
   into **credential**
6. Set **hostname** to the S3 endpoint without `https://`, in the form
   `<account_id>.r2.cloudflarestorage.com`

Each run of the backup decrypts the credential fresh, so a rotated key is in
use from the first run after the playbook stores it. Nothing to restart.
