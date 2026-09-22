# H1 Tourism Revenue Loop — prompt следующей сессии

Дата подготовки: 31.08.2026  
Назначение: начать H1 с H1-01 без повторного изучения H0, старых roadmap и
посторонней системы исследования вина/винодельни.

Ниже находится готовый prompt для новой Codex-сессии.

---

Продолжаем WinePool после закрытия H0 и согласования полного H1 Tourism
Revenue Loop.

## Авторитетные документы

Главный implementation source of truth:

- `docs/h1_tourism_revenue_loop_tz_2026_08_30.md`

Supporting plan только для публичных operator/catalog surfaces:

- `docs/h1_tourism_public_operator_catalog_discovery_plan_2026_08_30.md`

Общая очередь горизонтов:

- `docs/post_release_1_1_0_execution_roadmap_2026_08_25.md`

Перед любым доступом к production VPS обязательно полностью прочитать:

- `docs/powershell_ssh_sql_quoting_guide.md`

Другие старые Tourism-ТЗ и roadmap не читать, если конкретный факт нельзя
найти в этих документах, актуальных migrations/tests или текущем коде.
Система catalog research вина/винодельни не относится к этой сессии.

## Зафиксированное состояние

- H0 полностью закрыт 30.08.2026.
- H1 source-of-truth зафиксирован коммитом `9cc25188`.
- После него отдельным несвязанным коммитом `9ad47553` сохранена система
  исследования вина/винодельни; её не анализировать и не изменять.
- Published Tour Contract v1 работает на production.
- Canonical exact-tour web-first route:
  `/tourism/o/{operator_slug}/t/{tour_slug}`.
- Принятый production пример:
  `/tourism/o/yalta-excursions/t/yalta-massandra-tasting`.
- Есть один активный оператор и два published тура.
- Guest/auth lead form, consent evidence, idempotent submit, referral snapshot,
  statuses, audit events, Trip Center и operator CRM уже существуют.
- Android App Links приняты на реальном устройстве.
- iOS AASA/server contract проверен; реального Apple-устройства нет, поэтому
  physical acceptance остаётся явно незакрытым, но не блокирует H1-01.
- Пять non-test referral points подготовлены, но оставлены `is_active=false`
  до фактического размещения и проверки ответственного/SLA.
- Owner/QA/test traffic не входит в clean commercial показатели.
- Новый web UI нельзя проектировать или реализовывать без совместного owner
  visual acceptance. Это относится к operator page, catalog, claim/success UX
  и report layout.
- Push запрещён без отдельного указания владельца.

## Правильная нумерация

- H1.1a — существующая exact-tour page, baseline завершён в H0;
- H1.1b — public operator page;
- H1.1c — curated `/tourism` catalog/basic search;
- H1.2 — lead operations/SLA;
- H1.3 — Trip Center и guest-to-account continuity;
- H1.4 — outcome/revenue ledger;
- H1.5 — partner report v1;
- H1.6 — launch kit и 30-дневный pilot.

Основной H2 — Retail Availability and First Store. Advanced Tourism discovery
находится в отдельном growth backlog и не называется H2.

## Задача этой сессии: H1-01

Выполнить один сфокусированный пакет `bounded public data audit` и подготовить
реализацию Package A. Не начинать визуальную разработку operator/catalog.

### 1. Git и локальный baseline

1. Проверить branch/status/recent commits.
2. Не менять и не откатывать чужие изменения.
3. Подтвердить наличие коммитов `9cc25188` и `9ad47553` в ancestry текущего
   HEAD.
4. Прочитать полностью главный H1-ТЗ и только необходимые разделы supporting
   plan.

### 2. Один локальный discovery pass

Проверить только следующие поверхности:

- `lib/main_tourism_web.dart`;
- `lib/features/winery_tourism/data/winery_tourism_repository.dart`;
- `lib/features/winery_tourism/domain/published_tour_v1.dart`;
- `lib/features/winery_tourism/domain/tourism_experience.dart`;
- `lib/features/winery_tourism/domain/tourism_lead.dart`;
- `lib/features/winery_tourism/presentation/tourism_operator_screen.dart`;
- `lib/features/winery_tourism/presentation/tourism_list_screen.dart`;
- `lib/features/winery_tourism/presentation/tourism_published_web_screen.dart`;
- migrations `20260602_*tourism*` — `20260829_*tourism*` только по найденным
  зависимостям;
- `supabase/tests/20260827_published_tour_contract_v1.sql`;
- `supabase/tests/20260826_tourism_lead_submission_contract.sql`;
- `supabase/reports/h0_weekly_operating_report.sql`.

Не делать широкий аудит всего Flutter-приложения или всех migrations.

### 3. Read-only production audit

После чтения VPS-runbook выполнить один bounded audit без вывода email,
телефонов, UUID, contact destinations, comments или других ПДн в чат/log
handoff.

Проверить агрегатами и schema metadata:

1. Количество/current status публичных операторов и published tours.
2. Реальные public organization/experience fields и какие из них сейчас
   читаются anonymous client напрямую.
3. Сигнатуру и grants Published Tour RPC.
4. RLS/grants для:
   - `tourism_organizations`;
   - `tourism_experiences` и publication/snapshot tables;
   - `tourism_organization_members`;
   - `tourism_leads`/`tourism_lead_events`;
   - referral registry/outcomes.
5. Наличие legacy anonymous raw-table paths, которые потребуется заменить
   bounded RPC.
6. Какие approved public поля достаточны для operator card/page и catalog
   card без раскрытия members, notification routing, vehicles, leads и drafts.
7. Состояние подготовленных referral points только агрегатами:
   active/test counts и readiness gaps.

Если production доступ недоступен, завершить локальную часть и зафиксировать
один конкретный blocker; не подменять live facts предположениями.

### 4. Public contract, который нужно спроектировать

Подготовить точный additive contract Package A:

```text
get_public_tourism_operator_v1(operator_slug)
list_public_tourism_catalog_v1(query?, region?, cursor?, limit?)
record_tourism_acquisition_event_v1(...)
```

Требования:

- versioned RPC и явный allowlist колонок;
- только current published content;
- operator/catalog DTO не содержит internal IDs и ПДн, если они не нужны
  публичному маршруту;
- limit не больше 24, stable cursor;
- title тура — самостоятельное editorial field, не вычисляется из pickup или
  route points;
- `ref/referral/source` сохраняются до exact tour и submit;
- acquisition events append-only, privacy-safe, rate-limited и idempotent;
- opens/views/starts — server facts, submit/outcome остаются facts
  `tourism_leads/events`;
- никаких raw query, email, phone, comment, claim secret в analytics;
- paused/draft/archived content не раскрывается;
- anonymous grants выдаются только конкретным functions после contract tests.

### 5. Результаты H1-01

К концу сессии должны существовать:

1. Короткий фактический audit/handoff без ПДн: current/local/live/gap.
2. Таблица public allowlist для operator и catalog DTO.
3. Точные RPC signatures, pagination/error semantics и RLS/grant matrix.
4. Migration plan Package A с rollback/disable strategy.
5. SQL contract test plan:
   - publication filtering;
   - anonymous allowlist и raw-table denial;
   - stable pagination;
   - referral normalization;
   - event idempotency/rate limit/prohibited payload;
   - draft/private-field leakage denial.
6. Список найденных compatibility consumers перед закрытием raw reads.
7. Явная точка следующего owner decision: набор public fields/routing.

Если contract однозначен и не требует нового visual/product решения, можно
создать migration и SQL tests Package A в этой же сессии. Production mutation
разрешена только после:

- точного backup/control export по VPS-runbook;
- dry-run/contract checks;
- применения только migration этого H1-пакета;
- одного bounded smoke;
- отсутствия ПДн в сообщениях.

Не применять все локальные migrations подряд. Не активировать referral points
без подтверждённого физического размещения. Не деплоить новый operator/catalog
UI. Не менять accepted exact-tour дизайн без owner approval.

## Verification и commit

- Batch SQL/Dart checks; не повторять успешные H0 device/deploy tests без
  изменения соответствующего контракта.
- `git diff --check` обязателен.
- Зафиксировать завершённый H1-01 отдельным осмысленным commit.
- Не выполнять push.
- Финальный ответ должен назвать: live facts агрегатами, созданные artifacts,
  tests, production actions/backup (если были), commit и точный следующий
  owner gate.

Начинай самостоятельно. Первое короткое сообщение пользователю: какие public
server facts/RPC/RLS и локальные файлы проверяешь. Затем работай до готового
H1-01 результата, останавливаясь только перед новым визуальным решением или
реально отсутствующим разрешением.

## Completion note — 31.08.2026

Этот prompt исполнен. Итоговый статус и production evidence находятся в
`h1_01_public_data_audit_handoff_2026_08_31.md`; решение по контактам и
монетизации — в `h1_tourism_monetization_contact_decision_2026_08_31.md`.
Package A применён и smoke-verified, card allowlist/canonical routing/
publication-time ordering приняты, native public operator/catalog adapter
реализован. Следующим пакетом UUID/home-hero compatibility переведены на bounded
RPC и anonymous raw SELECT для семи tourism tables отозван в production. Не
использовать раздел «Задача этой сессии» как новый незакрытый handoff.
