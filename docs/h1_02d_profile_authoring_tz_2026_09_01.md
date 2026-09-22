# H1-02D — moderated operator and winery profile authoring

Date: 01.09.2026

Status: detailed implementation contract; ready for H1-02D1.

Architecture note (01.09.2026): shell, partner capability, package entitlement,
moderation and visual decisions are governed by
`h1_tourism_partner_crm_unified_tz_2026_09_01.md`. H1-02D1 may be released to
catalog admins first, but it must be implemented inside the shared Tourism CRM
shell and must not create a second independent cabinet.

Related sources:

- `h1_02_public_operator_winery_profiles_tz_2026_08_31.md`;
- `h1_02c_public_winery_page_handoff_2026_09_01.md`;
- `h1_02b_public_operator_page_handoff_2026_09_01.md`;
- `202608312330_add_h1_02a_public_profile_foundation.sql`;
- immutable `tourism_experience_publications` pattern.

## 1. Audit result and execution decision

H1-02A already delivered the normalized authoring tables, allowlisted section
types, media ownership, publication statuses and catalog-admin RLS. H1-02B/C
already render those records publicly.

The current tables must not be edited by a UI before an immutable publication
boundary exists: public page RPCs presently read rows with
`publication_status = 'published'` directly. Editing such a row could change a
live page before an explicit publish action.

H1-02D therefore starts with an immutable publication contract patterned after
`tourism_experience_publications`. The first rollout is catalog-admin-only, but
the editor uses the shared shell and capability boundary. Partner self-service
is enabled later through the same editor and a moderated proposal workflow; it
does not receive direct canonical or publication writes.

## 2. Required user outcome

An administrator can:

1. choose an operator or winery profile;
2. edit a draft without changing the live page;
3. see validation errors and a faithful draft preview;
4. submit and publish one immutable revision;
5. pause and resume public visibility without deleting history;
6. inspect revision, actor and time for every publication.

A visitor continues seeing the last successfully published revision until a
new revision is explicitly published. Invalid or unfinished drafts never leak.

## 3. Publication storage

Add `tourism_public_profile_publications`:

```text
id uuid primary key
subject_type operator | winery
organization_id uuid null
winery_id uuid null
schema_version smallint = 1
revision integer > 0
content_hash text
payload jsonb object
published_by uuid null
published_at timestamptz
```

Constraints:

- exactly one subject FK matches `subject_type`;
- revision is unique per subject;
- payload is an object and schema version is exactly 1;
- update and delete are rejected by an immutable trigger;
- authenticated catalog admins may read history;
- anon receives no table privileges.

Add `current_publication_id` to operator and winery profile rows. Existing
published pilot content is compiled and snapshotted during the migration so
H1-02B/C output does not disappear.

## 4. Draft and serving semantics

`current_publication_id` is the only editorial payload served publicly.
Authoring rows remain mutable draft source.

| Authoring state | Previous current revision served | Editing allowed |
|---|---:|---:|
| `draft` | yes | yes |
| `submitted` | yes | no, until reopened/rejected |
| `published` | yes, newly selected revision | explicit new draft |
| `rejected` | yes | after reopen |
| `paused` | no | no, until resume/reopen |
| `archived` | no | no |

Saving after a publication creates a new draft state but does not mutate or
hide the current revision. Publishing compiles a new revision and atomically
updates the pointer. Pause changes visibility only and preserves the pointer.

## 5. Versioned RPC boundary

Admin RPCs:

```text
list_admin_public_profiles_v1(subject_type, search, status, page, page_size)
get_admin_public_profile_authoring_v1(subject_type, subject_id)
save_admin_public_profile_draft_v1(subject_type, subject_id, expected_updated_at, draft)
compile_admin_public_profile_v1(subject_type, subject_id)
submit_admin_public_profile_v1(subject_type, subject_id, expected_updated_at)
publish_admin_public_profile_v1(subject_type, subject_id, expected_updated_at)
pause_admin_public_profile_v1(subject_type, subject_id, reason)
resume_admin_public_profile_v1(subject_type, subject_id)
```

Rules:

- every function checks `is_catalog_admin(auth.uid())` itself;
- optimistic concurrency uses `expected_updated_at`;
- save accepts only allowlisted fields, not arbitrary column/value maps;
- compile returns `{payload, errors, warnings, next_revision, content_hash}`;
- publish rejects non-empty errors and writes publication + pointer atomically;
- actor, time, transition and optional reason are auditable;
- no contact, member, business or moderation internals enter public payload.

Public `get_public_tourism_operator_page_v1` and
`get_public_winery_page_v1` read editorial modules from the immutable current
payload. Current verified tours remain dynamically joined so pausing a tour or
destination relation takes effect without republishing editorial content.

## 6. H1-02D1 — safe core editor

Initial winery route: `/admin/winery-profiles`. It is a deep link into the shared
Tourism CRM shell, not a separate navigation or duplicate product.

Desktop composition:

- left subject list with operator/winery switch, search and status filters;
- center structured form;
- right readiness/preview panel where width permits;
- phone/tablet use the same fields as sequential panels, not a squeezed
  three-column layout.

Editable core fields:

| Operator | Winery |
|---|---|
| headline | headline |
| story Markdown | visit summary |
| SEO title/description | story Markdown |
| | SEO title/description |

Canonical name, slug, winery identity, organization type and verified
relations are read-only. The UI provides save draft, preview, submit, publish,
pause and resume according to state and permission.

Draft preview uses the compiler payload and the same normalized DTO/rendering
primitives as public pages. It is never made accessible through an anonymous
URL and receives `noindex` semantics on web.

## 7. Validation

First-slice limits:

- headline: 12–160 characters;
- visit summary: 20–360 characters;
- story Markdown: 40–12000 characters;
- SEO title: 20–70 characters;
- SEO description: 50–180 characters;
- trim values and store empty optional fields as null;
- Markdown subset only: paragraphs, headings H2/H3, lists, emphasis and links;
  update 13.09.2026: the owner accepted underline in the editor toolbar (no
  font size or colour), which Markdown cannot store — see
  `h1_02_public_operator_winery_profiles_tz_2026_08_31.md` §8.2.2 «Редактор
  текста» before implementing storage;
- reject raw HTML, images, iframes, scripts, forms and external booking CTA;
- links require an explicit safe scheme and are not rendered as direct booking
  channels.

Warnings may flag missing optional SEO or short story. Errors block submit and
publish, not draft save.

## 8. Later slices

### H1-02D2 — structured modules

- allowlisted sections and order;
- winery facts with source and verification;
- featured wines restricted to the same canonical winery;
- FAQ structured as question/answer entries;
- no arbitrary HTML or page builder.

### H1-02D3 — media authoring

- semantic hero/feature/gallery slots;
- upload, derivative preview, alt, caption and rights confirmation;
- focal point and deterministic ordering;
- moderation before media can enter a publication;
- reuse `TourismEditorialGallery` behaviour.

### H1-02D4 — partner self-service proposals

- business membership determines subjects a partner may propose for;
- partner writes proposals, never canonical/publication rows;
- admin diff/review/approve/reject with reason;
- winery binding and tour destination relations remain separately moderated;
- disputed relationships can be challenged without deleting another
  operator's tour.
- access is evaluated as role capability + package entitlement + moderation
  state; payment never bypasses ownership, validation or review.

## 9. Tests and release gate

SQL:

- non-admin calls rejected;
- anon has no table or admin RPC access;
- draft save leaves current public payload byte-equivalent;
- stale `expected_updated_at` rejected;
- invalid Markdown/contact payload blocks publish;
- publication rows cannot be updated/deleted;
- revision increments and pointer switch are atomic;
- pause hides and resume restores exactly the same revision;
- operator cannot compile winery fields and vice versa.

Flutter/widget:

- state-specific actions and read-only canonical identity;
- unsaved-change warning;
- validation/readiness and stale-edit recovery;
- operator/winery field differences;
- preview matches compiler DTO;
- responsive 390, 834, 1024 and 1440 CSS px;
- no default white borders in the premium admin design system.

Production:

- scoped schema/function/data backup;
- transactional preflight and SQL contract;
- one bounded deployment and PostgREST reload;
- reconcile current pilot pages before/after migration;
- smoke draft save, preview, publish, pause/resume and existing public routes.

## 10. Non-goals of H1-02D1

- partner self-service;
- free-form page builder;
- direct canonical winery edits;
- unmoderated media upload;
- tour/destination editing inside the profile editor;
- contacts, checkout or external booking links;
- redesigning accepted H1-02B/C public pages.

## 11. Immediate implementation order

0. Сначала совместимая migration-поправка публичной страницы: на бою нет
   `tourism_organizations.winery_id` и `visit_reception_state`, а обычная
   `202609131610` перезапишет обёртку связей бренда/хозяйства из `131620`.
   Подробнее — H1-02 §8.2.2. Нужны backup, транзакционный smoke и сверка двух
   опубликованных страниц.
1. **Закрыто 13.09:** structured rich text v1, schema version, тип подачи,
   пресеты, факты, команда, ценности и свои фото вин доставлены миграциями
   `202609131640` и `202609131650`; они пока authoring source, не public DTO.
2. **Частично закрыто 13.09:** `202609131660` создала immutable publications
   и первые снимки опубликованных профилей; `202609131670` переключила winery
   DTO на снимок, не затронув живые туры и сеть бренда/хозяйства. Осталось
   переключить operator DTO тем же правилом. `202609131680` добавила для
   винодельни admin save/preview/publish/pause/resume; её backup:
   `/root/db_backups/winepool_pre_winery_profile_authoring_rpc_20260913.dump`
   (`SHA-256 42fecd58085122fefd717d5910d15c76ae02e208538848432ce4a527c735adb7`).
   События и CRM-программы в этот слой не входят.
3. **Начато 13.09:** Flutter authoring DTO/repository/providers и тёмный editor
   shell с предпросмотром. В нём готовы обложка/оформление и «История»:
   редактор `flutter_quill 11.4.2` является только интерфейсом, а перед
   сохранением его содержимое приводится к закрытому `structured rich text v1`.
   Delta Quill в базу не попадает. До полного renderer публичного профиля
   preview остаётся рабочим авторским preview, а не доказательством совпадения
   с публичной страницей.
   **Продолжение 13.09:** публичный DTO и страница уже читают из снимка
   structured story, редакционные факты и ценности; пресеты и остальные модули
   страницы ещё требуют полного визуального renderer.
4. **Факты начаты 13.09:** готовы отдельное атомарное сохранение, порядок,
   название, значение, источник, видимость и отметка проверки редакцией;
   публикация включает факты в неизменяемый снимок. Остаются команда, ценности,
   ключевые вина и подтверждение прав на фото.
5. Подключить опубликованные программы CRM и их service tags; затем отдельно
   модель, редактор и публичную страницу события.
6. widget/browser/physical QA, handoff, commit и решение о выпуске.

### Web admin deployment log

- 13.09.2026 — release `20260913_212339`: editor shell, structured story,
  facts, values and their public DTO renderer. `nginx -t` passed; unauthenticated
  `https://admin.winepool.ru` returned the expected Basic Auth `401`.
- 13.09.2026 — release `20260913_233743`: editor rebuilt to the accepted
  prototype (H1-02 §8.2.2): permanent ten-section menu, data column, preview
  of the real public page scaled from 1024/390 px. The public winery page body
  became the shared `PublicWineryPageView` coloured by the eight atmospheres.
  Connected for review: «Факты» and «Оформление» with autosave; other sections
  are visible but inactive until their slices. Codex's first shell screen was
  replaced; its RPCs, domain models and rich-text converter are kept.
- 14.09.2026 — release `20260914_002014`: «Обложка» (hero upload into the
  profile slot with focal point, canonical name/location/logo read-only,
  slogan), «История» (title, formatted text, quote, values with own captions,
  internal approval notes), «Посещение» (reception status, visit text,
  practical information) and «Публикация» (readiness, SEO, publish, pause,
  resume). Server contract: migration `202609140100` on production after
  backup `winepool_pre_winery_story_visit_authoring_20260914.dump`.
- 14.09.2026 — release `20260914_011207`: «Люди и место» (team up to ten
  with photos, formatted story and quote; place text, traits, up to three
  photos) and «Вина» (search in the winery's own and produced-for-brand
  catalogue wines, up to six picks with drag order, one main wine shown first
  and large, own photo only with the rights tick, internal reason note,
  read-only production links). Server contract: migrations `202609140200` and
  `202609140300` on production after backups
  `winepool_pre_winery_people_place_authoring_20260914.dump` and
  `winepool_pre_winery_featured_wines_authoring_20260914.dump`.
- 14.09.2026 — release `20260914_112957`: «Программы» — выбор и порядок
  опубликованных программ из Tourism CRM, услуги и размер группы в карточках,
  отдельный read-only список туров операторов и адаптивный предпросмотр
  (широкий, планшетный, телефонный). Server contract: migration
  `202609140500` on production after backup
  `winepool_202609140500_winery_profile_programs.dump`.
- 14.09.2026 — release `20260914_115745`: исправлены связи туров сторонних
  операторов с винодельнями и проверка данных в Tourism CRM. Незаданный
  «Размер группы» теперь публикуется как JSON `null`, а не обнуляет ответ
  предпросмотра. Server contract: migration `202609140510` after backup
  `winepool_202609140510_operator_links_preview_fix.dump`.
- 14.09.2026 — release `20260914_122257`: оператор управляет показом карточки
  своего тура отдельно на странице каждой связанной винодельни; проверенная
  связь при этом сохраняется. Уже подтверждённые связи по умолчанию показываются.
  Предпросмотр и публичная страница теперь выводят видимые туры операторов.
  Server contract: migration `202609140520` after backup
  `winepool_202609140520_operator_winery_page_visibility.dump`.
- 14.09.2026 — release `20260914_125529`: карточки списка CRM, публичная
  карточка тура и карточка на странице винодельни используют утверждённую
  обложку. В «Медиа» архивный файл можно удалить навсегда только когда он не
  нужен опубликованной ревизии или редактируемому блоку. Server contract:
  migration `202609140530` after backup
  `winepool_202609140530_tour_media_cleanup.dump`.
- 14.09.2026 — release `20260914_130354`: «Удалить навсегда» в архиве сразу
  недоступно для фотографии, которая нужна опубликованной ревизии или
  редактируемому блоку; пользователь не получает техническую ошибку после
  подтверждения.
- 14.09.2026 — release `20260914_135100`: карточка тура стороннего оператора
  в «Программах» повторяет утверждённую структуру с обложкой, параметрами и
  адаптивной компоновкой. Чип «Тур оператора · <название>» ведёт на публичную
  страницу оператора, а «Перейти к туру» — на публичную страницу тура. Server
  contract: migration `202609140540` after backup
  `winepool_pre_202609140540_operator_page_links_20260914_135100.dump`.
- 14.09.2026 — release `20260914_140200`: «Обложка» получила необязательное
  «Краткое описание» до 500 символов под слоганом. «Коротко о посещении»
  показывается в блоке «Посетить винодельню», а не дублируется в hero. Оба
  текста попадают в следующую immutable publication revision. Server contract:
  migration `202609140550` after backup
  `winepool_pre_202609140550_hero_description_20260914_140200.dump`.
- 14.09.2026 — release `20260914_152410`: «События» — список событий и редактор события с
  вкладками «Основное», «Дата и место», «Фото», «Программа вечера», «Участие»,
  «Публикация»; предпросмотр — страница события в атмосфере винодельни с
  карточкой даты, которая не уезжает при прокрутке; «Ближайшие события» на
  странице винодельни. Server contract: migration `202609140600` after backup
  `winepool_pre_winery_events_20260914.dump`. Следующий срез: адрес
  `/events/{slug}` на winepool.ru и заявка «Записаться через WinePool» в
  туристический кабинет.
- 14.09.2026 — без выкладки приложения: цепочки функций из миграций
  `202609140500` и `202609140550` схлопнуты в одну функцию каждая
  (`admin_get_winery_profile_authoring_v1`, `compile_tourism_public_profile_v1`,
  `get_public_winery_page_v1_base`), пять промежуточных звеньев удалены.
  Миграция `202609140700` сверила ответы со старыми цепочками для всех
  профилей (7 из 7 совпали); прежние определения —
  `/root/db_backups/winepool_pre_flatten_function_chains_20260914_functions.sql`.
  Новое поле профиля добавляется в эти функции напрямую, а не новым звеном.
- 20.09.2026 — страница события на winepool.ru (`/events/{slug}`, в приложении
  `/tourism/e/{slug}`) и работающая кнопка «Записаться»: заявка приходит в общий
  туристический кабинет, у неё ровно один предмет — тур или событие.
  Формулировки «через WinePool» у гостевых кнопок убраны. Server contract:
  migration `202609140800` after backup `winepool_pre_event_leads_20260920.dump`.
- 20.09.2026 — партнёрская почта: очередь `partner_mailbox_emails` и письма на
  `partners@winepool.ru` — новая заявка, обращение гостя по заявке и заявка без
  реакции партнёра дольше двух часов. Личные письма участников организаций не
  меняются. Migration `202609200200`; отправщик `/opt/winepool-notifs` обновлён,
  прежняя версия — `send_notification_emails.py.bak_20260920_partners`,
  адрес задан в `PARTNER_EMAIL`. Событие на несколько дней принимает выбранный
  гостем день (migration `202609200100`).
- 20.09.2026 — даты события списком: у события список «день + время начала и
  окончания» (`winery_event_occurrences`), в редакторе — «Когда проходит» с
  добавлением дат и помощником «повторять по дням недели», на странице — список
  дат, в заявке — выбор сеанса (`tourism_leads.event_occurrence_id`).
  Существующие события перенесены по дням диапазона. Migration `202609200400`
  after backup `winepool_pre_event_occurrences_20260920.dump`. Стоимость
  добавлена в партнёрское письмо (`202609200300`).

## Отложено сознательно: события оператора (решение 20.09.2026)

События заводит только редакция, страница события живёт в атмосфере
винодельни. Туроператор тоже должен уметь заводить события и показывать их на
своей публичной странице, но отдельный функционал для этого не нужен: у
события уже есть поле «Организатор» и место проведения «В другом месте».

Порядок: сначала партнёрский вход и роли (винодельня редактирует свою
страницу сама), затем — открыть тот же редактор события оператору и показать
его события на странице оператора. Не начинать раньше партнёрского входа:
иначе права придётся делать дважды.

## Расхождение с прототипом: «Вопросы и ответы» и «Почему можно доверять» (решение 21.09.2026)

ТЗ H1-02 ставило на страницу блоки «доверие: источник и дата обновления» и «до
5 FAQ с модерацией», H1-02D2 — «FAQ structured as question/answer entries».
В прототипе редактора раздела под них нет, а на странице оба блока были зашиты
в код одним текстом для всех виноделен — и говорили от лица площадки
(«Заявка остаётся внутри WinePool»).

Сделано:

- «Почему можно доверять» убран. Вместо него строка в подвале: «Страница
  проверена редакцией WinePool · обновлена <дата публикации>» —
  `wineryPageTrustLine`, дата берётся из публикации (`published_at`).
- «Вопросы и ответы» — двенадцатый пункт меню, после «Галереи». До пяти
  записей, вопрос до 160 знаков, ответ до 500, порядок перетаскиванием,
  переключатель «Показывать на странице». Под списком — частые вопросы одним
  нажатием (заготовка ставит только вопрос), ниже «Свой вопрос».
  Нет ни одного видимого вопроса с ответом — блока на странице нет.
- Сервер: `winery_profile_faq`, `admin_replace_winery_profile_faq_v1`, вопросы
  входят в публикацию (migration `202609210400`). Предел в пять записей
  проверяет функция, а не триггер — триггер предела ломал повторное
  сохранение «Ценностей» (`202609210200`).

Модерация вопросов отдельно от публикации не нужна: страницу и так выпускает
редакция, вопросы проходят вместе с ней.

## Расхождение с прототипом: раздел «Галерея» (решение 20.09.2026)

В прототипе десять пунктов меню, «Галереи» среди них нет, а блок «Галерея» на
публичной странице есть: там лежали витринные снимки запуска
(`h1-02/showcase/v1/...`), и заполнить блок через редактор было нельзя.

Добавлен одиннадцатый пункт меню — «Галерея» перед «Публикацией»: загрузка до
16 фотографий, порядок перетаскиванием (первая показывается крупно), подпись
для поисковых систем, галочка прав и выбор центра кадра нажатием по
фотографии. Без галочки прав фотография не публикуется и в предпросмотре не
показывается. Сборка публикации уже брала фотографии с ролью `gallery`, поэтому
серверная часть — только выдача редактору и `admin_replace_winery_profile_gallery_v1`
(migration `202609200500`).

Осталось: сохранять размеры фотографии при загрузке, чтобы рамка галереи
подстраивалась под кадр и не оставляла полей по бокам.
