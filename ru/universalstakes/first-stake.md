# Ваша первая ставка

> **Цель**: часовая гонка с одним победителем. Самое безопасное первое событие для тестирования.

Вы создадите один файл, перезапустите и протестируете полный цикл: депозит → проверка верха → завершение раунда →
претендовать на приз.

---

## 1. Создайте файл

Создайте `plugins/UniversalStakes/language/english/stakes/diamonds.yml` и вставьте это:
```yaml
display:
  display-name: "Diamond Race"
  name: "<aqua>Diamond Race"
  lore:
    - "<gray>Deposit diamonds and reach first place!"
  material: DIAMOND

alias: "diamonds"
withdrawals-enabled: true

messages:
  started:
    - "{prefix}<green>{stake} has been started."
  stopped:
    - "{prefix}<green>{stake} has been stopped and finalized."
  reminder:
    interval: "5m"
    message:
      - "{prefix}<aqua>{stake}</aqua> is active! Time left: <yellow>{time_left}</yellow>"

contributions:
  diamonds:
    display-name: "Diamonds"
    currency:
      type: ITEM
      material: DIAMOND
    points: 5
    display:
      material: DIAMOND
      name: "<aqua>Diamonds"
      lore:
        - "<gray>Deposited: {deposited}"
        - "<gray>1 diamond = {points-per-unit} points"
    limits:
      min-invest: 1
      max-invest: 64
      daily-limit: 256

time:
  duration: "1h"
  restart-delay: "10m"
  auto-restart: true

prizes:
  top:
    1:
      material: DIAMOND_BLOCK
      name: "<gold>First place"
      lore:
        - "<gray>Reward: 16 diamond blocks"
      need-free-slots: 1
      rewards:
        - "give %player% diamond_block 16"
```
> Хотите ограничить круг лиц, которые могут присоединиться? Добавьте `permission: "stake.diamonds"` вверху и разрешите этот узел в
> ваш плагин разрешений. Оставьте это для публичного мероприятия.

---

## 2. Запустите и протестируйте

**Перезапустите сервер.** Затем, как игрок с `universalstakes.use`:

1. Запустите `/diamonds`, чтобы открыть событие.
2. Запустите `/diamonds deposit`, нажмите **Бриллианты**, затем **щелкните левой кнопкой мыши** и введите `5` в чат.
   Вы только что внесли 5 бриллиантов → 25 очков.
3. Запустите `/diamonds top`, чтобы просмотреть таблицу лидеров.
4. Поскольку `withdrawals-enabled: true`, **щелкните правой кнопкой мыши** ромбы и введите `2`. Два бриллианта приходят
   прямиком обратно в инвентарь и 10 очков снимаются.
5. Как оператор, завершите тест досрочно с помощью `/stakes admin stop diamonds`.
6. Откройте `/stakes rewards` и получите приз. Убедитесь, что у вас есть 1 свободный слот инвентаря.
   (`need-free-slots: 1`), иначе вознаграждение останется в ожидании.

---

## 3. Измените это позже

Некоторые изменения применяются с помощью `/stakes admin reload`, другие требуют полного перезапуска. Краткая версия:

- **Применяется перезагрузка:** отображение, сообщения, напоминания, `withdrawals-enabled`, текст меню и добавление
  совершенно новый файл ставок.
- **Требуется перезагрузка**: `alias`, время, команды вознаграждения, уровни поддержки, `permission` и
  изменение валюты/баллов/лимитов ставки, раунд которой еще не завершен.

Точные правила перед редактированием прямой трансляции см. в разделе [Перезагрузка и перезапуск](Reload-and-Restart.md).

---

## Куда идти дальше

- Объяснение каждого поля: [Настройки ставки](Stake-Settings.md).
- Более одного победителя, возврат средств, утешительные призы: [Rewards](Rewards.md).
- Используйте деньги или специальные предметы вместо бриллиантов: [Currency](Currencies.md).
