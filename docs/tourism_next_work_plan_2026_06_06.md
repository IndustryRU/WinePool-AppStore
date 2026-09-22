# WinePool Tourism - ближайший план работ

Дата: 06.06.2026

Статус с 25.08.2026: historical feature backlog. Не является текущим порядком работ; выполнять только пункты, включённые в H0/H1 активного [post-1.1.0 roadmap](/R:/Flutter/Project/winepool_final/docs/post_release_1_1_0_execution_roadmap_2026_08_25.md).

Цель документа: коротко зафиксировать, куда двигаться после редизайна `/tourism`, страницы оператора и разделения media-слотов. Основное ТЗ остается в `docs/winery_tourism_platform_tz_2026_06_01.md`; этот документ - рабочий чек-лист ближайших проходов.

## 1. Media slots and content operations

Статус:

- migration `20260606_add_tourism_image_slots.sql` применена на production self-host Supabase;
- добавлены `tourism_organizations.hero_image_url` и `hero_image_storage_path`;
- добавлена таблица `tourism_home_hero_images` для глобальных обложек главной tourism-страницы;
- Flutter UI уже читает эти слоты, но пока использует pilot fallback из cover/gallery, если новые поля пустые.
- production pilot slots filled:
  - `/tourism`, wine mode: `tourism-media/home/wine/hero_barrels_20260606.png`;
  - operator `Едувялту`: `tourism-media/organizations/yalta-excursions/hero/eduvyaltu_operator_hero_20260606.jpg`;
  - tour `Дегустация вин в Массандре`: `tourism-media/experiences/yalta-massandra-tasting/cover/massandra_tour_cover_20260606.jpg`.

Следующие задачи:

1. Заполнить production-слоты:
   - `tourism_home_hero_images.hero_mode = wine`;
   - `tourism_home_hero_images.hero_mode = all`;
   - `tourism_home_hero_images.hero_mode = other`;
   - `tourism_organizations.hero_image_storage_path` для `Едувялту`;
   - `tourism_experiences.cover_image_storage_path` для `Дегустация вин в Массандре`.
2. Реализовать web/admin-интерфейс управления изображениями:
   - загрузка в Supabase Storage bucket `tourism-media`;
   - preview before publish;
   - replace without APK growth;
   - copyright/permission checkbox;
   - moderation status;
   - recommended crop/aspect hints.
3. После появления интерфейса убрать зависимость от ручного SQL для tourism media.

## 2. Tourism home `/tourism`

Сделано:

- buyer-home entry renamed to `WinePool туризм`;
- buyer-home tourism card gets a clear CTA `Подобрать тур`;
- premium hero;
- animated title by tour kind and region;
- sticky compact search/filter/tab block;
- tabs `Маршруты`, `Операторы`, `Карта`;
- filters: kind, region multi-select, date, departure, operator, transfer, duration, category;
- default focus remains wine-first;
- route list displays operator label.

Следующие задачи:

1. Сделать фильтры полностью data-backed:
   - дата/диапазон;
   - регион;
   - город отправления;
   - оператор;
   - длительность;
   - трансфер;
   - wine/all/other inventory.
2. Добавить empty states по фильтрам:
   - соседние даты;
   - соседние регионы;
   - оставить пожелание туриста;
   - сообщить, какой тур ищет пользователь.
3. Превратить карту из декоративной в data-backed surface:
   - точки маршрутов;
   - винодельни;
   - точки посадки;
   - coverage операторов.

### 2.1. Admin taxonomy and availability dictionaries

Решение после обсуждения 15.06.2026:

- поля `category_tags` и `availability_tags` не должны заполняться свободным текстом в продакшн-CMS;
- в БД должны храниться стабильные ключи, а в admin UI и public filters должны показываться локализованные подписи по текущему языку приложения;
- свободный ввод допустим только как временный pilot fallback, но не как основной способ наполнения каталога.

MVP-справочник категорий:

- `tasting` - `Дегустации` / `Tastings`;
- `winery` - `Винодельни` / `Wineries`;
- `gastronomy` - `Гастротуры` / `Food tours`;
- `history` - `История` / `History`;
- `nature` - `Природа` / `Nature`;
- `sea` - `Море` / `Sea`;
- `mountains` - `Горы` / `Mountains`;
- `family` - `Семейный формат` / `Family-friendly`;
- `private` - `Индивидуальный тур` / `Private tour`;
- `other` - `Другое` / `Other`.

MVP-справочник доступности:

- `today` - `Сегодня` / `Today`;
- `tomorrow` - `Завтра` / `Tomorrow`;
- `weekend` - `Выходные` / `Weekend`;
- `week` - `На неделе` / `This week`;
- `on_request` - `По запросу` / `On request`;
- `group_forming` - `По набору группы` / `Group forming`;
- empty / no value - `Не указано`, не показывать как public filter.

Будущая логика `group_forming`:

- в pilot это только честная подпись доступности: тур возможен, когда оператор набирает группу;
- не обещать туристу гарантированный выезд и свободные места до подтверждения оператором;
- после появления slot/departure engine считать интерес по заявкам на конкретную дату/время и показывать оператору счетчик спроса;
- публичный счетчик мест или набора группы можно показывать только после появления надежной модели `tourism_slots`, capacity и правил подтверждения.

Admin UX:

- заменить многострочные поля `Категории` и `Доступность` на chips/checkbox multi-select;
- в русской локали показывать русские подписи, в английской - английские;
- сохранять только ключи в `category_tags` / `availability_tags`;
- старые текстовые значения не должны ломать каталог: неизвестные теги можно скрывать из фильтров или показывать в admin как legacy/custom.

## 3. Operator showcase `/tourism/o/:operatorSlug`

Сделано:

- выбран premium direction: operator as public face, not a plain list;
- identity `Едувялту`, logo, website action and verified partner feel;
- flagship tour block;
- secondary tours block;
- link to customer requests.

Следующие задачи:

1. Перевести hero на явный `tourism_organizations.hero_image_storage_path`.
2. Добавить fallback/empty state для оператора без hero and without tours.
3. Подготовить масштабирование:
   - если один тур, показывать его как flagship without noisy secondary blocks;
   - если 3-5 tours, show compact list/filter mode;
   - если 10+ tours, add operator-level search and categories.
4. Позже добавить operator analytics:
   - QR source;
   - leads;
   - conversion to confirmed;
   - messages;
   - ticket attachments;
   - requested but unavailable tours.

## 4. Concrete tour page

Состояние: страница работает, но дизайн еще не проходил такой же Product Design pass, как `/tourism` and operator showcase.

Решение после продуктового обсуждения 06.06.2026:

- сообщения, фотобилет и детали поездки не должны оставаться спрятанными только внутри bottom sheet заявки;
- ближайший релиз должен добавить отдельный customer screen `Центр поездки`;
- страница конкретного тура должна вести в `Центр поездки`, если у туриста уже есть заявка;
- доставка бумажного билета должна стать optional service заявки, потому что часть туристов не доверяет удаленному переводу незнакомому агенту и хочет получить билет лично;
- отдельный полноценный мессенджер и новая таблица `tourism_lead_messages` откладываются.

Следующие задачи:

1. Реализовать `Центр поездки` как основной customer-интерфейс заявки:
   - route `/tourism/trip/:leadId`;
   - статус, посадка, машина, памятка, фотобилет, агент/организация;
   - статус доставки бумажного билета, если турист запросил услугу;
   - сообщения туриста и оператора на текущей модели `tourism_lead_events`;
   - deep-link из уведомлений туриста.
2. Добавить в форму заявки optional service `Нужна доставка бумажного билета`:
   - включается только если у оператора доступна доставка;
   - у оператора хранится зона доставки и ориентир по цене;
   - город/район, адрес или ориентир;
   - удобное время и комментарий;
   - контакт для курьера;
   - честный copy: WinePool не принимает оплату, не продает и не доставляет алкоголь, оператор уточнит стоимость доставки.
3. Разработать 3 концепции страницы тура:
   - practical request-first;
   - premium winery/route story;
   - operational ticket-ready page.
4. Собрать гибрид:
   - tour hero image from `tourism_experiences.cover_image_storage_path`;
   - short program;
   - route/stops;
   - pickup points with photos;
   - vehicle examples;
   - lead form;
   - current request status for tourist;
   - ticket photo and offline save;
   - paper ticket delivery availability as trust-building option;
   - mini-chat/messages.
5. Проверить guest/auth deep-link behavior from QR.

## 5. Admin/operator workflow

Сделано:

- operator can see assigned leads;
- lead details;
- statuses;
- trip fields;
- ticket photo;
- paper ticket delivery request/status/fee;
- customer/operator messages;
- notifications deep-link to exact lead.

Решение 15.06.2026: `/admin/tourism` должен развиваться не как внутренний экран WinePool admin, а как web-first CRM/кабинет, который работает в двух режимах:

- WinePool admin / superadmin:
  - видит все организации, все туры, все заявки и все статусы;
  - может создавать, редактировать, публиковать, ставить на паузу и архивировать любой тур;
  - может переназначать туры/заявки между организациями и агентами;
  - видит governance-поля, moderation/status notes и публикационный checklist.
- Partner tourism organization member (`owner`, `manager`, `agent`, later `viewer`):
  - входит в тот же `/admin/tourism`, но видит только свои organization tours and assigned leads;
  - может создавать и редактировать свои туры, медиа, точки посадки, авто и рабочие данные;
  - не видит чужие контакты, заявки, операторов и внутренние WinePool governance-поля;
  - публикация может быть прямой или через moderation, в зависимости от роли и trust level организации.

Target CRM layout:

- collapsible left tour drawer: search, filters, own/all visibility depending on role, create tour action;
- central editor with tabs: `Основное`, `Фильтры`, `Описание`, `Медиа`, `Маршрут`, `Посадки`, `Авто`, `Заявки`;
- right publish-readiness panel: public preview, completion checklist, warnings;
- on mobile the same roles apply, but drawer/editor/checklist collapse into stacked or separate views.

Design reference:

- [WinePool Tourism CRM.png](WinePool%20Tourism%20CRM.png) - approved hybrid mockup: collapsible tour drawer, tabbed center editor, right public preview and publication checklist.

Следующие задачи:

1. Rebuild `/admin/tourism` into web-first CRM shell:
   - collapsible tour drawer;
   - selected tour editor in the center;
   - right preview/checklist panel;
   - role-aware data scope.
2. Move existing bottom-sheet fields into editor tabs:
   - `Основное`;
   - `Фильтры`;
   - `Описание`.
3. Move nested entities into tabs:
   - `Медиа`;
   - `Маршрут`;
   - `Посадки`;
   - `Авто`.
4. Add role-aware actions:
   - WinePool admin can manage all organizations/tours;
   - partner member can manage only own organization tours;
   - viewer/read-only mode is possible later.
5. Add delivery controls in lead details:
   - status `requested/confirming/confirmed/out_for_delivery/delivered/cancelled/unavailable`;
   - delivery fee;
   - operator note;
   - customer notification and event history.
6. Add tourism media management to admin/operator panel.
7. Add grouped operational board:
   - by tour;
   - by date;
   - by status;
   - by vehicle/departure.
8. Add event/message model cleanup:
   - separate `tourism_lead_events`;
   - separate `tourism_lead_messages`;
   - read receipts;
   - message attachments later.
9. Prepare future departure/slot engine:
   - capacity;
   - vehicle assignment;
   - merging vehicles;
   - change notifications;
   - booking vs lead boundary.

Дополнение 15.06.2026:

- режим `По набору группы` не равен полноценному слоту;
- до slot engine заявки остаются `tourism_leads`, сгруппированные по желаемой дате/времени, без гарантии места;
- в operator/admin board нужно позже показывать demand counter по туру + preferred date/time: сколько заявок хочет конкретный выезд, сколько уже подтверждено, сколько ожидает оператора;
- после введения `tourism_slots` счетчики должны перейти на `slot_id`, `departure_at`, `capacity`, `confirmed_count`, `pending_count`, `available_count`.

## 6. Launch kit

Следующие задачи для пилота:

1. QR links:
   - operator showcase QR;
   - concrete tour QR;
   - booth/source tags.
2. Referral analytics:
   - `referral_source`;
   - `referral_code`;
   - `referral_url`;
   - conversion from QR -> lead -> confirmed.
3. Agent materials:
   - print QR card;
   - short instruction for booth sellers;
   - text for Telegram/WhatsApp;
   - short video script.
4. Store/marketing copy:
   - mention tourism without forcing registration;
   - keep 18+ and no alcohol sale positioning clear.
   - mention paper-ticket delivery only as optional operator-confirmed service, not as WinePool payment or alcohol delivery.
