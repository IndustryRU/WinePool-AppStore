# Сессия 2 · Подэтап 3c — приоритет очереди модерации по репутации вкладчика

Дата фиксации: 25.07.2026
Статус: implemented + migration applied on production (25.07.2026)

## 0. Статус применения — 25.07.2026

- Миграция `20260725_submitter_contribution_reputation.sql` применена на production.
- Backup: `/root/db_backups/winepool_pre_reputation_20260725.dump`,
  SHA-256 `ace3520d8cd4a59818c9a292b36855dd432b37d966cddecfc95e33bab91046cf`.
- Dry-run `BEGIN…ROLLBACK` чист; apply COMMIT.
- Проверка: под catalog-admin RPC вернул строки (напр. 0 approved / 1 rejected →
  `0.3333`; без истории → `0.5000`); под non-admin — `reputation_admin_required`.
- Клиент: `adminDraftCatalogSubmissionQueueProvider` дозапрашивает репутацию и
  переупорядочивает очередь по `reputation desc` со стабильным вторичным
  `submitted_at desc`; статусы/данные не меняются.
- static analysis чист. Клиентская часть едет с деплоем admin web.
Родитель: `docs/catalog_fast_search_session_2_tz_2026_07_18.md`, раздел 16.3
Предшественники: 3a (дедуп), 3b (field-level diff)

## 1. Цель

Учитывать историю подтверждённых/отклонённых вкладов автора **только для
сортировки очереди модерации**, чтобы заявки от проверенных вкладчиков всплывали
выше. Никакой автоматической публикации: очередь только переупорядочивается,
модератор по-прежнему решает по каждой заявке.

## 2. Метрика репутации

Для каждого пользователя:

- `approved_count` — число финализированных заявок со статусом `approved`, где он
  автор **или** контрибьютор (через `user_storage.release_proposal_submission_id`,
  т.е. с учётом augment-вкладов 3a);
- `rejected_count` — аналогично для `rejected`;
- `reputation = (approved + 1) / (approved + rejected + 2)` — сглаживание Лапласа:
  без истории `0.5` (нейтрально), стабильно одобряемый → к 1, отклоняемый → к 0.

## 3. Backend

RPC `get_submitter_contribution_reputation(p_user_ids uuid[])` →
`(user_id, approved_count, rejected_count, reputation)`:

- read-only, `security definer`, доступ только catalog-admin
  (`is_catalog_admin(auth.uid())`);
- считает по union (submitter ∪ contributor) финализированных заявок;
- возвращает строку на каждый запрошенный id (нейтральный дефолт при отсутствии
  истории).

## 4. Клиент

`adminDraftCatalogSubmissionQueueProvider`:

- после сбора `submitterIds` дозапросить репутацию батчем;
- переупорядочить очередь по `reputation desc`, стабильно сохраняя вторичный порядок
  `submitted_at desc` (декорируем исходным индексом, т.к. `List.sort` в Dart не
  гарантирует стабильность);
- никаких изменений статусов и записей — только порядок.

## 5. Не входит

- Автопубликация или авто-одобрение — запрещено gate.
- Видимый бейдж репутации в UI (опционально позже; 3c — только порядок).
- Локализация вшитых строк — общий отложенный проход.

## 6. Приёмка

- очередь у catalog-admin переупорядочена: авторы с более высокой долей одобрений
  выше; новые/без истории — нейтрально между «хорошими» и «плохими»;
- вторичный порядок по свежести сохраняется при равной репутации;
- никакие статусы/данные не меняются; RPC доступен только catalog-admin;
- static analysis чист; путь не вызывает AI/внешний search.

## 7. Миграция и деплой

- Миграция additive (один read-only RPC), применяется на production через VPS по
  runbook (backup → dry-run → apply → verify). Клиентская пересортировка — на
  admin web; деплой отдельно по команде владельца.
