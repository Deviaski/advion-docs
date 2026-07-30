---
order: 5
icon: coins
---

# Currencies

A contribution source takes a `currency` block. There are **four types**. Pick the one that matches
what players should deposit.

```yaml
contributions:
  diamonds:
    currency:
      type: ITEM          # or VAULT / PLACEHOLDER / CUSTOM_ITEM
      material: DIAMOND   # depends on type
    points: 5
```

## The four types at a glance

| `type` | What players deposit | Extra setup needed |
| --- | --- | --- |
| `ITEM` | A normal Minecraft item (diamonds, emeralds…). | `material:` only. |
| `VAULT` | Virtual money from your economy plugin. | Install Vault + an economy plugin. |
| `PLACEHOLDER` | An integer balance from an economy without a Vault bridge. | PlaceholderAPI + a `currencies.yml` entry. |
| `CUSTOM_ITEM` | An imported item snapshot (keys, named items, NBT, custom model data). | `/stakes admin importitem`. |

---

## `ITEM` — a plain Minecraft item

The simplest type. Players deposit a vanilla material from their inventory.

```yaml
currency:
  type: ITEM
  material: DIAMOND
```

`material:` is any valid Bukkit material name (uppercase, e.g. `DIAMOND`, `EMERALD`, `CARROT`).

---

## `VAULT` — virtual money

Use this when players should deposit server money instead of items.

```yaml
currency:
  type: VAULT
```

Requirements:

1. Install **Vault** and a Vault-compatible economy plugin. Restart the server.
2. Look for the console message confirming the economy connected.
3. Give players the deposit permission as usual.

If no economy is found, `VAULT` sources cannot accept deposits (item sources in the same stake still
work). UniversalStakes does **not** create money itself.

---

## `PLACEHOLDER` — economy without a Vault bridge

For economies that expose a balance through PlaceholderAPI but have no Vault hook (PlayerPoints,
ExcellentEconomy, CoinsEngine, …). The stake refers to a currency by its **ID**, and the actual
definition lives in `currencies.yml`.

```yaml
# in the stake file
contributions:
  gems:
    currency:
      type: PLACEHOLDER
      id: gems
    points: 1
```

```yaml
# in plugins/UniversalStakes/currencies.yml
currencies:
  gems:
    display-name: "Gems"
    symbol: ""
    raw-placeholder: "%coinsengine_balance_gems%"
    give-command: "ce give %player% gems %amount%"
    take-command: "ce take %player% gems %amount%"
    works-offline: false
    balance-check-after: true
    balance-check-delay-ticks: 5
```

### `currencies.yml` fields

| Field | Meaning |
| --- | --- |
| `display-name` | Required name used for validation. |
| `symbol` | Optional metadata (not auto-shown in current menus). |
| `raw-placeholder` | Must return **digits only** — no symbol, commas, decimals, color codes. |
| `give-command` / `take-command` | Run as **console**. Must contain `%player%` and `%amount%`. A leading `/` is optional. |
| `works-offline` | May the provider read/operate on an offline player? Normal deposits are started by online players. |
| `balance-check-after` | After the command, re-read the balance and require it changed by exactly the amount. |
| `balance-check-delay-ticks` | Wait this many ticks before the post-command balance check (for async-updating economies). |

### How a PLACEHOLDER deposit is confirmed

1. Read the player's balance from `raw-placeholder`.
2. Run `take-command` as console.
3. Wait `balance-check-delay-ticks`, then read the balance again.
4. The balance must have dropped by **exactly** the requested integer amount.
5. If it cannot be confirmed, the deposit is **cancelled without awarding points** and the audit log
   records `UNCONFIRMED`. It is never retried automatically.

> The generated `currencies.yml` ships PlayerPoints and ExcellentEconomy **templates**. Verify the
> currency ID, command alias and placeholder against your own economy config before using them.

> Never edit or remove an existing `currencies.yml` entry while a stake using it still has unclaimed
> refunds — pending refunds store the ID, not a copy of the old command. Reload permits **adding**
> entries only; changing/removing requires a restart and is unsafe until those rewards are resolved.

---

## `CUSTOM_ITEM` — imported item snapshots

For keys, renamed items, NBT items, or items with Custom Model Data.

### Step 1 — import the item

Hold the sample item in your **main hand** and run:

```text
/stakes admin importitem <item-id>
```

The item is saved globally as `plugins/UniversalStakes/imported-items/<item-id>.item`. The item ID is
global and can be reused by several stakes.

### Step 2 — use it in a stake

The contribution **source ID must equal the imported item ID**. Do **not** add `currency.id` or an
`item:` field — custom-item lookup always uses the source ID.

```yaml
contributions:
  magic_key:              # this ID must match the imported item ID
    currency:
      type: CUSTOM_ITEM
      match-mode: SIMILAR
    points: 10
```

### `match-mode` — how stacks are matched

| Mode | What it checks |
| --- | --- |
| `SIMILAR` | The normal, lenient choice. Matches the item type and key item data. |
| `EXACT` | Requires a full, exact match of the whole stack. |
| `CUSTOM_MODEL_DATA` | Checks material **and** custom model data only. |

Importing a new snapshot (or re-importing) takes effect after a **restart**.

---

## Mixing sources

A stake can have several contribution sources of different types. Their points simply add together;
the resources never convert into each other.

```yaml
contributions:
  diamonds:
    currency: { type: ITEM, material: DIAMOND }
    points: 5
  coins:
    currency: { type: VAULT }
    points: 1
```

10 diamonds (×5 = 50) + 1000 coins (×1 = 1000) = 1050 total points.