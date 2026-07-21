# Integrations

## Vault economy

> **Use Vault only when:** Players should deposit virtual money instead of items.

Install Vault and a Vault-compatible economy plugin, then restart the server. Set `currency.type: VAULT` in the stake. If no economy is found, Vault stakes cannot accept deposits; item stakes still work.

> **SCREENSHOT TO ADD:** Console message confirming that a Vault economy was connected.

## PlaceholderAPI

PlaceholderAPI is optional. When it is installed, UniversalStakes registers its expansion automatically.

The two shipped stake cards do not require PlaceholderAPI: their `{status}` and `{time_left}` values are supplied by UniversalStakes itself. If you put `%universalstakes_...%` tokens into custom stake lore or menu text, install PlaceholderAPI; otherwise those percent-delimited tokens remain visible as literal text.

```text
%universalstakes_<stake-id>_<key>%
```

Available keys: `display_name`, `status`, `time_left`, `total`, `participants`, `player_invested`, `player_place`, `top_name_1`, and `top_amount_1`.

Example: `%universalstakes_diamonds_top_name_1%` shows the first-place player in the `diamonds` stake.

## Custom items

Use this for keys, named items, NBT items, or items with Custom Model Data. Create the stake first, hold the sample item, then run:

```text
/stakes admin importitem <stake-id> <item-id>
```

Restart the server before an existing stake uses the imported item. Choose the matching strictness with `match-mode`; the three modes are explained in [Stake Settings Reference](Stake-Reference.md).
