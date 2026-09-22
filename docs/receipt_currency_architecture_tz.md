# ТЗ: мультивалютная архитектура WinePool для чеков, цен и аналитики

Дата актуализации: 22 июня 2026 г.

Статус: базовая мультивалютная архитектура реализована. Первый рабочий фокус `BYN` закрыт, `UZS` подключён вместе с провайдером чеков Узбекистана, ручные цены поддерживают выбор валюты в UI.

Статус production self-host на 22 июня 2026 г.:

- применена миграция `20260620_add_currency_foundation.sql`;
- развернута Edge Function `sync-exchange-rates`;
- настроен VPS systemd timer `winepool-sync-exchange-rates.timer` на ежедневный запуск в 04:20 MSK;
- загружены курсы НБРБ для `2026-05-28`, `2026-06-19`, `2026-06-20`;
- выполнен backfill существующего BYN-чека: `16.99 BYN` -> примерно `439 RUB` в `user_prices.price_base`;
- в production не осталось non-RUB receipt/user price строк без base-суммы.
- добавлен парсер чеков Узбекистана и валюта `UZS`;
- ценовые агрегаты карты и карты конкретного вина используют base-цены, чтобы BYN/UZS-покупки корректно участвовали в средних/медианах;
- карта покупок ограничена явными позициями вина из чеков, чтобы не строить агрегаты по служебным/неразобранным строкам.

## 1. Контекст

WinePool уже умеет получать чеки разных фискальных юрисдикций:

- Россия: чековая валюта `RUB`.
- Беларусь: чековая валюта `BYN`.
- Узбекистан: чековая валюта `UZS` после подключения UZ provider.
- Ручные сценарии: пользователь может внести покупку в любой поддерживаемой валюте, например `USD`.

Чековый flow и язык интерфейса - разные оси. Пользователь может использовать английский интерфейс и сканировать белорусский чек; русскоязычный пользователь может выбрать отображение цен в долларах; пользователь в Беларуси может хотеть видеть всю аналитику в `BYN`.

Главная цель: хранить финансовый факт покупки точно, а показывать цену в той валюте, которую выбрал пользователь.

## 2. Термины

### 2.1. Source / original currency

Валюта источника, в которой цена реально пришла в систему:

- из российского чека: `RUB`;
- из белорусского чека: `BYN`;
- из узбекского чека: `UZS`;
- из ручного ввода: валюта, выбранная пользователем рядом с полем цены.

Эта валюта является provenance-фактом и не должна теряться.

### 2.2. Display currency

Валюта интерфейса, выбранная пользователем в профиле. Именно в ней приложение показывает цены в обычных пользовательских поверхностях:

- карточки вин;
- погребок;
- карта покупок;
- списки чеков;
- статистика цены;
- аналитика;
- будущий винный индекс.

Display currency не зависит напрямую от языка интерфейса.

### 2.3. Base currency

Внутренняя нормализованная валюта для сравнений, медиан, подозрительных цен, индекса и кросс-страновой аналитики. На первом этапе base currency фиксируем как `RUB`, потому что текущая ценовая модель исторически рублевая.

Base currency - технический слой. Пользователь не обязан его видеть.

### 2.4. Historical rate

Курс на дату покупки, а не текущий курс. Для receipt/user price фактов нужно сохранять курс, которым была посчитана base price, рядом с записью.

## 3. Продуктовое правило отображения

### 3.1. Не показываем все валюты везде

Обычный интерфейс должен показывать одну цену в одной валюте: выбранной `display_currency`.

Неверный вариант для списков:

```text
16.99 BYN / 447 ₽ / $6.09
```

Правильный вариант для списков, если пользователь выбрал `RUB`:

```text
447 ₽
```

Правильный вариант для списков, если пользователь выбрал `USD`:

```text
$6.09
```

### 3.2. Где показывать original currency

Исходную валюту показываем только там, где важен источник факта:

- детали чека;
- детали конкретной покупки в погребке;
- audit/debug/admin surfaces;
- всплывающая детализация "как посчитано";
- экспорт/выгрузка.

Пример в деталях чека:

```text
Оплачено по чеку: 16.99 BYN
В вашей валюте: 447 ₽
Курс: 1 BYN = 26.32 RUB, НБРБ/ЦБ РФ, дата 2026-06-20
```

### 3.3. Приоритеты выбора display currency

1. `profiles.preferred_display_currency`, если пользователь выбрал валюту.
2. `profiles.home_country` или явный country hint, если он есть.
3. Регион устройства / геопозиция как слабая подсказка, только для дефолта.
4. `RUB` как текущий исторический default WinePool.

Язык интерфейса (`ru`, `en`, будущий `be`) не должен принудительно менять валюту.

## 4. Поддерживаемые валюты

Поддерживается в текущем коде:

- `RUB`;
- `BYN`;
- `USD`.
- `UZS`;

Следующие кандидаты:

- `EUR`.

Архитектура должна допускать добавление валют без миграции бизнес-логики.

## 5. Источники курсов

### 5.1. Беларусь, BY-first

Основной источник: API Национального банка Республики Беларусь.

Пример endpoint:

```text
https://api.nbrb.by/exrates/rates?periodicity=0
```

Важная особенность НБРБ: `Cur_OfficialRate` задаётся за `Cur_Scale` единиц валюты относительно `BYN`. Например:

- `USD`: 1 USD = N BYN;
- `RUB`: 100 RUB = N BYN.

Для BYN -> RUB можно считать через строку `RUB`:

```text
1 RUB = Cur_OfficialRate / Cur_Scale BYN
1 BYN = Cur_Scale / Cur_OfficialRate RUB
```

Для BYN -> USD:

```text
1 USD = Cur_OfficialRate BYN
1 BYN = 1 / Cur_OfficialRate USD
```

### 5.2. Россия

Источник: Банк России.

Официальный XML endpoint для дневных курсов:

```text
https://www.cbr.ru/scripts/XML_daily.asp?date_req=dd/mm/yyyy
```

Используем для проверки и для пар `RUB -> USD`, `RUB -> BYN`, `RUB -> UZS`, если нужна независимая рублевая база.

### 5.3. Узбекистан

Источник: Центральный банк Республики Узбекистан.

JSON endpoint:

```text
https://cbu.uz/ru/arkhiv-kursov-valyut/json/
```

Подключается вторым этапом после стабилизации BYN.

## 6. Модель данных

### 6.1. exchange_rates

Новая таблица:

```sql
CREATE TABLE public.exchange_rates (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  rate_date date NOT NULL,
  from_currency varchar(3) NOT NULL,
  to_currency varchar(3) NOT NULL,
  rate numeric NOT NULL,
  source text NOT NULL,
  quality text NOT NULL DEFAULT 'exact_historical',
  fetched_at timestamptz NOT NULL DEFAULT now(),
  raw_payload jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at timestamptz NOT NULL DEFAULT now(),
  UNIQUE (rate_date, from_currency, to_currency, source)
);
```

Правило значения `rate`:

```text
amount_to = amount_from * rate
```

То есть запись `BYN -> RUB = 26.32` означает:

```text
16.99 BYN * 26.32 = 447.37 RUB
```

Допустимые `quality`:

- `exact_historical`;
- `fallback_previous_day`;
- `fallback_latest`;
- `manual_override`;
- `missing`.

### 6.2. profiles

Добавить:

```sql
preferred_display_currency varchar(3) DEFAULT 'RUB'
preferred_manual_entry_currency varchar(3) DEFAULT 'RUB'
home_country varchar(2)
```

`preferred_display_currency` управляет тем, что видит пользователь.

`preferred_manual_entry_currency` управляет дефолтом в ручных формах цены.

### 6.3. user_receipts

Сейчас уже есть:

- `receipt_country`;
- `receipt_provider`;
- `provider_receipt_id`;
- `currency`;
- `total_amount`;
- `provider_payload`.

Добавить:

```sql
original_currency varchar(3)
base_currency varchar(3) DEFAULT 'RUB'
total_amount_original numeric
total_amount_base numeric
fx_rate_to_base numeric
fx_rate_date date
fx_source text
fx_quality text
```

Совместимость:

- `currency` временно остаётся и дублирует `original_currency`;
- `total_amount` временно остаётся как original amount для старого UI;
- новые агрегаты должны читать `*_base`, когда нужна сравнимая цена.

### 6.4. user_receipt_items

Добавить:

```sql
original_currency varchar(3)
base_currency varchar(3) DEFAULT 'RUB'
unit_price_original numeric
line_total_original numeric
unit_price_base numeric
line_total_base numeric
fx_rate_to_base numeric
fx_rate_date date
fx_source text
fx_quality text
```

Совместимость:

- `unit_price` и `line_total` остаются legacy-original для существующих экранов;
- новые статистические функции должны использовать `unit_price_base` / `line_total_base`.

### 6.5. user_prices

Критично для price stats и suspect price:

```sql
price_original numeric
original_currency varchar(3)
price_base numeric
base_currency varchar(3) DEFAULT 'RUB'
fx_rate_to_base numeric
fx_rate_date date
fx_source text
fx_quality text
```

Совместимость:

- `price` временно остаётся legacy-original;
- `get_wine_price_stats` и price validation переводятся на `price_base`;
- для старых RUB-цен backfill: `price_original = price`, `price_base = price`, `original_currency = 'RUB'`.

## 7. Сервис конвертации

### 7.1. Контракты

В Dart:

```dart
class CurrencyAmount {
  final double amount;
  final String currency;
}

class CurrencyConversionResult {
  final double amount;
  final String fromCurrency;
  final String toCurrency;
  final double rate;
  final DateTime rateDate;
  final String source;
  final String quality;
}
```

Сервисы:

- `CurrencyRateRepository`;
- `CurrencyConversionService`;
- `UserCurrencySettingsProvider`.

### 7.2. Правила

Если `fromCurrency == toCurrency`:

- `rate = 1`;
- `quality = exact_historical`;
- source = `identity`.

Если курса на дату покупки нет:

1. пробуем ближайший предыдущий день;
2. если нет, пробуем latest;
3. если нет, сохраняем original без base и помечаем `missing`, но не блокируем сохранение чека.

## 8. При сохранении BY-чека

Пример: пользователь сканирует BY-чек, строка вина `16.99 BYN`.

1. BY provider возвращает `currency = BYN`.
2. Receipt result несёт original amount `16.99 BYN`.
3. Перед сохранением receipt/items/prices вызываем conversion service:

```text
BYN -> RUB по purchase_date
```

4. В `user_receipts` пишем:

```text
total_amount_original = 22.56
original_currency = BYN
total_amount_base = 594.01
base_currency = RUB
fx_rate_to_base = 26.33
```

5. В `user_receipt_items` пишем:

```text
unit_price = 16.99              -- legacy original
unit_price_original = 16.99
original_currency = BYN
unit_price_base = 447.37
base_currency = RUB
```

6. В `user_prices` пишем:

```text
price = 16.99                   -- legacy original
price_original = 16.99
original_currency = BYN
price_base = 447.37
base_currency = RUB
```

7. Price validation сравнивает `price_base` с median `price_base`.

## 9. UI-правила

### 9.1. Обычные пользовательские поверхности

Показывают display currency:

- catalog / wine details price stats;
- cellar;
- map bottom sheet;
- receipt history summary;
- public wine price;
- analytics.

### 9.2. Детали чека

Показывают display currency как основную, original как provenance:

```text
Итого: 594 ₽
По чеку: 22.56 BYN
```

В раскрытии:

```text
Курс: 1 BYN = 26.33 RUB
Дата курса: 20.06.2026
Источник: НБРБ / cross-rate
```

### 9.3. Ручной ввод

Поле цены всегда имеет селектор валюты:

```text
[ 24.99 ] [ USD v ]
```

Default:

1. `preferred_manual_entry_currency`;
2. `preferred_display_currency`;
3. `RUB`.

## 10. Backend jobs

### 10.1. sync-exchange-rates

Supabase Edge Function или VPS/systemd job:

```text
sync-exchange-rates
```

Параметры:

- `date` optional, default today;
- `currencies` optional;
- `force` optional.

Обязанности:

1. загрузить курсы НБРБ;
2. сохранить `BYN -> RUB`, `RUB -> BYN`, `BYN -> USD`, `USD -> BYN`;
3. загрузить/проверить курсы ЦБ РФ для `RUB -> USD`, `USD -> RUB`, `BYN -> RUB`;
4. позже загрузить ЦБ Узбекистана для `UZS`;
5. вернуть diagnostics JSON.

### 10.2. backfill-exchange-rates

Отдельный скрипт/задача:

1. находит даты чеков без `*_base`;
2. подтягивает курсы на эти даты;
3. досчитывает `user_receipts`, `user_receipt_items`, `user_prices`;
4. не меняет original amounts.

## 11. Порядок реализации

### Этап 1. BYN foundation

1. Миграция:
   - `exchange_rates`;
   - currency fields в `profiles`;
   - original/base/fx fields в `user_receipts`, `user_receipt_items`, `user_prices`.
2. Edge Function или backend job `sync-exchange-rates` для НБРБ.
3. Dart currency service:
   - читает `exchange_rates`;
   - умеет identity conversion;
   - умеет fallback previous day/latest.
4. Receipt save path:
   - BYN original сохраняется;
   - RUB base считается и сохраняется.
5. `get_wine_price_stats` переводится на `price_base`.
6. Price validation переводится на `price_base`.

### Этап 2. Profile display currency

1. UI настройки валюты в профиле.
2. Provider выбранной display currency.
3. Форматтеры цены переводят base/original в display.
4. Receipt details показывают original provenance.

### Этап 3. Manual price currency

1. Селектор валюты в ручном вводе.
2. `preferred_manual_entry_currency`.
3. Конвертация ручной цены в base.

### Этап 4. UZS

1. Подключить ЦБ Узбекистана.
2. Добавить `UZS -> RUB`, `UZS -> USD`.
3. Прогнать UZ receipt flow по тому же контракту.

### Этап 5. USD/EUR public display

1. USD как полноценная display currency.
2. EUR по тем же правилам.
3. Винный индекс и расширенная аналитика.

## 12. Acceptance criteria для BY-first этапа

1. Белорусский чек сохраняет original values в `BYN`.
2. Белорусский чек сохраняет base values в `RUB`.
3. `user_prices.price_base` заполнен для BY receipt items.
4. `get_wine_price_stats` не смешивает `16.99 BYN` с `16.99 RUB`.
5. В обычном UI отображается только выбранная пользователем display currency.
6. В деталях чека доступна исходная цена в `BYN`.
7. Daily sync записывает свежие курсы в `exchange_rates`.
8. При отсутствии курса чек не теряется: сохраняется original, а base получает `fx_quality = missing` или досчитывается позже.

## 13. Важные запреты

- Не связывать валюту с языком интерфейса напрямую.
- Не показывать все валюты сразу в списках и карточках.
- Не перетирать original price после конвертации.
- Не использовать текущий курс для исторической покупки, если есть курс на дату покупки.
- Не считать suspect price по original price для разных валют.

## 14. Короткий промпт для следующей сессии

Продолжаем мультивалютность WinePool. BY receipt ingestion уже считаем рабочим: QR даёт УИ, дата вытаскивается OCR рядом с QR или выбирается пользователем, WebView provider аккуратно проходит Bitrix/reCAPTCHA, есть cooldown и локальный retry. Текущая задача - валютный слой: одна user-selected display currency в UI, original currency как факт покупки, base currency `RUB` для аналитики. Сначала реализовать BYN foundation: `exchange_rates`, daily sync НБРБ, original/base/fx поля в receipt tables/user_prices, перевод `get_wine_price_stats` и price validation на `price_base`.
