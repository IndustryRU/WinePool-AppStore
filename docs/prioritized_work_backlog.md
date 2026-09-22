# Prioritized Work Backlog

Последнее обновление: 23.06.2026 (добавлен пункт 15 `WinePool Cellar Pro`; полная переоценка всех старых приоритетов не проводилась)

> **Статус с 25.08.2026: historical / superseded as execution priority.**
>
> Этот файл сохраняется как история governance, receipt и раннего product backlog.
> Он больше не определяет текущую очередь работ после релиза 1.1.0.
> Активный source of truth: [post_release_1_1_0_execution_roadmap_2026_08_25.md](/R:/Flutter/Project/winepool_final/docs/post_release_1_1_0_execution_roadmap_2026_08_25.md).
> Пункты отсюда выполняются только если они включены в H0–H6 либо являются подтверждённым production defect/security blocker.

## Зачем Нужен Этот Файл

Этот файл сводит в один backlog работы, которые вытекают из уже проанализированных документов в `docs/`.

Он нужен как bridge между:

- source-of-truth документами
- roadmap/spec/handoff слоями
- практическим выбором следующего шага

Здесь собраны не только code/work items, но и document-sync работы, если они реально влияют на безопасность, governance или product clarity.

## Как Читать Ранжирование

Для каждого workstream фиксируем четыре характеристики:

- `Delivery priority`:
  - `Now` = лучше делать в ближайших сессиях
  - `Next` = логичный следующий слой после стабилизации текущих дыр
  - `Later` = важный, но не немедленный следующий шаг
  - `Deferred` = осознанно не смешивать с ближайшим delivery
  - `Completed` = уже закрыто; временно держим в файле как недавний baseline, чтобы не потерять sequencing context
- `Strategic value`:
  - `High`
  - `Medium`
- `Complexity`:
  - `S`
  - `M`
  - `L`
  - `XL`
- `Type`:
  - `governance`
  - `product/UX`
  - `data/calibration`
  - `docs`
  - `platform`

> Progress note, 22.04.2026:
> официальный следующий порядок теперь уже зафиксирован отдельно в
> [next_iterations_execution_order.md](/R:/Flutter/Project/winepool_final/docs/next_iterations_execution_order.md)
> и выглядит так:
> `Track B redesign -> Abuse Controls Phase 1 -> Abuse Controls Phase 2 -> optional instrumentation`.
>
> Поэтому backlog ниже нужно читать так:
> - как широкий ranked backlog по всем направлениям;
> - но не как более сильный приоритетный документ, чем новый execution-order.
>
> Практически это значит:
> - `Proposal path из buyer/business draft в canonical catalog` уже больше не `Next`, а landed baseline;
> - live calibration / receipt UX polish / alias coverage всё ещё важны,
>   но теперь идут как parallel receipt-quality cluster, а не как главный ближайший порядок поверх Track B.

## Ranked Backlog

### 1. Receipt buyer/review split и owner-scoped draft flow

- `Delivery priority`: `Completed`
- `Strategic value`: `High`
- `Complexity`: `M`
- `Type`: `governance + routing + UX`
- `Статус`: закрыто 15.04.2026
- Что входит:
  - вернуть buyer доступ к обычным receipt routes
  - оставить `/receipt-alias-memory` и похожие review surfaces за privileged capability
  - убрать зависимость owner draft-link flow от `canReviewReceipts`
  - привести CTA на `DraftWineDetailsScreen` к роли пользователя
- Почему это высоко:
  - здесь сейчас главный contract mismatch между docs и runtime policy
  - это влияет и на UX, и на permission correctness
- Основные источники:
  - [receipt_buyer_admin_governance_spec.md](/R:/Flutter/Project/winepool_final/docs/receipt_buyer_admin_governance_spec.md)
  - [role_business_policy_draft.md](/R:/Flutter/Project/winepool_final/docs/role_business_policy_draft.md)
  - [role_business_write_surface_inventory.md](/R:/Flutter/Project/winepool_final/docs/role_business_write_surface_inventory.md)

### 2. Phase 5 stabilization: alias/merge/rollback regression smoke

- `Delivery priority`: `Completed`
- `Strategic value`: `High`
- `Complexity`: `M`
- `Type`: `governance + verification`
- `Статус`: закрыто 15.04.2026 как `docs + local smoke + one concrete consistency fix`; live rollback smoke в writable среде остаётся отдельным future check
- Что входит:
  - прогнать admin alias CRUD
  - прогнать merge пары виноделен
  - проверить rollback/unmerge для новых merge
  - проверить post-merge navigation и read-side consistency
  - убедиться, что seller/business surfaces не регрессировали
- Почему это высоко:
  - core merge tooling уже landed и теперь опаснее всего скрытая регрессия, а не отсутствие функционала
- Основные источники:
  - [role_business_architecture_implementation_plan.md](/R:/Flutter/Project/winepool_final/docs/role_business_architecture_implementation_plan.md)
  - [role_business_policy_draft.md](/R:/Flutter/Project/winepool_final/docs/role_business_policy_draft.md)
  - [winery_merge_admin_spec.md](/R:/Flutter/Project/winepool_final/docs/winery_merge_admin_spec.md)

### 3. Doc sync для live Phase 5 и admin operational контекста

- `Delivery priority`: `Completed`
- `Strategic value`: `High`
- `Complexity`: `S`
- `Type`: `docs`
- `Статус`: закрыто 15.04.2026
- Что входит:
  - держать `implementation plan`, `policy draft`, `write surface inventory` в одном статусе
  - точечно дотянуть [administrator_guide.md](/R:/Flutter/Project/winepool_final/docs/administrator_guide.md) под merge history / rollback / operational merge UX
  - не давать historical handoff/spec файлам снова становиться source of truth
- Почему это высоко:
  - сейчас docs уже сложные, и главная угроза не отсутствие файла, а расхождение между живыми truth-документами
- Основные источники:
  - [documentation_map.md](/R:/Flutter/Project/winepool_final/docs/documentation_map.md)
  - [role_business_architecture_implementation_plan.md](/R:/Flutter/Project/winepool_final/docs/role_business_architecture_implementation_plan.md)
  - [administrator_guide.md](/R:/Flutter/Project/winepool_final/docs/administrator_guide.md)

### 4. Suspect price trust redesign

- `Delivery priority`: `Completed`
- `Strategic value`: `High`
- `Complexity`: `L`
- `Type`: `governance + data model + UX`
- `Статус`: закрыто 15.04.2026; migration `20260415_split_owner_receipt_price_confirmation.sql` применена на target DB
- Что входит:
  - развести personal confirmation и platform-trusted moderation
  - убрать семантическую путаницу вокруг `verified`
  - переименовать user actions в более честные owner-scoped действия
  - обновить SQL/RPC и wording
- Почему это высоко:
  - текущая модель слишком легко делает личное подтверждение похожим на платформенную истину
- Основные источники:
  - [receipt_buyer_admin_governance_spec.md](/R:/Flutter/Project/winepool_final/docs/receipt_buyer_admin_governance_spec.md)
  - [role_business_policy_draft.md](/R:/Flutter/Project/winepool_final/docs/role_business_policy_draft.md)
  - [role_business_write_surface_inventory.md](/R:/Flutter/Project/winepool_final/docs/role_business_write_surface_inventory.md)

### 5. Live calibration receipt matcher на реальных чеках

- `Delivery priority`: `Now`
- `Strategic value`: `High`
- `Complexity`: `M`
- `Type`: `data/calibration`
- `Статус`: in progress; preparatory export slice landed 15.04.2026, а 17.04.2026 receipt review уже получил honest missing-catalog UX baseline; следующий практический шаг — продолжать live calibration и закрывать повторяющиеся catalog gaps
- Что входит:
  - ручной bug bash по реальным чекам
  - сбор спорных кейсов
  - контроль counters `exact memory / family hits / false-promote risk`
  - проверка alias-heavy и same-winery кейсов
- Что уже подготовлено:
  - `Копировать calibration` теперь включает `strong_score / review_score` у `top-3 suggestions`
  - snapshot показывает `same_winery_fallback_shown` и `reserve_tie_break_used`
  - summary уже считает fallback/reserve counters alongside family-memory counters
  - review sheet уже честно показывает `Похоже, этого вина пока нет в каталоге`
  - для таких кейсов главный CTA уже смещён в `Создать черновик`, а похожие позиции спрятаны за `Показать похожие`
  - в шапке review sheet видны полные store details и общая сумма чека, так что live bug bash проще сверять с бумажным чеком
- Почему это now:
  - foundation уже есть, теперь нужна живая калибровка качества
- Основные источники:
  - [receipt_sprint_status.md](/R:/Flutter/Project/winepool_final/docs/receipt_sprint_status.md)
  - [receipt_bug_bash_calibration_checklist.md](/R:/Flutter/Project/winepool_final/docs/receipt_bug_bash_calibration_checklist.md)
  - [receipt_grape_detection_spec.md](/R:/Flutter/Project/winepool_final/docs/receipt_grape_detection_spec.md)

### 6. Receipt review UX polish: CTA, same-winery fallback, weak flows

- `Delivery priority`: `Now`
- `Strategic value`: `High`
- `Complexity`: `M`
- `Type`: `product/UX`
- `Статус`: mostly closed for MVP as of 17.04.2026; honest missing-catalog UX, compact action layout, show/hide similar toggle и safer same-winery fallback copy уже landed
- Что входит:
  - удерживать review sheet честным на неполном catalog
  - улучшать explanations для weak OCR / weak grape flows только по реальным calibration кейсам
  - не давать слабым lookalike-card спорить с основным CTA `Создать черновик`
- Почему это now:
  - это уже не архитектурный blocker и не отдельный feature-track; теперь это supporting polish вокруг live calibration и catalog coverage
- Основные источники:
  - [receipt_follow_up_backlog.md](/R:/Flutter/Project/winepool_final/docs/receipt_follow_up_backlog.md)
  - [receipt_sprint_status.md](/R:/Flutter/Project/winepool_final/docs/receipt_sprint_status.md)
  - [qr_receipt_draft_handoff_2026_03_24.md](/R:/Flutter/Project/winepool_final/docs/qr_receipt_draft_handoff_2026_03_24.md)

### 7. Raw receipt replay persistence

- `Delivery priority`: `Completed`
- `Strategic value`: `High`
- `Complexity`: `L`
- `Type`: `platform + product`
- `Статус`: закрыто 16.04.2026; локальный pending receipt review теперь сохраняется после первого scan, повторно открывается по `fn/fd/fp` без нового QR/API вызова и доступен из `receipt history`
- Что входит:
  - локально сохранять raw receipt/review context
  - уметь повторно открывать уже отсканированный чек без нового QR/API вызова
  - сделать это основой для retention loop и спокойного ручного разбора
- Что уже вошло в baseline:
  - auto-save pending review context сразу после успешного scan
  - реальная кнопка `Позже` в review sheet
  - reopen same QR по локальному draft вместо нового API-запроса
  - pending receipts в истории со статусом `требует разбора`
  - мягкое in-app reminder покрытие в profile flow:
    - reminder-card для неразобранных чеков
    - count badge у `История чеков`
    - локальный snooze `Позже на сегодня`
- Почему это было важно:
  - это один из самых практичных UX-улучшений после стабилизации текущего flow
- Основные источники:
  - [receipt_follow_up_backlog.md](/R:/Flutter/Project/winepool_final/docs/receipt_follow_up_backlog.md)
  - [draft_resolution_test_checkpoint.md](/R:/Flutter/Project/winepool_final/docs/draft_resolution_test_checkpoint.md)
  - [receipt_sprint_status.md](/R:/Flutter/Project/winepool_final/docs/receipt_sprint_status.md)

### 8. System-wide audit winery alias coverage

- `Delivery priority`: `Next`
- `Strategic value`: `Medium`
- `Complexity`: `M`
- `Type`: `data/calibration`
- Что входит:
  - пройтись по live `winery_aliases`
  - проверить латиницу, кириллицу, транслит и retail forms
  - найти системные gaps до новых parser-regressions
  - использовать read-only Supabase audit как основной безопасный инструмент
- Почему это next:
  - alias layer уже работает, теперь важнее coverage quality, чем новая механика
- Основные источники:
  - [receipt_follow_up_backlog.md](/R:/Flutter/Project/winepool_final/docs/receipt_follow_up_backlog.md)
  - [winery_alias_layer_spec.md](/R:/Flutter/Project/winepool_final/docs/winery_alias_layer_spec.md)
  - [SELF_DEBUG_GUIDE.md](/R:/Flutter/Project/winepool_final/docs/SELF_DEBUG_GUIDE.md)

### 9. Proposal path из buyer/business draft в canonical catalog

- `Delivery priority`: `Completed`
- `Strategic value`: `High`
- `Complexity`: `L`
- `Type`: `governance + product`
- `Статус`: moderation baseline landed 20-22.04.2026; non-admin submit/resubmit, admin queue/review, decision loop и read-side return уже реализованы
- Что входит:
  - убрать ожидание direct promote/create для non-admin
  - дать buyer/business честный CTA `Отправить на проверку`
  - сделать proposal/read-side/moderation path для новых wine/winery из draft
- Почему это было важно:
  - docs уже фиксировали boundary, и product-path действительно требовал полного moderated варианта
- Основные источники:
  - [receipt_buyer_admin_governance_spec.md](/R:/Flutter/Project/winepool_final/docs/receipt_buyer_admin_governance_spec.md)
  - [role_business_policy_draft.md](/R:/Flutter/Project/winepool_final/docs/role_business_policy_draft.md)
  - [role_business_write_surface_inventory.md](/R:/Flutter/Project/winepool_final/docs/role_business_write_surface_inventory.md)
  - [receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md](/R:/Flutter/Project/winepool_final/docs/receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md)

## Current Execution-Order Overlay

Этот блок нужен, чтобы backlog не расходился с уже утверждённым execution-order.

Исторический ближайший порядок на момент обновления 23.06.2026 был таким:

1. `Track B full draft details redesign`
2. `Abuse Controls Phase 1: draft-level guardrails`
3. `Abuse Controls Phase 2: user-level restrictions`
4. `Optional instrumentation / analytics layer`

Главные документы этого порядка:

- [next_iterations_execution_order.md](/R:/Flutter/Project/winepool_final/docs/next_iterations_execution_order.md)
- [receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md](/R:/Flutter/Project/winepool_final/docs/receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md)
- [draft_wine_details_redesign_tz_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/draft_wine_details_redesign_tz_2026_04_21.md)
- [draft_catalog_moderation_abuse_controls_tz_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/draft_catalog_moderation_abuse_controls_tz_2026_04_21.md)

### 10. Broader moderated catalog collaboration

- `Delivery priority`: `Later`
- `Strategic value`: `High`
- `Complexity`: `L`
- `Type`: `governance`
- Что входит:
  - расширить proposal model beyond narrow `winery_alias_add` / `winery_edit`
  - решить, какие ещё canonical changes пойдут через moderated path
  - заранее фиксировать tier и backend scope для новых proposal flows
- Почему later:
  - узкий moderated baseline уже есть, но безопаснее сначала стабилизировать текущий Phase 5 и receipt governance
- Основные источники:
  - [role_business_architecture_implementation_plan.md](/R:/Flutter/Project/winepool_final/docs/role_business_architecture_implementation_plan.md)
  - [role_business_policy_draft.md](/R:/Flutter/Project/winepool_final/docs/role_business_policy_draft.md)

### 11. Wine-level duplicate merge после winery merge

- `Delivery priority`: `Later`
- `Strategic value`: `High`
- `Complexity`: `XL`
- `Type`: `governance + data integrity`
- Что входит:
  - read-side detection дублей `wines` внутри одной canonical winery
  - admin-only wine merge RPC
  - audit + rollback
  - безопасная работа с downstream links: `offers`, `cellar`, `orders`, `receipt`, `draft references`
- Почему later:
  - это большой и рискованный следующий слой, который не стоит смешивать с текущей стабилизацией winery merge
- Основные источники:
  - [winery_merge_admin_spec.md](/R:/Flutter/Project/winepool_final/docs/winery_merge_admin_spec.md)
  - [role_business_architecture_implementation_plan.md](/R:/Flutter/Project/winepool_final/docs/role_business_architecture_implementation_plan.md)

### 12. Seller home -> полноценный business console

- `Delivery priority`: `Later`
- `Strategic value`: `Medium`
- `Complexity`: `L`
- `Type`: `product + platform`
- Что входит:
  - убрать transitional seller shell
  - довести business profile / locations / business-type sections
  - сделать seller surface более явно business-aware
- Почему later:
  - governance baseline уже задан, но это не такой немедленный integrity risk, как receipt boundary или merge regression
- Основные источники:
  - [role_business_architecture_spec.md](/R:/Flutter/Project/winepool_final/docs/role_business_architecture_spec.md)
  - [role_business_architecture_implementation_plan.md](/R:/Flutter/Project/winepool_final/docs/role_business_architecture_implementation_plan.md)

### 13. Fine-grained admin/business governance extensions

- `Delivery priority`: `Deferred`
- `Strategic value`: `Medium`
- `Complexity`: `XL`
- `Type`: `platform`
- Что входит:
  - granular admin grants
  - несколько managed wineries на один business
  - delegated managers
  - more granular business permissions
- Почему deferred:
  - docs прямо показывают, что baseline ещё лучше не усложнять premature RBAC-слоем
- Основные источники:
  - [role_business_architecture_spec.md](/R:/Flutter/Project/winepool_final/docs/role_business_architecture_spec.md)
  - [role_business_architecture_implementation_plan.md](/R:/Flutter/Project/winepool_final/docs/role_business_architecture_implementation_plan.md)
  - [administrator_guide.md](/R:/Flutter/Project/winepool_final/docs/administrator_guide.md)

### 14. Batch import каталога через normalization staging

- `Delivery priority`: `Later` (активируется триггерами, не календарём)
- `Strategic value`: `High`
- `Complexity`: `L`
- `Type`: `data/calibration + governance + platform`
- Что входит:
  - staging-таблицы `catalog_import_batches` / `catalog_import_rows`
  - серверный alias-aware матчинг каждой строки (barcode -> aliases -> normalized name -> fuzzy)
  - классификация `auto_link / create_new / needs_review / invalid` с explainable сигналами
  - admin-разбор `needs_review` по паттернам workbench
  - атомарное применение батча с записью aliases + `catalog_normalization_decisions` и откатом по `batch_id`
  - вывод из эксплуатации CSV-импорта первого поколения (кроме справочника сортов)
- Почему later:
  - без конкретного источника половина решений — гадание; первое поколение импорта покрывает микро-догрузки
- Триггеры активации и архитектурная рамка:
  - [batch_catalog_import_staging_concept_2026_06_12.md](/R:/Flutter/Project/winepool_final/docs/batch_catalog_import_staging_concept_2026_06_12.md)
- Основные источники:
  - [catalog_normalization_moderation_tz_2026_05_11.md](/R:/Flutter/Project/winepool_final/docs/catalog_normalization_moderation_tz_2026_05_11.md)
  - [add_bottle_universal_flow_tz_2026_06_12.md](/R:/Flutter/Project/winepool_final/docs/add_bottle_universal_flow_tz_2026_06_12.md)
  - [PARSER_INDEX.md](/R:/Flutter/Project/winepool_final/PARSER_INDEX.md)

### 15. WinePool Cellar Pro: учет расположения вина и платный погреб

- `Delivery priority`: `Next` для P0/P1, `Later` для Pro/P3
- `Strategic value`: `High`
- `Complexity`: `M` для P1, `L/XL` для QR/NFC и full Pro
- `Type`: `product/UX + platform + monetization`
- Что входит:
  - усилить SEO/лендинг под запросы `учет личного запаса вина`, `учет и расположение вина`, `винный погребок`;
  - добавить fake-door/waitlist и in-app teaser `Погреб Pro`;
  - построить приватные зоны хранения без координат поверх текущего `user_storage`, не ломая базовый погребок;
  - добавить `user_cellars`, `cellar_locations`, `cellar_stock_lots`, `cellar_inventory_events`;
  - дать пользователю размещение, перемещение, фильтр `Без места`, журнал движения и более точное открытие бутылки;
  - позже развить Pro: экспорт, напоминания, совместный доступ, tablet/desktop layout, QR/NFC и точные bottle instances.
- Почему next:
  - поисковый спрос уже пришёл именно по утилитарной боли учета коллекции;
  - WinePool уже имеет сильный ingestion/data foundation: чеки, add-bottle, личный погребок, карту покупок, цены и модерацию отсутствующих вин;
  - P0/P1 можно делать без вмешательства в текущий official moderation execution-order;
  - это один из первых естественных consumer-side paid layers, который не отбирает базовый функционал.
- Что не делать сразу:
  - NFC/QR exact bottle system;
  - 3D-схему погреба;
  - отображение мест хранения на геокарте;
  - полноценный paywall до legal/store review и проверки спроса;
  - B2B-склад или учет продаж алкоголя.
- Основные источники:
  - [wine_cellar_pro_tz_2026_06_23.md](/R:/Flutter/Project/winepool_final/docs/wine_cellar_pro_tz_2026_06_23.md)
  - [add_bottle_universal_flow_tz_2026_06_12.md](/R:/Flutter/Project/winepool_final/docs/add_bottle_universal_flow_tz_2026_06_12.md)
  - [receipt_sprint_status.md](/R:/Flutter/Project/winepool_final/docs/receipt_sprint_status.md)
  - [map_experience_roadmap_2026_04_25.md](/R:/Flutter/Project/winepool_final/docs/map_experience_roadmap_2026_04_25.md)
  - [seo_aso_growth_plan_2026_05_23.md](/R:/Flutter/Project/winepool_final/docs/seo_aso_growth_plan_2026_05_23.md)

### 16. Historical docs cleanup и аккуратная архивация

- `Delivery priority`: `Deferred`
- `Strategic value`: `Medium`
- `Complexity`: `S`
- `Type`: `docs`
- Что входит:
  - постепенно архивировать или упростить старые handoff/research файлы только после стабилизации текущих truth-docs
  - не удалять полезный historical context преждевременно
- Почему deferred:
  - сейчас важнее держать sync, чем aggressively чистить history
- Основные источники:
  - [documentation_map.md](/R:/Flutter/Project/winepool_final/docs/documentation_map.md)
  - [receipt_buyer_admin_governance_spec.md](/R:/Flutter/Project/winepool_final/docs/receipt_buyer_admin_governance_spec.md)

## Самые Практичные Ближайшие Кластеры

Если объединять backlog не по темам, а по реалистичным delivery-кластерам, получается так:

### Cluster A. Недавно закрытый governance baseline

- `1. Receipt buyer/review split и owner-scoped draft flow`
- `2. Phase 5 stabilization: alias/merge/rollback regression smoke`
- `3. Doc sync для live Phase 5 и admin operational контекста`
- `4. Suspect price trust redesign`

### Cluster B. Текущий активный draft moderation follow-up

- `Track B full draft details redesign`
- `Abuse Controls Phase 1: draft-level guardrails`
- `Abuse Controls Phase 2: user-level restrictions`

### Cluster C. Parallel receipt quality cluster

- `5. Live calibration receipt matcher`
- `6. Receipt review UX polish`
- `8. System-wide audit winery alias coverage`

### Cluster D. Следующие governance/product slices

- `10. Broader moderated catalog collaboration`
- `11. Wine-level duplicate merge после winery merge`
- `15. WinePool Cellar Pro P0/P1`

## Что Пока Осознанно Не Смешивать

По текущим документам лучше не смешивать с ближайшими slices:

- полный RBAC platform
- courier subsystem
- social/features redesign
- deep personalization
- aggressive cleanup historical docs до стабилизации truth-layer

## Короткий Итог

Если ранжировать по сочетанию `risk + value + readiness`, то верхний закрытый baseline уже такой:

1. receipt route/permission split
2. merge/rollback regression smoke
3. doc sync для live governance reality
4. suspect price trust redesign

А текущий активный набор теперь такой:

5. `Track B full draft details redesign`
6. `Abuse Controls Phase 1`
7. `Abuse Controls Phase 2`

Параллельный receipt-quality набор теперь такой:

- receipt calibration на реальных чеках
- receipt review UX polish
- system-wide alias coverage audit

А самый большой, но уже не ближайший шаг:

- `wine-level duplicate merge` после стабилизации текущего winery merge baseline

## Тот Же Список Простыми Словами

Ниже тот же backlog, но уже без технической формулировки.

### Что уже закрыто

1. Развести обычный пользовательский режим и служебный режим проверки чеков.
Этот слой уже закрыт как baseline: обычный пользователь больше не должен упираться в review-only ограничения там, где речь идёт о его собственных чеках и собственных черновиках.

2. Спокойно перепроверить новый функционал по alias и merge виноделен.
Этот этап тоже уже прошли: документы синхронизированы с live baseline, локальный smoke сделан, и один конкретный post-merge read-side gap уже исправлен. Отдельно в будущем всё ещё полезно прогнать rollback на свежем merge в writable среде.

3. Додержать документацию по merge-слою в одном состоянии.
Этот кусок уже закрыт как текущий doc-baseline: `implementation plan`, `policy draft`, `write surface inventory` и `administrator guide` приведены ближе к реальному состоянию.

4. Перепридумать логику подозрительных цен.
Этот этап тоже закрыт: теперь подтверждение пользователем своей цены не делает её автоматически “официально верной” для всей платформы. Для этого введён owner-scoped split между личным решением и будущим platform-trusted уровнем.

### Что лучше делать в ближайшее время

5. Полностью довести экран `Черновик вина`.
Теперь именно это главный следующий спринт. Moderation loop уже есть, а основной UX-gap остаётся в самом экране: его нужно сделать ясным, цельным и быстрым для пользователя.

6. Добавить первую волну anti-abuse защиты на уровне конкретного draft.
Сначала логично закрыть бесконечные циклы повторной отправки одного и того же проблемного кейса.

7. После этого добавить user-level ограничения на новые отправки.
Это более чувствительный governance-layer, поэтому его лучше делать только после улучшения UX и после draft-level guardrails.

8. Параллельно продолжать receipt-quality cluster.
Сюда относятся:
- живая калибровка matcher на реальных чеках;
- UX-polish weak/receipt-review flows;
- системный audit alias coverage.

### Что логично делать следующим слоем

9. Расширять moderated collaboration в каталоге.
Базовый путь `Отправить на проверку` уже реализован. Следующий слой здесь — не заново строить pipeline, а аккуратно расширять moderated collaboration дальше.

### Что важно, но лучше делать позже

10. Расширять moderated collaboration в каталоге.
То есть постепенно дать бизнесам больше аккуратных путей предлагать изменения в каталог, но без прямой записи в canonical данные.

11. Делать merge вин после merge виноделен.
Это уже большой отдельный этап. Он сложный, потому что у вин есть свои связи: офферы, погребок, заказы, чеки, draft-ы. Это точно не маленький follow-up, а самостоятельный крупный slice.

12. Доводить seller home до полноценной business console.
Это полезно, но сейчас не такой критичный риск, как permission mismatch в receipt flow или стабильность merge-логики.

13. Построить мощный инструмент массового пополнения каталога.
Старый CSV-импорт создаёт дубли и идёт мимо alias/audit-слоя. Когда появится новый крупный источник данных, импорт нужно строить как «модерацию пачкой»: staging, alias-aware матчинг, разбор спорных строк, применение с аудитом и откатом. Направление зафиксировано в `batch_catalog_import_staging_concept_2026_06_12.md`, начинать — по триггерам оттуда.

14. Построить WinePool Cellar Pro.
Это следующий сильный consumer-side слой: приватные зоны хранения без координат, расположение бутылок внутри погребка, движение, экспорт и будущий Pro-тариф. Начинать лучше не с NFC и не с тяжелого paywall, а с P0/P1: SEO/waitlist + простые зоны хранения поверх текущего погребка.

### Что пока лучше сознательно не трогать

15. Сложные расширения прав и ролей.
Например:
- очень детальные admin-права
- несколько виноделен на один business
- delegated managers
- более тяжёлый RBAC-слой

16. Агрессивную чистку старых документов.
Исторические handoff и research-файлы пока лучше не удалять. Сейчас важнее не “почистить всё старое”, а держать в порядке живые документы и не путать их с историческим контекстом.

## Совсем Коротко

Если сказать очень просто, то сейчас у нас три главных направления:

1. Дочинить границы прав и ролей там, где они ещё расходятся с реальным UX.
2. Стабилизировать и спокойно проверить новый merge/alias слой.
3. После этого вернуться к качеству receipt flow: калибровка, понятные действия, alias coverage и trust.

А большой следующий шаг после этой стабилизации:

4. Отдельно проектировать merge вин внутри уже объединённых виноделен.
