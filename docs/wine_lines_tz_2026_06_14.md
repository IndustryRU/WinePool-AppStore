# ТЗ: Линейки вин (wine lines) — Этап 1

Дата: 14.06.2026
Статус: на согласование перед реализацией.

## Цель

Ввести структурный слой **линейка** (range/line) — именованная серия вин внутри
одной винодельни (Agora → «Сокровища Крыма», Захарин → «Бухта Омега» /
«Авторские вина»). Линейка — это **факт о вине**, как винодельня/регион, а не
маркетинговая подборка (подборки = «коллекции», Этап 2, отдельный слой).

### Зачем

1. Убрать антипаттерн «фейковая винодельня на каждую линейку» (сейчас «Сокровища
   Крыма» заведена и как отдельная винодельня, и как часть Agora — каталог
   разрезан). Линейка даёт правильное место для такой группировки.
2. Карточка вина: винодельня → линейка (если есть).
3. Каталог: зависимый фильтр по линейке.
4. OCR этикетки: крупное название линейки («Сокровища Крыма») резолвится в
   «винодельня + линейка», а не плодит винодельню.

Не входит в Этап 1: геймификация, «собери набор», ачивки, произвольные
многие-ко-многим подборки — это Этап 2 (`docs/wine_collections_concept_2026_06_14.md`).

## Принцип

- Одно вино принадлежит **максимум одной линейке** (nullable).
- Линейка принадлежит **ровно одной винодельне** (`winery_id`).
- Линейка модерируется как канонический каталог (catalog admin), governance — как
  у `winery_aliases` (см. `20260511_add_catalog_normalization_aliases_and_decisions.sql`).
- Зеркалим проверенный alias-паттерн виноделен, чтобы не изобретать заново.

## Модель данных

### Таблица `public.wine_lines`

```
id              uuid pk default gen_random_uuid()
winery_id       uuid not null references public.wineries(id) on delete cascade
name            text not null
normalized_name text not null              -- public.normalize_catalog_alias_name(name)
description     text
is_deleted      boolean not null default false
created_at      timestamptz not null default now()
updated_at      timestamptz not null default now()

unique (winery_id, normalized_name)
index (winery_id)
index (normalized_name)
trigger update_updated_at_column
RLS: select для всех; write — catalog admin (как winery_aliases)
```

### `public.wines`

```
add column line_id uuid null references public.wine_lines(id) on delete set null
index (line_id)
```

### Таблица `public.wine_line_aliases` (для OCR-резолва; опционально, но рекомендуется)

Зеркало `winery_aliases`:
```
id, line_id uuid not null references public.wine_lines(id) on delete cascade,
alias_name text, normalized_alias_name text, alias_type text, is_primary bool,
created_at, updated_at
unique (line_id, normalized_alias_name); index (normalized_alias_name)
```
Backfill: alias из `wine_lines.name` (canonical/primary).

### Domain (Flutter)

- `WineLine { id, wineryId, name, description }` (новый файл
  `lib/features/wines/domain/wine_line.dart`).
- `Wine`: добавить `lineId` (+ опционально вложенный `WineLine? line` через join,
  по аналогии с `winery`). `Wine` — freezed → прогнать build_runner.

## Репозиторий / контроллеры

`lib/features/wines/data/wines_repository.dart` (или новый
`wine_lines_repository.dart`):
- `fetchLinesByWinery(wineryId)` → `List<WineLine>`.
- `fetchAllLines({wineryIds})` → для каталожного фильтра без выбранной винодельни.
- `createLine / renameLine / deleteLine` (через RPC под catalog admin) — admin CRUD.
- `assignWineLine(wineId, lineId?)` — выставить/снять линейку у вина.
- `searchWinesByLine(lineId)` — для перехода из карточки/линка.

Провайдеры (manual `FutureProvider.family`, как `fetchWineryByIdProvider`):
`fetchLinesByWineryProvider`, `fetchAllLinesProvider`, `fetchWineLineByIdProvider`.

## Карточка вина

`lib/features/wines/presentation/wine_details_screen.dart`:
- под винодельней показать линейку (если `line != null`): «Винодельня → Линейка».
- тап по линейке → каталог, отфильтрованный по этой линейке (deep-link на фильтр).

`winery_details_screen.dart` (опц.): группировать список вин винодельни по линейкам
(заголовки-секции), вне линейки — секция «Прочие».

## Каталог: зависимый фильтр по линейке

Зеркалим существующий фильтр винодельни:
- `lib/features/catalog/application/catalog_filters_provider.dart` (freezed) —
  добавить `List<String> lineIds` (прогнать build_runner на `.freezed/.g`).
- UI: `line_filter_widget.dart` + `line_selection_screen.dart` (копия
  `winery_filter_widget.dart` / `winery_selection_screen.dart`), подключить в
  `catalog_filters_panel.dart` / `advanced_filters_sheet.dart`.
- **Зависимое поведение (как просил пользователь):**
  - выбрана винодельня (одна/несколько) → список линеек = только этих виноделен;
  - винодельня не выбрана → список всех линеек, выбираем из полного списка.
- RPC выборки вин (`fetchWinesWithFilters` / `get_wines_with_*`) — добавить
  фильтр `wines.line_id in (lineIds)`.
- RPC доступных линеек `get_available_wine_lines(p_winery_ids uuid[] default null)`
  по образцу `20260109015252_create_get_available_wineries_function.sql` (с учётом
  только активных, не удалённых, видимых вин).

## OCR этикетки (интеграция с Этапом 2 разбора этикетки от 14.06.2026)

- Лексикон линеек: `Map<normalizedLineName, (wineryId, lineId, wineryName, lineName)>`
  из `wine_lines` + `wine_line_aliases` (метод в `ReceiptController`, рядом с
  `_getReceiptWineryNames()` / `buildLabelHints`).
- `wine_label_field_extractor` (`extractLabelNameAndVintage`) расширить параметром
  `knownLines`: если в тексте этикетки крупно встречается имя линейки —
  выставляем `winery = line.winery`, `line = line.name`, а название вина берём из
  остального (не из имени линейки). Для «Сокровища Крыма»: winery=Agora,
  line=«Сокровища Крыма», name=конкретное кюве (Бастардо/Мускат…).
- `LabelDraftPrefill`: добавить `lineId/lineName`, прокинуть в ручной черновик
  (поле «Линейка», зависящее от винодельни) и в matching как доп. сигнал
  (бонус к скорингу при совпадении линейки в той же винодельне).

## Админ / модерация

- В `add_edit_wine_screen.dart` — пикер линейки, **scoped по винодельне вина**
  (если винодельня не выбрана — пикер заблокирован). Возможность создать новую
  линейку на лету (как create-winery), с guard на дубль по `normalized_name` в
  пределах винодельни.
- Управление линейками винодельни (CRUD + merge двух линеек) — в
  `winery_details_screen` (admin-секция), по образцу управления winery alias
  (`20260403_add_admin_winery_alias_management_rpcs.sql`).
- Governance/каталожные изменения — как у остального канонического каталога.

## Взаимодействие с объединением виноделен

При `admin_merge_wineries(S → T)` (уже есть):
- сейчас вина S переезжают в T;
- **расширение (опционально, отдельным шагом):** предложить «вина S сложить в
  линейку T с именем S» — ровно кейс «Сокровища Крыма как линейка Agora».
  Реализовать как опциональный параметр RPC `p_move_into_line_name text` или
  отдельное admin-действие после merge. В Этап 1 можно не включать в RPC, а
  оставить ручное назначение линейки после merge.

## Миграция и применение

- Файл `supabase/migrations/2026XXXX_add_wine_lines.sql`: `wine_lines`,
  `wines.line_id`, `wine_line_aliases`, индексы, RLS, триггеры, backfill alias из
  name. Идемпотентно (`if not exists`).
- RPC: `admin_create_wine_line`, `admin_rename_wine_line`, `admin_delete_wine_line`,
  `admin_merge_wine_lines`, `admin_assign_wine_line` (security definer,
  `is_catalog_admin`); `get_available_wine_lines`.
- Применение на live по протоколу: backup → review SQL → apply под `-U postgres`
  (карта покупок не затрагивается → НЕ supabase_admin). См.
  `docs/powershell_ssh_sql_quoting_guide.md`.

## Тесты

- Unit: резолв линейки в `wine_label_field_extractor` (новый кейс «Сокровища
  Крыма» → winery=Agora + line), не ломая текущие 10 тестов.
- Unit: зависимый список линеек (winery выбрана/не выбрана).
- Регресс: `flutter analyze` + существующие receipt/label/merge тесты зелёные.

## Acceptance criteria

- У вина можно задать линейку (scoped по винодельне); в карточке видно
  «винодельня → линейка».
- В каталоге фильтр по линейке зависит от выбранной винодельни, как описано.
- OCR крупной линейки даёт winery+line, а не новую винодельню.
- Линейка модерируется (CRUD/merge) под catalog admin; дубли по normalized_name в
  пределах винодельни не плодятся.
- `flutter analyze` чист, тесты зелёные, миграция применяется по протоколу.

## Порядок реализации

1. Миграция (`wine_lines` + `wines.line_id` + `wine_line_aliases`) + RPC.
2. Domain + repository + провайдеры (+ build_runner).
3. Карточка вина (винодельня → линейка).
4. Каталог: `lineIds` в фильтрах + зависимый UI + RPC выборки/доступных линеек.
5. OCR: лексикон линеек + расширение экстрактора + поле в черновике.
6. Админ: пикер линейки в add/edit wine + CRUD/merge линеек.
7. analyze + тесты + применение миграции на live (backup→review→apply).

## Бэклог / будущие доработки (не Этап 1)

Согласовано 15.06.2026: текущая реализация линеек — **текстовая**. Ниже отложенные,
но зафиксированные направления.

### 1. Визуальное выделение линейки (отложено)

Дать линейке опциональную визуальную тему, чтобы пользователь видел отличие
(пример: премиальная серия с собственной графикой/яхтой).

- Дёшево добавить позже: опциональные поля у `wine_lines` — `accent_color`,
  `icon` (ключ иконки), `image_url`/`badge_url`; чип линейки в карточке и в
  каталоге рисовать с этим акцентом.
- По смыслу это **презентация/маркетинг** — органично ложится в слой коллекций
  (Этап 2) или отдельный «витринный» слайс. Требует кураторской работы и контроля
  единообразия дизайна, поэтому в Этап 1 не входит.

### 2. Уровень качества / тир (Reserve / Grand Reserve) — ОТДЕЛЬНЫЙ атрибут

Важно: **линейка ≠ уровень качества.**

- **Линейка** — брендовая серия производителя (напр. «Сокровища Крыма»,
  «Бухта Омега»). Это и есть Этап 1.
- **Тир качества** — `Reserve` / `Grand Reserve` / `Reserva` / `Gran Reserva` /
  «Гранд Резерв» и т.п. Это **ортогональный** атрибут конкретного вина: вино может
  одновременно быть в линейке И иметь тир.

Моделировать тир как отдельное поле/справочник у `wines` (например, enum/lookup
`quality_tier`), НЕ как линейку. Возможные применения: бейдж в карточке, фильтр в
каталоге, сигнал в матчинге/OCR. Отдельный небольшой слайс после Этапа 1.

### 3. Линейка ≠ хозяйство-производитель — 13.09.2026

Бренд может собирать вино у разных хозяйств (Tussock Jumper: Tempranillo из
Испании, Chardonnay от Orchidées, Maisons de Vin из Франции). Линейка остаётся
серией **винодельни вина**, то есть бренда. Хозяйство, где вино сделали, —
отдельная необязательная ссылка у вина, а не вторая линейка и не перенос вина
к хозяйству. Правила — `docs/wine_production_winery_tz_2026_09_13.md`.

При переносе вина в другую винодельню линейка прежней винодельни снимается,
если не выбрана линейка новой.

## Связанные документы

- `docs/winery_alias_layer_spec.md`
- `docs/wine_merge_admin_implementation_2026_06_14.md`
- `docs/label_field_extraction_2026_06_14.md`
- `docs/wine_collections_concept_2026_06_14.md` (Этап 2)
- `docs/wine_production_winery_tz_2026_09_13.md` (бренд и хозяйство-производитель)
