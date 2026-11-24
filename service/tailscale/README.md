# Tailscale

Joins the host to the tailnet with one Tailscale container. Apps on the host are published privately as [Tailscale Services](https://tailscale.com/kb/1552/tailscale-services) (`<name>.<tailnet>.ts.net`), so they need no public exposure.

## Before the first install

### 1. Allow a tag for the host

A Service host must be a tagged device, not one logged in as your user. In the admin console, go to **Access controls** and open the [JSON editor](https://console.tailscale.com/admin/acls/file). Add a tag for the host, merging into the existing `tagOwners` key if there is one:

```jsonc
"tagOwners": {
  "tag:device-name": ["autogroup:admin"],
},
```

### 2. Create an auth key

Go to **Settings** → [**Keys**](https://console.tailscale.com/admin/settings/keys) → **Generate auth key**:

- **Reusable:** off. The key only joins one device.
- **Ephemeral:** off. Otherwise the device is removed from the tailnet when it goes offline.
- **Tags:** the host's tag, e.g. `tag:device-name`.

Copy the key, and paste it when the playbook prompts for it. Tagged devices have key expiry disabled, so the device stays logged in after the auth key is spent.

## Usage

```shell
ansible-playbook -l some-host service/tailscale/playbook.yml
```

- **First install:** Tailscale reports `NeedsLogin`, so the playbook prompts for the auth key. The key is passed to the container for one run only, and the container is then recreated without it. It never lands on disk.
- **Re-runs and upgrades:** the device is already logged in, so there's no prompt. The playbook pulls the pinned image and recreates the container only if something changed.
- **Device removed from the tailnet:** the next run sees `NeedsLogin` again and prompts for a new key.

The device name comes from `TS_HOSTNAME` in `.env`, set to the inventory hostname. The login state lives in the `tailscale_state` volume, so keep it when recreating the container.

## Publishing an app as a Service

The playbook doesn't manage Services yet. Until it does, set each one up by hand. The steps below use NocoDB as the example.

### 1. Update the tailnet policy

In [**Access controls**](https://console.tailscale.com/admin/acls/file), merge these into the existing keys:

```jsonc
"autoApprovers": {
  "services": {
    "svc:nocodb": ["tag:device-name"],
  },
},
"grants": [
  // Keep any existing grants and add this one
  { "src": ["autogroup:member"], "dst": ["svc:nocodb"], "ip": ["443"] },
],
```

- `autoApprovers` approves the tagged host as the host of `svc:nocodb`, so you don't have to approve it by hand.
- The grant lets your devices reach the Service on port 443.

### 2. Define the Service

Go to **Services → Advertise → Define a Service**, set **Name** to `nocodb` and **Endpoints** to `tcp:443`, then click **Add service**.

### 3. Serve the app on the host

Connect the app container to the `tailscale` network, then serve it through the Service:

```shell
docker network connect tailscale nocodb-app
docker exec tailscale tailscale serve --service=svc:nocodb --https=443 http://nocodb-app:8080
```

The serve config is kept in the state volume, so it survives restarts. Inspect it with `docker exec tailscale tailscale serve get-config --all`.

Clients need Tailscale v1.86 or later to reach Services.
