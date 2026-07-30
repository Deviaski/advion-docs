# Troubleshooting

## The event does not appear

- The file must be in `plugins/UniversalStakes/language/<locale>/stakes/`, end in `.yml`, and have
  valid YAML indentation.
- Run `/stakes admin reload` and read the console — it names the file and the invalid field.
- A new file appears after reload, but a **renamed/edited existing event** may need a full restart for
  some changes (see [Reload & restart](Reload-and-Restart.md)).

## The alias command does not work

A new or renamed `alias` appears only after a **full restart** (commands register at startup). Also
check that another plugin is not already using that command, and that the alias is unique and not one
of the root commands in `config.yml`.

## A player cannot open or join an event

1. Give the command and menu nodes for that path, or the `universalstakes.use` bundle.
2. If the stake file has a non-empty `permission:`, grant that **exact** node too. Missing/empty means
   the event is public.
3. Check the round is **active**, the player has enough of the resource, and the deposit is within
   `min-invest`, `max-invest`, and `daily-limit`.

## A Vault event cannot take money

Install **Vault** and a Vault-compatible economy plugin, then restart. On startup, look for the
console message confirming the economy connected. UniversalStakes does not create money itself.

## A PLACEHOLDER currency says it could not be confirmed

- Make sure PlaceholderAPI **and** the target economy plugin are enabled.
- Test the exact `raw-placeholder` for the player. It must return **digits only**: no symbol, commas,
  decimals, color codes, or other text.
- Test the configured `give-command` and `take-command` from the console.
- With `balance-check-after: true`, the balance must change by **exactly** the requested integer
  amount after `balance-check-delay-ticks`. **Increase the delay** if the economy updates its
  placeholder asynchronously.
- Do **not** change an existing currency definition while it still has pending refunds — see
  [Database & safety](Database-and-Safety.md).

## A reward is missing

- Ask the player to run `/stakes rewards`. A confirmed temporary error leaves the reward `PENDING` for
  another click.
- If the server crashed during delivery or the result was ambiguous, the reward is `BLOCKED` to
  prevent duplication. Review the round in `/stakes admin logs <stake-id>` — its audit events show the
  result and error.
- An ambiguous currency withdrawal during a deposit is cancelled without crediting the stake and
  recorded as `UNCONFIRMED` (with operation ID and player).

## A withdrawal was deducted but not returned

A right-click withdrawal reduces the recorded contribution and then pays the resource **directly** —
it never appears in `/stakes rewards`. If the payout failed after the ledger change, the audit event
is `WITHDRAWAL` with status `PAYOUT_FAILED`.

1. Check the console diagnostic immediately.
2. After the round completes, verify the player, amount, currency type, operation ID, and error in
   `/stakes admin logs <stake-id>`.
3. Fix the provider or inventory problem and **compensate the player manually**. Do **not** tell them
   to repeat the withdrawal — the original contribution was already removed.

## A round is stuck / not accepting deposits

If the console reports a round in an **`ERROR`** state, that round has an unknown database status and
fails closed: no deposits are accepted. It requires **manual database reconciliation** — do not try to
force it open. Back up the database first, then inspect the round row for the affected stake.

## What to include in a support request

- Paper, Java, and UniversalStakes versions.
- Vault/economy version, if relevant.
- The **full console error**.
- The affected stake file (with passwords removed).
- Exact steps to reproduce.
