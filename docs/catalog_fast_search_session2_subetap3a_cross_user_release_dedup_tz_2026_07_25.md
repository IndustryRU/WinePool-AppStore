# Сессия 2 · Подэтап 3a — межпользовательская дедупликация предложений релиза

Дата фиксации: 25.07.2026
Статус: implemented + migration applied on production (25.07.2026)

## 0. Статус применения — 25.07.2026

- Миграция `20260725_cross_user_release_proposal_dedup.sql` применена на production.
- Backup: `/root/db_backups/winepool_pre_crossuser_dedup_20260725.dump`,
  SHA-256 `0ea894ad3f9e7bbe95425bd0c35f11bee7da1b93d6fb51bd4d1e68c3496dae88`.
- Dry-run `BEGIN…ROLLBACK` чист; apply COMMIT.
- Поведенческий транзакционный тест (откат): user A create → `augmented=false`;
  user B create той же идентичности → `augmented=true`, `contributor_count=2`, та
  же `submission_id`, активных заявок = 1; same-author повтор →
  `release_proposal_already_pending`; повторный `augment` идемпотентен
  (`contributor_count=2`, одна строка погреба, `contributions` не задваивается).
- Внутренние helper-функции лишены execute у public/anon/authenticated
  (`_augment_release_submission` принимает явный p_user_id — недоступен клиенту).
- Клиент: `ManualDraftRepository` возвращает `augmented`/`contributorCount`,
  добавлены `findActiveReleaseProposal`/`augmentWineReleaseProposal`; sheet
  показывает честное сообщение при дополнении; экран модерации показывает
  «Участников: N» и вклад каждого автора.
- static analysis изменённого набора чист.
Родитель: `docs/catalog_fast_search_session_2_tz_2026_07_18.md`, раздел 16.3
Roadmap: `docs/catalog_search_copilot_four_session_execution_plan_2026_07_17.md`

## 1. Цель среза

Когда два и более пользователя предлагают один и тот же релиз (одно вино + один
год/NV), не создавать параллельные заявки. Обнаруживать активный дубль **независимо
от автора** и предлагать пользователю **дополнить существующее** предложение своим
evidence и бутылкой. Сохранять авторство каждого вклада в audit trail, не раскрывая
ПДн пользователей друг другу.

Не входит в 3a (следующие срезы): field-level diff и независимое подтверждение полей
модератором (3b); приоритет очереди по истории вкладов (3c).

## 2. Аудит текущего состояния (проверено по коду 25.07.2026)

- `create_wine_release_proposal` (`20260720_add_user_wine_release_proposals.sql`)
  создаёт `draft_wines` + `draft_wine_sources` + `draft_catalog_submissions`
  (evidence по полям в `snapshot_payload`) + `draft_wine_actions` +
  `draft_catalog_submission_events` + строку `user_storage`
  (`release_proposal_submission_id`).
- Дедуп-guard (строки 104-122) ищет активную заявку **только того же автора**
  (`submitted_by_user_id = auth.uid()`) и бросает `release_proposal_already_pending`.
  Дубля от другого автора он не видит — создаётся параллельная заявка.
- Активные статусы уже перечислены: `submitted`, `in_review`,
  `needs_submitter_input`, `resubmitted`, `approved_needs_submitter_input`,
  `approved_in_review`.
- Есть смежные потоки: уточнение (`202607220200`), галерея фото
  (`202607220300/0400`), сохранение сообщения при повторной отправке
  (`202607220500`), полный review-flow (`20260722_complete_...`). Их
  переиспользуем, отдельный контур не заводим.

## 3. Модель идентичности и дубля

- Идентичность релиза: `wine_id` + (`vintage` или `NV`).
- Активный дубль = заявка `submission_type = 'new_release_for_existing_wine'`,
  `draft_wines.linked_wine_id = wine_id`, совпадающая идентичность, статус из
  активного списка.
- Штрихкод — усиливающий сигнал совпадения, но не обязателен: идентичность
  первична (один и тот же год одного вина = один релиз независимо от кода).

## 4. Поток 3a

### 4.1. Обнаружение (клиент до отправки)

Новый RPC `find_active_release_proposal(p_wine_id, p_vintage, p_is_non_vintage)`:

- возвращает активную заявку по идентичности, если есть: `submission_id`,
  `identity`, `contributor_count`, `is_own` (заявка вызывающего);
- `is_own = true` → клиент показывает «у вас уже есть активная заявка на этот
  релиз» (как сейчас);
- чужая активная заявка → клиент предлагает **«Дополнить существующее
  предложение»** (показать релиз и число участников), без раскрытия личности
  авторов;
- нет активной → обычное создание.

### 4.2. Дополнение (`augment_wine_release_proposal`)

Новый RPC `augment_wine_release_proposal(p_submission_id, bottle/evidence params,
p_idempotency_key)`:

- проверяет, что заявка активна и имеет тип `new_release_for_existing_wine`;
- добавляет `draft_wine_sources` вклад вызывающего (бутылка, покупка, штрихкод) к
  существующему `draft_wine`;
- вставляет строку `user_storage` вызывающего со ссылкой на существующий
  `submission_id` (его бутылка доступна сразу), идемпотентно по ключу действия;
- дополняет `snapshot_payload.contributions` записью вклада с авторством
  (`user_id`, поля, provenance) — агрегированное evidence; клиент и другие
  пользователи авторов не видят;
- один вклад на пару (submission, user): повторное дополнение обновляет свой вклад,
  а не плодит дубли; события аналитики/аудита не задваиваются;
- пишет `draft_wine_actions` + `draft_catalog_submission_events`
  `contribution_added` с id контрибьютора;
- возвращает `{submission_id, storage_id, augmented: true, contributor_count}`.

### 4.3. Серверная защита от гонок

`create_wine_release_proposal`:

- берёт `pg_advisory_xact_lock` по (wine_id, identity);
- повторно проверяет **межпользовательский** активный дубль под локом;
- если дубль появился (чужой) — не создаёт параллельную заявку, а вызывает ту же
  логику дополнения и возвращает `augmented: true` (клиент, пропустивший
  обнаружение, и одновременная отправка обрабатываются корректно);
- поведение same-author не меняется (остаётся `already_pending`).

## 5. Приватность и анти-абьюз

- Личность контрибьюторов не раскрывается ни клиенту, ни другим пользователям;
  наружу отдаются только `contributor_count` и значения полей.
- Провенанс с `user_id` хранится в `contributions`/источниках только для аудита и
  модерации (admin-only).
- Один активный вклад на (submission, user); идемпотентный ключ действия защищает
  от двойного тапа и повторной серверной записи (паттерн подэтапа 2).
- Дополнение не начисляет очки и не влияет на публикацию — только собирает evidence.

## 6. Минимальная видимость для модератора в 3a

- Модерационный экран заявки показывает `contributor_count` и агрегированный список
  evidence по полям (значения-кандидаты + provenance).
- Полноценный field-level diff и независимое подтверждение полей — в 3b, здесь не
  реализуется.

## 7. Тесты и приёмка

- migration/manual: два разных пользователя на одну идентичность → одна заявка с
  двумя вкладами; у обоих бутылка привязана к этой заявке; `contributor_count = 2`.
- same-author повтор → прежний `already_pending`, без дубля.
- гонка: два одновременных create с одной идентичностью → одна заявка, второй
  augmented.
- идемпотентность: повтор `augment` с тем же ключом не задваивает вклад/строку
  погреба/событие.
- одобрение существующей заявки материализует один релиз; evidence всех вкладов
  сохранён в provenance.
- статический анализ и unit проходят; ни один путь не вызывает AI/внешний search.

## 7a. Локализация (pending)

Новые пользовательские строки (честное сообщение о дополнении предложения в sheet,
«Участников: N» и вклад авторов в модерации) вшиты как русский текст по стилю
окружающего не-локализованного кода этих экранов. Отдельным проходом их нужно
вынести в ARB и перевести — см. раздел «Отложенная локализация» в
`docs/l10n_localization_architecture_2026_06_21.md`. На функциональность и gate не
влияет.

## 8. Миграция и деплой

- Миграция additive (новые RPC + правка `create_wine_release_proposal`),
  применяется на production self-host через VPS по runbook памяти проекта
  (backup → dry-run BEGIN/ROLLBACK → apply ON_ERROR_STOP → post-check). Полная
  Flutter/web/APK-сборка в рамках среза не запускается (кроме отдельной проверки
  клиентского sheet при необходимости).
