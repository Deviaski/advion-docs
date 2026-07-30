---
order: 9
icon: lock
---

# Commands & permissions

By default **nobody** has UniversalStakes permissions. Give regular players the bundle
**`universalstakes.use`**, and give staff the bundle **`universalstakes.admin`** (operators get it
automatically). You can also grant only the exact nodes listed below.

---

## Player commands

| Command | What it does |
| --- | --- |
| `/stakes` | Open the main menu. `/stake` and `/universalstakes` also work. |
| `/stakes <stake-id>` | Open one event. |
| `/stakes history` | Open your deposit history. |
| `/stakes rewards` | Open your waiting rewards. |
| `/stakes help` | Show player command help. |
| `/<alias>` | Open an event through its alias (e.g. `/diamonds`). |
| `/<alias> deposit` | Open contribution sources; left-click deposits, right-click withdraws (if enabled). |
| `/<alias> top [page]` | Show the leaderboard in chat. |
| `/<alias> help` | Show the commands for this event. |

The first value in `config.yml` under `command.aliases` is the **primary** root command; the rest are
aliases of the same full tree. They are case-insensitive and must be unique. Changing roots or event
aliases needs a **restart**.

---

## Staff commands

| Command | What it does |
| --- | --- |
| `/stakes admin reload` | Validate and reload language, menus, stake display/messages, and **new** `currencies.yml` entries. See [Reload & restart](Reload-and-Restart.md). |
| `/stakes admin start <stake-id>` | Start a new round. |
| `/stakes admin stop <stake-id>` | End the active round. |
| `/stakes admin importitem <item-id>` | Save the item in your hand as a reusable custom-item ID. |
| `/stakes admin logs <stake-id>` | Open completed rounds and their immutable audit logs. |
| `/stakes admin help` | Show administrator command help. |

Invalid syntax shows the editable `messages.commandUsage` message. A mistyped subcommand shows the
matching help page.

---

## Complete permission list

| Permission | Default | Grants |
| --- | --- | --- |
| `universalstakes.use` | false | All standard player command and menu nodes below. Does **not** bypass a stake's own `permission`. |
| `universalstakes.admin` | op | All admin command and audit-menu nodes below. |
| `universalstakes.command.main` | false | `/stakes` and configured root aliases. |
| `universalstakes.command.history` | false | `/stakes history`. |
| `universalstakes.command.rewards` | false | `/stakes rewards`. |
| `universalstakes.command.stake` | false | `/stakes <stake-id>`. |
| `universalstakes.command.help` | false | `/stakes help`. |
| `universalstakes.command.alias.open` | false | `/<stake-alias>`. |
| `universalstakes.command.alias.top` | false | `/<stake-alias> top [page]`. |
| `universalstakes.command.alias.help` | false | `/<stake-alias> help`. |
| `universalstakes.menu.main` | false | Main menu. |
| `universalstakes.menu.stake` | false | Selected event menu. |
| `universalstakes.menu.history` | false | History menu. |
| `universalstakes.menu.rewards` | false | Rewards menu. |
| `universalstakes.menu.admin.logs` | false | Completed-round audit menu. |
| `universalstakes.menu.admin.log-events` | false | Per-event audit menu. |
| `universalstakes.command.admin.help` | false | `/stakes admin help`. |
| `universalstakes.command.admin.reload` | false | `/stakes admin reload`. |
| `universalstakes.command.admin.start` | false | `/stakes admin start <stake-id>`. |
| `universalstakes.command.admin.stop` | false | `/stakes admin stop <stake-id>`. |
| `universalstakes.command.admin.importitem` | false | `/stakes admin importitem <item-id>`. |
| `universalstakes.command.admin.logs` | false | `/stakes admin logs <stake-id>`. |

---

## Per-stake access

Access to a specific event is set **in the stake file**, not by a generated permission node. Add
`permission` at the root only when the event must be restricted:

```yaml
permission: "stake.diamondrush"
```

- Missing or empty (`permission: ""`) → the event is **public** to anyone with the command/menu nodes.
- Non-empty → only players with that **exact** permission can see or join it.

Grant that node yourself in your permissions plugin. `universalstakes.use` does **not** grant it
automatically.