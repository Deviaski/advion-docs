# Menus

> **Simple rule:** Change text, icons, and lore first. Do not remove dynamic sections unless you intentionally want to remove their lists.

Menu files are in `plugins/UniversalStakes/gui/`:

| File | What it shows |
| --- | --- |
| `stakes-main.yml` | All stake cards. |
| `stakes-menu.yml` | One selected stake, its prizes, and player position. |
| `stakes-history.yml` | Player deposit history. |
| `stakes-rewards.yml` | Waiting rewards. |

## Easy visual changes

Change `title`, `name`, `lore`, `material`, and `texture` inside a button. Run `/stakes admin reload` to reload the menus.

> **SCREENSHOT TO ADD:** `stakes-main.yml` beside the matching in-game menu. Mark the title, a button, and the dynamic stake area.

## Understanding `pattern`

Every `pattern` row contains nine space-separated symbols. A symbol points to a `buttons` entry or a `dynamic-items` entry.

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

Here `+` makes the border and `@` is filled with stake cards. You may repeat symbols as often as needed.

## Button actions

Put actions in `actions.left`; they run from top to bottom. Common actions are `menu: stakes-main`, `sound: UI_BUTTON_CLICK 1 1`, `close`, `page: next`, `page: previous`, `invest: {stake_id}`, `claim`, and `claim: {reward_id}`.

The built-in dynamic lists provide their own data. Keep `stakes`, `prizes`, `history`, and `rewards` sections if you want those lists to appear.
