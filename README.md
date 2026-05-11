# inhive-public-config

Public mirror of encrypted bootstrap configuration for the [InHive VPN](https://app.inhive.ru) client.

## Purpose

InHive clients fetch a small encrypted manifest at startup to discover a rotating set of fallback endpoints in case the primary infrastructure is reachable but DPI-throttled. The primary location is `https://inhive.ru/api/bootstrap.json` — this repo is the **public fallback** served via GitHub raw and jsDelivr CDN, both of which remain accessible in regions where the primary domain is blocked.

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

- **encrypted_payload** — AES-256-GCM ciphertext of the fallback endpoint list. The AES key is embedded in each InHive release and rotated on release cadence.
- **signature** — Ed25519 signature over `encrypted_payload || key_version || updated_at`. Verified on-device before decryption.
- **key_version** — selects the correct decryption key from the client's embedded key map.

## Discovery order

1. Local cache (last successful fetch)
2. `https://inhive.ru/api/bootstrap.json` (primary)
3. `https://raw.githubusercontent.com/TwilgateLabs/inhive-public-config/main/bootstrap.json` (this repo)
4. `https://cdn.jsdelivr.net/gh/TwilgateLabs/inhive-public-config@main/bootstrap.json` (CDN edge cache)
5. Embedded fallback baked into the binary at build time

## Sync

The `bootstrap.json` file is updated automatically by a cron on the InHive backend. Manual edits in this repo will be overwritten — change the source upstream.

## License

MIT (repo structure & README). The encrypted payload has no separately licensable form.
