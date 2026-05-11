# inhive-public-config

Public mirror of encrypted bootstrap configuration for the [InHive VPN](https://app.inhive.ru) client.

## Purpose

InHive clients fetch a small encrypted manifest at startup to discover backup endpoints when the primary infrastructure is blocked. The primary location is `https://inhive.ru/api/bootstrap.json` — this repo is the **public fallback** that works through GitHub raw and jsDelivr CDN, both of which remain accessible in regions where `inhive.ru` is DPI-blocked (e.g. Russian mobile carriers).

## Wire format

```json
{
  "version": 1,
  "key_version": "v1",
  "updated_at": "2026-05-11T16:42:05Z",
  "encrypted_payload": "BASE64(AES-256-GCM ciphertext)",
  "signature": "BASE64(Ed25519 signature)"
}
```

- **encrypted_payload** — AES-256-GCM ciphertext of the actual endpoint list. The AES key is embedded in each InHive release and rotated every release.
- **signature** — Ed25519 signature over `encrypted_payload || key_version || updated_at`. Verified on-device before decryption.
- **key_version** — string identifier (e.g. `v1`, `v2`) used to select the correct decryption key from the client's embedded key map. Clients carry the last few key versions for graceful rollout.

## Access points (in client discovery order)

1. Local cache (last successful fetch on the device)
2. `https://inhive.ru/api/bootstrap.json` (primary)
3. `https://raw.githubusercontent.com/TwilgateLabs/inhive-public-config/main/bootstrap.json` (this repo)
4. `https://cdn.jsdelivr.net/gh/TwilgateLabs/inhive-public-config@main/bootstrap.json` (CDN edge cache of this repo)
5. Embedded fallback baked into the binary at build time

## Sync

The `bootstrap.json` file in this repo is updated automatically via an hourly cron on the InHive Latvia server. Manual edits in this repo will be overwritten — change the source on the server instead.

## Security note

Decrypting the payload yields connection credentials, but those credentials only work through an `nginx` forward-proxy on the server with a 2 MB transfer cap, GET-only methods and a 100 KB/s rate limit. The endpoint list is **not** a usable full VPN — it exists solely to bootstrap a real subscription fetch in network-restricted environments.

## License

Encrypted content has no separately licensable form. The repository structure and README are released under MIT.
