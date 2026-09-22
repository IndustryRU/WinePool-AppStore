# H0 — Measurement, Activation and Partner Attribution

Дата: 25.08.2026
Статус: active implementation source of truth для горизонта H0
Родительский roadmap: [post_release_1_1_0_execution_roadmap_2026_08_25.md](/R:/Flutter/Project/winepool_final/docs/post_release_1_1_0_execution_roadmap_2026_08_25.md)
Baseline: [product_metrics_baseline_2026_08_25.md](/R:/Flutter/Project/winepool_final/docs/product_metrics_baseline_2026_08_25.md)

## 1. Цель

Построить проверяемую сквозную систему измерения:

```text
канал/referral
-> public/store entry
-> регистрация
-> первое содержательное действие
-> повторное действие
-> partner lead
-> подтверждённый outcome
```

Система должна одновременно:

- давать владельцу честные product KPI;
- сохранять отдельный QA/crash diagnostic view;
- не передавать в аналитику PII, OCR, полный текст, штрихкод или контакт заявки;
- сверяться с серверными business facts;
- работать при малой аудитории и показывать абсолютные числа;
- подготовить H1 Tourism Revenue Loop.

## 2. Подтверждённый baseline

AppMetrica за 01.05–25.08 показывает 96 пользователей, 769 сессий и 888 экспортированных событий. Но product KPI загрязнены:

- один кластер Redmi Note 8 Pro формирует 2 574 profile-сессии и 1 018 крэшей;
- присутствуют emulator/test-like профили;
- `first_*` отправляется после каждого успешного действия;
- add-bottle `request_id` содержит barcode или фрагмент текстового идентификатора;
- profile export и engagement export имеют разный session scope;
- add-bottle matcher-калибровка смешана с внешним поведением.

Следствие: H0 начинается с контракта и очистки, а не с визуального dashboard.

## 3. Аудит текущей реализации

### 3.1. Клиентский канал

- SDK: `appmetrica_plugin ^3.4.0`;
- инициализация: `Analytics.init()` после `Supabase.initialize`;
- web-события не отправляются;
- ошибки отправки intentionally не ломают пользовательский flow;
- централизованный фасад: `lib/core/analytics/analytics.dart`;
- build mode и actor type до H0 не передавались.

### 3.2. Текущие consumer events

- onboarding completion;
- registration wall shown/action;
- add-bottle open/path/match/selection/rejection/draft/cellar;
- receipt/wine/tasting milestones;
- часть admin knowledge operations.

Нет единого контракта для:

- registration completed;
- authenticated session identity;
- catalog search/result open;
- review created;
- shop/place/route intent;
- app return;
- referral/public entry;
- tourism outcome.

### 3.3. Подтверждённые дефекты

#### Неверная семантика `first_*`

`firstTastingLogged`, `firstWineAdded` и `firstReceiptScanned` вызывались после каждого успешного действия без проверки первого факта. Исторические события сохраняются, но после H0.0 новые повторяемые события называются:

- `tasting_logged`;
- `wine_added`;
- `receipt_scanned`.

Настоящие first milestones считаются серверным weekly report или отдельным идемпотентным серверным контуром позже.

#### Утечка высококардинального исходного идентификатора

`request_id` содержит значения вида `barcode-...-8004939329623` и попадает в экспорт AppMetrica. После H0.0 наружу передаётся только случайный opaque `attempt_id`, стабильный в рамках текущего процесса/попытки. Сырой request id остаётся только локальным ключом дедупликации.

#### Client-only дедуп

Текущий set живёт только в памяти процесса и ограничен 2 000 ключами. Он защищает от повторного callback в одной сессии, но не является серверной аналитической истиной и не переживает restart.

### 3.4. Server facts

Для weekly report использовать факты, а не только client events:

- auth users/profiles — регистрация;
- `user_storage` — бутылка/погребок;
- `user_tastings` — дегустация;
- `receipts` и receipt items — сохранённый чек;
- reviews — созданный/опубликованный отзыв;
- catalog/draft proposals — вклад;
- `tourism_leads` + event history — заявка и статусы;
- будущий `completed/no_show` — outcome.

Точные таблицы и действующие функции подтверждаются runtime/database audit перед миграцией отчёта.

## 4. Термины

- `raw analytics` — все события, включая owner/QA/debug.
- `clean analytics` — external production traffic после правил исключения.
- `business fact` — серверная запись, подтверждающая выполненное действие.
- `actor_type` — `external`, `owner`, `qa`, `automated`.
- `build_mode` — `release`, `profile`, `debug`.
- `attempt_id` — обезличенный идентификатор одной пользовательской попытки.
- `referral_code` — стабильный код внешнего источника.
- `activation` — содержательное действие в первые семь дней после регистрации.
- `strong activation` — содержательные действия в два разных дня первых семи дней.

## 5. Неторгуемые правила

1. Product KPI по умолчанию строятся только на clean analytics.
2. Raw/QA данные не удаляются и доступны в диагностике.
3. Email, телефон, имя, OCR, review text, barcode, receipt fiscal values и полный search text не отправляются в AppMetrica.
4. Контакт tourism lead существует только в защищённом operational контуре.
5. Проценты всегда сопровождаются numerator/denominator.
6. Client event не подменяет server fact.
7. Малые cohorts не объединяются между платформами и версиями без явной маркировки.
8. Feature rollout не блокирует core UX при ошибке analytics.

## 6. Целевая event envelope

Обязательные параметры client event, когда применимо:

```text
build_mode
actor_type
platform            # AppMetrica profile dimension, не дублировать без нужды
attempt_id          # только для локальной воронки
entry
path
result
referral_code       # только нормализованный публичный код
```

Запрещённые параметры:

```text
email
phone
name
raw_ocr
search_text
barcode
request_id_with_input
receipt_fn/fd/fp
tourism_comment
review_text
```

Высококардинальные canonical IDs (`wine_id`, `tour_id`) добавляются только при подтверждённой аналитической задаче. Для регулярного owner report предпочтительны server-side aggregates.

## 7. Этапы реализации

### H0.0. Privacy and semantic hotfix

Scope:

- добавить `build_mode` ко всем AppMetrica events;
- default actor: release → external, debug/profile → qa;
- разрешить безопасный override через `WINEPOOL_ANALYTICS_ACTOR_TYPE`;
- заменить отправку сырого `request_id` на opaque `attempt_id`;
- сохранить стабильность attempt id внутри одной попытки;
- переименовать повторяемые `first_*` в truthful outcome events;
- добавить unit tests.

Acceptance:

- barcode/text fragment отсутствует в параметрах;
- одинаковый локальный request получает одинаковый opaque attempt;
- разные requests получают разные attempts;
- debug build маркируется QA;
- успешные повторные действия больше не называются first;
- пользовательский flow не меняется.

### H0.1. Actor classification

Перед реализацией выбрать итоговую серверную модель после live DB audit.

Рекомендуемая модель:

```text
analytics_actor_overrides
  user_id uuid primary key
  actor_type text check (external, owner, qa, automated)
  reason text
  is_active boolean
  created_at / updated_at
  created_by
```

Требования:

- direct public read запрещён;
- current user получает только собственный effective actor type через узкий RPC;
- admin управляет overrides через SQL/admin surface позднее;
- по умолчанию authenticated release user — external;
- owner и известные тестовые аккаунты вносятся явно;
- device model не является единственным основанием исключения;
- anonymous pre-auth events очищаются по build mode/campaign и показываются отдельно.

Client:

- обновляет effective actor после auth change;
- не отправляет email/user id как event parameter;
- при ошибке RPC использует conservative build default;
- logout сбрасывает actor context.

### H0.2. Event dictionary and instrumentation audit

Создать versioned dictionary со столбцами:

- name;
- owner;
- trigger;
- user/server scope;
- allowed params;
- forbidden params;
- idempotency;
- source of truth;
- raw/clean eligibility;
- retention;
- rollout version.

Минимальный consumer funnel:

```text
onboarding_completed
registration_started
registration_completed        # server fact preferred
catalog_search_started
catalog_result_opened
add_bottle_opened
add_bottle_path_selected
add_bottle_match_result
add_bottle_draft_created
wine_added
receipt_scanned
tasting_logged
review_created
catalog_proposal_submitted
shop_place_opened
route_opened
```

Add-bottle event invariant:

```text
attempt_id
  -> path
  -> one match_result
  -> optional selection/rejection
  -> draft_created OR wine_added OR abandoned
```

`abandoned` не отправляется на каждое закрытие экрана до отдельного lifecycle решения; его можно вычислять из attempt cohort.

### H0.3. Server-side owner report

Первый отчёт может быть SQL/RPC + admin-only export, без большого dashboard.

Периоды:

- day;
- ISO week;
- rolling 7/30 days;
- release cohort.

Обязательные показатели:

- registrations;
- activated users D0/D7;
- strong activation;
- users with first server fact by type;
- D1/D7/D30 return после достаточного окна;
- reviews/proposals/moderation outcomes;
- raw vs clean reconciliation;
- platform/version/source breakdown там, где источник надёжен.

Privacy:

- owner report агрегированный;
- строки отдельных external users не экспортируются;
- при cohort < 5 показывать absolute count без детального cross-breakdown либо применять agreed suppression.

### H0.4. Partner attribution

Проверить существующие `tourism_leads.referral_source`, `referral_code`, `referral_url` и public route preservation.

Цепочка:

```text
public_open
-> experience_view
-> lead_started
-> lead_submitted
-> contacted
-> confirmed
-> completed/no_show
```

Требования:

- referral code нормализован и ограничен по длине/алфавиту;
- raw arbitrary URL не используется как dimension owner dashboard;
- заявка хранит immutable acquisition snapshot;
- status history содержит actor и timestamp;
- completed подтверждается оператором с audit;
- один referral registry связывает код с partner/location/material.

### H0.5. Pilot readiness

- production smoke exact-tour QR;
- iOS/Android/web;
- guest/auth flows;
- age gate/deep link preservation;
- operator notification;
- response SLA;
- test lead явно помечается и исключается из commercial KPI;
- ручной report по одному referral code.

### H0.6. Weekly operating cadence

Раз в неделю фиксировать:

- store inputs;
- clean registrations/activation;
- raw vs clean delta;
- anomalies;
- crashes external vs QA;
- top funnel loss;
- partner funnel;
- одно принятое product action;
- состояние data quality.

## 8. Activation contract

### Activation v1

Authenticated user в первые семь календарных дней после регистрации создал хотя бы один server fact:

- bottle/storage;
- receipt;
- tasting/review;
- catalog contribution.

### Strong activation

Не менее двух qualifying facts в разные даты первых семи дней.

### Не считать activation

- screen view;
- registration wall tap;
- onboarding completion;
- search без результата/действия;
- internal/QA fact;
- тестовую tourism заявку.

## 9. Retention contract

- D1/D7/D30 строятся по registration cohort и clean external activity;
- календарная и rolling semantics фиксируются в отчёте;
- background/technical session не является meaningful return;
- дополнительно считать `meaningful return`: qualifying action после первого дня;
- до пяти пользователей в cohort показывать абсолютные числа и пометку low sample.

## 10. Crash contract

- AppMetrica crash diagnostics остаётся техническим источником;
- external release, owner release, QA/debug и emulator разделяются;
- не считать crash rate из profile-list cumulative sessions;
- основной KPI после очистки: crash-free external users/sessions по версии;
- кластер по модели/OS используется для диагностики, не для автоматического признания пользователя QA.

## 11. Migration and rollout

1. H0.0 — client-only hotfix, без migration.
2. H0.1 — additive schema/RPC после backup и live audit.
3. Client dual behavior: если actor RPC отсутствует/ошибся, build default.
4. Raw historical AppMetrica данные не переписываются; 25.08.2026 — semantic boundary.
5. New event names анализируются отдельным периодом и не склеиваются с `first_*` без пометки.
6. Dashboard/report rollout сначала admin/owner-only.

## 12. QA

Automated:

- opaque attempt mapping;
- request dedup;
- no raw request id in params;
- actor/build defaults;
- first/outcome event naming;
- actor RPC/RLS negative tests;
- report aggregation fixtures;
- raw/clean reconciliation.

Manual:

- debug event виден как qa/debug;
- release external event виден как external/release;
- owner override меняет actor после login;
- logout сбрасывает actor;
- barcode flow не раскрывает barcode в AppMetrica export;
- one attempt связывает match и outcome;
- server report совпадает с контрольными facts;
- test tourism lead исключён из partner KPI.

## 13. H0 acceptance criteria

H0 закрыт, когда:

1. raw и clean аудитория разделены;
2. owner/QA/automated не входят в product KPI;
3. build mode присутствует в новых events;
4. prohibited payload не попадает в аналитику;
5. outcome events имеют правдивую семантику;
6. activation считается по server facts;
7. weekly owner report воспроизводим;
8. referral проходит до tourism lead;
9. есть completed/no-show operational contract;
10. тестовая заявка исключается;
11. production smoke пройден;
12. подготовлены оператор и 3–5 pilot-точек H1.

## 14. Статус выполнения

### 25.08.2026 — аудит, H0.0 и код H0.1

- проанализированы AppMetrica audience/engagement/events/profiles;
- подтверждено сильное загрязнение owner/QA activity;
- найден повторяемый `first_*` contract mismatch;
- найден raw barcode/text-bearing `request_id` в аналитике;
- H0.0 hotfix реализован: `build_mode`, безопасный `actor_type`, честные
  outcome events и opaque `attempt_id`;
- H0.0 покрыт unit-тестами, целевой `flutter analyze` проходит без замечаний;
- подготовлена и применена на production additive H0.1 migration
  `202608251930_add_analytics_actor_overrides.sql` с закрытой таблицей,
  self-read RPC без PII и admin-only setter;
- production backup:
  `/root/db_backups/winepool_pre_h0_actor_overrides_20260825_152257.dump`
  (6.4 MB), SHA-256
  `74e5d3413891043de397d6445459a196bc61722108f23168631e09f9222b313d`;
- dry-run `BEGIN/ROLLBACK` и production apply с `ON_ERROR_STOP=1` прошли;
  post-check подтвердил RLS, отсутствие table read у `anon/authenticated`,
  `EXECUTE` только у `authenticated` и отказ admin setter для non-admin;
- PostgREST schema cache перезагружен через `NOTIFY pgrst`;
- клиент получает actor type при старте и auth change, а до применения
  migration безопасно использует build default;
- не выполнено: заполнение owner/QA overrides, authenticated device smoke,
  clean owner report и server-fact activation;
- следующий пакет: завершение H0.1 и H0.2/H0.3 — actor overrides, словарь событий, server facts и
  воспроизводимый raw/clean owner report.

### 25.08.2026 — завершение H0.1 и реализация H0.2/H0.3

- production boundary первого публичного RuStore-релиза подтверждён по
  release-документам: `1.0.3(2004)`, 23.05.2026;
- перед классификацией сохранены контрольные выгрузки overrides:
  `/root/db_backups/winepool_pre_h0_actor_assignment_20260825.dump`
  (SHA-256 `b3379f551acf928dfc84e4a4c3f75dbbfd282bb6778a7f8ac8ea1276ae15e391`)
  и `/root/db_backups/winepool_pre_h0_pre_release_roles_20260825.dump`
  (SHA-256 `5804e2781a91ebc2669a75597ae417658df5db8c92c284680d29158e753cfac6`);
- подтверждённые owner/developer и QA identities классифицированы без вывода
  email/UUID в отчёт; администраторский аккаунт, используемый для тестов,
  остаётся `qa`, поскольку actor override не изменяет admin permissions;
- аккаунты до публичного RuStore boundary классифицированы как QA; отдельно
  подтверждены synthetic-domain признаки; production aggregate: 29 active QA,
  owner-сегмент пока пуст, неподтверждённые аккаунты остаются external;
- versioned event dictionary создан в
  `docs/analytics_event_dictionary_v1_2026_08_25.md`; в нём разделены client,
  server-only и planned события, параметры, idempotency и source of truth;
- live DB audit подтвердил qualifying facts:
  `user_storage.created_at`, `user_tastings.created_at`,
  `user_receipts.created_at`, `reviews.created_at` и
  `draft_catalog_submissions.submitted_at`;
- additive migration `202608252100_add_h0_owner_report.sql` применена на
  production после dry-run; добавлены закрытый aggregate core и
  admin-only RPC `admin_h0_owner_report` без пользовательских строк/PII;
- Activation v1, D0, Strong activation, meaningful return и exact-calendar
  D1/D7/D30 считаются по server facts; незрелые retention cohorts возвращают
  `NULL`, cohort меньше 5 получает `low_sample=true`;
- raw reconciliation строится суммой actor segments; clean — только external;
  owner, QA и automated сохраняются отдельными строками;
- воспроизводимый psql export `supabase/reports/h0_owner_report.sql` выдаёт
  raw/clean summary, day, ISO week и rolling 7/30-day registration cohorts;
- SQL contract проверяет grants, negative non-admin access, actor/clean
  reconciliation, activation denominators, low-sample и retention maturity;
  authenticated production smoke admin RPC пройден;
- Dart contract фиксирует truthful outcome names и запрещает выдавать
  server-only milestones за client truth; целевые analytics tests проходят.

H0.1, H0.2 и H0.3 завершены. H0 целиком остаётся открытым: H0.4–H0.6 и общие
acceptance criteria по partner attribution/pilot cadence в этот пакет не входят.

### 25.08.2026 — H0.4–H0.6 partner attribution core

- live audit: 5 leads (`new=3`, `confirmed=2`), 0 referral snapshots,
  2 published experiences, 2 active organizations, 14 lead notifications;
- перед изменением создан backup
  `/root/db_backups/winepool_pre_h0_partner_attribution_20260825.dump`, SHA-256
  `011983208992aea4c9c823953913ae613d09cc1986ce40fbe0e63d77a1cd45ab`;
- production migration `202608252200_add_h0_partner_attribution.sql` применена:
  закрытый referral registry, normalized codes, immutable acquisition snapshot,
  `is_test`, audited `completed/no_show`, admin registry RPC и partner report;
- client funnel получил `tourism_public_open`, `tourism_experience_view`,
  `tourism_lead_started`, `tourism_lead_submitted`; наружу выходит только
  нормализованный public referral code, без contact/comment/raw URL;
- создан active production smoke code `h0_smoke_20260825` с `is_test=true`;
- transaction production smoke guest insert → snapshot → completed → audit
  прошёл с rollback; negative SQL contract и Dart normalization tests проходят;
- weekly operating SQL и pilot/operator runbook добавлены;
- H0.4 server/client core и H0.6 cadence готовы;
- H0.5 web exact-tour smoke пройден: canonical nested route сохраняет `ref`
  через trailing-slash redirect и возвращает HTTP 200 с public transition page
  и age gate. Public `winepool.ru` и Supabase/admin VPS размещены раздельно;
  это не blocker. До полного закрытия H0.5 остаются device guest/auth smoke,
  фактическое подтверждение operator notification и заполнение 3–5 pilot points.

### 26.08.2026 — device smoke и tourism leads compatibility hotfix

- cold Android deep link на exact-tour route открыл нужный тур и сохранил
  test referral; guest-заявка успешно создана с immutable operator/experience
  snapshot, `is_test=true`, статусом `new` и без authenticated creator;
- production notification создана для активного участника организации-
  оператора. Она не должна приходить catalog admin, который не состоит в этой
  tourism organization;
- подтверждён разрыв public CTA: переход через RuStore запускает установленное
  приложение на главной и теряет exact-tour/referral context; это отдельная
  задача app-link/store fallback;
- после успешной guest-отправки кнопка повторной заявки остаётся доступна;
  dedup/UX этого состояния требует отдельной доработки;
- live CRM выявил PostgREST ambiguity: H0 acquisition FK создали вторые пути
  `tourism_leads -> tourism_experiences/organizations`, а released client
  использовал неявные embeds. Счётчик показывал 6 заявок отдельным RPC, при
  этом ошибка списка маскировалась как пустой результат;
- перед hotfix сохранён schema backup
  `/root/db_backups/winepool_pre_h0_postgrest_relations_20260826.dump`, SHA-256
  `ddd9e6ed39e9f85d538203e28ba12d743ab420e18a6592f7088768ded2e27c94`;
- migration `202608260900_fix_h0_tourism_postgrest_relations.sql` прошла
  dry-run и применена на production: FK удалены только у immutable acquisition
  UUID snapshots, live operational FK сохранены, PostgREST cache перезагружен;
- новый SQL contract подтверждает единственный FK path к туру и организации;
  web client дополнительно использует explicit FK hints и показывает ошибку
  загрузки вместо ложного empty state;
- full catalog admin теперь отображается с активным Tourism capability,
  независимо от членства в конкретной организации;
- admin web release `20260826_202819` задеплоен на `admin.winepool.ru` после
  успешных analyze/build и `nginx -t`; endpoint без Basic Auth ожидаемо
  отвечает `401 Unauthorized`.

H0.4/H0.6 реализованы. Для окончательного закрытия H0.5 остаются подтверждение
CRM-листа после hotfix, решение store-to-app deep-link fallback и заполнение
3–5 реальных pilot points; текущая test-заявка исключена из коммерческих KPI.

Архитектурное решение по store/deep-link gap принято 26.08.2026: primary flow
будет web-first с переиспользуемой Flutter-формой, а native open/install —
optional continuation после заявки. Подробный implementation contract:
`docs/h0_tourism_web_first_lead_flow_tz_2026_08_26.md`. Этот минимальный
acquisition loop входит в H0.5; marketplace, кабинеты и расширение Tourism — H1.
H0.5 требует не только functional submit, но и owner visual acceptance
адаптивной premium tour surface на phone/tablet/desktop; content и form остаются
едиными, а layout-композиции адаптируются внутри одной Flutter design system.

Уточнение границы 30.08.2026: публичная operator page и `/tourism`
catalog/basic search важны для web, но не расширяют H0. В основной нумерации
это H1.1b operator page и H1.1c curated catalog/basic search. Advanced Tourism
discovery вынесен в growth backlog и не подменяет H2 Retail. Supporting plan:
`docs/h1_tourism_public_operator_catalog_discovery_plan_2026_08_30.md`; полный
implementation source of truth H1:
`docs/h1_tourism_revenue_loop_tz_2026_08_30.md`.

Manual CRM acceptance после hotfix подтвердил отображение всех 6 заявок и
переход test lead `new -> contacted -> confirmed -> completed`. Production
post-check подтвердил `completed_at`, outcome actor/timestamp, полный audit
trail и нулевую утечку этой заявки в commercial segment. Найден и исправлен
UI-дефект: для финальных `completed/no_show` больше не показываются действия,
возвращающие заявку в `confirmed/cancelled`.

Authenticated device smoke на втором туре также пройден: existing-lead guard
не дал создать дубль для первого тура, отдельный test referral создал ровно
одну authenticated заявку со всеми acquisition snapshots и двумя адресными
уведомлениями. Ветка `new -> contacted -> confirmed -> no_show` прошла;
production post-check подтвердил `no_show_at`, outcome actor/timestamp, audit
trail и нулевую commercial leak. Исправлены подписи фильтров финальных статусов,
которые ранее fallback-ились в `Новая`. Внутренний notification type пока
переиспользует legacy `draft_submission_created`; entity routing корректен,
но семантическое имя следует нормализовать отдельной additive migration.

### 29.08.2026 — Published Tour Contract и responsive web package

- contract-first gate H0.5 реализован: immutable Published Tour Contract v1,
  exact public RPC, CRM preview/publish и единый Dart adapter активированы;
- production backup, migration, SQL contract и read-only pilot smoke завершены;
  Massandra/Inkerman имеют валидные revision `3`, отдельные authored destination title,
  structured schedule/price, highlights/program/media semantics и не раскрывают
  internal identity/notification/vehicle recognition fields;
- утверждённые phone/tablet/desktop композиции реализованы в отдельном public
  Flutter Web entrypoint и собраны release target; shared lead form и stable
  success используются без отдельной HTML-формы;
- локальная visual QA пройдена на `390x844`, `834x1112`, `1440x1000`;
  canonical host утверждён как `winepool.ru/tourism/...`, технические assets —
  `/tourism/app/`; hidden production-like owner preview задеплоен с meta/HTTP
  noindex, self-hosted CanvasKit и route-specific shell, а будущий
  `tour.winepool.ru` зарезервирован только как QR redirect layer;
- H0.5 остаётся открытым до owner acceptance на реальном preview URL,
  одной web guest submission/duplicate-notification проверки, raw/clean
  attribution reconciliation и App/Universal Links. Git push и store release
  не входят в выполненный пакет.

Фактическая реализация и остающиеся операционные шаги зафиксированы в
`docs/h0_published_tour_contract_audit_2026_08_27.md` и
`docs/h0_tourism_web_first_lead_flow_tz_2026_08_26.md`.

### 30.08.2026 — закрытие H0

- все acceptance criteria H0.0–H0.6 закрыты; raw/clean, external/owner/QA,
  Activation v1, Strong activation, meaningful return и mature-only retention
  считаются по server facts;
- исправлен Sunday-дефект weekly runner: начало недели теперь вычисляет
  PostgreSQL `date_trunc('week', current_date)`, а не неоднозначная shell-фраза
  `monday this week`;
- первый production weekly baseline сохранён в
  `docs/h0_weekly_operating_report_2026_08_24.md`: external cohort `n=1`,
  Activation D0/7d `1/1`, strong `0`, meaningful return `0`, два qualifying
  actions; D7/D30 не созрели, все выводы помечены low sample;
- tourism reconciliation за неделю: commercial `0`, QA/test `3`, из них один
  completed и один no-show; test traffic не попал в commercial KPI;
- canonical `winepool.ru/tourism/o/.../t/.../` переключён с legacy RuStore CTA
  на принятую shared Flutter web-first страницу; consent, notification,
  idempotency, outcome workflow и raw/clean smoke уже подтверждены;
- Android App Links принят на реальном устройстве; iOS AASA/entitlement/server
  contract принят владельцем без device smoke из-за отсутствия Apple device;
- cold mobile baseline измеряется воспроизводимым
  `tool/measure_tourism_web.mjs`; logo уменьшил package с `20.81` до `19.57 MiB`,
  CanvasKit передаётся gzip. Оставшийся несжатый `main.dart.js` и дальнейшее
  ускорение — явный H1 performance backlog до масштабирования трафика;
- после отдельного backup registry подготовлены пять non-test pilot points,
  связанные с production operator/published tours. Они намеренно inactive до
  проверки реального размещения, ответственного и канала в H1; тестовые codes
  сохранены отдельно и активны только для QA.

**Итог: H0 закрыт.** Следующий разрешённый основной пакет — H1.1 public operator
page по отдельному bounded data audit и совместному owner visual gate. Публичный
интерфейс без участия владельца не перестраивается.
