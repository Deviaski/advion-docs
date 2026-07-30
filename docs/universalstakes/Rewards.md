---
order: 6
icon: gift
---

# Rewards

> **Important:** every reward command runs as the **server console**. Only trusted staff should edit
> stake files — a reward command can give items, money, or run any server command.

When a round finalizes, UniversalStakes ranks every participant and creates **pending rewards**.
Players claim them in `/stakes rewards`. This page covers how to configure those rewards.

---

## `need-free-slots` — protect the player's inventory

Add `need-free-slots` next to any `rewards` list that gives items. It is the number of **completely
empty** inventory slots a player must have before **any** command in that block runs.

```yaml
prizes:
  top:
    1:
      need-free-slots: 2
      rewards:
        - "give %player% diamond_block 64"
        - "give %player% enchanted_golden_apple 1"
        - "broadcast <gold>%player% won the race!"
```

- If the player has too few empty slots, the commands do **not** run and the reward stays pending for a
  later click.
- `0` disables the check.
- Count each command that can give an item (not stacks or amounts); commands like `broadcast` and
  `msg` don't count. The plugin does not parse command text, so this number is **your** promise that
  the configured commands fit.

---

## `prizes.top` — ranked winners

Each key is a finishing position. Keys must be contiguous positive integers starting at `1`.

```yaml
prizes:
  top:
    1:
      material: NETHER_STAR
      name: "<gold>Winner"
      lore: ["<gray>64 diamonds"]
      need-free-slots: 1
      rewards:
        - "give %player% diamond 64"
        - "broadcast <gold>%player% won the race!"
    2:
      material: DIAMOND
      name: "<aqua>Second place"
      need-free-slots: 1
      rewards:
        - "give %player% diamond 32"
```

| Field | Meaning |
| --- | --- |
| `material`, `name`, `lore` | How the prize looks in the menu. |
| `need-free-slots` | Empty slots required before commands run (see above). |
| `rewards` | Console commands run when the player claims. `%player%` and `%player_name%` become the winner's name. |

The prize `lore`/`name` can use `{rank}`, `{player_name}`, `{points}`, plus per-source tokens like
`{coins.display-name}` and `{coins.amount}`.

---

## `prizes.others` — consolation tiers

A numbered list of consolation tiers for players who did not win a top place.

- Tier `1` starts right after the ranked winners.
- Tier `2` starts after every player picked by tier `1`, and so on.
- A player receives **at most one** tier.
- Tier keys must be contiguous positive integers starting at `1`.

```yaml
prizes:
  others:
    1:
      type: COUNT
      amount: 100
      refund-percent: 50
      need-free-slots: 1
      rewards:
        - "give %player% emerald 10"
    2:
      type: PERCENT
      amount: 100
      refund-percent: 25
      need-free-slots: 2
      rewards:
        - "give %player% diamond 3"
```

| Field | Meaning |
| --- | --- |
| `type` | `COUNT` = a fixed number of the next players. `PERCENT` = a percentage of all participants. |
| `amount` | Player count (for `COUNT`), or a `0`–`100` percentage (for `PERCENT`). `100` gives the tier to everyone still unselected. |
| `refund-percent` | `0`–`100`. Returns that percentage of each selected player's **net deposit**, per source, rounded down. Item, Vault, PlaceholderAPI and custom-item deposits are each returned in their original form. |
| `need-free-slots` | Empty slots required before commands run. |
| `rewards` | Console commands run on claim. |

> The old single-tier key `prizes.others.near-top` is still read as tier `1`, but new files should use
> numbered tiers. Changing tier selection or reward delivery requires a **restart**.

---

## How players receive rewards

1. The round ends and finalizes. Pending rewards are created.
2. The player gets a chat message pointing to `/stakes rewards`.
3. They open the menu and either click one reward or use **Claim All**.
4. Commands run as console. If `need-free-slots` isn't met, nothing runs and the reward stays pending.

This keeps offline winners from losing prizes — rewards wait safely until claim.

### When delivery goes wrong

The plugin can confirm that a console command **returned success**, but it cannot prove an arbitrary
third-party command actually delivered its effect. So:

- A confirmed **temporary** error → the reward returns to `PENDING`; the player clicks again.
- An **ambiguous** failure (might have partly worked) → the reward becomes **`BLOCKED`** and is never
  replayed automatically (replaying could duplicate a prize). Review it in
  `/stakes admin logs <stake>` and resolve it manually.
- A server **crash during a claim** → a stuck `CLAIMING` lease older than 5 minutes moves to
  `BLOCKED` on startup and every 60 seconds. It stays visible to admins but cannot be claimed again.

See [Database & safety](Database-and-Safety.md) for the full list of automatic outcomes
(`BLOCKED`, `UNCONFIRMED`, `PAYOUT_FAILED`) and what to do about them.

---

## Size limits

Each reward block's commands have a **12,000-byte** encoded budget. Keep each command on one line;
newline, carriage-return and the reserved U+0001 separator are rejected. The full pending-reward
payload is limited to **60,000 UTF-8 bytes**, and each imported custom item to **32 KiB**. If
validation reports a limit, split the command list or simplify item metadata rather than bypassing it.