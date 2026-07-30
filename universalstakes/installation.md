# Installation

> **You need:** Paper 1.21.7 (the verified server version), Java 21, the UniversalStakes release JAR, and outbound HTTPS for the first start. Do not use a JAR whose name contains `SNAPSHOT` on a live server. Test later Paper versions on staging before production use.

## Install in seven steps

1. Stop the Minecraft server.
2. Put the UniversalStakes JAR in the server’s `plugins` folder.
3. Start the server once. UniversalStakes creates its folders and example files.
4. Stop the server again.
5. Open `plugins/UniversalStakes/`.
6. Edit or remove the example stake files in `stakes/`.
7. Start the server and run `/stakes` in game.

## First-start network requirement

The plugin JAR does not embed the database stack. Paper reads `plugin.yml` and downloads HikariCP, Jdbi, SQLite JDBC, MySQL Connector/J, and their transitive dependencies from Maven Central. This happens even though SQLite is the default database.

For an offline server, first start the exact same release on a staging Paper installation with network access, then transfer the complete Paper library cache together with the server. A missing or blocked Maven artifact prevents the plugin from loading; check the Paper console before troubleshooting UniversalStakes configuration.

> **SCREENSHOT TO ADD:** `plugins/UniversalStakes/` after the first start. Highlight `config.yml`, `stakes`, `gui`, and `language`.

## Optional plugins

| Plugin | Do I need it? |
| --- | --- |
| Vault and an economy plugin | Only for stakes that use virtual money (`VAULT`). |
| PlaceholderAPI | Only for placeholders in other plugins or custom text. Shipped stake cards use built-in `{status}` and `{time_left}` tokens. |

Item-based stakes work without both optional plugins.

## What the folders are for

- `config.yml` — database, input timeout, and global options.
- `stakes/` — one `.yml` file per competition.
- `gui/` — the inventory menus players see.
- `language/` — messages sent by the plugin.

**Next step:** [Create your first stake](First-Stake.md).
