# Wine production attributes and certifications

Дата: 12.07.2026  
Статус: backlog / связанное ТЗ к AI Catalog Research Copilot

## 1. Причина

Текущая группа `Экологические стили` смешивает четыре разных понятия:

- `Pet-Nat` — метод производства игристого вина;
- `Organic` — регулируемый подход/claim, часто подтверждаемый сертификацией;
- `Biodynamic` — подход к земледелию и виноделию, который может иметь сертификацию;
- `Natural` — редакционно сложный стиль без единого международного стандарта.

Новые факты вроде `Demeter` и `Vegan` нельзя корректно добавлять в этот же список как равнозначные checkbox:

- `Demeter` — конкретная сертификация/организация;
- `Vegan` — product suitability / особенность производства, а не экологический стиль.

## 2. Current implementation audit

Legacy booleans `wines.is_pet_nat`, `is_organic`, `is_biodynamic`, `is_natural` используются в:

- `Wine` domain model и JSON;
- add/edit wine form;
- wine details и characteristic icons;
- catalog cards, home and cellar;
- catalog filters state, filter UI и RPC parameters;
- OCR/label keyword extraction;
- public wine payloads and price/search RPC.

Следовательно, изменение должно быть additive. Удалять legacy columns в первом проходе нельзя.

## 3. Целевая taxonomy

### 3.1. Production methods

Методы хранить структурированно по группам, поскольку одно вино может иметь несколько совместимых техник.

#### Sparkling method (один основной)

- `traditional_method` — вторичная ферментация в бутылке;
- `charmat_method` / `tank_method` — вторичная ферментация в резервуаре;
- `ancestral_method_pet_nat` — méthode ancestrale;
- `transfer_method`;
- `continuous_method`;
- `carbonation` — искусственное насыщение CO₂, только при подтверждении.

#### Fermentation and extraction

- `spontaneous_fermentation` / indigenous yeast;
- `cultured_yeast`;
- `carbonic_maceration`;
- `semi_carbonic_maceration`;
- `skin_contact`;
- `whole_cluster`;
- `malolactic_fermentation` / `malolactic_blocked`;
- fermentation vessel: steel, concrete, amphora, oak and other controlled values.

#### Ageing and finishing

- ageing vessel: steel, concrete, amphora, neutral oak, new oak;
- `lees_ageing` with optional duration;
- `batonnage`;
- `solera`;
- `flor_ageing`;
- `unfiltered`;
- `unfined`.

Метод не объявляется экологическим claim.

Не требуется заполнять все возможные техники для каждого вина. Хранить только подтверждённые и полезные пользователю характеристики. Числовые параметры (например, 8 недель второй ферментации, 24 месяца на осадке, доля нового дуба) не кодировать десятками attributes: для них нужны optional value/unit/details на assignment либо отдельный production profile.

### 3.2. Farming and winemaking approaches

Примеры:

- `organic`;
- `biodynamic`;
- `natural_wine`;
- `low_intervention`;
- `regenerative`.

Подход может быть заявленным производителем, подтверждённым документом либо сертифицированным. Эти уровни нельзя смешивать.

### 3.3. Certifications

Примеры:

- `eu_organic`;
- `demeter`;
- `biodyvin`;
- национальные organic marks;
- `vegan_certified`, если существует конкретная certification organization;
- `kosher`, если требуется структурированный сертификат.

Хранить issuer, certificate/standard name, scope, validity dates, source and verification.

### 3.4. Product features and suitability

Примеры:

- `vegan`;
- `vegetarian`;
- `no_added_sulfites`;
- `low_sulfites`;
- `gluten_free` только если это продуктово оправдано и подтверждено;
- `kosher` как suitability, связанная с сертификатом.

Фраза `vegan` не означает `organic`, а `organic` не означает `vegan`.

## 4. Proposed data model

Для небольшого фиксированного vocabulary использовать catalog definitions и wine assignments, а не добавлять новые columns на каждый признак.

```sql
create table wine_attribute_definitions (
  id uuid primary key default gen_random_uuid(),
  code text not null unique,
  category text not null,
  display_name_ru text not null,
  display_name_en text,
  description text,
  icon_key text,
  filterable boolean not null default false,
  requires_source boolean not null default true,
  is_active boolean not null default true,
  sort_order integer not null default 0,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table wine_attribute_assignments (
  id uuid primary key default gen_random_uuid(),
  wine_id uuid not null references wines(id) on delete cascade,
  attribute_definition_id uuid not null references wine_attribute_definitions(id),
  claim_status text not null default 'unverified',
  source_url text,
  source_type text,
  evidence_text text,
  value_numeric numeric,
  value_text text,
  unit text,
  certificate_issuer text,
  certificate_name text,
  valid_from date,
  valid_until date,
  verified_by_user_id uuid references profiles(id),
  verified_at timestamptz,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  unique (wine_id, attribute_definition_id)
);
```

`claim_status`:

```text
ai_suggested
label_claimed
producer_claimed
source_confirmed
certified
moderator_verified
stale
rejected
```

Если одна сертификация относится ко всему хозяйству, в будущем добавить winery-level assignment и явное наследование с product-level override. Не копировать автоматически winery certification на каждое вино без проверки scope.

Метод может меняться по vintage или cuvée. После появления `wine_vintages` assignment должен поддерживать optional `wine_vintage_id`; wine-level запись означает устойчивую характеристику продукта, vintage-level — подтверждённое исключение/уточнение конкретного урожая.

## 5. UI contract

Форму разделить:

1. `Метод производства` — Pet-Nat/ancestral, traditional, Charmat и т. п.
2. `Подход` — organic, biodynamic, natural, low intervention.
3. `Сертификации` — Demeter, EU Organic и другие, с источником.
4. `Особенности` — Vegan и другие product features.

Для карточек использовать compact badges/icons только для проверенных public attributes. Не выводить больше двух-трёх badges в compact card; остальные доступны в подробной карточке.

В подробной карточке добавить блок `Производство`, сгруппированный по смыслу:

- основной метод (самая заметная строка для игристого);
- ферментация;
- выдержка;
- фильтрация/осветление;
- подтверждённые детали и длительность.

Если методов нет, блок скрывается. Не показывать `Неизвестно` и не подменять отсутствие информации общими выводами из типа вина. В compact wine card метод показывать только когда он действительно помогает различить стиль (например, `Традиционный метод`, `Pet-Nat`, `Amphora`); `Charmat` остаётся в details/filter, если карточка перегружена.

Фильтры по методам должны быть предметными и малочисленными. На старте: `Традиционный`, `Шарма`, `Pet-Nat`, `Карбоническая мацерация`, `Амфора`, `Выдержка в дубе`, `Без фильтрации`. Остальные значения доступны в details/search, но не обязаны становиться отдельными checkbox.

В wine details показать смысл badge и основание. Например:

```text
Биодинамическое
Сертификация Demeter · подтверждено производителем
```

Иконки должны кодировать category, а не создавать отдельный визуальный язык для каждого нового слова:

- method — bubbles/process icon family;
- approach — leaf/soil icon family;
- certification — seal/badge icon family;
- suitability — product/person icon family.

Фильтры каталога строятся по `filterable` definitions. Сертификация и feature не становятся фильтром автоматически только потому, что существуют в справочнике.

## 6. Admin moderation

- AI возвращает category, code, confidence and evidence source;
- модератор подтверждает каждый assignment отдельно;
- unsupported marketing language не создаёт verified claim;
- `Demeter` требует подтверждения сертификатом/официальной product page;
- `Vegan` требует product-specific label/official page; общий стиль винодельни недостаточен;
- при конфликте vintage/product scope assignment остаётся unverified.

## 7. Migration plan

### Phase A — additive compatibility

1. Создать definitions/assignments и seed для четырёх legacy attributes плюс `charmat_method`, `eu_organic`, `demeter`, `vegan`.
2. Backfill legacy true booleans в assignments с `source_type = legacy_backfill` и осторожным status, не выше `source_confirmed` без evidence.
3. Сохранить legacy columns, filters и payloads.
4. Dual-read: новый UI читает assignments, при их отсутствии использует legacy booleans.
5. Dual-write для четырёх legacy значений на переходном этапе.

### Phase B — UI and API

1. Обновить add/edit form, workbench и AI proposal panel.
2. Обновить details/card/home/cellar icons.
3. Перевести catalog filters/RPC на attribute codes с compatibility parameters.
4. Добавить admin dictionary management и audit.

### Phase C — cleanup

После метрик, backfill verification и release window удалить legacy UI wiring. Physical columns удалять только отдельной миграцией после подтверждения отсутствия старых клиентов/RPC.

## 8. Current Fidora moderation case

Для `Fidora Prosecco DOC Spumante Brut` в текущей форме:

- `is_organic = true`;
- `is_biodynamic = true`;
- `is_pet_nat = false`;
- `is_natural = false`.

Research metadata / future assignments:

- `charmat_method` — official product page;
- second fermentation duration: `8 weeks` — official product page;
- `eu_organic` — official product page/logo;
- `demeter` — official product page/logo;
- `vegan` — official product page/logo.

Текущее `tannins = 1` является UI-forced placeholder, а не подтверждённым значением. После nullable/0–5 migration его нужно вернуть в `null`, если редакционная методика не подтверждает значение.

## 9. Acceptance criteria

- Pet-Nat больше не подписан как экологический стиль;
- certification and vegan не моделируются отдельными columns;
- legacy catalog filters continue to work during migration;
- compact cards do not overflow from badges;
- wine details explain the meaning/source of attributes;
- AI cannot publish an attribute without moderator confirmation;
- historical data has a reconciliation report and no silent loss.
