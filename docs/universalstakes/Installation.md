---
order: 2
icon: download
---

# Installation

> **You need:** Paper 1.20.4 through 26.2, and the UniversalStakes release JAR. Use Java 21 on Paper
> 1.20.4–1.21.11, or Java 25 on Paper 26.1–26.2. Never use a JAR whose name contains `SNAPSHOT` on a
> live server.

## Install in 7 steps

1. **Stop** the Minecraft server.
2. Put the UniversalStakes JAR into the server's `plugins/` folder.
3. **Start** the server once. UniversalStakes creates its folders and example files.
4. **Stop** the server again.
5. Open `plugins/UniversalStakes/`.
6. In `config.yml`, choose your language with `locale: english` (or another installed pack). Edit the
   example events in `language/english/stakes/`.
7. **Start** the server and run `/stakes` in game.

That's it. The two shipped example events (*Coin Fever* and *Diamond Rush*) work right away for
item- and Vault-based deposits.

---

## First start needs internet (once)

The plugin JAR does **not** bundle the database libraries. On the first start, Paper reads
`plugin.yml` and downloads HikariCP, Jdbi, the SQLite driver, the MySQL driver, and their dependencies
from Maven Central. This happens even though SQLite is the default.

- Allow **outbound HTTPS** for that first start.
- For an **offline server**, first run the exact same release on a staging server that has internet,
  then copy Paper's full library cache over together with the server. If a needed library cannot be
  downloaded, the plugin will not load — check the Paper console before blaming the plugin config.

---

## Optional plugins

| Plugin | Do you need it? |
| --- | --- |
| **Vault** + an economy plugin | Only if you have stakes that deposit virtual money (`type: VAULT`). |
| **PlaceholderAPI** | Required only for `PLACEHOLDER` currencies. Optional otherwise — it also lets other plugins read UniversalStakes placeholders. |

Item-based events work with **neither** of these installed.

---

## A quick tour of the folder

After the first start you will see:

| Path | Purpose |
| --- | --- |
| `config.yml` | Global settings: database, command names, language, safety. |
| `currencies.yml` | Registry for external PlaceholderAPI currencies (ships with PlayerPoints + ExcellentEconomy examples). |
| `data/data.db` | The SQLite database (the default). |
| `imported-items/` | Custom-item snapshots created by `/stakes admin importitem`. |
| `language/<locale>/` | The active language pack: `messages.yml`, `menus/`, `stakes/`. |

For a full explanation of every file and how they fit together, see
[How it works](How-It-Works.md).

---

## Give access to players and staff

By default **nobody** has UniversalStakes permissions.

- Give regular players the bundle **`universalstakes.use`**.
- Give staff the bundle **`universalstakes.admin`** (operators get it automatically).

See [Commands & permissions](Commands-and-Permissions.md) for the full list and per-stake access.

---

**Next:** make a working event in 5 minutes — [Your first stake](First-Stake.md).