---
order: 4
icon: list-detailed
---

# Stake settings

> Use this page when you already have a stake file and want to understand one setting. For a
> copy-paste starter, use [Your first stake](First-Stake.md).

The **stake ID** is the file name without `.yml`. Use simple lowercase names such as
`weekly-diamonds.yml`. Stake files use **kebab-case** keys (`withdrawals-enabled`, `min-invest`).

This page covers each block top to bottom. The currency types themselves are explained in depth in
[Currencies](Currencies.md), and prizes in [Rewards](Rewards.md).

---

## Root settings

| Setting | Type | Default | Meaning |
| --- | --- | --- | --- |
| `alias` | text | `""` | Short command for this event, e.g. `diamonds` enables `/diamonds`. Empty = no alias. Unique; restart required to add/change. |
| `permission` | text | `""` | Permission node required to see and join. Empty/missing = public. Not auto-generated; grant it yourself. |
| `withdrawals-enabled` | true/false | `true` | Can players right-click to take resources back during an active round? Reload-safe. |

---

## `display` — the event card

This is how the event looks in the main menu list.

```yaml
display:
  display-name: "Diamond Race"
  name: "<aqua>Diamond Race"
  lore:
    - "<gray>Deposit diamonds and reach first place!"
  material: DIAMOND
  model-data: 0
```

| Field | Meaning |
| --- | --- |
| `display-name` | The human-readable name used in messages, history, rewards and placeholders. Blank falls back to `name`. |
| `name` | The item title shown on the card. Supports MiniMessage (`<aqua>`, `<gold>`…). |
| `lore` | The lines under the title. |
| `material` | A Minecraft material, e.g. `DIAMOND`, `PAPER`, `EMERALD`. |
| `model-data` | Custom model data for resource packs. Leave `0` if you don't use one. |

The card lore can use built-in tokens like `{status}`, `{time_left}`, and `{participants}` — these are
provided by the plugin itself and work without PlaceholderAPI.

---

## `messages` — start, stop, reminders

```yaml
messages:
  started:
    - "{prefix}<green>{stake} has been started."
  stopped:
    - "{prefix}<green>{stake} has ended. Winner: {winner-player-1}!"
  reminder:
    interval: "5m"
    message:
      - "{prefix}<aqua>{stake}</aqua> is active! Time left: <yellow>{time_left}</yellow>"
```

| Field | When it is sent |
| --- | --- |
| `started` | When a round starts. Broadcast to online players who can access the event. |
| `stopped` | When a round finalizes. Broadcast to online players who can access the event. |
| `reminder.message` | Every `reminder.interval` while the round is active, to online eligible players. |

Use an empty list (`[]`) for any of them to disable that message. The `reminder.interval` must be a
positive duration like `30s`, `5m`, or `1h`.

### Placeholders available in stake messages

`{stake}`, `{stake_raw}` (plain-text name), `{time_left}`, `{participants}`, `{total}` (total
points), `{alias}`. In `stopped` you also get `{winner-rank-1}`, `{winner-player-1}`,
`{winner-points-1}` … up to your highest configured top rank.

See [Messages & placeholders](Messages-and-Placeholders.md) for the full list and formatting.

---

## `contributions` — what players deposit

Every stake needs at least one entry. The map key is the **source ID** (1–64 chars, must start with a
lowercase letter or digit, then lowercase letters/digits/`_`/`-`; case-sensitive).

```yaml
contributions:
  diamonds:
    display-name: "Diamonds"
    currency:
      type: ITEM
      material: DIAMOND
    points: 5
    display:
      material: DIAMOND
      name: "<aqua>Diamonds"
      lore:
        - "<gray>Deposited: {deposited}"
    limits:
      min-invest: 1
      max-invest: 64
      daily-limit: 256
```

| Field | Meaning |
| --- | --- |
| `display-name` | Source name used in history and prize placeholders, available as `{<id>.display-name}`. |
| `currency` | What resource this is. See [Currencies](Currencies.md). |
| `points` | Positive whole number. Score = `amount × points`. |
| `display` | How the source looks in the deposit menu (`material`, `model-data`, `name`, `lore`). |
| `limits` | Per-player deposit limits (see below). |

The source `lore` can use `{source_id}`, `{display-name}`, `{deposited}`, `{points-per-unit}`,
`{min-invest}`, `{max-invest}`, and `{daily-limit}`.

### Limits

| Setting | Meaning |
| --- | --- |
| `min-invest` | Minimum size of one deposit. Must be positive and ≤ `max-invest`. |
| `max-invest` | Maximum size of one deposit. Must be positive and ≥ `min-invest`. |
| `daily-limit` | Max net deposit into this source per server day. `0` = unlimited. When set, must be ≥ `min-invest`. |

A **withdrawal** reduces the net amount counted toward the daily limit, so players can deposit again
after withdrawing.

> Currency, `points`, and limits can only be changed when the stake's round is fully `ENDED`. Stop the
> stake and wait for finalization first. See [Reload & restart](Reload-and-Restart.md).

---

## `time` — round timing

Durations use `d`/`h`/`m`/`s`, e.g. `30m`, `2h`, `1d`, or `1h30m`.

```yaml
time:
  duration: "24h"
  restart-delay: "1h"
  auto-restart: true
```

| Field | Meaning |
| --- | --- |
| `duration` | How long one round stays open. Must be greater than zero. |
| `restart-delay` | Cooldown after a round ends before the next starts. Must be greater than zero. |
| `auto-restart` | `true` = start the first round on startup and auto-start later rounds. `false` = a new stake stays stopped on startup and an ended stake stays stopped until staff run `/stakes admin start`. An active round is **always resumed** after a server restart, regardless of this setting. |

Changing `time.*` requires a full restart.

---

## `prizes` — winners and consolation

Two blocks: `top` (ranked places) and `others` (consolation tiers). Full details and examples are in
[Rewards](Rewards.md).

```yaml
prizes:
  top:
    1:
      material: NETHER_STAR
      name: "<gold>Winner"
      need-free-slots: 1
      rewards:
        - "give %player% diamond 64"
  others:
    1:
      type: COUNT
      amount: 10
      refund-percent: 50
      need-free-slots: 1
      rewards:
        - "give %player% emerald 10"
```

Rules to remember:

- `prizes.top` keys are finishing positions, contiguous starting at `1` (`1`, `2`, `3`…).
- `prizes.others` keys are consolation tiers, contiguous starting at `1`. A player gets **at most one**
  tier. Tier `1` starts right after the ranked winners; tier `2` starts after tier 1's players, etc.
- `refund-percent` is `0`–`100`, returned per source and rounded down.
- `type: COUNT` = a fixed number of next players; `type: PERCENT` = a percentage of all participants
  (`amount` then 0–100).
- Every reward command runs **as console**. `%player%` / `%player_name%` become the winner's name.

---

## Safety limits (plugin-wide)

These protect database arithmetic and reward payloads. Validation rejects values that exceed them —
shrink your config instead of bypassing them.

| Limit | Value |
| --- | --- |
| Any financial amount (deposit, withdrawal, total, refund) | at most `9,007,199,254,740,991` |
| One reward's serialized payload | at most `60,000` UTF-8 bytes |
| Encoded reward-command budget per reward block | `12,000` bytes (one line each, no reserved U+0001 char) |
| Imported custom-item snapshot | at most `32 KiB` binary |