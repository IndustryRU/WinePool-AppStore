# 🍷 WinePool Master Plan: Рефакторинг бизнес-архитектуры (v2.0)

> **Статус с 25.08.2026: historical architecture sketch, не source of truth.**
> Часть модели была развита и уточнена в `role_business_*`, `session_2_6_release_aware_offers_tz_2026_08_07.md`, `retail_stock_and_sync_2026_08_07.md` и tourism organization/membership модели. Текущая продуктовая очередь продавца и винодельни: H2/H3 в [post-1.1.0 roadmap](/R:/Flutter/Project/winepool_final/docs/post_release_1_1_0_execution_roadmap_2026_08_25.md). Не создавать миграции по SQL-эскизу ниже без runtime/database audit.

## 1. Цель
Трансформация WinePool из простого каталога в гибридную экосистему, объединяющую производителей (винодельни), ритейл и винный туризм. Переход от модели "один пользователь — один магазин" к модели "бизнес-сущности и точки присутствия".

---

## 2. Новая архитектура данных (SQL)

### 2.1. Таблица `businesses` (Бизнес-профили)
Отделяет коммерческую деятельность от личных данных пользователя.
- `id` (uuid, PK)
- `owner_id` (uuid, FK profiles.id) — владелец бизнеса
- `name` (text) — название бренда или сети
- `description` (text)
- `logo_url` (text)
- `banner_url` (text)
- `type` (enum: 'winery', 'retail', 'horeca')
- `managed_winery_id` (uuid, FK wineries.id, optional) — для производителей
- `is_partner` (boolean) — статус официального партнера WinePool
- `is_verified` (boolean) — пройдена ли проверка документов
- `created_at`, `updated_at`

### 2.2. Таблица `locations` (Точки присутствия)
Унифицированное хранилище всех физических адресов (магазинов, усадеб, складов).
- `id` (uuid, PK)
- `business_id` (uuid, FK businesses.id)
- `type` (enum: 'winery_estate', 'tasting_room', 'retail_shop', 'warehouse')
- `address_text` (text)
- `latitude`, `longitude` (numeric)
- `is_tourism_active` (boolean) — принимает ли точка туристов
- `phone`, `email` (specific to location)

---

## 3. Ключевые бизнес-механики

### 3.1. Логика "Прямая поставка" (VDS - Verified Direct Supply)
Система автоматически присваивает офферу статус "ПРЯМАЯ ПОСТАВКА", если:
`offer.business_id.managed_winery_id == wine.winery_id`
Это гарантирует покупателю, что он покупает вино непосредственно у создателя.

### 3.2. Система владения (Verification Workflow)
- По умолчанию загруженные винодельни имеют статус `management_status = 'unclaimed'`.
- Реальный владелец может подать заявку на "привязку" винодельни к его `business_id`.
- После одобрения админом, винодельня переходит в статус `claimed`, а бизнес получает права на редактирование карточек вин бренда.

### 3.3. B2B Слой
- Возможность установки "Оптовых цен" в офферах для других бизнес-аккаунтов.
- Ролевая модель: профиль может одновременно иметь корзину покупателя и панель управления заказами бизнеса.

---

## 4. План реализации (Step-by-Step)

### Шаг 1: Миграция базы данных
- [ ] Создание таблиц `businesses` и `locations`.
- [ ] Перенос существующих данных из `profiles.shop_name` и `profiles.managed_winery_id` в новую таблицу.
- [ ] Связывание существующих `offers` с новыми `business_id`.

### Шаг 2: App Layer (Flutter)
- [ ] Разработка моделей `Business` и `Location`.
- [ ] Создание `BusinessRepository` и `LocationRepository`.
- [ ] Обновление `WinesRepository` для поддержки фильтрации по `business_id`.

### Шаг 3: Кабинет управления
- [ ] Экран "Мой Бизнес" для управления точками (locations) и данными бренда.
- [ ] Механика Claim Winery (заявка на владение).

### Шаг 4: Продвинутый фильтр "МАГАЗИНЫ"
- [ ] Уровень 1: Популярные партнеры и винодельни.
- [ ] Уровень 2: Полный список с группировкой по типу и геолокации.
- [ ] Бейджи VDS в каталоге.

### Шаг 5: Туризм (Перспектива)
- [ ] Фильтр "Винный туризм" (поиск по `locations.is_tourism_active`).
- [ ] Кнопка "Маршрут до винодельни".
