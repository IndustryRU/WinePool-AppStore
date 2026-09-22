# Аудит готовности: завершение Session 2.5 и запуск Session 2.6

Дата: 04.08.2026
Режим: read-only аудит репозитория и документов. Production-БД не опрашивалась.
Основание: `docs/claude_handoff_sessions_2_5_2_6_3_2026_08_04.md`

## 1. Что фактически сделано в 2.5

Проверено по коду и миграциям, не по тексту документов.

| Этап ТЗ | Статус | Доказательство |
|---|---|---|
| 0. Baseline | готов | `session_2_5_legacy_knowledge_profile_2026_08_03.md`, backup `winepool_pre_knowledge_admin_20260803_120219.dump` |
| 1. Taxonomy foundation | готов (срез A, 04.08.2026) | `202608030100_extend_wine_knowledge_taxonomy.sql` + `202608040100_add_atlas_hierarchy_and_assignment_effect.sql` |
| 2. Manual admin CRUD | готов | `202608030200`, `202608030300`, `202608030400`; `admin_atlas_terms_screen.dart`, `admin_wine_knowledge_editor.dart` (1171 стр.), `admin_wine_knowledge_repository.dart` |
| 3A-0. Legacy mining | готов (dry-run) | `tool/legacy_knowledge_profile/`, `scripts/run_legacy_knowledge_dry_run.ps1`, 2 867 preview proposals, 9 тестов |
| 3A. AI proposal contract | **не начат** | нет `catalog_knowledge_proposals` ни в миграциях, ни в коде |
| 3B. Moderation apply | **не начат** | нет `admin_apply_catalog_knowledge_proposals`, нет `catalog_knowledge_moderation_events` |
| 4. Card/Atlas read path | **не начат** | `WineCardSourceData` не содержит `termAssignments`; `wine_terms` не читается публично (в `wines_repository.dart` есть только join `atlas_terms` внутри `wine_awards`) |
| 5. Rollout/backfill | **не начат** | seed Atlas в production не применялся (в dry-run прочитан 1 term) |

Визуальная часть закрыта: 45 aroma PNG, 22 pairing PNG, production/eco/style SVG,
15 placeholder-SVG под окончательными именами зарегистрированы в
`WineCardIconAsset` (`lib/features/wines/presentation/widgets/wine_card_icon.dart`).

## 2. Подтверждённые пробелы против ТЗ

Пункты 1 и 2 закрыты срезом A 04.08.2026; остальные открыты.

1. ~~`atlas_term_relations` — general → specific DAG.~~ Создан 04.08.2026:
   cycle- и kind-guard, `atlas_term_ancestor_ids`/`atlas_term_descendant_ids`,
   admin RPC list/save/delete, RLS, раздел «Иерархия Atlas» в admin UI.
2. ~~Колонка `wine_terms.effect`.~~ Добавлена 04.08.2026 со значениями
   `include`/`exclude`; `replace` остаётся apply-intent на стороне proposals.
   Exclude разрешён только на релизе и только поверх существующего wine-level
   include; в UI — «Не относится к этому релизу» / «Вернуть в релиз».
3. `catalog_knowledge_proposals` + `catalog_knowledge_moderation_events`.
4. `admin_apply_catalog_knowledge_proposals` и reject/restore RPC.
5. Публичный read path карточки для structured assignments.
6. Production seed таксономии (aroma/pairing/method/eco/style/certification).
7. Исправление alias `Слива` у term `Чёрная слива` (описано в §2.3 seed review,
   на production не выполнено).
8. Автотесты уровня БД и repository для 2.5 — в `test/` нет ни одного
   knowledge/atlas-теста.

## 3. Оценка остатка 2.5

Порядок величины по срезам мандатного workflow:

| Срез | Содержание | Оценка |
|---|---|---|
| A | `atlas_term_relations` + `wine_terms.effect` + hierarchy RPC + тесты | 1 срез |
| B | Seed таксономии (≈75 canonical terms + aliases + icon_key) в dry-run и apply | 1–2 среза |
| C | Повторный dry-run extractor, exact/alias coverage report | 0.5 среза |
| D | `catalog_knowledge_proposals` + materialization v3/legacy/v4 | 1–2 среза |
| E | Apply RPC + safe bulk policy + audit + идемпотентность | 1–2 среза |
| F | Workbench-панель предложений, review sheet, edit/reject/restore | 1–2 среза |
| G | Card read path, release merge, Atlas popover, l10n | 1–2 среза |
| H | Production rollout, backfill, эталонные вина, QA-матрица | 1 срез |

Итого 7–12 рабочих срезов, каждый со своей миграцией/тестами/коммитом.

## 4. Формальный блокер Session 2.6

`session_2_6_release_aware_offers_start_prompt_2026_07_29.md` требует: «Если
Session 2.5 не имеет статуса verified владельцем... не начинай миграцию offers».
Сейчас 2.5 не verified и не завершена. Блокер зафиксирован, обход не предлагается.

## 5. Предварительный аудит 2.6 (по репозиторию)

- `offers` имеет `vintage int` и `bottle_size_id`, но **не имеет**
  `wine_vintage_id` и ссылки на SKU (`lib/features/offers/domain/offer.dart`).
- `offers_repository.dart` читает `select('*')` с join `bottle_sizes` — то есть
  привязка к релизу сегодня строковая/числовая, а не FK.
- Целевой контракт 2.6 (wine → wine_vintage → SKU) потребует additive-миграции и
  backfill с очередью неоднозначных строк.
- Полный аудит фактической схемы `offers`, `wine_product_codes`, sellers и
  ценовых RPC требует живой production-БД по SSH: локальный readonly URL из `.env`
  указывает на устаревшую базу.

## 6. Вопросы к владельцу перед стартом

1. Порядок: сначала seed + карточка (видимый результат), затем proposal/apply —
   или строго по ТЗ (proposals → apply → карточка)?
2. Объём первого seed: весь утверждённый реестр (~75 terms) или только aroma +
   pairing первой волны?
3. Что считать «завершением 2.5» для перехода к 2.6: полный §22 или сокращённый
   набор критериев?
4. Кто применяет production-миграции: Claude по существующей VPS-инструкции или
   владелец вручную?
