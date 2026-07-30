# Commands and Permissions

By default, players have **no** UniversalStakes permissions. Give regular players
`universalstakes.use`; give staff `universalstakes.admin` (operators receive it by default).
Both are convenience bundles. You can instead grant only the exact command and menu nodes below.

## Player commands

| Command | Purpose |
| --- | --- |
| `/stakes` | Open the main menu. `/stake` and `/universalstakes` also work. |
| `/stakes <stake-id>` | Open one stake. |
| `/stakes history` | Open personal deposit history. |
| `/stakes rewards` | Open waiting rewards. |
| `/<alias>` | Open a stake through its alias. |
| `/<alias> invest <amount>` | Deposit currency. |
| `/<alias> withdraw <amount>` | Withdraw while the round is active. |
| `/<alias> top [page]` | Show the leaderboard in chat. |

The first value in `config.yml` under `command.aliases` is the primary root
command; the remaining values are aliases for the same full command tree.
They are case-insensitive, normalized to lowercase, and must be unique. A
restart is required after changing root or stake aliases.

## Staff commands

| Command | Purpose |
| --- | --- |
| `/stakes admin reload` | Reload global configuration, language, and menus, and validate stake files. Existing stake objects are kept until restart. |
| `/stakes admin start <stake-id>` | Start a new round. |
| `/stakes admin stop <stake-id>` | End the active round. |
| `/stakes admin importitem <stake-id> <item-id>` | Save the item in the main hand as custom stake currency. |
| `/stakes admin logs <stake-id>` | Open completed rounds for a stake and view their immutable administrator audit logs. |

## Complete permission list

| Permission | Default | Grants |
| --- | --- | --- |
| `universalstakes.use` | false | All standard player command and menu permissions listed below. It does not bypass a stake-specific `permission` setting. |
| `universalstakes.admin` | op | All administrator command and audit-menu permissions listed below. |
| `universalstakes.command.main` | false | `/stakes` and configured root aliases. |
| `universalstakes.command.history` | false | `/stakes history`. |
| `universalstakes.command.rewards` | false | `/stakes rewards`. |
| `universalstakes.command.stake` | false | `/stakes <stake-id>`. |
| `universalstakes.command.help` | false | `/stakes help`. |
| `universalstakes.command.alias.open` | false | `/<stake-alias>`. |
| `universalstakes.command.alias.top` | false | `/<stake-alias> top [page]`. |
| `universalstakes.command.alias.invest` | false | `/<stake-alias> invest <amount>`. |
| `universalstakes.command.alias.withdraw` | false | `/<stake-alias> withdraw <amount>`. |
| `universalstakes.command.alias.help` | false | `/<stake-alias> help`. |
| `universalstakes.menu.main` | false | Main stakes menu (`stakes-main.yml`). |
| `universalstakes.menu.stake` | false | Selected stake menu (`stakes-menu.yml`). |
| `universalstakes.menu.history` | false | Deposit-history menu (`stakes-history.yml`). |
| `universalstakes.menu.rewards` | false | Pending-rewards menu (`stakes-rewards.yml`). |
| `universalstakes.menu.admin.logs` | false | Completed-round audit menu (`stakes-admin-logs.yml`). |
| `universalstakes.menu.admin.log-events` | false | Individual audit-event menu (`stakes-admin-log-events.yml`). |
| `universalstakes.command.admin.help` | false | `/stakes admin help`. |
| `universalstakes.command.admin.reload` | false | `/stakes admin reload`. |
| `universalstakes.command.admin.start` | false | `/stakes admin start <stake-id>`. |
| `universalstakes.command.admin.stop` | false | `/stakes admin stop <stake-id>`. |
| `universalstakes.command.admin.importitem` | false | `/stakes admin importitem <stake-id> <item-id>`. |
| `universalstakes.command.admin.logs` | false | `/stakes admin logs <stake-id>`. |

## Per-stake access

Stake access is configured in the individual `stakes/<stake-id>.yml` file, not by a generated
`universalstakes.stake.<stake-id>` node. Add `permission` at the root of the stake file only when
the stake must be restricted:

```yaml
permission: "stake.diamondrush"
```

If `permission` is missing or is empty (`permission: ""`), the stake is public. If it contains a
non-empty value, only players with that exact permission can see or participate in the stake.
Grant that configured node separately in your permissions plugin; `universalstakes.use` does not
automatically grant it.

Changing an existing stake file or running `importitem` writes/validates the files but does not replace that stake's live runtime object. Perform a full server restart before testing or enabling the new settings. Alias changes also require a restart.
