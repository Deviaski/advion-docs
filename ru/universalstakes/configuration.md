# Конфигурация

На этой странице описаны два глобальных файла: `config.yml` (общесерверные настройки) и `currencies.yml`.
(внешние валюты PlaceholderAPI). Файлы для каждого события см. в разделе [Настройки ставок](Stake-Settings.md).

> `config.yml` использует ключи **camelCase** (`sqliteFile`, `poolSize`). `currencies.yml` использует
> **кебаб-кебаб** (`raw-placeholder`, `give-command`). Оба преднамеренны — точно скопируйте написание.

---

## `config.yml`
```yaml
debug: false
locale: "english"

database:
  type: sqlite
  host: localhost
  port: 3306
  name: universalstakes
  user: universalstakes
  password: ""
  sqliteFile: data/data.db
  poolSize: 10
  sslMode: VERIFY_IDENTITY
  trustCertificateKeyStoreUrl: ""
  trustCertificateKeyStorePassword: ""
  allowPublicKeyRetrieval: false

command:
  aliases:
    - universalstakes
    - stake
    - stakes

input:
  promptTimeoutSeconds: 120

banDisqualification:
  activeRounds: true
  endedRounds: false
  checkIntervalSeconds: 60

auditLog:
  retentionDays: 0
```
### Объяснено каждое поле

| Поле | По умолчанию | Значение |
| --- | --- | --- |
| `debug` | `false` | Выведите дополнительную диагностику (включая трассировку стека) на консоль. |
| `locale` | `english` | Языковой пакет загружен из `language/<locale>/`. Должен соответствовать установленной папке. |
| `database.type` | `sqlite` | `sqlite` или `mysql`. Неизвестное значение останавливает запуск вместо автоматического использования SQLite. |
| `database.host` | `localhost` | Хост MySQL (только MySQL). |
| `database.port` | `3306` | Порт MySQL (только MySQL). |
| `database.name` | `universalstakes` | Имя базы данных MySQL (только MySQL). |
| `database.user` / `password` | — | Учетные данные MySQL (только MySQL). |
| `database.sqliteFile` | `data/data.db` | Путь к файлу SQLite относительно папки плагина. |
| `database.poolSize` | `10` | Максимальный размер пула HikariCP. |
| `database.sslMode` | `VERIFY_IDENTITY` | Режим MySQL TLS. `VERIFY_IDENTITY` (рекомендуется), `VERIFY_CA`, `REQUIRED`, `PREFERRED` или `DISABLED`. |
| `database.trustCertificateKeyStoreUrl` | `""` | Необязательный URL-адрес хранилища доверенных сертификатов для MySQL TLS, например. `file:/path/to/truststore.p12`. |
| `database.trustCertificateKeyStorePassword` | `""` | Пароль для этого хранилища доверенных сертификатов. |
| `database.allowPublicKeyRetrieval` | `false` | Разрешить получение открытого ключа RSA. Сохраните `false`, если этого не требует ваша установка. |
| `command.aliases` | `universalstakes, stake, stakes` | Корневые команды. Первое является первичным; остальные являются псевдонимами одного и того же дерева. Для изменения требуется перезагрузка. |
| `input.promptTimeoutSeconds` | `120` | За несколько секунд до истечения срока действия подсказки «введите сумму» в чате. |
| `banDisqualification.activeRounds` | `true` | Удалить забаненных игроков из активных таблиц лидеров (без возврата средств). |
| `banDisqualification.endedRounds` | `false` | Удалите забаненных игроков из завершившихся раундов и удалите их невостребованные награды. |
| `banDisqualification.checkIntervalSeconds` | `60` | Как часто сканировать участников на наличие банов. |
| `auditLog.retentionDays` | `0` | Дни хранения событий административного аудита. `0` = хранить вечно. |

> Пароль базы данных и пароль хранилища доверенных сертификатов хранятся в виде **обычного текста** в `config.yml`. Ограничить
> доступ файловой системы к учетной записи сервера и не допускайте попадания этого файла в общие резервные копии/журналы. См.
> [База данных и безопасность](Database-and-Safety.md).

### Какая перезагрузка применяется

`/stakes admin reload` применяется `locale`, `debug`, `input`, `banDisqualification` и `auditLog`.
**Настройки базы данных и `command.aliases` требуют перезагрузки** (пул соединений и команды
строится только при запуске).

---

## `currencies.yml`

Требуется только в том случае, если вы используете валюты `PLACEHOLDER`. Сгенерированный файл содержит PlayerPoints и
**шаблоны** ExcellenceEconomy** — перед использованием сравните их с вашей экономикой.
```yaml
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
| Поле | Значение |
| --- | --- |
| `display-name` | Обязательное имя, используемое для проверки. |
| `symbol` | Необязательные метаданные (не отображаются автоматически в текущих меню). |
| `raw-placeholder` | Должен возвращать **только цифры** — без символов, запятых, десятичных знаков или цветовых кодов. |
| `give-command` / `take-command` | Запускать от имени **консоли**; должен содержать `%player%` и `%amount%`. Ведущий `/` не обязателен. |
| `works-offline` | Может ли провайдер работать на оффлайн плеере? |
| `balance-check-after` | Перечитайте баланс после команды и требуйте его изменения ровно на эту сумму. |
| `balance-check-delay-ticks` | Отмечено ожидание перед проверкой после команды (для асинхронной экономики). |

### Правила перезагрузки валют

- **Добавление** нового идентификатора: применен `/stakes admin reload`.
- **Редактирование или удаление** существующего идентификатора: требуется **перезапуск** и **небезопасно** при использовании ставки.
  у него все еще есть невостребованные возмещения (в ожидании возмещения хранится идентификатор, а не копия старой команды).

Подробную информацию о том, как поэтапно подтверждаются депозиты PLACEHOLDER, см. в разделе [Currency](Currencies.md).

---

## Доверие конфигурации

Файлы под `plugins/UniversalStakes/` являются частью доверенной административной поверхности вашего сервера. Некоторые
настроенные действия, выполняемые с **правами консоли**: действия меню `console:`, валюта PlaceholderAPI
`give`/`take` команды и команды вознаграждения. Это необходимо для интеграции экономики/предметов, но это
означает, что эти файлы никогда не должны быть доступны для записи обычными проигрывателями, непроверенной автоматизацией или веб-панелью.
роли предназначены только для модераторов. Относитесь к изменениям конфигурации так же, как к изменениям команд сервера — просмотрите их перед применением.
