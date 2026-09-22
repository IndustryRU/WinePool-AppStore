# Next Iterations Execution Order

Последнее обновление: 23.06.2026

> **Статус с 25.08.2026: historical / superseded.**
>
> Документ фиксирует завершённую апрельско-июньскую последовательность moderation/draft работ и больше не является официальным execution order.
> Текущий порядок после релиза 1.1.0 задаёт [post_release_1_1_0_execution_roadmap_2026_08_25.md](/R:/Flutter/Project/winepool_final/docs/post_release_1_1_0_execution_roadmap_2026_08_25.md).

## Зачем Нужен Этот Файл

Этот документ фиксирует официальный порядок следующих рабочих шагов после того, как moderated draft baseline уже доведён до рабочего состояния.

Здесь не весь backlog, а именно текущий execution-order:

- что уже закрыто как baseline;
- что является следующим активным спринтом;
- что идёт сразу после него;
- что пока сознательно не смешивается с ближайшими шагами.

Полный backlog:

- [prioritized_work_backlog.md](/R:/Flutter/Project/winepool_final/docs/prioritized_work_backlog.md)

Общая карта документации:

- [documentation_map.md](/R:/Flutter/Project/winepool_final/docs/documentation_map.md)

Текущий status-board по moderation baseline:

- [receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md](/R:/Flutter/Project/winepool_final/docs/receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md)

## Текущий Базовый Статус

На 22.04.2026 важно считать уже закрытым как working baseline:

1. Старый execution-order по ранним итерациям:
   - `Iteration 1. Receipt Boundary Alignment`
   - `Iteration 2. Phase 5 Stabilization And Operational Sync`
   - `Iteration 3. Suspect Price Trust Redesign`
2. Moderation pipeline по цепочке:
   - `draft -> submission -> admin review -> decision -> read-side return`
3. Внутренний moderation handoff:
   - `Iteration A` закрыта;
   - `Iteration B` закрыта;
   - `Iteration C` закрыта;
   - `Iteration D` частично закрыта;
   - `Track B` частично закрыт за счёт landed soft evidence layer.

Практически это означает:

- главный missing piece теперь не в отсутствии самого moderation loop;
- главный открытый UX-gap находится на экране `Черновик вина`;
- главный открытый governance-gap находится в anti-abuse слое поверх уже работающей модерации.

## Официальный Следующий Порядок

### 0. Статусный Документ, А Не Итерация

Перед началом следующих работ не нужно повторно изобретать текущее состояние.

Главный статусный документ сейчас такой:

- [receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md](/R:/Flutter/Project/winepool_final/docs/receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md)

Его роль:

- фиксировать, что уже landed;
- показывать, что осталось открытым;
- не быть самостоятельным implementation-спринтом.

Иными словами:

- это `source of current state`;
- но не `следующая задача`.

## Iteration 1. Track B Full Draft Details Redesign

### Главная цель

Довести экран `Черновик вина` до цельного, ясного и визуально согласованного product flow без изменения уже работающей moderation-логики.

### Почему это следующий шаг номер один

Moderation baseline уже работает.

Значит, самый сильный практический выигрыш сейчас даст не новый backend-slice, а улучшение экрана, через который проходят:

- новичок;
- опытный пользователь;
- пользователь, который получил запрос на уточнение;
- пользователь, который должен повторно отправить черновик;
- модератор косвенно, потому что качество входящих данных сильно зависит от ясности этого экрана.

Если сначала не убрать UX-перегруз и смешение блоков, есть риск:

- продолжать получать слабые или лишние отправки;
- путать пользователя competing CTA;
- лечить governance-ограничениями то, что частично вызвано неясным интерфейсом.

### Что входит

1. Полный IA/visual redesign экрана `Черновик вина`.
2. Нормализация иерархии:
   - что это за черновик;
   - в каком он состоянии;
   - что делать сейчас;
   - какие есть подтверждения;
   - какие есть candidate matches;
   - какие есть данные;
   - какие есть источники и история.
3. Финальное решение по конкуренции блоков:
   - `Что сделать дальше`
   - `Путь в каталог`
   - sticky action bar
4. Приведение evidence-блока к единому premium dark UI-языку.
5. Пересборка visual priority для:
   - `needs_submitter_input`
   - `submitted`
   - `in_review`
   - `rejected`
   - `approved / resolved`
6. Упрощение чтения suggested wines и вторичных секций.

### Что сознательно не входит

- новый anti-abuse policy layer;
- draft-level lock semantics;
- user-level submission restriction;
- mandatory completeness gate;
- отдельный OCR pipeline по фото этикетки;
- новый analytics stack.

### Основные документы для этой итерации

- [draft_wine_details_redesign_tz_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/draft_wine_details_redesign_tz_2026_04_21.md)
- [track_b_draft_details_evidence_implementation_plan_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/track_b_draft_details_evidence_implementation_plan_2026_04_21.md)
- [receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md](/R:/Flutter/Project/winepool_final/docs/receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md)
- [administrator_guide.md](/R:/Flutter/Project/winepool_final/docs/administrator_guide.md)

### Результат итерации

- экран `Черновик вина` перестаёт ощущаться как набор слабо связанных карточек;
- основной пользовательский next-step считывается за несколько секунд;
- evidence и moderation feedback становятся понятнее;
- качество повторных отправок и точность действий пользователя повышаются без ужесточения governance.

### Done when

- у экрана есть один явный главный workflow-центр;
- competing CTA больше не создают ощущение нескольких параллельных рабочих столов;
- evidence-block визуально встроен в общий стиль;
- moderation states читаются быстро и без противоречий;
- пользователь без глубокого чтения понимает, что делать дальше.

## Iteration 2. Moderation Abuse Controls Phase 1: Draft-Level Guardrails

### Главная цель

Закрыть бесконечные циклы повторной отправки одного и того же проблемного draft после того, как честный UX уже был улучшен.

### Почему это второй шаг

После завершения `Track B` у нас будет более честная картина:

- какие плохие кейсы действительно вызваны злоупотреблением;
- а какие были вызваны запутанным интерфейсом.

Только после этого разумно вводить жёсткие ограничения на уровне конкретного draft.

### Что входит

1. Draft-level lock / close semantics для повторной отправки.
2. Явная причина блокировки конкретного draft.
3. Понятный user-visible статус и запрет повторного submit там, где модератор закрыл кейс.
4. Moderator-side decision path для такого закрытия.
5. Backend enforcement в `submitDraftForCatalogReview(...)`.
6. Audit trail по таким решениям.

### Что сознательно не входит

- user-level ban / restriction;
- automated abuse scoring;
- auto-ban heuristics;
- platform-wide trust scoring для submitter.

### Основной документ для этой итерации

- [draft_catalog_moderation_abuse_controls_tz_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/draft_catalog_moderation_abuse_controls_tz_2026_04_21.md)

### Результат итерации

- один и тот же проблемный draft нельзя бесконечно гонять по кругу;
- модератор получает первый реальный anti-abuse инструмент;
- нагрузка на moderation loop снижается без преждевременного user-level punishment.

### Done when

- модератор может закрыть конкретный draft для повторной отправки;
- пользователь видит понятную причину и не может обойти запрет обычным resubmit;
- read side и admin side синхронно показывают это состояние;
- policy и runtime не расходятся.

## Iteration 3. Moderation Abuse Controls Phase 2: User-Level Restrictions

### Главная цель

Добавить следующий governance-layer только после того, как draft-level protection уже внедрена и понятна в эксплуатации.

### Почему это третий шаг

User-level restriction намного чувствительнее продуктово и governance-wise.

Её лучше делать только после того, как:

- честный UX уже улучшен;
- draft-level guardrails уже работают;
- есть основания утверждать, что проблема действительно на уровне поведения пользователя, а не конкретного кейса.

### Что входит

1. Временные или постоянные ограничения на новые catalog submissions пользователя.
2. Moderator controls для включения и снятия таких ограничений.
3. Понятные user-visible notices.
4. Backend enforcement на уровне submit path.
5. Audit trail и явные причины ограничения.

### Что сознательно не входит

- fully automated banning;
- сложный reputation engine;
- скрытые heuristic decisions без moderator decision;
- тяжёлый universal RBAC layer.

### Основной документ для этой итерации

- [draft_catalog_moderation_abuse_controls_tz_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/draft_catalog_moderation_abuse_controls_tz_2026_04_21.md)

### Результат итерации

- модератор получает не только draft-level, но и user-level anti-abuse контур;
- moderation loop становится более защищённым от серийного злоупотребления;
- governance layer усиливается уже поверх ясного UX и работающего baseline.

### Done when

- moderator может ограничить отправку новых черновиков для конкретного пользователя;
- пользователь честно видит ограничение и его причину;
- submit path реально блокируется backend-слоем, а не только UI;
- история решений сохраняется в audit trail.

## Optional Iteration 4. Moderation Instrumentation And Analytics

### Главная цель

Добавить отдельный observability-layer только после того, как UX и anti-abuse baseline уже стабилизированы.

### Почему это optional и почему после governance

Сейчас event trail уже существует и закрывает минимальный operational baseline.

Отдельный analytics/instrumentation slice имеет смысл только после того, как:

- экран черновика доведён до понятного состояния;
- draft-level anti-abuse внедрён;
- user-level restrictions внедрены или осознанно отложены.

Иначе есть риск собирать метрики по ещё неустоявшемуся UX и policy contract.

### Что может входить

- moderation funnel metrics;
- time-to-decision;
- clarification rate;
- resubmission quality metrics;
- abuse-control usage metrics;
- counters для product follow-up решений.

### Что сознательно не входит

- heavy BI layer;
- большой external analytics platform rollout;
- premature optimization до стабилизации UX/governance.

### Результат итерации

- moderation flow становится измеримым как product system;
- можно принимать следующие решения не только по ощущениям, но и по данным.

## Parallel / Post-Current Product Track. WinePool Cellar Pro

### Главная цель

Зафиксировать и постепенно реализовать новый consumer-side retention/monetization слой: учет личного запаса вина, физическое расположение бутылок, движение, экспорт, совместный доступ и будущий платный `Погреб Pro`.

### Почему это не ломает текущий execution-order

Текущий официальный порядок выше остается главным для moderation governance:

1. `Track B full draft details redesign`
2. `Abuse controls Phase 1`
3. `Abuse controls Phase 2`
4. `Optional instrumentation`

`Cellar Pro` не требует менять этот порядок, потому что первые P0/P1-срезы можно делать поверх уже работающих consumer-data потоков:

- `add-bottle` добавляет бутылки;
- receipt flow переносит покупки в погребок;
- `user_storage` уже хранит партии, цены, винтажи и источник;
- карта покупок уже отделяет `где куплено`;
- новый слой добавляет `где лежит`.

### Что можно делать параллельно

1. P0 validation:
   - SEO/landing `/wine-cellar`;
   - waitlist/fake-door;
   - in-app teaser;
   - аналитические события интереса.
2. P1 structured locations:
   - `user_cellars`;
   - `cellar_locations`;
   - `cellar_stock_lots`;
   - `cellar_inventory_events`;
   - placement/move/open flows без paywall и без NFC.

### Что сознательно не смешивать с ближайшим moderation-order

- full Pro paywall;
- QR/NFC exact bottle instances;
- tablet/desktop redesign;
- shared cellar/collaborators;
- сложную инвентаризацию больших коллекций.

### Основной документ

- [wine_cellar_pro_tz_2026_06_23.md](/R:/Flutter/Project/winepool_final/docs/wine_cellar_pro_tz_2026_06_23.md)

## Как Читать Документы В Этом Порядке

Если задача идёт по текущему официальному execution-order, документы лучше читать так:

1. [receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md](/R:/Flutter/Project/winepool_final/docs/receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md)
   Это status-board: что уже сделано и где мы находимся.
2. [draft_wine_details_redesign_tz_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/draft_wine_details_redesign_tz_2026_04_21.md)
   Это главный active-design document на ближайший спринт.
3. [track_b_draft_details_evidence_implementation_plan_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/track_b_draft_details_evidence_implementation_plan_2026_04_21.md)
   Это implementation companion к Track B.
4. [draft_catalog_moderation_abuse_controls_tz_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/draft_catalog_moderation_abuse_controls_tz_2026_04_21.md)
   Это следующий governance-layer после завершения Track B.

## Что Сейчас Специально Не Смешивать

В ближайшие итерации лучше не смешивать с этим execution-order:

- `wine-level duplicate merge`;
- seller home redesign;
- broader business console expansion;
- full Cellar Pro paywall / QR-NFC inventory beyond P0/P1;
- full OCR/media platform;
- heavy RBAC expansion;
- aggressive historical docs cleanup.

## Короткий Итог

Официальный порядок теперь такой:

1. `Track B full draft details redesign`
2. `Abuse controls Phase 1: draft-level guardrails`
3. `Abuse controls Phase 2: user-level restrictions`
4. `Optional instrumentation / analytics layer`

Параллельный продуктовый трек, который можно готовить без сбоя этого порядка:

- `WinePool Cellar Pro P0/P1`: SEO/waitlist + приватные зоны хранения без координат поверх текущего погребка.

А главный status-document для этого порядка:

- [receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md](/R:/Flutter/Project/winepool_final/docs/receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md)

То есть:

- moderation pipeline больше не строим заново;
- сначала делаем экран черновика по-настоящему сильным;
- потом усиливаем moderation governance;
- и только после этого, если нужно, отдельно достраиваем наблюдаемость и метрики.
