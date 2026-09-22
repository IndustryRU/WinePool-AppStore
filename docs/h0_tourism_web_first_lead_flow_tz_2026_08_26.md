# H0.5 — Web-first tourism lead flow

Дата: 26.08.2026; обновлено 30.08.2026
Статус: H0.5 complete; canonical web-first exact-tour flow active
Граница: завершение H0 acquisition/referral loop; без H1 marketplace и кабинетов.

Contract-first dependency перед responsive UI:
[`h0_published_tour_contract_audit_2026_08_27.md`](h0_published_tour_contract_audit_2026_08_27.md).

## 1. Проблема

Production device smoke подтвердил exact Android deep link, referral snapshot,
guest/authenticated leads, operator notifications и outcome audit. Одновременно
обнаружен acquisition-разрыв: CTA публичной страницы ведёт в RuStore до заявки.
Даже установленное приложение открывается из магазина на главной странице и
теряет exact tour/referral context. Для нового пользователя установка до заявки
создаёт лишнюю воронку отказа.

Текущая статическая страница тура также вручную дублирует published данные.
Это расходится с tourism platform contract: приложение, Flutter Web и QR entry
должны читать один snapshot, а статический HTML допустим только как тонкая
SEO/transition оболочка.

## 2. Решение

Primary acquisition flow становится web-first:

```text
QR / messenger / partner link
  -> exact public tour page with referral
  -> Оставить заявку
  -> responsive Flutter Web form
  -> one server create-lead contract
  -> success state
  -> optional Open WinePool / Install WinePool / Stay on web
```

Установка или наличие приложения не являются условием создания заявки.
Приложение предлагается после server success как способ получать статусы,
сообщения, билет, точку посадки и данные автомобиля.

## 3. One form, multiple shells

Нельзя создавать независимую HTML-форму. Существующая `_LeadRequestSheet`
рефакторится в публичный platform-neutral компонент, условно
`TourismLeadRequestForm`:

- единые controllers/validation и `TourismLeadRequest`;
- единый submit controller/repository/RPC;
- одинаковая referral normalization и analytics semantics;
- единый success/error/idempotency contract;
- никаких platform-specific веток в бизнес-правилах.

Различается только presentation shell:

- native Android/iOS: bottom sheet или fullscreen mobile form;
- mobile/desktop/tablet browser baseline: одна responsive form section в конце
  tour page; hero/sticky CTA выполняют scroll-to-form, а не создают вторую form;
- admin/operator продолжает использовать существующую CRM, не эту форму.

Fullscreen/bottom-sheet shell для mobile web остаётся допустимым последующим
экспериментом, если device smoke и funnel `cta_click -> form_start` покажут
проблему. Он обязан переиспользовать тот же component/state и не входит в
первоначальный baseline.

## 4. Public Flutter Web target

Не использовать защищённый `admin.winepool.ru`. Создать отдельный публичный
Flutter Web entrypoint/build, содержащий только tourism acquisition surface:

- public operator/tour read models;
- exact route и referral parsing;
- guest lead form и success state;
- Supabase public client, localization, consent и analytics;
- без admin workbench, scanners и тяжёлых catalog modules.

Причина отдельного entrypoint: текущая полная web-сборка содержит примерно
10.3 MB `main.dart.js` и 7 MB CanvasKit WASM до transport compression. Такой
payload нельзя считать безопасным для QR-трафика по мобильной сети без
измерения.

Статическая SEO-страница может остаться тонкой оболочкой, но расписание, цена,
медиа и операционные данные должны загружаться из published tourism snapshot,
а не поддерживаться второй ручной копией.

### 4.1. Hosting decision — принято владельцем 29.08.2026

Каноническая публичная и SEO-поверхность остаётся на основном домене:

```text
https://winepool.ru/tourism/
https://winepool.ru/tourism/o/{operator_slug}/
https://winepool.ru/tourism/o/{operator_slug}/t/{tour_slug}/
```

- Flutter JS/WASM/assets размещаются в техническом каталоге
  `https://winepool.ru/tourism/app/`; route shell сохраняет `<base href="/">`,
  а bootstrap отдельно задаёт `entrypointBaseUrl`, `assetBase` и self-hosted
  `canvasKitBaseUrl` под `/tourism/app/`, чтобы canonical URL не подменялся
  путём runtime assets;
- overview, operator и exact-tour URL не меняются и не отправляют пользователя
  на отдельный application-домен;
- exact-tour сохраняет лёгкую route-specific HTML SEO shell с `title`,
  description, canonical, Open Graph и structured data, после чего запускает
  Flutter surface из `/tourism/app/`;
- schedule, price, media, form и operational facts shell не дублирует: Flutter
  получает их из текущего Published Tour Contract snapshot;
- один вариант trailing slash выбирается и проверяется до переключения CTA;
  альтернативный вариант обязан отвечать единственным redirect без потери
  `source/ref/referral` query;
- `admin.winepool.ru` и `api.winepool.ru` не используются для public UI.

`tour.winepool.ru` резервируется на будущее как короткий QR/marketing address,
но не как canonical content host. Он должен только разрешать short code или
путь и перенаправлять на соответствующий `winepool.ru/tourism/...`, сохраняя
referral query. До завершения пилота используется временный redirect; постоянный
redirect включается только после стабилизации mapping. Поддомен не должен
создавать индексируемую копию страницы. DNS, TLS и redirect service этим
решением пока не изменяются.

### 4.2. Visual architecture: one system, adaptive compositions

`Переиспользовать Flutter` не означает растянуть mobile screen на desktop.
Переиспользуются source data, domain/presentation models, form contract,
design tokens и базовые components. Композиция меняется по breakpoint внутри
одной tourism design system:

- phone: vertical destination storytelling, edge-to-edge media, sticky bottom CTA;
- tablet: wider gallery and two-column content blocks;
- desktop: panoramic editorial hero with an overlaid booking ribbon, followed by
  an airy single-flow story; no permanent right booking rail in the accepted
  direction;
- large desktop: bounded max width, controlled line length and generous spacing.

Запрещены две независимо поддерживаемые формы или две ручные копии tour content.
Допустимы отдельные Flutter layout widgets (`MobileTourLayout`,
`DesktopTourLayout`), если они получают один view model и собираются из общих
sections/components. Это presentation reuse, а не pixel-identical layout.

### 4.2. Required desktop tour structure

Целевая desktop-композиция согласована владельцем 26.08.2026. Визуальный
референс: [H0 tourism desktop accepted hybrid](design_refs/h0_tourism_desktop_accepted_hybrid_2026_08_26.png).

Принятый принцип: верх страницы продаёт впечатление, середина укрепляет доверие,
низ снимает практические сомнения и принимает заявку.

```text
Panoramic hero, 75–85% of initial viewport
  - one real cover with controlled focal point; no carousel
  - light transparent navigation
  - local gradient behind trust / title / short value proposition
  - booking ribbon overlaid on the lower image edge:
    schedule / duration / transfer / honest split price / primary CTA
Editorial story
  - short emotional introduction
  - asymmetric photo gallery / why this tour
  - compact 3–4 step route/program
  - one wide scenic image break where content supports it
Practical trust section
  - transfer and pickup promise / inclusions
  - operator identity / payment / cancellation and important notes
Lead section
  - one calm, complete web form near the end of the page
  - optional app continuation only after submit or in a quiet secondary position
```

На mobile те же смысловые sections образуют последовательную историю, но
hero booking ribbon перестраивается в compact facts plus sticky bottom CTA, а
форма — в удобную phone composition. Порядок блоков может различаться по
платформе без расхождения данных.

Hero не содержит постоянно раскрытую форму и не конкурирует с заявкой кнопкой
`Открыть в WinePool`. Нажатие `Оставить заявку` ведёт к общей форме; конкретная
desktop-механика (scroll-to-section или side sheet) согласуется отдельно перед
реализацией. Визуальный референс фиксирует hierarchy и composition, но не
является источником фактических текстов или фотографий: production получает их
только из published snapshot.

### 4.2.1. Accepted mobile-web composition

Mobile-web композиция согласована владельцем 27.08.2026. Визуальный референс:
[H0 tourism mobile-web accepted](design_refs/h0_tourism_mobile_web_accepted_2026_08_27.png).
Это браузерная поверхность для первого входа с QR/referral на Android или
iPhone, а не native app shell.

```text
Mobile hero
  - transparent web header: WinePool / share / menu; no app back or bottom nav
  - real cover, controlled focal point and local readability gradient
  - trust / title / one value proposition / schedule / duration / transfer
  - overlaid booking ribbon: honest split price + primary web CTA
Intro
  - one full-width text column; no competing side image
Why this tour
  - one large photographic carousel frame at a time
  - swipe plus in-image arrows and pagination dots
  - title and short description on an in-image black-to-transparent gradient
Program
  - spacious vertical 3–4 step timeline; no parallel photo column
  - optional full-width scenic separator
Practical details
  - full-width accordion rows for transfer, inclusions, operator, payment,
    cancellation and important notes
Lead form
  - no outer card/frame; fields and CTA align to the accordion width
  - full-width stacked fields, readable consent and a large submit CTA
```

Accordion начинается свёрнутым; вся строка является touch target. Одновременно
открыт один раздел, а раскрытие не меняет горизонтальные размеры страницы.
Ключевые price/transfer facts не прячутся только в accordion и остаются видимы
в hero.

Responsive contract:

- базовый design viewport — `390 x 844` CSS px; обязательные проверки также на
  `320`, `375` и `430` CSS px;
- page gutters `20–24` CSS px, body text не меньше примерно `16` CSS px,
  interactive target не меньше `44` CSS px;
- post-hero content одноколоночный; единственная горизонтально прокручиваемая
  поверхность — photo carousel, без page-level horizontal overflow;
- scenic separator на phone получает увеличенную высоту и заголовок не более
  трёх строк; clipping текста ради сохранения desktop aspect ratio запрещён;
- hero price + CTA допускаются в одну строку только при сохранении читаемости;
  на узком viewport цена становится над кнопкой, а шрифты не уменьшаются ради
  сохранения строки;
- form fields имеют высоту минимум `52–56` CSS px, submit CTA — около `60` CSS
  px; legal consent остаётся читаемым;
- после ухода исходного hero CTA за viewport допустим compact sticky bottom CTA,
  который не перекрывает content и form controls;
- carousel имеет usable first frame без interaction, swipe, visible in-image
  arrows, dots, semantic labels и keyboard support;
- для 0/1 image используется честный static/fallback state без ложных dots и
  искусственного дублирования media.

Визуальный референс задаёт composition, hierarchy и density, но не разрешает
копировать сгенерированные тексты/фотографии в published data.

### 4.2.2. Accepted tablet-web composition

Tablet portrait композиция согласована владельцем 27.08.2026. Визуальный
референс: [H0 tourism tablet-web accepted](design_refs/h0_tourism_tablet_web_accepted_2026_08_27.png).
Базовый viewport — `834 x 1194` CSS px; на `768` CSS px каждая двухколоночная
section сохраняется только при выполнении readability/touch constraints, иначе
локально переходит в одну колонку.

Tablet не является увеличенным phone или сжатым desktop:

- hero сохраняет panoramic cover и overlaid booking ribbon;
- intro использует text/image split с достаточным gutter;
- `Почему сюда едут` использует master-detail carousel: активный большой кадр
  слева, очередь следующих preview справа;
- program использует timeline слева и restrained supporting media справа;
- practical details образуют две равные accordion columns;
- form допускает две колонки полей при минимальной высоте `56` CSS px и
  комфортном gap; submit CTA занимает полную ширину;
- form surface имеет только едва заметную границу/тон, без тяжёлой card frame.

Tablet carousel contract:

1. активный item показывает large photo, title и short description;
2. справа видны следующие preview items, а не статический список преимуществ;
3. tap/click по preview или arrow/swipe делает item активным;
4. после перехода очередь сдвигается: прошедший item уходит, следующий появляется;
5. loop допустим, но порядок детерминирован и не теряет selected/focus state;
6. количество media технически не ограничивает layout, но public story должна
   быть curated; baseline — `4–10` сильных кадров;
7. preview derivatives загружаются отдельно от full image, а дальние items —
   lazy/deferred, чтобы gallery не ухудшала initial LCP;
8. управление поддерживает touch, pointer, keyboard и semantic announcement;
9. для `0/1` media используется тот же honest static/fallback contract, что на
   mobile, без ложной carousel navigation.
10. tablet/desktop master transition не перескакивает мгновенно: новый кадр за
    `1200 ms` направленно входит со стороны правой очереди с мягкими
    fade/scale, предыдущий уходит в обратную сторону; три preview справа
    синхронно работают как вертикальная лента — верхний уходит вверх, остальные
    сдвигаются, новый входит снизу; системный reduced-motion отключает
    декоративное движение.

### 4.2.3. Accepted CTA and form interaction

Единый baseline для desktop, tablet и mobile web согласован владельцем
27.08.2026:

1. hero CTA `Оставить заявку` плавно прокручивает к единственной form section;
2. landing поддерживает semantic anchor `#request`; referral/query attribution
   при навигации к anchor не теряется и canonical URL не дублируется;
3. после scroll keyboard focus переводится в form region/первое поле только для
   keyboard-origin action; на touch нельзя самопроизвольно открывать экранную
   клавиатуру;
4. после ухода hero CTA за верх viewport появляется compact sticky CTA;
5. sticky CTA скрывается, когда form section достаточно видима, при success и
   при открытой экранной клавиатуре, если она перекрывает content;
6. повторный CTA возвращает к уже начатой form без очистки controllers,
   attempt id, consent или validation state;
7. submit блокирует повторное нажатие и показывает progress без layout shift;
8. validation/server error сохраняют введённые значения и переводят focus к
   первому actionable error с доступным announcement;
9. success атомарно заменяет form стабильным confirmation state, повторный
   submit недоступен; только здесь появляются optional open/install app actions;
10. browser back/forward и refresh не создают duplicate lead благодаря opaque
    receipt/local guard и server idempotency.

Mobile fullscreen/bottom sheet не реализуется в первом варианте. Решение о нём
принимается только после smoke на реальном Android/iPhone и измерения переходов
`cta_click -> form_view -> form_start -> submit_success`.

### 4.3. Visual quality requirements

Public tour page — acquisition surface, а не техническая карточка. Обязательны:

- сильный первый экран с реальной фотографией места/маршрута;
- responsive cover crops и focal point, чтобы главный объект не обрезался;
- gallery layouts для 0, 1, 2 и many images без пустых/сломанных состояний;
- типографическая иерархия, читаемая цена, длительность, трансфер и расписание;
- доверие: verified status, оператор, реальные points/program evidence;
- явное различие factual price и `на месте`/optional costs;
- primary CTA `Оставить заявку`, не `Установить приложение`;
- skeleton/error/empty states без layout shift и ложных обещаний;
- доступность: keyboard navigation, semantic labels, contrast, 200% text scale;
- responsive QA минимум на 320, 375, 768, 1024, 1440 и 1920 CSS px;
- отдельная проверка portrait/landscape и длинных localized strings.

Контентные требования к публикации тура: cover, минимум одна gallery image,
short value proposition, duration, schedule, price note, route/program,
pickup promise, cancellation terms и operator identity. Недостаточно заполненный
тур получает honest fallback, а не искусственно размноженные изображения.

### 4.4. Single content source and authoring

Оператор/admin заполняет тур один раз через существующую CRM. Public app/web
читает published projection из `tourism_experiences`, organization, stops,
media, pickup points и vehicles. Web-specific presentation metadata допустима
только как структурированное поле того же объекта (например, image focal point
или short SEO summary), а не как отдельная HTML-копия страницы.

Любое изменение published snapshot должно без ручной синхронизации отражаться:

1. в Android/iOS tour detail;
2. в mobile/desktop Flutter Web;
3. в QR/referral route;
4. в SEO metadata/structured data projection;
5. в operator preview.

### 4.5. SEO, sharing and performance

Flutter visual surface дополняется тонкой server/static SEO shell:

- canonical exact-tour URL;
- title/description/Open Graph image;
- structured data только из published projection;
- indexable factual text/fallback;
- referral query не создаёт canonical duplicates.

SEO shell не является второй CMS. Генерация или runtime fetch обязаны брать тот
же snapshot. До внешнего pilot измеряются, а не предполагаются:

- compressed JS/WASM payload;
- LCP/CLS/INP на throttled mobile network/device;
- время до появления CTA и до интерактивной формы;
- image bytes, responsive variants and cache headers;
- conversion steps `page -> form_start -> submit_success`.

Initial performance gate: первый полезный tour context и CTA не должны ждать
загрузки полного текущего WinePool bundle. Dedicated entrypoint, deferred
sections и responsive image derivatives являются частью реализации, если
измерения не проходят установленный перед pilot бюджет.

### 4.6. Visual acceptance gate

До production switch обязательны screenshot/golden и manual QA:

1. Massandra and Inkerman with real production published data;
2. mobile browser Android and iPhone dimensions;
3. tablet portrait/landscape;
4. desktop 1024/1440/1920;
5. 200% text scale and keyboard-only form;
6. slow image, missing image, long title and long price note;
7. form validation, submitting, server error and stable success;
8. visual comparison app vs web: one brand/system, not forced identical layout.

H0.5 не принимается только по факту `HTTP 200` или успешного submit. Требуется
явное owner visual acceptance public tour and form surfaces.

## 5. Unified server contract

Прямой anon insert заменяется одной additive server function/RPC для всех
клиентов. Требования:

- принимает published experience identity, contact fields и referral code;
- referral source/partner/experience/is_test получает только из registry;
- возвращает opaque lead receipt, без раскрытия внутренних UUID;
- server-side validation и нормализация;
- idempotency key/attempt key и защита от двойного submit;
- bounded rate limit по безопасному fingerprint/window без product analytics PII;
- honeypot/challenge escalation для аномального web-трафика;
- atomic lead + notifications;
- одинаковое поведение anon и authenticated, с optional `created_by`;
- audit и test/commercial exclusion сохраняются.

Service-role secret в web bundle запрещён. Публичный anon key допустим только
с RLS/RPC contract; таблица registry остаётся закрытой.

## 6. Consent and data handling

До отправки web-форма показывает обязательное согласие на обработку ПДн со
ссылками на актуальные документы. Не добавлять сторонние CAPTCHA/CDN/analytics,
пока не проверены 152-ФЗ, cookies и трансграничная передача. Contact/comment не
передавать в AppMetrica/Метрику; analytics получает только нормализованный
referral и outcome event.

Compliance gate уточнён по owner QA 29.08.2026:

- consent является отдельным unchecked checkbox, а не частью submit button или
  общего пользовательского соглашения;
- рядом доступны отдельные `/personal-data-consent.html` и `/privacy.html`;
- клиент блокирует submit без checkbox, но авторитетная проверка выполняется RPC;
- lead хранит version, document path, server timestamp, source URL и bounded
  technical evidence; сырой IP в lead не сохраняется;
- consent для туристической заявки не означает согласие на рекламную рассылку;
  marketing opt-in при необходимости вводится отдельно;
- production CTA нельзя переключать, пока negative RPC test и consent-backed
  guest smoke не пройдены.

## 7. Success and continuity

Success composition согласована владельцем 27.08.2026:

- [desktop success reference](design_refs/h0_tourism_desktop_success_accepted_2026_08_27.png);
- [mobile-web success reference](design_refs/h0_tourism_mobile_web_success_accepted_2026_08_27.png).

После успешной заявки:

- форма заменяется stable success state и повторный submit блокируется;
- показывается спокойное подтверждение без modal/toast/fullscreen takeover и
  без confetti;
- heading `Заявка отправлена` и человеческий следующий шаг объясняют, что
  специалист свяжется для уточнения деталей и подтверждения поездки;
- summary показывает только non-contact facts: тур, дата/пожелание, количество
  гостей и место отправления; email/телефон/messenger не повторяются;
- payment note отображается только если соответствует published terms тура и не
  обещает неподтверждённую окончательную стоимость;
- `Вернуться к туру` сохраняет пользователя на текущей странице;
- `Открыть в WinePool` — secondary CTA;
- Android install CTA ведёт в RuStore, iOS — в App Store;
- install/open actions не являются условием успеха и визуально не конкурируют с
  confirmation;
- optional continuation объясняет конкретную пользу без ложного обещания
  автоматической привязки guest lead: быстрый возврат к выбранному туру,
  сохранение маршрутов и планирование поездки; рядом явно сказано, что для уже
  отправленной заявки установка не обязательна и специалист свяжется по
  указанному контакту;
- opaque receipt по умолчанию скрыт под `Детали заявки` и раскрывается только
  как support reference без UUID/PII;
- desktop/tablet summary может быть двухколоночным; mobile остаётся
  одноколоночным с touch targets минимум `44` CSS px;
- sticky CTA, fields, consent и submit button после success отсутствуют;
- success получает accessible focus/announcement без самопроизвольного открытия
  mobile keyboard;
- повторное открытие/refresh использует opaque receipt/local guard и server
  idempotency, а не только временный Flutter state.

Референсы задают hierarchy и tone, а не буквальные пиксели: width, spacing и
line wrapping доводятся на `320/375/390/430/768/834/1024/1440/1920` в ходе
screenshot/manual QA.

## 8. App Links / Universal Links

Verified links — отдельный подпункт того же H0.5, но не blocker web submission:

- Android: HTTPS intent filters + `assetlinks.json`;
- iOS: Associated Domains + `apple-app-site-association`;
- exact path/query/referral должны сохраняться;
- browser остаётся полноценным fallback;
- store переход не должен обещать deferred deep link, пока он не проверен.

Текущее состояние не удовлетворяет этому contract: Android manifest содержит
только custom auth callback, iOS — custom URL scheme без Associated Domains.

Implementation update 30.08.2026:

- Android HTTPS intent filter добавлен только для canonical
  `https://winepool.ru/tourism/o/...`; `autoVerify=true`, package
  `ru.winepool.app`;
- `landing/.well-known/assetlinks.json` сформирован по SHA-256 production release
  certificate; debug certificate в production association не включён;
- iOS Runner получил Associated Domains entitlement `applinks:winepool.ru` во
  всех трёх build configurations;
- AASA contract ограничен `/tourism/o/*/t/*`, но production AASA нельзя
  публиковать до получения фактического Apple Team ID. Template хранится в
  `tool/templates/apple-app-site-association.template.json`; placeholder не
  является deployable artifact;
- query string, включая `ref`, не переписывается platform association и
  передаётся существующему GoRouter вместе с exact path;
- `tour.winepool.ru` по-прежнему не включён ни в Android, ни в iOS association.

Android acceptance 30.08.2026: release-signed build `1.1.0 (2012)` на Android
11 получил verified domain state `always`; canonical exact-tour URL с test
`ref` сразу открыл приложение на странице тура «Дегустация вин в Массандре»
без browser/chooser. Android App Links считаются принятыми. Открытым остаётся
только iOS AASA/signing/device acceptance после получения Apple Team ID.

iOS association update 30.08.2026: Apple Team ID подтверждён владельцем как
`6ZD23YR74Z`; production AASA связывает `6ZD23YR74Z.ru.winepool.app` только с
`/tourism/o/*/t/*`. Query string не ограничивается AASA components и передаётся
приложению вместе с URL. Открытым после публикации остаётся signed iOS build и
device acceptance; iOS store availability/deferred deep link не заявляются.

Owner acceptance decision 30.08.2026: реального Apple-устройства нет, поэтому
iOS Universal Links принимаются для текущего H0 как `contract/server verified`:
entitlement подключён к Debug/Release/Profile, AASA JSON contract покрыт тестом,
production endpoint отвечает прямым `200 application/json`, Team/App ID и path
совпадают. Статус `device verified` не выставляется; реальный iPhone smoke
перенесён до появления устройства и не блокирует закрытие текущего H0-пакета.

## 9. Platform acceptance matrix

| Сценарий | Ожидаемый результат |
|---|---|
| Android, app отсутствует | web form submit -> success -> optional RuStore |
| Android, app установлен | web form доступна; secondary verified link открывает exact tour |
| iPhone, app отсутствует | web form submit -> success -> optional App Store |
| iPhone, app установлен | web form доступна; Universal Link открывает exact tour |
| Desktop/tablet | responsive web form, без требования установить приложение |
| Guest | lead + immutable referral snapshot, no account required |
| Authenticated | lead + creator + same referral snapshot |
| Repeated submit | один lead, stable success, no duplicate notifications |
| Test referral | QA row visible, commercial KPI contribution = 0 |

## 10. H0/H1 boundary

В H0 входят:

1. shared form extraction;
2. lightweight public Flutter Web entrypoint and hosting;
3. unified idempotent create-lead RPC with abuse baseline;
4. consent/success states;
5. platform smoke for web, Android and iPhone browser;
6. verified native links where signing/domain files are available;
7. raw/clean partner report reconciliation after live test.

В H1 остаются marketplace expansion, partner self-service cabinets, payments,
slot inventory, broad tourism redesign, multi-operator discovery and advanced
CRM automation.

Утверждённая последовательность публичного web после H0 вынесена в
`docs/h1_tourism_public_operator_catalog_discovery_plan_2026_08_30.md`:

1. H1.1b — public operator page `/tourism/o/{operator_slug}`;
2. H1.1c — `/tourism` как curated catalog с базовым query search;
3. Tourism Growth backlog — advanced availability/map/personalized
   multi-operator discovery; основной H2 остаётся retail-контуром.

Полный implementation source of truth H1:
`docs/h1_tourism_revenue_loop_tz_2026_08_30.md`.

Текущий lightweight web entrypoint поддерживает только exact-tour/preview и не
должен расширяться этими поверхностями до закрытия H0. Native list/operator
screens являются источником логики, но не готовым публичным web-дизайном.

## 11. Implementation order

1. Audit/refactor published tour view model and extract shared visual sections.
2. Extract shared form without behavior change; add widget/controller tests.
3. Add idempotent server RPC, migration/backup, SQL security and concurrency tests.
4. Build mobile/tablet/desktop responsive compositions and visual fixtures.
5. Add dedicated public web entrypoint, SEO shell and exact-tour/request route.
6. Measure and reduce initial payload/images before pilot switch.
7. Point primary public CTA to web form; keep install/open secondary.
8. Add stable success/duplicate guard and notification semantics cleanup.
9. Configure and verify Android App Links and iOS Universal Links.
10. Run visual/platform/device matrix and owner-only partner report.
11. Deploy public web/landing atomically, update H0 status and commit; no store
   release or push without separate owner instruction.

## 12. Definition of done

H0.5 закрыт, когда человек с QR на Android, iPhone или desktop может отправить
ровно одну attributed заявку без установки приложения; operator получает её;
test traffic исключается; success state не создаёт дублей; приложение является
полезным optional continuation; все поверхности используют один form/server
contract и один published tourism source of truth.

## 13. Implementation status — 27.08.2026

- production backup перед server foundation:
  `/root/db_backups/winepool_pre_h0_web_first_rpc_20260826.dump`, SHA-256
  `687692ce095a6e058b87d6ef999b842045fc321c452622adf437bbf730a7a954`;
- migration `202608261030_add_idempotent_tourism_lead_rpc.sql` прошла dry-run
  и применена на production;
- `submit_tourism_lead` возвращает opaque attempt receipt, разрешает только
  published experience, применяет registry attribution и создаёт lead атомарно;
- unique attempt и rate trigger защищают от duplicate notifications и bounded
  repeated submissions; released direct-insert client остаётся совместимым;
- SQL contract проверяет grants, trigger/constraint и double-submit с rollback;
- repository переключён с direct insert на RPC; один attempt UUID живёт весь
  lifecycle формы и переиспользуется при retry;
- fields/validation/submission вынесены в shared `TourismLeadRequestForm`, а
  текущий native bottom sheet оставлен presentation shell без визуальных правок;
- targeted Flutter tests и analyze проходят;
- desktop, mobile-web и tablet-web visual directions согласованы владельцем и
  сохранены как reference fixtures; единый scroll-to-form CTA baseline также
  согласован, включая mobile sticky/focus semantics;
- desktop и mobile success states согласованы и сохранены как reference
  fixtures; точные width/spacing остаются частью реализации и screenshot QA.
- production/admin/app audit выявил прямую зависимость public clients от raw
  tables, отсутствие atomic published snapshot и structured price/schedule/story;
  contract v1 и минимальный pre-UI package зафиксированы отдельным audit/TZ.

## 14. Implementation status — 29.08.2026

Выполнено:

- Published Tour Contract v1 gate реализован и активирован на production;
  Massandra и Inkerman имеют проверенные immutable revision `3`;
- exact native route переключён на один bounded RPC projection и сохраняет
  operator/tour/referral context; публичный web использует тот же Dart model;
- добавлен отдельный entrypoint `lib/main_tourism_web.dart`, тонкий
  `web/tourism_index.html` и воспроизводимая сборка
  `tool/build_tourism_web.ps1`; release `main.dart.js` составляет примерно
  `3.38 MiB` до transport compression против примерно `10.3 MiB` полной
  админской сборки;
- реализована согласованная responsive composition: panoramic hero и booking
  ribbon, одноколоночный phone intro, полноширинная phone carousel с gradient,
  tablet/desktop master-detail carousel, program, scenic separator, practical
  accordions, scroll-to-form CTA и mobile sticky CTA;
- одна shared `TourismLeadRequestForm` используется native и web shells;
  web success не повторяет contact data, скрывает opaque receipt в деталях и
  предлагает приложение только как optional continuation;
- локальная release-сборка и visual smoke пройдены на `390x844`, `834x1112` и
  `1440x1000`; destination title вынесен в отдельное обязательное CRM/contract
  поле и больше не меняется при перестановке route points;
- production SQL smoke проверяет anon exact fetch, canonical mismatch, public
  allowlist, pilot story completeness и released-app legacy read.
- owner QA формы выявил отсутствие обязательного consent checkbox и server
  evidence; дефект признан blocker публичного CTA и закрывается отдельной
  consent migration/UI patch до guest submission smoke.

Consent gate реализован и активирован 29.08.2026:

- migration `202608292230_add_tourism_lead_consent_evidence.sql` применена на
  production после backup
  `/root/db_backups/winepool_pre_tourism_consent_20260829_175009.dump`, SHA-256
  `128ce0624e7bc7f360f2379d39fef13889919944cff5a5834c9d979db6b35a9a`;
- shared native/web form показывает отдельный unchecked checkbox и ссылки на
  versioned consent/privacy; submit без checkbox блокируется клиентом;
- RPC требует `consent_given=true` и точную текущую version/page, сохраняет
  server timestamp, source URL, bounded user-agent и per-attempt IP hash без
  сырого IP; historical direct-insert rows не переписывались;
- SQL contract прошёл anon, idempotency, evidence и negative consent tests с
  rollback; HTTP smoke подтвердил `tourism_consent_required` без checkbox и
  успешное прохождение consent validation с последующим bounded validation;
- hidden preview runtime обновлён, production SHA-256 совпадает с local build;
  public CTA по-прежнему не переключён;
- политика уточнена: простое использование Сервиса не заменяет отдельное
  согласие; production `/privacy.html` отвечает `200` с новой редакцией.

Owner guest smoke завершён 29.08.2026 на отдельном test referral:

- попытка без checkbox не создала lead; после явного consent создана ровно одна
  заявка и один opaque attempt;
- consent version/page/server timestamp/source evidence заполнены полностью;
- registry классифицировал заявку как `is_test=true`;
- создано одно operator notification для одного получателя, admin fallback не
  потребовался;
- reconciliation за smoke window: raw `1`, clean `0`, excluded test delta `1`,
  `low_sample=true`; контакты, UUID и иные ПДн в audit output не выводились;
- success state дополнен мягким optional-app объяснением без обещания
  автоматической привязки guest lead; runtime deploy backup:
  `build/ftp-backups/winepool-crisis-landing-20260829_181643`.

Production-like hidden preview задеплоен 29.08.2026:

- owner URL:
  `https://winepool.ru/tourism-preview/o/yalta-excursions/t/yalta-massandra-tasting/`;
- shell имеет meta + HTTP `noindex`, canonical ведёт на утверждённый
  `/tourism/o/...`, runtime и CanvasKit self-hosted под `/tourism/app/`;
- текущая production page/CTA не изменена; HTTP/MIME smoke и desktop/phone
  visual smoke пройдены без console error;
- воспроизведение, scoped deploy и rollback зафиксированы в
  `docs/h0_tourism_public_hosting_preview_runbook_2026_08_29.md`.

До переключения реального public CTA остаются отдельные операционные шаги:

1. выполнить owner visual acceptance на production-like preview и замерить
   first load на мобильной сети; CanvasKit/WASM и изображения остаются главным
   payload risk;
2. выполнить одну web guest submission с отдельными QA-данными, проверить
   operator notification, duplicate retry и owner-only raw/clean attribution;
3. добавить Android App Links и iOS Universal Links для canonical
   `winepool.ru`; `tour.winepool.ru` в association не включать, пока он только
   redirect layer;
4. только после этих проверок заменить primary landing CTA и выполнить
   атомарный deploy. Store release и Git push этим пакетом не выполняются.

### H0.5 closure — 30.08.2026

Все четыре пункта выше завершены: owner принял phone/tablet/desktop preview и
проверил реальное Android-устройство; consent-backed guest smoke,
notification/dedup и raw/clean reconciliation пройдены; Android App Links
приняты на устройстве, iOS Universal Links приняты как contract/server verified
из-за отсутствия Apple device; canonical exact-tour shell переключена на общий
Flutter web-first flow. Payload baseline и известный nginx JS-compression gap
зафиксированы в hosting runbook и не скрываются перед H1.
