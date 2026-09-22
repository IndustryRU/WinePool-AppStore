# ТЗ: WinePool Cellar Pro — учет личного запаса вина, расположение бутылок и платный погреб

Дата: 23.06.2026  
Статус: feature source of truth для `Cellar Pro`; с 25.08.2026 реализация начинается только после подтверждения входного gate H4 в активном [post-1.1.0 roadmap](/R:/Flutter/Project/winepool_final/docs/post_release_1_1_0_execution_roadmap_2026_08_25.md).
Инициатор: поисковый спрос по запросам `винный дневник`, `программа для учета личного запаса вина`, `программа для учета и расположения вина`, `винный погребок`  
Связанные документы:

- [add_bottle_universal_flow_tz_2026_06_12.md](/R:/Flutter/Project/winepool_final/docs/add_bottle_universal_flow_tz_2026_06_12.md)
- [receipt_sprint_status.md](/R:/Flutter/Project/winepool_final/docs/receipt_sprint_status.md)
- [map_experience_roadmap_2026_04_25.md](/R:/Flutter/Project/winepool_final/docs/map_experience_roadmap_2026_04_25.md)
- [seo_aso_growth_plan_2026_05_23.md](/R:/Flutter/Project/winepool_final/docs/seo_aso_growth_plan_2026_05_23.md)
- [my_places_user_shops_tz_2026_06_13.md](/R:/Flutter/Project/winepool_final/docs/my_places_user_shops_tz_2026_06_13.md)
- [sommelier_experts_platform_tz_2026_06_19.md](/R:/Flutter/Project/winepool_final/docs/sommelier_experts_platform_tz_2026_06_19.md)

---

## 1. Короткое Резюме

WinePool уже имеет сильное ядро для личного винного погребка:

- `user_storage` хранит бутылки пользователя;
- `user_tastings` хранит дегустации;
- чековый flow переносит покупки в погребок;
- add-bottle flow добавляет бутылку без чека через штрихкод, этикетку или поиск;
- карта покупок показывает, где пользователь покупал вина;
- аналитика погребка уже строится на личных данных.

Новый слой `WinePool Cellar Pro` должен превратить текущую вкладку `Храню` из простого списка бутылок в полноценную систему личного учета:

```text
что есть дома -> где лежит -> когда куплено -> сколько стоит -> когда пить
             -> куда перемещено -> что открыто -> что осталось
```

Главный продуктовый принцип:

> Базовый погребок остается бесплатным и простым. Платный слой продает не "доступ к своим данным", а порядок, структуру, аналитику, экспорт, совместный доступ и работу с физическим расположением бутылок.

---

## 2. Почему Это Нужно

### 2.1. Поисковый Сигнал

Первые органические/поисковые запросы показывают спрос не только на "винный дневник", но и на утилитарные сценарии:

- `винный дневник`;
- `дневник вина`;
- `приложение для ведения дегустационных заметок`;
- `приложение для записи вин`;
- `программа для учета личного запаса вина`;
- `программа для учета и расположения вина`;
- `программа для учета алкоголя в личном винном погребе`;
- `винный погребок`.

Это важный сигнал: часть пользователей ищет не социальную сеть и не каталог, а инструмент, который заменяет таблицу, заметки, бумажный блокнот и память.

### 2.2. Рыночная Ниша

У зарубежных сервисов класса CellarTracker / InVintory / Vinotag сильные стороны:

- учет коллекции;
- места хранения;
- barcode / QR / NFC;
- drink windows;
- valuation;
- web / tablet интерфейсы;
- отчеты.

Но для российского пользователя у WinePool есть уникальное преимущество:

- чековый ввод через российские фискальные чеки;
- локальный каталог российских вин;
- ручное добавление бутылки через этикетку/штрихкод;
- личная карта покупок;
- модерация отсутствующих вин;
- локализация и юридически осторожное позиционирование.

Поэтому правильная стратегия не "копировать кастомный погреб на 20 000 бутылок", а постепенно построить массовый личный погребок с Pro-слоем для тех, у кого запас уже стал достаточно большим, чтобы болеть.

### 2.3. Продуктовая Роль В WinePool

`Cellar Pro` становится retention + monetization layer:

- пользователь возвращается, чтобы найти бутылку;
- открывает приложение перед ужином;
- отмечает открытие бутылки;
- получает напоминания по окну зрелости;
- видит стоимость и структуру коллекции;
- приглашает сомелье или близкого человека посмотреть погреб;
- при росте коллекции естественно переходит на платный тариф.

---

## 3. Текущий Baseline В Коде И Базе

### 3.1. Уже Есть

Клиент:

- `lib/features/cellar/` — вкладки `Пробовал`, `Храню`, `Аналитика`;
- `lib/features/cellar/domain/models.dart` — `UserStorageItem` с ценой, датой, винтажом, источником чека и магазином;
- `lib/features/cellar/application/cellar_controller.dart` — добавление, обновление, удаление, `drinkBottle`;
- `lib/features/add_bottle/` — универсальный flow добавления бутылки;
- `lib/features/wines/application/receipt_controller.dart` — receipt ingestion, draft flow, label suggestions;
- `lib/features/map/` — личная карта покупок.

База:

- `public.user_storage`;
- `public.user_tastings`;
- `public.user_receipts`;
- `public.user_receipt_items`;
- `public.draft_wines`;
- `public.draft_wine_sources`;
- `public.user_prices`;
- `get_user_storage()`;
- `add_to_user_storage(...)`;
- `update_user_storage_item(...)`;
- `delete_user_storage_item(...)`;
- `get_user_purchase_map()`;
- `get_user_purchase_map_wines(...)`.

Сильная текущая модель:

- `user_storage` уже хранит партии: `wine_id + quantity + vintage + purchase_price + purchase_date`;
- есть связь с чеком: `source_receipt_id`, `source_receipt_item_id`;
- есть магазин покупки: `source_shop_name`, `source_shop_address`, `source_shop_lat`, `source_shop_lng` — это география покупки/чека, не география хранения;
- есть валютный контекст: `purchase_price_original/base`, currency/fx fields;
- есть автодобавление после approve manual/draft источников.

### 3.2. Чего Не Хватает Для Запроса "Учет И Расположение Вина"

Текущий `user_storage.quantity` отвечает на вопрос:

```text
сколько таких бутылок есть у пользователя
```

Но не отвечает на вопросы:

```text
где лежит каждая бутылка?
какие 2 бутылки из 6 лежат в шкафу, а какие 4 в коробке?
какая бутылка была открыта?
что перемещалось?
какой стеллаж заполнен?
какие бутылки не распределены по местам?
можно ли распечатать QR-метки?
можно ли дать доступ сомелье только к погребу?
```

Для Pro-слоя нужна новая физическая модель хранения, но без ломки текущего простого погребка.

Важно: физическая модель хранения не должна превращаться в геокарту личной коллекции. `Где куплено` и `где лежит` — разные сущности:

- `где куплено / где дегустировали` может иметь координаты и отображаться на карте;
- `где лежит` — только приватная пользовательская структура внутри погребка: `Дом`, `Дача`, `Винный шкаф`, `Стеллаж A`, `Полка 2`, `Ячейка 12`;
- для хранения не сохраняем `lat/lng` и не показываем точки на публичной или личной геокарте;
- пользователь может назвать место как угодно, но WinePool не должен требовать точный адрес дома, дачи, виллы, гаража или офсайт-хранилища.

---

## 4. Целевое Позиционирование

### 4.1. Название Направления

Рабочее название:

```text
WinePool Cellar Pro
```

Русские формулировки в интерфейсе:

- `Погреб Pro`;
- `Зоны хранения`;
- `Схема погребка`;
- `Учет коллекции`;
- `Разместить бутылки`;
- `Переместить`;
- `Открыть бутылку`;
- `Журнал движения`;
- `Экспорт коллекции`.

### 4.2. Не Позиционировать Как

Не позиционировать как:

- складской учет алкоголя для торговли;
- B2B-склад;
- средство стимулирования покупки алкоголя;
- сервис доставки/продажи;
- инвестиционный советник;
- гарантия рыночной стоимости вина.

Правильная рамка:

```text
информационно-справочный сервис для личной коллекции 18+
```

### 4.3. Платность

Базовая логика:

- бесплатно: текущий погребок, дегустации, простое хранение, добавление бутылок;
- Pro: физические места, схема хранения, движение бутылок, напоминания, экспорт, совместный доступ, расширенная аналитика, QR/NFC-метки.

Важно: не отбирать уже существующий базовый функционал у пользователей.

---

## 5. Целевые Пользователи

### 5.1. Beginner / Casual

Коллекция: 3-20 бутылок.  
Проблема: помнить, что уже пробовал и что лежит дома.  
Нужно:

- быстро добавить бутылку;
- увидеть список;
- открыть бутылку и поставить оценку;
- не перегружать интерфейс полками и схемами.

Для него базовый погребок должен остаться легким.

### 5.2. Home Collector

Коллекция: 20-200 бутылок.  
Проблема: бутылки лежат в шкафу, коробках, холодильнике, на даче; уже сложно помнить расположение.  
Нужно:

- места хранения;
- фильтр `где лежит`;
- напоминания `пора открыть`;
- стоимость коллекции;
- экспорт;
- быстрый поиск по названию/винтажу/месту.

Это главный MVP-пользователь `Cellar Pro`.

### 5.3. Advanced Collector

Коллекция: 200+ бутылок.  
Проблема: нужна точность, движение, метки, контроль ячеек.  
Нужно:

- иерархия погребов;
- стеллажи/полки/ячейки;
- QR/NFC;
- журнал движения;
- бутылочные экземпляры;
- отчет по окнам зрелости и стоимости.

Это P2/P3, не первый релиз.

### 5.4. Sommelier / Consultant

Не владелец коллекции, а помощник пользователя.  
Нужно:

- получить snapshot погреба;
- дать рекомендацию;
- не видеть лишние приватные заметки без разрешения.

Связь с `Живые сомелье`: Cellar Pro должен дать структурированный контекст для консультации.

---

## 6. Продуктовый Scope

### 6.1. P0: Market Validation / Fake Door

Цель: проверить спрос до тяжёлой реализации.

Входит:

1. Обновить `/wine-cellar` и/или главную:
   - блок `Учет расположения бутылок`;
   - CTA `Хочу Погреб Pro`;
   - форма раннего доступа с `source=cellar_pro`.
2. Добавить в приложении мягкую карточку в пустом/заполненном погребке:
   - `Скоро: зоны хранения и схема погребка`;
   - CTA `Сообщить, когда появится`.
3. Собрать события:
   - `cellar_pro_teaser_viewed`;
   - `cellar_pro_interest_clicked`;
   - `cellar_pro_waitlist_joined`.

Не входит:

- paywall;
- SQL физической модели;
- изменение `user_storage`.

На P0 собираем контакт и подтверждение интереса через ранний доступ.

### 6.2. P1: Быстрый MVP "Где Лежит"

Цель: закрыть поисковый интент `программа для учета и расположения вина` самым коротким рабочим срезом.

Входит:

1. Один или несколько пользовательских погребов/мест:
   - `Дом`;
   - `Винный шкаф`;
   - `Холодильник`;
   - `Дача`;
   - пользовательское название.
   Эти значения являются пользовательскими ярлыками, а не географическими точками. В P1 не хранить координаты и не требовать точный адрес места хранения.
2. Простая иерархия:
   - место;
   - зона/стеллаж;
   - полка/ячейка.
3. Распределение количества из `user_storage` по местам:
   - 3 бутылки на `Шкаф / Полка 1`;
   - 2 бутылки на `Коробка / Дача`;
   - 1 бутылка без места.
4. Фильтр и поиск в `Храню`:
   - все;
   - без места;
   - конкретное место;
   - пить сейчас;
   - винтаж/страна/сорт.
5. Действия:
   - `Разместить`;
   - `Переместить`;
   - `Открыть`;
   - `Списать`;
   - `Добавить заметку`.
6. Журнал движения:
   - добавлено;
   - размещено;
   - перемещено;
   - открыто;
   - списано.

Не входит:

- отдельная строка на каждую бутылку;
- QR/NFC;
- визуальная 3D-схема;
- сложный desktop UI;
- платёжная инфраструктура.

### 6.3. P2: Cellar Pro

Цель: сделать платный слой достаточно ценным для коллекционеров.

Входит:

1. Расширенная схема мест:
   - несколько погребов;
   - стеллажи;
   - полки;
   - ячейки;
   - вместимость.
2. Dashboard:
   - всего бутылок;
   - оценочная стоимость;
   - без места;
   - в окне зрелости;
   - стареет / пить скоро;
   - топ регионов/стилей/винтажей.
3. Напоминания:
   - `вино входит в окно зрелости`;
   - `давно не открывали ничего из этой зоны`;
   - `есть бутылки без места`;
   - `проверьте коллекцию`.
4. Экспорт:
   - CSV;
   - PDF;
   - "инвентаризационный список";
   - "карта расположения".
5. Совместный доступ:
   - read-only;
   - can edit locations;
   - can open/write tasting;
   - temporary expert access.
6. Web/tablet layout:
   - широкая таблица;
   - фильтры слева;
   - детали справа;
   - быстрые массовые операции.

### 6.4. P3: Exact Bottle / QR / NFC

Цель: поддержать большие коллекции, где партии уже недостаточно.

Входит:

1. Бутылочные экземпляры:
   - уникальный `bottle_instance`;
   - QR-code;
   - optional NFC tag;
   - статус `in_storage/opened/consumed/gifted/lost`;
   - точное место.
2. Печать / генерация меток:
   - QR на лист A4;
   - export PNG/PDF;
   - настройка размера.
3. Быстрое сканирование:
   - scan QR -> открыть карточку конкретной бутылки;
   - scan -> переместить;
   - scan -> открыть/списать.
4. Инвентаризация:
   - режим `Проверка полки`;
   - отсканировал все бутылки в ячейке;
   - подсветка расхождений.

Не делать раньше P2: это дорогой слой, который нужен только пользователям с реальной болью.

---

## 7. UX Требования

### 7.1. Основной Принцип UX

Не превращать базовый погребок в складскую систему для всех.

Вкладка `Храню` должна оставаться понятной:

- текущий список бутылок;
- статистика;
- быстрые действия.

Pro-слой появляется прогрессивно:

- если у бутылки нет места, показываем мягкий CTA `Указать место`;
- если мест нет, показываем `Создать место хранения`;
- если пользователь не хочет, список работает как раньше.

### 7.2. Навигация

Текущий `MyCellarScreen` имеет 3 вкладки:

- `Пробовал`;
- `Храню`;
- `Аналитика`.

В P1 не добавлять четвертую вкладку, чтобы не ломать IA.

Изменения:

- во вкладке `Храню` добавить фильтр-чипы:
  - `Все`;
  - `Без места`;
  - `Пить сейчас`;
  - `Зоны`;
- добавить action в app bar / overflow:
  - `Зоны хранения`;
  - `Экспорт` (если Pro);
  - `Инвентаризация` (P3).

Отдельный экран:

```text
/my-cellar/locations
```

или query/modal внутри `/my-cellar?tab=1&panel=locations` — решить при реализации с учетом GoRouter.

### 7.3. Stored Wine Card

Карточка вина во вкладке `Храню` должна показать:

- название;
- винодельня;
- винтаж;
- количество;
- средняя/общая цена;
- место:
  - если одно: `Шкаф / Полка 2`;
  - если несколько: `3 места`;
  - если нет: `Без места`;
- окно зрелости:
  - `Пить сейчас`;
  - `До 2028`;
  - `Окно не указано`.

Действия:

- `Разместить`;
- `Переместить`;
- `Открыть`;
- `Дегустация`;
- `Подробнее`.

### 7.4. Размещение Бутылок

Flow:

1. Пользователь нажимает `Разместить`.
2. Sheet показывает:
   - текущая партия;
   - всего бутылок;
   - уже размещено;
   - без места;
   - выбор места;
   - количество.
3. Если места нет:
   - inline `Создать место`;
   - минимум: название + тип.
4. Save:
   - создаёт/обновляет `cellar_stock_lot`;
   - пишет событие в `cellar_inventory_events`.

### 7.5. Открыть Бутылку

Текущий `drinkBottle(UserStorageItem item)` просто уменьшает `quantity` или удаляет запись.

Новый flow:

1. Пользователь нажимает `Открыть`.
2. Если у партии одно место:
   - по умолчанию списываем оттуда.
3. Если несколько мест:
   - спросить `Откуда открыли?`.
4. Действия:
   - уменьшить `cellar_stock_lot.quantity`;
   - уменьшить `user_storage.quantity`;
   - создать `cellar_inventory_events.event_type = opened`;
   - предложить `Добавить дегустацию`;
   - prefill tasting purchase context.

Текст должен быть нейтральный: `Открыть бутылку`, `Списать из погребка`, не `выпить больше`.

### 7.6. Location Manager

Экран `Зоны хранения`:

- список мест верхнего уровня;
- счетчик бутылок;
- вместимость;
- сколько без места;
- поиск по коду/названию;
- создание/редактирование.

Типы:

- `Погреб`;
- `Винный шкаф`;
- `Холодильник`;
- `Стеллаж`;
- `Полка`;
- `Коробка`;
- `Ячейка`;
- `Другое`.

Иерархия в P1 может быть простой:

```text
Дом
  Винный шкаф
    Полка 1
    Полка 2
  Коробка
Дача
```

В P2 можно добавить capacity grid.

### 7.7. Tablet / Desktop

P1 может быть mobile-first.

Но архитектура UI должна не мешать широкому layout:

- список мест слева;
- таблица бутылок в центре;
- детали выбранной партии справа;
- bulk actions в toolbar.

Flutter уже multi-platform; не создавать отдельный desktop project. Достаточно адаптивного layout внутри `features/cellar`.

---

## 8. Data Model

### 8.1. Ключевое Архитектурное Решение

Не ломаем `user_storage`.

`user_storage` остается агрегированной партией и совместимостью для:

- текущего UI;
- аналитики;
- карты;
- reviews/tastings context;
- wine merge;
- receipt/draft backfill.

Физическое расположение добавляется отдельным слоем:

```text
user_storage (партия)
  -> cellar_stock_lots (части партии по местам)
       -> cellar_locations (иерархия мест)
```

Это позволяет:

- не создавать строку на каждую бутылку в P1;
- распределить 6 бутылок по 2-3 местам;
- держать старый `quantity` как общий источник;
- позже добавить exact bottle instances без миграции всего погреба.

### 8.2. Таблица `user_cellars`

```sql
CREATE TABLE public.user_cellars (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  name text NOT NULL,
  kind text NOT NULL DEFAULT 'home',
  description text,
  target_temperature_min numeric,
  target_temperature_max numeric,
  target_humidity_min numeric,
  target_humidity_max numeric,
  sort_order integer NOT NULL DEFAULT 0,
  is_archived boolean NOT NULL DEFAULT false,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  CONSTRAINT user_cellars_kind_check CHECK (
    kind IN ('home', 'wine_fridge', 'cabinet', 'cellar', 'offsite', 'other')
  )
);
```

RLS:

- owner select/insert/update/delete;
- shared access через отдельную таблицу в P2.

Индексы:

```sql
CREATE INDEX idx_user_cellars_user ON public.user_cellars(user_id, is_archived, sort_order);
```

### 8.3. Таблица `cellar_locations`

Иерархическое дерево мест внутри погреба.

```sql
CREATE TABLE public.cellar_locations (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  cellar_id uuid NOT NULL REFERENCES public.user_cellars(id) ON DELETE CASCADE,
  parent_location_id uuid REFERENCES public.cellar_locations(id) ON DELETE CASCADE,
  location_type text NOT NULL DEFAULT 'shelf',
  code text,
  name text NOT NULL,
  description text,
  capacity_bottles integer,
  sort_order integer NOT NULL DEFAULT 0,
  is_archived boolean NOT NULL DEFAULT false,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  CONSTRAINT cellar_locations_type_check CHECK (
    location_type IN ('cellar', 'zone', 'rack', 'shelf', 'row', 'slot', 'box', 'fridge', 'other')
  ),
  CONSTRAINT cellar_locations_capacity_check CHECK (
    capacity_bottles IS NULL OR capacity_bottles >= 0
  )
);
```

Важно:

- `user_id` денормализован для простого RLS и быстрых запросов.
- `code` может быть `A-01`, `R2-S3`, `Box-4`.
- В P1 `code` не обязан быть уникальным глобально, но желательно сделать уникальным в рамках `cellar_id` для неархивных мест:

```sql
CREATE UNIQUE INDEX idx_cellar_locations_cellar_code_active
ON public.cellar_locations(cellar_id, lower(code))
WHERE code IS NOT NULL AND is_archived = false;
```

### 8.4. Таблица `cellar_stock_lots`

Распределение количества из `user_storage` по местам.

```sql
CREATE TABLE public.cellar_stock_lots (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  storage_item_id uuid NOT NULL REFERENCES public.user_storage(id) ON DELETE CASCADE,
  location_id uuid REFERENCES public.cellar_locations(id) ON DELETE SET NULL,
  quantity integer NOT NULL,
  note text,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  CONSTRAINT cellar_stock_lots_quantity_check CHECK (quantity > 0)
);
```

Ограничение целостности:

```text
sum(cellar_stock_lots.quantity where storage_item_id = X) <= user_storage.quantity
```

Postgres CHECK это напрямую не выразит. Нужна RPC/trigger защита:

- все изменения lots делать через RPC;
- RPC проверяет сумму;
- прямые writes закрыть RLS/policies или не выдавать grants на insert/update/delete без RPC.

### 8.5. Таблица `cellar_inventory_events`

Журнал движения и аудита.

```sql
CREATE TABLE public.cellar_inventory_events (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  storage_item_id uuid REFERENCES public.user_storage(id) ON DELETE SET NULL,
  stock_lot_id uuid REFERENCES public.cellar_stock_lots(id) ON DELETE SET NULL,
  from_location_id uuid REFERENCES public.cellar_locations(id) ON DELETE SET NULL,
  to_location_id uuid REFERENCES public.cellar_locations(id) ON DELETE SET NULL,
  event_type text NOT NULL,
  quantity integer NOT NULL DEFAULT 1,
  reason text,
  metadata jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at timestamptz NOT NULL DEFAULT now(),
  CONSTRAINT cellar_inventory_events_type_check CHECK (
    event_type IN (
      'created',
      'placed',
      'moved',
      'opened',
      'consumed',
      'gifted',
      'lost',
      'adjusted',
      'deleted'
    )
  ),
  CONSTRAINT cellar_inventory_events_quantity_check CHECK (quantity > 0)
);
```

События не являются источником истины для количества в P1, но дают:

- audit;
- историю;
- будущую инвентаризацию;
- отчеты.

### 8.6. P3 Таблицы Для Exact Bottle

Не создавать в P1.

Будущий слой:

```sql
CREATE TABLE public.cellar_bottle_instances (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  storage_item_id uuid NOT NULL REFERENCES public.user_storage(id) ON DELETE CASCADE,
  stock_lot_id uuid REFERENCES public.cellar_stock_lots(id) ON DELETE SET NULL,
  location_id uuid REFERENCES public.cellar_locations(id) ON DELETE SET NULL,
  bottle_code text,
  qr_token text,
  nfc_uid text,
  status text NOT NULL DEFAULT 'in_storage',
  opened_at timestamptz,
  consumed_at timestamptz,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);
```

P3 миграция должна иметь backfill только для пользователей, которые включили exact-mode. Нельзя автоматически создавать тысячи строк для всех.

### 8.7. Sharing / Collaborators

P2:

```sql
CREATE TABLE public.cellar_collaborators (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  cellar_id uuid NOT NULL REFERENCES public.user_cellars(id) ON DELETE CASCADE,
  owner_user_id uuid NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  collaborator_user_id uuid REFERENCES public.profiles(id) ON DELETE CASCADE,
  invite_email text,
  role text NOT NULL DEFAULT 'viewer',
  status text NOT NULL DEFAULT 'pending',
  expires_at timestamptz,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  CONSTRAINT cellar_collaborators_role_check CHECK (
    role IN ('viewer', 'location_manager', 'tasting_editor', 'admin')
  ),
  CONSTRAINT cellar_collaborators_status_check CHECK (
    status IN ('pending', 'active', 'revoked', 'expired')
  )
);
```

P1 не делать.

---

## 9. RPC / API Contract

Все write операции P1 делать через RPC, чтобы не получить рассинхрон между `user_storage.quantity` и `cellar_stock_lots`.

### 9.1. `get_user_cellars()`

Возвращает:

- cellars;
- count locations;
- count bottles placed;
- count unplaced;
- capacity summary.

### 9.2. `create_user_cellar(...)`

Параметры:

- name;
- kind;
- description;
- optional temperature/humidity targets.

Поведение:

- создает cellar;
- если это первый cellar пользователя, можно создать default location `Без места` нельзя: unplaced должен оставаться вычисляемым, а не псевдо-местом.

### 9.3. `upsert_cellar_location(...)`

Создает/редактирует location.

Проверки:

- owner;
- parent belongs to same cellar and same user;
- capacity >= current placed count;
- no cycles.

### 9.4. `archive_cellar_location(p_location_id, p_move_to_location_id default null)`

Архивирует location.

Правила:

- если внутри есть stock lots:
  - либо `p_move_to_location_id` обязателен;
  - либо location нельзя архивировать;
  - UX должен предложить перемещение.

### 9.5. `place_storage_item(...)`

```sql
place_storage_item(
  p_storage_item_id uuid,
  p_location_id uuid,
  p_quantity integer,
  p_note text default null
)
```

Поведение:

- проверяет owner;
- проверяет, что `p_quantity` <= unplaced quantity;
- если lot для same `storage_item_id + location_id` уже есть, увеличивает его;
- пишет event `placed`.

### 9.6. `move_cellar_stock(...)`

```sql
move_cellar_stock(
  p_storage_item_id uuid,
  p_from_location_id uuid,
  p_to_location_id uuid,
  p_quantity integer,
  p_note text default null
)
```

Поведение:

- уменьшает from lot;
- увеличивает/создает to lot;
- пишет event `moved`.

### 9.7. `open_cellar_bottle(...)`

```sql
open_cellar_bottle(
  p_storage_item_id uuid,
  p_location_id uuid default null,
  p_quantity integer default 1,
  p_create_tasting_draft boolean default false
)
```

Поведение:

- уменьшает `user_storage.quantity`;
- уменьшает lot, если location задан;
- если quantity в `user_storage` стала 0, не обязательно удалять строку физически: текущий код удаляет через `delete_user_storage_item`. Решить в реализации:
  - либо сохранить старое поведение;
  - либо добавить soft status `depleted` отдельной миграцией.
- пишет event `opened`;
- возвращает payload для prefill tasting.

Важно: чтобы не сломать текущий UI, P1 может продолжать удалять empty storage item, но events сохранятся со `storage_item_id SET NULL`.

### 9.8. `get_storage_item_location_breakdown(p_storage_item_id uuid)`

Возвращает:

- lots;
- unplaced quantity;
- location breadcrumbs.

### 9.9. `get_cellar_inventory_events(...)`

Фильтры:

- date range;
- event_type;
- storage_item_id;
- location_id.

### 9.10. `export_cellar_inventory(...)`

P2.

Возвращает signed download URL или JSON payload для client-side CSV.

Форматы:

- CSV;
- PDF later.

---

## 10. Flutter Architecture

### 10.1. Модуль

Не создавать отдельный top-level feature, если можно расширить текущий `features/cellar`.

Рекомендуемая структура:

```text
lib/features/cellar/
  domain/
    models.dart                 // current
    cellar_location.dart         // new
    cellar_stock_lot.dart        // new
    cellar_inventory_event.dart  // new
    cellar_pro_entitlement.dart  // later
  data/
    cellar_repository.dart       // current + RPC methods
    cellar_locations_repository.dart
  application/
    cellar_controller.dart       // current aggregate writes
    cellar_locations_controller.dart
    cellar_inventory_controller.dart
  presentation/
    my_cellar_screen.dart        // integrate chips/actions
    cellar_locations_screen.dart
    cellar_location_details_screen.dart
    widgets/
      cellar_location_picker_sheet.dart
      cellar_place_bottles_sheet.dart
      cellar_move_stock_sheet.dart
      cellar_open_bottle_sheet.dart
      cellar_location_badge.dart
```

Если файл `my_cellar_screen.dart` продолжит расти, вынести Stored tab в отдельный файл:

```text
presentation/stored_wines_view.dart
```

### 10.2. State Management

Riverpod providers:

- `userCellarsProvider`;
- `cellarLocationsProvider(cellarId)`;
- `storageLocationBreakdownProvider(storageItemId)`;
- `cellarInventoryEventsProvider(filter)`;
- `cellarProEntitlementProvider` (P2/P3).

Invalidation:

- после `place/move/open` инвалидировать:
  - `cellarStorageProvider`;
  - `analyticsProvider`;
  - `storageLocationBreakdownProvider(storageItemId)`;
  - `userCellarsProvider`;
  - relevant map only if purchase data changed (обычно нет).

### 10.3. Add Bottle Integration

`candidate_confirm_sheet` должен получить optional поле:

- `Место хранения`;
- default empty;
- если пользователь выбрал место, после `addToStorage` вызвать `place_storage_item` для returned storage id.

Текущий `addToUserStorage` в Dart возвращает `void`, хотя SQL RPC возвращает `uuid`. Нужно изменить:

- `CellarRepository.addToUserStorage` -> `Future<String?>`;
- `CellarController.addToStorage` -> `Future<String?>`;
- все существующие callers могут игнорировать результат.

Это изменение важно для P1, иначе нельзя надежно сразу разместить новую бутылку.

### 10.4. Receipt Integration

После bulk-add из receipt review:

P1 safe option:

- после сохранения чека показать sheet:
  - `Разместить добавленные бутылки?`;
  - `Все в одно место`;
  - `Позже`.

Не пытаться заставить пользователя назначать место для каждой строки во время review чека. Receipt review уже сложный.

P2:

- batch placement by group;
- правила по умолчанию:
  - `вина из этого магазина -> временная зона`;
  - `новые бутылки без места -> напоминание`.

### 10.5. Tasting Integration

При `Открыть бутылку`:

- предлагать `Добавить дегустацию`;
- передавать:
  - wine;
  - storageItemId;
  - purchasePrice;
  - purchaseDate;
  - sourceReceiptId;
  - sourceShopName;
  - vintage.

Существующий route `/wine/:id/add-tasting` уже поддерживает часть этого контекста.

### 10.6. Map Integration

Карта покупок не является картой хранения.

Не смешивать:

- `где куплено` = purchase map;
- `где лежит` = cellar locations.

UI может показывать оба факта в карточке:

```text
Куплено: ВинЛаб, Москва
Лежит: Дом / Шкаф / Полка 2
```

Но карта в `features/map` должна оставаться purchase map.

### 10.7. Sommelier Integration

P2:

- из `Погреб Pro` CTA `Разобрать с сомелье`;
- пользователь выбирает:
  - весь погреб;
  - одну зону;
  - selected bottles;
  - include notes yes/no;
  - include prices yes/no.
- request payload в эксперты:
  - `context_type = cellar`;
  - `storage_item_ids`;
  - `location_ids`;
  - privacy flags.

---

## 11. Entitlements / Paywall

### 11.1. Принцип

Не блокировать базовый список `Храню`.

Варианты gating:

Free:

- список хранения;
- дегустации;
- аналитика базовая;
- добавление через чек/штрихкод/этикетку;
- одно простое место хранения или ограниченное количество размещенных бутылок (решить после fake-door).

Pro:

- несколько мест хранения;
- иерархия;
- перемещение;
- журнал;
- напоминания;
- экспорт;
- совместный доступ;
- tablet/desktop advanced layout;
- QR labels.

### 11.2. Техническая Модель Entitlement

Не реализовывать платежи в P1.

Для P1 достаточно feature flag:

```text
AppFlags.cellarProEnabled
```

Для P2:

```sql
CREATE TABLE public.user_entitlements (
  user_id uuid NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  feature_key text NOT NULL,
  status text NOT NULL,
  source text,
  starts_at timestamptz,
  expires_at timestamptz,
  metadata jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  PRIMARY KEY (user_id, feature_key)
);
```

`feature_key`:

- `cellar_pro`;
- later `experts`;
- later `tourism_partner_tools`.

`status`:

- `trial`;
- `active`;
- `expired`;
- `revoked`.

Payment provider decision отдельно. До legal/store review не привязывать ТЗ к конкретному провайдеру.

### 11.3. Pricing Hypothesis

Не фиксировать в коде.

Рабочие гипотезы:

- `199-399 ₽/мес`;
- `1490-2990 ₽/год`;
- early adopter lifetime/annual discount.

Проверять через fake-door/waitlist.

---

## 12. Analytics

### 12.1. Product Events

P0/P1:

```text
cellar_pro_teaser_viewed {entry}
cellar_pro_interest_clicked {entry}
cellar_location_created {kind, depth}
cellar_storage_placed {storage_item_id, quantity, location_depth}
cellar_stock_moved {quantity, from_depth, to_depth}
cellar_bottle_opened {with_location: bool, prompted_tasting: bool}
cellar_location_filter_used {filter_type}
cellar_unplaced_filter_used
```

P2:

```text
cellar_export_started {format}
cellar_export_completed {format}
cellar_pro_paywall_viewed {entry}
cellar_pro_trial_started
cellar_collaborator_invited {role}
cellar_expert_snapshot_started
```

### 12.2. North Star Metrics

Для P1:

- % active users with >=1 storage item and >=1 location;
- median time from bottle add to location assignment;
- % bottles without location;
- open bottle flow completion;
- add bottle -> placed conversion.

Для Pro:

- teaser click rate;
- waitlist conversion;
- trial start;
- Pro activation: user created 2+ locations and placed 10+ bottles;
- monthly active collectors;
- export usage;
- location search/filter usage.

---

## 13. SEO / Landing Requirements

### 13.1. Новые Кластеры

Добавить в SEO план:

- `программа для учета личного запаса вина`;
- `программа для учета и расположения вина`;
- `учет винной коллекции`;
- `учет домашнего винного погреба`;
- `приложение для учета вина`;
- `винная коллекция приложение`;
- `где хранится вино приложение`.

### 13.2. Лендинг `/wine-cellar`

Добавить блок:

```text
Учет расположения вина

Если бутылок стало больше, чем полок в памяти, WinePool поможет отмечать,
где лежит каждая партия: шкаф, полка, коробка, дача или винный холодильник.
```

CTA:

- `Хочу учет мест хранения`;
- `Сообщить о запуске Погреб Pro`.

### 13.3. Отдельная Страница Позже

Если запросы начнут расти:

```text
/wine-cellar-pro
```

Но на P0 лучше усилить текущую `/wine-cellar`, чтобы не распылять индекс.

---

## 14. Legal / Privacy / Store Guardrails

### 14.1. Alcohol-Safe Copy

Использовать:

- `Открыть бутылку`;
- `Списать из погребка`;
- `Отметить дегустацию`;
- `Пора проверить окно зрелости`;
- `Информационный учет коллекции`;
- `Личная история`.

Избегать:

- `пейте чаще`;
- `купите еще`;
- `успейте выпить`;
- `выгодно инвестировать`;
- `гарантированная стоимость`.

### 14.2. Personal Data

В Pro появляются новые данные:

- приватные ярлыки зон хранения: `Дом`, `Дача`, `Винный шкаф`, `Стеллаж A`, `Полка 2`, `Ячейка 12`;
- стоимость коллекции;
- collaborator access.

Privacy requirements:

- storage location names private by default;
- не хранить `lat/lng` для мест хранения;
- не требовать и не поощрять ввод точного адреса дома, дачи, виллы, гаража или офсайт-хранилища;
- не отображать места хранения на карте;
- карта WinePool показывает места покупки/дегустации из чеков и пользовательских отметок, а не места хранения коллекции;
- не показывать зоны хранения в публичных отзывах, карточках вина, community-карте или SEO-поверхностях;
- не передавать сомелье без явного выбора;
- в export предупреждать, что файл содержит личные данные;
- collaborator roles должны быть revocable.

### 14.3. Security

RLS must enforce:

- владелец видит свой cellar;
- collaborator видит только active invited cellars и только allowed fields;
- service role/admin не получает consumer cellar browsing UI без operational причины.

В P1 collaborators отсутствуют, значит RLS проще: owner only.

---

## 15. Implementation Plan

### Проход 0. Product Validation + Docs + SEO

Цель: проверить спрос и подготовить поверхность.

Сделать:

1. Обновить `docs/documentation_map.md`.
2. Обновить `docs/prioritized_work_backlog.md`.
3. Обновить `docs/seo_aso_growth_plan_2026_05_23.md`.
4. Обновить `/wine-cellar`:
   - блок `Учет расположения`;
   - waitlist CTA.
5. Добавить AppMetrica events для teaser.
6. Добавить in-app teaser в `MyCellarScreen`.

Проверка:

- HTML валиден;
- ссылки работают;
- события уходят;
- тексты legal-safe.

### Проход 1. Backend Foundation For Locations

Сделать миграцию:

- `user_cellars`;
- `cellar_locations`;
- `cellar_stock_lots`;
- `cellar_inventory_events`;
- indexes;
- RLS owner-only;
- RPC:
  - `get_user_cellars`;
  - `create_user_cellar`;
  - `upsert_cellar_location`;
  - `place_storage_item`;
  - `move_cellar_stock`;
  - `get_storage_item_location_breakdown`.

Важно:

- не менять `user_storage` structure;
- не менять existing RPC behavior;
- direct writes к lots закрыть или не использовать в клиенте.

Проверка:

- SQL compile;
- RLS owner isolation;
- place more than unplaced quantity fails;
- moving unavailable quantity fails;
- archiving non-empty location fails or requires move.

### Проход 2. Flutter Location Manager

Сделать:

- domain models;
- repository/controller;
- `CellarLocationsScreen`;
- create/edit/archive location;
- breadcrumbs;
- empty state;
- picker sheet.

Проверка:

- flutter analyze;
- create/edit/archive e2e;
- localization ru/en/be/uz where needed;
- no regressions in existing cellar tabs.

### Проход 3. Stored Tab Integration

Сделать:

- location badge on storage cards;
- filters `Все / Без места / Зоны`;
- `Разместить`;
- `Переместить`;
- location breakdown sheet;
- invalidations.

Проверка:

- 1 storage item, no locations;
- 1 storage item, one location;
- 1 storage item split across 2 locations;
- unplaced quantity calculation;
- update/delete storage item behavior.

### Проход 4. Open Bottle Flow

Сделать:

- replace direct `drinkBottle` UI path with sheet;
- choose location when split;
- decrement lot + user_storage through RPC;
- event `opened`;
- CTA `Добавить дегустацию`.

Проверка:

- open from single location;
- open from split locations;
- open unplaced;
- final bottle removes/depletes correctly;
- tasting prefill works.

### Проход 5. Add Bottle / Receipt Placement

Сделать:

- `addToStorage` returns storage id;
- confirm sheet can select location;
- after receipt bulk add, optional `Разместить все`;
- manual draft approve storage backfill remains compatible.

Проверка:

- barcode add -> choose location -> appears placed;
- label add -> choose location;
- receipt add -> later placement;
- approved draft -> unplaced until assigned.

### Проход 6. Pro Analytics / Reminders / Export

Сделать:

- dashboard cards;
- drink window reminders using `ideal_drink_from/to`;
- export CSV;
- Pro teaser/paywall behind feature flag.

Проверка:

- no alcohol-promotional copy;
- export includes location and unplaced;
- reminders can be disabled.

### Проход 7. Tablet/Desktop Layout

Сделать:

- responsive breakpoint;
- side panel locations;
- table/list hybrid;
- detail panel.

Проверка:

- desktop web;
- tablet width;
- mobile unchanged.

### Проход 8. Exact Bottle / QR/NFC

Only after real demand.

Сделать:

- exact-mode opt-in;
- bottle instances;
- QR token;
- label export;
- scan -> bottle detail;
- inventory mode.

---

## 16. Acceptance Criteria

P1 считается готовым, если:

- пользователь может создать место хранения;
- пользователь может назначить часть бутылок из `user_storage` в место;
- одна партия может быть разделена между несколькими местами;
- список `Храню` показывает место/без места;
- фильтр `Без места` работает;
- перемещение между местами работает;
- открытие бутылки корректно списывает из выбранного места;
- невозможно разместить больше бутылок, чем есть в партии;
- невозможно списать больше бутылок, чем есть в месте;
- журнал движения пишет основные события;
- существующий receipt/add-bottle/tasting flow не регрессировал;
- базовый пользователь может игнорировать Pro-функции и продолжать пользоваться старым списком.

P2 считается готовым, если:

- пользователь понимает ценность Pro без ощущения, что у него забрали базовый погребок;
- экспорт работает;
- напоминания не звучат как стимулирование употребления;
- расширенная аналитика корректно работает с базовой валютой;
- есть feature flag / entitlement boundary.

---

## 17. QA Plan

### 17.1. Backend

- RLS: user A не видит cellars/locations/lots/events user B.
- `place_storage_item`:
  - quantity 1;
  - quantity > unplaced fails;
  - invalid storage item fails;
  - invalid location owner fails.
- `move_cellar_stock`:
  - full move;
  - partial move;
  - from empty fails;
  - same source/target no-op or fails gracefully.
- `open_cellar_bottle`:
  - unplaced;
  - one location;
  - split locations;
  - last bottle.
- migration rollback strategy documented.

### 17.2. Flutter

- empty cellar;
- existing storage without locations;
- create first location;
- place bottles;
- split bottles;
- filter by location;
- open bottle;
- add tasting after open;
- add bottle with location;
- receipt add then place later;
- offline/network error states;
- localization length on mobile.

### 17.3. Regression

- receipt QR scan;
- pending receipt review;
- add bottle barcode;
- add bottle label;
- draft queue;
- approve draft -> user_storage;
- wine details -> add tasting;
- map purchase points;
- wine merge moves `user_storage.wine_id` and does not touch location lots incorrectly.

---

## 18. Risks

### 18.1. Scope Creep

Риск: сразу уйти в NFC/3D/desktop и не выпустить полезный P1.

Митигация:

- P1 только locations + lots + movement;
- exact bottles и NFC только после demand.

### 18.2. Quantity Desync

Риск: `user_storage.quantity` и `cellar_stock_lots` расходятся.

Митигация:

- writes only via RPC;
- server-side sum checks;
- periodic diagnostic query;
- UI shows unplaced as computed.

### 18.3. UI Overload

Риск: casual users испугаются складского интерфейса.

Митигация:

- progressive disclosure;
- Pro controls hidden behind `Зоны`;
- old list remains default.

### 18.4. Legal / Store Copy

Риск: напоминания могут выглядеть как стимулирование употребления.

Митигация:

- wording review;
- focus on учет/окно зрелости/информацию;
- user controls for reminders.

### 18.5. Privacy

Риск: пользовательские названия зон хранения раскрывают личную информацию о доме, даче, вилле, гараже или офсайт-хранилище.

Митигация:

- private by default;
- no public surface;
- no map surface;
- no `lat/lng` for storage;
- clear copy distinction: `место покупки` vs `зона хранения`;
- explicit share/export warnings.

---

## 19. Open Questions

Решить до прохода 1:

1. Нужно ли в Free разрешить одно место хранения или весь P1 держать за feature flag до Pro?
2. Как назвать тариф в UI: `Погреб Pro`, `Cellar Pro`, `Коллекция Pro`?
3. Нужно ли добавлять `storage_location_label` прямо в `user_storage` как быстрый fallback или сразу идти через `cellar_stock_lots`?
   - Рекомендация: сразу `cellar_stock_lots`, потому что split по местам иначе станет переделкой.
4. При `quantity = 0` удаляем `user_storage` как сейчас или вводим depletion history?
   - Рекомендация P1: сохранить старое поведение, events дадут историю; soft depleted — отдельный slice.
5. Нужно ли импортировать старые `ideal_drink_from/to` в Pro reminders сразу?
   - Рекомендация: да, но без push на P1.
6. Нужен ли отдельный `/wine-cellar-pro` лендинг?
   - Рекомендация: сначала усилить `/wine-cellar`.

---

## 20. Prompt Для Следующей Сессии Реализации P0/P1

```text
Продолжаем WinePool. Нужно начать направление Cellar Pro по ТЗ:
docs/wine_cellar_pro_tz_2026_06_23.md.

Перед работой прочитай:
- docs/wine_cellar_pro_tz_2026_06_23.md
- docs/add_bottle_universal_flow_tz_2026_06_12.md
- docs/receipt_sprint_status.md
- lib/features/cellar/domain/models.dart
- lib/features/cellar/data/cellar_repository.dart
- lib/features/cellar/application/cellar_controller.dart
- lib/features/cellar/presentation/my_cellar_screen.dart
- supabase/migrations/20260613_add_manual_shop_to_user_storage.sql
- supabase/migrations/20260620_add_manual_draft_currency_context.sql

Если задача P0:
1. Обнови landing/wine-cellar.html: блок "Учет расположения вина" + CTA раннего доступа.
2. Добавь in-app teaser в MyCellarScreen без изменения storage behavior.
3. Добавь AppMetrica events для teaser.
4. Обнови SEO/ASO план.
5. Проверка: flutter analyze по затронутым, HTML ссылки/мета.

Если задача P1 backend:
1. Создай миграцию user_cellars/cellar_locations/cellar_stock_lots/cellar_inventory_events.
2. Добавь owner-only RLS.
3. Добавь RPC для create/upsert location, place, move, location breakdown.
4. Не ломай user_storage и существующий add_to_user_storage.
5. Обязательный privacy guardrail: зоны хранения — только пользовательские ярлыки без lat/lng и без отображения на карте; география есть только у мест покупки/дегустаций из чеков/карты.
6. Проверка: SQL compile, RLS owner isolation, quantity sum checks, отсутствие координат у storage locations.

Не делай QR/NFC, exact bottle instances, paywall и desktop redesign в первом проходе.
```

---

## 21. Короткий Итог Для Roadmap

`Cellar Pro` — новый post-MVP retention/monetization workstream.

Readiness:

- `P0` можно делать сразу: SEO + waitlist + teaser.
- `P1` можно делать после ближайшего release/QA: structured locations поверх `user_storage`.
- `P2/P3` только после реального сигнала от пользователей с коллекциями.

Главная зависимость:

- не блокирует текущий receipt/draft moderation execution-order;
- усиливает add-bottle и SEO;
- становится естественным платным слоем после базового погребка.
