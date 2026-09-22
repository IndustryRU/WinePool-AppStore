# Сессия 2 · Подэтап 2 — request-level аналитика и идемпотентность

Дата фиксации: 24.07.2026
Статус: implemented + migration applied on production (24.07.2026)
Родитель: `docs/catalog_fast_search_session_2_tz_2026_07_18.md`, разделы 9 и 16.1
Roadmap: `docs/catalog_search_copilot_four_session_execution_plan_2026_07_17.md`

> H0 privacy correction 25.08.2026: client-side raw `request_id` оказался высококардинальным и содержал barcode/text-bearing fragments. Для дедупликации он остаётся локально, но в AppMetrica заменяется на opaque `attempt_id`. Актуальный контракт: [h0_measurement_activation_partner_attribution_tz_2026_08_25.md](/R:/Flutter/Project/winepool_final/docs/h0_measurement_activation_partner_attribution_tz_2026_08_25.md).

## 1. Цель

Дать каждому поиску стабильный `request_id`, сшивающий события поиска, выбора,
отказа, перехода в draft и добавления в погреб в одну воронку, и гарантировать, что
двойной тап, медленная сеть или повторный callback не создадут дубликаты бутылок и
дублей событий. Не отправлять в аналитику сырой OCR и полный пользовательский текст.

Подэтап не расширяет discovery-UX и не трогает релизный трек (2.5/2.6). Он закрывает
раздел 16.1 ТЗ Сессии 2.

## 2. Аудит текущего состояния (проверено по коду 24.07.2026)

### 2.1. Аналитика

- Канал аналитики уже есть: `Analytics` (Яндекс AppMetrica, `lib/core/analytics/analytics.dart`).
- Воронка add-bottle уже шлётся: `add_bottle_match_result`, `add_bottle_candidate_selected`,
  `add_bottle_candidate_rejected`, `add_bottle_added_to_cellar`, `add_bottle_draft_created`,
  `add_bottle_guest_wall_shown`.
- Сырой OCR и полный текст запроса в события не попадают — приватное правило соблюдено.
- Пробел: ни одно событие не несёт `request_id`. `requestId` генерируется ad-hoc в трёх
  экранах поиска (`text-…`, `barcode-…`, `label-…`), возвращается в `CatalogSearchResult`,
  но не передаётся в `Analytics.*` и не доходит до confirm-sheet. Сшить воронку нельзя.
- Дедупликации повторных событий одного исхода в рамках одного поиска нет.

### 2.2. Идемпотентность

- Клиент confirm-sheet имеет guard `_isSaving`: блокирует кнопку в полёте
  (`candidate_confirm_sheet.dart`). Но state виджета сбрасывается после ошибки, и это не
  защищает от повторной серверной записи при ретрае удачно прошедшего запроса.
- Сервер `add_to_user_storage` использует семантический merge: находит совпадающую строку
  и `quantity += p_quantity`. Это НЕ идемпотентно: честный повтор того же запроса
  увеличит количество второй раз.
- Предложение релиза уже имеет серверный guard (активная заявка того же автора на тот же
  wine+год/NV блокируется) и клиентский `_isSubmitting` — частичная защита.
- Готовый паттерн в репозитории: `award_user_experience` / `user_experience_events` —
  unique index + `on conflict do nothing`. Используем этот же подход.

## 3. Контракт request_id / action_id

- `request_id` — стабильный идентификатор одного поиска. Источник — `CatalogSearchResult.requestId`
  (эхо входного id). Экран читает его из результата и передаёт во все downstream-события
  этого поиска: match_result, candidate_selected, candidate_rejected, added_to_cellar,
  draft_created. Регенерируется только при новом поиске.
- `action_id` — идентификатор одного мутирующего действия (ключ идемпотентности).
  Генерируется один раз на экземпляр confirm-sheet (одно намерение добавить). Повторный
  тап/ретрай в том же sheet использует тот же `action_id`.
- В аналитику не передаются: сырой OCR, полный текст запроса, фотографии.

## 4. Серверная идемпотентность добавления в погреб

Отдельный ledger дедупликации вместо перезаписи ключа на агрегированной строке:

```text
user_storage_add_actions(user_id, idempotency_key, storage_id, created_at)
  unique(user_id, idempotency_key)
  RLS enabled, без клиентских политик (доступ только через SECURITY DEFINER RPC)
```

`add_to_user_storage` получает новый необязательный `p_idempotency_key text default null`:

- при `p_idempotency_key is not null`:
  - `pg_advisory_xact_lock` по `(user_id, key)` — сериализация одинаковых ключей;
  - если в ledger уже есть строка с этим ключом → вернуть сохранённый `storage_id` без
    инкремента (idempotent replay);
- иначе выполнить существующий merge/insert без изменений (обратная совместимость для
  чеков и ручных путей — там merge желателен);
- после записи бутылки при непустом ключе записать `(user_id, key, storage_id)` в ledger
  `on conflict do nothing`.

Additive-миграция: старые вызовы без ключа работают как раньше. Все прочие callers
`add_to_user_storage` не меняются (ключ nullable, дефолт null).

## 5. Клиентская реализация

- `Analytics.*` add-bottle методам добавить необязательный `String? requestId`; писать его
  в params только при наличии. Добавить дедуп once-per-`request_id` для событий,
  которые должны быть не более одного раза на поиск (match_result, added_to_cellar,
  draft_created, candidate_rejected); candidate_selected дедуп по `request_id+wine+rank`.
- Экраны поиска (`name`, `label`, `barcode`) читают `result.requestId` и передают его в
  события и в `showAddBottleConfirmSheet(..., requestId: …)`.
- Confirm-sheet принимает `requestId`, генерирует `action_id` один раз, передаёт
  `idempotencyKey` в `CellarController.addToStorage` → `CellarRepository.addToUserStorage`
  → RPC, и `requestId` в `Analytics.addBottleAddedToCellar`.
- Кнопки add/propose остаются защищёнными клиентскими guard-флагами.

## 6. Область и не-область

Входит: сквозной `request_id` в аналитике, серверная идемпотентность добавления в
погреб, клиентский дедуп повторных событий, тесты повтора.

Не входит: полный `action_id` для propose/draft (используются существующие guard'ы —
author-level для propose, client-флаги), серверный дашборд (готовим только запрос долей
исходов), релизные пункты 2.5/2.6.

## 7. Тесты и приёмка

- unit: `barcodeOutcome`/`rankedOutcome` без регрессий; дедуп аналитики (повтор события
  одного `request_id` не отправляется дважды).
- migration/manual: двойной вызов `add_to_user_storage` с одним ключом создаёт одну
  бутылку/один инкремент; разные ключи и вызовы без ключа работают как раньше.
- статический анализ затронутого набора без ошибок.
- ручной QA: двойной тап «Добавить в погреб» → одна бутылка; ретрай после ошибки не
  создаёт дубля.

## 8. Миграция и деплой

- Миграция additive, применяется на production self-host Supabase через VPS по runbook
  памяти проекта (backup → транзакционный dry-run BEGIN/ROLLBACK → apply с ON_ERROR_STOP,
  post-check). Полная Flutter/web/APK-сборка не запускается в рамках подэтапа.

## 9. Статус применения — 24.07.2026

- Миграция `20260724_add_cellar_add_idempotency.sql` применена на production
  self-host Supabase через VPS.
- Backup: `/root/db_backups/winepool_pre_cellar_idempotency_20260724.dump` (3.0 MB),
  SHA-256 `7d88f4427c7d4b2a0efa9c932a263b508b13d0effbb87b5416d0284dd3d487f1`.
- Pre-check: ровно одна перегрузка `add_to_user_storage` (14 арг), устаревших нет;
  ledger-таблицы не было.
- Dry-run `BEGIN…ROLLBACK` с `ON_ERROR_STOP=1` прошёл без ошибок.
- Post-check: `user_storage_add_actions` создана, RLS включён, PK + два FK на месте,
  функция получила 15-й аргумент `p_idempotency_key text`.
- Поведенческий транзакционный тест (с откатом): два вызова с одним
  `p_idempotency_key` вернули один и тот же `storage_id`, в ledger одна строка —
  повторная запись/двойной инкремент не происходят.
- Клиентская часть (request_id в аналитике, action_id, дедуп) и фикс навигации
  «Редактировать вино» проверяются ручным QA на устройстве; static analysis чист,
  unit-тесты: policy 7/7 + analytics dedup 4/4.
