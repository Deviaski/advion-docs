---
order: 8
icon: comment
---

# Messages & placeholders

UniversalStakes sends two kinds of chat messages: **plugin-wide** messages (errors, prompts, help)
from `messages.yml`, and **per-stake** lifecycle messages (`started`, `stopped`, `reminder`) that live
in each stake file. This page covers both, plus the placeholders you can use in text.

---

## Text formatting (MiniMessage)

All text supports MiniMessage tags in angle brackets:

| Tag | Example | Result |
| --- | --- | --- |
| Color | `<red>Stop!` | Red text |
| Gold | `<gold>Winner` | Gold text |
| Gradient | `<gradient:#FFAA00:#FFD700>text` | Gold gradient |
| Bold | `<b>Bold</b>` | Bold text |
| Click | `<click:run_command:'/stakes'>Open</click>` | Clickable text |
| Hover | `<hover:show_text:'<gray>Hi'>Info</hover>` | Tooltip on hover |

Built-in tokens use **curly braces** (`{prefix}`, `{stake}`). Keep MiniMessage tags in angle brackets.

---

## Plugin-wide messages (`messages.yml`)

This file holds everything that is not specific to one event. It has three sections:

```yaml
messages:
  prefix: '<gray>[UniversalStakes]</gray> '
  noPermission: '{prefix}<red>You do not have permission to do that: {permission}.'
  investmentSuccess: '{prefix}You invested {amount}. Total: {total}.'
  rewardClaimed: '{prefix}Claimed {count} pending reward(s).'
  # …and many more

time:
  day: d
  hour: h
  minute: m
  second: s

help:
  stakes: [ ... ]
  admin: [ ... ]
  alias: [ ... ]
```

- `messages.prefix` is the `{prefix}` token used by most other messages.
- `time.*` are the unit suffixes shown in remaining-time strings like `1d2h`.
- `help.*` are the help pages for `/stakes help`, `/stakes admin help`, and `/<alias> help`.

Edit the text to match your server's style, then `/stakes admin reload`.

---

## Per-stake messages (in the stake file)

Each stake can use its own wording for start, end, and reminders:

```yaml
messages:
  started:
    - "{prefix}<green>{stake} has been started."
  stopped:
    - "{prefix}<green>{stake} ended! 1st: {winner-player-1} ({winner-points-1} pts)"
  reminder:
    interval: "5m"
    message:
      - "{prefix}<aqua>{stake}</aqua> is active! Time left: <yellow>{time_left}</yellow>"
```

Use `[]` to disable any of them. These are sent only to online players who can access the event.

### Available placeholders

| Token | Available in | Meaning |
| --- | --- | --- |
| `{prefix}` | all | The `messages.prefix` value. |
| `{stake}` | all | The event's display name (formatted). |
| `{stake_raw}` | all | The display name as plain text. |
| `{time_left}` | reminder, started | Remaining time, e.g. `12m30s`. |
| `{participants}` | all | Number of participants. |
| `{total}` | all | Total points invested in the round. |
| `{alias}` | all | The stake's command alias. |
| `{permission}` | noPermission | The missing permission node. |
| `{winner-rank-N}` | stopped | Rank number N. |
| `{winner-player-N}` | stopped | Player at rank N (`No player selected` if empty). |
| `{winner-points-N}` | stopped | Points for rank N. |

`N` goes from `1` up to your highest configured `prizes.top` rank.

---

## PlaceholderAPI placeholders

If PlaceholderAPI is installed, UniversalStakes registers an expansion automatically. You can then
use these in **any** plugin that supports PlaceholderAPI (scoreboards, chat, TAB, etc.) and in custom
menu/stake text:

```text
%universalstakes_<stake-id>_<key>%
```

Example: `%universalstakes_diamonds_top_name_1%` shows the first-place player in the `diamonds` event.

### Available keys

| Key | Returns |
| --- | --- |
| `display_name` | The event's display name. |
| `status` | `Active` or `Ended` (from `messages.yml`). |
| `time_left` | Remaining time string. |
| `total` | Total points in the round. |
| `participants` | Number of participants. |
| `player_invested` | The viewing player's points in the event. |
| `player_place` | The viewing player's rank, or `-` if not on the board. |
| `top_name_N` | Name of the player at rank N. |
| `top_amount_N` | Points of the player at rank N. |

> Despite the legacy `amount` name, `total`, `player_invested` and `top_amount_N` return **leaderboard
> points**, not raw units from one source.

The shipped event cards use built-in `{status}` and `{time_left}` tokens, so they work **without**
PlaceholderAPI. You only need PlaceholderAPI installed if you put `%universalstakes_…%` tokens into
your own custom text, or if you use `PLACEHOLDER` currencies.