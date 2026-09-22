# Карта переиспользования

Что в проекте **уже есть**. Открывается до написания кода — правило 1 в
[`CLAUDE.md`](../CLAUDE.md).

Столбец «Параметры» важнее остальных: он показывает, как приспособить
механизм под новый сценарий, не копируя его.

Последнее обновление: 13.09.2026

---

## Поиск и выбор вина

| Механизм | Файл | Что делает | Параметры |
|---|---|---|---|
| **Фасад поиска** | `features/add_bottle/application/catalog_fast_search_service.dart` | Три канала: текст, штрихкод, этикетка. Ранжирование — чековым матчером. Возвращает исход: точное, кандидаты, конфликт, ничего | `CatalogSearchInput.text/barcode/label` |
| **Выбиратель вина** | `features/wines/presentation/widgets/wine_catalog_picker.dart` | Готовый диалог поверх фасада: антидребезг, отсечение устаревших ответов, аналитика, ветка «в каталоге нет» | `actionLabel`, `actionIcon`, `emptyActionLabel`, `onEmptyAction`, `initialQuery` |
| **Карточка кандидата** | `features/add_bottle/presentation/catalog_search_candidate_card.dart` | Объясняет, почему вино предложено: сигналы, уверенность, выпуск, объём | `compact`, `primaryActionLabel`, `primaryActionIcon`, `showOpenWineAction`, `discoveryMode` |
| **Сканер штрихкода** | `features/wines/presentation/widgets/bottle_barcode_scanner_dialog.dart` | Диалог камеры, работает и в вебе | — |
| **Поиск по кодам** | `wines_repository.searchWinesByProductCode` | Конфликт-осознанный: не схлопывает код, относящийся к нескольким винам | — |

**Никогда** не звать `searchWinesFuzzy` напрямую из UI: это кирпич внутри
матчера, без ранжирования и объяснений.

## Каталог вина

| Механизм | Файл | Что делает |
|---|---|---|
| Менеджер выпусков | `features/wines/presentation/widgets/admin_wine_release_manager.dart` | Выпуски, коды, фото, **тумблеры объёмов** |
| Форма выпуска | `features/wines/presentation/widgets/wine_release_dialog.dart` → `showWineReleaseDialog` | Год или NV, название, ABV, статус. Одна на карточку вина и на форму предложения |
| Объёмы вина для карточки | `wines_repository.fetchWineVolumeOptions` → `get_wine_volume_options` | Позиции выпуска с ценой «от» и наличием |
| Селектор объёма | `features/wines/presentation/wine_details/wine_volume_selector.dart` | Премиальные карточки объёма в карточке вина |
| Формат объёма | `features/wines/domain/wine_volume_option.dart` → `formatBottleVolume` | «0,75 л», «1,5 л». Второй копии не заводить |
| Знания вина | `features/admin/presentation/widgets/admin_wine_knowledge_editor.dart` | Ароматы, сочетания, методы — через Atlas |
| География винодельни | `features/wines/presentation/winery_geography_line.dart` → `wineryGeographyLine`, `wineryGeographyParts`, `wineryRegionLabel` | «Страна · Регион»; у мультирегиональной — «Несколько стран» / «Страна · Несколько регионов». Своих склеек `countryCode, regionName` не писать |
| Хозяйство-производитель вина | `wine_production_partners`; `wines_repository.fetchWineProductionWinery`, `setWineProductionWinery`, `fetchPublicWineCountByProductionWinery` | Бренд — винодельня вина, хозяйство — необязательная строка (пока одна на вино). Запись только RPC `admin_set_wine_production_winery`. Ключ таблицы — только `id`, иначе ломается `wineries(*)` у 1.1.0 |
| Поле и поиск хозяйства | `features/wines/presentation/widgets/wine_production_winery_field.dart` → `WineProductionWineryField`, `showWinerySearchDialog` | Поиск винодельни поверх `searchWineriesFuzzyScored`: `excludeWineryIds`, `onlySingleLocation`. Второй поиск виноделен не писать |
| Связи бренда и хозяйства | RPC `get_winery_production_links` → `wineryProductionLinksProvider`, `winesProducedByWineryProvider` | Хозяйства бренда и бренды хозяйства с числом публичных вин. Готовая функция для Tourism и публичных страниц — логику не копировать |
| Публичная страница винодельни | `features/winery_tourism/presentation/winery_page/public_winery_page_view.dart` → `PublicWineryPageView` | Одна страница и для сайта, и для предпросмотра редактора. `controller.scrollTo(block)`, `highlightedBlock`, `onBlockTap`, `tourLocation`; порядок блоков — по `presentation_type`. Второй рендер страницы не заводить |
| Атмосферы страницы винодельни | `winery_page/winery_page_palette.dart` → `WineryPagePalette.byKey`, `WineryPageTheme.wrap` | Восемь пресетов `visual_preset`, семантические роли цвета. Виджеты страницы берут цвет только через `context.winePalette` / `WineryPagePalette.of`; вне страницы — Terroir Dark |
| Элементы редактора страницы | `winery_profile_editor/winery_editor_kit.dart` → `EdTokens`, `EdTextField`, `EdButton`, `EdOptionCard`, `EdToggle`, `EdSegmented`, `EdCheck` (галочка с подсветкой, пока не отмечена), `EdAttentionMark`, `EdAddDashedButton` | Оболочка редактора всегда Terroir Dark. Новый раздел редактора собирается из них, а не из стандартных полей Material |
| Форматированный текст редактора | `winery_profile_editor/winery_editor_rich_text.dart` → `EdRichTextField` + `WineryStructuredRichText` | Панель прототипа (абзац/подзаголовок, жирный, курсив, подчёркнутый, списки, https-ссылка) поверх Quill; хранится только structured rich text v1. Для людей и событий — он же |
| Ценности винодельни | `winery_page/winery_value_catalog.dart` → `WineryValueDefinition` | Закрытый список из 12: название, иконка, стандартная подпись. Страница и редактор читают один справочник |
| Практическая информация винодельни | RPC `admin_replace_winery_profile_practical_facts_v1`, `WineryPracticalFact.keys` | Одна строка на ключ (адрес, часы, парковка, доступность, дорога, перед поездкой). Сайт и соцсети сервер отклоняет. На страницу попадает только видимое и проверенное |
| Фото профиля в слот | `winery_tourism_repository.uploadTourismMediaAsset(presentationRole:, altText:, rightsConfirmed:)`, `updateMediaFocalPoint` | `hero`/`feature` заменяют прежнее фото слота; центр кадра хранится в `focal_x/focal_y` и применяется обложкой страницы |
| Команда и место винодельни | RPC `admin_replace_winery_profile_people_v1`, `admin_save_winery_profile_place_v1`; `WineryProfilePerson`, `WineryProfilePlace` | До 10 человек (первый — крупно), фото `highlight`; текст места — раздел `terroir`, характеристики в `content.traits`, до трёх фото `scenic`. Загрузка — `uploadWineryProfileMedia` |
| Ключевые вина страницы винодельни | RPC `admin_replace_winery_profile_featured_wines_v1`; `WineryFeaturedWine`; кандидаты — `wineryPageWineCandidatesProvider` | До 6 своих вин и сделанных для брендов, одно главное (на странице первым и крупно), своё фото `highlight` только с галочкой прав. Пометка «почему это вино» внутренняя, не публикуется |
| События винодельни | таблица `winery_events`; RPC `admin_get_winery_events_v1`, `admin_create_winery_event_v1`, `admin_save_winery_event_v1`, `admin_set_winery_event_status_v1` (publish/restore/cancel/delete), публичные `get_public_winery_event_v1`, `get_public_winery_events_v1`; `WineryEvent`, `PublicWineryEventView`, `WineryUpcomingEvents` | Своя страница `/events/{slug}`, адрес строится из названия при первой публикации. Готовность — `winery_event_problems_v1` и `WineryEvent.errors` с одними правилами. Фото — `tourism_media_assets` с владельцем `winery_event`, галочка прав у каждого фото |
| Даты проведения события | `winery_event_occurrences`; `WineryEventOccurrence`; границы списка пересчитывает `sync_winery_event_dates_v1` | Один день — одна запись, два сеанса — две. Заявка ссылается на сеанс; при удалении даты заявка остаётся, теряя только ссылку |
| Галерея винодельни | RPC `admin_replace_winery_profile_gallery_v1`; `WineryProfileGalleryPhoto`; раздел `winery_editor_gallery_section.dart` | Фотографии с ролью `gallery` у профиля. Порядок, подпись, центр кадра и права; без прав фотография не показывается |
| Галерея страницы | `TourismEditorialGallery` | Большой кадр целиком по высоте рамки (вертикальные бутылки не обрезаются), плавная смена, лента миниатюр прокручивается, когда выделение доходит до края |
| Заявка гостя | одна форма `TourismLeadRequestForm` с предметом заявки: тур каталога (`experience`) или событие винодельни (`event`); RPC `submit_tourism_lead` и `submit_winery_event_lead_v1` | У события дата уже назначена, поэтому выбора даты и доставки билета в форме нет. Заявка приходит в общий туристический кабинет, название события хранится в самой заявке |
| Письма партнёрского контура | таблица `partner_mailbox_emails`, триггеры на `tourism_leads` и `tourism_lead_events`, RPC `enqueue_unanswered_lead_partner_emails_v1`; отправка — `send_notification_emails.py`, адрес в `PARTNER_EMAIL` | Всё, что требует реакции партнёра, дублируется на общий адрес WinePool. Не подменяет личные письма участников: это отдельная очередь, как `admin_auth_event_emails` |
| Общие части публичной страницы винодельни | `WineryPageTopBar`, `WineryPageFooter`, `WineryPageWidth`, `WineryStructuredDocument`, `winerySectionStyle` в `public_winery_page_view.dart` | Для страниц, которые живут в атмосфере винодельни (событие). Не копировать шапку и подвал |
| Программы страницы винодельни | RPC `admin_replace_winery_profile_programs_v1`; `WineryProfileProgram`; кандидаты — Tourism CRM | Опубликованные программы винодельни выбираются и сортируются отдельно от CRM; в снимок страницы попадают только публичные поля, включая услуги и размер группы. Туры сторонних операторов — только чтение. |
| Подпись стиля вина | `wineStyleLabel(color, type, sugar)` в `wine_characteristics.dart`; разбор строк базы — `wineColorFromDb`, `wineTypeFromDb`, `wineSugarFromDb` | «Красное сухое», «Игристое брют». Для DTO вне модели `Wine` |
| Публичная сеть бренда и хозяйств | `get_public_winery_page_v1` → `production_partners`, `produces_for`; `PublicWineryProductionLinkV1` | Tourism получает готовую сеть только через RPC; `public_path` есть лишь у опубликованного профиля, иначе переход в каталог с фильтром по винодельне |
| Путь к странице винодельни | `features/wines/presentation/winery_routes.dart` → `wineryDetailsRoute` | Учитывает ветку продавца `/seller-home` |
| Кандидаты в хозяйства из исследования | `features/admin/application/research_identity_roles.dart` → `productionWineryCandidateNames` | Роль `production_winery` из поля отчёта `identity_roles`, самый свежий отчёт с ролями, порог уверенности |
| Перенос вина в другую винодельню | RPC `admin_move_wine_to_winery`, история `wine_winery_moves` | Не меняет географию вина, снимает чужую линейку, может оставить прежнюю винодельню хозяйством. Объединение дублей для одного вина не использовать |
| Мультирегиональная винодельня | `wineries.geography_scope` (`single`, `multi_region`, `multi_country`), триггер `validate_wine_geography` | Фиктивных стран и регионов в справочниках нет. Вину такой винодельни регион обязателен (`wine_geography_required`); при переводе винодельни вина забирают её прежнюю географию |

## Торговый контур

| Механизм | Файл / RPC | Что делает |
|---|---|---|
| Репозиторий розницы | `features/admin/data/admin_retail_repository.dart` | Бренды, точки, предложения, продавцы, поиск людей |
| Создание продавца | `admin_create_business` | Владелец по выбору, сразу подтверждён |
| Смена владельца | `admin_set_business_owner` | Прежний владелец остаётся менеджером (триггер) |
| Поиск пользователя | `admin_search_users` | По нику, имени, почте. Почта — из `auth.users`, только сервером |
| Точка продажи | `admin_save_location`, `admin_list_locations` | Экран принимает `?businessId=` |
| Предложения | `upsert_offer`, `admin_list_offers` | Экран принимает `?locationId=`; RPC берёт массив точек |
| Товарная позиция | `resolve_wine_sku` | Единственная точка записи, идемпотентна |
| Право публикации | `can_publish_for_business` | Подтверждённый бизнес + `is_business_manager` |
| Геокодер | edge `geocode-receipt` | Кэш, нечёткий поиск, `force` |
| Настройки продавца | `business_settings`, `get_business_settings` | Порог «мало», сроки протухания цены. Чтение подставляет умолчания |
| История предложений | `offer_state_history` | Смены цены и наличия, пишутся триггером. Основа для «обычно есть в наличии» |

## Права и роли

| Механизм | Где | Что |
|---|---|---|
| Возможности пользователя | `features/business/application/business_context_provider.dart` | `canModerateBusinesses`, `canManageOffers`, … |
| Проверки в мутациях | `features/business/application/mutation_policy_guards.dart` | `ensureCanModerateBusinesses` и родственные |
| Серверные флаги | `core/config/remote_feature_flags.dart` → `is_feature_enabled` | Выдача функции по ролям без пересборки |
| Участники бизнеса | `business_members`, `is_business_manager` | Заложено под кабинет продавца |

## Карточка вина (премиальный слой)

| Механизм | Файл |
|---|---|
| Токены стиля | `features/wines/presentation/wine_details/wine_card_tokens.dart` |
| Шапка с бутылкой и галереей | `wine_card_hero.dart` |
| Заголовок раздела | `wine_card_section_heading.dart` |
| «Где купить» со свёрткой по сети | `wine_purchase_places_section.dart` |

Цвета, шрифты, радиусы и длительности — только из токенов. Прямых
`Color(0x…)` в карточке быть не должно.

## Общее

| Механизм | Файл |
|---|---|
| Единая дизайн-система | `docs/winepool_design_system.md` → `core/theme/app_colors.dart`, `app_typography.dart`, `app_theme.dart` |
| Цена с валютой | `common/widgets/currency_text.dart` |
| Просмотр фото на весь экран | `common/widgets/fullscreen_network_image_viewer.dart` |
| Звонок по номеру из заявки | `core/utils/phone_dialer.dart` → `openPhoneDialer` |
| Вложения в диалоге с туристом | `tourism_lead.dart` → `TourismLeadAttachment`, RPC `operator_tourism_lead_message` |
| Роль и права в Tourism CRM | `winery_tourism_providers.dart` → `tourismWorkspaceAccessProvider`, `TourismWorkspaceAccess.can(orgId, 'tour.edit')` |
| Показ тура на странице винодельни | Tourism CRM → тур → «Маршрут» → «Показ на страницах виноделен»; `tourism_experience_destinations.is_public_on_winery_page`, RPC `tourism_set_experience_winery_page_visibility_v1` |
| Переходы из тура оператора | Карточка в профиле винодельни получает `operator_slug` и `tour_slug` из `admin_get_winery_profile_authoring_v1`; ведёт на `/tourism/o/<оператор>` и `/tourism/o/<оператор>/t/<тур>` |
| Тексты обложки и посещения | `hero_description` — необязательный текст под слоганом обложки (до 500); `visit_summary` — описание в блоке «Посетить винодельню». Оба входят в immutable snapshot публикации |
| Обложка тура и очистка архива | Левая карточка CRM, публичная карточка и страница винодельни берут утверждённую обложку опубликованного тура; «Медиа» → архив → «Удалить навсегда» доступно только для материала без ссылок в публикациях и блоках |
| Отзывы о винах на странице винодельни | строки — `winery_wine_review_rows_v1(winery_id)`, имя автора для открытых страниц — `review_public_author_name_v1`; выбор — `winery_profile_featured_reviews`; блок `WineryWineReviewsBlock`, полный список — `showWineryWineReviewsPanel` |
| Карточка записи «Готовы приехать?» | `WineryVisitCard`, выбор главного варианта — `_visitCard` в `public_winery_page_view.dart` |
| QR-код на экране и печать карточки | `visit_review_qr.dart`: `BarcodeWidget` (пакет `barcode_widget`) на экране, PDF — `pdf` + `printing`, шрифт Lato из `assets/fonts` (встроенные шрифты PDF без кириллицы) |
| Отзывы о посещениях | ссылка/QR — `partner_get_visit_review_link_v1`, форма `/review/<токен>` — `TourismVisitReviewScreen`, блок на страницах — `VisitReviewsBlock`, модерация — «Отзывы → О посещениях» |
| Вопросы и ответы винодельни | `winery_profile_faq`, `admin_replace_winery_profile_faq_v1`; редактор — `winery_editor_faq_section.dart` (заготовки `wineryFaqPresets`); на странице — блок `WineryPageBlock.faq`, строка доверия в подвале — `wineryPageTrustLine` |
| Демонстрационная запись | `wineries.is_demo`, `tourism_organizations.is_demo` (дети — туры, события, вина — получают признак триггером; вино уходит в `visibility_scope = 'admin_only'` — новых значений видимости не заводить: установленное приложение их не разберёт). Прячет база: RLS на `wineries`, `wines`, `tourism_*` плюс фильтр в `list_public_tourism_catalog_v1`. Одиночные страницы открываются по прямой ссылке и показывают полосу `WineryDemoBanner` |
| Инициалы человека в кружке | `tourism_public_labels.dart` → `tourismPersonInitials` |
| Прокси картинок хранилища | `core/utils/storage_url.dart` → `proxyStorageUrl` |
| Аналитика воронки | `core/analytics/analytics.dart` |
| Оборот предложения через карту | `features/offers/domain/offer.dart` → `offerToDeepJson` |
| Локализация | `flutter gen-l10n`, ARB в `lib/l10n/` (ru, en, be, uz) |
| Push-уведомления (Android: RuStore + FCM) | приём — универсальная библиотека RuStore в `MainApplication`, показ — только `WinePoolNotificationPresenter.kt` (второго пути во Flutter нет); токены обоих каналов — мост `ru.winepool.app/push` → `PushTokenRegistrationService` → RPC `register_push_token_v1` (переназначает токен вошедшему), снятие при выходе — `unregisterCurrentToken()` в `AuthController.signOut`; отправка с сервера — `winepool_push.py` рядом с `send_notification_emails.py` (статус `app_notifications.push_status`, типы — триггер `app_notifications_mark_push_v1`); тестовая отправка — `scripts/send_universal_push.py`. Новый тип — по `docs/push_notifications_guide.md`: поля `link` (путь экрана) и `channel` (`service`/`news`) задаёт сервер, версия приложения не нужна |
| Предложить уведомления в момент пользы | `features/notifications/presentation/push_opt_in_prompt.dart` → `PushOptInPrompt.afterLeadSubmitted`; при запуске разрешение не спрашивать. Новый повод — новый метод с тем же запоминанием отказа |

## Организация партнёра: команда, пакет, письма

Всё серверное, применено на бой 07.09.2026. Экранов пока нет — RPC ждут UI.

| Механизм | RPC / таблица | Что делает | Кому доступно |
|---|---|---|---|
| Состав команды | `tourism_list_members(org)` | Участники с почтой и именем, открытые приглашения, занято мест из скольких, можно ли назначать менеджера | право `team.manage` |
| Добавить человека | `tourism_add_member(org, email, role, …)` | Есть аккаунт — добавляет и шлёт письмо; нет — заводит приглашение с токеном на 14 дней. Держит лимит мест и иерархию ролей | `team.manage` |
| Принять приглашение | `tourism_accept_invitation(token)` | Сверяет почту с аккаунтом, заводит участника | любой вошедший |
| Отозвать приглашение | `tourism_revoke_invitation(id)` | — | `team.manage` |
| Смена роли и снятие | `tourism_set_member_role`, `tourism_remove_member`, `tourism_set_member_flags` | Менеджер не трогает владельца и менеджеров; последнего владельца снять и понизить нельзя никому | `team.manage` |
| Состояние пакета | `tourism_package_state(org)` | Текущая подписка, история, все четыре пакета с составом, исключения, открытый запрос, расход лимитов | `team.manage` или каталог |
| Назначить пакет | `admin_set_tourism_package(org, code, starts, ends, is_pilot, note)` | Закрывает прежнюю подписку, открывает новую, приводит команду к лимиту, пишет владельцу | каталог |
| Попросить пакет | `tourism_request_package_change(org, code, period, note)` | Заявка каталогу с письмом каждому администратору | `billing.manage` (владелец) |
| Исключения сверх пакета | `admin_set_tourism_entitlement_override`, `admin_remove_tourism_entitlement_override` | Точечная возможность на срок с причиной. `placement.featured` подключается только так | каталог |
| Приведение к лимиту мест | `tourism_apply_seat_limit(org, actor)` | Гасит лишних снизу вверх по старшинству, возвращает своих же при повышении, понижает менеджеров, если роли нет в пакете | внутренняя |
| Журнал организации | `tourism_organization_events` | Кто кого добавил, снял, понизил; смены пакета и исключений | владелец, менеджер, каталог |
| Письмо и колокольчик | `tourism_queue_notification(...)` → `app_notifications` | Одна строка вместо своей рассылки. Очередь разбирает `send-notification-emails` по таймеру | внутренняя |
| Письмо на почту без аккаунта | `tourism_organization_invitations` + третий источник в `send_notification_emails.py` | Единственный путь написать тому, у кого нет профиля | сервис |

Своей проверки прав в этих RPC не писать: всё идёт через `tourism_can` и
`tourism_assert_member_authority`. Лимит галереи держит триггер
`enforce_tourism_gallery_limit` на `tourism_media_assets` — не дублировать его
проверкой в клиенте.

---

## Как дополнять эту карту

Добавили механизм, который пригодится второй раз, — строка сюда **в том же
коммите**. Карта без обновления хуже отсутствующей: она врёт.
