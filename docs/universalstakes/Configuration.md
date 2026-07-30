---
order: 10
icon: settings
---

# Configuration

This page covers the two global files: `config.yml` (server-wide settings) and `currencies.yml`
(external PlaceholderAPI currencies). For the per-event files, see [Stake settings](Stake-Settings.md).

> `config.yml` uses **camelCase** keys (`sqliteFile`, `poolSize`). `currencies.yml` uses
> **kebab-case** (`raw-placeholder`, `give-command`). Both are intentional — copy the spelling exactly.

---

## `config.yml`

```yaml
debug: false
locale: "english"

database:
  type: sqlite
  host: localhost
  port: 3306
  name: universalstakes
  user: universalstakes
  password: ""
  sqliteFile: data/data.db
  poolSize: 10
  sslMode: VERIFY_IDENTITY
  trustCertificateKeyStoreUrl: ""
  trustCertificateKeyStorePassword: ""
  allowPublicKeyRetrieval: false

command:
  aliases:
    - universalstakes
    - stake
    - stakes

input:
  promptTimeoutSeconds: 120

banDisqualification:
  activeRounds: true
  endedRounds: false
  checkIntervalSeconds: 60

auditLog:
  retentionDays: 0
```

### Every field explained

| Field | Default | Meaning |
| --- | --- | --- |
| `debug` | `false` | Print extra diagnostics (incl. stack traces) to the console. |
| `locale` | `english` | The language pack loaded from `language/<locale>/`. Must match an installed folder. |
| `database.type` | `sqlite` | `sqlite` or `mysql`. An unknown value stops startup rather than silently using SQLite. |
| `database.host` | `localhost` | MySQL host (MySQL only). |
| `database.port` | `3306` | MySQL port (MySQL only). |
| `database.name` | `universalstakes` | MySQL database name (MySQL only). |
| `database.user` / `password` | — | MySQL credentials (MySQL only). |
| `database.sqliteFile` | `data/data.db` | SQLite file path relative to the plugin folder. |
| `database.poolSize` | `10` | HikariCP max pool size. |
| `database.sslMode` | `VERIFY_IDENTITY` | MySQL TLS mode. `VERIFY_IDENTITY` (recommended), `VERIFY_CA`, `REQUIRED`, `PREFERRED`, or `DISABLED`. |
| `database.trustCertificateKeyStoreUrl` | `""` | Optional truststore URL for MySQL TLS, e.g. `file:/path/to/truststore.p12`. |
| `database.trustCertificateKeyStorePassword` | `""` | Password for that truststore. |
| `database.allowPublicKeyRetrieval` | `false` | Allow RSA public-key retrieval. Keep `false` unless your setup requires it. |
| `command.aliases` | `universalstakes, stake, stakes` | Root commands. First is primary; the rest are aliases of the same tree. Restart required to change. |
| `input.promptTimeoutSeconds` | `120` | Seconds before a chat "type the amount" prompt expires. |
| `banDisqualification.activeRounds` | `true` | Remove banned players from active leaderboards (no refund). |
| `banDisqualification.endedRounds` | `false` | Remove banned players from ended rounds and delete their unclaimed rewards. |
| `banDisqualification.checkIntervalSeconds` | `60` | How often to scan participants for bans. |
| `auditLog.retentionDays` | `0` | Days to keep admin audit events. `0` = keep forever. |

> The database password and truststore password are stored as **plain text** in `config.yml`. Restrict
> filesystem access to the server account and keep this file out of shared backups/logs. See
> [Database & safety](Database-and-Safety.md).

### What reload applies

`/stakes admin reload` applies `locale`, `debug`, `input`, `banDisqualification`, and `auditLog`.
**Database settings and `command.aliases` require a restart** (the connection pool and commands are
built only at startup).

---

## `currencies.yml`

Only needed if you use `PLACEHOLDER` currencies. The generated file ships PlayerPoints and
ExcellentEconomy **templates** — verify them against your economy before use.

```yaml
currencies:
  gems:
    display-name: "Gems"
    symbol: ""
    raw-placeholder: "%coinsengine_balance_gems%"
    give-command: "ce give %player% gems %amount%"
    take-command: "ce take %player% gems %amount%"
    works-offline: false
    balance-check-after: true
    balance-check-delay-ticks: 5
```

| Field | Meaning |
| --- | --- |
| `display-name` | Required name used for validation. |
| `symbol` | Optional metadata (not auto-shown in current menus). |
| `raw-placeholder` | Must return **digits only** — no symbol, commas, decimals, or color codes. |
| `give-command` / `take-command` | Run as **console**; must contain `%player%` and `%amount%`. Leading `/` optional. |
| `works-offline` | May the provider operate on an offline player? |
| `balance-check-after` | Re-read the balance after the command and require it changed by exactly the amount. |
| `balance-check-delay-ticks` | Ticks to wait before the post-command check (for async economies). |

### Reload rules for currencies

- **Adding** a new ID: applied by `/stakes admin reload`.
- **Editing or removing** an existing ID: requires a **restart** and is **unsafe** while a stake using
  it still has unclaimed refunds (pending refunds store the ID, not a copy of the old command).

For how PLACEHOLDER deposits are confirmed step by step, see [Currencies](Currencies.md).

---

## Configuration trust

Files under `plugins/UniversalStakes/` are part of your server's trusted admin surface. Some
configured actions run with **console authority**: menu `console:` actions, PlaceholderAPI currency
`give`/`take` commands, and reward commands. That is necessary for economy/item integrations, but it
means these files must never be writable by ordinary players, unreviewed automation, or web-panel
roles meant only for moderators. Treat config edits like server command edits — review before applying.