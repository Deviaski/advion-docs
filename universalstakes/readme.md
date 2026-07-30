# UniversalStakes Wiki

UniversalStakes turns surplus items and money on your server into a **timed leaderboard race**.
Players choose an event, deposit resources, earn points, and climb the board. When the timer ends,
the top players win the prizes you configured. Everyone else can get a consolation reward or a refund.

You do **not** need any coding experience to run this plugin. If you can edit a `.yml` text file, you
can run UniversalStakes.

---

## Start here

| I want to… | Read this page |
| --- | --- |
| Understand what the plugin does before touching anything | [How it works](How-It-Works.md) |
| Install the plugin for the first time | [Installation](Installation.md) |
| Make a working event in 5 minutes | [Your first stake](First-Stake.md) |
| Understand every setting in a stake file | [Stake settings](Stake-Settings.md) |
| Choose what players deposit (items, money, custom items) | [Currencies](Currencies.md) |
| Set up prizes, refunds and consolation rewards | [Rewards](Rewards.md) |
| Change the in-game menus players see | [Menus](Menus.md) |
| Change chat messages and placeholders | [Messages & placeholders](Messages-and-Placeholders.md) |
| Give players and staff access | [Commands & permissions](Commands-and-Permissions.md) |
| Understand `config.yml` and `currencies.yml` | [Configuration](Configuration.md) |
| Know when to reload and when to restart | [Reload & restart](Reload-and-Restart.md) |
| Choose SQLite or MySQL and keep data safe | [Database & safety](Database-and-Safety.md) |
| Fix a problem | [Troubleshooting](Troubleshooting.md) |

---

## The big picture (30-second version)

1. You create an **event** by dropping a small `.yml` file into the plugin folder.
2. The event runs for a time you choose (for example 1 hour). This one run is called a **round**.
3. Players open the menu, pick a resource, and **deposit** it. Each deposited unit is worth a number
   of **points** you set. Points decide the leaderboard.
4. When the timer ends, the round **finalizes**: the top players get their prizes, and players near
   the top can get a consolation reward or a percentage of their deposit back.
5. Prizes wait safely in `/stakes rewards` until the player claims them — even if they were offline.

For the full version with the round lifecycle, the file layout, and what happens behind the scenes,
read [How it works](How-It-Works.md).

---

## Words you will see a lot

These terms appear everywhere in this wiki. Skim them once and the rest makes sense.

| Term | Meaning |
| --- | --- |
| **Stake** | One event/competition, for example *Diamond Rush*. |
| **Stake ID** | The file name without `.yml`. `diamonds.yml` → stake ID `diamonds`. |
| **Round** | One timed run of a stake. A stake can run many rounds over time. |
| **Alias** | A short command for a stake, for example `/diamonds`. |
| **Contribution source** | One resource a stake accepts, with its own currency, point value and limits. A stake can have several. |
| **Points** | The leaderboard score. One deposited unit gives the source's `points` value. |
| **Deposit** | Putting resources in to gain points. |
| **Withdraw** | Taking resources back out during an active round (if allowed). |
| **Finalize** | The moment a round ends and rewards are calculated and stored. |
| **Pending reward** | A prize waiting in `/stakes rewards` to be claimed. |

---

## One rule that saves you headaches

> Always **test a new event on a test server** before opening it to real players, and **back up the
> database** before updating the plugin or changing big settings. These two habits prevent almost every
> painful problem in this wiki.
