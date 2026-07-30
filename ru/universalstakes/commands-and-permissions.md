# Команды и разрешения

По умолчанию **никто** не имеет разрешений UniversalStakes. Подарите постоянным игрокам пакет
**`universalstakes.use`** и передайте персоналу пакет **`universalstakes.admin`** (операторы получают его
автоматически). Вы также можете предоставить только те узлы, которые перечислены ниже.

---

## Команды игрока

| Команда | Что он делает |
| --- | --- |
| `/stakes` | Откройте главное меню. `/stake` и `/universalstakes` также работают. |
| `/stakes <stake-id>` | Откройте одно мероприятие. |
| `/stakes history` | Откройте историю депозитов. |
| `/stakes rewards` | Откройте свои ожидающие награды. |
| `/stakes help` | Показать справку по командам игрока. |
| `/<alias>` | Откройте событие через его псевдоним (например, `/diamonds`). |
| `/<alias> deposit` | Открытые источники вклада; щелкните левой кнопкой мыши по вкладам, щелкните правой кнопкой мыши по выводу средств (если включено). |
| `/<alias> top [page]` | Покажите таблицу лидеров в чате. |
| `/<alias> help` | Показать команды для этого события. |

Первое значение в `config.yml` под `command.aliases` — это **основная** корневая команда; остальные
псевдонимы одного и того же полного дерева. Они не чувствительны к регистру и должны быть уникальными. Изменение корней или события
aliases требуется **перезапуск**.

---

## Команды персонала

| Команда | Что он делает |
| --- | --- |
| `/stakes admin reload` | Проверьте и перезагрузите язык, меню, отображение/сообщения ставок и **новые** записи `currencies.yml`. См. [Перезагрузить и перезапустить](Reload-and-Restart.md). |
| `/stakes admin start <stake-id>` | Начать новый раунд. |
| `/stakes admin stop <stake-id>` | Завершить активный раунд. |
| `/stakes admin importitem <item-id>` | Сохраните предмет в руке как многоразовый идентификатор индивидуального предмета. |
| `/stakes admin logs <stake-id>` | Открыть завершенные раунды и их неизменяемые журналы аудита. |
| `/stakes admin help` | Показать справку по командам администратора. |

Неверный синтаксис отображает редактируемое сообщение `messages.commandUsage`. Ошибочно введенная подкоманда показывает
соответствующая страница справки.

---

## Полный список разрешений

| Разрешение | По умолчанию | Гранты |
| --- | --- | --- |
| `universalstakes.use` | ложный | Все стандартные команды игрока и узлы меню ниже. **Не** обходит собственный `permission` доли. |
| `universalstakes.admin` | оп | Все команды администратора и узлы меню аудита приведены ниже. |
| `universalstakes.command.main` | ложный | `/stakes` и настроенные корневые псевдонимы. |
| `universalstakes.command.history` | ложный | `/stakes history`. |
| `universalstakes.command.rewards` | ложный | `/stakes rewards`. |
| `universalstakes.command.stake` | ложный | `/stakes <stake-id>`. |
| `universalstakes.command.help` | ложный | `/stakes help`. |
| `universalstakes.command.alias.open` | ложный | `/<stake-alias>`. |
| `universalstakes.command.alias.top` | ложный | `/<stake-alias> top [page]`. |
| `universalstakes.command.alias.help` | ложный | `/<stake-alias> help`. |
| `universalstakes.menu.main` | ложный | Главное меню. |
| `universalstakes.menu.stake` | ложный | Меню выбранного события. |
| `universalstakes.menu.history` | ложный | Меню истории. |
| `universalstakes.menu.rewards` | ложный | Меню наград. |
| `universalstakes.menu.admin.logs` | ложный | Завершенное меню аудита. |
| `universalstakes.menu.admin.log-events` | ложный | Меню аудита по событию. |
| `universalstakes.command.admin.help` | ложный | `/stakes admin help`. |
| `universalstakes.command.admin.reload` | ложный | `/stakes admin reload`. |
| `universalstakes.command.admin.start` | ложный | `/stakes admin start <stake-id>`. |
| `universalstakes.command.admin.stop` | ложный | `/stakes admin stop <stake-id>`. |
| `universalstakes.command.admin.importitem` | ложный | `/stakes admin importitem <item-id>`. |
| `universalstakes.command.admin.logs` | ложный | `/stakes admin logs <stake-id>`. |

---

## Доступ по ставкам

Доступ к конкретному событию устанавливается **в файле ставки**, а не сгенерированным узлом разрешений. Добавить
`permission` в корне только тогда, когда событие необходимо ограничить:
```yaml
permission: "stake.diamondrush"
```
- Отсутствует или пусто (`permission: ""`) → событие **публично** для всех, у кого есть узлы команд/меню.
- Непусто → только игроки с таким **точным** разрешением могут видеть его или присоединиться к нему.

Предоставьте этот узел самостоятельно в своем плагине разрешений. `universalstakes.use` **не** предоставляет это
автоматически.
