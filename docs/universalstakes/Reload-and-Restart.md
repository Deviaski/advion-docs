---
order: 11
icon: refresh
---

# Reload & restart

`/stakes admin reload` **validates every edited file before applying it**. If validation fails, the
currently running registry and currency definitions stay in use. Read the console error, fix the file,
and run the command again.

This page tells you exactly what each kind of edit needs.

---

## Applied by `/stakes admin reload`

- The `locale` setting and all files in the selected `language/<locale>/` pack (menus, messages).
- `debug`, `input`, `banDisqualification`, and `auditLog` in `config.yml`.
- An existing stake's `display`, `messages` (including the reminder interval — it restarts a fresh
  interval after reload), and `withdrawals-enabled`.
- The **presentation** fields of contribution sources (`display-name`, `display`) and ranked prizes
  (`material`, `model-data`, `name`, `lore`). Financial rules and reward delivery do not change.
- A **new** stake file. Its startup state follows its `time.auto-restart`, but its `alias` command is
  not registered until a restart. It is still reachable via `/stakes <stake-id>`.
- **Removal** of an ended or not-yet-started stake. An active/closing/finalizing/initializing stake
  cannot be removed by reload.
- **New IDs added** to `currencies.yml`.
- Currency, point, and limit rules below `contributions` — but **only after the current round is fully
  `ENDED`**. Stop the stake and wait for finalization first.

---

## Require a full server restart

- `command.aliases` in `config.yml`, and any new or renamed stake `alias` (commands register only at
  startup).
- **Database settings** in `config.yml` (the connection pool is built only at startup).
- An existing stake's `permission`, timing (`duration`/`restart-delay`/`auto-restart`), reward
  commands, numbered consolation tiers, and `need-free-slots`.
- Contribution **currency, points, limits, or custom-item matching** while a round is not fully ended.
- Imported custom-item snapshots (after `/stakes admin importitem`).
- **Editing or removing** an existing `currencies.yml` definition (reload allows additions only —
  existing entries are protected because deferred rewards may still refer to them).

---

## Change contribution rules without a restart

Use this safe workflow to change a live stake's currency/points/limits:

1. Stop the stake: `/stakes admin stop <stake-id>`.
2. Wait until finalization completes — the round must be `ENDED`.
3. If you need a new external currency, **add** a new `currencies.yml` entry. Do **not** alter an
   existing one.
4. Change the stake's `contributions` block and run `/stakes admin reload`.

This keeps the finalized round and its pending rewards attached to their original currency ID, while
the next round uses the new one. Keep old `currencies.yml` entries until their pending refunds are
claimed or deliberately resolved.

---

## Startup and auto-restart behaviour

- `auto-restart: false` → a new stake does **not** start just because the server starts, and an ended
  stake stays stopped after a restart. Start it explicitly with `/stakes admin start <stake-id>`.
- `auto-restart: true` → the first round is created on startup. After a round ends, the next starts
  after `restart-delay`; if the server was offline when that delay elapsed, the new round starts on the
  next startup.
- A round that was **already active** when the server stopped is **always resumed** on restart,
  regardless of `auto-restart`, so deposits are never silently discarded.