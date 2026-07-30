# Create Your First Stake

> **Goal:** Create a one-hour diamond competition with one winner. This is the safest first configuration to test.

## 1. Create the file

Create `plugins/UniversalStakes/stakes/diamonds.yml` and paste this:

```yaml
display:
  name: "<aqua>Diamond Race"
  lore:
    - "<gray>Deposit diamonds and reach first place!"
  material: DIAMOND

alias: "diamonds"

messages:
  started: "{prefix}<green>{stake} has been started."
  stopped: "{prefix}<green>{stake} has been stopped and finalized."
  reminder:
    interval: "5m"
    message: "{prefix}<aqua>{stake}</aqua> is active! Time left: <yellow>{time_left}</yellow>"

# Optional. Omit this line (or set it to "") to make the stake public.
# permission: "stake.diamonds"

currency:
  type: ITEM
  material: DIAMOND

time:
  duration: "1h"
  restart-delay: "10m"
  auto-restart: true

limits:
  min-invest: 1
  max-invest: 64
  daily-limit: 256

prizes:
  top:
    1:
      material: DIAMOND_BLOCK
      name: "<gold>First place"
      lore:
        - "<gray>Reward: 16 diamond blocks"
      rewards:
        - "give %player% diamond_block 16"
```

## 2. Start and test it

Restart the server. Then, as a player with permission:

1. Run `/diamonds` to open the stake.
2. Run `/diamonds invest 5` to deposit five diamonds.
3. Run `/diamonds top` to see the ranking.
4. As an operator, run `/stakes admin stop diamonds` to end the test round.
5. Open `/stakes rewards` and claim the prize.

> **SCREENSHOT TO ADD:** The Diamond Race card in the main menu, then the opened stake menu.

## 3. Change it later

You may change the name, description, icon, limits, and rewards in the file. `/stakes admin reload` validates the file but preserves an existing stake's live object so an active round cannot change rules halfway through. Perform a full server restart to apply the edited stake or imported item. A **new or renamed alias** also needs a full restart.

Need a field explained? Read [Stake Settings Reference](Stake-Reference.md). Need more than one winner or refunds? Read [Rewards](Rewards.md).
