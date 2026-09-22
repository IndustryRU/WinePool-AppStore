# H0.5a — Published Tour Contract v1 audit

Дата: 27.08.2026; реализация завершена 29.08.2026
Статус: implemented, production activated and smoke verified
Граница: техническая зависимость H0 web-first tourism; без полного Tourism CRM
redesign, marketplace, payments и operator self-service.

Связанный визуальный/lead-flow контракт:
[`h0_tourism_web_first_lead_flow_tz_2026_08_26.md`](h0_tourism_web_first_lead_flow_tz_2026_08_26.md).

## 1. Решение по результатам аудита

Не начинать responsive public web UI поверх прямых `select('*')` из текущих
tourism-таблиц. Сначала выполнить короткий contract-first пакет:

1. зафиксировать `Published Tour Contract v1`;
2. отделить редактируемое состояние от атомарно опубликованного snapshot;
3. структурировать schedule, price и visual story;
4. дать app/web один versioned public projection;
5. минимально укрепить admin authoring/readiness/publish/preview;
6. только затем собирать утверждённые desktop/tablet/mobile compositions.

Полный редизайн CRM и native tour screen до web UI не требуется. Требуется
стабильная граница между authoring и всеми public surfaces.

## 2. Что проверено

Read-only аудит охватил:

- production PostgreSQL schema, constraints, policies, triggers и фактическую
  заполненность tourism records;
- оба production published tours и одного production tour operator;
- `TourismExperience`/draft models;
- repository queries/hydration и public route resolution;
- Tourism CRM editor, readiness checklist, stops/pickups/vehicles/media editors;
- native public tour composition;
- статическую exact-tour landing page;
- утверждённые desktop/tablet/mobile/form/success references.

Production mutation, backup и migration для аудита не требовались и не
выполнялись. Email, phone, UUID, account identity и другие персональные данные в
документ не включены.

## 3. Production facts — 27.08.2026

### 3.1. Общая картина

| Fact | Massandra | Inkerman |
|---|---:|---:|
| status | published | published |
| CRM readiness | 100% | 86% |
| stops | 6 | 6 |
| pickup points | 1 | 1 |
| active vehicles | 2 | 1 |
| approved experience media | 9 | 8 |
| approved cover | 1 | 1 |
| approved gallery | 8 | 7 |
| captioned experience media | 9 | 0 |
| media with copyright/license metadata | 3 | 0 |
| pickup photos | 3 | 0 |

CRM readiness проверяет только title, cover, non-empty price string, category,
pickup presence, cancellation text и vehicle photo. Он не проверяет content,
schedule, price semantics, gallery order/captions, highlights, program quality,
operator identity, SEO, consent or published snapshot.

### 3.2. Фактические content gaps

- обе цены — одна display string (`price_note`), без currency/amount/payment
  stage/required-vs-optional semantics;
- расписание — одна display string (`nearest_date_label`); recurrence, weekdays,
  time zone, on-request и exceptions не структурированы;
- у Massandra `duration_label` заполнен, но `duration_filter` пуст;
- у Massandra availability tags отсутствуют, у Inkerman они coarse и не
  являются реальным расписанием;
- `Почему сюда едут` как authored entity отсутствует;
- Massandra media частично curated, Inkerman gallery assets имеют одинаковый
  `sort_order=100` и не имеют captions;
- admin upload не сохраняет title/caption/alt/copyright/license/focal point и
  присваивает gallery один default sort order;
- stop time offsets у обоих туров не заполнены; route location evidence
  заполнено только частично;
- Inkerman pickup не имеет photo evidence; active vehicle также без фото;
- operator содержит базовые public brand/contact fields, но нет public about,
  verified/trust statement и authoring readiness для identity block;
- нет SEO title/description/OG projection из общего source of truth;
- нет aggregate published revision: child updates не поднимают
  `tourism_experiences.updated_at`.

### 3.3. Content corruption risk already visible

CRM helper `_lines` делит list-like fields по newline **и запятой**. Поэтому
смысловые пункты с запятыми уже раздроблены в production arrays. Например одна
фраза о переносе/отмене группы и одна составная cost item превращаются в
несколько независимых bullets. Для v1 list editor должен разделять пункты только
явным row/newline action; запятая остаётся частью текста.

## 4. Current chain and gaps

| Layer | Current behavior | Gap for accepted public UI |
|---|---|---|
| DB | normalized base tables plus free-text/arrays | no versioned public contract or atomic snapshot |
| Public RLS | anon reads raw published rows and related tables | payload exposes internal schema and changes with every table edit |
| Repository | `select('*')`, then per-tour/per-child hydration | N+1 queries, tight schema coupling, unsuitable initial web LCP |
| Media | polymorphic `owner_type/owner_id`; generic asset roles | no FK by owner, no alt/focal/aspect/responsive derivatives/story binding |
| Admin save | direct row update; child editors delete then insert | published tour can be temporarily/partially inconsistent; no transaction |
| Publish | changing `status` is sufficient | no validation gate, revision, immutable payload, preview/publish separation |
| CRM readiness | seven presence checks | false confidence: does not cover accepted UI or content quality |
| Native app | highlights inferred from stops and gallery index | title/photo pairing is accidental; captions from DB are discarded |
| Public route | tour resolves by global tour slug | operator slug in `/o/{operator}/t/{tour}` is not validated/canonicalized |
| Static landing | Massandra content duplicated in HTML | second CMS, stale app-first CTA, no Inkerman/source synchronization |
| Tests | lead/RLS contracts and shared lead form | no published-tour schema, mapper, authoring, ordering or snapshot tests |

## 5. Critical technical findings

### P0-1. Public clients depend on internal tables

`WineryTourismRepository` selects all 34 experience columns and separately
queries stops, pickup points, vehicles and media. Every table/schema refactor can
break all clients. A single detail may require multiple sequential requests;
catalog hydration multiplies them per tour.

Required: one versioned public projection/RPC response with an explicit field
allowlist. Raw-table RLS remains temporarily for released client compatibility,
but new web/app code must not depend on it.

### P0-2. No draft/published isolation

Admin writes the same rows that anon reads. Stops, pickups and vehicles use
`delete -> insert` from the client without a transaction. A failed insert after
delete loses the child set, and a published reader can observe the intermediate
state.

Required: editable source plus atomic immutable publication snapshot. Public
surfaces read only the current valid snapshot.

### P0-3. Price and schedule have no semantics

Display strings cannot reliably produce honest split price, total, payment note,
calendar schema, filters, analytics or responsive wrapping.

Required: structured recurrence and ordered price components; human display
labels remain optional authored/fallback fields, not the source of truth.

### P0-4. Approved visual story cannot be authored

Accepted carousel needs curated slide title, description, image, order and crop.
Current app discards media caption metadata and pairs stops with images by list
index modulo image count. Current admin cannot edit captions or reorder gallery
reliably.

Required: authored highlights and media presentation metadata.

### P0-5. Existing readiness is not a publish gate

`published` and `100%` do not imply that the approved page can be rendered.
Inkerman is already public at 86%, and Massandra can show 100% without schedule
semantics, SEO, highlights or image focal points.

Required: server-side v1 validator used by publish RPC and admin preview.

### P1. Public data minimization and route correctness

- raw public vehicle rows include operational fields such as license plate and
  recognition hint before a lead is assigned;
- approved media policy is based on media status only, not on published owner;
- polymorphic media ownership has no database FK;
- exact-tour route ignores operator/tour mismatch;
- public static HTML duplicates production facts manually.

These are not blockers for the audit, but belong in the contract package or
staged compatibility cleanup.

## 6. Published Tour Contract v1

The public contract is a versioned, PII-free, presentation-ready projection. It
contains facts and content, not Flutter widgets or breakpoint decisions.

### 6.1. Envelope

```json
{
  "schema_version": 1,
  "revision": 1,
  "content_hash": "opaque-hash",
  "published_at": "timestamp",
  "canonical_path": "/tourism/o/operator-slug/t/tour-slug",
  "tour": {},
  "operator": {},
  "seo": {}
}
```

Internal UUIDs, responsible user, notification endpoints, source notes and
unassigned vehicle recognition data are absent.

### 6.2. Tour identity and hero

```text
slug
title
destination_title
subtitle
short_value_proposition
description
tour_kind / experience_type
city / region / departure_city
verified_route
cover_media
```

`short_value_proposition` is distinct from long description and bounded for
hero/OG usage.

### 6.3. Schedule and duration

```text
schedule.mode: recurring | fixed_dates | on_request
schedule.timezone
schedule.weekdays[]
schedule.start_time
schedule.summary
schedule.exceptions[] (optional, deferred until needed)
duration.minutes
duration.summary
```

Existing strings are preserved as migration fallback only until both tours are
normalized.

### 6.4. Pricing

```text
pricing.currency
pricing.components[]:
  - key
  - label
  - amount_minor
  - payment_stage: on_confirmation | before_trip | on_site | optional
  - required
pricing.summary
pricing.no_payment_now_message (derived only when facts allow it)
```

No total is inferred when components are optional or conditions are unknown.

### 6.5. Visual story and media

```text
cover:
  url/source set, alt, caption, aspect ratio, focal_x, focal_y
highlights[]:
  key, title, description, media, sort_order
gallery[]:
  media, sort_order
media:
  role, alt, caption, copyright/license, width, height, variants[]
```

Required roles support accepted compositions: `cover`, `highlight`, `gallery`,
`program`, `scenic`. One asset may be reused by reference, not duplicated in
storage. Fallback for 0/1 image follows the approved visual TZ.

### 6.6. Program and practical information

```text
program_steps[]:
  key, title, description, time_offset/duration, location, media, sort_order
transfer:
  included, summary, departure_city, pickup_promise
included[]
not_included[]
requirements[]
payment_terms
cancellation_terms
important_notes[]
```

Operational assigned pickup/vehicle data does not belong in anonymous
acquisition payload. Public tour promises and post-lead Trip Center facts remain
separate concepts.

### 6.7. Operator, lead and SEO

```text
operator:
  slug, name, type, verified, short_about, logo, website
lead:
  enabled, booking_mode, consent_document_version
seo:
  title, description, canonical_path, og_media, structured_data_projection
```

Contact/notification secrets are excluded. Public business contacts may be
added later through an explicit allowlist, not `select('*')`.

## 7. Authoring and publication model

Recommended additive model:

1. existing normalized tables remain authoring source during H0 transition;
2. new structured schedule/price/highlight/media-presentation entities are
   added without removing released columns;
3. `tourism_experience_publications` stores immutable `revision`, validated
   `payload jsonb`, `content_hash`, `published_at` and current/superseded state;
4. `admin_publish_tour_v1` validates and writes one complete snapshot inside one
   transaction;
5. `get_published_tour_v1(operator_slug, tour_slug)` returns the current snapshot
   and rejects operator/tour mismatch;
6. preview calls the same compiler/validator against authoring data but does not
   activate the revision;
7. child edits no longer affect new public clients until explicit publish;
8. legacy raw-table SELECT/RLS is retained during staged released-app migration,
   then tightened in a separately verified compatibility step.

This avoids a destructive rewrite and lets CRM UX evolve later without changing
public clients.

## 8. Minimal pre-UI implementation package

### A. Database and contracts

1. Add structured schedule, pricing, highlights and media presentation data.
2. Add immutable publications plus compile/validate/publish/get RPCs.
3. Backfill and manually verify Massandra and Inkerman.
4. Add deterministic media ordering and published-owner media filtering.
5. Add aggregate revision/hash and exact operator/tour canonical resolution.
6. Keep lead RPC identity compatible with slug during rollout.

### B. Minimal CRM hardening

1. Replace comma-splitting list input with row/newline items.
2. Add structured price and schedule editors.
3. Add highlight editor and media caption/alt/order/focal controls.
4. Expand readiness from seven presence checks to server validator output.
5. Separate `Сохранить черновик`, `Предпросмотр` and `Опубликовать`.
6. Publish only through atomic RPC; surface field-level validation errors.
7. Preview renders from compiled v1 payload, the same payload app/web consume.

Full CRM visual redesign is not required for this package.

### C. App/web adapter

1. Introduce `PublishedTourV1` domain model independent of DB row names.
2. Add repository fetch by `(operatorSlug, tourSlug)` using one RPC.
3. Map native current tour screen through the same adapter before/with web.
4. Remove inferred stop/image carousel pairing.
5. Make static SEO shell read/generate from the same published payload.
6. Do not hand-copy Massandra/Inkerman facts into HTML.

### D. Tests and acceptance

Required SQL tests:

- anon can read only current valid publication;
- draft edits do not change current public payload;
- failed validation/publish is atomic;
- operator/tour mismatch returns not found/canonical error;
- structured price/schedule compile deterministically;
- media order/focal/highlight references are stable;
- no internal UUID/responsible/contact secret/vehicle recognition data leaks;
- legacy direct reads remain available only for the declared compatibility
  window.

Required Dart/widget tests:

- v1 JSON mapping and unknown optional fields;
- 0/1/many media fallbacks;
- price component rendering without false totals;
- recurring/on-request schedule rendering;
- list items preserve commas;
- admin validator errors and draft/preview/publish states;
- exact route canonicalization;
- native and web fixtures consume identical v1 facts.

Required live smoke:

- Massandra and Inkerman compile to v1 without PII;
- admin draft edit does not alter public revision;
- explicit publish changes revision once;
- released app compatibility remains intact during the transition;
- one exact-tour fetch replaces current N+1 public hydration.

## 9. Work explicitly deferred

- full Tourism CRM information-architecture redesign;
- operator self-service and role expansion;
- real-time seat/slot inventory;
- payments/refunds inside WinePool;
- complex seasonal exception calendar;
- multilingual authoring workflow;
- marketplace/discovery expansion;
- full native tourism redesign unrelated to shared contract adoption.

## 10. Exit gate before responsive web UI

Web visual implementation may start when:

1. v1 schema and compiler are covered by SQL tests;
2. both production tours have valid current snapshots;
3. admin can save draft, preview and atomically publish required content;
4. app adapter can read v1 without losing current public facts;
5. hero, price, schedule, carousel, program, practical accordion, form and SEO
   each receive explicit data or an approved honest fallback;
6. exact-tour request is one bounded public projection, not N+1 raw-table reads.

This gate protects the approved UI from foreseeable schema churn without making
H0 depend on an idealized full CRM rebuild.

## 11. Фактическая реализация — 29.08.2026

Exit gate выполнен полностью:

- migration
  `supabase/migrations/202608271200_add_published_tour_contract_v1.sql`
  применена на production после полного backup
  `/root/db_backups/winepool_pre_published_tour_v1_20260827_220410.dump`;
  SHA-256 backup:
  `a32c382b5c49b6085a93f22b77daa176b22c0e6a1b4dcab49651d29e382b6036`;
- добавлены immutable `tourism_experience_publications`, current publication
  pointer, compiler/validator, preview, atomic publish и exact public get RPC;
- public RPC принимает только `(operator_slug, tour_slug)`, не возвращает UUID,
  ответственных сотрудников, notification endpoints, госномер или vehicle
  recognition hints; минимальные grants проверены под ролью `anon`;
- schedule, duration, pricing, payment terms, highlights, media presentation,
  operator identity и SEO нормализованы для Massandra и Inkerman; additive
  migration `202608292100_add_tourism_destination_title.sql` отделяет authored
  hero `destination_title` от города и точек маршрута, а route-derived значение
  оставляет только первоначальным backfill;
- CRM разделяет `Сохранить черновик`, server preview и atomic publish; добавлены
  structured schedule/price, newline-only lists, highlights и media
  caption/alt/order/focal controls;
- `PublishedTourV1` и repository adapter используются exact native/web route;
  released-app raw-table read сохранён на переходный период;
- SQL contract покрывает права, точную canonical пару, immutable snapshot,
  atomic failed publish, explicit republish, сохранение запятых и отсутствие
  внутренних полей; Dart tests покрывают v1 mapping, unknown optional fields,
  media fallbacks и authoring parsers;
- production smoke
  `supabase/tests/20260827_published_tour_contract_production_smoke.sql`
  подтвердил revision `3` обоих пилотов, `Массандра`/`Инкерман`, по четыре
  highlights, по шесть program steps и совместимость legacy anon read.

Изменение черновика после этого шага не влияет на public app/web до явного
`Опубликовать`. Полный Tourism CRM redesign, operator self-service и удаление
legacy SELECT остаются отдельными последующими пакетами.
