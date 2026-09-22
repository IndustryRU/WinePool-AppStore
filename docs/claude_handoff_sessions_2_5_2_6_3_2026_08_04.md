# Handoff Claude: завершение 2.5, затем 2.6 и подготовка 3

Работай в `R:\Flutter\Project\winepool_final`. Начни с `git status` и чтения:

- `docs/session_2_5_wine_knowledge_moderation_atlas_tz_2026_08_01.md`;
- `docs/session_2_5_atlas_seed_review_v1_2026_08_03.md`;
- `docs/session_2_5_release_experience_and_wine_card_tz_2026_07_29.md`;
- всех документов `docs/session_2_6_*`.

Последняя завершённая точка: pairing wave — 22 production PNG, resolver,
локализации и тесты. Не переделывай утверждённую карточку и иконки без причины.

## Временные visual slots

Все перечисленные файлы уже существуют, добавлены в `WineCardIconAsset` и
намеренно содержат одну одинаковую SVG-заглушку:

`production_traditional_method.svg`, `production_charmat_method.svg`,
`production_lees_aging.svg`, `production_stainless_steel.svg`,
`production_bottle_fermentation.svg`, `production_maceration.svg`,
`production_spontaneous_fermentation.svg`, `production_flor_aging.svg`,
`style_en_rama.svg`, `style_col_fondo.svg`, `feature_unfiltered.svg`,
`feature_unfined.svg`, `feature_low_so2.svg`, `certification_fallback.svg`,
`atlas_term_fallback.svg`.

Контракт: используй эти окончательные имена в UI/resolver/Atlas. Не переименовывай
и не рисуй новые варианты. Позже Codex заменит только SVG-содержимое. Маркер
`atlas_visual_placeholder.svg` является исходной технической заглушкой.

## Сначала завершить session 2.5

1. Довести Atlas term/assignment/moderation flow по утверждённому ТЗ: уровни
   wine/release, наследование без ложного удаления общих данных, source,
   confidence только там, где он нужен, удаление assignment отдельно от term.
2. Подключить canonical methods/styles/features/certifications к карточке через
   перечисленные `WineCardIconAsset`; неизвестный опубликованный term использует
   `atlasTermFallback`, certification без разрешённого логотипа —
   `certificationFallback`.
3. Сохранить AI-first архитектуру: proposals → evidence → moderator decision →
   assignment; не записывать AI-вывод напрямую как подтверждённый факт.
4. Завершить безопасный backfill legacy descriptions: dry-run, aliases,
   ambiguity queue, idempotency, audit и rollback. Не применять неоднозначную
   «сливу» автоматически.
5. Проверить admin UX и public wine card на base/release inheritance, empty,
   loading, error, narrow screen и отсутствие overflow.
6. Применять миграции только по существующей VPS-инструкции; перед каждой —
   backup/rollback plan и проверка диска. Не тянуть лишние Docker images.

## Затем session 2.6

Продолжай только после зелёной матрицы 2.5. Следуй `session_2_6_*`: release-aware
offers, receipt observations и partner offers — разные важные источники;
агрегируй одинаковые торговые точки, сохраняй latest price и release/SKU scope,
не ломай legacy fallback и старые APK. Выполни миграции, RPC/API, admin/mobile UI,
RLS, индексы, тесты и документацию одним согласованным контрактом.

## Session 2.5 принята 07.08.2026

Приёмка — `session_2_5_acceptance_2026_08_07.md`: 14 критериев §22 закрыты,
2 исключения (AI v4 отложен владельцем, апелласьоны невозможны без географии),
1 известный дефект перенесён в Session Atlas (награды читаются публично без
фильтра статуса, на живых данных не проявляется).

В каталоге 3919 подтверждённых фактов на 130 терминах Atlas. Рабочей очереди
предложений нет: осталось 106 географии, 15 заблокированных наград и 10
кандидатов в термины.

**Можно начинать Session 2.6.**

## Очерёдность сессий, утверждена 06.08.2026

```
Session 2.5 → Session 2.6 → релиз 1.1.0 → Session Atlas → Session 3
```

**Session Atlas** вставлена между релизом и третьей сессией:
`session_atlas_publishing_tz_2026_08_06.md`. Она строит публикационный контур —
статьи Atlas в БД, секции, редактор в админке, справочник конкурсов и публичные
маршруты. Основание — `atlas_knowledge_base_consolidated_analysis_2026_08_06.md`.

Причина, по которой она не раньше: 2.6 стоит на критическом пути к релизу, а
Atlas на нём не стоит. Причина, по которой она не позже: 130 заведённых
терминов ведут в никуда, награды нельзя применять без справочника конкурсов, а
QR офлайн-точки туризма требует публичной веб-страницы.

## Отложено по решению владельца — не потерять

**Фильтры каталога по технологии производства** (решение 06.08.2026, делать не
сейчас). Pet-Nat переехал в карточке из эко-признаков к технологиям, но в
каталоге остался чипом среди эко-фильтров. Нужно завести группу фильтров по
технологии, перенести туда Pet-Nat и решить, читают ли новые фильтры
`wine_terms` напрямую вместо булевых колонок. Подробности и обоснование —
`session_2_5_wine_card_structured_production_2026_08_06.md` §4.

**География Atlas**: 93 предложения регионов и апелласьонов ждут решения о
сопоставлении с существующим справочником — см.
`session_2_5_review_queue_guide_2026_08_05.md` §4.

**Десять кандидатов в термины** без иконок и без решения — там же, §2.

## Session 3

Не начинай широкую реализацию без отдельного ТЗ. Разрешено подготовить аудит,
backlog, зависимости и критерии готовности на основе фактического состояния
после 2.6. Не смешивай незавершённые миграции 2.6 с новой схемой session 3.

## Обязательный порядок работы

Малые срезы: read-only audit → план → код/миграция → targeted tests → ручная
матрица → документация → отдельный commit. Сохраняй пользовательские изменения,
не делай destructive git operations. В конце каждого среза сообщай: что готово,
что проверено, миграции/деплой, commit, риски и следующий шаг.

Минимальные проверки: `dart format`, `flutter analyze` затронутых файлов,
targeted Flutter tests, backend/RLS tests для изменённой схемы, `git diff --check`.
