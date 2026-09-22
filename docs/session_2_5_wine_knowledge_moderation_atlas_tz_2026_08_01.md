# Session 2.5 — наполнение карточки: AI → модерация → Atlas → карточка

Дата: 01.08.2026

Статус: утверждено владельцем 03.08.2026; дополнено 03.08.2026 контуром
legacy knowledge mining и максимального переиспользования AI research

Прогресс 03.08.2026: срез 1 (taxonomy/assignments) и срез 2 (ручной admin
CRUD + Flutter admin UI) реализованы и проверены PostgreSQL runtime-contract
через `BEGIN/ROLLBACK`, статическим анализом и widget-тестами. Миграции
`202608030100_extend_wine_knowledge_taxonomy.sql` и
`202608030200_add_wine_knowledge_admin_rpcs.sql` применены на production одной
транзакцией; PostgREST schema cache перезагружен. Backup перед применением:
`/root/db_backups/winepool_pre_knowledge_admin_20260803_120219.dump` (3,2 МБ).
Post-apply admin CRUD smoke выполнен с `ROLLBACK`, тестовых записей не осталось.
Дополнительно реализованы исправления Atlas aliases, динамическая ручная форма,
release knowledge CRUD и счётчики знаний релиза. Следующий срез — `3A-0 Legacy
knowledge mining`, затем `3A research-report.v4 + proposal materialization`.

Read-only production baseline `3A-0` выполнен 03.08.2026 и зафиксирован в
`session_2_5_legacy_knowledge_profile_2026_08_03.md`: 1 051 активное вино,
1 042 описания, 983 aroma-сигнала, 240 explicit pairing-сигналов и 52 уже
оплаченных отчёта v3 на 46 вин. Следующий шаг внутри `3A-0` — версионированный
extractor/normalizer и dry-run clusters без записей в production.

Extractor v1 и production dry-run завершены в тот же день. Результат:
2 867 preview proposals, 204 clusters, 695 legacy wines с распознанными facts,
13 конфликтов и 0 safe candidates до taxonomy seed. Код и QA описаны в
`session_2_5_legacy_knowledge_extractor_v1_results_2026_08_03.md`. Перед
materialization требуется product checkpoint первой волны seed и исправление
слишком широкого alias `Слива` у term `Чёрная слива`.

Срез A (04.08.2026) закрыл два пробела Этапа 1, найденные аудитом
`session_2_5_2_6_completion_audit_2026_08_04.md`: `atlas_term_relations`
(general → specific DAG, cycle- и kind-guard, ancestors/descendants) и
`wine_terms.effect` (`include`/`exclude`) с правилом «exclude только на релизе и
только поверх существующего wine-level include». Миграция
`202608040100_add_atlas_hierarchy_and_assignment_effect.sql` применена на
production 04.08.2026. Backup:
`/root/db_backups/winepool_pre_atlas_hierarchy_20260804_215927.dump`,
SHA-256 `58b36ee14b0294a94c2f44cac2ae4bfa0a220d7826c0dc1bc783e0b3033ce7a2`.
Контракт `supabase/tests/20260804_atlas_hierarchy_and_effect_contract.sql`
пройден в dry-run и повторно после apply; PostgREST перезагружен. В admin UI
появились раздел «Иерархия Atlas» в карточке термина и действие «Не относится к
этому релизу» с обратимым «Вернуть в релиз».

Владелец 03.08.2026 утвердил general → specific hierarchy и отдельные canonical
terms `Слива` / `Чёрная слива`. Полный product review draft seed, aliases,
parents и icon coverage вынесен в
`session_2_5_atlas_seed_review_v1_2026_08_03.md`; production seed ещё не
применялся.

Владелец направления: WinePool

Область: admin web, Supabase/Postgres, AI Catalog Research Copilot, Flutter-карточка вина, WinePool Atlas

Этот документ развивает и уточняет раздел 13
`session_2_5_release_experience_and_wine_card_tz_2026_07_29.md` и Phase A/B
`wine_production_attributes_certifications_tz_2026_07_12.md`. При расхождении
по admin input, assignments и research apply приоритет имеет настоящее ТЗ.

## 1. Цель

Создать единый безопасный контур, в котором AI-помощник исследует вино, находит
и нормализует факты, прикладывает доказательства, а модератор с минимальным
числом действий подтверждает, уточняет или отклоняет предложения. Подтверждённые
данные структурированно сохраняются на уровне вина или конкретного релиза,
появляются в карточке и связываются со справочником WinePool Atlas.

Контур должен покрывать семь направлений:

1. справочник Atlas;
2. структурированные термины и характеристики вина;
3. характеристики конкретного релиза;
4. применение результатов AI-исследования;
5. структурированные награды;
6. совместимость со старыми полями и клиентами.
7. извлечение и повторное использование знаний из существующего каталога и уже
   оплаченных research reports без повторного веб-исследования.

Главный продуктовый результат: модератор не перепечатывает найденную информацию,
но AI никогда не публикует спорный факт в каталог без явного модераторского
решения.

## 2. Исходное состояние и подтверждённый разрыв

На 01.08.2026 уже существуют:

- `catalog_research_jobs` и неизменяемые `catalog_research_reports`;
- `research-report.v3`, содержащий основные поля, вкус, eco-флаги,
  `production_attributes`, награды, источники и предупреждения;
- Workbench, показывающий найденные методы, подходы, сертификации, особенности,
  апелласьон и награды;
- таблицы `atlas_terms`, `wine_terms`, `wine_awards`;
- release foundation: `wine_vintages`, `wine_vintage_images`,
  `wine_product_codes`;
- карточка с блоками «Особенности», «Аромат», «Вкус», «Хорошо сочетается» и
  release-aware наградами;
- четыре legacy-флага: Pet-Nat, organic, biodynamic, natural;
- legacy-поля `wines.aroma`, `wines.pairing`, `wines.awards`;
- `wines.description`, где у импортированных и пользовательских вин уже могут
  находиться ароматы, pairing, методы и другие факты;
- `wine_vintages.characteristics jsonb`, которое карточка умеет читать для
  ручного сбора, дубовой выдержки и апелласьона.

Однако:

- Workbench применяет только whitelist базовых полей, вкуса, описаний и четырёх
  eco-флагов;
- production findings, апелласьон и награды в отчёте read-only;
- у `wine_terms` нет рабочего admin CRUD и полноценного read path карточки;
- нет `wine_vintage_terms` или другого структурированного release-level
  назначения;
- у aroma/pairing нет admin-полей;
- у `wine_awards` есть публичное чтение, но нет законченного admin CRUD;
- справочник Atlas не имеет административной поверхности;
- подтверждение/отклонение отдельных AI-предложений не сохраняется как
  самостоятельное аудируемое решение.
- структурированные данные `research-report.v3` сохраняются, но не
  материализуются в назначения Atlas;
- старые `aroma`, `pairing` и `description` не профилируются как корпус знаний,
  а происхождение отдельного фрагмента старого описания хранится неполно.

Следовательно, визуальная карточка опережает контур наполнения. Настоящее ТЗ
закрывает этот разрыв без создания второй админки или второго research pipeline.

## 3. Неизменяемые принципы

1. **AI предлагает — модератор публикует.** Research worker не пишет напрямую в
   `wines`, `wine_vintages`, назначения Atlas или `wine_awards`.
2. **Факт и доказательство неразделимы.** Для research-назначения хранится URL,
   тип источника, confidence и краткое основание.
3. **Уровень данных явный.** Вино и релиз не смешиваются. Факт релиза не
   становится свойством всех урожаев.
4. **Сертификация не равна подходу.** Organic claim, EU Organic certificate,
   Demeter и Vegan — разные сущности/виды.
5. **Подтверждённое важнее предположительного.** Конфликт или слабый источник не
   попадает в публичную карточку одним bulk-action.
6. **Atlas — единый словарь.** Не создавать свободные дубли «ручной сбор»,
   `hand harvested`, `hand-picked`; aliases приводят их к одному термину.
7. **Additive rollout.** Старые поля и APK продолжают работать до отдельного
   cleanup-релиза.
8. **Одна модераторская поверхность.** Расширяется текущий Workbench и текущие
   редакторы вина/релиза.
9. **Оплаченное исследование используется повторно.** Существующий report v3
   сначала материализуется без нового сетевого поиска; повторное исследование
   запускается только при реальном дефиците доказательств.
10. **Legacy-текст — источник proposal, а не canonical truth.** Извлечённые из
    старых полей значения проходят тот же match, review, audit и apply-контур.
11. **Описание и факты разделены.** Система извлекает короткие структурированные
    факты, но не копирует исходный редакционный текст в Atlas term.

## 4. Границы среза

### 4.1. Входит

- словарь терминов Atlas и aliases;
- назначения терминов на вино и релиз;
- aroma и pairing как структурированные термины;
- методы, подходы, сертификации, eco, особенности и апелласьон;
- числовые/текстовые уточнения назначения: например, `12 мес.`;
- предложения research report, их состояние и аудит;
- безопасное массовое применение;
- ручное редактирование тех же данных;
- structured awards и release binding;
- карточный dual-read и Atlas interaction;
- миграции, RLS, RPC, backfill, тесты и production runbook.
- read-only профилирование существующего корпуса `wines`;
- построение Atlas candidates и aliases из legacy aroma/pairing/description;
- reuse materialization существующих immutable reports v3;
- release override/exclusion для наследуемых wine-level характеристик.

### 4.2. Не входит

- автоматическая публикация без модератора;
- генерация полноценных статей Atlas самим этим срезом;
- публичное пользовательское редактирование справочника;
- фильтры каталога по каждому новому термину;
- наследование сертификации винодельни на все её вина;
- удаление legacy-колонок;
- внешние рейтинги критиков;
- commerce/SKU-особенности Session 2.6.

## 5. Терминология и уровни

### 5.1. Виды терминов `atlas_terms.kind`

- `appellation` — апелласьон/ЗГУ/ЗНМП;
- `method` — технологический метод;
- `approach` — подход к земледелию или виноделию;
- `certification` — конкретная сертификация/стандарт;
- `eco` — сильный экологический пользовательский сигнал;
- `feature` — особенность/пригодность продукта;
- `style` — стиль, включая Pet-Nat;
- `aroma` — ароматический дескриптор;
- `pairing` — гастрономическое сочетание;
- `award_program` — конкурс/программа наград;
- существующие `grape`, `region` сохраняются.

### 5.2. Scope назначения

- `wine` — устойчивое свойство продукта/кюве;
- `release` — подтверждено только для `wine_vintage_id`;
- `unknown` допустим только в proposal, но не в опубликованном assignment.

У назначения также есть семантика:

- `include` — добавить/подтвердить term;
- `exclude` — на уровне релиза не наследовать конкретный wine-level term;
- `replace` — модераторское действие для singleton-класса: release-level
  `include` становится эффективным значением вместо wine-level singleton.

`replace` хранится как решение/apply intent, а эффективная модель состоит из
release-level `include` и политики singleton. Для multi-value исключение
хранится явным release-level `exclude`.

Примеры:

- «натуральное вино» для всей карточки → `wine`;
- «выдержка 12 месяцев в дубе» для 2023 → `release`;
- «чёрная слива» из общей техкарты без года → `wine`;
- награда Decanter 2024 за релиз 2021 → отдельная `wine_awards` с
  `wine_vintage_id=2021` и `year=2024`.

## 6. Целевая модель данных

Названия миграций уточняются при реализации и получают фактическую дату. Ниже —
обязательный логический контракт.

### 6.1. Расширение `atlas_terms`

Сохранить существующие поля и добавить:

```sql
status text not null default 'draft'
  check (status in ('draft','published','archived')),
category_key text,
default_icon_key text,
created_by uuid,
updated_by uuid,
published_at timestamptz,
metadata jsonb not null default '{}'
```

`is_published` на переходе синхронизируется со `status='published'`. Удалять его
в этом срезе нельзя.

### 6.2. `atlas_term_aliases`

```sql
id uuid primary key,
term_id uuid not null references atlas_terms(id) on delete cascade,
locale text,
alias text not null,
normalized_alias text not null,
source text not null default 'moderator',
created_at timestamptz not null,
unique (term_id, normalized_alias)
```

Отдельный уникальный индекс должен предотвращать ситуацию, когда один
нормализованный alias активен у двух опубликованных терминов одного `kind`.

### 6.2.1. `atlas_term_relations`

General → specific hierarchy хранится отдельными отношениями, потому что Atlas
является DAG, а не строгим деревом: один aroma может принадлежать нескольким
семействам.

```sql
parent_term_id uuid not null references atlas_terms(id) on delete cascade,
child_term_id uuid not null references atlas_terms(id) on delete cascade,
relation_type text not null default 'is_a'
  check (relation_type in ('is_a')),
sort_order integer not null default 0,
created_by uuid,
created_at timestamptz not null,
primary key (parent_term_id, child_term_id, relation_type),
check (parent_term_id <> child_term_id)
```

Admin RPC обязан:

- разрешать relation только между terms одного `kind`;
- предотвращать прямые и транзитивные циклы;
- не удалять child assignment при изменении hierarchy;
- возвращать ancestors/descendants для effective display merge;
- поддерживать несколько parents, например `Вишня` → `Красные ягоды` и
  `Косточковые фрукты`.

Public read использует relations для подавления general parent только внутри
одного effective evidence/assignment context; hierarchy не превращает child в
автоматическое назначение всех его parents.

### 6.3. Эволюция `wine_terms` в assignments

Не создавать параллельную семантически равную таблицу. Расширить существующую
`wine_terms`:

```sql
id uuid primary key default gen_random_uuid(),
wine_id uuid not null references wines(id) on delete cascade,
wine_vintage_id uuid references wine_vintages(id) on delete cascade,
term_id uuid not null references atlas_terms(id) on delete restrict,
status text not null default 'verified'
  check (status in ('verified','source_claimed','archived')),
source text not null
  check (source in ('moderator','research','import','legacy_backfill')),
effect text not null default 'include'
  check (effect in ('include','exclude')),
confidence numeric,
evidence_url text,
evidence_kind text,
evidence_note text,
value_text text,
value_numeric numeric,
value_unit text,
sort_order integer not null default 0,
verified_by uuid,
verified_at timestamptz,
provenance jsonb not null default '{}',
created_at timestamptz not null,
updated_at timestamptz not null
```

Старый composite primary key `(wine_id, term_id)` заменить суррогатным `id`.
Добавить partial unique indexes:

- `(wine_id, term_id)` при `wine_vintage_id is null` и status не archived;
- `(wine_vintage_id, term_id)` при `wine_vintage_id is not null` и status не archived.

RPC/trigger обязан проверить, что `wine_vintage_id` принадлежит `wine_id`.
`effect='exclude'` разрешён только при `wine_vintage_id is not null`, требует
существующего или предложенного wine-level `include` и не публикуется как
самостоятельная положительная характеристика.

`source_claimed` может быть виден в экспертном режиме, но в обычную публичную
карточку входят только `verified`. Четыре legacy-флага продолжают отображаться
fallback-путём независимо от этой таблицы до завершения backfill QA.

### 6.4. Research proposals

Создать `catalog_knowledge_proposals`:

```sql
id uuid primary key,
origin_type text not null check (origin_type in (
  'research_report_v4','research_report_v3','legacy_field',
  'legacy_description','catalog_import'
)),
origin_ref text not null,
report_id uuid references catalog_research_reports(id) on delete cascade,
job_id uuid references catalog_research_jobs(id) on delete cascade,
submission_id uuid references draft_catalog_submissions(id),
wine_id uuid references wines(id),
proposal_key text not null,
proposal_type text not null
  check (proposal_type in ('term_assignment','award')),
proposal_effect text not null default 'include'
  check (proposal_effect in ('include','exclude','replace')),
term_kind text,
suggested_term_slug text,
suggested_title_ru text,
matched_term_id uuid references atlas_terms(id),
scope_hint text check (scope_hint in ('wine','release','unknown')),
wine_vintage_id uuid references wine_vintages(id),
release_hint jsonb,
value_payload jsonb not null default '{}',
confidence numeric,
verification_state text not null,
evidence jsonb not null default '[]',
extraction_payload jsonb not null default '{}',
extractor_version text,
moderation_status text not null default 'pending'
  check (moderation_status in ('pending','accepted','edited','rejected','superseded')),
decision_note text,
decided_by uuid,
decided_at timestamptz,
applied_assignment_id uuid references wine_terms(id),
applied_award_id uuid references wine_awards(id),
created_at timestamptz not null,
updated_at timestamptz not null,
unique (origin_type, origin_ref, proposal_key)
```

Предложение — не публичный факт. Его RLS разрешает читать только catalog admin и
service role.

Правила целостности origin:

- для `research_report_v3/v4` обязательны `report_id`, `job_id` и фактическая
  версия schema;
- для legacy origin обязательны `wine_id`, имя исходного поля и fingerprint
  исходного значения в `origin_ref/extraction_payload`;
- изменение исходного legacy-текста создаёт новую версию proposals и переводит
  только старые pending в `superseded`; решения модератора не переписываются;
- один proposal-контур используется для AI и backfill, отдельной временной
  таблицы импорта не создаётся.

### 6.5. Audit

Создать `catalog_knowledge_moderation_events`:

```sql
id uuid primary key,
proposal_id uuid,
actor_user_id uuid not null,
action text not null,
before_payload jsonb,
after_payload jsonb,
request_id uuid not null,
created_at timestamptz not null,
unique (actor_user_id, request_id, proposal_id, action)
```

Нужны действия: `accept`, `accept_edited`, `reject`, `restore`, `archive`,
`manual_create`, `manual_update`, `bulk_accept`.

### 6.6. Награды

Существующую `wine_awards` сохранить и дополнить при отсутствии:

- `status`: `verified/source_claimed/archived`;
- `confidence`;
- `evidence_note`;
- `provenance jsonb`;
- `verified_at`;
- `created_by`, `updated_by`;
- `sort_order`.

Правила:

- `program_term_id` предпочтительнее `program_name`;
- `program_name` — fallback для ещё не заведённого конкурса;
- `year` — год присуждения, не винтаж;
- `wine_vintage_id` — релиз, которому присуждена награда;
- изображение конкретной медали публикуется только при заполненных
  `badge_source_url` и `badge_license_note`;
- без изображения используется asset уровня gold/silver/bronze/neutral.

## 7. Контракт AI research report v4

Существующий v3 не ломается. Новый worker пишет `research-report.v4` и сохраняет
прежние разделы для обратной совместимости.

Добавить:

```json
{
  "knowledge_proposals": [
    {
      "proposal_key": "method.traditional.release-2021",
      "kind": "method",
      "canonical_slug_candidate": "traditional-method",
      "display_value_ru": "Традиционный метод",
      "scope_hint": "release",
      "effect": "include",
      "release_ref": {
        "vintage": 2021,
        "is_non_vintage": false,
        "release_name": null
      },
      "value": {
        "text": null,
        "number": null,
        "unit": null
      },
      "confidence": 0.94,
      "verification_state": "official_source",
      "evidence": [
        {
          "url": "https://producer.example/tech-sheet",
          "publisher": "producer",
          "source_kind": "official_product_page",
          "note": "Метод указан в технической карте",
          "quote_or_fragment": "Традиционный метод производства",
          "accessed_at": "2026-08-01T00:00:00Z"
        }
      ],
      "suggested_action": "link_existing"
    }
  ],
  "award_proposals": []
}
```

Обязательные правила агента:

1. Не выводить отсутствие упоминания как `false`.
2. Не превращать маркетинговое «бережное производство» в organic/natural.
3. Certification требует конкретного issuer/стандарта или официального знака.
4. Награда требует официального сайта конкурса, производителя либо другого
   проверяемого первичного источника.
5. Ароматы разрешено извлекать из официальной техкарты/дегустационной заметки;
   нельзя выдумывать из сорта или цвета.
6. Pairing из официальной рекомендации сохраняется как найденный факт;
   эвристика может быть отдельным предложением только с
   `verification_state='inferred'` и не включается в безопасный bulk apply.
7. Если год/релиз неоднозначен, `scope_hint='unknown'` и требуется выбор
   модератора.
8. Каждый proposal получает стабильный `proposal_key`, чтобы повторный запуск
   был идемпотентен и мог supersede старое предложение.
9. Для числовых параметров использовать value/unit, а не создавать термины
   `oak-12-months`, `oak-18-months`.
10. Для aromas и pairing возвращать нормализуемые отдельные descriptors, а не
    одно предложение со всей исходной фразой.
11. Если источник явно описывает отличие релиза от общей карточки, агент может
    предложить `effect='exclude'` или `effect='replace'`, но такие предложения
    никогда не входят в safe bulk без проверки модератора.

### 7.1. Политика источников

| Источник | Пример | Допуск в safe bulk |
|---|---|---|
| Реестр/сертифицирующая организация | Demeter, EU organic register | да для соответствующей certification |
| Официальный сайт конкурса | страница результата/медали | да для awards |
| Официальная техкарта производителя | PDF/product page | да для состава, метода, aromas и параметров релиза |
| Официальная этикетка/контрэтикетка | evidence photo | да для явно читаемого факта |
| Сайт производителя без техкарты | marketing/product copy | только для прямого утверждения, не для certification без подтверждения |
| Импортёр/дистрибьютор | карточка продукта | review, если нет первичного источника |
| Ритейлер/маркетплейс | коммерческая карточка | review; не safe для спорных claims/awards |
| Профильное медиа/эксперт | статья/обзор | review; допустим как дополнительный source |
| Блог/UGC/агрегатор | неподтверждённый текст | не safe bulk |
| Вывод модели из сорта/стиля | inference | никогда не safe bulk |

Confidence не повышает слабый источник до сильного. Для certification и awards
source policy имеет приоритет над числовым threshold.

### 7.2. Materialization

Research worker после записи immutable report может создать/обновить только
`catalog_knowledge_proposals`. Это не считается canonical catalog write.

- прямой insert в `wine_terms`, `wine_awards`, `wines`, `wine_vintages` запрещён;
- старые pending proposals того же `proposal_key` переводятся в `superseded`,
  если новый отчёт изменил значение;
- принятые/отклонённые решения не переписываются новым report;
- повтор materialization одного report не создаёт дублей.

Тот же materializer обязан поддерживать v3 adapter. Он без нового web research
преобразует уже сохранённые `catalog_values`, `environmental_flags`,
`production_attributes` и `awards` в proposals. Данные, которых в v3 нет в
структурированном виде, не восстанавливаются догадкой из editorial-текста.

### 7.3. Legacy knowledge mining

Существующий каталог является самостоятельным входом proposal pipeline.
Обработка выполняется в два режима.

**Report/dry-run:**

- количество непустых `wines.aroma`, `wines.pairing`, `wines.description`;
- распределение по происхождению вина, если его можно восстановить;
- частотный список descriptors и исходные контексты;
- exact alias matches, новые term candidates, неоднозначности, отрицания;
- предполагаемый scope и доля строк, которую нельзя безопасно разобрать;
- оценка числа assignments до любых записей.

**Proposal materialization:**

1. Сначала читать отдельные поля `aroma` и `pairing`.
2. Затем извлекать явно размеченные части description (`Аромат:`, `Вкус:`,
   `Гастрономия:`, `Метод:` и эквиваленты).
3. Свободный narrative разбирать только как review candidate; модель обязана
   учитывать отрицания и неопределённость.
4. Canonical term не создавать автоматически: сначала exact alias match, затем
   кластер term candidates с частотой и примерами для модератора.
5. Один canonical term объединяет словоформы и синонимы; исходные варианты
   сохраняются как aliases с `source='legacy_backfill'` только после решения.
6. Старый текст и его fingerprint сохраняются в evidence/provenance; публично
   выводится структурированный факт, а не скопированный фрагмент описания.
7. При явно указанном и однозначно найденном vintage предложение связывается с
   релизом; без года предлагается wine scope; конфликт годов требует review.
8. Повторный запуск с той же версией extractor идемпотентен и не расходует
   бюджет внешнего исследования.

Приоритет legacy evidence: отдельное поле → размеченный раздел description →
свободный narrative → inference. Неизвестное происхождение не может стать
`verified` автоматически: proposal получает `source_claimed` до review.

## 8. Atlas admin

Добавить раздел `/admin/atlas-terms` в существующий admin web.

### 8.1. Список

- поиск по title, slug и aliases;
- фильтр `kind`, `status`, наличие статьи, наличие иконки;
- счётчик активных assignments;
- индикатор возможного дубля;
- архивирование вместо физического удаления используемого термина.

### 8.2. Форма

- slug;
- kind;
- `title_ru/en`, `short_ru/en`;
- aliases с locale;
- `icon_key`, fallback category icon;
- `image_url` только когда это необходимо;
- `article_slug`;
- draft/published/archived;
- preview компактного элемента карточки.

### 8.3. Создание из AI proposal

Если alias/slug не сопоставился:

- строка предлагает «Создать термин»;
- форма предзаполнена названием, kind и источником;
- модератор может сначала выбрать найденный существующий термин;
- новый термин по умолчанию `draft`;
- если модератор одновременно подтверждает содержание и публикацию, допустима
  одна review-форма с двумя явными флажками: «создать» и «опубликовать»;
- assignment нельзя публиковать на draft-термин: либо термин публикуется в той же
  транзакции, либо assignment остаётся непубличным.

## 9. Ручной редактор вина и релиза

Использовать один компонент `WineKnowledgeEditor`, меняя target scope.

### 9.1. В редакторе вина

Новый раздел «Характеристики и знания»:

- экологические признаки;
- методы;
- подходы;
- сертификации;
- особенности;
- апелласьон;
- ароматы;
- хорошо сочетается.

Каждая группа — searchable autocomplete по `atlas_terms.kind`. Выбранное
значение показывает источник/статус и удаляется одним действием.

### 9.2. В `AdminWineReleaseManager`

После основных данных и фото добавить «Характеристики релиза» с теми же
группами и явной подписью релиза. Для oak aging и подобных фактов доступны
optional number/unit/detail.

Текущие `grape_composition` и `characteristics jsonb` должны получить нормальные
поля ввода. До завершения миграции редактор dual-write известных ключей:

- `hand_harvest`;
- `oak_aging`, `oak_months`;
- `appellation`.

### 9.3. Ароматы и pairing

- сортировка drag-and-drop или компактными стрелками;
- до 8 приоритетных элементов на карточке, остальные не теряются и доступны
  горизонтальной прокруткой/расширением;
- неизвестный аромат не скрывается, но сначала должен стать Atlas term;
- свободная строка в `wines.aroma/pairing` остаётся только fallback и не является
  основным способом нового ввода.

## 10. Модераторский UX AI proposals

### 10.1. Структура панели

В текущем Workbench добавить блок «Предложения к заполнению»:

1. **Можно применить безопасно** — сильный источник, нет конфликта, scope найден;
2. **Нужно проверить** — средняя уверенность, новый термин, неоднозначный scope;
3. **Конфликты** — источники расходятся или значение отличается от каталога;
4. **Отклонённые** — свёрнутая история с возможностью восстановления.

### 10.2. Строка предложения

Показывает:

- вид и значение;
- текущее значение каталога;
- уровень: «всё вино» или релиз;
- confidence в понятной форме, не как единственное основание;
- источник и кнопку открытия;
- краткую evidence note;
- состояние Atlas match;
- действия: принять, изменить, отклонить.

### 10.3. Минимизация усилий

Основной сценарий:

1. модератор нажимает «Применить безопасные факты»;
2. review sheet показывает количество и группировку по target;
3. по умолчанию выбраны только proposals, удовлетворяющие policy;
4. одним подтверждением они применяются транзакционно;
5. новые/неоднозначные/конфликтные остаются для точечной проверки.

Не включать автоматически в safe bulk:

- inferred pairing;
- новый неопубликованный термин;
- certifications без issuer/официального evidence;
- награды без надёжного source;
- конфликтующие данные;
- неизвестный release scope;
- удаление/замену существующего verified-факта.

### 10.4. Редактирование перед применением

Модератор может изменить:

- Atlas term;
- scope и релиз;
- value/unit/detail;
- award program, level, score, year;
- evidence note;
- статус публикации создаваемого термина.

Редактирование переводит proposal в `edited`, а audit хранит исходный и
применённый payload.

### 10.5. Повторные исследования

- уже принятый идентичный факт показывается как «уже в каталоге»;
- новый источник может быть прикреплён к provenance без дублирования assignment;
- противоречие создаёт conflict, а не молча перезаписывает verified data;
- отклонённое предложение того же факта не предлагается снова без нового
  существенного evidence; оно помечается как повтор.

## 11. RPC и транзакционные операции

Прямые записи Flutter-клиента в canonical tables запретить. Нужны минимум:

- `admin_upsert_atlas_term(...)`;
- `admin_upsert_atlas_term_alias(...)`;
- `admin_upsert_wine_term_assignment(...)`;
- `admin_archive_wine_term_assignment(...)`;
- `admin_upsert_wine_award(...)`;
- `admin_archive_wine_award(...)`;
- `admin_apply_catalog_knowledge_proposals(p_proposal_ids uuid[],
  p_edits jsonb, p_request_id uuid)`;
- `admin_reject_catalog_knowledge_proposal(...)`;
- `admin_restore_catalog_knowledge_proposal(...)`.

`admin_apply_catalog_knowledge_proposals` обязан:

1. проверить роль catalog admin;
2. заблокировать proposals `for update`;
3. проверить, что они pending и относятся к ожидаемому wine/submission;
4. проверить связь release → wine;
5. проверить published term либо создать/опубликовать его по явному edit;
6. upsert assignment/award без дублей;
7. записать decision и audit;
8. завершить всё одной транзакцией;
9. безопасно вернуть прежний результат при повторе `request_id`.

Возвращаемый summary:

```json
{
  "applied": 7,
  "edited": 1,
  "skipped_already_applied": 2,
  "conflicts": 0,
  "created_terms": 1,
  "created_awards": 2
}
```

## 12. Публичное чтение и карточка

### 12.1. Read model

Расширить `WineCardSourceData`:

- `termAssignments`;
- Atlas term metadata;
- существующие awards;
- unavailable parts, чтобы ошибка knowledge source не ломала hero.

Repository делает один согласованный запрос/RPC для публичных verified
assignments и published terms либо параллельные независимые запросы с partial
failure policy.

Обязательные индексы: assignments по `wine_id`, `wine_vintage_id`, `term_id`,
`status`; terms по `kind/status/slug`; aliases по `normalized_alias`; proposals
по `report_id`, `wine_id`, `moderation_status`. Для типичной карточки knowledge
read не должен создавать N+1 запрос по каждому термину.

### 12.2. Правила scope

В состоянии «Все»:

- показывать wine-level assignments;
- награды — по принятому контракту Session 2.5: все релизы и wine-level;
- release-only методы/ароматы не выдавать за свойства всего вина.

При выбранном релизе:

- показывать wine-level устойчивые facts;
- добавлять release-level facts;
- сначала удалять из effective set wine-level terms, для которых существует
  release-level `effect='exclude'`;
- release-level assignment того же singleton-класса перекрывает wine-level;
- multi-value aromas/pairings объединяются без дублей, release-specific идут
  первыми;
- награды фильтруются по выбранному релизу плюс wine-level fallback.

### 12.3. Singleton и multi-value

Singleton-классы конфигурируются metadata/taxonomy policy, например основной
sparkling method или appellation. Aromas, pairings, certifications и features —
multi-value. Release `replace` singleton реализуется effective merge, а не
удалением общего назначения. Для multi-value используется точечный `exclude`.

### 12.4. Atlas interaction

Тап по элементу:

- всегда открывает короткое пояснение `short_*`;
- при published `article_slug` доступно «Читать в Атласе»;
- отсутствие статьи не скрывает факт;
- evidence/source показывается только в экспертном режиме.

## 13. Dual-read, dual-write и backfill

### 13.1. Приоритет чтения

1. verified structured assignment;
2. release `characteristics` для известных legacy-ключей;
3. четыре `wines.is_*` флага;
4. `wines.aroma` / `wines.pairing`;
5. существующая pairing-эвристика только при отсутствии явных данных.

### 13.2. Dual-write

На переходе:

- изменение четырёх eco/style терминов синхронизирует соответствующие `is_*`;
- ручной сбор/oak/appellation релиза синхронизируются с известными ключами
  `characteristics`;
- новые structured aroma/pairing могут обновлять legacy-строку для старых APK,
  но legacy-строка не должна становиться источником истины;
- structured awards не пишутся в `wines.awards`.

Dual-write реализуется серверным RPC, а не двумя несвязанными Flutter-вызовами.

### 13.3. Backfill

1. Выполнить corpus profiling без записей и зафиксировать baseline/statistics.
2. seed Atlas для существующих semantic assets и четырёх флагов.
3. Сформировать clusters ароматов/pairing/других descriptors из legacy-полей,
   показать частоту, aliases и примеры модератору.
4. true legacy eco/style flags → assignments `source='legacy_backfill'`.
5. известные release characteristics → release assignments.
6. aroma/pairing строки с exact Atlas alias → proposals, а не прямые writes.
7. description findings и неоднозначный текст → review proposals; исходный текст
   оставить legacy и не терять.
8. v3 reports → proposals через adapter без повторного AI/web research.
9. apply выполняется только общим moderation RPC после dry-run/sample QA.
10. backfill идемпотентен, версионирует extractor и имеет dry-run статистику.

Удаление legacy разрешается только отдельным ТЗ после метрик использования и
минимум одного стабильного store release.

## 14. RLS и безопасность

- anon/authenticated читают только verified assignments опубликованных Atlas
  terms и публично видимых wine/release;
- draft/source_claimed/archived видит только catalog admin;
- proposals, evidence и moderation events не публичны;
- authenticated не получает direct insert/update/delete на Atlas, assignments,
  awards и proposals;
- admin RPC повторно проверяет `is_catalog_admin(auth.uid())`;
- service role может писать reports/proposals, но research worker не получает
  canonical apply RPC;
- URLs сохраняются как evidence, но UI открывает их безопасно и не выполняет
  содержимое страницы;
- все изменения имеют actor, timestamp и request id;
- hard delete терминов/assignments/awards из UI не используется.

## 15. Seed taxonomy

Первый обязательный seed должен покрыть используемые assets:

- style: Pet-Nat;
- eco/approach: organic, biodynamic, natural;
- methods: traditional method, Charmat, ancestral, oak aging, hand harvest;
- certification: EU Organic, Demeter;
- feature: Vegan;
- appellation: generic ЗГУ/ЗНМП только как taxonomy entries, конкретные зоны —
  отдельные terms;
- 8 aroma assets текущей карточки;
- 8 pairing assets текущей карточки;
- базовые award programs добавляются по мере подтверждённых данных, а не
  выдумываются seed-ом.

Каждый seed term имеет стабильный slug/icon_key и aliases RU/EN. Seed не должен
перезаписывать отредактированный модератором текст при повторном применении.

После baseline seed Atlas расширяется не сырым списком фраз, а утверждёнными
кластерами legacy corpus. Частота помогает приоритизировать термин, но сама по
себе не является доказательством корректности или основанием публикации.

## 16. Flutter/admin архитектура

Предлагаемая структура:

```text
lib/features/wine_knowledge/
  domain/
    atlas_term.dart
    wine_term_assignment.dart
    catalog_knowledge_proposal.dart
  data/
    wine_knowledge_repository.dart
  application/
    wine_knowledge_providers.dart
    moderation_policy.dart
  presentation/
    admin_atlas_terms_screen.dart
    wine_knowledge_editor.dart
    research_knowledge_proposals_panel.dart
    knowledge_proposal_review_sheet.dart
```

Awards могут остаться в `features/wines/domain`, но CRUD и proposal apply должны
использовать общий moderation service/repository.

Не помещать новую логику в уже перегруженный `receipt_controller.dart` или
монолит Workbench. Workbench подключает вынесенную панель и callbacks.

Все новые пользовательские строки карточки и Atlas добавляются через ARB и
`flutter gen-l10n`. Admin-строки также должны быть вынесены в l10n в рамках
нового модуля, даже если старые admin-экраны ещё содержат hardcode. Интерактивные
иконки имеют semantic label/tooltip; confidence не кодируется одним цветом.

## 17. Analytics и операционные метрики

Admin events без пользовательской PII:

- `catalog_knowledge_proposals_viewed`;
- `catalog_knowledge_safe_apply_opened`;
- `catalog_knowledge_bulk_applied`;
- `catalog_knowledge_proposal_edited`;
- `catalog_knowledge_proposal_rejected`;
- `atlas_term_created_from_research`;
- `catalog_knowledge_conflict_seen`.

Поля: count, kinds, scope distribution, report schema, duration, result. Не
отправлять evidence URL в AppMetrica.

Операционные KPI:

- медианное число кликов на один подтверждённый факт;
- доля safe bulk / manual review;
- доля edited/rejected;
- число созданных дублей Atlas terms (целевое 0);
- conflict rate;
- процент карточек с aroma, methods, certifications, awards;
- RPC error/idempotent replay rate.

## 18. Этапы реализации и commit checkpoints

### Этап 0. Baseline и документация

- inventory production counts;
- backup + SHA-256;
- фиксация report v3 samples;
- commit только документации.

### Этап 1. Taxonomy foundation

- миграции Atlas aliases/status;
- `atlas_term_relations` и cycle-safe hierarchy RPC;
- расширение `wine_terms`;
- RLS/read RPC;
- seed и dry-run backfill;
- domain/repository tests.

Контрольный коммит: `feat: add wine knowledge taxonomy foundation`.

### Этап 2. Manual admin CRUD

- Atlas admin;
- `WineKnowledgeEditor` для wine/release;
- awards CRUD;
- dual-write known legacy values.

Контрольный коммит: `feat: add wine knowledge admin editors`.

### Этап 3A-0. Legacy knowledge mining

- production read-only profiling `aroma/pairing/description`;
- origin coverage и baseline report;
- deterministic splitter/normalizer с версией extractor;
- term candidate clustering, aliases, frequency и contexts;
- dry-run proposal statistics;
- v3 report adapter без повторного исследования;
- sample QA до materialization.

Контрольный коммит: `feat: profile legacy wine knowledge`.

### Этап 3A. AI proposal contract

- report v4 schema;
- единая materialization для v4, v3 и legacy origins;
- proposal repository/panel;
- source/confidence/conflict visualization.

Контрольный коммит: `feat: prepare catalog knowledge proposals`.

### Этап 3B. Moderation apply

- transactional RPC;
- safe bulk policy;
- edit/reject/restore;
- include/exclude/replace и release effective preview;
- audit/idempotency;
- AppMetrica admin events.

Контрольный коммит: `feat: moderate and apply catalog knowledge`.

### Этап 4. Card and Atlas read path

- structured terms in `WineCardSourceData`;
- release-aware merge;
- Atlas popover/article link;
- expert-mode evidence;
- legacy fallback tests.

Контрольный коммит: `feat: render structured wine knowledge`.

### Этап 5. Production rollout and backfill

- production backup;
- migration dry-run via rollback;
- apply + PostgREST reload/restart;
- seed/backfill;
- reference wines;
- mobile/web smoke;
- handoff/status docs.

## 19. Автоматические тесты

### 19.1. Database

- не-admin не может писать;
- release другого вина отклоняется;
- повтор request id не создаёт дубль;
- partial unique indexes работают для wine/release scopes;
- bulk apply атомарен;
- draft term не утечёт в public read;
- archived assignment не читается;
- audit создаётся для каждого решения;
- dual-write eco/characteristics согласован;
- backfill повторяем без изменений.
- legacy extractor повторяем без дублей proposals;
- изменение legacy source supersede только pending proposals;
- один legacy term candidate не создаёт несколько canonical terms;
- release exclude удаляет только наследуемый term выбранного релиза;
- release singleton replace не изменяет wine-level assignment;
- hierarchy отклоняет self-link, cross-kind link и транзитивный цикл;
- один child поддерживает несколько parents без дублирования assignment;

### 19.2. Domain/repository

- report v3 продолжает читаться;
- v3 adapter не запускает новый research job и сохраняет evidence ids;
- v4 proposal parse всех kinds;
- legacy aroma/pairing splitter учитывает aliases, разделители и отрицания;
- unknown kind не ломает весь отчёт;
- partial source failure не ломает карточку;
- selected release merge корректен;
- general parent подавляется specific child только по display policy;
- exact alias match и duplicate protection;
- award year не смешивается с vintage.

### 19.3. Widget

- safe/review/conflict groups;
- 0/1/many proposals;
- новый term flow;
- edit scope/value;
- review sheet и summary;
- manual editor на 320/390/desktop;
- 0/1/8/many aromas/pairings;
- Atlas term without article;
- expert evidence state.

## 20. Ручная QA-матрица

Минимум два эталонных вина:

1. тихое красное: два релиза, oak months, hand harvest, appellation, 5 aromas,
   4 pairings, 3 awards;
2. игристое: NV + vintage, traditional/Charmat distinction, certification,
   conflicting source и award points.

Проверить:

- AI report → proposal materialization;
- v3 report → proposals без дополнительных затрат;
- legacy corpus dry-run → clusters → proposals;
- проверка минимум выборки exact/review/rejected legacy findings;
- safe bulk;
- точечное редактирование;
- reject/restore;
- создание нового term;
- wine/release scope;
- повторный research;
- старый APK видит legacy fallback;
- новый APK видит structured data;
- «Все» не показывает release-only fact как общий;
- конкретный релиз показывает свои отличия;
- Atlas popover и article link;
- web admin и Android APK;
- отсутствие данных не создаёт пустых блоков.

## 21. Production deployment

Для каждой миграционной пачки:

1. review SQL и список затрагиваемых объектов;
2. production backup + SHA-256;
3. dry-run в транзакции с `rollback`;
4. apply;
5. post-check counts, constraints, RLS, grants, RPC signatures;
6. `NOTIFY pgrst, 'reload schema'` и restart `supabase-rest`, если требуется;
7. seed/backfill сначала в dry-run/report mode;
8. smoke admin read/write и public read;
9. фиксация backup, migration hashes и результата в handoff.

Rollback должен архивировать/отключать новый read path без удаления уже
подтверждённых фактов. Destructive down-migration для production не готовится.

## 22. Критерии приёмки

Срез завершён только когда:

- модератор может вручную завести и отредактировать Atlas term;
- один и тот же term назначается вину или конкретному релизу;
- AI v4 создаёт предложения с evidence и confidence;
- существующие v3 reports создают proposals без повторного исследования;
- legacy aroma/pairing/description проходят dry-run, clustering и общий
  moderation pipeline;
- safe bulk применяет только policy-safe proposals;
- edit/reject/restore работают и аудируются;
- повторный apply идемпотентен;
- методы, подходы, certifications, features, appellation, aromas и pairings
  появляются в карточке из structured data;
- awards создаются/редактируются и корректно фильтруются по релизу;
- release-level include/exclude/replace корректно формирует effective facts, не
  разрушая общие данные вина;
- draft/unverified данные не видны публично;
- старые eco/aroma/pairing/characteristics продолжают работать fallback-путём;
- два эталонных вина заполнены через UI, без ручного SQL;
- автоматические тесты и ручная QA-матрица пройдены;
- production migration/backfill/handoff документированы.

## 23. Утверждённые архитектурные решения

Владелец продукта подтвердил 03.08.2026 все пять решений без изменений:

1. `source_claimed` показывать только в экспертном режиме.
2. Safe bulk применять только для официального/сильного источника при
   confidence ≥ 0.85 и отсутствии конфликтов; сила источника важнее числовой
   оценки confidence.
3. Новый Atlas term по умолчанию создавать в статусе `draft`.
4. Structured aroma/pairing один release cycle записывать также в legacy-строки
   для совместимости со старыми APK.
5. Release-level assignments реализовать расширением `wine_terms`, без второй
   параллельной таблицы.
6. Старые `wines.aroma`, `wines.pairing` и `wines.description` использовать как
   корпус знаний через proposals, а не прямой backfill в public assignments.
7. Существующие `research-report.v3` материализовать повторно без нового
   оплачиваемого web research.
8. Atlas seed расширять утверждёнными кластерами legacy corpus; словоформы и
   синонимы хранить aliases, не отдельными canonical terms.
9. Релиз может дополнять, заменять singleton или исключать наследуемый
   multi-value term; effective merge обязан сохранять wine-level source data.
10. Ручной редактор остаётся fallback, основной массовый путь — AI/legacy
    proposal → минимальная модерация → transactional apply.
11. Atlas использует general → specific hierarchy; в одном evidence fragment
    specific term подавляет отображение своего general parent.
12. `Слива` и `Чёрная слива` — разные canonical aroma terms; aliases `Слива` и
    `Plum` не принадлежат term `Чёрная слива`.

С этого подтверждения документ является source of truth для реализации.

## 24. Связанные документы

- `docs/session_2_5_release_experience_and_wine_card_tz_2026_07_29.md`;
- `docs/session_2_5_wine_card_flutter_implementation_plan_2026_07_30.md`;
- `docs/session_2_5_wine_card_visual_master_2026_07_30.md`;
- `docs/session_2_5_legacy_knowledge_profile_2026_08_03.md`;
- `docs/session_2_5_legacy_knowledge_extractor_v1_results_2026_08_03.md`;
- `docs/session_2_5_atlas_seed_review_v1_2026_08_03.md`;
- `docs/ai_catalog_research_copilot_tz_2026_07_12.md`;
- `docs/wine_production_attributes_certifications_tz_2026_07_12.md`;
- `docs/release_experience_and_wine_card_roadmap_2026_07_24.md`;
- `supabase/migrations/202607300200_create_wine_card_knowledge_tables.sql`.
