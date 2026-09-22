# H1 — Tourism Revenue Loop: implementation specification

Дата: 30.08.2026  
Статус: **active implementation source of truth H1**  
Предшествующий gate: H0 закрыт 30.08.2026  
Authority: уточняет раздел H1 основного
`post_release_1_1_0_execution_roadmap_2026_08_25.md`, не меняя очередь H2–H6.
Supporting visual/product sequence публичных operator/catalog surfaces:
`h1_tourism_public_operator_catalog_discovery_plan_2026_08_30.md`.
Accepted implementation specification публичных профилей оператора и
винодельни:
`h1_02_public_operator_winery_profiles_tz_2026_08_31.md`.
Accepted monetization/contact clarification:
`h1_tourism_monetization_contact_decision_2026_08_31.md`.

## 1. Цель и проверяемый результат

H1 должен доказать не наличие красивой Tourism-функции, а один честный
повторяемый контур:

```text
измеримая ссылка/QR
→ public operator/catalog/exact-tour
→ consent-backed заявка
→ ответ ответственного в SLA
→ подтверждённая поездка
→ completed/no-show с actor audit
→ partner report
→ согласованный финансовый результат или осознанный waived
```

H1 завершён только после минимум одного реального внешнего `completed`,
сходящегося отчёта и подтверждения партнёром полезности. Код, тестовые заявки и
пустая витрина сами по себе H1 не закрывают.

### Исполнительское уточнение — 12.09.2026

Из-за завершения сезона у текущего оператора первый живой контур запускается
через винодельню, работающую круглый год. Страница винодельни, её собственные
программы посещения, заявки и общий кабинет образуют один ограниченный первый
пакет. Полный ассортимент остаётся в существующем общем каталоге WinePool и
открывается по фильтру винодельни; в H1 не создаётся второй каталог. Складские
остатки, наличие и цены в конкретных точках, а также связь с учётной системой
остаются следующим слоем после проверки спроса на посещение.

Подробный порядок, продуктовая формулировка и границы пилота записаны в
[`tourism_h1_compact_plan_2026_09_12.md`](tourism_h1_compact_plan_2026_09_12.md).

## 2. Место H1 в общей очереди

- H1 — Tourism Revenue Loop целиком;
- H2 — Retail Availability and First Store, не Tourism discovery;
- H3 — Winery Unified Value Center после фактов H1/H2;
- advanced Tourism discovery является отложенным growth backlog и не блокирует
  H1 gate.

Внутри H1 используется только следующая нумерация:

| Package | Результат |
|---|---|
| H1.1a | canonical exact-tour web-first baseline, завершён в H0 |
| H1.1b | public operator page и shared public-profile foundation |
| H1.1c | curated `/tourism` catalog/basic search |
| H1.2 | lead operations, SLA, assignment and notifications |
| H1.3 | customer Trip Center и guest-to-account continuity |
| H1.4 | outcome and revenue ledger |
| H1.5 | partner report v1 |
| H1.6 | pilot launch kit и 30-дневный запуск |

H1.1b/H1.1c могут разрабатываться рядом с H1.2, но не должны задерживать
активацию первых проверенных referral-точек и обработку реальных заявок.

## 3. Зафиксированный baseline после H0

Уже существует и переиспользуется:

- Published Tour Contract v1 с immutable revision и bounded anonymous RPC;
- два published тура одного активного оператора;
- canonical exact-tour Flutter Web page и единая native/web lead form;
- explicit consent evidence, rate limit и idempotent `submit_tourism_lead`;
- immutable referral snapshot и разделение `is_test`/commercial;
- статусы `new/contacted/confirmed/cancelled/completed/no_show`;
- assigned organization/user, pickup point, vehicle, tourist memo и ticket;
- customer actions, operator messages, Trip Center и app notifications;
- actor/timestamp audit в `tourism_lead_events`;
- raw/clean H0 report и server-fact outcome;
- Android App Links; iOS AASA/server contract без device acceptance;
- пять подготовленных non-test referral points, пока `is_active=false`.

Baseline не означает, что следующие разрывы решены:

- operator/catalog public clients ещё читают raw operational projections или
  используют устаревшие статические страницы;
- переходы lead частично выполняются прямым update без единой server state
  machine и optimistic concurrency;
- нет автоматического SLA reminder/escalation;
- guest lead нельзя безопасно связать с созданным после заявки аккаунтом;
- notification новой tourism-заявки местами использует legacy semantic type;
- opens/views/starts не являются надёжными server facts partner report;
- нет versioned partner terms и revenue ledger;
- нет owner/partner report v1, где funnel, SLA, outcome и деньги сходятся;
- подготовленные pilot points не прошли фактическую активацию на местах;
- текущий cold mobile baseline тяжёлый: дальнейшее ускорение обязательно до
  масштабирования платного трафика.

## 4. Scope и non-goals

### 4.1. В scope

- H1.1a hardening и H1.1b/H1.1c public entry;
- один оператор, 1–3 актуальных тура и 5–10 referral-точек;
- guest и authenticated заявки;
- операционная обработка, SLA и адресные уведомления;
- customer Trip Center и безопасное продолжение guest → account;
- completed/no-show и post-visit action;
- ручная модель оплаты за квалифицированную заявку или success fee за
  подтверждённый визит без приёма денег внутри WinePool;
- агрегированный owner/partner report;
- материалы и журнал фактически размещённых точек;
- производственный 30-дневный pilot.

### 4.2. Не входит

- оплата тура или алкоголя внутри WinePool;
- acquiring, split payments, чеки и автоматическая фискализация;
- slot/seat inventory и гарантированная online booking;
- универсальная ERP туроператора;
- сложная транспортная диспетчеризация;
- публичные кабинеты магазинов/виноделен за пределами pilot need;
- marketplace многих операторов, карта, рекомендации и сложные фильтры;
- массовое featured-размещение и рекламный аукцион;
- самостоятельная визуальная перестройка без owner acceptance.

## 5. Роли и права

| Role | Public content | Leads | Trip data | Outcome | Money/report |
|---|---|---|---|---|---|
| guest | published only | submit | только success/claim contract | нет | нет |
| authenticated customer | published | own/claimed | own Trip Center/actions | post-visit response | нет |
| operator agent | published + assigned context | assigned organization | update assigned trip | normal transitions | агрегаты без денег |
| operator manager/owner | то же | organization-wide | assign/manage | confirm outcome/correction request | own partner report/confirmation |
| WinePool admin | all approved/draft contexts | all | all | audited override | terms, ledger, export |
| automated worker | no UI | SLA jobs only | no arbitrary edit | no outcome | aggregate refresh only |

Правила:

- actor определяется сервером по `auth.uid()` и membership, не параметром;
- `viewer` никогда не изменяет lead;
- agent не меняет partner terms и financial status;
- customer не видит operator notes, internal contacts, attribution internals и
  financial ledger;
- partner report не раскрывает ПДн туристов;
- service role не используется клиентом и не подменяет RLS/RPC tests.

## 6. Lead lifecycle contract

### 6.1. Нормальная state machine

```text
new → contacted → confirmed → completed
  └──────────────→ cancelled
                 → no_show
```

Допустимые переходы:

| From | To | Кто | Обязательные условия |
|---|---|---|---|
| new | contacted | agent+ | первый реальный контакт/попытка, timestamp |
| new | cancelled | agent+ или допустимый customer request | reason |
| contacted | confirmed | agent+ | согласованы дата/следующий шаг |
| contacted | cancelled | agent+ | reason |
| confirmed | completed | agent+, предпочтительно manager | поездка прошла, guests fact |
| confirmed | no_show | agent+, предпочтительно manager | дата наступила, reason |
| confirmed | cancelled | agent+ | отмена до результата, reason |

Обратные переходы обычному UI запрещены. Ошибка исправляется отдельным
admin/manager correction RPC с обязательной причиной и immutable audit event.
`completed` означает состоявшийся визит, а не закрытие карточки.

### 6.2. Server API

В H1.2 прямой generic update статуса заменяется RPC:

```text
transition_tourism_lead_v1(
  lead_id,
  expected_updated_at,
  target_status,
  reason?,
  actual_guests_count?
)
```

RPC обязан:

- проверить membership/role и принадлежность lead;
- проверить разрешённый transition;
- использовать optimistic concurrency через `expected_updated_at`;
- установить server timestamps/actor;
- записать один audit event;
- идемпотентно создать нужное уведомление;
- для `completed/no_show` создать или обновить outcome/ledger draft;
- никогда не принимать actor, organization или monetary result от клиента без
  серверной проверки.

## 7. H1.1 — Conversion-safe public entry

### 7.1. Routes

```text
/tourism
/tourism/o/{operator_slug}
/tourism/o/{operator_slug}/t/{tour_slug}
/wineries/{winery_slug}
```

- H1.1a exact-tour остаётся canonical и единым submit surface;
- H1.1b operator page показывает только approved public fields и current
  published tours;
- winery page остаётся отдельной canonical сущностью, но использует общие с
  operator page визуальные токены, секции и gallery-компоненты;
- связь тура с винодельней всегда explicit и verified: её нельзя выводить по
  совпадению названия, адреса или координат;
- H1.1c catalog использует curated выдачу и query `/tourism?q=...`;
- `tour.winepool.ru` остаётся short redirect layer, не canonical;
- `ref/referral/source` сохраняются operator → tour → form;
- query search не создаёт индексируемые canonical-дубли;
- public title/subtitle являются самостоятельными редакционными полями тура и
  не вычисляются из pickup point, winery, route stop или названия оператора.

### 7.2. Bounded public projections

Добавить versioned RPC, а не anonymous raw-table reads:

```text
get_public_tourism_operator_v1(operator_slug)
list_public_tourism_catalog_v1(query?, region?, cursor?, limit?)
```

Allowlist operator projection:

- slug, public name, organization type;
- verified marker, approved short story, city/region;
- logo/hero media public URLs and alt text;
- current published-tour cards from Published Tour Contract.

До зафиксированной заявки публичная operator/catalog/exact-tour projection не
возвращает website, phone, email, messenger или иной direct booking channel.
Название, логотип, verified identity и история партнёра остаются публичными для
доверия, но единственный pre-lead conversion path ведёт через общую WinePool
lead form. Прямой контакт раскрывается только в контексте уже созданной заявки:
оператору — для обработки, туристу — после подтверждения поездки через Trip
Center, памятку/билет или адресное сообщение.

Запрещены также notification destinations, private contact fields,
member/user IDs, business internals, draft rows, vehicles и lead data. Catalog
RPC возвращает не более 24 элементов за страницу, stable cursor и только
current publication.

### 7.3. UI и visual gate

На 31.08.2026 owner’ом приняты desktop-first направления operator page,
winery page и общая gallery-механика. Детальный контракт зафиксирован в
`h1_02_public_operator_winery_profiles_tz_2026_08_31.md`.

Открытым visual gate остаётся:

1. catalog desktop/tablet/mobile;
2. catalog loading, empty, error, paused/unpublished и zero-results;
3. переходы между catalog/operator/exact-tour с referral;
4. lightweight pre-Flutter shell/skeleton, если он визуально заметен.

Существующий native `TourismOperatorScreen` остаётся источником поведения.
Web-композиция строится по принятому H1-02 ТЗ, без организации-level «логики
маршрута». Навигация профиля — якорная; туры не скрываются вкладками. Галерея
единая для tour/operator/winery: desktop/tablet использует крупный активный кадр
и вертикальную ленту превью, mobile — swipe/PageView со стрелками и
индикатором; в обоих случаях доступен fullscreen.

### 7.4. SEO/share

- route-specific title, description, canonical, OG image и JSON-LD берутся из
  bounded public projection;
- sitemap содержит только разрешённые current routes;
- paused/archived/unpublished не раскрывают snapshot и возвращают честный
  fallback/HTTP contract;
- shell не становится второй CMS.

### 7.5. Performance gate

До расширения платного трафика:

- cold transferred bytes и Flutter first frame измеряются
  `tool/measure_tourism_web.mjs` на фиксированном профиле;
- первый полезный title/value/CTA должен появляться до полной загрузки тяжёлых
  carousel sections;
- изображения имеют responsive derivatives и lazy loading вне hero;
- убрать несжатый JS delivery либо компенсировать его более лёгким entrypoint;
- `CLS <= 0.1`, нет horizontal overflow и fatal console errors;
- baseline/последующий результат записываются рядом, без ложного CanvasKit LCP.

Текущие `9.17 MB / 17.845 s` являются исходной точкой, не целевым качеством.
Перед первой масштабной/платной кампанией требуется отдельный performance budget
decision на реальном целевом устройстве и сети.

## 8. H1.2 — Lead operations and SLA

### 8.1. Operator queue

Минимальные views:

- новые и просроченные;
- в работе;
- подтверждённые ближайшие;
- completed/no-show/cancelled archive;
- filters: tour, responsible, date, referral, status;
- cursor pagination, server counts и явная ошибка вместо ложного empty state.

Чтение выполняется bounded RPC с public/internal DTO, а не широким `select *`.
Contact ПДн выдаются только члену назначенной организации или admin.

### 8.2. Assignment

- у lead всегда одна assigned organization;
- default responsible берётся из experience/member policy;
- если active responsible отсутствует, lead попадает в unassigned queue и
  уведомляет organization owner/manager;
- reassignment требует agent+ и создаёт audit event;
- удаление/пауза member не оставляет невидимые заявки.

### 8.3. SLA

- target `new → contacted`: 15–60 минут;
- `first_response_due_at` вычисляется сервером при создании;
- reminder отправляется один раз до/на deadline;
- overdue escalation идёт manager/admin только если `contacted_at is null`;
- worker/job идемпотентен и хранит `reminded_at/escalated_at`;
- scheduler вызывает ограниченный RPC; механизм cron подтверждается на
  production отдельно, а не предполагается документацией.

### 8.4. Notifications

Нормализовать semantic types:

```text
tourism_lead_created
tourism_lead_sla_reminder
tourism_lead_sla_escalated
tourism_lead_status_changed
tourism_lead_trip_changed
tourism_lead_customer_action
tourism_lead_operator_message
tourism_lead_outcome_recorded
```

Legacy `draft_submission_created` больше не создаётся для новых tourism leads,
но исторические notifications не переписываются. Dedup key строится по
lead/event/recipient, а не текущему времени.

### 8.5. Concurrency and failure UX

- все critical mutations имеют optimistic version/updated timestamp;
- конфликт показывает свежую карточку и не затирает чужое изменение;
- notification failure не откатывает business fact, но попадает в retry queue;
- UI не маскирует RPC/PostgREST ошибку как отсутствие заявок;
- retry transition не создаёт второй event/notification.

## 9. H1.3 — Customer Trip Center and continuity

### 9.1. Authenticated baseline

Trip Center продолжает использовать существующие:

- status и timeline;
- operator messages/customer clarification;
- pickup/time/vehicle/recognition hint;
- ticket/photo/download;
- cancellation policy и customer request;
- ticket delivery, если реально используется.

В H1 добавляются notifications для `completed/no_show`, post-visit CTA и
честные empty/pending states, когда trip data ещё не назначены.

Прямой способ связи с оператором не является публичным discovery field. Он
появляется у туриста только после `confirmed` либо раньше в адресном сообщении,
если это необходимо для исполнения уже зафиксированной заявки. Раскрытие не
удаляет WinePool attribution и не превращает заявку в неучтённый direct lead.

### 9.2. Guest → account claim

Guest submit не должен навсегда терять continuity. Рекомендуемый контракт:

1. submit RPC возвращает opaque one-time claim secret только при создании;
2. DB хранит только hash, expiry и consumed timestamp;
3. success state хранит secret локально ограниченное время и мягко предлагает
   войти/зарегистрироваться;
4. после auth вызывается `claim_tourism_lead_v1(secret)`;
5. RPC связывает lead с `auth.uid()`, инвалидирует secret и пишет audit;
6. повторный claim тем же user идемпотентен, другим user запрещён;
7. raw lead UUID, phone/email и claim secret не попадают в analytics/logs/URL.

Claim не обязателен для обработки заявки: оператор связывается по оставленному
каналу. Он нужен только для безопасного Trip Center и app notifications.

### 9.3. Post-visit action

После `completed` customer получает максимум один релевантный CTA:

- сохранить посещение/дегустацию;
- оставить review;
- вернуться к маршруту/оператору.

CTA создаёт существующий canonical server fact. `no_show/cancelled` не
показывают ложный post-visit сценарий.

## 10. H1.4 — Outcome and revenue ledger

### 10.1. Принцип

WinePool не принимает оплату тура. Ledger фиксирует доказанный результат и
ручные расчёты с партнёром. Он не является кассой, invoice provider или
бухгалтерской системой.

### 10.2. Допустимые базы монетизации и qualified lead

Предыдущие pilot-решения WinePool допускали две честные базы:

1. фикс за квалифицированную заявку/группу;
2. success fee за подтверждённый состоявшийся визит.

H1 сохраняет обе модели. Публичное размещение само по себе не является платным
результатом. Конкретная база и ставка выбираются только в versioned terms после
первых фактов пилота; первые `N` заявок могут быть явно `waived`.

`qualified_lead` не равен любому submit. Для pilot-контракта это external,
non-test, consent-backed и не дублирующая заявка на current published тур, где
есть имя, допустимый контакт, дата/период, число гостей и успешная доставка в
назначенную организацию. Сервер создаёт qualification candidate, а решение
`qualified/rejected` фиксируется отдельным actor/timestamp/reason event по
правилам terms. Допустимые rejection reasons ограничены: duplicate, spam,
invalid_or_unreachable_contact, out_of_scope, customer_withdrew_before_contact.
Оператор может предложить rejection, но не может единолично обнулить fee:
спорный случай подтверждает WinePool admin с сохранением evidence reference без
ПДн в partner report.

### 10.3. Versioned partner terms

Добавить `tourism_partner_terms`:

```text
id, organization_id, experience_id?
basis: qualified_lead | completed_visit | manual
model: not_applicable | fixed | per_guest | percent | manual | waived
currency: RUB
fixed_fee_minor?
per_guest_fee_minor?
percent_basis_points?
effective_from, effective_to?
status: draft | approved | expired
agreement_reference?, approved_by, approved_at
created_at, updated_at
```

- одновременно действует максимум один approved contract на одинаковый scope;
- monetary values хранятся integer minor units, проценты — basis points;
- `agreement_reference` — внутренний bounded reference, не публичный документ;
- terms фиксируют qualification definition, attribution window и правило, что
  перевод уже полученного WinePool lead в телефон/мессенджер не удаляет
  attribution и согласованную обязанность по отчёту/расчёту;
- значения snapshot’ятся в ledger и не меняют прошлый outcome задним числом.

### 10.4. Lead revenue ledger

Добавить `tourism_revenue_ledger` один-к-одному с lead:

```text
lead_id unique
organization_id, experience_id, referral snapshot
basis_fact: qualified_lead | completed | no_show
qualification status/actor/time/reason snapshot?
outcome: completed | no_show | null
actual_guests_count
terms snapshot / model / currency
basis_amount_minor?
winepool_amount_minor?
status: not_applicable | pending | confirmed | invoiced | paid | waived
confirmed_by/at, invoiced_at, paid_at, waived_reason?
created_at, updated_at
```

И append-only `tourism_revenue_ledger_events` с actor, old/new status, monetary
snapshot и reason. Запрещено физически удалять ledger обычному UI.

### 10.5. Calculation

- `qualified_lead + fixed`: фикс после audited qualification fact;
- `completed_visit + fixed`: фикс после confirmed completed fact;
- `per_guest`: rate × actual guests;
- `percent`: basis amount × basis points / 10 000 с documented rounding;
- `manual`: admin вводит итог и причину;
- `waived/not_applicable`: сумма ноль, причина/статус явные;
- no-show по умолчанию `not_applicable`, если terms явно не говорят иное.

Gross/basis относится только к туристической услуге. Алкогольные покупки и их
оплата в H1 ledger не включаются.

До выбора terms qualification/submit/completed хранятся как отдельные facts и
не преобразуются задним числом друг в друга.

### 10.6. Legal/accounting gate

До первого `invoiced/paid` владелец отдельно подтверждает договорную,
налоговую и документальную схему. До этого разрешены `pending`, `confirmed`,
`waived`, но UI не обещает автоматический счёт или юридически завершённую оплату.

## 11. H1.5 — Partner report v1

### 11.1. Server facts for acquisition

Для opens/views/starts добавить privacy-safe append-only
`tourism_acquisition_events` и RPC `record_tourism_acquisition_event_v1`:

```text
event_type: public_open | operator_view | experience_view | lead_started
opaque session/attempt id
server occurred_at
operator/experience/referral snapshot
is_test, safe actor_type, build_mode
canonical path, без raw query/ПДн
```

RPC нормализует referral по registry, ограничивает cardinality/rate,
дедуплицирует один milestone и не принимает contact/comment/email/phone/IP.
`lead_submitted` и последующие этапы берутся из `tourism_leads/events`, а не
дублируются клиентским truth.

### 11.2. Report RPC

`admin_tourism_partner_report_v1(from, to, organization, include_test=false)`
и owner/member-safe вариант должны возвращать только агрегаты:

- public opens, operator/tour views, lead starts/submits;
- qualified/rejected qualification facts и bounded rejection reasons;
- contacted/confirmed/completed/no-show/cancelled;
- conversion numerator/denominator на каждом шаге;
- median/P90 first response и SLA breaches;
- breakdown по referral source/code/material;
- post-visit actions;
- ledger confirmed/invoiced/paid/waived totals;
- low-sample и data-quality flags.

Raw/clean правила H0 сохраняются. Test/QA не смешивается с commercial.
Отчёт не содержит customer rows, contact, comments, UUID или claim secrets.

### 11.3. Delivery

Первый формат — owner-generated CSV/PDF плюс защищённый aggregate preview.
Полноценный BI и самостоятельный кабинет не являются gate первого платежа.
Каждый weekly report содержит anomaly, top loss, one action и data-quality
state.

## 12. H1.6 — Pilot launch kit

### 12.1. Registry readiness

Расширить referral registry операционными полями без ПДн:

- responsible role/label;
- placement type;
- material revision;
- contact channel verified timestamp;
- placement verified/activated/last checked timestamps;
- paused reason.

Фактический contact остаётся в private organization/member data. Код становится
active только после проверки destination, QR, responsible и SLA.

### 12.2. Materials

Для каждой активной точки:

- QR A5/A4 или compact card;
- short measurable URL;
- название тура/оператора и честная формулировка заявки;
- инструкция сотруднику в 3–5 предложениях;
- FAQ без обещания guaranteed booking;
- private escalation sheet;
- material revision и дата проверки.

### 12.3. Activation procedure

1. открыть QR в mobile browser без приложения;
2. проверить canonical exact tour/referral;
3. проверить Android installed/uninstalled behavior;
4. iOS device — при появлении устройства; до этого пометка contract-only;
5. выполнить test lead через отдельный QA code;
6. проверить одного recipient и SLA path;
7. активировать commercial code;
8. записать placement date и ответственного;
9. повторять health check минимум еженедельно.

### 12.4. 30-day targets

- 1–3 актуальных тура;
- один ответственный оператор;
- 5–10 активных referral-точек;
- 100 meaningful public opens;
- 10 submitted external leads;
- 3–5 confirmed completed visits;
- ноль потерянных заявок из-за routing/notification;
- минимум один partner payment signal или документированный `waived` с
  готовностью обсуждать следующий платный цикл.

Это цели, не обещания. Малый трафик сам по себе не провал; провал — потерянные
заявки, отсутствие ответа или невозможность подтвердить результат.

## 13. Data model и migration packages

Миграции additive, backward-compatible и применяются по одной после backup,
dry-run и SQL contract. Нельзя применять все локальные migration подряд.

### Package A — H1 public measurement

- bounded operator/catalog RPC;
- acquisition events/RPC;
- referral operational fields;
- public grants только exact functions.

### Package B — H1 lead operations

- server transition/assignment RPC;
- qualification status/actor/reason и audited dispute path;
- optimistic concurrency;
- SLA timestamps/job;
- normalized notification types/dedup;
- claim hash/expiry/RPC;
- completed/no-show customer notifications.

### Package C — H1 money facts

- partner terms;
- revenue ledger/events;
- outcome-to-ledger trigger/RPC;
- strict RLS and audit.

### Package D — H1 report

- aggregate report core/RPC;
- CSV/owner report runner;
- weekly operating extension.

Не создавать отдельные параллельные lead/booking tables, пока текущий lead
contract способен выразить pilot. Full booking/slot engine остаётся вне H1.

## 14. Security, privacy and abuse controls

- anon получает только bounded public RPC и submit/acquisition RPC;
- raw operational tables закрываются от anon после compatibility audit;
- contact/comment/consent evidence доступны строго operator organization/admin;
- checkbox согласия по умолчанию выключен; submit без согласия запрещён, а
  server fact хранит purpose, policy/consent version, surface и server time;
- privacy/consent links доступны до submit; формулировки, роли
  operator/controller, локализация и retention проходят отдельный legal review
  до коммерческого масштабирования;
- public reports и analytics не содержат ПДн;
- claim secrets хэшируются, имеют expiry, single-use и не логируются;
- acquisition session/attempt opaque и не связывается с advertising identity;
  network data может кратковременно использоваться только для abuse control и
  не сохраняется как поле partner analytics;
- referral snapshot immutable после lead submit;
- rate limit есть отдельно для events, submit и claim attempts;
- RLS проверяет membership status/role и assigned organization;
- financial rows доступны admin и ограниченному owner/manager scope;
- destructive delete outcome/ledger запрещён; correction только audit RPC;
- backup/dump с ПДн не копируется в репозиторий и не выводится в чат.

## 15. Analytics dictionary additions

Client/server semantics:

| Event/fact | Source of truth | Notes |
|---|---|---|
| tourism_public_open | acquisition RPC | server timestamp, referral snapshot |
| tourism_operator_view | acquisition RPC | bounded operator |
| tourism_experience_view | acquisition RPC | exact published tour |
| tourism_lead_started | acquisition RPC | one per opaque attempt |
| tourism_lead_submitted | tourism_leads | server fact |
| tourism_lead_qualified | lead qualification event | server/admin-audited fact; не client event |
| tourism_lead_contacted | lead status/event | server fact |
| tourism_lead_confirmed | lead status/event | server fact |
| tourism_visit_completed | outcome status/event | server fact |
| tourism_visit_no_show | outcome status/event | server fact |
| tourism_post_visit_action | existing target server fact | do not fake client success |
| tourism_revenue_confirmed/paid/waived | ledger/event | admin/partner fact |

`actor_type`, `build_mode`, `is_test` и privacy payload policy наследуются из
H0 event dictionary. Event text, phone, email, comments and raw URLs forbidden.

## 16. UI surfaces and owner checkpoints

| Surface | Baseline | H1 action | Owner visual gate |
|---|---|---|---|
| exact-tour web | accepted | performance/error hardening | только заметные изменения |
| operator web | composition accepted 31.08.2026 | H1.1b build | принят; повторно только при заметном изменении |
| winery web | composition accepted 31.08.2026 | отдельный reviewable H1-02C slice | принят; не блокирует H1-03 |
| catalog web | legacy/static | H1.1c build | обязательно до реализации |
| operator CRM | working admin list | SLA/assignment/conflict UX | перед layout redesign |
| customer Trip Center | implemented | claim/outcome/post-visit | перед новым claim/success UX |
| partner report | absent | aggregate preview/export | согласовать первый report layout |

Codex останавливается перед каждым новым визуальным решением. Schema, RPC,
tests и read-only audits можно выполнять заранее, если они не фиксируют
непринятую композицию.

## 17. Test plan

### 17.1. SQL contracts

- anon public allowlist и raw-table denial;
- exact operator/catalog publication filtering;
- referral normalization/snapshot/test exclusion;
- event idempotency/rate limit/prohibited payload;
- every allowed/forbidden status transition;
- stale optimistic update rejected;
- assignment/member role/RLS matrix;
- SLA reminder/escalation dedup;
- guest claim success, expiry, replay and cross-user denial;
- completed/no-show actor/timestamp and correction audit;
- qualified lead candidate, allowed rejection reasons, dispute audit и
  отсутствие partner self-waive;
- terms overlap rejection and deterministic calculation;
- ledger immutability/RLS/events;
- report raw-clean reconciliation and no PII;
- failed transaction leaves no partial business fact.

### 17.2. Dart tests

- public DTO/model forward compatibility;
- referral preservation across routes;
- pagination/loading/empty/error states;
- lead state machine labels/actions;
- conflict/retry does not duplicate mutation;
- claim continuation and secret redaction;
- Trip Center status/outcome/post-visit states;
- report low-sample/NULL semantics;
- notification deep-link routing.

### 17.3. Web/device QA

- desktop/tablet/mobile accepted breakpoints;
- browser with and without app installed;
- Android App Link exact route/referral;
- iOS contract, then physical device when available;
- keyboard/touch/accessibility and no overflow;
- cold/warm performance profiles;
- submit success/duplicate/claim/account flow;
- operator notification and SLA escalation;
- partner CSV/PDF totals match DB aggregates.

### 17.4. Production smoke

Только отдельные `is_test=true` codes. Один bounded E2E:

```text
open → view → start → submit → contacted → qualified/rejected
→ confirmed → completed/no_show → report → test excluded
```

Не повторять device/build/deploy проверки без изменения соответствующего
контракта. Ни email, ни phone, ни UUID не выводятся в handoff.

## 18. Rollout and rollback

1. Read-only production audit и точный backup package.
2. Package A migration, SQL contract, public RPC smoke.
3. Public data adapter без visual redesign.
4. H1-02A contract/data readiness, затем H1-02B operator implementation/deploy.
5. H1-02C winery profile отдельным reviewable пакетом, не задерживая H1-03.
6. Package B и operator/customer smoke.
7. Активировать одну commercial point и наблюдать SLA.
8. Package C после принятия ledger semantics/legal boundary.
9. Package D и первый partner report.
10. Расширить до 5–10 точек только после отсутствия потерянных leads.
11. Через 30 дней H1 gate review.

Каждый deploy scoped и имеет backup. Rollback кода не удаляет созданные leads,
events/outcomes/ledger; schema rollback по умолчанию — disable feature/RPC и
сохранение данных, а не destructive drop.

## 19. Implementation order по сессиям

### H1-01 — bounded public data audit

- live schema/RLS/RPC audit;
- operator/catalog DTO и Package A design;
- никаких UI изменений;
- owner принимает public fields и routing.

### H1-02 — public operator and winery profiles

- H1-02A: schema/RPC/media audit и explicit tour → winery relation design;
- H1-02B: public operator page, unified editorial gallery, SEO, tests и deploy;
- после H1-02B H1-03 может продолжаться независимо;
- H1-02C: public winery visit page с раздельными direct winery programs и
  verified third-party tours;
- H1-02D: moderated structured authoring без free-form page builder;
- по одному navigation/referral smoke для operator и winery routes.

### H1-03 — lead operations/SLA

- Package B transition, assignment, notifications, scheduler;
- CRM conflict/error/SLA UX без большого cabinet redesign;
- production test lead.

### H1-04 — customer continuity

- guest claim contract;
- совместный success/claim UX gate;
- Trip Center outcome/post-visit completion.

### H1-05 — catalog/basic search

- совместный H1.1c visual gate;
- curated catalog, bounded query and SEO;
- при двух турах не строить сложные filters.

### H1-06 — outcome/revenue ledger

- partner terms decision и legal boundary;
- Package C migration/RPC/audit;
- один test completed и waived/confirmed ledger flow.

### H1-07 — partner report and launch

- Package D;
- launch materials и activation 5–10 points;
- 30-day weekly cadence и final gate review.

Порядок H1-02/H1-03 может идти параллельно организационно, но production
изменения остаются отдельными reviewable пакетами.

## 20. Acceptance H1 → next horizon

H1 закрывается, когда одновременно:

1. public route ведёт к конкретному актуальному туру без обязательной установки;
2. referral сохраняется от open до lead/outcome/report;
3. минимум 5 точек активированы и имеют health journal;
4. external lead получает ответ в измеряемом SLA;
5. есть минимум один реальный external completed visit;
6. completed подтверждён actor/timestamp и не является тестом;
7. guest/auth customer не теряет continuity, Trip Center безопасен;
8. partner report сходится с events/leads/ledger;
9. partner подтверждает полезность;
10. есть реальный payment signal: confirmed/invoiced/paid либо осознанный
    `waived` с готовностью обсуждать следующий цикл;
11. нет утечки ПДн, test/QA в commercial и потерянных notification;
12. owner принимает решение continue/correct/pivot по 30-дневным данным.

## 21. Open decisions и безопасные defaults

| Decision | Когда нужен | Default до решения |
|---|---|---|
| фактические 5–10 placements/responsibles | перед активацией codes | inactive |
| operator/winery page composition и gallery | принято 31.08.2026 | следовать `h1_02_public_operator_winery_profiles_tz_2026_08_31.md`; заметные отклонения вернуть на owner review |
| claim UX и срок token | H1-04 | 72 часа, single-use, только после auth |
| catalog composition | H1-05 | реализацию UI не начинать |
| monetization basis/rate: qualified lead или completed success fee | H1-06 | первые согласованные `N` фактов `waived`, далее `pending`, без обещания оплаты |
| legal/tax invoice flow | до invoiced/paid | никаких платежных действий в WinePool |
| paid traffic performance budget | до масштабирования | только ограниченный pilot traffic |
| iOS physical acceptance | при доступном устройстве | contract/server verified marker |

Решение open item не должно задним числом менять immutable acquisition,
consent, outcome или terms snapshot.
