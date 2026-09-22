# H1.1b/H1.1c — Public Tourism operator and catalog plan

Дата: 30.08.2026
Статус: operator/winery profile direction accepted 31.08.2026; catalog visual
gate remains open
Граница: публичный web discovery поверх уже принятой exact-tour страницы.

Нумерация подчиняется основному post-1.1.0 roadmap. Это не отдельные горизонты
H1/H2: operator page и catalog являются подпакетами H1.1, а основной H2
остаётся `Retail Availability and First Store`. Полный implementation contract
H1: `docs/h1_tourism_revenue_loop_tz_2026_08_30.md`.
Решение по direct contacts и допустимым monetization bases:
`docs/h1_tourism_monetization_contact_decision_2026_08_31.md`.
Принятый detailed implementation contract страниц бюро и винодельни:
`docs/h1_02_public_operator_winery_profiles_tz_2026_08_31.md`.

## 1. Решение

Публичная exact-tour страница закрывает QR, рекламу и прямую ссылку, но сама по
себе остаётся тупиковым acquisition entry. Полноценному Tourism web нужны:

1. страница оператора со всеми его опубликованными турами;
2. публичная витрина `/tourism` с базовым поиском;
3. позднее — расширенный multi-operator discovery.

Эти поверхности важны, но не входят в H0. H0 сначала закрывает измеримый
web-first lead loop и canonical CTA. Расширение начинается отдельным H1-пакетом,
чтобы не задерживать и не размывать H0 acceptance.

## 2. Canonical route hierarchy

```text
/tourism
/tourism/o/{operator_slug}
/tourism/o/{operator_slug}/t/{tour_slug}
/wineries/{winery_slug}
```

- `/tourism` — публичная витрина и базовый discovery;
- `/tourism/o/{operator_slug}` — публичная страница оператора;
- exact-tour route — уже реализованный источник заявки;
- `/wineries/{winery_slug}` — canonical winery profile с visit-модулями;
- базовый search state живёт в query `/tourism?q=...`, а не создаёт множество
  дублирующих индексируемых маршрутов;
- `tour.winepool.ru` остаётся будущим коротким QR redirect layer, не canonical.

## 3. H1.1b — public operator page

Первый пакет после H0, потому что он одновременно повышает доверие, создаёт
cross-sell между турами и устраняет тупик exact-tour страницы.

Минимальный состав:

- premium hero и verified partner identity;
- краткая публичная история/описание оператора;
- территория работы без website/phone/email/messenger до зафиксированной
  WinePool-заявки;
- только опубликованные туры из bounded public projection;
- единые карточки с title, cover, duration, schedule и price semantics;
- переходы на canonical exact-tour routes;
- корректные empty/error/loading states;
- route-specific title, description, canonical, social preview и sitemap entry.

Существующий native `TourismOperatorScreen` является источником поведения и
известных данных, но не готовым web-макетом. Его нельзя просто выставить наружу:
web-композиция отдельно проектируется и принимается владельцем на
phone/tablet/desktop до реализации.

Owner visual gate закрыт 31.08.2026. Принята полноразмерная desktop-first
композиция с адаптивным reflow, общей WinePool design system и без стандартных
белых рамок. Organization-level route chain удаляется: маршрут принадлежит
конкретному туру, а не странице бюро. Внутри страницы используется anchor
navigation `Туры · Об операторе · Галерея`, а не скрывающие SEO-content tabs.

Tour inventory рендерится детерминированно: один flagship; при 2–4 турах все;
при 5+ flagship, до трёх curated cards и честный переход в catalog state,
отфильтрованный по оператору. Случайная выдача запрещена.

Общий public profile shell также принят для canonical winery page. Это не
объединяет winery и operator entities: винодельня показывает собственные
программы отдельно от проверенно связанных туров сторонних бюро из разных
городов. Реализация требует explicit tour-to-winery relation и описана в
H1-02 detailed contract.

## 4. H1.1c — public catalog and basic search

Второй пакет после operator page. При текущем малом inventory это должна быть
выразительная витрина, а не визуально пустая «поисковая система».

Первый состав `/tourism`:

- curated/featured опубликованные туры;
- направления и регионы, только если в них есть предложения;
- блок операторов;
- поиск по названию, месту, оператору и краткому public description;
- небольшой набор доказанно полезных фильтров;
- shareable query state и понятный zero-results recovery;
- переходы в operator и exact-tour pages.

Фильтры не показываются ради количества. Они добавляются только когда inventory
создаёт реальный выбор. При двух турах приоритет — хорошие карточки, история и
навигация, а не сложная панель фильтров.

## 5. Tourism Growth backlog — advanced discovery

После роста предложения, вне обязательного gate H1, остаются:

- поиск по реальной доступности и датам;
- карта и геопоиск;
- персональное ранжирование и рекомендации;
- сложная сортировка и комбинации фильтров;
- множество операторов и региональные SEO landing pages;
- live slot inventory и комбинирование поездок.

Это не должно блокировать H1 operator/catalog baseline и не переименовывается
в H2: приоритет основного H2 принадлежит retail-контуру. Возврат к advanced
Tourism discovery выполняется отдельным roadmap-решением после фактов пилота и
роста inventory.

## 6. Architecture and reuse contract

- exact tour, operator и catalog используют один Published Tour Contract и
  общие public-safe adapters/components;
- operator и winery используют общий public profile component system, но
  разные canonical entities, routes и bounded DTO;
- public clients не читают raw operational tables;
- для operator/catalog создаются bounded public RPC/projections с allowlist;
- lead form, consent, idempotency, success state и referral contract остаются
  общими, отдельные HTML-формы запрещены;
- карточки, media semantics и route generation переиспользуются между Flutter
  app и web, а layout адаптируется по shell/breakpoint;
- query/referral нельзя терять при переходах между public surfaces;
- operator identity публикуется только из явно approved полей; direct contact
  остаётся private, не входит в public operator/catalog/exact-tour DTO и
  раскрывается адресно только для исполнения уже зафиксированной заявки.
- winery aggregation использует только verified canonical destination relation;
  совпадение title/address/coordinates не является связью;
- tour/operator/winery используют один master-detail gallery component:
  desktop/tablet — active frame и вертикальная preview rail со стрелками,
  phone — swipe/PageView, controls и progress.

## 7. SEO and indexing

- exact-tour и operator pages индексируются с уникальными canonical metadata;
- `/tourism` индексируется как основная витрина;
- произвольные search/filter query combinations по умолчанию не создают
  индексируемые дубли; canonical ведёт на устойчивую публичную страницу;
- sitemap содержит только опубликованные и разрешённые routes;
- удалённый/paused/unpublished content возвращает предсказуемый fallback и не
  раскрывает draft payload.

## 8. Visual acceptance gate

Статус visual gates:

1. operator page full composition: accepted 31.08.2026;
2. winery sibling composition: accepted 31.08.2026;
3. shared gallery behavior: accepted 31.08.2026;
4. catalog/search page: remains open;
5. loading, empty, error и zero-results states: verify during implementation.

Codex не перестраивает эти web-интерфейсы без участия владельца. Мелкие
implementation adjustments допустимы только внутри уже принятой композиции.

## 9. Implementation order

1. H0 закрыт 30.08.2026; exact-tour baseline считается H1.1a.
2. Выполнить H1.1 public data audit и утвердить bounded projections.
3. Выполнить H1-02A contract/data audit по detailed profile ТЗ.
4. Реализовать и принять `/tourism/o/{operator_slug}` как обязательный H1-02B.
5. Реализовать winery visit profile H1-02C отдельным reviewable slice, не
   задерживая H1-03 lead operations.
6. Совместно согласовать H1.1c catalog/basic-search composition.
7. Реализовать и принять `/tourism` с базовым query search, не задерживая
   H1.2–H1.6 pilot operations.
8. Возвращаться к advanced discovery только после достаточного inventory,
   измеримого спроса и отдельного решения по общей очереди горизонтов.

## 10. Definition of done

H1.1 public entry baseline готов, когда пользователь может начать с
`/tourism`, найти опубликованный тур, проверить оператора, открыть canonical tour
и отправить одну attributed заявку через общий contract на phone/tablet/desktop;
draft/internal данные не раскрываются, SEO routes воспроизводимы, а raw/clean
measurement продолжает работать без новых параллельных источников истины.
