# Database and Safety

> **Before updating the plugin:** Stop the server and back up the database. This is the one habit that prevents most data-loss problems.

## SQLite: the default choice

SQLite needs no separate database server. Its file is `plugins/UniversalStakes/data.db`. Use it for a normal single Minecraft server.

Do not open or edit `data.db` while the server is running.

## MySQL: for external storage

Use MySQL when your server already has a database server or you need data outside the game server. Edit `config.yml`:

MySQL does **not** enable a multi-server network: exactly one active UniversalStakes plugin instance may use a database. Do not point multiple Paper servers at the same schema; the current lifecycle and financial coordination is process-local.

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

Configuration keys are case-sensitive and use the Java field names shown above. `sqliteFile`, `poolSize`, `input.promptTimeoutSeconds`, and the `banDisqualification` section are also camelCase. Only `sqlite` and `mysql` are accepted for `database.type`; an unknown value stops plugin startup instead of silently opening a SQLite database.

`VERIFY_IDENTITY` is the secure default for MySQL: it encrypts the connection, verifies the certificate chain, and checks that the certificate matches `host`. Put the database CA in a dedicated PKCS#12/JKS truststore and use its absolute `file:` URL. Do not set `allowPublicKeyRetrieval: true` as a substitute for TLS. Modes `REQUIRED` and `VERIFY_CA` weaken hostname verification; `PREFERRED`/`DISABLED` permit plaintext and must not be used for a remote database.

Create a dedicated least-privilege database user instead of using `root`. UniversalStakes creates its own tables and indexes. Restrict the database account and network ACL to the Minecraft host, because pending reward records can contain commands that later execute as console.

Both the database password and truststore password are stored as plain text in `config.yml`; restrict filesystem access to the server account and keep this file out of backups or logs shared with third parties.

For SQLite, the relevant key is `database.sqliteFile` (default `data.db`); MySQL and pool settings are ignored.

Other root configuration keys include:

- `input.promptTimeoutSeconds`: seconds before a chat investment prompt expires (default `120`).
- `banDisqualification.activeRounds`: remove banned players from active leaderboards (default `true`).
- `banDisqualification.endedRounds`: remove banned players from ended rounds and delete unclaimed rewards (default `false`).
- `banDisqualification.checkIntervalSeconds`: ban scan interval (default `60`).

## Keep the server safe

- Test reward commands on a test server before enabling a stake.
- Never give players or untrusted moderators write access to plugin files.
- Back up the database before changing database type or editing many stakes.
- Do not deliberately stop the server while rewards are being claimed.

If the server crashes during a claim, a `CLAIMING` lease older than five minutes moves to `BLOCKED` during startup or the 60-second sweep. It remains in the database but cannot be claimed again, preventing duplicate external commands or payments. Confirmed delivery errors return the reward to `PENDING` for another player click.

## Automatic outcomes and administrator audit

The plugin never requires a reconciliation command. Every completed round can be inspected with `/stakes admin logs <stake-id>`; the menu shows immutable events, including player, amount, currency, operation ID, result, and any error.
Only rounds with events recorded after this feature is installed appear in the menu; historical delivery outcomes are not reconstructed.

- A reward whose external result cannot be confirmed is `BLOCKED` and recorded as `UNCONFIRMED`; it is not replayed.
- An investment whose external withdrawal cannot be confirmed is automatically cancelled without stake credit and recorded as `UNCONFIRMED`.
- `audit-log.retention-days: 0` keeps audit entries indefinitely. A positive value removes only expired audit entries; it does not delete financial or round records.

## Safety limits

- Any financial amount (investment, withdrawal, total, or refund input) must be no greater than `9,007,199,254,740,991`.
- A pending reward's serialized payload is limited to `60,000` UTF-8 bytes.
- Configured reward commands have a `12,000`-byte encoded budget per reward block and must be single lines without the reserved U+0001 payload separator.
- An imported custom-item snapshot is limited to `32 KiB` of serialized binary data.

These bounds prevent integer/precision overflow and unbounded database payloads. Reduce amounts, command lists, or item metadata when validation rejects a value; never edit oversized values directly into the database.
