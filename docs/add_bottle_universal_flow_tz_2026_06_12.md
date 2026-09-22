# ТЗ: Универсальный Flow «Добавить Бутылку» — Погребок Без Чека, Manual Drafts И Мгновенная Видимость Вклада

Дата: 12.06.2026
Проект: WinePool
Статус: в реализации. Проход 0 (backend foundation) выполнен 12.06.2026: миграции A/B/C написаны (`20260612_add_manual_draft_sources.sql`, `20260612_add_manual_draft_storage_autoadd.sql`, `20260612_add_create_manual_draft_wine_rpc.sql`, ожидают review -> backup -> dry-run -> apply на live), Dart-слой `lib/features/add_bottle/` (domain + manual_draft_repository) и фасад `wine_text_hints.dart` созданы, flutter analyze чистый. Решение по пункту 6 прохода 0: вместо физического переноса хелперов из receipt_controller сделан export-фасад (ноль риска регрессии); физический перенос — отложенный рефакторинг.

СТАТУС ПРОХОДОВ:
- Проход 0 (backend foundation) — ВЫПОЛНЕН 12.06.2026. Миграции A/B/C применены на live (backup: /root/db_backups/winepool_pre_addbottle_20260612.dump), верифицированы. Коммиты: 3a2e17a (docs), c22b734 (backend).
- Проход 1 (skeleton + путь «По названию» + погребок) — ВЫПОЛНЕН 12.06.2026. Route /add-bottle (path=name -> поиск, иначе хаб); экраны hub/name-search/confirm-sheet/manual-draft; группа «На проверке» в погребке вместо _DraftsBanner + FAB «Добавить вино» + новый empty state; CTA в пустом поиске приложения; 53 l10n-ключа ru/en. flutter analyze чистый. Ручной e2e на устройстве — ожидает QA.
- Решения по ходу прохода 1: readiness в manual draft = 5 признаков (фото станет шестым в проходе 3); чипсы хранят color/type enum.name, sugar toDbValue(); похожие вина в manual draft через searchWines top-3 (полный suggestion engine — позже); XP за качество manual-полей не начисляется (follow-up); confirm-leave и guest wall — проход 5; rejected-драфты в группе «На проверке» не показываются (механика dismissal — проход 5).
- Полировка прохода 1 по фидбеку пользователя 12.06.2026: CTA «Добавить вино, которого нет» показывается ВСЕГДА при активном запросе (футер под результатами), а не только в пустой выдаче — в глобальном поиске И в каталоге (+ кнопка в пустой выдаче каталога); погребок: статистика сверху, группа «На проверке» после неё свёрнутым блоком (2 позиции + «Ещё N»/«Свернуть»), карточки черновиков с фото (photo_url добавлен в DraftWineSummary/провайдер) и полным названием в 2 строки.
- Проход 2 (путь «Штрихкод») — ВЫПОЛНЕН 12.06.2026. Экран сканера на mobile_scanner (EAN-13/8, UPC-A/E; QR детектится только для подсказки «это QR чека» с переходом в чековый сканер), permission UX по паттерну receipt scanner (denied/permanently denied/unavailable/init failed), exact match через существующий winesRepository.searchWineByBarcode (limit 1; кейс дублей barcode в каталоге схлопывается до первого — реальный фикс это catalog dedupe), found -> confirm sheet, not found -> sheet «станете первооткрывателем» -> manual draft c prefillBarcode и entry_method=barcode, fallback-кнопки «Ввести вручную» и «Найти по названию», карточка пути в хабе. Ручной QA на устройстве — ожидает.
- Контекст-факт (проверен 12.06.2026 по live DB и коду): российские чеки НЕ приносят EAN позиций (110 RU-позиций, 0 с barcode; вино в ЕГАИС, не в «Честном знаке»), белорусские приносят gtin_code (by_receipt_response_parser). Сканирование бутылки — единственный способ задействовать barcode-покрытие каталога для RU.
- Discoverability-доработка 12.06.2026 (по фидбеку: «как пользователь поймёт, что можно сканировать штрихкод?»): кнопка «Добавить вино» на главной под «Сканировать чек» (пункт прохода 5 вытянут раньше); чековый QR-сканер теперь детектит EAN/UPC и показывает подсказку «Это штрихкод бутылки, а не QR чека» со SnackBar-действием перехода в /add-bottle?path=barcode (обработка чека осталась строго за QR-форматом, cooldown подсказки 6 сек).
- Проход 3 (путь «Этикетка» + новый матчинг) — ВЫПОЛНЕН 13.06.2026.
  - Новый матчинг-конвейер: `WinesRepository.searchWinesByAlias` (поиск по `wine_aliases.normalized_alias_name`, 1168 строк на проде — устойчивее ilike по name); `BottleLabelMatcher` (application/bottle_match_controller.dart) — нормализация receipt-matcher подходом, извлечение полей через WineLabelTextProcessor + parseReceiptWineHints, параллельный поиск alias/name/winery, explainable ранжирование по token coverage + alias/winery бонусы, пороги high>=0.72 / medium>=0.4 / low; domain bottle_candidate.dart (BottleMatchSignal, BottleCandidate, LabelMatchResult, LabelDraftPrefill).
  - Экран add_bottle_label_scan_screen.dart: камера/галерея, OCR через ocrServiceProvider (Yandex для кириллицы, ML Kit latin + предупреждение про кириллицу), стадии idle/recognizing/results/error; high -> авто confirm sheet; кандидаты с сигналом совпадения; «Это не оно» -> manual draft.
  - ManualDraftScreen расширен: rich prefill (winery/country/region/vintage/color/sugar/type) + фото этикетки (блок с миниатюрой, камера/галерея, +5 XP подсказка); фото грузится в draft_wine_photos (`<uid>/<draftId>`) после создания черновика и до submit, через saveDraftWinePhoto; readiness теперь 6 признаков (добавлено фото).
  - Старый прототип удалён: wine_label_ocr_screen.dart + wine_label_search_controller.dart + их тест; /wine-label-ocr -> redirect на /add-bottle?path=label. Карточка «Этикетка» в хабе (вторая после штрихкода).
  - flutter analyze по затронутым чистый. Ручной QA на устройстве (10 реальных бутылок: кириллица/латиница, из каталога и нет) — ожидает.
  - Решения/долги: авто-alias по подтверждению («Это оно») не пишется — P1 «память этикеток»; barcode-in-image как сигнал пока не выделяется отдельно (ML Kit barcode из того же кадра — follow-up); confirm-leave/guest wall/аналитика — проход 5.
- Проход 4 (модерация manual-заявок) — ВЫПОЛНЕН 13.06.2026.
  - DraftWineSourceEntry: добавлен `sourceKind` (+ `isManual`), `unitPrice` стал nullable (по миграции A); manual shop-гео (`shop_lat`/`shop_lng`) маппится в lat/lng при загрузке; оба source-select'а грузят `source_kind` + manual shop-колонки.
  - Snapshot: каждый источник несёт `source_kind`; `source_summary.primary_source_kind` (manual, если все источники ручные) для очереди.
  - Публичный `user_prices` backfill теперь берёт только receipt-источники (`source_kind='receipt'`) — ручная цена без чека не влияет на доверенную цену каталога.
  - Очередь модерации: бейдж «Чек»/«Вручную» (manual оранжевый), счётчик чеков скрыт для manual.
  - Details/workbench: заголовок и интро evidence-секции переключаются на «Добавлено пользователем вручную» для manual; цена скрывается, если не указана.
  - Nullable price починен на всех display-сайтах (draft details, admin details, workbench).
  - Триггер автодобавления (миграция B) проверен как `fn_has_manual_branch=t` при применении; реальный approve→user_storage e2e — через админку на живой manual-заявке (write-тест на проде намеренно не запускался автономно).
  - flutter analyze чистый по затронутым; 57 тестов wines/application проходят.
  - Долг: entry_method (barcode/label/name под-тип) в snapshot/UI пока не выделяется — для бейджа достаточно source_kind; уведомления используют общий контур (тексты не привязаны к чеку) — отдельная проверка письма на manual в проходе 5/QA.
- Доработки 13.06.2026 по фидбеку (перед проходом 5):
  - Задача 1 (коммит 3715aac): label-результаты переписаны на проверенный движок ранжирования. Новый публичный `ReceiptController.buildLabelCatalogSuggestions(ocrText)` переиспользует lexicon/alias-индекс виноделен, parseReceiptWineHints, structured candidate collection, alias-поиск и explainable scoring, отдаёт `DraftWineSuggestion`. Карточки результата теперь богатые: fullscreen-фото по тапу, винодельня, reason-pill, `SuggestionConfidenceBar` (%), блоки «почему похоже»/«проверьте», difference-note, open-card + в погребок. Кастомные `BottleCandidate`/`BottleMatchSignal` удалены.
  - Задача 2 (коммит 7a90ec1): в ручном добавлении поле «Где куплено» получило быстрые чипы магазинов пользователя из его чеков (`fetchKnownShops`/`knownShopsProvider`/`KnownShop`); тап переносит имя+адрес+гео в контекст бутылки. Полный раздел «Мои места» с переименованием — отдельный слайс.
  - Задача 3: написано ТЗ [my_places_user_shops_tz_2026_06_13.md](/R:/Flutter/Project/winepool_final/docs/my_places_user_shops_tz_2026_06_13.md), зарегистрировано в documentation_map.
- Доработки confirm sheet 13.06.2026: винтаж с этикетки прокидывается в диалог найденного вина; добавлено место покупки (поле + пикер). Backend: миграция `20260613_add_manual_shop_to_user_storage.sql` (RPC add_to_user_storage получил manual shop-параметры) применена на live после review.
- Пикер магазина 13.06.2026 (по фидбеку: чипы с юр.названиями нечитаемы): отдельный shop_picker_sheet.dart с режимами «Списком» (полное имя + адрес + дата) и «На карте» (Yandex MapKit, выбор точки тапом), на данных userPurchaseMapProvider. Подключён в confirm sheet и manual draft вместо чипов.
- Проход 5 (guest mode, аналитика, polish) — ВЫПОЛНЕН 13.06.2026 (частично):
  - Guest mode: безопасный подход без сериализации — при госте в момент сохранения (confirm sheet и manual draft) показывается мягкая стена регистрации (showRegistrationWallSheet, kind openCellar/createDraft). После регистрации форма остаётся, пользователь повторяет сохранение. Полная continuity с авто-replay manual draft + локальное фото — осознанный долг (PendingGuestActionType.addBottleDraft), вынесен в follow-up.
  - Аналитика (AppMetrica): добавлены события add_bottle_opened (хаб), add_bottle_path_selected (barcode/label/name), add_bottle_match_result (label/barcode), add_bottle_added_to_cellar (с path/price/place), add_bottle_draft_created (entry_method/photo/barcode/submitted), add_bottle_guest_wall_shown.
  - Polish: confirm-leave (PopScope) на manual draft при несохранённом вводе; double-tap guard через _isSaving уже был; rate-limit и ошибки локализованы.
  - flutter analyze чистый по add_bottle. Коммиты: ed237ed (shop picker), + текущий (guest/analytics/polish).
  - ОСТАЁТСЯ для полного закрытия прохода 5: ручной прогон QA-плана (раздел 12) на устройстве; doc sync administrator_guide/changelog; опционально полная guest continuity и offline-сценарии матчинга.
- Следующий: ручной QA на устройстве по разделу 12 + опциональные долги выше.
Контекст: главный activation/retention-слой следующего релиза. Закрывает разрыв `установка -> первая бутылка в погребке` для пользователя без чека.

Связанные документы:

- [draft_wine_flow_spec.md](/R:/Flutter/Project/winepool_final/docs/draft_wine_flow_spec.md) — базовая UX/data модель draft flow (расширяется этим ТЗ)
- [catalog_normalization_moderation_tz_2026_05_11.md](/R:/Flutter/Project/winepool_final/docs/catalog_normalization_moderation_tz_2026_05_11.md) — модераторский workbench (intake расширяется этим ТЗ)
- [next_rustore_update_feedback_plan_2026_05_27.md](/R:/Flutter/Project/winepool_final/docs/next_rustore_update_feedback_plan_2026_05_27.md) — данные воронки и P0 «замкнуть модерацию на пользователя»
- [mvp_catalog_contribution_gamification_2026_05_09.md](/R:/Flutter/Project/winepool_final/docs/mvp_catalog_contribution_gamification_2026_05_09.md) — XP-слой (переиспользуется без изменений весов)
- [draft_catalog_moderation_abuse_controls_tz_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/draft_catalog_moderation_abuse_controls_tz_2026_04_21.md) — anti-abuse контур (Phase 1 становится обязательным до открытия manual intake)
- [tz_guest_mode_retention_2026_04_25.md](/R:/Flutter/Project/winepool_final/docs/tz_guest_mode_retention_2026_04_25.md) — guest-first принципы и pending guest actions

---

## 1. Зачем Это Нужно

### 1.1. Диагноз

Данные после первой рекламы (04.06.2026): RuStore 690 просмотров -> 10 установок -> 0 регистраций.

Onboarding уже исправлен (нейтральная кнопка `В приложение`), но второй слой проблемы остаётся: **самый сильный flow приложения — чек с QR — требует артефакта, которого у нового пользователя в момент установки почти никогда нет.** Все остальные пути добавления вина либо слабые, либо тупиковые:

- сканирование этикетки (`/wine-label-ocr`) — прототип вне дизайн-системы; при ненайденном вине кнопка `Добавить вино` показывает snackbar «в разработке» — мёртвый конец в ключевом сценарии «стою с бутылкой»;
- пустой поиск говорит «Попробуйте изменить запрос» — пользователь, который хочет отдать данные о вине, не может этого сделать;
- черновик вина можно создать только из строки чека: live-проверка 12.06.2026 подтвердила, что все 41 draft в проде имеют `source_type = 'receipt_unmatched'`;
- бутылка из заявки появляется в погребке только **после** одобрения модератором; до этого черновики живут в отдельной очереди за баннером, а не на полке.

### 1.2. Решение

Один универсальный flow «Добавить бутылку» с тремя входами (штрихкод, этикетка, название), который:

1. мгновенно матчит против каталога (включая alias-слои);
2. при находке добавляет в погребок в 1-2 тапа;
3. при отсутствии — создаёт предзаполненный черновик, отправляет в существующий модерационный конвейер **и сразу показывает бутылку в погребке со статусом «на проверке»**;
4. после одобрения бутылка автоматически «превращается» в каноническое вино (расширение уже работающей механики `20260527`).

Ключевой аргумент за приоритет штрихкода: live-проверка 12.06.2026 показала, что **1164 из 1170 вин каталога (99.5%) имеют barcode**. Сканирование штрихкода — самый дешёвый и самый точный вход.

### 1.3. North Star метрика итерации

```text
Время от установки до первой бутылки в погребке < 2 минут, без чека.
```

Поддерживающие метрики:

- install -> first bottle added (любым путём);
- add_bottle opened -> bottle added (конверсия flow);
- доля добавлений по путям: barcode / label / name / receipt;
- not found -> draft submitted (конверсия в вклад);
- draft submitted -> approved time (SLA модерации);
- guest add attempt -> registration (захват ценности).

---

## 2. Главные Принципы

1. **Бутылка важнее каталога.** Пользователь пришёл положить бутылку на полку, а не наполнять каталог. Каталожный вклад — побочный продукт, оформленный как благодарность (XP уже есть).
2. **Никогда не тупик.** Каждая ветка flow заканчивается либо бутылкой в погребке, либо черновиком в погребке «на проверке». Ветки «ничего нельзя сделать» не существует.
3. **Мгновенная видимость.** Результат действия виден сразу на полке. Модерация — фоновый процесс, который улучшает карточку, а не условие появления бутылки.
4. **Переиспользование конвейера.** Draft -> submission -> workbench -> atomic finalize -> XP -> уведомления -> автодобавление: всё уже построено. Это ТЗ добавляет новые входы в конвейер, а не новый конвейер.
5. **Guest-first.** Сканировать и распознавать можно гостем; регистрация запрашивается в момент сохранения (паттерн `PendingGuestActionType` уже есть).
6. **Manual intake не открывается без rate limits.** Anti-abuse Phase 1 (draft-level) — обязательная часть поставки, не follow-up.

---

## 3. Текущее Состояние (Факты Кода И Live DB На 12.06.2026)

### 3.1. Код

| Поверхность | Файл | Состояние |
|---|---|---|
| Погребок | `lib/features/cellar/presentation/my_cellar_screen.dart` (1110 строк) | вкладки «Продегустировано»/«Хранение»; empty state с CTA `Сканировать чек` и `Добавить вручную` (ведёт в `/catalog`); черновики — только баннер `_DraftsBanner` со ссылкой на `/draft-wines` |
| Cellar data | `lib/features/cellar/data/cellar_repository.dart` | RPC `get_user_storage`, `add_to_user_storage`, `update_storage_item_quantity`, `delete_user_storage_item`, `get_user_analytics`, `add_user_tasting` |
| Добавление из карточки вина | `lib/features/wines/presentation/wine_details_screen.dart` (~2030-2080) | AlertDialog: количество/цена/винтаж/дата -> `cellarController.addToStorage` |
| Label OCR | `lib/features/wines/presentation/wine_label_ocr_screen.dart` (365 строк) | прототип: коричневая палитра, сырой OCR-текст, берёт `wines.first`, «Добавить вино» при not-found — snackbar-заглушка |
| Label matching | `lib/features/wines/application/wine_label_search_controller.dart`, `lib/services/wine_label_text_processor.dart` | многоступенчатый поиск по `searchWines`/`searchAll`, НЕ использует alias-слои и нормализацию receipt matcher |
| OCR-сервисы | `lib/services/yandex_ocr_service.dart`, `google_mlkit_text_recognition` | ML Kit сконфигурирован `TextRecognitionScript.latin` — кириллические этикетки распознаёт только Yandex OCR; выбор сервиса в `ocrServiceProvider` |
| Поиск | `lib/features/search/presentation/search_results_screen.dart` (~251-260) | пустой результат -> `AppEmpty` без CTA |
| Draft creation | `lib/features/wines/application/receipt_controller.dart` -> `createDraftWineFromReceiptItem` (~10727) | работает только от receipt item; использует `parseReceiptWineHints`, нормализацию, grape detection |
| Submission | `receipt_controller.dart` -> `submitDraftForCatalogReview` (~5082) | `submission_type = 'new_wine_from_draft'`; resubmit loop, XP за уточнение |
| Guest continuity | `lib/features/auth/application/guest_action_continuity_controller.dart` | `PendingGuestActionType.addToStorage` уже есть с continuity после регистрации |
| Router | `lib/core/router.dart` | `/draft-wines` помечен capability `create_draft`; `/wine-label-ocr` существует |
| Barcode deps | `pubspec.yaml` | `mobile_scanner ^6.0.2`, `google_mlkit_barcode_scanning ^0.14.0` уже подключены |

### 3.2. Live DB (боевой self-host, проверено через SSH 12.06.2026)

> Важно: `SUPABASE_DB_READONLY_URL` в `.env` указывает на устаревшую cloud-базу (`aws-1-eu-central-1.pooler.supabase.com`). Аудит и проверки этого ТЗ выполнены против боевой базы на VPS (`docker exec supabase-db psql`). `docs/SELF_DEBUG_GUIDE.md` нужно актуализировать.

Факты схемы:

- `draft_wine_sources.receipt_id` и `receipt_item_id` — `NOT NULL` (FK на `user_receipts`/`user_receipt_items`); `unit_price NOT NULL`. **Manual-источник без миграции невозможен.**
- `user_storage.wine_id` — `NOT NULL`; receipt-контекст (`source_receipt_id`, `source_receipt_item_id`, `source_shop_name`, `source_shop_address`, `source_shop_lat/lng`) уже есть (миграция `20260526`).
- `add_approved_draft_submission_to_user_storage(p_submission_id)` + триггер `handle_approved_draft_submission_storage` (миграция `20260527`): добавляет бутылки в погребок после approve, но **только** для sources с `receipt_id IS NOT NULL AND receipt_item_id IS NOT NULL` — manual-источники выпадут.
- Alias-инфраструктура полная: `wine_aliases`, `winery_aliases`, `country_aliases`, `region_aliases`, `grape_variety_aliases`, `catalog_normalization_decisions` — всё существует на проде.
- `user_experience_events`, `app_notifications` — существуют.
- Данные: 1170 вин (1164 с barcode), 41 draft (все `receipt_unmatched`), 17 submissions (13 approved), 49 строк `user_storage` у 9 пользователей. Объём маленький — миграции безопасны.
- Статусы в живых данных: `draft_wines.status`: `new/linked_to_existing/promoted/enriched/merged`; `submissions.status`: `approved/submitted/in_review/rejected`.

---

## 4. Scope Поставки

### P0 — это ТЗ, обязательный объём

1. Хаб «Добавить бутылку» (route `/add-bottle`) с тремя входами: штрихкод, этикетка, название.
2. Путь A: сканирование штрихкода -> exact match -> добавление в погребок.
3. Путь B: фото этикетки -> OCR -> матчинг через нормализацию + alias-слои -> подтверждение/добавление.
4. Путь C: поиск по названию внутри flow + CTA «Добавить вино, которого нет» в пустом поиске приложения.
5. Ветка not-found: создание manual draft (предзаполнение из OCR/штрихкода/запроса, фото прикреплено) -> отправка на модерацию -> **бутылка сразу в погребке со статусом «на проверке»**.
6. Backend: миграция manual sources, расширение автодобавления `20260527` на manual-источники, rate limits для manual drafts.
7. Workbench: корректное отображение заявок без чека (фото — главное evidence).
8. Замена `/wine-label-ocr` новым flow (redirect со старого route).
9. Guest mode: распознавание гостем, регистрация в момент сохранения.
10. Аналитические события flow.

### P1 — следующий слой после P0 (отдельные slices, в этом ТЗ — только рамка)

- Обогащение диалога добавления из карточки вина: место покупки (магазин/гео), быстрая оценка «уже пил».
- Push/in-app «Ваше вино добавлено в каталог» с deep link на `/wine/:id` (часть уже сделана в email-контуре).
- Barcode-first дедупликация в модерации: предупреждение модератору при совпадении barcode заявки с каталогом.
- Авто-link по памяти этикеток (расширение «Память строк из чеков» из feedback-плана).
- Abuse Controls Phase 2 (user-level restrictions).

### P2 — retention-слой погребка (отдельное будущее ТЗ)

Source of truth для этого слоя теперь:
[wine_cellar_pro_tz_2026_06_23.md](/R:/Flutter/Project/winepool_final/docs/wine_cellar_pro_tz_2026_06_23.md).

- «Окно зрелости»: использование `ideal_drink_from/to` + push «2 бутылки входят в пик».
- Быстрая фиксация «Открыл бутылку» за 10 секунд -> оценка -> публичный отзыв.
- Еженедельный дайджест вклада на базе `user_experience_events`.
- Рекомендации «что открыть сегодня».
- Места хранения, расположение бутылок, перемещение и журнал движения — через `Cellar Pro`, не внутри add-bottle scope.

### Не входит в scope (осознанно)

- AI-автозаполнение черновика по фото без ручного подтверждения пользователем полей.
- Распознавание винтажа/этикетки через внешние ML-сервисы кроме уже подключённых OCR.
- Социальная лента, шеринг полки.
- Изменение схемы `user_storage` под draft-бутылки (см. решение 6.3 — client-side merge в MVP).
- Массовый импорт каталога.
- Перестройка receipt matcher.

---

## 5. Целевой UX

### 5.1. Точки входа

1. **Погребок**: FAB `+ Добавить вино` на обеих вкладках (заменяет текущую пару кнопок в empty state; в empty state — те же действия крупными кнопками: `Сканировать штрихкод`, `Сфотографировать этикетку`, `Найти по названию`, `Сканировать чек`).
2. **Главная** (`buyer_home_screen.dart`): действие `Добавить вино` в блоке быстрых действий рядом со `Сканировать чек` (не вместо).
3. **Пустой поиск** (`search_results_screen.dart`): под `AppEmpty` кнопка `Добавить вино, которого нет в каталоге` -> `/add-bottle?query=<текущий запрос>` (сразу путь C с предзаполненным запросом).
4. **Карточка вина остаётся как есть** (добавление найденного вина) — flow ведёт на неё в ветке «найдено», либо показывает собственный compact-подтверждающий sheet (см. 5.5).

### 5.2. Хаб `/add-bottle`

Экран в дизайн-системе приложения (тёмная premium-тема, Playfair Display для заголовка):

- заголовок: `Добавить бутылку`;
- подзаголовок: `Сфотографируйте штрихкод или этикетку — мы найдём вино в каталоге`;
- три крупные карточки-действия:
  - `Штрихкод` (icon barcode) — «самый быстрый способ»;
  - `Этикетка` (icon camera) — «если штрихкода нет под рукой»;
  - `По названию` (icon search) — «введите название вручную»;
- внизу вторичная ссылка: `Есть чек с QR? Сканировать чек` -> `/receipt-qr-scanner`.

Параметры route: `/add-bottle?path=barcode|label|name&query=<prefill>`. При наличии `path` хаб пропускается и открывается сразу нужный шаг (для deep-входов из поиска и погребка).

### 5.3. Путь A: Штрихкод

1. Камера через `mobile_scanner` (паттерн и permission-обработка — из `receipt_qr_scanner_screen.dart`, включая denied/permanently denied UX, который уже доведён там).
2. Распознан EAN/UPC -> запрос exact match по `wines.barcode` (нормализованное сравнение: trim, без пробелов).
3. **Найдено одно вино** -> экран подтверждения (5.5).
4. **Найдено несколько** (дубли в каталоге) -> список кандидатов, выбор -> 5.5.
5. **Не найдено** -> переход в ветку not-found (5.6) с предзаполненным `barcode_draft`; предложить сфотографировать этикетку («Сфотографируйте этикетку, чтобы мы добавили это вино в каталог»).
6. Fallback-действия на экране сканера: `Ввести штрихкод вручную`, `Перейти к этикетке`, `Из галереи`.

### 5.4. Путь B: Этикетка

1. Камера/галерея (паттерн permission UX тот же).
2. OCR: использовать `ocrServiceProvider` как сейчас, но с фиксом кириллицы:
   - если выбран ML Kit — распознавать **двумя** проходами (`TextRecognitionScript.latin` + ML Kit по умолчанию для кириллицы недоступен в standalone-скрипте, поэтому: при наличии Yandex OCR ключа кириллический проход делать через Yandex; если ключа нет — честно предупредить, что кириллические этикетки распознаются хуже);
   - результат OCR пользователю в сыром виде **не показывать** — только извлечённые кандидаты полей.
3. Матчинг распознанного текста (раздел 7) -> топ-кандидаты с confidence.
4. UI кандидатов: карточки `фото-миниатюра каталога + название + винодельня + сигнал совпадения` (паттерн confidence strip и highlight совпавших слов уже реализован в draft details suggestions — переиспользовать виджеты).
5. **Уверенный кандидат** -> экран подтверждения (5.5) с возможностью `Это не оно`.
6. **Слабые кандидаты / пусто** -> ветка not-found (5.6); фото этикетки автоматически становится `photo_url` черновика (upload в bucket `draft_wine_photos`, RLS уже user-scoped).

### 5.5. Состояние «Найдено»: Подтверждение И Параметры Бутылки

Bottom sheet или экран:

- карточка вина (фото, название, винодельня, регион, рейтинг);
- главный CTA: `В погребок`;
- вторичный: `Открыть карточку вина`;
- блок параметров (все опциональны, collapsed по умолчанию кроме количества):
  - количество (default 1, stepper);
  - цена за бутылку;
  - винтаж;
  - дата покупки (default сегодня);
  - где куплено (текстовое поле + опционально текущая геолокация — поля `source_shop_*` в `user_storage` уже есть; для MVP достаточно `shop_name`, гео — если дёшево);
- после сохранения: snackbar `«<Вино>» в погребке` + переход `/my-cellar?tab=1` (паттерн уже есть в `wine_details_screen.dart`).

Вызов: существующий `cellarController.addToStorage` (расширить параметрами `sourceShopName/Address/Lat/Lng` — RPC `add_to_user_storage` уже принимает receipt-поля; проверить и при необходимости расширить сигнатуру RPC на shop-поля, они в таблице есть).

### 5.6. Состояние «Не Найдено»: Manual Draft

Принцип: **не длинная форма, а подтверждение догадок**. Экран `Добавим это вино в WinePool`:

1. Вверху — фото этикетки (если есть) или CTA `Добавить фото` (сильная рекомендация: бейдж `+5 очков и быстрее проверка`).
2. Поля с предзаполнением из OCR/штрихкода/поискового запроса:
   - название (обязательное, единственное обязательное поле);
   - винодельня;
   - цвет / сахар / тип (chips);
   - страна / регион;
   - винтаж;
   - объём;
   - штрихкод (предзаполнен из пути A; кнопка `Сканировать`);
   - сорт(а) винограда.
3. Индикатор `Готовность к проверке: N из 6` — переиспользовать существующий компонент из receipt draft flow.
4. Блок «Возможно, это уже есть» — топ-3 существующих вин (suggestion engine из draft details) + топ-3 похожих drafts; CTA `Это оно` сразу уводит в 5.5.
5. Блок параметров бутылки (как в 5.5: количество, цена, дата, место) — **эти данные пойдут в `draft_wine_sources` manual-источник**, чтобы после approve бутылка с ценой и местом перенеслась в `user_storage`.
6. Главный CTA: `Сохранить и отправить на проверку` (submit включён по умолчанию; чекбокс `Только сохранить черновик` для отказа от отправки — инверсия текущего тумблера, потому что в этом flow интент очевиден).
7. После сохранения:
   - draft создан (`source_type = 'manual'`, см. 6.1);
   - manual source создан с параметрами бутылки;
   - submission отправлен (если не отключено);
   - снимается экран успеха: `Бутылка на вашей полке. Обычно проверяем за 1-2 дня — мы пришлём уведомление` + CTA `В погребок`;
   - **в погребке бутылка видна сразу** (5.7).

### 5.7. Бутылка «На Проверке» В Погребке

Решение MVP (см. трейд-офф в 6.3): вкладка «Хранение» рендерит объединённый список:

1. Секция/группа `На проверке` сверху списка (или интегрированные карточки с бейджем) — данные из `draft_wines` текущего пользователя со статусами `new/needs_review/enriched` + активной submission, у которых есть manual/receipt source с интентом хранения;
2. Карточка draft-бутылки: фото черновика, название, винодельня, количество/цена из source, бейдж `На проверке` (жёлтый) / `Нужно уточнение` (оранжевый, тап -> draft details) / `Отклонено` (серый, тап -> причина);
3. Тап -> `DraftWineDetailsScreen` (существует);
4. Итоговые счётчики полки (`X бутылок на сумму Y`) **не включают** draft-бутылки; в шапке группы — собственный счётчик `N на проверке`;
5. Существующий `_DraftsBanner` в этой вкладке заменяется этой группой (баннер остаётся на вкладке «Продегустировано» как есть либо удаляется — решить при реализации по месту);
6. После approve: триггер уже создаёт строку `user_storage` -> при следующем обновлении полки draft-карточка исчезает, появляется каноническая бутылка. Realtime-уведомление (`app_notifications`) уже приходит — по нему инвалидировать `cellarStorageProvider` и `draftWineQueueProvider`, чтобы «превращение» происходило на глазах.

### 5.8. Guest Mode

- Хаб, сканер, OCR, матчинг и просмотр кандидатов доступны гостю полностью (ценность до стены);
- Стена регистрации — в момент `В погребок` или `Сохранить и отправить на проверку`:
  - найденное вино: существующий `PendingGuestActionType.addToStorage` (continuity уже реализован);
  - manual draft: новый `PendingGuestActionType.addBottleDraft` — сохранить введённые поля + путь к локальному файлу фото в pending payload, после регистрации повторить создание draft;
- Стена мягкая (bottom sheet, объяснение «чтобы бутылка сохранилась на вашей полке», `Закрыть` доступен) — по принципам guest-retention ТЗ.

### 5.9. Тексты (Baseline Копирайта)

```text
Хаб: «Добавить бутылку»
Сабтайтл: «Сфотографируйте штрихкод или этикетку — мы найдём вино в каталоге»
Найдено: «Похоже, это {name}»
Не найдено (barcode): «Этого вина ещё нет в WinePool. Добавьте его — станете первооткрывателем»
Не найдено (label): «Не нашли точное совпадение. Проверьте похожие или добавьте новое вино»
Успех (найдено): «“{name}” в погребке»
Успех (draft): «Бутылка на вашей полке. Проверим данные и добавим вино в каталог — обычно это 1-2 дня»
Бейдж: «На проверке» / «Нужно уточнение» / «Отклонено»
Поиск-тупик: «Добавить вино, которого нет в каталоге»
```

Все строки — через l10n (`lib/l10n/app_ru.arb` + `app_en.arb`), не хардкодом.

---

## 6. Данные И Backend

### 6.1. Миграция 1: Manual Sources

`supabase/migrations/2026MMDD_add_manual_draft_sources.sql`:

1. `draft_wine_sources`:
   - `receipt_id` -> nullable;
   - `receipt_item_id` -> nullable;
   - `unit_price` -> nullable (или оставить NOT NULL DEFAULT 0 — решить при реализации; рекомендация: nullable, чтобы не выдумывать нулевые цены);
   - новая колонка `source_kind text NOT NULL DEFAULT 'receipt'` со значениями `receipt | manual`;
   - CHECK: `(source_kind = 'receipt' AND receipt_id IS NOT NULL AND receipt_item_id IS NOT NULL) OR (source_kind = 'manual')`;
   - новые опциональные колонки контекста покупки для manual: `shop_address text`, `shop_lat double precision`, `shop_lng double precision` (`shop_name`, `purchase_date`, `quantity`, `unit_price` уже есть);
   - индекс по `(draft_wine_id, source_kind)`.
2. `draft_wines.source_type`: новое допустимое значение `manual` (если есть CHECK — расширить; метод входа писать в `draft_wine_actions.metadata.entry_method`: `barcode | label_ocr | name_search`).
3. RLS: проверить существующие политики `draft_wine_sources` — они написаны под receipt-провенанс; убедиться, что insert/select собственных manual-источников разрешён владельцу draft.
4. Backfill не нужен (все существующие sources — receipt).

### 6.2. Миграция 2: Автодобавление Manual-Бутылок После Approve

#### P1 follow-up 12.07.2026: contribution counters diverge from source-of-truth

Наблюдение на реальных пользователях: XP и уровень растут, но профиль показывает `0` чеков и `0` бутылок, хотя подтверждённые submission/receipt и строки погребка существуют.

Подтверждённая архитектурная причина для moderation/manual path:

- `admin_finalize_catalog_normalization_decision` вызывает `award_user_experience`;
- `award_user_experience` обновляет `experience_points` и `level`, но не `user_levels.wines_added`/`receipts_scanned`;
- `add_approved_draft_submission_to_user_storage` идемпотентно создаёт `user_storage`, но не синхронизирует `wines_added`;
- профиль читает mutable counters из `user_levels`, поэтому XP ledger, storage и counters расходятся.

Receipt path требует отдельного replay-аудита: `add_experience_for_scan` обновляет counters только при первом успешно вставленном XP event. Нужно проверить, что все современные receipt save paths вызывают эту RPC ровно один раз и не используют только generic `award_user_experience`.

Исправление вести отдельным P1 bugfix slice, связанным с add-bottle/moderation acceptance, а не помещать внутрь AI worker:

1. Зафиксировать семантику: `receipts_scanned` = число уникальных сохранённых пользовательских чеков; `wines_added` в пользовательском UI переименовать/трактовать как число бутылок в погребке и определить, считать ли quantity или storage rows.
2. Выбрать canonical sources: `user_receipts` для чеков и `user_storage` для бутылок. Mutable counters не должны быть единственным source-of-truth.
3. Предпочтительно считать профиль через агрегирующую RPC/view либо поддерживать counters транзакционными idempotent triggers на canonical tables.
4. Не увеличивать bottles counter повторно при moderation approve, если соответствующая `user_storage` уже существует.
5. Выполнить dry-run reconciliation для всех пользователей: показать stored counter, derived count и delta.
6. После проверки сделать idempotent backfill для существующих пользователей, включая текущие реальные кейсы Юлии и Татьяны.
7. Добавить integration tests: manual submission -> approve -> storage -> count; receipt -> save -> count; repeated approve/save -> count unchanged; quantity semantics; rejected submission -> count unchanged.
8. XP ledger оставить отдельной метрикой: исправление counters не должно повторно начислять опыт.

Расширить `add_approved_draft_submission_to_user_storage`:

- текущий SELECT по `draft_wine_sources` отбирает только `receipt_id IS NOT NULL AND receipt_item_id IS NOT NULL` — добавить ветку для `source_kind = 'manual'`:
  - `purchase_price` <- `dws.unit_price`;
  - `purchase_date` <- `dws.purchase_date`;
  - `quantity` <- `GREATEST(COALESCE(dws.quantity,1),1)`;
  - `vintage` <- `v_draft.vintage_draft`;
  - `source_shop_name/address/lat/lng` <- из manual-колонок source;
  - `source_receipt_id/item_id` -> NULL;
- идемпотентность для manual: NOT EXISTS по `(user_id, wine_id, source_draft_source_id)` — рекомендуемый способ: добавить в `user_storage` nullable колонку `source_draft_source_id uuid` и дедуплицировать по ней (receipt-ветка остаётся на receipt-паре);
- повторный прогон функции не создаёт дублей ни для одной ветки (это уже acceptance в `20260527`, сохранить).

### 6.3. Draft-Бутылки В Погребке: Принятое Решение

Рассматривались два варианта:

- **(a) Унификация схемы**: `user_storage.wine_id` nullable + `draft_wine_id` + CHECK exactly-one. Чисто, но трогает все cellar RPC (`get_user_storage`, `add_to_user_storage`, analytics), модель `UserStorageItem` и каждый UI-консьюмер. Высокий риск регрессии ради MVP.
- **(b) Client-side merge (ПРИНЯТО для P0)**: `user_storage` не меняется; вкладка «Хранение» дополнительно читает свои drafts (провайдер уже есть — `draftWineQueueProvider`) и рендерит группу `На проверке`. Параметры бутылки живут в manual source. После approve серверный триггер сам создаёт каноническую строку.

Вариант (a) зафиксировать как post-MVP unification, отдельным slice, когда появится потребность (аналитика по pending, сортировки и т.п.).

### 6.4. Barcode Pre-Check

- Клиентский запрос exact match по `wines.barcode` (через `wines_repository`); проверить наличие индекса на `wines.barcode` — если нет, добавить в миграцию 1 (`create index if not exists idx_wines_barcode on wines (barcode) where barcode is not null`).
- В ветке not-found пути A barcode обязательно сохраняется в `barcode_draft` (это +3 XP по существующим правилам и сильный сигнал модератору).
- В блоке «Возможно, это уже есть» (5.6) barcode-совпадение — наивысший приоритет с явным лейблом `Совпадает штрихкод`.

### 6.5. Rate Limits (Abuse Controls Phase 1 Для Manual Intake)

Сервер-сайд (RPC/триггер, не только UI), значения конфигурируемые:

- максимум **5 активных** manual drafts без решения модератора на пользователя;
- максимум **10 manual drafts в сутки** на пользователя;
- при превышении — понятная ошибка: `Вы уже отправили N вин на проверку. Дождитесь решения модератора — мы пришлём уведомление`;
- лимиты не распространяются на receipt-драфты (там есть естественный лимит чеков, и контур уже живёт);
- админ/модератор исключены из лимитов;
- хранить счётчики не нужно — считать запросом по `draft_wines` (объёмы маленькие).

Согласовать с [draft_catalog_moderation_abuse_controls_tz_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/draft_catalog_moderation_abuse_controls_tz_2026_04_21.md): этот пункт — частичная реализация его Phase 1, не конфликтующая.

### 6.6. Контроллер/Repository Слой (Flutter)

Не расширять `receipt_controller.dart` (16427 строк — уже перегружен). Создать фичу `lib/features/add_bottle/`:

```text
lib/features/add_bottle/
  application/add_bottle_controller.dart      # state machine flow
  application/bottle_match_controller.dart    # матчинг barcode/label/name -> кандидаты
  data/manual_draft_repository.dart           # createManualDraftWine, createManualDraftSource
  domain/add_bottle_models.dart               # AddBottlePath, BottleCandidate, BottleParams, DraftBottlePrefill
  presentation/add_bottle_hub_screen.dart
  presentation/barcode_scan_step.dart
  presentation/label_scan_step.dart
  presentation/name_search_step.dart
  presentation/candidate_confirm_sheet.dart
  presentation/manual_draft_screen.dart
  presentation/widgets/...
```

`createManualDraftWine` повторяет структуру `createDraftWineFromReceiptItem` (insert draft + source + action + XP best-effort + draft quality XP), но:

- `source_type: 'manual'`;
- `draft_wine_sources.source_kind: 'manual'`, receipt-поля NULL, параметры бутылки из формы;
- `draft_wine_actions.metadata.entry_method`: `barcode | label_ocr | name_search`;
- фото: upload в `draft_wine_photos` (существующий путь `<uid>/...`), затем `photo_url`;
- отправка на проверку — существующий `submitDraftForCatalogReview` (он не привязан к receipt и должен заработать как есть; проверить snapshot builder на NULL receipt-полях).

Хелперы парсинга (`parseReceiptWineHints`, `normalizeReceiptItemName`, inference-функции) сейчас приватные в `receipt_controller.dart` — вынести используемые в отдельный shared-модуль (например, `lib/features/wines/application/wine_text_hints.dart`) без изменения поведения, чтобы оба контроллера их переиспользовали.

---

## 7. Матчинг Этикетки И Названия

Текущий `wine_label_search_controller.dart` не использует нормализацию и alias-слои. Целевой конвейер для путей B и C:

1. Нормализация текста — та же функция, что в receipt matcher (`normalizeReceiptMatcherText` / `normalize_catalog_alias_name` подход).
2. Извлечение кандидатов полей из OCR-текста: переиспользовать `WineLabelTextProcessor` + `parseReceiptWineHints` (цвет/сахар/тип/объём/сорта уже детектятся).
3. Поиск кандидатов (порядок приоритета):
   - exact barcode (если в кадре этикетки распознался и штрихкод — ML Kit barcode умеет это из того же изображения);
   - `wine_aliases.normalized_alias_name` exact/prefix;
   - `wines.name` нормализованный full-text/ilike + фильтр по найденной винодельне;
   - `winery_aliases` -> сужение по винодельне -> вина этой винодельни;
   - fuzzy по словам (текущий `_multiStageSearch` как последний ярус).
4. Ранжирование: повторить explainable-подход draft suggestions (score + главный сигнал: `Совпадает штрихкод` / `Совпадает название и винодельня` / `Совпадает alias`), а не молчаливый score.
5. Пороговые правила:
   - score >= high -> сразу экран подтверждения с одним кандидатом;
   - middle -> список топ-3 с `Это не оно -> Добавить новое вино`;
   - low/пусто -> ветка not-found.
6. Каждое подтверждение пользователя (`Это оно` после label-матча) — потенциальный будущий `wine_aliases` сигнал; в P0 просто логировать событие аналитики, авто-alias не писать (это P1 «память этикеток»).

---

## 8. Модерация: Изменения Workbench

Минимальные — конвейер общий:

1. **Evidence panel**: для manual-заявок нет чека -> вместо блока «строка чека / магазин / цена» показывать `Источник: добавлено пользователем вручную (штрихкод | этикетка | поиск)` + фото как главное evidence + параметры бутылки из manual source (цена/место, если указаны). Не показывать пустые receipt-блоки.
2. **Очередь заявок** (`admin_draft_catalog_submissions_screen.dart`): бейдж источника `Чек` / `Вручную`; фильтр по источнику.
3. **Snapshot payload** (`_buildDraftCatalogSubmissionSnapshot`): проверить на NULL receipt-полях, добавить `entry_method` в snapshot.
4. **Финализация** (`admin_finalize_catalog_normalization_decision`): изменений не требует — автодобавление в погребок делает расширенный RPC из 6.2 (проверить, что триггер срабатывает на manual-заявках).
5. **Уведомления**: контур `app_notifications` + email общий для всех заявок — проверить, что тексты не ссылаются на чек.

---

## 9. Аналитика Flow

События (через существующий аналитический слой; если его нет — минимально в `user_experience_events` не писать, это не XP; завести лёгкую таблицу `app_events` или логировать в существующий механизм, решить при реализации по факту наличия):

```text
add_bottle_opened {entry: cellar|home|search_empty|deeplink}
add_bottle_path_selected {path: barcode|label|name}
add_bottle_match_result {path, result: single|multiple|none, candidates_count}
add_bottle_added_to_cellar {path, wine_id, with_price: bool, with_place: bool}
add_bottle_draft_created {path, readiness_score, has_photo, has_barcode}
add_bottle_draft_submitted {draft_id}
add_bottle_guest_wall_shown / add_bottle_guest_wall_converted
```

Дашборд-вопросы, на которые должны отвечать данные: какой путь работает, где отваливаются, конверсия not-found -> draft, время до первой бутылки.

---

## 10. UX-Состояния И Ошибки

Обязательные состояния:

- camera permission denied / permanently denied (переиспользовать готовый UX из receipt scanner);
- OCR не вернул текст -> «Не удалось прочитать этикетку. Попробуйте при лучшем освещении или добавьте по названию»;
- Yandex OCR недоступен/нет ключа -> деградация на ML Kit + предупреждение для кириллицы;
- сеть недоступна на шаге матчинга -> retry + переход к ручному вводу;
- upload фото упал -> черновик сохраняется без фото, фото можно дослать из draft details (существующий путь);
- rate limit -> текст из 6.5;
- двойное нажатие «Сохранить» -> идемпотентность на клиенте (disable + единый draft);
- уход с экрана manual draft с заполненными полями -> confirm-leave.

---

## 11. Критерии Приёмки

Функциональные:

- [ ] из погребка, главной и пустого поиска можно открыть `/add-bottle`;
- [ ] штрихкод существующего вина -> бутылка в погребке за <= 3 тапа после распознавания;
- [ ] штрихкод несуществующего вина -> предзаполненный черновик со штрихкодом;
- [ ] фото этикетки вина из каталога -> правильный кандидат в топ-1/топ-3 (проверить на 10 реальных бутылках);
- [ ] фото этикетки несуществующего вина -> черновик с прикреплённым фото и предзаполненными полями;
- [ ] поиск по названию «нет в каталоге» -> CTA -> черновик с предзаполненным названием;
- [ ] manual draft виден в погребке как `На проверке` сразу после сохранения;
- [ ] manual draft попадает в обычную модерационную очередь, workbench открывает его без чека и без ошибок;
- [ ] approve manual-заявки -> бутылка в `user_storage` с ценой/датой/местом из формы, draft-карточка из группы `На проверке` исчезает;
- [ ] повторный approve/повторное сохранение модерации не создаёт дублей;
- [ ] reject -> карточка `Отклонено` с причиной, бутылка не появляется в канонической части;
- [ ] запрос уточнения -> бейдж `Нужно уточнение`, ответ пользователя идёт по существующему resubmit loop;
- [ ] XP начисляются по существующим правилам (фото +5, штрихкод +3, поля +2, принятие +20/+30) идемпотентно;
- [ ] гость проходит распознавание, упирается в мягкую стену на сохранении, после регистрации действие доигрывается;
- [ ] rate limits работают сервер-сайд и дают понятную ошибку;
- [ ] старый route `/wine-label-ocr` редиректит на `/add-bottle?path=label`, прототипный экран удалён;
- [ ] receipt flow, draft queue/details, workbench, существующее автодобавление receipt-бутылок — без регрессий.

Технические:

- [ ] `flutter analyze` без новых ошибок;
- [ ] `dart format` по новым файлам;
- [ ] миграции применены на live только после review + backup (протокол как в `20260527`: backup -> dry-run -> apply);
- [ ] RLS: пользователь не может читать/писать чужие manual sources;
- [ ] таргетные тесты: манипуляции source_kind CHECK, идемпотентность расширенного RPC, rate limit, матчинг-ранжирование (unit по score), prefill из hints.

Продуктовые (через 2 недели после релиза):

- [ ] время install -> первая бутылка измеряется и медиана < 2 минут;
- [ ] доля установок с >= 1 бутылкой выросла против baseline;
- [ ] появились первые manual drafts от реальных пользователей.

---

## 12. QA-План (Ручной Прогон Перед Релизом)

1. Чистая установка -> гость -> хаб -> штрихкод существующего вина -> стена -> регистрация -> бутылка на полке (continuity).
2. Штрихкод несуществующего вина -> черновик -> отправка -> бутылка `На проверке` -> approve в admin web -> realtime «превращение» карточки.
3. Этикетка кириллическая (Yandex OCR) и латинская (ML Kit) — по одному реальному вину из каталога.
4. Этикетка вина не из каталога -> черновик с фото -> модерация -> отклонение -> бейдж и причина.
5. Запрос уточнения -> правка полей -> resubmit -> XP за уточнение один раз.
6. Пустой поиск -> CTA -> черновик с названием из запроса.
7. Rate limit: создать 5 активных manual drafts -> шестой блокируется с понятным текстом.
8. Регрессия: полный receipt QR прогон по `moderation_e2e_qa_prompt_2026_05_08.md` маршрутам 3-7.
9. Камера denied / permanently denied на обоих сканерах.
10. Просадка сети на шаге матчинга и на upload фото.

---

## 13. Порядок Реализации: Проходы

Каждый проход оставляет приложение в рабочем состоянии и заканчивается `dart format` + `flutter analyze` + ручной smoke.

- **Проход 0. Backend foundation**: миграция manual sources (6.1), расширение автодобавления (6.2), barcode index (6.4), rate limit RPC (6.5). Без UI.
- **Проход 1. Feature skeleton + путь C**: фича `add_bottle`, route, хаб, поиск по названию, manual draft screen (без камеры), CTA в пустом поиске, создание manual draft + submission, группа `На проверке` в погребке.
- **Проход 2. Путь A: штрихкод**: сканер, exact match, подтверждение, ветка not-found с barcode prefill.
- **Проход 3. Путь B: этикетка**: OCR, новый матчинг-конвейер (раздел 7), кандидаты с объяснениями, фото -> черновик; redirect старого route, удаление прототипа.
- **Проход 4. Workbench + модерация**: evidence для manual-заявок, бейджи источника, проверка snapshot/notify/finalize на manual, e2e через admin web.
- **Проход 5. Guest mode, аналитика, polish, QA**: pending guest action, события, тексты/l10n, confirm-leave, прогон QA-плана, doc sync.

Параллельно с проходом 0 можно закрыть pending e2e QA автодобавления receipt-бутылок (`20260527`) — он входит в release gate feedback-плана.

---

## 14. Doc Sync Обязательства

При реализации синхронно обновлять:

- этот файл — статусы критериев приёмки по проходам;
- [documentation_map.md](/R:/Flutter/Project/winepool_final/docs/documentation_map.md) — ссылка на это ТЗ как source of truth по add-bottle flow;
- [administrator_guide.md](/R:/Flutter/Project/winepool_final/docs/administrator_guide.md) — manual-заявки в очереди модератора;
- [moderation_receipt_catalog_manual_qa_plan_2026_05_08.md](/R:/Flutter/Project/winepool_final/docs/moderation_receipt_catalog_manual_qa_plan_2026_05_08.md) — новые QA-маршруты;
- [next_rustore_update_changelog_2026_05_25.md](/R:/Flutter/Project/winepool_final/docs/next_rustore_update_changelog_2026_05_25.md) — пункты «Что нового»;
- [SELF_DEBUG_GUIDE.md](/R:/Flutter/Project/winepool_final/docs/SELF_DEBUG_GUIDE.md) — предупреждение, что `SUPABASE_DB_READONLY_URL` указывает на устаревшую cloud-базу, live-аудит через VPS.

---

## 15. Промпты Для Реализации

### 15.1. Мастер-Промпт (вставлять в начало каждой сессии реализации)

```text
Привет! Продолжаем WinePool — Flutter/Supabase (self-host) проект:
R:\Flutter\Project\winepool_final

Главный документ задачи (source of truth, читать первым и целиком):
docs/add_bottle_universal_flow_tz_2026_06_12.md

Контекст одним абзацем: строим универсальный flow «Добавить бутылку» (/add-bottle)
с тремя входами — штрихкод, этикетка, название. Найденное вино добавляется в погребок
в 1-2 тапа; ненайденное превращается в manual draft, уходит в существующий
модерационный конвейер и СРАЗУ отображается в погребке со статусом «На проверке».
После approve серверный триггер переносит бутылку в user_storage. Это activation-слой:
North Star — «установка -> первая бутылка < 2 минут без чека».

Жёсткие правила:
- Это расширение существующего draft/moderation конвейера, НЕ новый конвейер.
  Переиспользуй: draft_wines/draft_wine_sources/draft_wine_actions,
  submitDraftForCatalogReview, workbench, award_user_experience, app_notifications,
  bucket draft_wine_photos, add_approved_draft_submission_to_user_storage.
- НЕ расширяй receipt_controller.dart (он уже 16k+ строк) — новая логика живёт
  в lib/features/add_bottle/; общие хелперы парсинга выноси в shared-модуль
  без изменения поведения.
- user_storage в P0 НЕ меняется под draft-бутылки: группа «На проверке» в погребке —
  client-side merge из draft-провайдеров (решение 6.3 ТЗ).
- draft_wine_sources: receipt_id/receipt_item_id становятся nullable + source_kind
  ('receipt'|'manual') + CHECK; production-данные не ломать (все текущие 41 source —
  receipt).
- Manual intake не открывается без сервер-сайд rate limits (раздел 6.5 ТЗ).
- XP-веса и правила НЕ менять — только переиспользовать существующие идемпотентные
  начисления.
- Дизайн — тёмная premium-тема приложения (AppColors, Playfair Display), все строки
  через l10n (app_ru.arb / app_en.arb), без хардкода.
- Гость может сканировать и смотреть кандидатов; стена регистрации только в момент
  сохранения (паттерн PendingGuestActionType + continuity).
- Перед изменениями: git status — не трогать чужие незакоммиченные изменения.
- Миграции на live self-host Supabase применять только после review SQL, по протоколу
  backup -> dry-run -> apply (как 20260527). Подключение к live БД для аудита —
  только через SSH на VPS (docs/powershell_ssh_sql_quoting_guide.md);
  SUPABASE_DB_READONLY_URL в .env указывает на УСТАРЕВШУЮ cloud-базу, не верить ей.

Ключевые существующие файлы:
- lib/core/router.dart
- lib/features/cellar/presentation/my_cellar_screen.dart
- lib/features/cellar/data/cellar_repository.dart, application/cellar_controller.dart
- lib/features/wines/application/receipt_controller.dart
  (createDraftWineFromReceiptItem ~10727, submitDraftForCatalogReview ~5082 —
  как образцы, не как место для нового кода)
- lib/features/wines/presentation/draft_wine_details_screen.dart,
  draft_wine_queue_screen.dart (suggestion/confidence/readiness виджеты —
  переиспользовать)
- lib/features/wines/presentation/wine_label_ocr_screen.dart (прототип на удаление)
- lib/features/wines/application/wine_label_search_controller.dart,
  lib/services/wine_label_text_processor.dart, lib/services/yandex_ocr_service.dart
- lib/features/wines/presentation/receipt_qr_scanner_screen.dart
  (permission UX камеры — образец)
- lib/features/search/presentation/search_results_screen.dart
- lib/features/auth/application/guest_action_continuity_controller.dart
- lib/features/admin/presentation/admin_draft_catalog_submission*.dart
- supabase/migrations/20260320_add_draft_wine_flow.sql,
  20260421_add_draft_wine_photo_storage.sql,
  20260526_add_receipt_context_to_storage_and_reviews.sql,
  20260527_add_approved_draft_to_user_storage.sql

Факты live DB (проверены 12.06.2026, раздел 3.2 ТЗ): 1170 вин (99.5% с barcode),
41 draft (все receipt_unmatched), draft_wine_sources.receipt_* NOT NULL,
user_storage.wine_id NOT NULL + source_shop_* колонки уже есть.

После каждого прохода:
- dart format по изменённым файлам;
- flutter analyze по затронутым файлам;
- ручной smoke основного receipt flow (не регрессировать);
- обновить статусы в docs/add_bottle_universal_flow_tz_2026_06_12.md (раздел 11);
- отдельные коммиты по смысловым блокам, на коммит и push спрашивать подтверждение.

Выполняй проходы по порядку из раздела 13 ТЗ. Не пытайся сделать всё за один заход.
Начни с прохода, указанного ниже.
```

### 15.2. Проход 0: Backend Foundation

```text
Задача прохода 0: подготовить backend для manual drafts без какого-либо UI.

Сначала прочитай целиком:
- docs/add_bottle_universal_flow_tz_2026_06_12.md (разделы 6, 3.2)
- supabase/migrations/20260320_add_draft_wine_flow.sql
- supabase/migrations/20260527_add_approved_draft_to_user_storage.sql
- supabase/migrations/20260526_add_receipt_context_to_storage_and_reviews.sql

Сделать:
1. Миграция A (manual sources, раздел 6.1 ТЗ):
   - draft_wine_sources: receipt_id/receipt_item_id -> nullable; unit_price -> nullable;
     + source_kind text NOT NULL DEFAULT 'receipt' CHECK in ('receipt','manual');
     + CHECK консистентности receipt-полей; + shop_address/shop_lat/shop_lng;
     + индекс (draft_wine_id, source_kind);
   - расширить допустимые source_type у draft_wines значением 'manual'
     (проверь, есть ли CHECK constraint вообще);
   - аудит и при необходимости фикс RLS политик draft_wine_sources под
     owner-insert/select manual-источников.
2. Миграция B (автодобавление manual-бутылок, раздел 6.2 ТЗ):
   - user_storage + nullable колонка source_draft_source_id uuid;
   - CREATE OR REPLACE add_approved_draft_submission_to_user_storage: добавить
     ветку source_kind='manual' (поля и идемпотентность по ТЗ);
   - receipt-ветка и её идемпотентность не должны измениться ни на йоту —
     сравни диффом со старым телом функции.
3. Миграция C (rate limits, раздел 6.5 ТЗ):
   - серверная проверка при insert manual draft (рекомендуется SECURITY DEFINER
     RPC create_manual_draft_wine(...) который атомарно: проверяет лимиты
     (5 активных / 10 в сутки), создаёт draft + source + action и возвращает id;
     либо BEFORE INSERT триггер — выбери и обоснуй);
   - admin/moderator исключены (используй существующий механизм проверки роли,
     посмотри can_bypass_catalog_governance).
4. Индекс wines.barcode, если отсутствует (проверь через live audit по SSH).
5. Dart-слой: lib/features/add_bottle/data/manual_draft_repository.dart с методами
   createManualDraftWine / createManualDraftSource (или обёртка над RPC из п.3) +
   domain-модели. Без UI.
6. Вынести из receipt_controller.dart используемые хелперы
   (parseReceiptWineHints-обвязка, normalize-функции, inference name/winery/country)
   в shared-модуль lib/features/wines/application/wine_text_hints.dart.
   Только перенос + реэкспорт, поведение не менять, receipt_controller должен
   компилироваться и работать как раньше.

Проверка:
- flutter analyze по затронутым файлам;
- SQL миграций прогнать локально/на staging, если доступно; на live — только
  backup -> dry-run -> apply, и СНАЧАЛА показать мне SQL на review;
- юнит-тест на идемпотентность расширенного RPC, если есть тестовая инфраструктура
  для SQL — иначе зафиксируй ручной сценарий проверки в отчёте;
- в отчёте: список новых objects, какие constraints что блокируют, что осталось
  на проход 1.
```

### 15.3. Проход 1: Feature Skeleton + Путь «По Названию» + Погребок

```text
Задача прохода 1: рабочий вертикальный срез без камеры: хаб -> поиск по названию ->
найдено(в погребок)/не найдено(manual draft -> модерация -> «На проверке» в погребке).

Сначала прочитай:
- docs/add_bottle_universal_flow_tz_2026_06_12.md (разделы 5.1, 5.2, 5.5, 5.6, 5.7, 6.6, 10)
- lib/core/router.dart
- lib/features/cellar/presentation/my_cellar_screen.dart
- lib/features/wines/presentation/draft_wine_details_screen.dart (readiness/suggestions виджеты)
- lib/features/search/presentation/search_results_screen.dart

Сделать:
1. Route /add-bottle (+query-параметры path и query) в router.dart, guard как у
   обычных buyer-маршрутов.
2. add_bottle_hub_screen: три карточки путей (barcode и label пока ведут на
   заглушку «в следующем проходе» НЕ показывать — просто не рендерить эти карточки
   до их проходов, либо рендерить и сразу вести в name_search; выбери чистый вариант
   и зафиксируй) + ссылка на /receipt-qr-scanner.
3. name_search_step: поле поиска + результаты (существующий поисковый репозиторий),
   выбор вина -> candidate_confirm_sheet.
4. candidate_confirm_sheet (раздел 5.5): карточка вина, параметры бутылки
   (количество/цена/винтаж/дата/место), вызов cellarController.addToStorage
   (+ при необходимости расширить RPC add_to_user_storage shop-полями — колонки
   в таблице уже есть), success -> /my-cellar?tab=1.
5. manual_draft_screen (раздел 5.6): предзаполнение из запроса, readiness-индикатор,
   блок «Возможно, это уже есть» (топ-3 через существующий suggestion-механизм),
   параметры бутылки -> manual source, CTA «Сохранить и отправить на проверку» ->
   createManualDraftWine + submitDraftForCatalogReview; обработай rate limit ошибку.
6. Погребок: группа «На проверке» на вкладке Хранение (раздел 5.7): данные из
   draft-провайдеров, карточки с бейджами статусов, тап -> DraftWineDetailsScreen,
   счётчики полки draft-бутылки не включают; реши судьбу _DraftsBanner на этой вкладке.
7. Пустой поиск: CTA «Добавить вино, которого нет в каталоге» ->
   /add-bottle?path=name&query=<запрос>.
8. l10n: все новые строки в app_ru.arb + app_en.arb.

Проверка:
- dart format + flutter analyze;
- ручной e2e: поиск несуществующего названия -> CTA -> черновик -> погребок
  «На проверке» -> approve через admin web (или прямым SQL на live по протоколу) ->
  карточка превратилась в каноническую бутылку;
- регрессия: receipt flow создания черновика из чека работает как раньше;
- скриншоты погребка с группой «На проверке» (пустое и заполненное состояния).
```

### 15.4. Проход 2: Путь «Штрихкод»

```text
Задача прохода 2: barcode-вход — самый быстрый happy path (99.5% каталога с barcode).

Сначала прочитай:
- docs/add_bottle_universal_flow_tz_2026_06_12.md (разделы 5.3, 6.4)
- lib/features/wines/presentation/receipt_qr_scanner_screen.dart
  (permission UX: denied/permanently denied/unavailable — переиспользовать подход)

Сделать:
1. barcode_scan_step на mobile_scanner (формат EAN-13/EAN-8/UPC; QR игнорировать
   с подсказкой «Это QR чека — отсканируйте его в разделе чеков» + кнопка перехода).
2. Exact match по wines.barcode (нормализация: trim/только цифры). Один кандидат ->
   candidate_confirm_sheet; несколько -> список; ноль -> manual_draft_screen
   с предзаполненным barcode_draft и подсказкой сфотографировать этикетку.
3. Fallback-действия на экране: ввести штрихкод вручную, перейти к этикетке,
   выбрать фото из галереи.
4. Карточка пути «Штрихкод» появляется в хабе.
5. Аналитические события пути (раздел 9 ТЗ), если слой событий уже заведён
   в проходе 1 — иначе перенести оба в проход 5 и отметить в ТЗ.

Проверка:
- dart format + flutter analyze;
- реальный Android-прогон: штрихкод вина из каталога (возьми любой barcode
  селектом с live DB) и несуществующий штрихкод;
- permission denied / permanently denied сценарии;
- скриншоты сканера и confirm sheet.
```

### 15.5. Проход 3: Путь «Этикетка» + Новый Матчинг

```text
Задача прохода 3: label-вход с нормальным матчингом и удаление прототипа.

Сначала прочитай:
- docs/add_bottle_universal_flow_tz_2026_06_12.md (разделы 5.4, 7)
- lib/features/wines/application/wine_label_search_controller.dart (текущая логика)
- lib/services/wine_label_text_processor.dart
- lib/services/yandex_ocr_service.dart
- lib/features/wines/application/wine_text_hints.dart (из прохода 0)
- lib/features/wines/presentation/draft_wine_details_screen.dart
  (suggestion ranking/explanations/confidence strip — образец и источник виджетов)

Сделать:
1. label_scan_step: камера/галерея -> OCR через ocrServiceProvider; сырой текст
   пользователю не показывать; кириллица: при ML Kit предупредить про деградацию,
   при наличии Yandex OCR использовать его (раздел 5.4 ТЗ).
2. bottle_match_controller: конвейер из раздела 7 ТЗ — нормализация receipt-matcher
   подходом, извлечение полей (WineLabelTextProcessor + hints), поиск по приоритету:
   barcode-in-image -> wine_aliases -> wines.name+winery -> winery_aliases ->
   fuzzy; explainable ranking (score + главный сигнал).
3. UI кандидатов: топ-3 карточки с сигналом совпадения и confidence; пороги
   high/middle/low из раздела 7; «Это не оно» -> manual_draft_screen с фото
   и prefill из OCR.
4. В ветке not-found фото автоматически грузится в draft_wine_photos и становится
   photo_url черновика; ошибка upload не теряет форму (раздел 10).
5. Старый экран: route /wine-label-ocr -> redirect на /add-bottle?path=label;
   wine_label_ocr_screen.dart удалить; wine_label_search_controller.dart либо
   удалить, либо свести к новому матчингу — без мёртвого кода.
6. Карточка пути «Этикетка» в хабе.

Проверка:
- dart format + flutter analyze;
- ручной прогон на 10 реальных бутылках (микс кириллица/латиница, из каталога и нет);
  зафиксируй в отчёте: сколько в топ-1, сколько в топ-3, сколько not-found;
- юнит-тесты на ранжирование (фикстуры OCR-текстов -> ожидаемый порядок кандидатов);
- скриншоты каждого состояния пути.
```

### 15.6. Проход 4: Модерация Manual-Заявок

```text
Задача прохода 4: модераторский контур принимает manual-заявки как первоклассные.

Сначала прочитай:
- docs/add_bottle_universal_flow_tz_2026_06_12.md (раздел 8)
- docs/catalog_normalization_moderation_tz_2026_05_11.md (evidence panel, finalize)
- lib/features/admin/presentation/admin_draft_catalog_submissions_screen.dart
- lib/features/admin/presentation/admin_draft_catalog_submission_details_screen.dart
- workbench-экран каталожной нормализации (найди по route
  /admin/draft-submissions/:id/catalog-resolution)

Сделать:
1. Evidence panel: для source_kind='manual' скрыть receipt-блоки, показать
   «Источник: добавлено вручную (штрихкод|этикетка|поиск)» из
   draft_wine_actions.metadata.entry_method + фото крупно + параметры бутылки
   (цена/место/дата, если заполнены).
2. Очередь: бейдж «Чек»/«Вручную», фильтр по источнику.
3. _buildDraftCatalogSubmissionSnapshot: убедиться, что NULL receipt-поля не ломают
   snapshot; добавить entry_method.
4. Проверить (и починить, если нужно): admin_finalize_catalog_normalization_decision
   и триггер handle_approved_draft_submission_storage корректно отрабатывают
   manual-заявку end-to-end: approve -> user_storage строка с данными из manual
   source -> уведомление пользователю -> XP.
5. Email/in-app тексты уведомлений: не ссылаются на чек для manual-заявок.

Проверка:
- dart format + flutter analyze;
- e2e через https://admin.winepool.ru (или локальный admin build): manual-заявка
  каждого из маршрутов — approve-create, approve-link, request-changes -> resubmit,
  reject;
- после approve проверить на live DB (read-only через SSH): строка user_storage,
  идемпотентность повторного вызова RPC;
- регрессия receipt-заявки: открыть и одобрить одну receipt-заявку, убедиться
  что ничего не изменилось.
```

### 15.7. Проход 5: Guest Mode, Аналитика, Polish, QA, Doc Sync

```text
Задача прохода 5: довести поставку до release gate.

Сначала прочитай:
- docs/add_bottle_universal_flow_tz_2026_06_12.md (разделы 5.8, 9, 10, 11, 12, 14)
- lib/features/auth/application/guest_action_continuity_controller.dart
- lib/features/home/presentation/buyer_home_screen.dart (обработка pending actions)
- docs/tz_guest_mode_retention_2026_04_25.md (принципы стен)

Сделать:
1. Guest mode: сканирование/матчинг доступны гостю; стена на сохранении:
   - найдено -> существующий PendingGuestActionType.addToStorage;
   - manual draft -> новый PendingGuestActionType.addBottleDraft c payload полей
     формы + локальный путь фото; continuity после регистрации доигрывает создание
     draft + submission.
2. Точка входа «Добавить вино» на главной (рядом со «Сканировать чек», не вместо).
3. Аналитические события из раздела 9 (если слой событий не появился раньше —
   реализовать минимальный и обновить ТЗ решением).
4. UX-полировка по разделу 10: confirm-leave, двойные тапы, офлайн-сценарии,
   тексты ошибок.
5. Прогнать полный QA-план из раздела 12, зафиксировать результаты чек-листом
   в ТЗ (раздел 11 — проставить [x]).
6. Doc sync по разделу 14 (documentation_map, administrator_guide, QA-план,
   changelog, SELF_DEBUG_GUIDE про устаревший readonly URL).

Проверка:
- dart format + flutter analyze по всему затронутому;
- релевантные flutter test;
- финальный отчёт: что изменилось, какие файлы/миграции, известные риски,
  что предложить в «Что нового» RuStore.
```

---

## 16. Открытые Вопросы (Решить До Или Во Время Прохода 1)

1. Хаб как отдельный экран или bottom sheet поверх текущего экрана? (Рекомендация: отдельный экран — глубже ощущение «основного» действия, проще deep links.)
2. Показывать ли в группе `На проверке` старые receipt-драфты пользователя (сейчас за баннером)? (Рекомендация: да, унифицировать — у них тоже есть бутылочный интент и автодобавление.)
3. `unit_price` в `draft_wine_sources` — nullable или DEFAULT 0? (Рекомендация: nullable; нулевые цены загрязняют будущую чековую статистику.)
4. Нужен ли отдельный `PendingGuestActionType.addBottleDraft` или достаточно сериализовать форму в существующий механизм? (Решить по факту структуры payload.)
5. Слой аналитических событий: есть ли он уже, или заводим минимальную таблицу? (Аудит в проходе 1.)
