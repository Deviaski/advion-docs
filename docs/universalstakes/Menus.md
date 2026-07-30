---
order: 7
icon: table
---

# Menus

> **Simple rule:** change text, icons and lore first. Do not remove a `dynamic-items` section unless
> you intentionally want to hide that list.

Menu files live in `plugins/UniversalStakes/language/<locale>/menus/`. Edit them, then run
`/stakes admin reload` to apply.

## The seven menu files

| File | What it shows |
| --- | --- |
| `stakes-main.yml` | All event cards (the home menu). |
| `stakes-menu.yml` | One selected event: its prizes and the player's current place. |
| `deposit-menu.yml` | Contribution sources; left-click deposits, right-click withdraws (if allowed). |
| `stakes-history.yml` | The player's deposit history. |
| `stakes-rewards.yml` | Waiting rewards, with a Claim All button. |
| `stakes-admin-logs.yml` | Completed rounds, for administrators. |
| `stakes-admin-log-events.yml` | Immutable financial/reward events for one round (admin audit). |

---

## Easy visual changes

Inside any button you can change `title` (menu title), `name`, `lore`, `material`, and `texture` (for
player-head icons). Then `/stakes admin reload`. You do not need to touch the layout for that.

---

## Understanding `pattern`

Every menu is drawn from a `pattern`. Each row has **nine** space-separated symbols. A symbol points
to either a `buttons` entry or a `dynamic-items` entry. A `.` is an empty slot.

```yaml
pattern:
  - "+ + + + + + + + +"
  - "+ @ @ @ @ @ @ @ +"
buttons:
  '+':
    material: BLACK_STAINED_GLASS_PANE
    name: " "
dynamic-items:
  stakes:
    key: "@"
```

Here `+` is the border and `@` is filled with event cards. You can repeat a symbol as many times as
you like. The number of rows in `pattern` sets the menu size.

---

## Buttons

A button maps a pattern symbol to an item and optional actions:

```yaml
buttons:
  'X':
    material: IRON_DOOR
    name: "<red>Back"
    actions:
      left:
        - "sound: UI_BUTTON_CLICK 1.0 1.0"
        - "menu: stakes-main"
```

`actions.left` runs on a left-click, `actions.right` on a right-click. Actions run **top to bottom**.

---

## Button actions (the full list)

| Action | What it does |
| --- | --- |
| `menu: stakes-main` | Open another menu. Use `menu: stakes-menu {stake_id}` to pass a stake. |
| `sound: UI_BUTTON_CLICK 1.0 1.0` | Play a sound with optional volume and pitch. |
| `close` | Close the inventory. |
| `page: next` / `page: previous` | Move through a dynamic list's pages. |
| `invest: {stake_id}` | Open that stake's contribution selector. |
| `contribution: {stake_id}\|{source_id}` | Prompt for an amount. Left-click deposits; right-click withdraws when `withdrawals-enabled: true`. |
| `claim` | Claim all pending rewards. |
| `claim: {reward_id}` | Claim one specific reward. |
| `message: messages.someKey` | Send a message defined in `messages.yml`. |
| `command: stakes` | Run a command **as the player**. |
| `console: give %player% diamond 1` | Run a command **as the console**. |
| `permission: some.node \| deny: messages.noPermission` | Stop the remaining actions if the player lacks the node. |
| `delay: 20` | Wait this many ticks before running the rest of the action chain. |

`{...}` tokens are filled in from the menu's context before the action runs.

> `console:` and reward commands run with **console authority**. Only trusted staff should edit menu
> files. See [Database & safety](Database-and-Safety.md).

---

## Dynamic sections (auto-filled lists)

A `dynamic-items` entry fills its pattern symbol with an automatic list. Each one has a `key` (the
pattern symbol) and the list is paginated automatically.

| Section name | Filled with |
| --- | --- |
| `stakes` | The event cards. |
| `prizes` | One slot per configured top prize for the open stake. |
| `self` | The viewer's own place and invested amount. |
| `contributions` | The contribution sources for the open stake. |
| `history` | The viewer's deposit history entries. |
| `rewards` | The viewer's pending rewards. |
| `audit-rounds` | Completed rounds (admin menu). |
| `audit-events` | Events for one completed round (admin menu). |

Keep the matching section if you want that content to appear. All paginated sections share the same
current page; a single-item widget like `self` stays visible while a bigger list is paged.

---

## The `empty` fallback

When a dynamic list has nothing to show, the `empty` block is placed at its `slot`:

```yaml
empty:
  slot: 22
  material: BARRIER
  name: "<red>No rewards yet"
  lore:
    - "<gray>Play in events and win prizes."
```

---

## Reload safety

Before replacing the live menus, reload **validates** every menu file. If a menu has an invalid
material, layout, dynamic provider, action, or menu reference, the reload is **rejected** and the
previously working menu stays active. The console names the file and field. Validation checks
structure, not the safety of commands — that remains your responsibility.