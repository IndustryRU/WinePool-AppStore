# H0 tourism pilot runbook

Дата: 25.08.2026. Scope: H0.4–H0.6, без развития H1 tourism UI.

## Exact-tour QR

Формат: `https://winepool.ru/tourism/o/{operator}/t/{tour}?ref={code}`.
Код — 2–64 символа, lowercase Latin/digits/`_`/`-`; он заранее создаётся в
`tourism_referral_registry`. Произвольный URL не является dashboard dimension.

Production test registry code: `h0_smoke_20260825`, `is_test=true`. Любая заявка
с этим acquisition snapshot исключается из commercial KPI, но остаётся в QA.

## Smoke checklist

1. HTTP route возвращает Flutter page, сохраняет path и query через age gate.
2. Android/iOS/web открывают точный опубликованный tour, не общий каталог.
3. Guest и authenticated формы создают lead с неизменяемым referral snapshot.
4. Operator notification создаётся; ответственный подтверждает получение.
5. Оператор проводит `new → contacted → confirmed → completed/no_show`.
6. `tourism_lead_events` содержит actor/timestamp outcome.
7. Test lead виден в QA и не входит в commercial KPI.

Database transaction smoke пунктов 3–7 пройден 25.08.2026 с rollback. HTTP
exact-tour smoke пройден: canonical nested route сохраняет `ref` при redirect на
URL с завершающим `/` и возвращает `200` с public WinePool Tourism transition
page и age gate. Public site и Supabase/admin действительно размещены на разных
IP; это штатное разделение, а не blocker.

## Operator SLA

- `new → contacted`: целевой SLA до 60 минут, пилотная цель 15–60 минут;
- `confirmed` ставится только после реального подтверждения следующего шага;
- `completed/no_show` ставит authenticated operator/admin после даты поездки;
- outcome без actor audit запрещён;
- weekly report показывает leads старше 60 минут без `contacted_at`.

## Pilot directory gate

Перед внешней раздачей QR владелец фиксирует 3–5 точек: registry code,
организацию/тур, location/material label, ответственного и проверенный канал
связи. Тестовый code не превращается в commercial автоматически.

### Prepared directory — 30.08.2026

После control backup
`/root/db_backups/winepool_pre_h0_pilot_directory_20260830.dump` (5 754 bytes,
SHA-256 `f8387a4f8f0399a2bf4c9b936bd8c6bc77dc015f031b9a7887a1b9829471ad13`)
в production registry подготовлены пять non-test точек:

| Code | Channel/location | Tour | State |
|---|---|---|---|
| `yalta_operator_site_massandra` | сайт оператора | Массандра | inactive |
| `yalta_operator_social_massandra` | социальный канал оператора | Массандра | inactive |
| `yalta_pickup_qr_massandra` | точка отправления в Ялте, printed QR | Массандра | inactive |
| `massandra_route_qr` | точка маршрута, printed QR | Массандра | inactive |
| `inkerman_route_qr` | точка маршрута, printed QR | Инкерман | inactive |

Все пять связаны с production operator и конкретным published experience, но
намеренно имеют `is_active=false`: подготовка H0 не выдаёт QR и не создаёт
ложный commercial traffic. В H1 каждая точка активируется отдельно только после
проверки фактического размещения, ответственного и рабочего канала связи.

## Weekly cadence

Запускать `supabase/reports/h0_weekly_operating_report.sql`, затем записывать:
store inputs, clean registrations/activation, raw-clean delta, anomalies,
external/QA crashes, top funnel loss, partner funnel, одно product action и
data-quality state. Проценты допустимы только с numerator/denominator; cohort
меньше пяти всегда помечается low sample.
