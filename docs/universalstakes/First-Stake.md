---
order: 3
icon: rocket
---

# Your first stake

> **Goal:** a 1-hour diamond race with one winner. The safest first event to test.

You will create one file, restart, and test the full loop: deposit → check the top → end the round →
claim the prize.

---

## 1. Create the file

Create `plugins/UniversalStakes/language/english/stakes/diamonds.yml` and paste this:

```yaml
display:
  display-name: "Diamond Race"
  name: "<aqua>Diamond Race"
  lore:
    - "<gray>Deposit diamonds and reach first place!"
  material: DIAMOND

alias: "diamonds"
withdrawals-enabled: true

messages:
  started:
    - "{prefix}<green>{stake} has been started."
  stopped:
    - "{prefix}<green>{stake} has been stopped and finalized."
  reminder:
    interval: "5m"
    message:
      - "{prefix}<aqua>{stake}</aqua> is active! Time left: <yellow>{time_left}</yellow>"

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
        - "<gray>1 diamond = {points-per-unit} points"
    limits:
      min-invest: 1
      max-invest: 64
      daily-limit: 256

time:
  duration: "1h"
  restart-delay: "10m"
  auto-restart: true

prizes:
  top:
    1:
      material: DIAMOND_BLOCK
      name: "<gold>First place"
      lore:
        - "<gray>Reward: 16 diamond blocks"
      need-free-slots: 1
      rewards:
        - "give %player% diamond_block 16"
```

> Want to restrict who can join? Add `permission: "stake.diamonds"` at the top and grant that node in
> your permissions plugin. Leave it out for a public event.

---

## 2. Start and test it

**Restart the server.** Then, as a player who has `universalstakes.use`:

1. Run `/diamonds` to open the event.
2. Run `/diamonds deposit`, click **Diamonds**, then **left-click** and type `5` in chat.
   You just deposited 5 diamonds → 25 points.
3. Run `/diamonds top` to see the leaderboard.
4. Because `withdrawals-enabled: true`, **right-click** Diamonds and type `2`. Two diamonds come
   straight back to your inventory and 10 points are removed.
5. As an operator, end the test early with `/stakes admin stop diamonds`.
6. Open `/stakes rewards` and claim the prize. Make sure you have 1 free inventory slot
   (`need-free-slots: 1`) or the reward stays pending.

---

## 3. Change it later

Some edits apply with `/stakes admin reload`, others need a full restart. The short version:

- **Reload applies:** display, messages, reminders, `withdrawals-enabled`, menu text, and adding a
  brand-new stake file.
- **Restart required:** the `alias`, timing, reward commands, consolation tiers, `permission`, and
  changing the currency/points/limits of a stake whose round is not fully ended.

See [Reload & restart](Reload-and-Restart.md) for the exact rules before editing a live event.

---

## Where to go next

- Every field explained: [Stake settings](Stake-Settings.md).
- More than one winner, refunds, consolation prizes: [Rewards](Rewards.md).
- Use money or custom items instead of diamonds: [Currencies](Currencies.md).