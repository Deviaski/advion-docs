---
order: 1
icon: information
---

# How it works

This page explains the whole plugin in plain language: where the files live, what a round is, and
what happens step by step when players deposit, when the timer ends, and when they claim rewards.
You do not need to read the source code — everything here is what matters for running the plugin.

---

## The file layout

After the first start, your `plugins/UniversalStakes/` folder looks like this:

```text
plugins/UniversalStakes/
├── config.yml              ← global settings (database, commands, language, safety)
├── currencies.yml          ← registry for external PlaceholderAPI currencies (optional)
├── data/
│   └── data.db             ← the SQLite database (default storage)
├── imported-items/         ← snapshots of custom items (made by /stakes admin importitem)
└── language/
    └── english/            ← the active language pack (one folder per language)
        ├── messages.yml    ← chat messages sent by the plugin
        ├── menus/
        │   ├── stakes-main.yml
        │   ├── stakes-menu.yml
        │   ├── deposit-menu.yml
        │   ├── stakes-history.yml
        │   ├── stakes-rewards.yml
        │   ├── stakes-admin-logs.yml
        │   └── stakes-admin-log-events.yml
        └── stakes/
            ├── coinfever.yml       ← one file per event
            └── diamondrush.yml
```

### What each file does

| File / folder | What it controls |
| --- | --- |
| `config.yml` | Database type, command names, language choice, ban handling, audit retention, debug. See [Configuration](Configuration.md). |
| `currencies.yml` | Definitions for `PLACEHOLDER` currencies (economies without a Vault bridge). Only needed if you use that type. See [Currencies](Currencies.md). |
| `data/data.db` | The SQLite database. Do not open or edit it while the server runs. |
| `imported-items/*.item` | Custom-item snapshots used by `CUSTOM_ITEM` contribution sources. |
| `language/<locale>/messages.yml` | All plugin chat messages. See [Messages & placeholders](Messages-and-Placeholders.md). |
| `language/<locale>/menus/*.yml` | The inventory menus players see. See [Menus](Menus.md). |
| `language/<locale>/stakes/*.yml` | One file per event. This is where you spend most of your time. See [Stake settings](Stake-Settings.md). |

> The plugin ships **19 language packs**. On the very first start it extracts them all into
> `language/`. You can delete the ones you do not use — they are never recreated automatically. Pick
> one with `locale:` in `config.yml` (for example `english`).

### Two naming styles to remember

- **`config.yml`** uses `camelCase` keys: `sqliteFile`, `poolSize`, `promptTimeoutSeconds`.
- **Stake files and `currencies.yml`** use `kebab-case` keys: `withdrawals-enabled`, `raw-placeholder`.

Both are normal — the plugin reads them that way on purpose. Just copy the spelling from this wiki or
from the example files.

---

## What a "stake" is made of

Every event is one `.yml` file with the same building blocks:

```text
stake file
├── display        how the event looks in the main menu (icon, name, lore)
├── alias          the short command, e.g. /diamonds
├── permission     optional: who can see and join this event
├── withdrawals    can players take resources back during a round?
├── messages       what is broadcast when the round starts, runs, and ends
├── contributions  the resources players can deposit (currency + points + limits)
├── time           round length, cooldown, auto-restart
└── prizes         top-place prizes + consolation tiers + refunds
```

You only need to fill in the blocks you care about. The full field-by-field reference is in
[Stake settings](Stake-Settings.md).

---

## How points work

Points are the leaderboard score. Each contribution source has a `points` value. When a player
deposits an amount, they earn **`amount × points`**.

```text
Example: a "diamonds" source with points: 5
  Player deposits 10 diamonds  →  10 × 5 = 50 points
  Player deposits 3 more       →  total 13 diamonds = 65 points
```

If a stake has several sources, the points from each are simply added together. **Sources never
convert into each other** — 10 diamonds and 20 carrots stay separate, they just both add to the same
score. `points` must be a positive whole number (no decimals).

---

## The round lifecycle (events)

A **round** is one timed run of a stake. Behind the scenes each round moves through a few states.
You do not control these states directly, but knowing them helps you understand reload rules and
error messages.

```text
              (auto-restart or /stakes admin start)
                            │
                            ▼
        ┌──── ACTIVE ───────────────────────┐
        │  Players deposit and withdraw.    │
        │  Reminders broadcast on interval. │
        │  Timer counts down.               │
        └──────────────┬───────────────────┘
                       │ timer ends  OR  /stakes admin stop
                       ▼
                  CLOSING  ← wait for any in-flight deposits to settle
                       │
                       ▼
                FINALIZING  ← calculate results, store pending rewards
                       │
                       ▼
                   ENDED  ← broadcast winners, prizes become claimable
                       │
                       │  (if auto-restart: true)
                       │  wait restart-delay, then a new ACTIVE round starts
                       ▼
                   next round…
```

### What happens at each step

- **Round starts** → the plugin records a `ROUND_STARTED` audit event and broadcasts
  `messages.started` to every online player who can access the event.
- **While active** → players deposit/withdraw. Every `reminder.interval` (for example `5m`), the
  `reminder.message` is broadcast to online players who can access the event.
- **Round ends** (timer or admin stop) → the round moves through `CLOSING` → `FINALIZING` → `ENDED`.
  Finalizing calculates the ranking, creates the **pending rewards**, and then broadcasts
  `messages.stopped` (with the winner placeholders) to eligible online players.
- **After it ends** → if `auto-restart: true`, the plugin waits `restart-delay` and starts a new
  round automatically. If the server was offline when that delay elapsed, the new round starts on the
  next startup. A round that was **active when the server stopped is resumed** on restart, so deposits
  are never silently lost.

### The one error state

If the database ever contains a round status the plugin does not recognize, that round goes into an
**`ERROR`** state and fails closed: **no deposits are accepted** until an administrator fixes the
database row. This is deliberate — it never silently reopens a broken round. You will see a clear
console error naming the stake and round.

---

## What happens when a player deposits

1. The player opens the deposit menu and left-clicks a contribution source.
2. The plugin asks in chat: *type the amount to invest, or cancel*.
3. The player types a whole number.
4. The plugin checks: is the round active? Is the amount inside `min-invest`, `max-invest`, and the
   `daily-limit`? Does the player actually have that much of the resource?
5. If everything is fine, the resource is taken from the player and the points are added to the
   leaderboard.
6. For external currencies (PlaceholderAPI), the plugin runs your configured `take-command` and then
   **checks the balance changed by exactly that amount**. If it cannot confirm the change, the
   deposit is cancelled **without** awarding points and the audit log records `UNCONFIRMED`.

A **right-click** starts a withdrawal (only if `withdrawals-enabled: true` and the round is active).
The resource is returned to the player **immediately and directly** — it never goes through
`/stakes rewards`. The matching points are removed from the board.

---

## What happens when a round ends and rewards are claimed

1. On finalize, the plugin ranks every participant by points.
2. Top-place prizes (`prizes.top`) are created as **pending rewards** for the ranked players.
3. Consolation tiers (`prizes.others`) are handed to the next players, plus any configured
   `refund-percent` of their deposit.
4. Each affected player gets a chat message pointing them to `/stakes rewards`.
5. The player opens `/stakes rewards`, clicks a reward (or **Claim All**), and the configured reward
   commands run **as the server console**.

> Reward commands run with **console authority**. That is why only trusted staff should edit stake
> files. See [Database & safety](Database-and-Safety.md).

### What if delivery fails?

- A **temporary** error returns the reward to `PENDING`, so the player can just click again.
- An **ambiguous** failure (the command might have partly worked) marks the reward **`BLOCKED`** so it
  is never replayed automatically — replaying could give the prize twice. An admin reviews it in
  `/stakes admin logs <stake>` and resolves it manually.
- If the server **crashes mid-claim**, a stuck `CLAIMING` lease older than 5 minutes is moved to
  `BLOCKED` on startup (and every 60 seconds). It stays visible to admins but cannot be claimed again.

---

## Where to go next

- Make your first event now: [Your first stake](First-Stake.md).
- Understand every field: [Stake settings](Stake-Settings.md).
- Set up prizes and refunds: [Rewards](Rewards.md).