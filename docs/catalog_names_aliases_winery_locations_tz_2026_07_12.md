# WinePool — ТЗ: стандартизация названий, alias graph и точки виноделен

Дата: 12.07.2026  
Статус: discovery / связанный backlog  
Связано с:

- `catalog_normalization_moderation_tz_2026_05_11.md`;
- `wine_lines_tz_2026_06_14.md`;
- `label_line_ocr_tz_2026_06_15.md`;
- `ai_catalog_research_copilot_tz_2026_07_12.md`;
- tourism/wiki roadmap.

## 1. Цель

Создать единый и расширяемый стандарт:

- canonical names каталожных сущностей;
- локализованных отображаемых названий;
- транслитераций, переводов, чековых и OCR-алиасов;
- поиска по любому подтверждённому варианту;
- вычисляемых display titles без денормализации winery в `wine.name`;
- нескольких адресов/координат винодельни с различным назначением;
- безопасной публикации туристических точек только после подтверждения посещаемости.

## 2. Фактическое состояние на 12.07.2026

Уже существуют:

- `country_aliases` — `locale`, `source`, `is_primary`;
- `region_aliases` — scope по country, `locale`, `source`, `is_primary`;
- `winery_aliases` — alias type, locale, primary и admin CRUD;
- `grape_variety_aliases` — alias type, match tier, locale;
- `wine_aliases` — source, locale, confidence, primary;
- `wine_lines` и `wine_line_aliases`;
- canonical alias backfill;
- alias memory в atomic normalization finalize;
- receipt alias memory как отдельный обучающий слой;
- чтение `wine_line_aliases` в OCR/label flow;
- merge tooling для wine/winery/wine lines.

Alias foundation уже зрелый; создавать параллельную несовместимую систему нельзя.

### Выявленные пробелы

1. Нет единого словаря `alias_type` между сущностями.
2. `wine_line_aliases` не содержит `locale`, `source`, `created_by`, `confidence` и полноценного CRUD UI.
3. Canonical name, localized display name, transliteration и receipt alias местами смешиваются.
4. Нет явного стандарта вычисляемого wine display title.
5. Нет общей админ-поверхности управления aliases стран/регионов/вин/линеек.
6. `catalog_normalization_decisions.entity_type` пока не включает `wine_line`.
7. Не зафиксированы collision rules для одинакового alias у разных сущностей.
8. Алиасы помогают matching, но не решают системно локализованное отображение.
9. У winery только `location_text`, `latitude`, `longitude`; невозможно отличить legal/production/visitor location.
10. Нельзя безопасно понять, какую точку показывать туристу.

## 3. Стандарт canonical names

### Wine

`wine.name` хранит официальное название кюве/позиции без повторения winery.

Пример:

```text
winery.name = Lucien Dagonet et Fils
wine.name   = Tradition Brut
```

Не хранить:

```text
wine.name = Lucien Dagonet et Fils Tradition Brut Champagne
```

### Winery

Canonical — официальное используемое хозяйством написание, без искусственного перевода.

### Wine line

Canonical — официальное название серии/линейки внутри одной winery. Generic quality marks (`Reserve`, `Grand Reserve`) не становятся line без доказательства структуры производителя.

### Country/region/grape

Canonical выбирается по текущему продуктовому языку каталога, а оригинальные и международные формы сохраняются aliases/localizations.

## 4. Display policy

### Список/поиск

```text
Tradition Brut
Lucien Dagonet et Fils · Champagne
```

### Детальная карточка

```text
Lucien Dagonet et Fils
Tradition Brut
Традисьон Брют  // только если подтверждённая локализация полезна
```

### Compact surface без subtitle

```text
Lucien Dagonet et Fils · Tradition Brut
```

Display title вычисляется на read side и не записывается в `wine.name`.

## 5. Unified alias taxonomy

Рекомендуемые типы:

```text
canonical
official_alternative
localized_official
transliteration
translation
importer_name
retailer_name
receipt_alias
ocr_variant
brand
historical
abbreviation
common_misspelling
moderation
```

Каждый alias должен иметь:

- target entity;
- исходную строку;
- normalized string;
- locale, если известна;
- alias type;
- source/source URL или audit reference;
- confidence/verification status;
- primary flag только в рамках locale/type policy;
- creator/moderator;
- timestamps;
- enabled/rejected state вместо физического удаления обучающих данных.

Alias, предложенный AI, не становится trusted до подтверждения модератором.

## 6. Localized names

Не объединять языки в одной строке вида:

```text
Tradition Brut (Традисьон Брют)
```

Первый этап может переиспользовать alias tables с `locale` и типами `localized_official` / `transliteration`. Если продукту потребуется независимое редактирование display по locale, добавить универсальную таблицу:

```sql
catalog_entity_names (
  id uuid primary key,
  entity_type text not null,
  entity_id text not null,
  locale text not null,
  name text not null,
  normalized_name text not null,
  name_type text not null,
  is_display_primary boolean not null default false,
  source text,
  verification_status text not null default 'proposed',
  created_by_user_id uuid,
  created_at timestamptz not null,
  updated_at timestamptz not null
)
```

Не вводить эту таблицу до аудита реального использования существующих alias tables: предпочтительно расширить текущий задел, если он покрывает display policy.

## 7. Search/matching policy

Приоритет:

1. exact validated barcode;
2. trusted receipt memory exact;
3. canonical normalized exact;
4. verified alias exact;
5. winery alias + wine/line alias composition;
6. localized/transliterated exact;
7. controlled fuzzy within the same winery;
8. broad fuzzy только как moderation candidate.

Для generic wine names (`Chardonnay`, `Reserve`, `Brut`) winery является обязательным disambiguation signal.

Search index должен включать:

- canonical wine;
- computed winery + wine;
- wine aliases;
- winery aliases;
- line + line aliases;
- country/region aliases;
- barcode;
- transliterations;
- подтверждённые receipt aliases.

## 8. Wine line alias hardening

Расширить `wine_line_aliases`:

```sql
alter table wine_line_aliases
  add column locale text,
  add column source text not null default 'manual',
  add column confidence numeric,
  add column created_by_user_id uuid references profiles(id),
  add column verification_status text not null default 'verified';
```

Добавить:

- admin create/update/delete-or-disable RPC;
- UI aliases в winery details рядом с line management;
- alias preview в workbench;
- `wine_line` в normalization decisions;
- создание raw line alias при moderator finalize;
- collision check только внутри winery для line aliases;
- локализованные/транслитерированные line names в search.

## 9. Alias administration UX

Для country, region, winery, grape, line и wine:

- список aliases;
- locale/type/source/status;
- primary marker;
- поиск коллизий;
- merge/redirect target;
- добавить/редактировать/отклонить;
- audit history;
- preview: как сущность находится по этому alias;
- AI suggestions отдельно от verified aliases;
- bulk backfill только через dry run.

## 10. Winery locations

Добавить one-to-many `winery_locations`, не удаляя сразу legacy-поля winery.

```sql
create table winery_locations (
  id uuid primary key default gen_random_uuid(),
  winery_id uuid not null references wineries(id) on delete cascade,
  location_type text not null,
  name text,
  address_text text,
  country_code varchar references countries(code),
  region_id uuid references regions(id),
  locality text,
  postal_code text,
  latitude double precision,
  longitude double precision,
  phone text,
  email text,
  website text,
  booking_url text,
  visitor_instructions text,
  opening_hours jsonb,
  appointment_required boolean,
  accepts_visitors boolean,
  tourism_visibility text not null default 'hidden',
  verification_status text not null default 'unverified',
  source_url text,
  source_type text,
  verified_by_user_id uuid references profiles(id),
  verified_at timestamptz,
  last_checked_at timestamptz,
  notes text,
  created_at timestamptz not null,
  updated_at timestamptz not null
);
```

### Location types

```text
legal
head_office
estate
production
cellar
visitor_center
tasting_room
shop
vineyard
event_space
other
```

### Tourism visibility

```text
hidden
internal_candidate
public_by_appointment
public_open
temporarily_closed
permanently_closed
```

### Verification

```text
unverified
source_confirmed
partner_confirmed
moderator_verified
stale
rejected
```

## 11. Tourism publication rules

Для будущей winery wiki одна запись должна отвечать на конкретный вопрос, а не пытаться быть универсальным «адресом винодельни»:

- `legal` — где зарегистрировано хозяйство;
- `production` / `cellar` — где действительно делают или выдерживают вино;
- `visitor_center` / `tasting_room` / `shop` — куда разрешено приехать гостю;
- контакты, часы работы, необходимость записи и инструкция для посетителя относятся именно к конкретной location;
- если производство и приём гостей находятся по одному адресу, это подтверждается явно, но роли точки всё равно сохраняются.

Уточнение 13.09.2026: `production` — площадка **той же** винодельни. Если вино делает другая компания со своим именем и историей (хозяйство, работающее на бренд), это не location, а отдельная каталожная винодельня, связанная с вином как хозяйство-производитель: `docs/wine_production_winery_tz_2026_09_13.md`.

Для пользовательского интерфейса главным туристическим адресом считается не `production`, а лучшая подтверждённая публичная visitor-location. Производственный адрес может показываться в wiki как факт, но не должен автоматически строить маршрут для туриста.

На туристической карте показываются только:

- visitor/tasting/shop/estate точки с подтверждённым приёмом гостей;
- либо production/cellar, если официальный источник явно приглашает посетителей;
- с `public_by_appointment` или `public_open`;
- с проверенными координатами;
- хотя бы с одним актуальным способом связи или официальной инструкцией по посещению;
- с датой последней проверки.

Нельзя публиковать как туристическую точку:

- только юридический адрес;
- предположительный адрес из retailer;
- координаты, полученные AI без проверяемого адреса;
- production site без подтверждения посещаемости;
- частный дом только потому, что он указан в registry.

Priority точки winery:

1. подтверждённый visitor center/tasting room;
2. estate с официальным visitor information;
3. production/cellar by appointment;
4. legal — только internal, если не совпадает с public location.

### Winery card UI contract

Наличие адреса или координат само по себе не означает туристическую доступность. В карточке винодельни разделить два блока:

1. `Местоположение` — подтверждённый адрес/регион и карта с нейтральной подписью `Производство`, `Виноградник`, `Офис` или другой `location_type`.
2. `Посещение` — показывается только при наличии подтверждённой visitor-location или visit product.

Допустимые публичные badges:

- `Принимает гостей`;
- `Дегустации`;
- `Экскурсии`;
- `По предварительной записи`;
- `Фирменный магазин`;
- `Временно закрыто`.

Для `Посещение` показывать только применимые действия: `Записаться`, `Позвонить`, `Написать`, `Открыть сайт`, `Построить маршрут`. Рядом указывать дату проверки информации. Если посещение не подтверждено, не показывать отрицательный badge `Туристам недоступно`: нейтральная формулировка `Информация о посещении не подтверждена` допустима только на подробной странице, чтобы отсутствие сведений не воспринималось как запрет.

В админке нужны независимые controls `accepts_visitors`, `tastings_available`, `tours_available`, `appointment_required`, `shop_available`, источник подтверждения и `last_checked_at`. AI может предложить эти признаки по официальной странице, но не активирует публичные badges автоматически.

## 12. Wiki winery

`wineries` остаётся canonical identity. Wiki и tourism используют связанные сущности:

- locations;
- contacts;
- people/winemakers;
- history facts;
- sources;
- media assets;
- visit products/tours;
- verification history.

## 13. Inline creation of a missing grape variety during moderation

### Current state

- `GrapePickerSheet` is a shared selection component used both in the ordinary add/edit wine form and in the draft moderation flow.
- The shared picker currently only filters and selects existing `grape_varieties`.
- The protected RPC `admin_create_grape_variety` and repository method `CatalogNormalizationRepository.createGrapeVariety` already exist.
- The RPC checks the catalog-admin role, rejects a duplicate canonical name and a collision with an existing exact alias, creates the canonical alias, and can remember the raw moderation value as `moderation_submission`.

### Required UX

Do not make creation available to ordinary users. Extend the shared picker with an optional admin capability, for example `onCreateGrape`; when the callback is absent, the component behaves exactly as it does now.

In the moderation/admin context:

1. Moderator searches by the name found on the label or in a source.
2. Search must include canonical names and exact aliases, not only `grape_varieties.name`.
3. If there is no match, show `Добавить сорт «…»` below the search field or in the empty state.
4. Open a compact confirmation form without closing the wine workflow.
5. Required field: canonical name. Optional fields: original/raw name, locale, origin country/region, short internal note and source URL.
6. Before creation show possible canonical and alias collisions.
7. Call only the protected normalization RPC; do not insert into `grape_varieties` directly from UI.
8. After success refresh the picker, select the new variety automatically and return to the same scroll/search context.
9. If the entered spelling is an alias of an existing variety, offer `Добавить как алиас`, not a duplicate variety.
10. Record actor, submission id and source in the audit trail.

The first increment may reuse the current schema (`name`, `origin_region`, aliases). Country/region of origin should later be normalized into references rather than permanently stored as one display string.

### AI Copilot behaviour

AI may propose that a missing string is a new grape variety, but it must also provide possible synonyms and collision candidates. Creation remains an explicit moderator action. A low-confidence OCR fragment must never create a canonical grape automatically.

Один длинный AI-generated `description` не должен становиться источником истины. Wiki copy генерируется из verified fact pack.

## 13. AI Copilot integration

Copilot возвращает отдельно:

- legal location candidate;
- production candidate;
- visitor candidate;
- evidence для каждого;
- geocoding result;
- visitability evidence;
- conflicts;
- freshness.

AI не выставляет `tourism_visibility=public_*` и не публикует координаты. Это решение модератора/партнёра.

По names/aliases Copilot предлагает:

- canonical original;
- localized display;
- transliteration;
- importer/retailer name;
- receipt/OCR aliases;
- line candidate;
- collision warnings.

## 14. Миграция legacy winery coordinates

1. Инвентаризировать `wineries.location_text/latitude/longitude`.
2. Создать `winery_locations` как additive layer.
3. Перенести legacy значения в `internal_candidate` без автоматической tourism publication.
4. Классифицировать source/type, где возможно.
5. Сохранить legacy read fallback.
6. Перевести UI и map read на confirmed primary location.
7. Удаление legacy полей — отдельное позднее решение после backfill и QA.

## 15. Этапы

### A. Audit

- counts и coverage всех alias tables;
- duplicate/collision report;
- locale/type/source completeness;
- фактическое использование в search/matcher/UI;
- audit legacy winery coordinates.

### B. Naming policy

- computed display title;
- original/localized presentation;
- unified alias taxonomy;
- line alias hardening;
- admin alias UX.

### C. Winery locations

- migration/RLS/RPC;
- moderator CRUD;
- source/evidence;
- geocoding и coordinate validation;
- legacy backfill.

### D. Tourism/wiki

- public visibility workflow;
- stale re-verification;
- map/read model;
- partner confirmation;
- wiki fact pack.

## 16. Acceptance criteria

- foreign canonical wine name не содержит дублированную winery;
- UI однозначно показывает winery рядом с wine;
- поиск находит canonical по русской транслитерации и чековому alias;
- line находится по localized/transliterated aliases;
- алиасы не создаются без audit/source;
- collision не приводит к auto-match без winery/country scope;
- winery поддерживает несколько разнотипных locations;
- legal address не публикуется туристам автоматически;
- каждая public tourism point имеет visitability evidence и last checked date;
- AI может предлагать, но не публиковать name/location/media;
- существующий catalog/receipt flow не ломается при additive migration.

## 17. Первый реальный кейс

Submission `76726ac4-996d-4a29-92b6-768e762e8f0d` показал:

- raw `Lucien Dagonet et Fils` является winery, а не wine name;
- canonical wine — `Tradition Brut`;
- display должен быть `Lucien Dagonet et Fils · Tradition Brut`;
- barcode exact помогает связать русские retailer names;
- пользовательский vintage 2020 конфликтует с NV bottle;
- AI смешал неподтверждённый адрес Festigny с winery в Boursault;
- legal address 5 rue и retailer/manufacturer address 7 rue требуют типизации;
- ни один из них нельзя автоматически считать visitor location;
- line `Tradition` требует отдельного решения: подтверждённая line или часть wine name;
- Gemini free-form answer выдал маркетинговую copy и пропустил evidence/status по полям.
