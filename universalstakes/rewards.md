# Rewards

> **Important:** A reward command is run by the server console. Only trusted staff should be allowed to edit stake files.

Each reward block's configured commands have a `12,000`-byte encoded budget. Keep every command on one line; newline, carriage-return, and the reserved U+0001 payload separator are rejected. The complete deferred reward payload is limited to `60,000` UTF-8 bytes, and each serialized custom item to `32 KiB`. Split large command lists or simplify item metadata when validation reports a limit.

## Rank rewards

Under `prizes.top`, each key is a finishing position. The `material`, `name`, and `lore` describe the reward in the menu. The `rewards` list gives the actual prize.

```yaml
prizes:
  top:
    1:
      material: NETHER_STAR
      name: "<gold>Winner"
      lore: ["<gray>64 diamonds"]
      rewards:
        - "give %player% diamond 64"
        - "broadcast <gold>%player% won the race!"
```

`%player%` and `%player_name%` become the winner’s name.

## Consolation rewards

`prizes.others.near-top` gives rewards to players immediately below the ranked winners.

- `type: COUNT` selects a fixed number of players.
- `type: PERCENT` selects a percentage of all participants.
- `refund-percent` returns part of each selected player’s own deposit. It always rounds down.

```yaml
others:
  near-top:
    type: COUNT
    amount: 5
    refund-percent: 50
    rewards:
      - "give %player% emerald 3"
```

## How players receive rewards

When a round ends, rewards wait in `/stakes rewards`. A player may claim one reward or use **Claim All**. This keeps offline winners from losing rewards.

> **SCREENSHOT TO ADD:** Rewards menu with one reward card and the Claim All button.
