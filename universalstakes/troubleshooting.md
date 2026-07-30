# Troubleshooting

## The stake does not appear

Check that the file is in `plugins/UniversalStakes/stakes/`, ends in `.yml`, and has valid YAML indentation. Run `/stakes admin reload` to validate it and read the console, then perform a full server restart before expecting edits to an existing stake to apply.

## The alias command does not work

A new or renamed `alias` appears only after a full server restart. Also make sure another plugin does not already use that command.

## A player cannot open or join a stake

Give the command and menu nodes needed for that path, or the `universalstakes.use` bundle. If the
stake file has a non-empty root `permission:` value, also grant that exact value. A missing or empty
`permission:` means the stake is public. Then check that the round is active, the player has enough
currency, and the deposit is inside the configured min, max, and daily limits.

## A Vault stake cannot take money

Install Vault and an economy plugin. On startup, look for the console message confirming the economy connection. UniversalStakes does not create virtual money by itself.

## A reward is missing

Ask the player to run `/stakes rewards`. A confirmed temporary error leaves the reward available for another click. If the server crashed during delivery or the external result was ambiguous, the reward becomes `BLOCKED` to prevent duplication. Review the affected completed round through `/stakes admin logs <stake-id>`; its immutable audit events show the result and error.

An ambiguous investment withdrawal is automatically cancelled without crediting the stake. The same audit menu records it as `UNCONFIRMED`, including the operation ID and player.

## What to include in a support request

Include Paper, Java, and UniversalStakes versions; Vault/economy version if relevant; the full console error; the affected stake file with passwords removed; and exact steps to reproduce the problem.
