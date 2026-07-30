---
order: 12
icon: shield
---

# Database & safety

> **Before updating the plugin:** stop the server and **back up the database**. This one habit
> prevents most data-loss problems.

---

## SQLite — the default

SQLite needs no separate database server. Its file is `plugins/UniversalStakes/data/data.db`. Use it
for a normal single Minecraft server.

- Do **not** open or edit `data.db` while the server is running.
- The relevant `config.yml` key is `database.sqliteFile` (default `data/data.db`); MySQL and pool
  settings are ignored.

---

## MySQL — external storage

Use MySQL when you already have a database server or need the data outside the game server.

```yaml
database:
  type: mysql
  host: localhost
  port: 3306
  name: universalstakes
  user: universalstakes
  password: "your-password"
  poolSize: 10
  sslMode: VERIFY_IDENTITY
  trustCertificateKeyStoreUrl: "file:/absolute/path/to/mysql-client-truststore.p12"
  trustCertificateKeyStorePassword: "change-me"
  allowPublicKeyRetrieval: false
```

> MySQL does **not** enable a multi-server network. **Exactly one** active UniversalStakes instance
> may use a database. Do not point multiple Paper servers at the same schema — lifecycle and financial
> coordination are process-local.

### Secure MySQL setup

- `VERIFY_IDENTITY` is the secure default: it encrypts the connection, verifies the certificate chain,
  and checks the certificate matches `host`. Put the database CA in a dedicated PKCS#12/JKS truststore
  and use its absolute `file:` URL.
- Do **not** set `allowPublicKeyRetrieval: true` as a substitute for TLS. `REQUIRED` and `VERIFY_CA`
  weaken hostname verification; `PREFERRED`/`DISABLED` allow plaintext and must not be used for a
  remote database.
- Create a **dedicated least-privilege** database user instead of `root`. The plugin creates its own
  tables and indexes. Restrict the account and network ACL to the Minecraft host — pending reward
  records can contain commands that later run as console.
- The database password and truststore password are plain text in `config.yml`. Restrict file access
  to the server account and keep this file out of shared backups/logs.

Changing database settings requires a **restart** (the pool is built only at startup).

---

## Keep the server safe

- Test reward commands on a test server before enabling a stake for real players.
- Never give players or untrusted moderators write access to plugin files.
- Back up the database before changing database type or editing many stakes.
- Do not deliberately stop the server while rewards are being claimed.
- Keep every `currencies.yml` ID and its give/take commands **unchanged** while a stake using it has
  unclaimed currency refunds. A pending refund stores the currency ID, not a historical copy of its
  command template — changing/removing it and restarting can send an old refund through the new
  command or make it impossible to claim. Wait until those rewards are resolved.

---

## Automatic outcomes (no reconciliation command needed)

The plugin never requires a manual reconciliation command. Every completed round can be inspected with
`/stakes admin logs <stake-id>`; the menu shows **immutable** events — player, amount, currency,
operation ID, result, and any error. Only rounds with events recorded after this feature was installed
appear; historical delivery outcomes are not reconstructed.

| Outcome | When it happens | What to do |
| --- | --- | --- |
| `PENDING` | A reward is waiting to be claimed, or a confirmed temporary error put it back. | Player clicks again in `/stakes rewards`. |
| `BLOCKED` | A reward command was ambiguous (might have partly worked), so it is not replayed automatically. | Review `/stakes admin logs <stake>` and resolve manually. |
| `BLOCKED` (from crash) | A `CLAIMING` lease older than 5 minutes during a crash. Moved to `BLOCKED` on startup / every 60 s. | Same — review and resolve manually; it cannot be claimed again. |
| `UNCONFIRMED` (deposit) | An external currency withdrawal during a deposit could not be confirmed. | The deposit is cancelled **without** stake credit. Not retried. Check the provider. |
| `PAYOUT_FAILED` (withdrawal) | A right-click withdrawal reduced the ledger but the payout to the player failed. | Compensate the player manually. The contribution was already removed — do **not** tell them to repeat the withdrawal. |

`auditLog.retentionDays: 0` keeps audit events forever. A positive value removes only expired audit
events; it never deletes financial or round records.

---

## Safety limits (plugin-wide)

These bounds prevent integer/precision overflow and unbounded database payloads. Validation rejects
values that exceed them — reduce amounts, command lists, or item metadata instead of bypassing them,
and never edit oversized values directly into the database.

| Limit | Value |
| --- | --- |
| Any financial amount (investment, withdrawal, total, refund input) | at most `9,007,199,254,740,991` |
| A pending reward's serialized payload | at most `60,000` UTF-8 bytes |
| Encoded reward-command budget per reward block | `12,000` bytes (one line each, no reserved U+0001 char) |
| An imported custom-item snapshot | at most `32 KiB` binary |