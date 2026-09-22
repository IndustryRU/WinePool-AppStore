# WinePool - ТЗ: Винодельни, Винный Туризм, Экскурсии И Бронирование

Дата: 01.06.2026

Статус: целевой post-MVP source of truth для направления `winery tourism`.

Обновление 02.06.2026: первый Yalta pilot slice начал реализовываться как легкий lead/request слой без оплаты и без slot engine. В production self-hosted Supabase заведены базовые tourism-таблицы, storage bucket `tourism-media`, один опубликованный маршрут `Дегустация вин в Массандре`, media assets, pickup point, тестовые vehicles и таблица `tourism_leads`. В приложении включен feature flag `AppFlags.wineryTourismEnabled = true` для внутренней проверки `/tourism`, карточки тура, формы `Оставить заявку` и админского просмотра входящих заявок на `/admin/tourism`.

## 1. Контекст

WinePool уже выпустил MVP как винный дневник, каталог, чековый flow, личную карту покупок, публичные отзывы и партнерскую винную ленту. Следующий логичный слой - аккуратно развить направление виноделен и винного туризма, не смешивая его с отложенным старым commerce-контуром.

Старый commerce не отменяется и не признается ошибочным. Это отдельный стратегический слой WinePool - каталог, офферы, корзина, checkout, заказы и seller-flow, - который ждет своего часа, когда в России сложатся благоприятные юридические, политические и экономические условия для безопасного запуска. Tourism-направление должно развиваться параллельно как discovery/lead/request слой, не оживляя прямую продажу алкоголя внутри приложения раньше времени.

- публичные карточки виноделен;
- premium winery wiki / brand passport pages как лицо винодельни внутри WinePool;
- туристические точки виноделен и дегустационные залы;
- экскурсии, дегустации, винные маршруты;
- расписания, заявки и бронирование;
- route/discovery слой на карте;
- кабинет винодельни или оператора;
- административная модерация и governance;
- партнерская лидогенерация без превращения WinePool в алкомаркет.

Важное стратегическое уточнение: страница винодельни в WinePool - это не техническая карточка и не сухой справочник. Для сильных партнеров это должно стать лицом бренда в приложении: качественная редакционная страница, первый слой WinePool Wiki, где можно показать историю, людей, терруар, архитектуру, стиль, вина, маршруты и уникальную изюминку хозяйства. Виноделен не так много, поэтому у WinePool должна быть возможность делать каждую страницу вручную, внимательно и дорого, если партнер готов вкладываться.

Это направление было ранее зафиксировано в:

- `WINERIES_SPECIFICATION.md` - "Паспорт винодельни", карточки экскурсий, слоты, бронирование, QR check-in.
- `docs/SELLERS_ARCHITECTURE_SPECS.md` - `businesses`, `locations`, `is_tourism_active`, "винный туризм".
- `docs/mvp_launch_strategy_2026_04_23.md` - winery layer сознательно выведен из MVP в post-MVP.
- `docs/mvp_launch_execution_plan_2026_04_23.md` - winery layer как связка `винодельня -> вина -> атлас -> покупки/дегустации -> партнерский статус`.
- `docs/map_experience_roadmap_2026_04_25.md` - travel/wine tourism и "винные маршруты".
- `docs/winery_partnership_pitch_2026_04_24.md` - WinePool как discovery/reference traffic, а не продавец алкоголя.
- `ANALYTICS_RECOMMENDATIONS.md` - лидогенерация для виноделен через "Забронировать дегустацию".

## 2. Главная продуктовая рамка

WinePool не должен становиться приложением "купи алкоголь". Новый слой должен быть описан пользователю как:

> Винный дневник и справочник, который помогает находить винодельни, изучать их историю, смотреть доступные экскурсии и оставлять заявку на визит.

То есть:

- можно показать карточку винодельни;
- можно показать адрес, телефон, сайт, мессенджер и маршрут;
- можно показать информационную карточку экскурсии;
- можно дать пользователю оставить заявку на визит или перейти на внешний канал партнера;
- можно учитывать бронирование услуги посещения, если legal review подтвердит схему;
- нельзя превращать карточку в рекламу покупки алкоголя;
- нельзя делать CTA вида `Купить вино`, `Заказать алкоголь`, `Доставка вина`.

## 3. Юридическая рамка

Этот раздел не заменяет юридическую консультацию. Перед платежами, публичными партнерскими размещениями и широким запуском tourism-модуля нужен отдельный legal review.

Базовые ограничения, которые должны учитываться в архитектуре:

- Дистанционная розничная продажа алкогольной продукции физлицам в РФ является юридически чувствительной зоной. Поэтому WinePool не должен принимать оплату за алкоголь, доставку алкоголя или оформлять заказ алкогольной продукции внутри приложения.
- Реклама алкогольной продукции в интернете ограничена. Карточки вин, виноделен и туров должны быть информационно-справочными, без прямого побуждения к покупке или употреблению.
- Age gate 18+ остается обязательным до показа пользовательских сценариев, связанных с вином и дегустациями.
- Тексты, фотографии, баннеры, пуши и стор-листинги проходят moderation/legal copy check: без "лучшее", "скидка на вино", "купи", "пей", образов употребления несовершеннолетними, обещаний пользы для здоровья.
- Если в приложении появляется оплата экскурсии/дегустации как услуги, это отдельный этап после legal review. До этого MVP tourism-layer работает как lead/request/external booking.

Ссылки для legal-checkpoint:

- ФЗ-171, регулирование оборота алкогольной продукции: `http://www.consultant.ru/document/cons_doc_LAW_8368/`
- ФЗ-38 "О рекламе", статья 21: `http://www.consultant.ru/document/cons_doc_LAW_58968/`
- ФАС, разъяснения и практика по рекламе алкоголя: `https://fas.gov.ru/`
- Официальный портал правовой информации для сверки актуальных редакций: `http://pravo.gov.ru/`

## 4. Цели

### 4.1. Для пользователя

Пользователь должен:

- открыть винодельню из карточки вина, атласа, карты или поиска;
- понять, где находится винодельня и что у нее можно посетить;
- увидеть доступные экскурсии/дегустации/маршруты;
- построить маршрут до винодельни или точки сбора;
- оставить заявку на визит;
- сохранить винодельню или тур в избранное;
- получить напоминание о подтвержденном визите;
- после визита отметить check-in, дегустацию или отзыв.

### 4.2. Для винодельни

Винодельня должна:

- иметь публичную карточку бренда;
- показать историю, фото, контакты, локации, список вин;
- указать туристические точки: усадьба, дегустационный зал, магазин при винодельне;
- публиковать экскурсии и дегустации;
- управлять расписанием, вместимостью, ограничениями;
- принимать заявки и подтверждать/отклонять их;
- видеть базовую аналитику интереса.

### 4.3. Для туроператора / точки продаж экскурсий

В реальной Ялте и Крыму часто работает не только сама винодельня, но и туристический оператор или офлайн-точка продаж. Поэтому система должна поддерживать:

- экскурсии, где destination - винодельня, а provider - туроператор;
- экскурсионное бюро, у которого есть несколько направлений, включая винные и не винные;
- блок `Другие экскурсии оператора`, чтобы не отсекать основную программу партнера, но не размывать wine-first фокус WinePool;
- точку сбора, отличную от адреса винодельни;
- несколько возможных точек посадки по одному маршруту, если автобус/минивэн собирает туристов в разных местах города;
- понятную навигацию к точке посадки для туриста, который плохо ориентируется на курорте;
- фото-ориентиры точки посадки: фасад, перекресток, ближайший магазин/кафе/остановка, вид со стороны движения;
- текстовую инструкцию "как найти место посадки" без ручной пересылки фото и объяснений через мессенджер;
- маршрут с несколькими остановками;
- referral source: QR/flyer/промокод точки продаж;
- ручное создание заявки оператором в будущем;
- статистику, сколько заявок пришло с конкретной точки/QR.

При этом общий публичный раздел WinePool должен оставаться wine-first. В `/tourism` по умолчанию показываются винные, дегустационные, гастрономические и около-винные маршруты. Не винные экскурсии оператора допускаются как вторичный inventory-блок на странице оператора или в контексте уже выбранного бюро, но не должны превращать WinePool в универсальный агрегатор всех экскурсий.

Внешний вход на главной странице приложения называется `WinePool туризм`, а не `Винный туризм`: это дает пространство для вторичных не винных экскурсий операторов, при этом внутри `/tourism` default filter остается wine-first. Карточка входа на `/buyer-home` должна быть заметной и иметь понятный CTA `Подобрать тур`.

### 4.4. Для администратора WinePool

Администратор должен:

- создавать и чистить canonical winery card;
- проверять заявки бизнеса на тип `winery` или `tourism_operator`;
- привязывать business к canonical winery через governance;
- модерировать публичные тексты, фото, маршруты, расписания;
- управлять партнерским статусом;
- останавливать публикацию спорного тура;
- видеть жалобы, отмены, no-show и спорные случаи.

## 5. Non-goals первого среза

В первый релиз направления не входят:

- внутренняя продажа алкоголя;
- доставка алкоголя;
- полноценный marketplace билетов с оплатой внутри приложения;
- динамическое ценообразование;
- сложный seat map;
- собственная навигация turn-by-turn;
- live tracking автобуса;
- публичные рейтинги туроператоров без модерации;
- автоматическая публикация непроверенных данных винодельни.

## 6. Роли и права

### 6.1. Buyer / guest

Гость:

- видит публичные карточки виноделен и туров после age gate;
- видит карту и route CTA;
- может открыть внешний сайт/телефон/мессенджер;
- в первом lead-pilot может оставить заявку без аккаунта, если указал имя и контакт;
- после отправки заявки получает мягкий registration CTA: регистрация дает уведомления, `Мои заявки`, детали поездки, изменения машины/точки сбора и offline-билет.

Авторизованный buyer:

- оставляет заявку с предзаполнением известных данных профиля;
- видит свои tourism-заявки и историю статусов;
- в будущей фазе может отменять заявку/booking в допустимое время;
- получает уведомления;
- добавляет винодельню/тур в избранное;
- после визита может оставить отзыв/дегустацию.

### 6.2. Business owner

Business owner с `businesses.type = winery`:

- управляет своим business profile;
- управляет locations;
- предлагает изменения canonical winery profile через moderated proposal;
- создает tourism experiences, schedules, slots;
- обрабатывает заявки по своим experiences;
- видит аналитику только по своим объектам.

Business owner с `businesses.type = tourism_operator`:

- управляет экскурсиями, где он является оператором;
- указывает destination wineries/stops;
- обрабатывает заявки по своим маршрутам;
- не редактирует canonical winery без модерации.

### 6.3. Admin / moderator

Admin:

- видит все tourism entities;
- может править canonical data;
- может approve/reject/pause tourism content;
- может bind/unbind business к winery;
- может включать partner/verified;
- может видеть audit log.

Moderator:

- видит очередь tourism submissions;
- может approve/reject content;
- не может менять финансовые/partner flags без admin capability.

## 7. Архитектурный принцип

> **Состояние на 07.09.2026.** Принцип «не создавать второй каталог виноделен»
> в силе, но цепочка ниже (`businesses.managed_winery_id` → `locations` → туры)
> за три месяца не задействована ни разу: туры пошли через
> `tourism_organizations`. Что построено на самом деле и что с этим делать —
> [`winery_unified_model_2026_09_07.md`](winery_unified_model_2026_09_07.md).


Не создавать второй независимый каталог виноделен.

Текущая система уже имеет:

- `wineries` как canonical catalog entity;
- `businesses` как владеемая бизнес-сущность;
- `locations` как физические точки;
- `businesses.managed_winery_id` как governance binding;
- admin moderation для business/winery binding;
- map layer на Yandex MapKit;
- guest mode, age gate, registration walls;
- notifications baseline.

Tourism-layer должен расширять именно это:

```text
wineries
  canonical winery truth
  |
  | 1:0..1 via governance
  v
businesses(type=winery)
  owner-managed business profile
  |
  | 1:n
  v
locations(type=wineryEstate/tastingRoom, is_tourism_active=true)
  physical tourism points
  |
  | 1:n
  v
tourism_experiences
  экскурсии, дегустации, маршруты
  |
  | 1:n
  v
schedules / slots / bookings
```

Для туроператора:

```text
businesses(type=tourism_operator)
  |
  v
tourism_experiences(provider_business_id)
  |
  v
tourism_experience_stops(destination_winery_id/location_id)
```

## 8. Доменные сущности

### 8.1. Winery public profile

Canonical `wineries` остается source of truth для названия, страны, региона, описания, логотипа и базовых полей. Для публичного tourism-слоя нужны дополнительные publish fields.

Рекомендуется не перегружать `wineries`, а добавить 1:1 таблицу:

`winery_public_profiles`

- `winery_id`
- `slug`
- `headline`
- `story_text`
- `visit_summary`
- `winemaker_name`
- `winemaker_photo_url`
- `hero_image_url`
- `hero_video_url`
- `gallery_status`
- `publication_status`: `draft`, `submitted`, `published`, `paused`, `archived`
- `published_at`
- `published_by`
- `updated_at`

Почему отдельная таблица:

- canonical winery остается каталожной сущностью;
- tourism/storytelling copy можно модерировать отдельно;
- проще откатить публичную публикацию без ломки каталога;
- self-service business proposals не получают прямого write в canonical `wineries`.

### 8.1.1. Winery Wiki / Brand Passport strategy

Публичная страница винодельни должна проектироваться как первый этап WinePool Wiki:

- не просто "карточка производителя";
- не просто SEO-страница;
- не рекламный лендинг с кнопкой покупки;
- а редакционная, живая, качественно оформленная страница о хозяйстве.

Цель:

- показать лицо винодельни;
- объяснить, чем она отличается от соседей;
- связать историю, людей, землю, стиль вин и пользовательский опыт;
- дать пользователю доверительный вход в бренд;
- дать партнеру страницу, которую не стыдно отправить клиенту, туристу, журналисту или дистрибьютору.

Так как виноделен относительно немного, WinePool должен поддерживать не только шаблонный профиль, но и ручной editorial layout.

### 8.1.2. Уровни глубины страницы винодельни

`Basic`

- название;
- регион/страна;
- краткое описание;
- список вин;
- базовые контакты/сайт, если публично разрешены;
- один логотип/изображение;
- статус: справочная страница.

`Verified`

- подтвержденный business owner;
- актуальные контакты;
- locations;
- публичная история;
- фотогалерея;
- tourism blocks;
- отметка `Проверено`.

`Editorial`

- WinePool пишет и вычитывает материал;
- история хозяйства;
- основатель/винодел/команда;
- терруар и сорта;
- стиль винодельни;
- 5-15 качественных фото;
- связки с Атласом;
- подборка ключевых вин;
- tourism/visit блоки;
- редакционная подпись или дата обновления.

`Partner Premium`

- ручная кураторская верстка;
- уникальные секции под конкретную винодельню;
- расширенная фотогалерея/видео;
- заметные, но аккуратные partner markers;
- приоритет в curated подборках, если legal/copy review позволяет;
- расширенная аналитика просмотров и действий;
- отдельный редакционный пакет от WinePool.

Важно: premium не означает "мы называем винодельню лучшей" и не дает права на агрессивную алкогольную рекламу. Premium означает глубину, качество оформления, актуальность данных, редакционную работу и расширенную витрину внутри справочного продукта.

### 8.1.3. Возможные блоки страницы

Страница должна собираться из модульных блоков:

- Hero: фото/видео, название, регион, короткая фраза.
- `История`: происхождение хозяйства, ключевые даты, люди.
- `Философия`: стиль, подход к виноделию, органика/биодинамика/эксперименты, если подтверждено.
- `Терруар`: почвы, климат, высоты, экспозиции, карта региона.
- `Виноградники`: участки, сорта, возраст лоз.
- `Винодел`: портрет, цитата, биография.
- `Архитектура и место`: усадьба, подвал, дегустационный зал, виды.
- `Ключевые вина`: curated список с пояснениями.
- `Что попробовать`: информационная подборка без прямого CTA купить.
- `Посетить`: экскурсии, дегустации, точки посадки, маршруты.
- `Материалы WinePool Атласа`: статьи региона/сорта/стиля.
- `Отзывы и дегустации пользователей`: после модерации и privacy review.
- `Где встречалось в чеках WinePool`: только обезличенно и по правилам community map.
- `Медиа`: галерея, видео, панорамы.
- `FAQ`: как добраться, сезонность, нужна ли запись, язык экскурсии, доступность.
- `Обновлено`: дата и источник обновления.

### 8.1.4. Редакционный пакет как продукт

WinePool должен иметь возможность продавать винодельне не "рекламу алкоголя", а редакционно-информационный пакет оформления страницы:

- интервью 30-90 минут;
- сбор фактуры;
- редакционный текст 800-2500 слов;
- подбор/обработка фото;
- факт-чек с представителем винодельни;
- оформление страницы в WinePool;
- публикация и дальнейшие обновления;
- базовая аналитика просмотров/переходов/заявок.

Возможные пакеты:

- `Profile Cleanup`: привести базовую страницу в порядок.
- `Editorial Passport`: полноценный материал о винодельне.
- `Tourism Ready`: страница + экскурсии + точки посадки + маршруты.
- `Premium Partner`: расширенная страница, галерея, аналитика, curated placements.

Юридическая формулировка в договорах и интерфейсе: информационно-справочное размещение / редакционное оформление профиля / сопровождение данных, а не реклама продажи алкогольной продукции.

### 8.1.5. Качество страницы

Definition of quality для хорошей страницы винодельни:

- понятно, где находится винодельня;
- понятно, почему она отличается;
- есть человеческое лицо: владелец, винодел, команда или история;
- есть качественные фото;
- нет пустых маркетинговых общих слов;
- нет непроверенных превосходных утверждений;
- вина связаны с каталогом;
- tourism-блоки имеют реальные адреса, расписания или понятный способ связи;
- страница выглядит как часть WinePool, но сохраняет индивидуальность винодельни;
- пользователь после прочтения понимает, хочет ли он сохранить винодельню, прочитать о регионе или поехать на экскурсию.

### 8.1.6. Данные для модульной страницы

Для поддержки уникальных premium-страниц добавить layer:

`winery_profile_sections`

- `id`
- `winery_id`
- `section_type`: `hero`, `story`, `terroir`, `winemaker`, `vineyards`, `wines`, `tourism`, `gallery`, `atlas_links`, `faq`, `custom`
- `title`
- `subtitle`
- `body_markdown`
- `media_asset_ids`
- `linked_wine_ids`
- `linked_article_ids`
- `display_style`: `standard`, `feature`, `quote`, `timeline`, `gallery`, `map`, `wine_carousel`, `custom`
- `sort_order`
- `publication_status`
- `updated_at`

`winery_profile_facts`

- `winery_id`
- `fact_key`
- `fact_value`
- `source`
- `verified_at`

Это позволит не хардкодить один шаблон для всех. Basic-страницы могут рендериться стандартно, premium/editorial - из набора секций.

### 8.2. Business

Существующий `businesses` расширить:

- добавить enum value `tourism_operator`;
- оставить `winery`, `retail`, `horeca`;
- `managed_winery_id` по-прежнему admin-only и только для `type = winery`;
- `is_partner`, `is_verified` сохраняются как admin governance flags.

### 8.3. Location

Существующие `locations` расширить полями:

- `title`
- `city`
- `country`
- `region_id`
- `timezone`
- `route_hint`
- `parking_hint`
- `meeting_point_hint`
- `pickup_instruction`
- `pickup_landmark_text`
- `pickup_side_of_street`
- `pickup_vehicle_hint`
- `pickup_contact_phone`
- `public_transport_hint`
- `opening_hours_json`
- `is_public`
- `moderation_status`
- `geocoded_at`
- `geocode_provider`
- `map_preview_image_url` optional.

`LocationType`:

- `wineryEstate`
- `tastingRoom`
- `retailShop`
- `warehouse`
- add `pickupPoint` if tourism operators need meeting points;
- add `tourOffice` if offline tourism sales points become business-owned locations.

Для `pickupPoint` обязательны не только координаты, но и human-readable ориентиры. В Крыму и курортных городах пользователь часто не знает местность, поэтому точка посадки должна быть самостоятельным объектом навигации:

- точный адрес или ближайший адрес;
- короткое название: `Набережная, у памятника Ленину`;
- инструкция: `Ждите автобус у остановки со стороны моря`;
- ориентиры рядом: магазин, кафе, перекресток, остановка, вывеска;
- фото точки и подхода к ней;
- контакт на случай, если турист не нашел место;
- время, когда нужно быть на точке до отправления.

### 8.3.1. Pickup point media

Точки посадки нуждаются в отдельной галерее, потому что обычное фото винодельни здесь бесполезно. Цель - заменить ручное объяснение продавца в мессенджере.

`tourism_pickup_point_media`

- `id`
- `location_id`
- `image_url`
- `caption`
- `view_type`: `main_view`, `approach_from_sea`, `approach_from_city`, `nearby_landmark`, `parking_spot`, `vehicle_stop`
- `sort_order`
- `moderation_status`
- `created_at`

Минимум для публикации pickup point:

- 1 основное фото точки;
- 1 фото ближайшего ориентира;
- текстовая инструкция;
- координаты;
- короткое название.

### 8.3.2. Route pickup points

Один маршрут может иметь несколько точек посадки. Пользователь в заявке/билете должен видеть именно свою выбранную или назначенную точку.

`tourism_experience_pickup_points`

- `id`
- `experience_id`
- `location_id`
- `order_index`
- `pickup_time_offset_minutes`
- `exact_pickup_time` nullable, если время фиксированное;
- `is_default`
- `is_active`

Если маршрут поддерживает выбор точки посадки, booking должен хранить:

- `pickup_location_id`
- `pickup_time`
- `pickup_note_snapshot`

Snapshot нужен, чтобы изменение точки в будущем не ломало уже выданный билет.

### 8.4. Tourism experience

`tourism_experiences` - главный продуктовый объект:

- экскурсия по винодельне;
- дегустация;
- мастер-класс;
- маршрут с трансфером;
- винный ужин / pairing event, если legal copy позволяет;
- сезонная программа.

Поля:

- `id`
- `provider_business_id`
- `primary_winery_id`
- `primary_location_id`
- `title`
- `subtitle`
- `description`
- `experience_type`: `winery_tour`, `tasting`, `masterclass`, `route`, `festival`, `private_visit`
- `duration_minutes`
- `min_guests`
- `max_guests`
- `age_limit`: default `18`
- `language_codes`: `['ru']`, later `['ru','en']`
- `price_from`
- `price_currency`
- `price_note`
- `payment_mode`: `none`, `external`, `on_site`, `in_app_future`
- `booking_mode`: `external_url`, `lead_request`, `managed_slots`
- `external_booking_url`
- `contact_phone`
- `contact_messenger_url`
- `included_text`
- `not_included_text`
- `requirements_text`
- `cancellation_policy_text`
- `accessibility_text`
- `is_transport_included`
- `meeting_location_id`
- `status`: `draft`, `submitted`, `approved`, `published`, `paused`, `rejected`, `archived`
- `moderation_notes`
- `published_at`
- `created_by`
- `updated_at`

### 8.5. Experience stops

Нужны для маршрутов и операторских экскурсий.

`tourism_experience_stops`

- `id`
- `experience_id`
- `order_index`
- `winery_id` nullable
- `location_id` nullable
- `title`
- `description`
- `planned_duration_minutes`
- `arrival_offset_minutes`
- `latitude`
- `longitude`
- `address_text`

Пример: Ялта -> Массандра -> дегустационный зал -> смотровая/обед -> Ялта.

### 8.6. Media

`tourism_media_assets`

- `id`
- `owner_type`: `winery_profile`, `experience`, `location`
- `owner_id`
- `asset_type`: `image`, `video`, `cover`
- `url`
- `caption`
- `copyright_owner`
- `license_note`
- `moderation_status`
- `sort_order`

Обязательное правило: без прав на фото не публиковать. Для партнеров нужна явная галочка/поле подтверждения прав.

Изображения и видео нельзя включать в Flutter assets и в итоговый APK/AAB, особенно с учетом размещения в Aurora Store. Приложение должно получать media по URL из удаленного хранилища. Рекомендуемая схема:

- Supabase Storage bucket: `tourism-media`;
- `home/wine/` - hero-изображение главной tourism-страницы для режима "Винные";
- `home/all/` - hero-изображение главной tourism-страницы для режима "Все туры";
- `home/other/` - hero-изображение главной tourism-страницы для режима "Другие";
- `organizations/{organization_id}/hero/` - hero-изображение страницы конкретного оператора/винодельни/бюро;
- `experiences/{experience_id}/cover/` - главное фото тура;
- `experiences/{experience_id}/gallery/` - галерея тура;
- `pickup-points/{pickup_point_id}/` - фото точки посадки и ориентиров;
- `vehicles/{vehicle_id}/` - фото автомобиля/минивэна;
- `wineries/{winery_id}/tourism/` - фото страницы винодельни и tourism-блоков;
- в БД хранить `storage_path`, публичный/signed URL, размеры, mime type, copyright/moderation metadata;
- приложение кэширует remote images, но не бандлит tourism media внутрь сборки.

Правило слотов изображений для tourism UI:

- главная `/tourism` не берет случайную фотографию конкретного тура как постоянный брендовый hero. Для нее задаются отдельные глобальные обложки по режимам каталога: `wine`, `all`, `other`;
- страница оператора `/tourism/o/:operatorSlug` использует собственный `tourism_organizations.hero_image_url/storage_path`, который может загрузить оператор или администратор через рабочее место;
- карточка конкретного тура использует `tourism_experiences.cover_image_url/storage_path`, а дополнительные фото остаются в gallery;
- fallback из cover/gallery допустим только в pilot/dev-режиме, когда слот еще не заполнен, но не является целевой моделью данных;
- массовое добавление и замена изображений должны выполняться через web/admin-интерфейс, чтобы одно изменение в БД сразу отражалось в приложении, Flutter web/QR-страницах и операторской витрине.

### 8.7. Schedule

Расписание лучше разделить на шаблон и конкретные слоты.

`tourism_schedules`

- `id`
- `experience_id`
- `timezone`
- `starts_on`
- `ends_on`
- `recurrence_rule`
- `weekday_mask`
- `start_time`
- `slot_duration_minutes`
- `capacity`
- `booking_cutoff_minutes`
- `cancellation_cutoff_minutes`
- `status`: `active`, `paused`, `archived`

`recurrence_rule` можно начать простым JSON:

```json
{
  "type": "weekly",
  "weekdays": [2, 4, 6],
  "times": ["11:00", "15:00"]
}
```

### 8.8. Schedule exceptions

`tourism_schedule_exceptions`

- `id`
- `schedule_id`
- `date`
- `type`: `closed`, `extra_slot`, `capacity_override`, `time_override`
- `start_time`
- `capacity`
- `reason`

### 8.9. Slots

`tourism_slots`

- `id`
- `experience_id`
- `schedule_id`
- `starts_at`
- `ends_at`
- `capacity`
- `reserved_count`
- `confirmed_count`
- `status`: `open`, `full`, `closed`, `cancelled`
- `source`: `generated`, `manual`

Слоты можно генерировать:

- заранее на 30-90 дней cron/job;
- лениво RPC при открытии календаря;
- вручную для простого MVP.

Первый управляемый срез: manual slots + простая генерация на 60 дней.

### 8.10. Booking

Booking - это будущий управляемый слой с подтвержденными местами, слотами, capacity control и, возможно, оплатой услуги после legal review.

В текущем Phase 2 Yalta pilot основная сущность - `tourism_leads`: легкая заявка без гарантированного места и без slot engine. Подтвержденная lead-заявка может показывать туристу детали поездки, фотобилет и памятку, но терминологически это еще не полноценный `tourism_booking`, пока не включены слоты, capacity и правила отмены.

`tourism_bookings`

- `id`
- `user_id`
- `experience_id`
- `slot_id` nullable for lead/external mode
- `provider_business_id`
- `status`: `requested`, `confirmed`, `rejected`, `cancelled_by_user`, `cancelled_by_business`, `expired`, `checked_in`, `completed`, `no_show`
- `guest_count`
- `guest_name`
- `guest_phone`
- `guest_email`
- `comment`
- `preferred_date`
- `preferred_time`
- `pickup_location_id`
- `pickup_time`
- `pickup_note_snapshot`
- `total_price_snapshot`
- `payment_status`: `not_required`, `pay_on_site`, `external_pending`, `external_paid`, `refunded`
- `confirmation_code`
- `checkin_qr_token`
- `referral_source`
- `referral_code`
- `created_at`
- `confirmed_at`
- `cancelled_at`
- `completed_at`

Правила:

- пользователь видит только свои bookings;
- business owner видит bookings только своего `provider_business_id`;
- admin видит все;
- телефон/email пользователя показываются business owner только после заявки и только в рамках обработки брони.

### 8.10.1. Transport assignment

Для маршрутов с трансфером booking должен поддерживать динамическое назначение транспорта. В начале сезона спрос может быть нестабильным: оператор обещает один автомобиль, а ближе к выезду объединяет группы, меняет машину или пересаживает часть туристов в другой минивэн/автобус. Это нормальная операционная реальность, и приложение не должно показывать устаревший номер автомобиля.

`tourism_vehicles`

- `id`
- `provider_business_id`
- `display_name`
- `vehicle_type`: `car`, `minivan`, `bus`, `other`
- `brand`
- `model`
- `color`
- `license_plate`
- `capacity`
- `driver_name`
- `driver_phone`
- `photo_url`
- `front_photo_url`
- `side_photo_url`
- `interior_photo_url`
- `license_plate_photo_url`
- `recognition_hint`
- `size_hint`
- `boarding_hint`
- `status`: `active`, `inactive`

`tourism_transport_assignments`

- `id`
- `experience_id`
- `slot_id`
- `provider_business_id`
- `vehicle_id` nullable;
- `vehicle_snapshot_json`
- `driver_snapshot_json`
- `pickup_location_id`
- `pickup_time`
- `status`: `planned`, `confirmed`, `changed`, `cancelled`
- `change_reason`
- `created_at`
- `updated_at`

`tourism_booking_transport_assignments`

- `id`
- `booking_id`
- `transport_assignment_id`
- `assigned_at`
- `notified_at`
- `acknowledged_at`
- `status`: `assigned`, `changed`, `cancelled`

Правила:

- booking может иметь текущую transport assignment;
- оператор может заменить vehicle/driver/pickup_time до отправления;
- изменение должно писать audit event;
- пользователь получает уведомление, если поменялись машина, номер, водитель, точка или время посадки;
- старое назначение сохраняется в истории, чтобы поддержка понимала, что было обещано раньше.
- ticket UI всегда берет актуальные фото и признаки из текущего `transport_assignment.vehicle_snapshot_json`;
- при смене автомобиля визуальный блок билета должен обновиться вместе с номером/маркой;
- если автомобиль временный и не заведён в базе, оператор может заполнить snapshot вручную, но для повторяющихся машин нужна база `tourism_vehicles`.

В ticket UI нужно писать не "машина навсегда закреплена", а:

> Транспорт может быть уточнен ближе к отправлению. Мы пришлем уведомление, если машина или номер изменятся.

### 8.10.2. Vehicle media and recognition hints

Для Ялты и других курортных городов фотография машины почти так же важна, как номер. Турист может не понимать марку/модель, парковка может быть забита, машина может остановиться чуть дальше от точки сбора, а посадка часто идет быстро.

У каждого часто используемого автомобиля должна быть visual profile:

- главное фото;
- фото спереди;
- фото сбоку;
- фото номера, если это допустимо для оператора;
- фото салона/посадочной двери optional;
- цвет;
- тип/габарит: легковая, минивэн, микроавтобус, автобус;
- текст `Как узнать машину`: например, `белый Mercedes Sprinter с табличкой "Экскурсии" на лобовом стекле`;
- подсказка посадки: `водитель останавливается у остановки, дверь со стороны моря`.

Если оператор меняет автомобиль:

- пользователь получает уведомление;
- в билете появляется блок `Машина изменилась`;
- старые фото не показываются как актуальные;
- новые фото подгружаются из vehicle profile или snapshot;
- кнопка `Показать фото машины` открывает fullscreen gallery.

Минимум для публикации транспорта в pilot:

- номер или понятный идентификатор;
- цвет;
- тип транспорта;
- 1 фото;
- recognition hint.

### 8.11. Booking status history

`tourism_booking_events`

- `id`
- `booking_id`
- `actor_user_id`
- `actor_type`: `user`, `business`, `admin`, `system`
- `event_type`
- `old_status`
- `new_status`
- `note`
- `created_at`

Нужно для разборов "почему заявка исчезла" и поддержки.

### 8.11.1. Lead ticket artifact / offline-pass

В pilot фотобилет хранится на уровне `tourism_leads`, потому что реальная операционная схема в офлайн-точках часто выглядит так: турист оплатил или подтвердил поездку вне WinePool, оператор выписал бумажный билет, сфотографировал его и отправил туристу.

WinePool должен превратить этот процесс в управляемый артефакт:

- оператор может прикрепить или заменить фото билета в деталях заявки;
- файл хранится в Supabase Storage bucket `tourism-media` под префиксом `lead-tickets/...`;
- в `tourism_leads` хранится текущий `ticket_photo_url` и `ticket_photo_storage_path`;
- событие замены пишется в `tourism_lead_events` с типом `ticket_updated`;
- авторизованный турист видит фото билета в `Мои заявки` и в карточке тура;
- турист получает in-app notification, если билет добавлен или заменен;
- desktop web должен открывать native `Save as` для self-contained HTML ticket;
- mobile должен сохранять фото билета в галерею/альбом `WinePool` и готовить HTML-памятку через системный share/save flow;
- HTML ticket должен быть самодостаточным: тур, статус, дата/время посадки, точка сбора, машина, агент/организация, рекомендации и embedded фото билета;
- если фото заменено, UI показывает только актуальную версию, а история остается в событиях заявки.

Открытые вопросы для следующей фазы:

- нужен ли QR/pass ID для проверки посадки;
- нужно ли хранить несколько версий фото билета в UI или только текущую;
- как чистить старые файлы из Storage при замене/удалении заявки;
- какие Android/Aurora permissions и UX нужны для гарантированного сохранения в галерею.

### 8.11.2. Paper ticket delivery service

Проблема: часть туристов отказывается от удаленной покупки экскурсии, если единственный вариант - перевести деньги незнакомому агенту и получить фотобилет в мессенджере. Это выглядит рискованно из-за распространенных мошеннических схем. При этом турист готов купить экскурсию, если бумажный билет привезут лично и деньги передаются при получении.

WinePool должен поддержать optional service `Доставка бумажного билета`:

- это доставка бумажного билета/документа на экскурсию, а не доставка алкоголя;
- WinePool не принимает оплату внутри приложения;
- оплата билета и доставки происходит вне WinePool по договоренности с оператором;
- в pilot доставку выполняет туроператор/экскурсионное бюро, если не подключена отдельная служба WinePool;
- стоимость доставки может быть ориентировочной в карточке тура и подтвержденной оператором в заявке;
- примерный порядок цены для Ялты: 200-400 рублей в зависимости от удаленности, но точная сумма задается оператором.

Availability / pricing:

- в release 1.0.5 доставка включается на уровне оператора;
- у оператора хранится текстовая зона доставки, например `Ялта и ближайшие районы`;
- у оператора хранится ориентир по цене: min/max в рублях и policy text;
- если тур идет из другого города, оператор должен явно указать это в зоне/описании;
- в следующей фазе delivery zones можно вынести в отдельную таблицу с городом, районом, радиусом, price rules and courier assignment.

Customer flow:

1. Турист открывает тур.
2. В форме заявки видит option `Нужна доставка бумажного билета`, если услуга доступна для города/оператора.
3. При включении заполняет адрес/ориентир, удобное время, контакт и комментарий.
4. После отправки заявки видит в `Центре поездки` блок `Доставка билета`.
5. Оператор подтверждает возможность доставки, стоимость и статус.
6. Турист получает уведомления об изменениях доставки.

Operator flow:

1. Оператор видит delivery-запрос в деталях заявки.
2. Уточняет наличие мест и возможность доставки.
3. Назначает статус доставки и стоимость.
4. При необходимости пишет комментарий туристу.
5. После передачи билета отмечает `Билет доставлен`.

Минимальная модель данных для release 1.0.5:

- `tourism_organizations.ticket_delivery_enabled boolean not null default false`
- `tourism_organizations.ticket_delivery_area_label text`
- `tourism_organizations.ticket_delivery_fee_hint_min_rub integer`
- `tourism_organizations.ticket_delivery_fee_hint_max_rub integer`
- `tourism_organizations.ticket_delivery_policy_text text`
- `tourism_leads.ticket_delivery_requested boolean not null default false`
- `tourism_leads.ticket_delivery_city text`
- `tourism_leads.ticket_delivery_address text`
- `tourism_leads.ticket_delivery_time_window text`
- `tourism_leads.ticket_delivery_contact text`
- `tourism_leads.ticket_delivery_comment text`
- `tourism_leads.ticket_delivery_status text not null default 'not_requested'`
- `tourism_leads.ticket_delivery_fee_rub integer`
- `tourism_leads.ticket_delivery_operator_note text`

Allowed `ticket_delivery_status`:

- `not_requested`
- `requested`
- `confirming`
- `confirmed`
- `out_for_delivery`
- `delivered`
- `cancelled`
- `unavailable`

Events/notifications:

- изменение delivery-полей оператором пишет `tourism_lead_events.event_type = ticket_delivery_updated`;
- customer notification deep-links to `/tourism/trip/:leadId`;
- история заявки должна показывать, когда доставка была запрошена, подтверждена, отменена или выполнена.

UI copy guardrails:

- не писать `оплатите в приложении`;
- не писать `WinePool гарантирует доставку`;
- не писать `доставка вина`;
- писать: `Оператор уточнит возможность доставки бумажного билета и стоимость. Оплата происходит вне WinePool.`

### 8.12. Favorites / follows

`winery_follows`

- `user_id`
- `winery_id`
- `created_at`

`tourism_experience_favorites`

- `user_id`
- `experience_id`
- `created_at`

### 8.13. Check-in

`tourism_checkins`

- `id`
- `booking_id`
- `user_id`
- `experience_id`
- `location_id`
- `checked_in_at`
- `source`: `qr_scan`, `manual_business`, `admin`

Check-in открывает будущие бейджи:

- "Был на винодельне";
- "Крымский маршрут";
- "Первая дегустация на винодельне".

### 8.14. Tourism interest / wish request

`tourism_interest_requests`

- `id`
- `user_id` nullable for guest/local-to-auth continuation;
- `city`
- `region_id`
- `desired_from_date`
- `desired_to_date`
- `guest_count`
- `desired_experience_type`
- `desired_winery_id` nullable
- `free_text`
- `pickup_area_text`
- `contact_phone`
- `contact_email`
- `notify_when_available`
- `status`: `new`, `clustered`, `planned`, `matched`, `closed`
- `matched_experience_id`
- `source_filters_json`
- `created_at`
- `updated_at`

Назначение:

- собирать спрос, когда пользователь ничего не нашел;
- видеть однотипные запросы;
- превращать спрос в новые маршруты/туры;
- дать туроператору или администратору аргумент для переговоров с винодельней.

Это не booking. Пользователю нужно явно писать:

> Мы сохраним пожелание и сообщим, если появится подходящий маршрут.

### 8.15. Tourism demand clusters

`tourism_demand_clusters`

- `id`
- `city`
- `region_id`
- `experience_type`
- `desired_winery_id`
- `request_count`
- `guest_count_sum`
- `date_range_summary`
- `top_free_text_terms`
- `status`: `new`, `reviewing`, `operator_contacted`, `experience_created`, `dismissed`
- `assigned_admin_id`
- `created_at`
- `updated_at`

Кластеры можно строить вручную в админке на первом этапе, позже - scheduled job / RPC.

## 9. База данных: пример SQL-контракта

Ниже не финальная миграция, а контракт для реализации.

```sql
create type public.business_type_new as enum (
  'winery',
  'retail',
  'horeca',
  'tourism_operator'
);

create type public.tourism_experience_type as enum (
  'winery_tour',
  'tasting',
  'masterclass',
  'route',
  'festival',
  'private_visit'
);

create type public.tourism_experience_category as enum (
  'wine',
  'gastronomy',
  'nature',
  'history',
  'sea',
  'mountains',
  'city',
  'culture',
  'other'
);

create type public.tourism_publication_status as enum (
  'draft',
  'submitted',
  'approved',
  'published',
  'paused',
  'rejected',
  'archived'
);

create type public.tourism_booking_status as enum (
  'requested',
  'confirmed',
  'rejected',
  'cancelled_by_user',
  'cancelled_by_business',
  'expired',
  'checked_in',
  'completed',
  'no_show'
);

create table public.winery_public_profiles (
  winery_id uuid primary key references public.wineries(id) on delete cascade,
  slug text unique,
  headline text,
  story_text text,
  visit_summary text,
  winemaker_name text,
  winemaker_photo_url text,
  hero_image_url text,
  hero_video_url text,
  publication_status public.tourism_publication_status not null default 'draft',
  moderation_notes text,
  published_at timestamptz,
  published_by uuid references public.profiles(id),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table public.winery_profile_sections (
  id uuid primary key default gen_random_uuid(),
  winery_id uuid not null references public.wineries(id) on delete cascade,
  section_type text not null,
  title text,
  subtitle text,
  body_markdown text,
  media_asset_ids uuid[],
  linked_wine_ids uuid[],
  linked_article_ids uuid[],
  display_style text not null default 'standard',
  sort_order integer not null default 0,
  publication_status public.tourism_publication_status not null default 'draft',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table public.winery_profile_facts (
  id uuid primary key default gen_random_uuid(),
  winery_id uuid not null references public.wineries(id) on delete cascade,
  fact_key text not null,
  fact_value text not null,
  source text,
  verified_at timestamptz,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table public.tourism_experiences (
  id uuid primary key default gen_random_uuid(),
  provider_business_id uuid not null references public.businesses(id) on delete cascade,
  operator_organization_id uuid,
  default_responsible_user_id uuid references public.profiles(id) on delete set null,
  primary_winery_id uuid references public.wineries(id) on delete set null,
  primary_location_id uuid references public.locations(id) on delete set null,
  title text not null,
  subtitle text,
  description text,
  experience_type public.tourism_experience_type not null,
  experience_category public.tourism_experience_category not null default 'wine',
  is_wine_related boolean not null default true,
  duration_minutes integer,
  min_guests integer not null default 1,
  max_guests integer not null default 1,
  age_limit integer not null default 18,
  language_codes text[] not null default array['ru'],
  price_from numeric(12,2),
  price_currency text not null default 'RUB',
  price_note text,
  payment_mode text not null default 'none',
  booking_mode text not null default 'lead_request',
  external_booking_url text,
  contact_phone text,
  contact_messenger_url text,
  included_text text,
  not_included_text text,
  requirements_text text,
  cancellation_policy_text text,
  accessibility_text text,
  is_transport_included boolean not null default false,
  meeting_location_id uuid references public.locations(id) on delete set null,
  status public.tourism_publication_status not null default 'draft',
  moderation_notes text,
  published_at timestamptz,
  created_by uuid references public.profiles(id),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint tourism_experiences_capacity_check check (max_guests >= min_guests),
  constraint tourism_experiences_age_check check (age_limit >= 18)
);

create table public.tourism_organizations (
  id uuid primary key default gen_random_uuid(),
  business_id uuid references public.businesses(id) on delete set null,
  name text not null,
  organization_type text not null default 'tour_operator',
  city text,
  region text,
  contact_phone text,
  contact_messenger_url text,
  notification_email text,
  notification_telegram_chat_id text,
  website_url text,
  logo_url text,
  hero_image_url text,
  hero_image_storage_path text,
  status text not null default 'active',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint tourism_organizations_type_check
    check (organization_type in ('tour_operator', 'winery', 'offline_point', 'winepool_internal')),
  constraint tourism_organizations_status_check
    check (status in ('active', 'paused', 'archived'))
);

create table public.tourism_home_hero_images (
  id uuid primary key default gen_random_uuid(),
  hero_mode text not null,
  storage_bucket text not null default 'tourism-media',
  storage_path text,
  public_url text,
  status text not null default 'active',
  sort_order integer not null default 100,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint tourism_home_hero_images_mode_check
    check (hero_mode in ('wine', 'all', 'other')),
  constraint tourism_home_hero_images_status_check
    check (status in ('draft', 'active', 'archived')),
  constraint tourism_home_hero_images_source_check
    check (
      nullif(trim(coalesce(storage_path, '')), '') is not null
      or nullif(trim(coalesce(public_url, '')), '') is not null
    )
);

create table public.tourism_organization_members (
  id uuid primary key default gen_random_uuid(),
  organization_id uuid not null references public.tourism_organizations(id) on delete cascade,
  user_id uuid not null references public.profiles(id) on delete cascade,
  member_role text not null default 'agent',
  is_default_responsible boolean not null default false,
  receives_new_lead_notifications boolean not null default true,
  status text not null default 'active',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  unique (organization_id, user_id),
  constraint tourism_organization_members_role_check
    check (member_role in ('owner', 'manager', 'agent', 'viewer')),
  constraint tourism_organization_members_status_check
    check (status in ('active', 'paused', 'archived'))
);

create table public.tourism_leads (
  id uuid primary key default gen_random_uuid(),
  experience_id uuid not null references public.tourism_experiences(id) on delete cascade,
  assigned_organization_id uuid references public.tourism_organizations(id) on delete set null,
  assigned_user_id uuid references public.profiles(id) on delete set null,
  status text not null default 'new',
  customer_name text not null,
  contact text not null,
  preferred_date_label text,
  guests_count integer not null default 1,
  comment text,
  operator_note text,
  assigned_pickup_location_id uuid,
  assigned_vehicle_id uuid,
  pickup_time_label text,
  tourist_memo text,
  ticket_photo_url text,
  ticket_photo_storage_path text,
  source text not null default 'app',
  created_by uuid references public.profiles(id) on delete set null,
  contacted_at timestamptz,
  confirmed_at timestamptz,
  cancelled_at timestamptz,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint tourism_leads_status_check
    check (status in ('new', 'contacted', 'confirmed', 'cancelled')),
  constraint tourism_leads_source_check
    check (source in ('app', 'admin', 'offline_point', 'import')),
  constraint tourism_leads_guests_count_check check (guests_count between 1 and 99)
);

create table public.tourism_lead_events (
  id uuid primary key default gen_random_uuid(),
  lead_id uuid not null references public.tourism_leads(id) on delete cascade,
  actor_user_id uuid references public.profiles(id) on delete set null,
  actor_type text not null default 'system',
  event_type text not null,
  old_status text,
  new_status text,
  note text,
  created_at timestamptz not null default now()
);

create table public.tourism_experience_stops (
  id uuid primary key default gen_random_uuid(),
  experience_id uuid not null references public.tourism_experiences(id) on delete cascade,
  order_index integer not null default 0,
  winery_id uuid references public.wineries(id) on delete set null,
  location_id uuid references public.locations(id) on delete set null,
  title text,
  description text,
  planned_duration_minutes integer,
  arrival_offset_minutes integer,
  latitude numeric,
  longitude numeric,
  address_text text,
  created_at timestamptz not null default now()
);

create table public.tourism_schedules (
  id uuid primary key default gen_random_uuid(),
  experience_id uuid not null references public.tourism_experiences(id) on delete cascade,
  timezone text not null default 'Europe/Moscow',
  starts_on date not null,
  ends_on date,
  recurrence_rule jsonb,
  weekday_mask integer[],
  start_time time,
  slot_duration_minutes integer,
  capacity integer not null default 1,
  booking_cutoff_minutes integer not null default 180,
  cancellation_cutoff_minutes integer not null default 1440,
  status text not null default 'active',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table public.tourism_slots (
  id uuid primary key default gen_random_uuid(),
  experience_id uuid not null references public.tourism_experiences(id) on delete cascade,
  schedule_id uuid references public.tourism_schedules(id) on delete set null,
  starts_at timestamptz not null,
  ends_at timestamptz not null,
  capacity integer not null,
  reserved_count integer not null default 0,
  confirmed_count integer not null default 0,
  status text not null default 'open',
  source text not null default 'manual',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint tourism_slots_capacity_check check (capacity >= 0),
  constraint tourism_slots_time_check check (ends_at > starts_at)
);

create table public.tourism_bookings (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references public.profiles(id) on delete cascade,
  experience_id uuid not null references public.tourism_experiences(id) on delete restrict,
  slot_id uuid references public.tourism_slots(id) on delete set null,
  provider_business_id uuid not null references public.businesses(id) on delete restrict,
  status public.tourism_booking_status not null default 'requested',
  guest_count integer not null default 1,
  guest_name text,
  guest_phone text,
  guest_email text,
  comment text,
  preferred_date date,
  preferred_time time,
  pickup_location_id uuid references public.locations(id) on delete set null,
  pickup_time timestamptz,
  pickup_note_snapshot text,
  total_price_snapshot numeric(12,2),
  payment_status text not null default 'not_required',
  confirmation_code text unique,
  checkin_qr_token text unique,
  referral_source text,
  referral_code text,
  created_at timestamptz not null default now(),
  confirmed_at timestamptz,
  cancelled_at timestamptz,
  completed_at timestamptz,
  constraint tourism_bookings_guest_count_check check (guest_count > 0)
);

create table public.tourism_booking_events (
  id uuid primary key default gen_random_uuid(),
  booking_id uuid not null references public.tourism_bookings(id) on delete cascade,
  actor_user_id uuid references public.profiles(id) on delete set null,
  actor_type text not null,
  event_type text not null,
  old_status text,
  new_status text,
  note text,
  created_at timestamptz not null default now()
);

create table public.tourism_vehicles (
  id uuid primary key default gen_random_uuid(),
  provider_business_id uuid not null references public.businesses(id) on delete cascade,
  display_name text,
  vehicle_type text not null default 'minivan',
  brand text,
  model text,
  color text,
  license_plate text,
  capacity integer,
  driver_name text,
  driver_phone text,
  photo_url text,
  front_photo_url text,
  side_photo_url text,
  interior_photo_url text,
  license_plate_photo_url text,
  recognition_hint text,
  size_hint text,
  boarding_hint text,
  status text not null default 'active',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table public.tourism_transport_assignments (
  id uuid primary key default gen_random_uuid(),
  experience_id uuid not null references public.tourism_experiences(id) on delete cascade,
  slot_id uuid references public.tourism_slots(id) on delete cascade,
  provider_business_id uuid not null references public.businesses(id) on delete cascade,
  vehicle_id uuid references public.tourism_vehicles(id) on delete set null,
  vehicle_snapshot_json jsonb,
  driver_snapshot_json jsonb,
  pickup_location_id uuid references public.locations(id) on delete set null,
  pickup_time timestamptz,
  status text not null default 'planned',
  change_reason text,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table public.tourism_booking_transport_assignments (
  id uuid primary key default gen_random_uuid(),
  booking_id uuid not null references public.tourism_bookings(id) on delete cascade,
  transport_assignment_id uuid not null references public.tourism_transport_assignments(id) on delete cascade,
  assigned_at timestamptz not null default now(),
  notified_at timestamptz,
  acknowledged_at timestamptz,
  status text not null default 'assigned'
);
```

## 10. RLS и backend enforcement

### 10.1. Read policies

Public read:

- published winery profiles;
- published tourism experiences;
- public tourism locations with `is_tourism_active = true`;
- published slots with limited fields.

Authenticated user read:

- all public read;
- own tourism leads where `created_by = auth.uid()`;
- own bookings;
- own follows/favorites.

Business owner read:

- their business;
- their locations;
- their experiences in all statuses;
- leads/bookings for their experiences;
- contact fields for leads/bookings after request creation.

Tourism organization member read:

- active organizations where the user is a member;
- experiences assigned to those organizations;
- leads assigned to those organizations;
- contact fields only for assigned leads;
- internal operator notes only inside operator/admin surfaces.

Admin read:

- all.

### 10.2. Write policies

User:

- create lead/request with contact data in Phase 2;
- create booking in Phase 3 managed booking;
- cancel/clarify own lead when product rules allow;
- cancel own booking if status allows in Phase 3;
- create favorite/follow;
- create post-visit review if booking completed/checked_in or ordinary review flow allows.

Business owner:

- create/update own locations, but public visibility requires moderation;
- create/update tourism experiences for owned business;
- create/update schedules and slots for owned experiences;
- confirm/reject/cancel own leads/bookings;
- cannot set `published` directly unless capability allows trusted partner direct publish.

Tourism organization member:

- take lead into work for own organization;
- update assigned lead status: `new -> contacted -> confirmed/cancelled`;
- write `operator_note`;
- assign lead to self or another active member of the same organization if member role allows;
- update pilot trip fields: assigned pickup point, vehicle, pickup time and tourist memo;
- attach/replace lead photo ticket in `tourism-media/lead-tickets/...`;
- update vehicle/driver assignment only when the experience belongs to the organization and the user has `owner`, `manager` or permitted `agent` role.

Admin/moderator:

- approve/publish/pause/reject;
- set partner/verified flags;
- resolve complaints;
- change binding;
- create tourism organizations and membership links;
- reassign leads between organizations and agents;
- override lead/booking statuses with audit event.

### 10.3. RPC/functions

Обязательные RPC:

- `get_public_winery_profile(p_winery_id_or_slug)`
- `get_public_winery_tourism(p_winery_id)`
- `search_tourism_experiences(p_region_id, p_city, p_type, p_date_from, p_date_to, p_guests)`
- `get_tourism_experience_details(p_experience_id)`
- `create_tourism_lead(p_experience_id, p_customer_name, p_contact, p_preferred_date_label, p_guests_count, p_comment, p_source)`
- `operator_update_tourism_lead_status(p_lead_id, p_status, p_operator_note)`
- `operator_assign_tourism_lead(p_lead_id, p_assigned_user_id)`
- `get_tourism_slots(p_experience_id, p_from, p_to)`
- `create_tourism_booking(p_experience_id, p_slot_id, p_guest_count, p_contact_json, p_referral_json)`
- `cancel_tourism_booking(p_booking_id, p_reason)`
- `business_confirm_tourism_booking(p_booking_id, p_note)`
- `business_reject_tourism_booking(p_booking_id, p_note)`
- `business_assign_transport(p_slot_id, p_vehicle_id, p_booking_ids, p_pickup_location_id, p_pickup_time, p_note)`
- `business_change_booking_transport(p_booking_id, p_transport_assignment_id, p_reason)`
- `admin_publish_tourism_experience(p_experience_id)`
- `admin_pause_tourism_experience(p_experience_id, p_reason)`

Для slot booking нужен transactional RPC, который:

- lock slot row;
- проверяет capacity;
- создает booking;
- увеличивает reserved/confirmed count по статусу;
- пишет `tourism_booking_events`;
- отправляет notification task.

## 11. Flutter modules

Рекомендуемая структура:

```text
lib/features/winery_tourism/
  domain/
    winery_public_profile.dart
    tourism_experience.dart
    tourism_slot.dart
    tourism_booking.dart
    tourism_lead.dart
    tourism_stop.dart
  data/
    winery_tourism_repository.dart
    tourism_booking_repository.dart
  application/
    public_winery_profile_provider.dart
    tourism_experience_search_provider.dart
    tourism_booking_controller.dart
    tourism_lead_submit_controller.dart
    business_tourism_controller.dart
  presentation/
    public_winery_profile_screen.dart
    tourism_experience_details_screen.dart
    tourism_search_screen.dart
    tourism_list_screen.dart
    my_tourism_bookings_screen.dart
    tourism_booking_sheet.dart
    business_tourism_dashboard_screen.dart
    business_experience_editor_screen.dart
    admin_tourism_moderation_screen.dart
    admin_tourism_experiences_screen.dart
```

Почему отдельный feature folder:

- не смешивать consumer tourism UI с текущими `wines`;
- сохранить связь через ids и providers;
- легче feature-flag'ить весь слой.

### 11.1. Mobile-first и полноценные web/admin surfaces

Пользовательские экраны WinePool остаются mobile-first, потому что основной продукт - мобильное приложение. Но часть tourism/winery surfaces должна проектироваться сразу с учетом полноценной web/desktop версии, потому что там нужно вводить и редактировать большой объем данных.

Актуализация для QR/web-входа: публичные страницы тура и оператора тоже не должны становиться отдельными вручную поддерживаемыми лендингами. Один тур создается и редактируется в одном месте (`tourism_experiences` + stops/pickup/media/vehicles/provider), а приложение, Flutter web/public route и SEO/QR transition pages читают тот же published snapshot. Лендинг `winepool.ru` может ссылаться на эти страницы и давать маркетинговый контекст, но не должен хранить копию расписания, цены, точки посадки, машины или фотоконтента.

Mobile-first:

- `/tourism`;
- `/tourism/:experienceId`;
- ticket/booking screen;
- pickup point screen;
- route preview;
- check-in / post-visit flow;
- user-facing winery profile.

Responsive public web required for QR/acquisition:

- public operator page: `/tourism/o/:operatorSlug`;
- public tour page: `/tourism/o/:operatorSlug/t/:tourSlug`;
- web fallback for `/tourism/:experienceId`;
- QR/referral landing state: app installed vs app not installed;
- guest lead request form, if legal/product decision allows web lead submission.

Wide-screen web/admin required:

- admin tourism moderation;
- winery profile section builder;
- media gallery management;
- experience editor;
- schedule/slot editor;
- pickup point editor with photos;
- vehicle/transport assignment dashboard;
- booking dashboard;
- B2B analytics dashboard;
- featured placement/package management.

Требование к реализации:

- single source of truth: operator/admin changes must update app card, web tour page, QR page and future landing references without manual duplication;
- consumer screens can be responsive but optimized for phone;
- public tourism pages must have a real desktop/tablet layout for QR traffic from laptops, hotel tablets and web previews, but remain mobile-first for tourists scanning QR on phones;
- admin/business screens must use adaptive layouts with wider grids/tables/forms on desktop;
- large text/media forms should not be forced into narrow mobile-only UI;
- admin web build (`admin.winepool.ru`) должен быть first-class target для тяжелого редактирования;
- туроператорские и винодельческие рабочие места должны поддерживать полноценную работу на большом экране: массовое редактирование, таблицы, календарные сетки, медиа-галереи, назначение транспорта и сверку заявок;
- mobile admin can exist as emergency/light mode, not as the only way to manage content.

## 12. Роутинг

Публичные:

- `/wineries`
- `/wineries/:idOrSlug`
- `/wineries/:idOrSlug/tourism`
- `/tourism`
- `/tourism/:experienceId`
- `/tourism/o/:operatorSlug`
- `/tourism/o/:operatorSlug/t/:tourSlug`
- `/tourism/:experienceId/book`
- `/tourism/bookings`

Business:

- `/business/tourism`
- `/business/tourism/experiences/new`
- `/business/tourism/experiences/:id/edit`
- `/business/tourism/bookings`
- `/business/locations`

Admin:

- `/admin/tourism`
- `/admin/tourism/experiences/:id`
- `/admin/tourism/featured`
- `/admin/wineries/:id/public-profile`
- `/admin/businesses/:id/locations`

Feature flags:

- `AppFlags.wineryTourismEnabled`
- `AppFlags.tourismBookingEnabled`
- `AppFlags.tourismPaymentsEnabled` default false
- `AppFlags.businessTourismSelfServiceEnabled`

## 13. Пользовательские сценарии

### 13.1. Открыть винодельню из карточки вина

1. Пользователь открывает карточку вина.
2. Блок "Винодельня" показывает:
   - название;
   - регион;
   - короткое описание;
   - CTA `Открыть винодельню`.
3. Экран винодельни показывает:
   - hero;
   - история/описание;
   - вина этой винодельни;
   - локации;
   - экскурсии/дегустации, если есть published experiences;
   - карта и маршрут.

### 13.2. Найти экскурсию или дегустационный тур в общем каталоге

Это отдельный first-class сценарий. Пользователь может не знать конкретную винодельню. Его мысль может быть простой: "Мы на курорте, давай еще что-нибудь продегустируем". Поэтому WinePool должен давать быстрый вход не только в winery pages, но и в общий перечень туров/дегустаций.

1. Пользователь открывает раздел `Туризм` (`/tourism`).
2. Видит общий каталог published tourism experiences:
   - экскурсии на винодельни;
   - дегустации;
   - маршруты с несколькими винодельнями;
   - программы с трансфером;
   - частные визиты;
   - сезонные события.
3. Может быстро отфильтровать:
   - город/регион: `Ялта`, `Крым`;
   - дата или период;
   - длительность;
   - есть трансфер / без трансфера;
   - точка посадки рядом;
   - тип программы;
   - количество гостей;
   - язык;
   - примерная стоимость, если поле опубликовано.
4. Каждая карточка в списке показывает:
   - название тура;
   - основные винодельни/остановки;
   - длительность;
   - ближайшую доступную дату или режим `по заявке`;
   - наличие трансфера;
   - город/точку сбора;
   - цену/price note, если разрешено;
   - CTA `Подробнее`.
5. Пользователь открывает тур и видит:
   - карту маршрута;
   - точки, куда заезжает маршрут;
   - точки посадки;
   - расписание/доступные даты;
   - подробное описание;
   - кнопку заявки.

### 13.3. Найти винодельни рядом с Ялтой

1. Пользователь открывает `Туризм` или карту.
2. Фильтр город/регион: `Ялта`, `Крым`.
3. Видит:
   - винодельни;
   - дегустационные залы;
   - маршруты с точкой сбора в Ялте;
   - программы с трансфером.
4. Может открыть маршрут или оставить заявку.

### 13.4. Оставить заявку на дегустацию

1. Пользователь открывает experience.
2. Нажимает `Оставить заявку`.
3. Указывает имя, контакт, желаемую дату, количество гостей и комментарий.
4. Подтверждает 18+ через общий age gate приложения.
5. Создается `tourism_lead` со статусом `new`.
6. Пользователь получает сообщение `Заявка отправлена`.
7. WinePool admin видит заявку в `/admin/tourism`.
8. Ответственный tour agent/operator получает notification и видит заявку в своем операторском контуре.

Для первого pilot допускается отправка заявки гостем без обязательной регистрации, потому что главная цель - не потерять туриста с офлайн-точки или курортного сценария. Если пользователь авторизован, `created_by` сохраняется и заявка может быть связана с его будущим билетом/историей.

### 13.5. Обработка заявки ответственным за тур

1. Система определяет `assigned_organization_id` и `assigned_user_id` по experience:
   - сначала `tourism_experiences.default_responsible_user_id`;
   - затем default responsible member организации;
   - если ответственный не задан, заявка попадает в очередь организации и в admin overview.
2. Ответственный tour agent/operator получает notification как можно быстрее.
3. Открывает экран `Мои заявки` или операторский dashboard.
4. Видит новую заявку с контактом, датой, количеством гостей, комментарием, маршрутом, точками посадки и текущей информацией по транспорту.
5. Берет заявку в работу: статус `contacted`.
6. Связывается с туристом, уточняет наличие мест, дату, точку посадки, автомобиль/минивэн и условия оплаты на месте.
7. Если поездка подтверждается, ставит `confirmed`.
8. Если мест нет, группа не набралась, турист отказался или направление отменено, ставит `cancelled` с внутренней заметкой.
9. Пользователь получает notification о подтверждении/отмене, если он авторизован или если подключен внешний канал уведомлений.
10. Admin WinePool видит историю действий и может переназначить заявку.

### 13.5.1. Переход lead -> booking

`tourism_lead` - это легкая заявка без гарантированного места. `tourism_booking` появляется только когда включен slot/booking engine или когда оператор явно подтверждает место и WinePool должен показать пользователю полноценный билет.

Переход:

1. Lead `new/contacted`.
2. Оператор подтверждает дату/места/точку посадки.
3. Система создает `tourism_booking` или связывает lead с существующим slot.
4. Lead получает статус `confirmed`.
5. Пользователь видит билет/подтверждение с pickup point, vehicle snapshot и кодом/QR, если этот уровень включен.

### 13.6. Построить маршрут

1. Пользователь нажимает `Маршрут`.
2. Если в booking назначена `pickup_location_id`, route строится именно до этой точки посадки.
3. Если у маршрута несколько точек посадки, но пользователь еще не выбрал точку, экран сначала показывает список точек с адресами, временем и фото-ориентирами.
4. Если есть только общая точка сбора - route строится до meeting location.
5. Если нет - до primary location.
6. Перед открытием внешней карты пользователь видит карточку точки посадки:
   - точное время сбора;
   - адрес;
   - текст "как найти";
   - фото места;
   - ближайшие ориентиры;
   - телефон/мессенджер для связи.
7. Приложение открывает external navigation:
   - Yandex Maps / Yandex Navigator deep link;
   - fallback `geo:`/browser URL.
8. В приложении не реализуется turn-by-turn.

### 13.6.1. Найти точку посадки на курорте

1. Пользователь открывает подтвержденную заявку или билет.
2. Видит блок `Где садиться`:
   - название точки;
   - время, когда нужно быть на месте;
   - мини-карту;
   - 2-5 фотографий локации;
   - список ориентиров рядом.
3. Открывает фото на весь экран, чтобы свериться на месте.
4. Нажимает `Построить маршрут`, если нужно.
5. Если пользователь все равно не находит место, нажимает `Связаться с организатором`.

Этот сценарий считается обязательным для крымского pilot, потому что сейчас продавцы экскурсий часто вручную фотографируют место, отправляют фото в мессенджер и текстом объясняют, где ждать автобус. WinePool должен снять эту операционную нагрузку.

### 13.6.2. Машина или водитель изменились перед выездом

1. Пользователь получил подтверждение маршрута и видит в билете:
   - точку посадки;
   - время;
   - машину/минивэн, если уже назначены;
   - номер автомобиля;
   - фото автомобиля;
   - подсказку "как узнать машину";
   - водителя/контакт, если оператор публикует эти данные.
2. В процессе набора группы оператор объединяет автомобили или пересаживает часть туристов.
3. Business/operator открывает booking/slot dashboard и меняет transport assignment:
   - vehicle;
   - driver;
   - license plate;
   - pickup time;
   - pickup point, если нужно.
4. Система:
   - сохраняет старое и новое значение в audit history;
   - обновляет билет пользователя;
   - отправляет push/in-app/email notification;
   - показывает в билете заметный статус `Транспорт изменился`.
5. Пользователь открывает уведомление и видит:
   - что изменилось;
   - актуальную машину/номер/водителя;
   - актуальные фото машины;
   - визуальные признаки: цвет, габарит, табличка/надпись/место остановки;
   - актуальную точку и время посадки;
   - кнопку `Понятно`;
   - кнопку связи с организатором.
6. После acknowledgement статус можно пометить как `acknowledged_at`, чтобы оператор видел, кто уже открыл изменение.

Это критично для начала сезона, когда спрос нестабилен и группы часто объединяются. Приложение должно снижать тревожность туриста и операционную нагрузку продавца/оператора.

### 13.7. Check-in на винодельне

1. Пользователь приезжает.
2. В booking есть QR-код.
3. Представитель винодельни сканирует QR в business/admin mode или вводит код.
4. Booking становится `checked_in`.
5. Пользователь получает предложение:
   - создать дегустацию;
   - оставить отзыв;
   - получить бейдж.

Check-in должен быть не просто отметкой посещения, а началом post-visit loop. После визита WinePool знает контекст: какую винодельню пользователь посетил, какой тур/дегустацию, какие вина могли быть в программе, где это произошло и когда. Этот контекст нужно использовать, чтобы снизить трение при создании дегустации и усилить ценность дневника.

### 13.7.1. Post-visit flow

После check-in или после времени завершения confirmed booking пользователь получает ненавязчивый экран/уведомление:

- `Что запомнилось?`;
- `Добавить дегустацию`;
- `Оставить отзыв о визите`;
- `Сохранить винодельню`;
- `Посмотреть вина винодельни`;
- `Поделиться маршрутом/впечатлением`.

Если у experience задан список `featured_wine_ids` или `tasting_set`, экран предлагает быстрый выбор вин:

- пользователь отмечает вина, которые пробовал;
- оценка и заметка создаются быстрее, чем через обычный поиск;
- дегустация получает контекст `source = winery_visit`;
- можно привязать фото бокала/этикетки/локации.

### 13.7.2. Visit memory card

В профиле пользователя и в карточке винодельни можно показывать личный блок:

> Вы были здесь 12 июня 2026

Карточка визита содержит:

- винодельню;
- тур/маршрут;
- дату;
- фото пользователя, если добавлены;
- дегустации, созданные из визита;
- отзыв;
- бейджи;
- кнопку `Вернуться к визиту`.

Это превращает туризм в часть личной винной истории, а не в одноразовую заявку.

### 13.7.3. Badges and achievements

Минимальные бейджи:

- `Первая винодельня`;
- `Крымский маршрут`;
- `Дегустация на месте`;
- `3 винодельни`;
- `Вернулся на винодельню`;
- region-specific badges, e.g. `Южный берег Крыма`.

Бейдж должен выдаваться по check-in/booking completion, а не только по ручному отзыву, чтобы пользователь получил мгновенное подкрепление.

### 13.7.4. Winery-side value

Для винодельни/оператора check-in дает:

- подтвержденное посещение;
- show-up/no-show аналитику;
- понимание, какие программы приводят людей;
- post-visit review prompt;
- агрегированную аналитику интереса без раскрытия лишних персональных данных.

### 13.7.5. Anti-fraud and privacy

Check-in не должен превращаться в накрутку:

- QR token одноразовый или ограниченный по времени;
- business/admin может вручную отметить check-in только по своей booking;
- пользователь не может сам бесконечно отмечаться без подтвержденного booking или гео/QR-фактора;
- публично не показывать факт посещения без согласия пользователя;
- отзыв о визите проходит те же moderation rules, что и другие публичные отзывы.

### 13.8. Офлайн-точка продаж экскурсий

1. На точке в Ялте размещен QR на страницу WinePool tourism experience или route collection.
2. URL содержит `referral_source=yalta_booth_001`.
3. Пользователь открывает карточку маршрута.
4. Оставляет заявку.
5. В booking сохраняются referral fields.
6. В админке видна статистика заявок по источнику.

### 13.9. Оставить пожелание на тур, которого нет

1. Пользователь открывает `/tourism`.
2. Применяет фильтры: город, дата, формат, трансфер, винодельня.
3. Подходящих туров нет.
4. Empty state предлагает:
   - соседние даты/регионы;
   - кнопку `Оставить пожелание`.
5. Пользователь заполняет короткую форму:
   - откуда хочет ехать;
   - куда хочет попасть или какой формат интересует;
   - желаемые даты;
   - количество гостей;
   - комментарий;
   - уведомить, если появится подходящий тур.
6. WinePool сохраняет `tourism_interest_request`.
7. Пользователь получает честное сообщение: `Это не заявка на бронирование. Мы сохраним пожелание и сообщим, если появится подходящий маршрут`.
8. Администратор видит накопление похожих запросов.
9. Если таких запросов много, WinePool/оператор может:
   - связаться с винодельней;
   - собрать новый маршрут;
   - добавить программу;
   - уведомить заинтересованных пользователей.

## 14. UI/UX требования

### 14.1. Публичная карточка винодельни

Блоки:

- hero image/video;
- название, регион, verified/partner badge;
- короткое описание;
- actions: `Маршрут`, `Позвонить`, `Сайт`, `Подписаться`;
- `Вина этой винодельни`;
- `Посетить` - tours/experiences;
- `Локации`;
- `Материалы атласа`;
- отзывы/посещения, позже.

Для editorial/premium страницы UI должен поддерживать:

- модульные секции в произвольном порядке;
- крупные качественные изображения;
- quote-блоки;
- timeline истории;
- карту/терруарный блок;
- галереи;
- wine carousel с пояснениями редактора;
- FAQ;
- custom section без выпуска новой версии приложения, если структура укладывается в `winery_profile_sections`.

Правила:

- не делать hero как маркетинговый баннер "купи";
- тексты информационные;
- партнерский статус отображать аккуратно: `Партнер WinePool`, не "лучший выбор".
- индивидуальность винодельни важнее шаблонной одинаковости: страница должна давать возможность показать "изюминку" хозяйства.

### 14.2. Tourism search

Раздел `Туризм` должен быть не служебным списком виноделен, а полноценной витриной доступных экскурсионных и дегустационных направлений.

Стратегия выдачи: `/tourism` остается wine-first. По умолчанию в общий каталог попадают experiences с `is_wine_related = true`: винодельни, дегустации, винные маршруты, гастро/винные программы, экскурсии с посещением виноделен. Experiences с `is_wine_related = false` не показываются в основной выдаче без отдельного контекстного входа, чтобы WinePool не становился универсальным агрегатором экскурсий.

Основные режимы:

- `Список` - быстрый просмотр всех подходящих туров;
- `Карта` - точки маршрутов, винодельни, точки посадки;
- `Дата` - ближайшие доступные программы;
- `Маршруты` - multi-stop routes.

Верхние быстрые чипы:

- `Сегодня`;
- `Завтра`;
- `С трансфером`;
- `Дегустации`;
- `На винодельню`;
- `Из Ялты`;
- `2-4 часа`;
- `На весь день`.

Фильтры:

- регион/город;
- дата;
- тип: экскурсия, дегустация, маршрут, частный визит;
- есть трансфер;
- длительность;
- доступные места;
- язык;
- price range optional.
- category: `wine`, `gastronomy`, `nature`, `history`, `sea`, `mountains`, `city`, `culture`, `other` для операторских страниц и будущих расширенных фильтров.

### 14.2.1. Управляемые справочники категорий и доступности

Статус на 15.06.2026: MVP CMS получила управляемые поля `tour_kind`, `category_tags`, `departure_city`, `duration_filter`, `availability_tags`, но `category_tags` и `availability_tags` не должны оставаться свободным текстовым вводом для реального наполнения каталога.

Правило данных:

- в БД хранятся стабильные ключи;
- в admin UI, public filters и карточках показываются локализованные подписи;
- русская локаль показывает русские значения, английская - английские;
- неизвестные legacy/custom tags не должны ломать каталог и могут быть скрыты из public filters до ручной нормализации.

MVP category dictionary:

- `tasting` - RU `Дегустации`, EN `Tastings`;
- `winery` - RU `Винодельни`, EN `Wineries`;
- `gastronomy` - RU `Гастротуры`, EN `Food tours`;
- `history` - RU `История`, EN `History`;
- `nature` - RU `Природа`, EN `Nature`;
- `sea` - RU `Море`, EN `Sea`;
- `mountains` - RU `Горы`, EN `Mountains`;
- `family` - RU `Семейный формат`, EN `Family-friendly`;
- `private` - RU `Индивидуальный тур`, EN `Private tour`;
- `other` - RU `Другое`, EN `Other`.

MVP availability dictionary:

- `today` - RU `Сегодня`, EN `Today`;
- `tomorrow` - RU `Завтра`, EN `Tomorrow`;
- `weekend` - RU `Выходные`, EN `Weekend`;
- `week` - RU `На неделе`, EN `This week`;
- `on_request` - RU `По запросу`, EN `On request`;
- `group_forming` - RU `По набору группы`, EN `Group forming`;
- empty/no value - admin label `Не указано`; not a public filter option.

`group_forming` means the tour is possible only after the operator gathers enough demand. In the lead-request pilot it is an honest availability hint, not a promise of a fixed departure, confirmed seat or remaining capacity. Public copy must not imply guaranteed departure until an operator confirms the request.

Future demand counter:

- before slot engine: count `tourism_leads` by experience + requested/preferred date/time label and show this as operator demand, not as public capacity;
- after slot engine: count by `tourism_slots.id` / `departure_at` with `capacity`, `pending_count`, `confirmed_count` and `available_count`;
- any public "group forming" or "places left" indicator must be backed by reliable slot/capacity data and transactional booking rules.

Сортировки:

- ближайшая дата;
- рядом;
- популярные/партнерские, только если disclosure понятный;
- недавно добавленные.

Платное выделение возможно, но только в рамках отдельного `Featured / Partner placement` слоя с маркировкой и legal review. По умолчанию поиск не должен превращаться в непрозрачный аукцион. Пользователь должен понимать, где органическая выдача, а где партнерское/продвигаемое размещение.

Допустимые форматы выделения:

- pinned card в верхней зоне `Партнерский маршрут`;
- featured rail `Рекомендуемые маршруты WinePool`;
- badge `Партнер WinePool`;
- повышенная видимость в curated подборке `Из Ялты`;
- larger media card для premium-партнера;
- отдельный block `Маршруты недели` после editorial approval.

Недопустимые форматы:

- скрытая реклама без маркировки;
- вытеснение всех органических результатов платными;
- формулировки `лучший`, `самый выгодный`, `купите`, `скидка на вино`;
- платное размещение непроверенного тура;
- продвижение карточек без актуального контакта/расписания/точек посадки.

Ranking guardrails:

- paid placement может дать boost, но не должен обходить обязательные фильтры пользователя;
- если пользователь выбрал дату, показывать только релевантные или clearly marked nearby alternatives;
- если пользователь выбрал `с трансфером`, не поднимать тур без трансфера;
- если тур paused/rejected/unpublished, он не показывается независимо от оплаты;
- user trust важнее paid boost.

### 14.2.1. Другие экскурсии оператора

Если provider/operator ведет не только винные маршруты, WinePool должен поддержать вторичный блок `Другие экскурсии оператора`.

Premium showcase rendering rule:

Concept 3 (`Wine Journey Story`) is now the preferred public face for tourism organizations because it supports two important business cases without separate duplicated templates:

- **Winery / estate with one flagship visit**: render the page as a premium winery-facing showcase. Large hero media becomes the face of the winery, the route chain shows the guest journey to the estate/cellar/tasting, and the single flagship tour is enough. If there are no secondary offers, hide `Другие экскурсии оператора` entirely. Do not force an artificial catalog/list layout just because the organization has only one tour.
- **Tour operator / excursion bureau with several routes**: use the same premium hero and one flagship route at the top, then show `Другие экскурсии оператора` or a future compact list/filter mode below. This keeps the page emotional and branded while still scaling to 3-20+ programs.

This means the operator/winery showcase must be conditional by available inventory:

- one wine-related tour -> hero + flagship card + request/my-requests/trusted/legal blocks;
- several wine-related tours -> hero + flagship card + additional wine route cards/list;
- wine + non-wine inventory -> wine-first hero/flagship, then secondary `Другие экскурсии оператора`;
- no secondary inventory -> no empty secondary block or placeholder.

Product rationale: the same surface becomes both (1) the first WinePool Wiki / premium passport layer for wineries and (2) a scalable acquisition page for excursion bureaus. The admin/content model should therefore allow high-quality hero media, route highlights and flagship-tour curation per organization, not only per individual tour.

Где показывать:

- на странице tourism organization / экскурсионного бюро;
- в карточке оператора, если пользователь пришел из конкретного wine-related тура;
- в админке/операторском кабинете при управлении программой;
- в будущем - в отдельной вкладке `Все предложения оператора`.

Где не показывать по умолчанию:

- в главной organic выдаче `/tourism`;
- в подборках `Винные маршруты`;
- в блоках винодельни, если экскурсия не связана с винодельней или дегустацией.

Правила:

- wine-related tours остаются главным направлением и должны быть проработаны от и до первыми;
- не wine-related tours можно хранить, принимать заявки и показывать в operator context;
- визуально отделять блок от основного WinePool wine tourism: заголовок `Другие экскурсии оператора`, без смешивания с винными маршрутами;
- не давать paid placement не wine-related экскурсиям в wine-first выдаче без отдельного продуктового и legal review;
- не использовать такие экскурсии как повод размывать WinePool brand promise.

Карточка тура в списке:

- cover image;
- title;
- route summary: `Ялта -> Массандра -> ...`;
- duration;
- nearest date / `по заявке`;
- pickup summary;
- transfer flag;
- available seats if managed slots enabled;
- price note;
- small map/route icon;
- CTA `Подробнее`.

Пустое состояние:

- если по фильтрам ничего нет, показывать соседние даты/регионы;
- если нет данных в городе, показать `Оставить интерес` или `Сообщить, что ищете тур из этого города`.

Пустое состояние должно быть demand-sensing механизмом, а не тупиком. Если турист не нашел желаемый маршрут, он может оставить пожелание:

- куда хочет поехать;
- из какого города/точки выезда;
- на какие даты;
- сколько гостей;
- интересующий формат: дегустация, винодельня, маршрут на день, семейный формат, премиальный private tour;
- какие винодельни/локации интересуют, если знает;
- телефон/email optional;
- согласие получить уведомление, если такой тур появится.

Эти заявки не являются booking и не обещают организацию тура. Это сигнал спроса для WinePool, оператора и будущих партнеров.

### 14.3. Experience details

Блоки:

- cover;
- title/subtitle;
- winery/provider;
- location/meeting point;
- duration;
- guest range;
- age 18+;
- schedule/available slots;
- included/not included;
- route/stops;
- cancellation policy;
- contact;
- CTA.

Если experience включает трансфер или посадку в автобус/минивэн, дополнительно:

- блок `Точки посадки`;
- список pickup points с временем сбора;
- фото-ориентиры каждой точки;
- CTA `Выбрать точку посадки`;
- предупреждение `Приходите за N минут до отправления`.

CTA labels:

- `Оставить заявку`;
- `Уточнить в WhatsApp/Telegram`;
- `Открыть сайт партнера`;
- `Маршрут`.

Не использовать:

- `Купить`;
- `Заказать алкоголь`;
- `Выпить`;
- `Скидка на дегустацию`.

### 14.4. Lead request sheet / future booking sheet

В Phase 2 это форма заявки, а не гарантированное бронирование. UI должен избегать обещания места до подтверждения оператором.

Шаги:

1. желаемая дата через date picker; слот появляется только после Phase 3;
2. точка посадки, если у маршрута несколько pickup points;
3. количество гостей;
4. имя и контакт: телефон, мессенджер или e-mail;
5. комментарий;
6. подтверждение 18+;
7. отправка.

States:

- loading;
- lead submitted;
- duplicate/active request hint;
- guest registration CTA after successful submit;
- offline/network error.

Future Phase 3 booking states:

- slot no longer available;
- booking submitted;
- already booked;
- auth required for managed booking/payment.

### 14.5. Business dashboard

Разделы:

- today's visits;
- pending requests;
- confirmed bookings;
- vehicle/transport assignments;
- experiences;
- schedules/slots;
- locations;
- profile/publication status;
- analytics.

Для винодельни dashboard должен быть не "админкой ради админки", а платным B2B value center. Винодельня должна понимать, что WinePool приводит людей, помогает не терять заявки и показывает измеримую пользу.

Ключевые виджеты:

- `Сегодня`: ожидаемые визиты, точки посадки, подтвержденные гости, контактные действия.
- `Заявки`: новые, ожидают ответа, подтвержденные, отмененные, просроченные.
- `Транспорт`: назначенные машины, свободные места, изменения, кто из туристов уведомлен.
- `Воронка`: просмотры профиля -> просмотры туров -> клики маршрута/контакта -> заявки -> подтверждения -> check-in -> отзывы/дегустации.
- `Источники`: WinePool app, QR на офлайн-точке, конкретный referral code, партнерский канал, материал Атласа.
- `Маршруты`: какие программы дают больше заявок и реальных визитов.
- `Точки посадки`: какие pickup points чаще выбирают и где бывают проблемы/no-show.
- `Отзывы после визита`: новые отзывы, средняя оценка визита, темы из отзывов после модерации.
- `Контент`: какие секции страницы смотрят, какие вина открывают после профиля.

Минимум в first business analytics slice:

- просмотры профиля;
- просмотры experience;
- route/contact clicks;
- booking requests;
- confirmed bookings;
- check-ins;
- cancellation/no-show;
- referral source breakdown.

## 15. Админская панель

### 15.1. Admin tourism overview

Экран `/admin/tourism`:

- published pilot experiences;
- incoming `tourism_leads`;
- pending experiences;
- pending winery public profiles;
- pending location changes;
- active featured placements;
- new tourism interest requests and demand clusters;
- complaints;
- recently published;
- booking anomalies.

Фильтры:

- status;
- provider type;
- region;
- partner only;
- missing legal fields;
- missing location coordinates.

В pilot-slice `/admin/tourism` уже должен уметь:

- показать опубликованные tourism experiences;
- открыть preview пользовательской карточки;
- показать входящие заявки с именем, контактом, датой, количеством гостей, статусом и комментарием;
- обновляться pull-to-refresh;
- не давать публичному пользователю доступ к контактам заявок.

Admin WinePool - это роль контроля и governance. Он видит все маршруты, все заявки, все организации и может вмешаться, но не обязан быть ежедневным оператором каждой экскурсии.

### 15.1.1. Web-first tourism CRM and partner cabinet

Current execution note (01.09.2026): the shared shell, visual system,
role/entitlement separation and moderation workflow are specified in
`h1_tourism_partner_crm_unified_tz_2026_09_01.md`. This section remains the
platform-level contract; the newer document has precedence for CRM
implementation details.

Решение 15.06.2026: `/admin/tourism` должен стать не только внутренней админкой WinePool, но и будущим кабинетом партнёрских туристических агентств. Один и тот же интерфейс используется в разных permission scopes.

Target layout:

Detailed implementation requirements for the tour editor and shared media
workflow are maintained in
`docs/h1_tourism_crm_tour_editor_tz_2026_09_03.md` and take precedence for this
surface.

- collapsible left tour drawer:
  - search;
  - status filters;
  - organization/all scope depending on role;
  - create tour action;
  - compact tour rows with status, operator, missing-data markers.
- central editor:
  - selected tour title/status/actions;
  - tabs `Основное`, `Фильтры`, `Описание`, `Медиа`, `Маршрут`, `Посадки`, `Авто`, `Заявки`, `Запросы`;
  - `Заявки` содержит только бронирования выбранного тура и группирует их по
    дате без каскада вложенных карточек;
  - `Запросы` содержит общий неудовлетворённый спрос туристов, отдельные
    статусы обработки и агрегаты для будущей аналитики/создания новых программ;
  - form fields are web-first, not hidden inside a narrow bottom sheet.
- right publish-readiness panel:
  - live public card preview;
  - completion checklist;
  - warnings before publish;
  - missing media/legal/route/pickup indicators.

Approved visual reference:

- [WinePool Tourism CRM.png](WinePool%20Tourism%20CRM.png) - web-first hybrid CRM mockup with collapsible tour drawer, tabbed center editor and persistent preview/checklist panel.

Permission modes:

- `WinePool admin / superadmin`:
  - sees all tourism organizations, tours, media, leads and custom tour requests;
  - can create/edit/publish/pause/archive any tour;
  - can assign or reassign operator organization and responsible agent;
  - can see moderation/governance fields, internal notes and cross-organization analytics;
  - can override publication if needed.
- `tourism organization owner / manager`:
  - sees only tours and leads belonging to organizations where the user is an active `tourism_organization_members` member;
  - can create and edit tours for own organization;
  - can manage own media, route stops, pickup points, vehicles and lead operations;
  - can submit for moderation or publish directly depending on future trust policy;
  - cannot see other organizations' contacts, leads, media or internal WinePool governance fields.
- `tourism organization agent`:
  - primarily manages leads, assigned pickup/vehicle/ticket/messages for own organization;
  - may edit operational fields if allowed by organization policy;
  - tour content editing may be disabled or limited to draft changes.
- `viewer` later:
  - read-only access to own organization tours/leads and analytics.

Data/RLS implications:

- UI filters are not security. Supabase RLS/RPC must enforce organization scope for non-admin users.
- `operator_organization_id` on `tourism_experiences` is the ownership boundary for partner-visible tours.
- lead visibility for partner members must be based on `tourism_leads.assigned_organization_id`.
- media, pickup points, route stops and vehicles inherit scope from their parent experience.
- WinePool admin can query all rows through admin capabilities; partner users query only rows in their active organizations.

UX implications:

- in partner mode the left drawer title should read like `Мои туры` or organization name, not `Все туры`;
- organization selector is visible only to WinePool admin or multi-organization users;
- publish button can show `Отправить на проверку` for organizations that require moderation;
- right checklist should distinguish hard blockers from recommendations;
- unavailable tabs/actions should be hidden or disabled with a clear reason, not silently fail.

### 15.2. Experience moderation

Moderator видит:

- title/description;
- provider business;
- destination winery;
- locations/stops;
- media;
- schedule;
- price/payment mode;
- external links;
- legal risk checklist.

Actions:

- approve;
- reject with reason;
- request changes;
- pause published;
- archive duplicate.

### 15.3. Winery public profile moderation

Actions:

- edit canonical fields if admin;
- approve public profile;
- manage winery profile sections;
- preview editorial/premium page before publication;
- reject media;
- link atlas article;
- set featured order.

Admin/editor surface для premium-страниц:

- section builder;
- markdown/body editor;
- media picker;
- linked wines picker;
- linked atlas articles picker;
- preview as user;
- publication checklist;
- changelog/audit.

Для первого среза достаточно admin-only редактора. Self-service редактирование premium-секций винодельней можно добавлять позже через moderated proposals.

### 15.4. Business verification

Для винодельни:

- check business type = winery;
- check `managed_winery_id`;
- check `is_verified`;
- check contacts;
- check locations.

Для туроператора:

- new `business_type = tourism_operator`;
- проверка названия/контактов/документов вне приложения;
- no direct winery ownership.

### 15.5. Operator / tour agent dashboard

Операторский контур не равен админке WinePool. Это рабочее место человека, который отвечает за конкретные экскурсии и должен быстро обработать заявку.

Экран `Мои заявки` / `/business/tourism/leads`:

- список новых заявок по организациям, где пользователь active member;
- фильтры `Новые`, `В работе`, `Подтвержденные`, `Отмененные`;
- карточка заявки: тур, дата, гости, контакт, комментарий, источник, время создания;
- быстрые действия:
  - `Взять в работу`;
  - `Подтвердить`;
  - `Отменить`;
  - `Назначить на меня`;
  - `Открыть карточку тура`;
  - `Открыть точки посадки`;
  - `Связаться`;
- поле `operator_note`, невидимое туристу;
- журнал событий по заявке.

Для MVP можно показывать одного ответственного на организацию. Но backend должен сразу поддерживать несколько участников организации, потому что у одной экскурсионной точки или туроператора в сезон может быть несколько агентов и смен.

#### 15.5.1. Операционная доска при нескольких турах

Плоский список `Входящие заявки` подходит только для первого pilot, когда в системе 1-3 маршрута. Для реальной работы туроператора или экскурсионного бюро экран должен масштабироваться до операционной доски:

- верхний уровень: `Мои туры / маршруты`;
- каждый tour card показывает:
  - название;
  - категорию (`wine`, `gastronomy`, `nature`, `history`, `other`);
  - ближайшие даты;
  - количество новых/в работе/подтвержденных заявок;
  - заполненность по ближайшим выездам, если capacity уже известна;
  - проблемные сигналы: нет машины, нет точки посадки, не хватает мест, много неразобранных заявок;
- внутри тура заявки группируются не только по статусу, но и по `preferred_date` / `slot_date`;
- внутри даты появляется группа выезда:
  - время;
  - vehicle/driver/pickup configuration;
  - capacity/seats taken/seats pending;
  - список заявок этой даты;
  - действия `Назначить машину`, `Объединить группу`, `Пересадить`, `Уведомить туристов`.

Для первого этапа, пока нет slot engine, можно использовать `preferred_date_label` как мягкую группировку. После появления `tourism_slots` UI должен перейти на точные `slot_id`, `departure_at`, `capacity` и `vehicle_assignment_id`.

Если оператор ведет 20+ программ, экран `/admin/tourism` не должен показывать один длинный список заявок. Нужны:

- фильтр по организации, если пользователь состоит в нескольких tourism organizations;
- фильтр по маршруту;
- фильтр по дате;
- фильтр по статусу;
- search по имени/контакту;
- default view `Сегодня / Завтра / Ближайшие`;
- отдельная вкладка `Новые заявки`, чтобы ничего не потерять.

Цель: оператор должен отвечать не на вопрос "какие заявки вообще есть?", а на вопрос "что мне нужно обработать по конкретному выезду и конкретной машине?".

### 15.6. Tourism organizations and members

Не добавлять глобальную роль `tour_agent` в `profiles.role` как основной механизм доступа. `profiles.role` остается крупным системным уровнем (`buyer`, `seller`, `administrator`). Tourism-роль должна жить в membership-таблице.

Модель:

- `tourism_organizations` - туроператор, винодельня, офлайн-точка, внутренний координатор WinePool;
- `tourism_organization_members` - связь обычных пользователей WinePool с organization;
- `member_role` - `owner`, `manager`, `agent`, `viewer`;
- один пользователь может быть агентом в одной организации и обычным buyer в остальном приложении;
- одна организация может иметь одного агента на pilot и несколько агентов позже;
- одна заявка может быть назначена на организацию и конкретного агента.

#### 15.6.1. Multi-context organization/account

Один пользователь и один business context могут совмещать несколько ролей:

- винодельня ведет публичный профиль и WinePool Wiki страницу;
- она же может быть продавцом собственных товаров в будущих commerce-сценариях;
- она же может быть tourism provider: проводить дегустации, принимать заявки, вести расписание и назначать транспорт/точки сбора;
- отдельное экскурсионное бюро может быть tourism operator без статуса винодельни;
- WinePool admin остается governance-ролью и не должен быть ежедневным исполнителем каждой экскурсии.

Пример pilot-модели:

- account `gg@gg.gg`;
- `profiles.role = seller`;
- business `GGG`, type `winery`;
- tourism organization `GGG`, `organization_type = winery`, linked to business `GGG`;
- тот же account может временно быть `agent` в `Экскурсии Ялта`, чтобы обрабатывать внешний pilot route;
- доступ к `/admin/tourism` должен даваться через `tourism_organization_members`, а не через полный `admin` profile role.

Из этого следует:

- в профиле/кабинете пользователь видит несколько рабочих контекстов, а не одну жесткую роль;
- недоступные admin-разделы должны быть скрыты, а не просто вести на home;
- `/admin` для tourism-only пользователя может быть узким операционным хабом с единственным входом `Туризм и заявки`;
- полный WinePool admin dashboard показывается только `administrator/admin`;
- одна учетная запись может управлять winery content, seller surfaces и tourism operations без дублирования логинов.

Пилотная настройка:

1. Создать organization `Экскурсии Ялта` или аналогичную.
2. Привязать одного пользователя как `owner/agent` и `is_default_responsible = true`.
3. Привязать Massandra pilot experience к этой organization.
4. Все новые заявки назначать на эту organization и default responsible member.
5. Admin WinePool сохраняет полный доступ и право переназначения.

## 16. Карта и маршруты

Статус на 15.06.2026: текстовое поле `route_summary` is a public short route preview, not the source of truth for maps. It can show 3-5 human-readable points in cards and operator pages, but map rendering must use structured route/pickup data.

Future route/map data model:

- `tourism_stops` should evolve into the structured source for tour program points: title, description, order, optional photo, optional coordinates, stop type and visit duration;
- `tourism_pickup_points` remains the source for boarding points: address, instruction, landmarks, photo, pickup time and coordinates;
- public map can show route stops, wineries/venues and pickup points as separate visual layers;
- if coordinates are missing, the UI should show the text route and avoid pretending that a precise map route exists;
- customer `Центр поездки` should show the assigned pickup point and vehicle details, while the public tour page should show possible pickup points and example vehicles only.

### 16.1. Использовать текущий map baseline

Уже есть:

- Yandex MapKit;
- receipt map;
- `LocationMapPreview`;
- full-screen map;
- геокодинг через Yandex.

Tourism-layer должен переиспользовать:

- static/mini preview в карточках;
- full-screen map для интерактива;
- external route launcher.

### 16.2. Новые map objects

`TourismMapPoint`:

- `id`
- `type`: `winery`, `tasting_room`, `meeting_point`, `pickup_point`, `tour_office`
- `title`
- `subtitle`
- `latitude`
- `longitude`
- `address`
- `experience_count`
- `next_available_at`
- `is_partner`
- `landmark_text`
- `photo_count`

`TourismRoutePreview`:

- `experience_id`
- `title`
- `duration_minutes`
- `stops`
- `pickup_points`
- `polyline` optional;
- `map_bounds`
- `nearest_slot_at`
- `booking_mode`

### 16.2.1. Pickup point map card

Pickup point на карте должен быть полезен даже без звонка продавцу.

Bottom sheet точки посадки показывает:

- название точки;
- ближайший адрес;
- время сбора для текущего booking/route;
- инструкцию "как найти";
- фото-карусель;
- ориентиры рядом;
- кнопку `Открыть маршрут`;
- кнопку связи с организатором.

Если точка относится к конкретному билету, UI должен явно писать:

> Ваша точка посадки

Если пользователь смотрит маршрут до бронирования:

> Возможная точка посадки

### 16.2.2. Pickup point photo requirements

Фотографии должны помогать ориентироваться на местности:

- общий вид точки;
- вид с той стороны, откуда обычно идет турист;
- ближайшая вывеска/магазин/кафе;
- перекресток или остановка;
- место, где останавливается автобус/минивэн.

Фото открываются fullscreen через общий viewer. Для слабого интернета нужен thumbnail/cache, но это не блокер первого pilot.

### 16.3. Route behavior

Phase 1:

- external link to Yandex Maps/Navigator;
- fallback web URL;
- no route calculation inside app.
- tourism detail can show route preview from stored stops/pickup points even without turn-by-turn navigation.

Phase 2:

- show travel hints;
- show distance from city center if user granted approximate location;
- route preview line for multi-stop route if polyline is provided manually or by API.

Phase 3:

- saved wine routes;
- shareable route card;
- offline-friendly route details.

## 17. Уведомления

Использовать существующий `app_notifications` + email timer baseline.

Events:

- booking requested -> business;
- booking confirmed -> user;
- booking rejected -> user;
- booking cancelled -> counterparty;
- vehicle/driver changed -> user;
- pickup time/location changed -> user;
- reminder 24 hours before;
- reminder 2 hours before optional;
- post-visit review prompt;
- business daily digest.

Push/email copy must be neutral:

- `Ваша заявка на экскурсию подтверждена`;
- `Напоминание о визите на винодельню`;
- `Изменилась машина для вашего маршрута`;
- `Изменилась точка или время посадки`;
- no promotional alcohol wording.

## 18. Analytics

Consumer events:

- `winery_profile_viewed`
- `tourism_search_opened`
- `tourism_experience_viewed`
- `tourism_route_opened`
- `tourism_booking_started`
- `tourism_booking_submitted`
- `tourism_booking_confirmed_seen`
- `tourism_checkin_completed`
- `tourism_post_visit_prompt_shown`
- `tourism_visit_tasting_created`
- `tourism_visit_review_created`

Business metrics:

- profile views;
- experience views;
- route clicks;
- contact clicks;
- booking requests;
- confirmation rate;
- cancellation rate;
- check-in / show-up rate;
- no-show rate;
- post-visit review rate;
- visit-to-tasting conversion;
- referral source conversion;
- pickup point usage;
- route stop interest;
- winery follow/save count.

### 18.0.1. Pilot launch analytics and referral baseline

Tourism pilot is not only a feature; it is a new acquisition channel. The first agent/QR launch must be measured separately from generic RuStore traffic.

Required first-pilot counters:

- QR scans / deep-link opens per tour and referral source;
- landing/tour card views from agent QR;
- app installs attributable to referral where technically possible;
- guest lead submissions from referral;
- auth conversion after tourism lead;
- notification opt-in / notification seen for confirmed lead;
- ticket opened / offline ticket saved;
- route/pickup point opened before departure;
- post-visit return actions: receipt scan, tasting note, review, check-in.

Baseline from pre-tourism public traffic 04.06.2026:

```text
RuStore: 690 page views -> 10 installs -> 0 registrations
YouTube Shorts: 901 views, 47.9% watched to end
```

Product hypothesis: tourism QR traffic should convert better than broad advertising because the tourist has an immediate reason to install: ticket, pickup point, vehicle photo, operator messages and trip reminders.

### 18.1. B2B analytics для виноделен

Это отдельная коммерческая ценность WinePool. Винодельни и операторы платят не только за красивую страницу, но и за измеримость: сколько людей увидели профиль, откуда пришли заявки, сколько туристов реально доехало, какие программы работают.

Воронка:

```text
profile_view
  -> experience_view
  -> route/contact/booking_start
  -> booking_submitted
  -> booking_confirmed
  -> checkin_completed
  -> post_visit_review / tasting_created / follow
```

Основные отчеты:

- `Обзор периода`: неделя/месяц/сезон.
- `Заявки и визиты`: requested, confirmed, checked-in, cancelled, no-show.
- `Источники`: WinePool search, winery page, tourism catalog, QR/referral, Atlas article, partner media.
- `География интереса`: город пользователя только агрегированно, если доступно и privacy-safe.
- `Туры`: какие experiences собирают просмотры, заявки и реальные check-in.
- `Точки посадки`: какие pickup points используются, где выше no-show.
- `Контент`: какие блоки страницы и вина открывают.
- `Отзывы после визита`: публичные отзывы и агрегированные оценки.

Уровни доступа:

- `Free/Basic`: очень кратко - просмотры профиля и количество заявок.
- `Verified`: заявки, подтверждения, check-ins, базовые источники.
- `Partner`: полная воронка, referral breakdown, pickup point analytics, отзывы.
- `Premium`: сезонные отчеты, рекомендации по контенту/маршрутам, export CSV/PDF, ручной monthly summary от WinePool.

Privacy rules:

- не отдавать сырые пользовательские заметки;
- не отдавать контакты пользователей вне конкретной заявки;
- не показывать user-level поведение вне owned bookings;
- источники и география только агрегированно;
- выгрузки должны проходить privacy review.

### 18.2. B2B monetization packages

Пакеты для виноделен могут объединять страницу и аналитику:

- `Profile Cleanup`: оформление базовой страницы + базовые просмотры.
- `Editorial Passport`: редакционная страница + profile analytics.
- `Tourism Ready`: туры, точки посадки, заявки + booking funnel.
- `Premium Partner`: расширенная страница, tourism funnel, referral analytics, сезонные отчеты.

Пакеты также могут включать entitlement на повышенную видимость в разделе `Туризм`:

- `Profile Cleanup`: без boost, только привести страницу в порядок.
- `Editorial Passport`: усиление доверия через качественную страницу, без обязательного tourism boost.
- `Tourism Ready`: участие в региональных tourism-подборках при наличии published experiences.
- `Premium Partner`: расширенная видимость в `/tourism`, featured-блоки, premium-карточки и route collections при прохождении moderation/legal review.

Ценообразование фиксируется отдельно, но продуктовая логика такая: WinePool продает не рекламу алкоголя, а качественное информационное присутствие, lead/request инфраструктуру, аналитику привлеченного интереса и пакетную видимость внутри справочно-туристического раздела.

### 18.3. Featured tourism placement

Раздел `Туризм` может стать отдельной монетизационной поверхностью. Базовая модель: повышенная видимость не продается как разовая рекламная кнопка, а является частью B2B-пакета винодельни/оператора. То есть ранжирование, featured-блоки и расширенные карточки зависят от того, какой пакет оплатила и какой статус получила винодельня, но только при соблюдении quality/legal guardrails.

Продуктовая модель:

- paid visibility продается как часть информационно-партнерского пакета: `Verified`, `Partner`, `Premium`;
- рекламируем не алкоголь и не покупку вина, а посещение/экскурсионную программу/страницу винодельни;
- каждый promoted item проходит moderation/legal copy checklist;
- пользователь видит disclosure: `Партнерский маршрут`, `Партнер WinePool`, `Продвигаемое размещение` - конкретная формулировка выбирается после legal review;
- performance считается по impressions, clicks, route opens, booking starts, submitted requests, confirmed visits, check-ins.

Package entitlements:

- `Free/Basic`: органическая выдача, без платного boost; базовая карточка.
- `Verified`: доверительный badge, участие в органической выдаче выше непроверенных при равной релевантности.
- `Partner`: partner badge, возможность попадать в featured rail по региону/городу, расширенная карточка тура, referral analytics.
- `Premium`: максимальная доступная видимость в рамках guardrails: top featured card, premium media card, curated route collection, highlighted map pin, seasonal placement.

Важно: пакет дает право на повышенную видимость, но не гарантирует показ вопреки фильтрам пользователя, качеству данных или legal review. Если тур нерелевантен дате/городу/типу запроса, пакет не должен проталкивать его наверх.

Форматы:

- top pinned card in `/tourism`;
- featured carousel by city/region;
- premium card in route list;
- sponsored route collection, e.g. `Винные маршруты из Ялты`;
- highlighted map pin;
- editorial selection with explicit partner disclosure;
- seasonal placement, e.g. `Июньские дегустации`.

Ограничения:

- не продвигать unpublished/paused/rejected experiences;
- не продвигать experiences без понятных контактов, маршрута или точки посадки;
- не продвигать карточки с юридически рискованными формулировками;
- не показывать paid result выше явно нерелевантного фильтрам пользователя;
- не продавать "первое место навсегда";
- ограничить долю paid cards на первом экране, чтобы раздел оставался полезным.

Рекомендуемые правила выдачи:

- organic relevance score first: date, location, type, availability, quality score;
- partner boost second;
- diversity rule: не более 1-2 sponsored cards подряд;
- quality floor: paid boost работает только если заполнены фото, описание, маршрут, контакты, pickup points where needed;
- clear labels in UI.

Admin controls:

- package level and entitlement status;
- featured window start/end if placement is seasonal;
- promoted entity allowed by package: winery, experience, route collection;
- target region/city;
- target dates;
- package/manual override;
- disclosure label;
- preview in `/tourism`;
- pause featured placement;
- performance report.

Open legal checkpoint:

- финальная формулировка disclosure;
- можно ли использовать слово `реклама` или лучше `партнерский материал` / `партнерский маршрут`;
- какие ограничения нужны для размещений, связанных с дегустациями;
- требуется ли отдельная маркировка интернет-рекламы через ОРД/ЕРИР, если placement будет квалифицирован как реклама.

Admin metrics:

- pending moderation count;
- average moderation time;
- rejected content reasons;
- stale requests.
- featured placement impressions/clicks/booking starts;
- campaign conversion by region/source.
- interest requests by city/region/type;
- demand clusters converted into new experiences.

Privacy:

- business analytics are aggregated;
- no raw user notes;
- no export of unrelated user behavior.

## 19. Интеграция с существующими разделами

### 19.1. Wine details

Add block:

- `О винодельне`;
- `Посетить винодельню`, if published tourism exists;
- `Маршрут`, if location exists.

### 19.2. Atlas

Atlas article can link to:

- winery profile;
- region tourism map;
- curated route;
- related experiences.

### 19.3. Map

Existing map remains purchase map.

Add separate mode/entry:

- `Моя карта`;
- `Сообщество`;
- `Винодельни`, if feature flag enabled.

Do not mix purchase points and tourism points without clear segment control.

### 19.4. Profile

Add:

- `Мои визиты`;
- `Избранные винодельни`;
- `Заявки на экскурсии`.

### 19.5. Reviews / Cellar

After check-in:

- prompt to add tasting note;
- link review to `tourism_booking_id` optional;
- show "пробовал на винодельне" context if user consents.

## 20. Модерация контента

Каждый published object должен проходить checklist:

- title does not contain direct alcohol sale CTA;
- description informational;
- media rights confirmed;
- no images of minors;
- no health claims;
- no direct remote alcohol sale;
- contacts are valid;
- coordinates valid;
- provider verified enough for public listing;
- cancellation/contact terms present;
- age 18+ visible.

Content status flow:

```text
draft -> submitted -> approved -> published
                   -> rejected
published -> paused -> published
published -> archived
```

Trusted partner direct publish может быть добавлен позже, но по умолчанию все public changes go through moderation.

## 21. Безопасность и антиабуз

Риски:

- spam bookings;
- fake businesses;
- scraping contacts;
- malicious external URLs;
- overbooking;
- no-show abuse;
- partner dispute.

Mitigations:

- rate limit booking creation per user/device/IP;
- require auth for booking;
- email/phone validation for repeated bookings;
- external URL allowlist/scan;
- transaction lock on slot booking;
- audit events;
- admin pause;
- no public display of user's phone/email;
- business owner can only see contacts for bookings on owned experiences.

## 22. Data migration and seed strategy

### 22.1. Existing wineries

For wineries already in catalog:

- create empty `winery_public_profiles` lazily;
- do not publish automatically;
- show minimal winery block only from existing `wineries` fields;
- `Посетить` appears only when published tourism data exists.

### 22.2. Crimea/Yalta pilot

Pilot dataset:

- 5-10 wineries in Crimea;
- 3-5 routes from Yalta/nearby tourist points;
- 10-20 pickup points around Yalta tourist zones with coordinates, instructions and photos;
- 1-2 tourism operators or admin-managed routes;
- QR/referral codes for offline excursion sheets.

Pilot import can be CSV-backed:

- wineries;
- locations;
- experiences;
- schedules/manual slots;
- referral source names.

### 22.3. Data quality

Minimum publishable winery:

- name;
- region/country;
- short description;
- one location or website/contact;
- at least one media asset or logo optional;
- publication status.

Minimum publishable tourism experience:

- provider;
- title;
- type;
- description;
- location or meeting point;
- age limit 18+;
- booking/contact mode;
- status published.

## 23. Implementation phases

### Phase 0. Documentation, flags, legal copy

Goal: prepare foundation.

Tasks:

- add this doc to documentation map;
- define feature flags;
- legal/copy checklist;
- decide whether `tourism_operator` business type is required in first migration;
- prepare pilot data template.

DoD:

- architecture accepted;
- no code path exposes unfinished feature to public users.

### Phase 1. Public winery profiles and tourism locations

Goal: пользователь может открыть винодельню и увидеть tourism-ready locations.

Scope:

- `winery_public_profiles`;
- `winery_profile_sections` for editorial/premium-ready layout;
- expanded `locations`;
- public winery profile screen;
- `Винодельня` block in wine details;
- map preview and route CTA;
- admin profile/location moderation.
- admin-only section editor baseline.

No booking yet.

DoD:

- published winery opens from wine details;
- editorial winery page can render at least hero, story, gallery, key wines and visit sections;
- location map works;
- route opens external navigation;
- unpublished wineries are not exposed as full public pages.

### Phase 2. Experiences as informational cards and external/lead CTA

Goal: опубликовать экскурсии без сложного slot engine.

Scope:

- `tourism_experiences`;
- `tourism_experience_stops`;
- media;
- public wine-first tourism search;
- `experience_category` and `is_wine_related` fields;
- list/map/date modes for `/tourism`;
- route preview in experience detail;
- experience detail;
- CTA modes: external URL, phone/messenger, lead request without slots;
- referral source tracking.
- secondary operator inventory model for non-wine excursions, without showing them in the main wine-first `/tourism` feed by default.

DoD:

- Yalta pilot can list excursions;
- tourist can open `/tourism`, compare wine-related tours and see route maps/stops before choosing a winery;
- user can open and inquire;
- admin can pause content.
- non-wine operator excursions can be stored as future/secondary inventory without diluting WinePool's wine tourism catalog.

### Phase 2A. Yalta pilot data and public lead form

Goal: проверить спрос на 1-3 реальных экскурсионных направлениях из Ялты без оплаты и без slot engine.

Scope:

- `tourism-media` bucket;
- `tourism_experiences`;
- `tourism_stops`;
- `tourism_pickup_points`;
- `tourism_vehicles`;
- `tourism_media_assets`;
- `tourism_leads`;
- `/tourism`;
- `/tourism/:id`;
- lead request sheet;
- `/admin/tourism` with incoming leads.

DoD:

- pilot route opens in `/tourism`;
- real media loads from Supabase Storage, not bundled APK assets;
- tourist can submit a lead;
- lead is stored in Supabase;
- admin can see incoming lead;
- feature flag intentionally enabled for internal pilot.

Current status 02.06.2026:

- Massandra/Yalta pilot route seeded;
- storage media uploaded;
- test lead flow verified end-to-end;
- admin lead view verified;
- `tourism_leads` stores basic fields and has been extended with organization assignment/status workflow for the pilot.

Current status 03.06.2026:

- guest and authenticated tourist lead flow verified;
- guest post-submit registration CTA added;
- authenticated tourist receives personal in-app notification for submitted tourism lead;
- authenticated tourist receives personal in-app notification when lead status changes to `В работе`, `Подтверждена` or `Отменена`;
- operator/admin receives in-app notification for new tourism lead;
- `gg@gg.gg` configured as multi-context account: seller/winery business `GGG`, tourism organization owner for `GGG`, and pilot agent for `Экскурсии Ялта`;
- tourism-only operator access to `/admin` and `/admin/tourism` enabled without full WinePool admin rights;
- `/admin` hides inaccessible full-admin actions for tourism-only users;
- `/admin/tourism` now has pilot grouping by tour and preferred date, plus quick status filters `Все`, `Новая`, `В работе`, `Подтверждена`, `Отменена`;
- `/admin/tourism` now has lead details sheet with full lead data, operator note, quick status actions and event history;
- lead details can store pilot trip fields: assigned pickup point, assigned vehicle, pickup time label and tourist memo;
- pickup date/time is selected through date/time picker, and tourist memo is prefilled from tour pickup instructions while remaining editable;
- authenticated tourist can see latest own lead and trip details on the tour card; trip field changes create customer in-app notification;
- customer lead card shows assigned agent, responsible organization and booking contact when available;
- authenticated tourist has `/tourism/my-requests` list with all own tourism leads and trip details; entry points exist in tourism section and profile;
- `/tourism/my-requests` has customer-side quick filters `Активные`, `Все`, `Новые`, `В работе`, `Подтверждены`, `Отменены`, plus nearest trip highlight for accumulated request history;
- customer can open lead details from `/tourism/my-requests`; the details surface shows status, trip data, responsible contacts and attached photo ticket;
- operator can attach or replace a lead photo ticket after offline/remote payment; the file is stored in Supabase Storage under `tourism-media/lead-tickets/...` and is visible to the tourist for boarding;
- tourist can download/save a self-contained offline HTML ticket: embedded photo ticket plus memo with tour, status, pickup time, pickup point, vehicle, agent/organization and recommendations;
- customer can send clarification/change/cancel messages from own lead details; messages are written to `tourism_lead_events`, notify the responsible operator and deep-link to the exact lead in `/admin/tourism?leadId=...`;
- operator can answer from lead details; customer receives an in-app notification that deep-links to `/tourism/my-requests?leadId=...` and opens the exact request instead of the generic tour card;
- customer and operator lead details include a lightweight conversation block with role/name labels and message input, plus expandable operational history: recent events are shown by default, full history opens on demand;
- open customer lead details refresh when a related notification arrives, so the mini-chat updates without forcing the tourist to leave and reopen the screen;
- operator notification focus and customer notification focus are verified for large queues: the UI scrolls/highlights the target lead card after navigation;
- current UI is still a lightweight pilot operator board and must evolve into slot/departure/vehicle board before scaling to many tours, dates and cars.

Current status 04.06.2026:

- public QR-friendly route alias added: `/tourism/o/:operatorSlug/t/:tourSlug`;
- route alias renders the same `TourismExperienceDetailsScreen` by `tourSlug`, not a duplicated landing page;
- `?ref=...`, `?referral=...` and `?source=...` query values are passed into the lead request flow;
- migration `20260604_add_tourism_lead_referral_fields.sql` applied on production self-host Supabase: `tourism_leads.referral_source`, `referral_code`, `referral_url`, with indexes for source/code analytics;
- first MVP supports exact tour QR: `winepool.ru/tourism/o/yalta-excursions/t/yalta-massandra-tasting?ref=booth_01`, assuming web hosting rewrites this URL to the Flutter web app.
- authenticated tourism deep links are treated as stable public product routes and must not fall back to `/buyer-home` during session/profile restore;
- public tourism QR routes bypass onboarding after age confirmation; the age gate remains mandatory, but an already age-confirmed guest must land on the requested operator/tour page instead of restarting onboarding;
- public operator showcase route added: `/tourism/o/:operatorSlug`;
- migration `20260604_add_tourism_organization_public_slug.sql` applied on production self-host Supabase: `tourism_organizations.slug`, public read policy for active slugged organizations, and pilot slug `yalta-excursions`;
- operator showcase reads the same `tourism_organizations` and `tourism_experiences.operator_organization_id` data as app/admin surfaces; it does not duplicate tour content;
- Product Design pass applied to the operator showcase: mobile-first QR entry, real hero image from the first available tour, clear operator identity, compact post-request value strip, improved wine-tour cards and a restrained secondary block for non-wine inventory;
- operator showcase preserves `ref`/`referral` and optional `source` query values when opening a concrete tour, so booth/agent QR attribution is not lost before lead submission;
- pilot alias supported: `yalta-excursion` is normalized to canonical organization slug `yalta-excursions`, so a singular/plural QR typo does not show an empty operator page;
- selected showcase direction: Concept 3 (`Wine Journey Story`) as the emotional/premium public face, Concept 1 (`QR clarity`) as the practical request layer, Concept 2 (`Operator itinerary board`) deferred as a future compact list/filter mode for 3-5+ tours;
- operator showcase hero now leads with `Винные маршруты из Ялты`, keeps operator identity underneath, shows a compact flagship route timeline and uses a clearer tour CTA;
- operator showcase was aligned closer to the selected premium reference: transparent top controls over a large remote hero image, icon route chain `Ялта -> Массандра -> Музей -> Дегустация -> Магазин`, flagship tour card with vertical media/CTA, legal note, `Мои заявки`, secondary operator excursions and trusted operator block;
- organization brand fields added for public showcase: `tourism_organizations.website_url` and `logo_url`; the first Yalta partner uses the public identity `Едувялту`, label `Центр экскурсий`, website `eduvyaltu.ru` and a logo stored in Supabase Storage instead of bundled app assets;
- Yalta operator hero copy changed from generic `Винные маршруты из Ялты` / service promise line to partner-facing `Экскурсии из Ялты по Крыму` plus a visible external website link. This keeps WinePool as the lead/request layer while giving the paid partner a proper branded face;
- concrete tour page includes a compact lead/request flow hint near the top: no in-app payment, operator confirmation, then pickup/vehicle/ticket/messages in the request.

Current status 06.06.2026:

- Product Design pass applied to `/tourism`: premium discovery hero, compact sticky search/filter/tab block, animated catalog title by tour mode/region, route/operator/map tabs, flagship route card, verified operators and compact route list;
- tour type filter added: default `Винные`, optional `Все` and `Другие`; region filter is multi-select and drives title copy (`... России`, `... Крыма`, etc.);
- tourism media slots split explicitly: global home hero images by mode (`tourism_home_hero_images.hero_mode = wine/all/other`), organization hero image (`tourism_organizations.hero_image_url/storage_path`) and tour cover (`tourism_experiences.cover_image_url/storage_path`);
- this removes the pilot ambiguity where the same cover/gallery photo could appear on the main tourism page, operator showcase and concrete tour card for different semantic purposes;
- migration `20260606_add_tourism_image_slots.sql` applied on production self-host Supabase via VPS: `tourism_organizations.hero_image_url`, `hero_image_storage_path`, table `tourism_home_hero_images`, public read RLS for active rows and catalog-admin manage policy;
- current Flutter UI reads configured home hero slots and organization hero slot, with cover/gallery fallback only for pilot continuity while slots are empty;
- production pilot media slots filled: `home/wine/hero_barrels_20260606.png` for `/tourism`, `organizations/yalta-excursions/hero/eduvyaltu_operator_hero_20260606.jpg` for `Едувялту`, and `experiences/yalta-massandra-tasting/cover/massandra_tour_cover_20260606.jpg` as approved tour cover;
- future admin/operator UI must expose upload/replace controls for these slots instead of requiring manual SQL or bundled assets.

Immediate future work after 06.06.2026:

1. Implement `Центр поездки` as the primary customer surface for an existing tourism lead: route `/tourism/trip/:leadId`, status, pickup, vehicle, memo, ticket photo, paper-ticket delivery status, responsible agent/organization and customer/operator messages.
2. Add optional paper ticket delivery request for Yalta pilot: customer request fields, operator confirmation/status/fee, event history and customer notifications.
3. Continue Product Design pass for the concrete tour page: separate tour hero, route/program, pickup points, vehicle block, lead form, request status and CTA into `Центр поездки`.
4. Prepare Yalta QR/launch kit and validate referral preservation from QR to lead.
5. Add admin/operator media controls for tourism: upload, replace, preview, moderation status, copyright confirmation, recommended aspect ratio and file size.
6. Move from pilot fallback logic to explicit required media validation before publication: route card must have cover, operator showcase must have hero, home hero modes must have global images.
7. Convert decorative map tab into data-backed route/map surface: region points, route stops, pickup points and operator coverage.
8. Make tourism filters real data filters end-to-end: kind, region, operator, departure city, transfer, duration, date/range and future slot availability.

### Phase 2B. Operator assignment and lead workflow

Goal: превратить входящую заявку в минимально управляемый операционный процесс.

Scope:

- `tourism_organizations`;
- `tourism_organization_members`;
- lead fields: `assigned_organization_id`, `assigned_user_id`, `operator_note`, timestamps;
- lead event history;
- operator notification on new lead;
- authenticated tourist notification on lead status change;
- operator dashboard `Мои заявки`;
- admin reassignment;
- statuses `new`, `contacted`, `confirmed`, `cancelled`;
- one default responsible user per organization for pilot, multi-agent support in schema.
- narrow operator access to `/admin/tourism` without full WinePool admin;
- hide unavailable admin actions for tourism-only users;
- pilot grouping by tour and preferred date;
- quick status filters for lead queue cleanup;
- lead details sheet with contact, date, assignment, comment, operator note and event history;
- pilot trip fields on lead: assigned pickup point, assigned vehicle, pickup time label, tourist memo;
- date/time picker for pickup time and editable tourist memo prefilled from pickup point address, landmarks, instruction and default timing;
- customer-side latest lead card on tour details;
- customer notification when pickup point, vehicle, pickup time or tourist memo changes;
- customer-visible responsible contact: assigned agent, organization, tour booking contact;
- customer "My requests" screen with statuses, trip details and CTA back to tour;
- customer "My requests" filters by request status and highlights nearest active trip;
- customer lead details sheet with request facts, trip facts, support context and photo ticket;
- lead ticket photo upload for operator with Storage-backed URL, customer notification and event history entry;
- ticket offline save action from customer surfaces; desktop web opens native "Save as" dialog for self-contained HTML ticket, mobile saves ticket image to gallery album `WinePool` and prepares HTML memo through system share/save flow;
- document and prepare transition from pilot grouped lead list to full slot/departure operator board.

DoD:

- new lead auto-assigns to organization/default responsible;
- responsible agent receives notification;
- authenticated tourist receives in-app notification when operator changes lead status to `contacted`, `confirmed` or `cancelled`;
- agent can mark lead `В работе`, `Подтверждена`, `Отменена`;
- admin can see and override assignment/status;
- several agents per organization can be added later without schema rewrite.
- tourism-only account sees only accessible tourism operations in admin hub;
- existing pilot can be tested under a non-admin operator account.
- operator can filter incoming leads by status without leaving `/admin/tourism`;
- operator sees lead groups by tour and requested date in the pilot UI.
- authenticated tourist can filter own request history by status and quickly open nearest active trip.
- operator can open lead details, update internal note and see status/note history.
- operator can assign pickup point, vehicle, pickup time and tourist memo before full booking engine.
- operator can choose pickup date/time from picker and quickly start tourist memo from route defaults.
- operator can attach a photo ticket to a lead; authenticated tourist can open the photo ticket from own request details and tour card.
- authenticated tourist can save ticket photo and trip memo for offline boarding.
- authenticated tourist sees latest own request and trip details in tour details screen.
- authenticated tourist receives in-app notification when operator updates trip details.
- authenticated tourist sees who is responsible for the request and how to contact the organizer.
- authenticated tourist can open a single list of all own tourism requests from tourism and profile.

### Phase 2C. Customer-side lead actions and conversation

Goal: дать туристу не только смотреть заявку, но и управляемо реагировать, если планы изменились или появились вопросы. На pilot-этапе это не полноценный messenger, а аккуратный lead conversation: короткие сообщения, события истории и точные уведомления в нужную заявку.

Scope:

- customer message input in `/tourism/my-requests` lead details;
- `Уточнить заявку`: короткое сообщение оператору внутри lead conversation/history;
- `Отменить заявку`: customer cancellation request event before full booking engine;
- `Изменить дату/количество гостей`: request-change event before full booking engine;
- customer-visible status/explanation after action;
- operator message input in `/admin/tourism` lead details;
- operator and customer notifications with exact lead deep-links;
- event types: `customer_clarification`, `customer_change_requested`, `customer_cancel_requested`, `operator_message`;
- actor labels: role plus profile/display name where available, e.g. `Турист, Михаил`, `Оператор, Алексей`;
- expandable history in both customer and operator details: show last 3-4 events by default, full audit trail on demand;
- realtime-ish refresh: app notification stream invalidates the open `tourismLeadEventsProvider(leadId)`;
- optional external contact CTA if communication пока идет по телефону/мессенджеру.

DoD:

- authenticated tourist can send clarification from own lead details;
- operator sees customer clarification in lead details/event history;
- cancellation/change request is not lost and creates notification for responsible agent;
- operator can send an answer from lead details and tourist sees it in own request details;
- tourist notification opens `/tourism/my-requests?leadId=...` and auto-opens/highlights the exact request;
- operator notification opens `/admin/tourism?leadId=...` and auto-opens/highlights the exact lead;
- recent history and full history are both available to operator and tourist from the UI, not only from the database;
- open customer lead details refresh after a related notification without manual navigation;
- guest leads remain contact-only unless the guest later logs in and lead is linked.

### Phase 3. Managed bookings and manual slots

Goal: подтвержденные места внутри WinePool без оплаты.

Scope:

- `tourism_slots`;
- `tourism_bookings`;
- transport assignments for routes with transfer;
- booking sheet;
- business booking dashboard;
- operator board grouped by route, date, slot/departure, vehicle and lead status;
- confirm/reject/cancel flows;
- vehicle/driver change notifications;
- notifications;
- booking status history.

DoD:

- lead can be converted into booking when a place is confirmed;
- user sees confirmed visit;
- capacity cannot be overbooked in transactional RPC.
- operator can change assigned vehicle and user receives timely notification.
- operator can see all requests for one departure date as one operational group.

### Phase 4. Business self-service for wineries/operators

Goal: партнеры сами ведут расписание и контент, но публикация модерируется.

Scope:

- business tourism dashboard;
- experience editor;
- schedule/slot editor;
- media upload;
- profile proposals;
- moderation queue.

DoD:

- business owner can create draft experience;
- admin approves;
- published card appears publicly.

### Phase 5. Tourism map and route discovery

Goal: отдельный discovery слой на карте.

Scope:

- tourism map mode;
- filters by region/city/type/date;
- meeting point and route stop display;
- pickup point display with photos, landmarks and exact user-assigned boarding point;
- route CTA;
- saved/favorite route;
- pilot "Ялта wine routes" collection.
- empty-state interest request flow.

DoD:

- user can find winery tourism options by map;
- user can open their ticket and confidently find the pickup point without separate messenger instructions;
- user can leave a wish request if no suitable tour exists;
- purchase map and tourism map remain visually separated.

### Phase 6. Check-in, post-visit loop and analytics

Goal: close full loop.

Scope:

- QR check-in;
- badge;
- create tasting/review from visit;
- business analytics;
- B2B winery dashboard with funnel, sources, show-up/no-show and referral analytics;
- referral reports.

DoD:

- confirmed user can check in;
- visit can become tasting/review;
- business sees aggregate funnel.
- winery can see how many visitors WinePool attracted and which routes/sources converted.

### Phase 7. Payments, if legal review approves

Goal: optional future paid booking for service, not alcohol sale.

Scope:

- payment provider decision;
- receipt/fiscalization/legal copy;
- refunds;
- payment status;
- App Store/RuStore rules review.

DoD:

- legal opinion stored;
- payment flow approved by store policy;
- no alcohol sale/delivery in app.

## 24. Acceptance checklist

### 24.1. Consumer

- Guest can view published winery after age gate.
- Guest can submit Phase 2 lead request with name and contact.
- Guest sees post-submit registration CTA with clear benefits, not a blocking wall.
- Auth user can submit lead request with profile data prefilled where available.
- Auth user sees own tourism requests in `/tourism/my-requests`.
- Auth user sees latest own request, responsible contact, trip details and ticket photo in tour card/request details.
- Auth user can save offline ticket/memo from customer surfaces.
- Auth user can send a clarification/change/cancel message from own request details and see operator answers in the same request.
- Auth user can open a notification about an operator answer and land on the exact request details, not a generic screen.
- Auth user can expand full request history from request details.
- Managed booking can require auth only in Phase 3+.
- Route opens external map.
- All empty/loading/error states are localized.
- No text uses direct alcohol purchase CTA.

### 24.2. Business

- Business owner/operator sees only own experiences/leads/bookings.
- Business owner cannot publish without moderation.
- Business owner cannot bind arbitrary winery.
- Lead status changes, trip detail changes and ticket photo changes write event history.
- Customer messages and operator answers write event history and are visible in lead details.
- Operator can open a notification about a customer message and land on the exact lead details.
- Slot capacity cannot go negative or over capacity in Phase 3 managed booking.
- Tourism-only operator can work with `/admin/tourism` without full WinePool admin rights.

### 24.3. Admin

- Admin can see pending tourism content.
- Admin can approve/reject/pause.
- Admin can inspect media and external links.
- Admin can see provider/winery binding.
- Audit trail exists for publication decisions.

### 24.4. Privacy/security

- User contact is hidden from public.
- Business/operator sees contacts only for its own leads/bookings.
- External links are sanitized.
- RLS tests cover user/business/admin boundaries.
- Account deletion cascades or anonymizes lead/booking personal data according to privacy policy.

## 25. Test plan

Unit tests:

- lead status transitions;
- booking capacity calculation for Phase 3;
- experience filter parsing;
- route URL builder;
- legal copy validator basic forbidden words list.

Repository/RPC tests:

- create lead as guest with contact;
- create lead as authenticated user and bind `created_by`;
- customer can read only own leads;
- operator can read/update only leads for assigned organization;
- ticket photo storage path is readable by the owning tourist and responsible operator/admin;
- lead event is written on status, trip fields, ticket photo updates, customer messages and operator answers;
- customer can create only own lead action events;
- operator can create answer events only for leads assigned to an accessible tourism organization;
- create booking as user in Phase 3;
- cannot book unpublished experience;
- cannot overbook slot in Phase 3;
- business cannot access other business lead/booking;
- admin can pause experience;
- user can cancel own booking only in allowed statuses in Phase 3.

Widget tests:

- winery profile screen;
- experience details;
- lead request sheet;
- my tourism requests list and filters;
- customer lead details with trip fields, responsible contacts and ticket photo;
- customer lead conversation input, exact notification focus and expandable history;
- operator lead card/details/history;
- operator lead conversation input, exact notification focus and expandable history;
- admin moderation card.

Manual QA:

- Yalta pilot flow from QR/referral;
- guest submits lead and receives clear success/registration CTA;
- authenticated tourist receives notification after lead submit/status/trip/ticket updates;
- operator account `gg@gg.gg` sees tourism hub and assigned leads without full admin powers;
- `/admin/tourism` grouping by tour/date and filters by status;
- confirmed lead shows exact pickup point, pickup time, photos, landmarks and route button;
- change assigned vehicle/license plate/driver after confirmation and verify user notification + updated trip card/ticket;
- when vehicle changes, customer UI shows new vehicle photos and recognition hints, not stale previous vehicle media;
- operator attaches/replaces photo ticket and tourist sees it in request details and tour card;
- desktop web ticket save opens native Save As dialog for HTML memo;
- mobile ticket save stores photo in gallery album `WinePool` and prepares HTML memo through system share/save flow;
- tourist sends a clarification from `/tourism/my-requests`; operator receives notification and opens the exact lead in `/admin/tourism?leadId=...`;
- operator answers from lead details; tourist receives notification and opens the exact request in `/tourism/my-requests?leadId=...`;
- with customer request details already open, operator answer appears after notification refresh without leaving the screen;
- customer and operator can expand full request history and collapse it back to recent events;
- tourist can open pickup point photos fullscreen and understand where to wait for bus/minivan;
- weak Android map performance;
- offline/network errors during lead request and ticket save;
- auth continuation from post-submit registration CTA;
- route button with/without Yandex Maps installed.

## 26. Open decisions

1. Добавлять ли `tourism_operator` в `BusinessType` сразу или вести первые маршруты через `tourism_organizations` поверх текущих `businesses`.
2. First release принят как `lead_request` без concrete slots, чтобы быстрее проверить спрос. Slot engine переносится после первых реальных заявок.
3. Где хранить pilot referral codes: отдельная таблица или `referral_source/referral_code` строками в lead/booking.
4. Нужна ли отдельная публичная вкладка `Туризм` в нижней навигации или только входы из карты/атласа/винодельни.
5. Какие крымские винодельни и маршруты идут в первый seed.
6. Resolved for pilot: реальные заявки обрабатывает не WinePool admin как основной оператор, а `tourism organization` + default responsible tour agent; WinePool admin сохраняет контроль, переназначение и модерацию.
7. Нужны ли English-language fields для туристов уже в первом tourism pilot.
8. Какую legal-safe формулировку использовать для paid placement: `Партнерский маршрут`, `Партнерский материал`, `Продвигаемое размещение` или иной вариант после legal review.
9. Нужна ли маркировка интернет-рекламы/ОРД для featured tourism placements, если конкретный формат будет квалифицирован как реклама.
10. Resolved for lead pilot: гость может оставлять tourism lead/request без регистрации, если указал контакт; уведомления и история доступны после регистрации/логина.
11. Кто в операционном процессе отвечает за превращение demand cluster в новый маршрут: WinePool admin, туроператор или винодельня.
12. Partially resolved: первым каналом включены in-app notifications; Telegram/email остаются будущими escalation/backup-каналами.
13. Нужно ли хранить external messenger thread/contact metadata для заявок, если фактическая коммуникация частично идет вне WinePool.
14. Является ли HTML offline-ticket финальным форматом пилота или временным артефактом до QR/pass ID в managed booking.
15. Нужны ли lifecycle/cleanup правила для старых ticket photo файлов в Supabase Storage при замене билета или удалении заявки.
16. Какие Android/Aurora permissions и UX нужны для надежного сохранения ticket photo в галерею на целевых устройствах.
17. Нужно ли выделять lead conversation в отдельную таблицу `tourism_lead_messages`, если событийной модели `tourism_lead_events` станет тесно для нормального чата, вложений и прочитанных сообщений.

## 27. Recommended first slice

Для ближайшей реализации не начинать с полной системы расписаний. Слишком большой blast radius.

Рекомендуемый первый slice:

1. Experience cards with `lead_request` for Yalta pilot.
2. Tourism media in Supabase Storage, not bundled APK assets.
3. Pickup points for Yalta routes: coordinates, photos, landmarks and instructions.
4. Vehicle records with photos/recognition hints, because cars can change dynamically.
5. Admin overview with incoming leads.
6. Operator organization and one default responsible agent.
7. Lead status workflow and notifications.
8. Keep `/tourism` wine-first and use `Другие экскурсии оператора` only as secondary operator context.
9. Public winery profile and editorial-ready winery wiki sections: hero, story, gallery, key wines, visit block.
10. Referral tracking for Yalta/offline QR.

### 27.1. Launch kit for Yalta tourism pilot

Before real clients are sent through the pilot, prepare the launch materials in parallel with engineering:

- QR per concrete tour/departure or at least per tour, with stable referral source (`agent`, `offline_point`, `tour_sheet`, `qr_yalta_*`);
- printed/flyer placement for the excursion point and agent scripts: why install WinePool, what the tourist receives, what remains optional;
- short text for the agent's Telegram/Instagram/VK channel: ticket, pickup photos, vehicle info, operator messages, wine diary after the trip;
- one YouTube Shorts scenario about the tour flow: QR from agent -> tour card -> pickup point/photo vehicle -> trip -> receipt/check-in after visit;
- route content checklist before publication: cover, gallery, pickup point photos, vehicle photos, landmarks, address, boarding instruction, memo, operator contact;
- RuStore screenshots/description after onboarding fix: emphasize `Без обязательной регистрации, начните пользоваться сразу`;
- launch rule: do not buy additional paid traffic until the free-entry onboarding update is shipped and measured.

External QR/web entry must not send a first-time tourist directly to RuStore with no context. QR should open a lightweight WinePool web transition page first.

The transition page must be a rendering of the same published tourism data, not a separate landing copy maintained by hand. The operator/admin edits the tour once; the app route, public web route and QR entry page reflect the same source.

Recommended web hierarchy:

```text
winepool.ru/tourism
  public wine-first tourism overview, optional for broad discovery

winepool.ru/tourism/o/{operator_slug}
  operator page: trusted public face of one excursion bureau/operator

winepool.ru/tourism/o/{operator_slug}/t/{tour_slug}
  concrete tour page: preferred QR destination
```

Operator page purpose:

- explain who the operator is and why they are trusted;
- show wine-related tours first;
- show `Другие экскурсии оператора` as secondary inventory, not mixed into the main WinePool wine-first catalog;
- provide one place for the operator's Telegram/Instagram/VK/bio link;
- let a tourist who is not ready for the exact QR tour compare 2-5 options from the same operator.

Concrete tour page purpose:

- QR/flyer/deep-link destination for one tour, e.g. `winepool.ru/tourism/o/yalta-excursions/t/massandra-tasting?ref=booth_01`;
- show title, cover, short route, duration, price note, pickup promise, vehicle/photo ticket benefits;
- show CTA `Открыть в WinePool`, `Установить WinePool`, and optionally `Оставить заявку`;
- if app is installed, open the app route `/tourism/{experience_id_or_slug}?ref=...`;
- if app is not installed, explain the value before sending to RuStore/Aurora, then allow the tourist to return and open the same tour;
- preserve referral fields in URL and in `tourism_leads.referral_source/referral_code/referral_url`.

MVP recommendation:

- first QR codes should point to concrete tour pages, not generic operator pages;
- operator page should still exist as a public hub and fallback for agent bio/social links;
- do not build a large universal tourism marketplace landing before there are several operators/regions;
- do not create static per-tour HTML pages with duplicated content, except as a very thin shell that fetches/render shared published data;
- deferred deep link after install is desirable but not required for the first pilot. The acceptable MVP path is: QR -> tour web page -> install/open -> repeat tap or rescan QR -> exact app tour.

Decision on navigation: during the first Yalta pilot, tourism does not need to become a bottom navigation tab. Keep entries from home/profile/map/winery/deep-link QR. Reconsider bottom navigation only after several regions/operators and stable non-local demand.

Почему:

- быстро проверяет спрос;
- сразу начинает формировать WinePool Wiki и premium-поверхность для будущих партнерских пакетов;
- можно наполнить крымскими предложениями из реальных экскурсионных листов;
- не требует платежей;
- не требует полной slot engine;
- сразу снимает реальную боль офлайн-продавцов: меньше ручных объяснений и пересылки фото точки посадки;
- уже дает WinePool новый seasonal use case.

После первых 20-50 реальных заявок можно решать, нужен ли managed calendar.

Фактически начатый slice 02-04.06.2026 уже закрыл пункты 1-7 в базовом виде, а также добавил customer `Мои заявки`, детали поездки, ответственного агента, фотобилет, offline-save, customer-side actions, operator replies, точные deep-link уведомления, раскрываемую историю и первый QR/referral web route alias на конкретный тур. Следующие сильные шаги: 1) проверить end-to-end заявку с `?ref=...` и убедиться, что `referral_source/referral_code/referral_url` сохранились; 2) подготовить Yalta launch kit с QR/referral, текстом агента, shorts-сценарием и полным контентом маршрута до передачи реальным туристам; 3) привести pilot-диалог и операционную историю к более чистой модели (`tourism_lead_events` vs отдельные `tourism_lead_messages` + read receipts).

## 28. Короткая формула направления

WinePool Tourism - это не продажа вина и не алкомаркет.

Это слой:

```text
Вино -> Винодельня -> История -> Локация -> Экскурсия -> Заявка -> Визит -> Дегустация/Отзыв
```

Он должен усиливать главный продукт WinePool: личный винный дневник, карту впечатлений и доверительный справочник по винодельням.
