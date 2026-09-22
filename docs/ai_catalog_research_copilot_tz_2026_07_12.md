# WinePool — ТЗ: AI Research Copilot для модерации новых вин

Дата: 12.07.2026  
Статус: проектное ТЗ для реализации MVP  
Приоритет: высокий  
Основной экран: `/admin/draft-submissions/:id/catalog-resolution`

## 1. Резюме решения

WinePool требуется встроенный AI-ассистент модератора, который по данным пользовательской заявки исследует вино и винодельню, подготавливает структурированные предложения по полям каталога, доказательства и редакционные тексты, но не принимает финальное каталожное решение самостоятельно.

Целевой эффект:

- сократить среднее ручное время обработки новой позиции с 25–30 до 5–10 минут;
- удерживать среднюю полную переменную стоимость исследования в пределах 20 RUB при допустимом hard cap 100 RUB для сложной бутылки;
- сохранить премиальный редакционный стиль WinePool;
- не допускать автоматической публикации выдуманных или неподтверждённых данных;
- встроить решение в существующий normalization workbench и текущий audit trail;
- переиспользовать результаты исследования винодельни для следующих вин и тем самым снижать стоимость каждой последующей карточки.

Решение не является автономным модератором. Это управляемый research workflow с обязательным human-in-the-loop подтверждением.

## 2. Контекст существующей системы

В проекте уже реализованы:

- очередь `/admin/draft-submissions`;
- dossier заявки `/admin/draft-submissions/:id`;
- normalization workbench `/admin/draft-submissions/:id/catalog-resolution`;
- frozen `snapshot_payload` заявки;
- фото, строки чеков, штрихкод, предложения похожих вин и история модерации;
- выбор или создание country, region, winery, grape variety и wine;
- alias memory и normalization decisions;
- атомарная финализация через RPC `admin_finalize_catalog_normalization_decision`;
- admin-only governance для canonical catalog;
- статусы качества `verified`, `limited`, `receipt_only` и visibility scope.

AI Copilot должен расширять этот контур, а не создавать второй способ публикации.

## 3. Термины

- **Submission** — запись `draft_catalog_submissions`.
- **Research job** — один серверный запуск AI-исследования заявки.
- **Research report** — зафиксированный результат job: идентификация, источники, предложения, конфликты, тексты и usage.
- **Suggestion** — предложение значения для одного конкретного поля.
- **Evidence** — фрагмент источника, подтверждающий suggestion.
- **Source** — веб-страница, этикетка, штрихкод, строка чека либо существующая запись WinePool.
- **Confidence** — оценка уверенности системы, не являющаяся статистической гарантией.
- **Apply** — перенос выбранного предложения в локальное состояние формы workbench.
- **Finalize** — существующая атомарная операция публикации решения модератором.
- **Cost guard** — серверный лимит расходов на один job.

## 4. Цели и KPI

### 4.1. Основные цели MVP

1. Автоматически идентифицировать новую бутылку по фото, штрихкоду, чеку и draft snapshot.
2. Найти и ранжировать достоверные источники.
3. Подготовить поля вина и при необходимости новой винодельни.
4. Для каждого существенного значения показать происхождение, confidence и конфликты.
5. Сформировать оригинальные русскоязычные описания в стиле WinePool только из подтверждённых фактов.
6. Дать модератору возможность применить одно поле, группу или все безопасные предложения.
7. Не менять canonical catalog без обычного действия модератора.
8. Измерять стоимость, длительность и процент принятых предложений.

### 4.2. Целевые показатели пилота

По результатам минимум 30 реальных заявок:

- медианное время `research_requested -> report_ready`: не более 90 секунд;
- P95 времени исследования: не более 180 секунд;
- медианное активное время модератора на новую карточку: не более 10 минут;
- не менее 70% предложенных непустых полей применяются без правки;
- не менее 90% отчётов содержат минимум один доступный модератору источник;
- 100% утверждений редакционного описания трассируются к фактам отчёта;
- 0 автоматических canonical writes;
- 0 автоматических публикаций web-изображений;
- средняя полная переменная стоимость по пилотной выборке: не более 20 RUB;
- медианная полная переменная стоимость обычного job: не более 15 RUB;
- P90 полной переменной стоимости: не более 50 RUB;
- ни один job не может превысить 100 RUB;
- не менее 95% job завершаются без ручного технического перезапуска.

### 4.3. Метрика экономического эффекта

Для каждого завершённого решения собирать:

- `moderation_started_at`;
- `research_requested_at`;
- `report_ready_at`;
- `finalized_at`;
- оценку активных минут модератора;
- AI cost в USD и RUB;
- число применённых, исправленных и отклонённых suggestions.

Расчёт:

`saved_minutes = baseline_minutes - actual_active_minutes`, где начальный baseline равен 27.5 минуты и затем заменяется медианой контрольной ручной выборки.

## 5. Non-goals MVP

В MVP не входят:

- автоматическое одобрение submission;
- самостоятельное создание вина, винодельни, региона или сорта агентом;
- публикация первого найденного изображения;
- генерация или перерисовка этикетки/логотипа;
- обучение собственной ML-модели;
- multi-agent framework и «дискуссии» между несколькими моделями;
- обход сайтов, запрещающих автоматический доступ;
- массовая обработка всей очереди без действия администратора;
- автоматическая юридическая оценка лицензии изображения;
- замена существующего parser/import pipeline;
- изменение пользовательского submission snapshot после отправки.

## 6. Роли и права

### 6.1. Модератор

Может:

- запустить исследование доступной ему заявки;
- видеть job, sources, evidence, suggestions и usage;
- применить предложения в форму;
- отклонить или вручную изменить предложения;
- запросить расширенное исследование с подтверждением дополнительного бюджета;
- выбрать медиа-кандидата и отдельно подтвердить его использование;
- выполнить существующую финализацию.

### 6.2. Обычный пользователь

Не видит:

- внутренние prompts;
- confidence;
- внутренние источники и комментарии модератора;
- стоимость исследования;
- технические ошибки job.

Получает только существующий итог модерации.

### 6.3. Серверный worker

Может писать только в AI research tables и storage-зону временных производных файлов. Он не получает прямой разрешённый путь создания canonical catalog entities.

## 7. Пользовательский сценарий

### 7.1. Основной happy path

1. Модератор открывает normalization workbench.
2. Система показывает существующие draft evidence и похожие canonical wines.
3. Модератор нажимает `Подготовить карточку с AI`.
4. Backend создаёт job с immutable input snapshot, целевым soft budget 20 RUB и hard cap 100 RUB.
5. UI показывает этапы выполнения и не блокирует уход со страницы.
6. Worker анализирует внутренние данные и фото, выполняет ограниченный web research, валидирует результат и сохраняет report.
7. Модератор получает уведомление/обновление экрана.
8. В workbench появляется блок `AI-исследование`:
   - итог идентификации;
   - потенциальные дубли;
   - предложения по вину;
   - предложения по винодельне;
   - источники и конфликты;
   - описание;
   - медиа-кандидаты;
   - стоимость и время.
9. Модератор применяет отдельные поля либо `Применить уверенные`.
10. Применённые данные попадают только в локальное состояние формы.
11. Модератор проверяет/редактирует карточки обычными инструментами.
12. Финализация выполняется существующим RPC.
13. В normalization payload добавляется ссылка на report и список принятых suggestions.

### 7.2. Повторный запуск

Повторный запуск разрешён, если:

- первый job завершился ошибкой;
- появились новые evidence от пользователя;
- модератор изменил идентификационный запрос;
- отчёт устарел;
- требуется расширенный поиск.

По умолчанию UI предлагает использовать готовый свежий report. Повтор не должен происходить автоматически.

### 7.3. Расширенное исследование

Если job израсходовал soft budget 20 RUB и остались существенные конфликты, workflow может продолжиться автоматически только при наличии обоснованного следующего шага. UI показывает повышенный расход и текущий прогноз:

`Сложное исследование. Потрачено X ₽, прогноз до N ₽.`

Отдельное подтверждение модератора требуется перед выходом за 60 RUB. Абсолютный hard cap одного report family в MVP — 100 RUB. Таким образом, большинство карточек остаются дешёвыми, но сложная бутылка не обрывается искусственно на средней целевой стоимости.

## 8. UX normalization workbench

### 8.1. Размещение

На desktop блок AI размещается отдельной колонкой/панелью рядом с существующей формой нормализации. На узком экране — раскрываемым разделом перед финальной action bar.

### 8.2. Состояния панели

- `not_started` — CTA и краткое объяснение;
- `queued` — задача поставлена в очередь;
- `extracting` — анализ фото и исходных данных;
- `searching` — поиск источников;
- `verifying` — сопоставление и проверка конфликтов;
- `writing` — подготовка редакционного текста;
- `ready` — отчёт доступен;
- `partial` — отчёт доступен, часть этапов не завершилась;
- `budget_exhausted` — лимит достигнут;
- `failed_retryable` — возможен повтор;
- `failed_final` — автоматический повтор бессмыслен;
- `cancelled`.

### 8.3. Карточка suggestion

Каждое поле показывает:

- название поля;
- предлагаемое значение;
- текущее значение workbench;
- confidence badge;
- тип источника;
- число подтверждений и конфликтов;
- `Почему предложено`;
- `Применить` / `Отменить применение`;
- предупреждение, если suggestion менялся после применения.

Цвет confidence не должен быть единственным сигналом. Нужны текстовые статусы:

- `Подтверждено` — confidence >= 0.90 и нет конфликта;
- `Вероятно` — confidence 0.75–0.899;
- `Требует проверки` — confidence < 0.75 или есть конфликт;
- `Не найдено`.

### 8.4. Массовое применение

`Применить уверенные` переносит только suggestions, которые:

- имеют confidence >= 0.90;
- подтверждены источником допустимого уровня;
- не содержат conflicts;
- прошли enum/reference validation;
- не перезаписывают вручную изменённое модератором поле без дополнительного подтверждения.

Описания и изображения никогда не входят в массовое применение автоматически.

### 8.5. Сравнение дублей

До предложения создать новое wine AI-панель показывает до пяти canonical candidates с причинами:

- совпадение barcode;
- нормализованное название;
- winery;
- винтаж/линейка;
- цвет/type/sugar;
- grapes;
- визуальное сходство этикетки при наличии.

При barcode exact match default CTA — `Проверить существующее`, а не `Создать новое`.

### 8.6. Стоимость

В технической части панели показывать:

- `Стоимость исследования: 8.40 ₽`;
- `Целевая стоимость: 20 ₽ · максимум: 100 ₽`;
- количество model и search calls;
- возможность раскрыть технические детали.

Стоимость не должна визуально конкурировать с качественными предупреждениями.

## 9. Источники и правила доверия

### 9.1. Приоритет источников

Уровень A:

- официальный сайт производителя;
- официальный technical sheet/PDF производителя;
- фото физической этикетки пользователя;
- официальные реестры appellation/регионов.

Уровень B:

- официальный сайт импортёра или дистрибьютора;
- официальный магазин/представительство бренда;
- авторитетный отраслевой каталог с редакционной проверкой.

Уровень C:

- крупные специализированные магазины;
- агрегаторы и профессиональные базы;
- публичные карточки баров/ресторанов с фото бутылки.

Уровень D:

- маркетплейсы;
- пользовательские публикации;
- поисковые snippets без открытия источника;
- форумы и социальные сети.

Уровень D нельзя использовать как единственное подтверждение для `verified` карточки.

### 9.2. Правила evidence

Каждый factual suggestion обязан иметь минимум один evidence item, кроме:

- результата детерминированной нормализации текста;
- ссылки на существующую canonical entity;
- значения, напрямую прочитанного из исходного draft snapshot, которое явно помечено `source=draft`.

Evidence содержит короткий фрагмент, но не полную копию защищённого текста.

### 9.3. Конфликты

Если источники расходятся, нельзя выбирать значение только по большинству. Система должна учитывать уровень источника, дату, винтаж и рынок.

Примеры обязательных конфликтов:

- разные сорта или их доли;
- разная крепость;
- разные сахар/type/color;
- одно название относится к разным винтажам или рынкам;
- winery и brand смешаны;
- бутылка является специальной retail/private label версией.

## 10. AI workflow

### Шаг 0. Сбор immutable input

Сохранить:

- submission id и draft wine id;
- snapshot payload;
- связанные source rows;
- photo URLs или подписанные временные ссылки;
- barcode;
- top existing suggestions;
- релевантные canonical candidates;
- текущие справочники enum;
- prompt/schema versions;
- budget snapshot и курс USD/RUB.

Нельзя передавать модели лишние персональные данные пользователя, полный чек, адрес магазина или profile data, если они не нужны для идентификации.

### Шаг 1. Local-first candidate search

До web search выполнить:

- exact barcode lookup;
- aliases lookup;
- normalized wine/winery name lookup;
- trigram/full-text candidate search;
- фильтрацию по country/region/color/type/sugar;
- reuse ранее подтверждённого research knowledge.

Если найден exact barcode с сильным canonical match, web research сокращается до проверки идентичности.

### Шаг 2. Vision extraction

Модель получает оптимизированные версии пользовательских фото и извлекает:

- front/back label text;
- brand/winery;
- cuvée/wine name;
- vintage;
- appellation;
- country/region;
- volume;
- alcohol;
- barcode digits, если видимы;
- grapes и признаки organic/biodynamic/natural;
- importer/producer information;
- uncertainty per extracted element.

Нельзя считать OCR-текст истинным без нормализации и cross-check.

### Шаг 3. Формирование поисковых запросов

Формируются максимум:

- 3 базовых запроса обычного job;
- до 2 уточняющих запросов при конфликте;
- 1 запрос по winery, если её нет в reusable knowledge;
- итого не более 6 web search calls без расширенного бюджета.

Запросы должны сочетать наиболее различимые признаки: quoted wine name, winery, vintage, barcode, appellation.

### Шаг 4. Source acquisition

Система выбирает не более 5 candidate pages и глубоко обрабатывает не более 3 основных источников в обычном job.

Для каждого source фиксируются:

- resolved URL;
- domain;
- title;
- source tier;
- language;
- published/observed date, если доступна;
- HTTP/content status;
- content fingerprint;
- признаки official/retailer/aggregator;
- связь с конкретным vintage/market;
- ограничения использования media.

### Шаг 5. Fact extraction

Каждый источник преобразуется в компактный набор фактов по строгой JSON schema. Модель не должна получать повторно всю страницу на каждом следующем шаге.

### Шаг 6. Entity resolution

Сопоставить факты с:

- существующей winery;
- country и region;
- grape varieties;
- wine line;
- существующими wines;
- aliases.

Неизвестная entity возвращается как create candidate, но не создаётся.

### Шаг 7. Conflict and confidence engine

Confidence рассчитывается сервером из комбинации:

- надёжности источника;
- числа независимых подтверждений;
- прямоты evidence;
- согласованности с этикеткой;
- соответствия vintage/рынку;
- согласованности с canonical data;
- model self-assessment только как слабого дополнительного сигнала.

Модель не является единственным источником итогового confidence.

### Шаг 8. Editorial generation

Сначала создаётся `verified_fact_pack`, затем отдельным вызовом генерируются:

- описание вина;
- краткое описание вина;
- описание винодельни, только если она новая или отсутствует качественный текст;
- editorial warnings.

Генератор не получает неподтверждённые facts и не имеет web tool.

### Шаг 9. Deterministic validation

Перед сохранением report проверить:

- JSON schema;
- допустимость enum;
- UUID/reference candidates;
- URL scheme/domain;
- числовые диапазоны;
- длину и язык текстов;
- отсутствие персональных данных;
- наличие evidence у factual suggestions;
- отсутствие unsupported factual claims в descriptions;
- cost cap.

### Шаг 10. Report publish

Report сохраняется одной версией и становится immutable. Повторный job создаёт новую версию; старый отчёт не перезаписывается.

## 11. Поля результата

### 11.1. Wine identity

- `canonical_name_ru`;
- `original_name`;
- `vintage` как отдельное research-поле, даже если текущая модель Wine его не хранит;
- `barcode`;
- `winery_candidate_id` или `new_winery_candidate`;
- `line_candidate_id`;
- `country_code`;
- `region_id` или `new_region_candidate`;
- `color`;
- `type`;
- `sugar`;
- `alcohol_level`;
- `serving_temperature`;
- `grape_variety_candidates` с долями, если подтверждены;
- `is_pet_nat`;
- `is_organic`;
- `is_biodynamic`;
- `is_natural`;
- `catalog_quality_status_suggestion`;
- `visibility_scope_suggestion`;
- `description`;
- `short_description` как research-only поле до решения о хранении в canonical schema;
- `awards` как структурированный список только при подтверждаемом официальном/конкурсном источнике; отсутствие найденных наград фиксируется явно и не означает, что у вина их никогда не было.

Не предлагать sweetness/acidity/tannins/saturation как объективные значения без утверждённой продуктовой методики WinePool.

### 11.2. Winery

- `canonical_name`;
- `alternative_names`;
- `country_code`;
- `region_id` или region candidate;
- `website`;
- `location_text`;
- `latitude`, `longitude`;
- `founded_year`;
- `winemaker`;
- `description`;
- `logo_candidates`;
- `banner_candidates`;
- контакты только с официального источника.

### 11.3. Media candidate

- `kind`: bottle/logo/banner;
- source page URL;
- direct asset URL, если безопасно и доступно;
- preview asset path;
- width/height/mime;
- official-source flag;
- detected subject;
- transformation proposal;
- rights status: `unknown`, `official_candidate`, `user_supplied`, `approved`, `rejected`;
- moderator approval metadata.

### 11.4. Awards and competition results

Поиск наград входит в стандартный research contract как дешёвый optional stage. Его отсутствие не блокирует карточку, но worker обязан вернуть состояние этапа, а не молча пропустить раздел: `found`, `not_found`, `not_searched` или `needs_review`.

Каждая запись `awards[]` содержит:

- `competition_name`, нормализованный `competition_key`, `competition_edition_year` и `organizer_name`;
- `award_level`: `grand_gold`, `gold`, `silver`, `bronze`, `trophy`, `commended`, `score` или `other`;
- оригинальное название награды и редакционный перевод;
- категорию конкурса, если она указана;
- `wine_name_as_awarded` — название вина в форме конкурсного результата;
- `wine_vintage` либо `is_non_vintage`, если это указано источником;
- `score` и `score_scale`, если опубликованы баллы;
- дату/год присуждения;
- `source_url`, `source_type`, короткий `evidence_text`;
- `match_confidence` и причины сопоставления с canonical wine;
- `verification_status`: `official_result`, `producer_claim`, `secondary_source`, `conflicted` или `moderator_verified`.

Приоритет источников: официальный реестр/PDF конкурса; официальный сайт организатора; technical sheet или страница конкретного вина у производителя; однозначная публикация производителя. Отраслевой каталог допустим как lead, но не как единственное основание для публичной медали.

Нельзя автоматически переносить награду на всю линейку, другой винтаж или одноимённое вино; год конкурса нельзя считать винтажом. Поисковый snippet, магазин или нечитаемый значок медали на бутылке не являются достаточным подтверждением.

Награды не добавляются автоматически в публичное описание. После подтверждения они показываются отдельным блоком `Награды`, из которого можно открыть награду, выпуск конкурса и сведения об организаторе.

## 12. Редакционная спецификация WinePool

### 12.1. Общий тон

- спокойный, точный, современный;
- экспертный, но понятный неспециалисту;
- без восторженной рекламы;
- без сравнений «лучший», «идеальный», «непревзойдённый»;
- без обращения к читателю;
- без вымышленных историй и оценок.

### 12.2. Структура описания вина

1. Что это за вино и откуда оно.
2. Состав/метод производства, если подтверждены.
3. Характер стиля только по источникам или очевидной классификации.
4. Контекст подачи без категоричных гастрономических обещаний.

Рекомендуемая длина: 450–750 знаков. Короткая версия: 120–220 знаков.

### 12.3. Структура описания винодельни

1. Местоположение и история.
2. Масштаб/терруар/подход, если подтверждены.
3. Отличительная особенность хозяйства.

Рекомендуемая длина: 500–900 знаков.

### 12.4. Запрещённые паттерны

- «настоящий шедевр»;
- «идеальный выбор»;
- «подарит незабываемые эмоции»;
- неподтверждённые ноты вкуса и аромата;
- утверждения о наградах без источника;
- копирование предложения источника длиннее короткого evidence excerpt;
- шаблонные повторы между карточками.

### 12.5. Проверка оригинальности

Сравнить generated description с извлечёнными source passages. При слишком высокой текстовой близости выполнить одну дешёвую переформулировку. Факты при этом не меняются.

### 12.6. Вкусовой профиль и экологические признаки

Поля `sweetness`, `acidity`, `tannins`, `saturation` со шкалой 1–5 не являются извлекаемыми паспортными характеристиками. Copilot может подготовить предложение только по утверждённой шкале WinePool и должен вернуть для каждого значения:

- proposed value 1–5;
- confidence;
- основание: технические данные, несколько согласующихся tasting sources либо редакционная inference;
- короткое объяснение для модератора.

Inference не должна отображаться как подтверждённый факт. При отсутствии достаточных данных поля остаются пустыми или помечаются `не удалось оценить`; нельзя заполнять их случайным «средним» профилем. Для игристого вина танины по умолчанию не выводятся из одного только цвета или сорта.

`is_organic`, `is_biodynamic`, `is_natural` и другие ecological claims предлагаются только при явном подтверждении официальным источником, сертификацией или этикеткой. Маркетинговые формулировки вроде «бережное виноделие» не являются подтверждением. Модератор видит источник и подтверждает каждый флаг отдельно.

Нормализация методов, approaches, certifications и product features описана в связанном `wine_production_attributes_certifications_tz_2026_07_12.md`. Copilot не должен смешивать Pet-Nat, Demeter и Vegan в одну категорию `eco`.

#### Current implementation audit and required correction

На 12.07.2026 четыре nullable-поля уже присутствуют в `wines` и используются `get_user_analytics` для усреднённой паутины предпочтений. В `wine_details_screen.dart` отдельного блока профиля нет. При этом `add_edit_wine_screen.dart` инициализирует каждый slider значением `1`, если в базе `null`; это смешивает два разных состояния — «минимальное значение» и «нет данных». Финальная analytics RPC дополнительно применяет `COALESCE(..., 0)`, а chart заменяет null на `0.0`.

Требуемое поведение:

- сохранить `null` как полноценное состояние каждого измерения; `0` означает подтверждённое отсутствие признака, а не отсутствие данных;
- новая карточка вина не получает `1/1/1/1` автоматически;
- в форме добавить явный переключатель `Профиль подтверждён / Добавить профиль`, после включения доступны значения 1–5;
- сброс профиля возвращает все четыре поля в `null`;
- пользовательская аналитика усредняет только заполненные значения и показывает coverage: сколько бутылок участвовало в расчёте;
- если coverage недостаточен, паутина не рисуется либо показывается как `Недостаточно данных`, а не как нулевой профиль;
- карточка вина показывает только подтверждённые оси: частичный профиль отображается отдельными шкалами/индикаторами без пустых строк;
- компактная паутина показывается только когда профиль признан пригодным для публикации и заполнены все четыре оси; неизвестные оси нельзя дорисовывать нулями или соединять с центром;
- AI proposal хранится отдельно от canonical values до подтверждения модератором.

Для интерпретируемости требуется утвердить якоря шкалы 1, 3 и 5 для каждой оси. В частности, `sweetness` как сенсорное восприятие нельзя автоматически приравнивать к юридической категории `dry/brut`, а `tannins` для белых и игристых вин не следует заполнять фиктивным минимумом только ради завершённой диаграммы.

Итоговая шкала: `null = неизвестно`, `0 = подтверждённо отсутствует`, `1 = очень низко`, `2 = низко`, `3 = средне`, `4 = высоко`, `5 = очень высоко`. Confidence оценивается отдельно по каждой оси. Низкая уверенность в кислотности не должна скрывать независимо подтверждённую сладость или насыщенность.

UI contract карточки вина: использовать только шкальные индикаторы, не radar chart. Для каждого non-null значения показывается отдельная строка с названием и значением 0–5; для null строка полностью отсутствует. Пустые placeholder-шкалы и подпись `нет данных` внутри профильного блока не нужны; если все четыре значения null, весь блок скрывается.

### 12.7. Обязательная полнота research report

Модель и worker работают по versioned JSON contract. Ни один обязательный раздел нельзя опустить. Если сведения не найдены, раздел возвращает `status`, пустое/`null` значение и причину.

Обязательные разделы верхнего уровня:

- `identity`, `wine`;
- `taste_profile` с четырьмя независимыми осями, evidence и confidence;
- `winery`, `winery_location`, `winery_contacts`;
- `production_attributes`, `awards`;
- `editorial`, `media`, `sources`, `warnings`;
- `completeness`.

`winery_location` раздельно хранит legal, production и visitor address и координаты. `completeness` содержит checklist по каждому разделу: `complete`, `partial`, `not_found`, `not_applicable` или `not_searched`. Отчёт может быть готов при `not_found`, но не проходит schema validation, если обязательный раздел отсутствует. Поэтому координаты, e-mail, профиль и награды становятся явным результатом либо явным пробелом, а не зависят от памяти модели.

## 13. Изображения

### 13.1. Пользовательское фото бутылки

Допустимые автоматические операции:

- orientation correction;
- perspective correction;
- crop;
- background removal;
- лёгкая коррекция экспозиции/баланса белого;
- resize и web optimization;
- генерация preview.

Запрещено:

- дорисовывать отсутствующие элементы;
- менять этикетку или текст;
- генерировать новую бутылку по образцу;
- скрывать повреждения, способные изменить идентификацию.

Processed asset остаётся draft до подтверждения модератора.

### 13.1.1. Целевой semi-automatic bottle cutout workflow

Текущая ручная операция Photoshop должна быть заменена подготовкой draft-кандидата:

1. Сохранить исходное фото неизменным как moderation evidence.
2. Проверить, что найден главный объект `bottle`, а не только этикетка или бокал.
3. Исправить EXIF orientation и при необходимости перспективу.
4. Выполнить foreground segmentation без генеративной дорисовки.
5. Удалить фон и сохранить alpha mask.
6. Очистить только явно изолированные фоновые артефакты; не стирать пробку, фольгу, тонкие края и прозрачное стекло.
7. Обрезать по объекту с едиными безопасными полями, центрировать бутылку и привести к утверждённому canvas/aspect ratio WinePool.
8. Экспортировать lossless PNG или WebP с alpha; отдельно создать лёгкий preview.
9. Показать результат на тёмном, светлом и checkerboard фоне.
10. Модератор выбирает `Использовать`, `Подправить края`, `Другой кандидат` либо `Оставить исходник/загрузить вручную`.

Workbench media UX до автоматической обработки:

- тап по фото открывает полноразмерный viewer без возврата на страницу submission;
- viewer содержит `Скачать оригинал` и, когда существует, `Скачать обработанный`;
- скачивание сохраняет исходный файл без recompress/resize и с безопасным понятным именем (`submission_<id>_source.<ext>`);
- кнопка доступна только авторизованному модератору и не раскрывает приватный storage URL дольше необходимого;
- загрузка/скачивание и выбор canonical candidate записываются в audit;
- ошибки CORS/signed URL показываются явно, а не открывают пустую вкладку.

Автоматически публиковать processed asset нельзя. После одобрения он загружается в `wine_images`, а provenance связывает canonical asset с исходником, submission, processor/version и модератором.

### 13.1.2. Quality gates

Кандидат не предлагается как готовый, если:

- объект обрезан сверху или снизу;
- потеряны фольга, горлышко, контур или значительная часть прозрачного стекла;
- внутри бутылки появились прозрачные «дыры»;
- остался заметный прямоугольник исходного фона;
- присутствуют руки, ценник, полка или соседние предметы;
- разрешение недостаточно для карточки;
- маска имеет выраженный белый/цветной halo на тёмном фоне.

В этих случаях UI показывает предупреждение и сохраняет возможность ручной замены. Для первой версии достаточно автоматического cutout плюс выбора кандидата; встроенная кисть refine mask относится ко второму этапу.

### 13.1.3. Source and rights status

Факт загрузки изображения пользователем не доказывает право WinePool использовать его как постоянное каталожное медиа. Для каждого кандидата хранить `user_supplied`, `official`, `retailer`, `unknown` и URL/контекст происхождения, если известен. Фото с неизвестным или web/retailer происхождением допускается как moderation evidence и временный candidate, но его публикация требует решения модератора по принятой media policy. Предпочтение для canonical-карточки: собственное фото пользователя с необходимым согласием либо официальное product media с допустимыми условиями использования.

### 13.2. Web media

- не hotlink-публиковать;
- не переносить в canonical storage автоматически;
- показывать источник и rights status;
- скачивание/перенос выполнять только после действия модератора;
- логотип предпочтительно брать из official SVG/PNG;
- генеративная перерисовка логотипа запрещена;
- banner можно оставить отсутствующим: placeholder лучше недостоверного изображения.

### 13.3. Winery logo and hero candidates from the official website

Поддержать два разных media ownership mode:

1. `claimed_partner` — винодельня управляет расширенной/платной карточкой и сама загружает либо явно подтверждает logo, hero, gallery and brand usage rights.
2. `editorial_unclaimed` — WinePool создаёт базовую редакционную карточку (включая иностранные winery); worker только находит candidates на подтверждённом официальном домене, а модератор решает, использовать ли их.

Автоматический discovery на official domain:

- logo: JSON-LD `Organization.logo`, header/footer `<img>`, inline SVG, manifest icons, `apple-touch-icon`, favicon только как fallback;
- hero/banner: `og:image`, `twitter:image`, первый крупный hero/background image, official press/media kit, estate/about page;
- сохранить original URL, page URL, DOM/context selector, dimensions, MIME, file hash, discovery method and timestamp;
- отсеять cookie banners, stock placeholders, blog thumbnails, team portraits, bottle packshots and images smaller than configured thresholds;
- deduplicate identical files and rank transparent SVG/PNG highest for logo, wide high-resolution estate/architecture/vineyard media highest for hero;
- logo candidate может получить automatic transparent trim/preview, но не генеративную перерисовку;
- hero candidate допускает crop proposals под WinePool aspect ratios, но original pixels сохраняются и moderator controls crop/focal point.

Workbench показывает 3–5 candidates в каждой группе с official source link, preview на светлой/тёмной теме, resolution, crop preview и actions `Использовать`, `Скачать`, `Открыть источник`, `Загрузить другой`. Candidate не переносится в canonical storage до явного действия.

Rights model не следует выводить из технической доступности файла. `official_domain` означает provenance, но не автоматически reusable license. Хранить отдельно `rights_status`: `unknown`, `editorial_reviewed`, `partner_granted`, `licensed`, `rejected`. Для claimed partner публикация требует brand/media confirmation в договоре или кабинете. Для editorial unclaimed применяется утверждённая conservative media policy; при сомнении оставить initials/logo placeholder и нейтральный gradient hero.

Платность относится к управлению и расширению карточки, а не к истинности каталога: партнёр получает self-service, verified badge, richer wiki/tourism/media and analytics, но не может менять canonical facts без governance. Иностранные и непартнёрские winery сохраняют бесплатную базовую редакционную карточку без обещания полного media coverage.

## 14. Предлагаемая модель данных

Имена являются рекомендуемыми и могут быть уточнены при миграции.

### 14.1. `catalog_research_jobs`

```sql
id uuid primary key
submission_id uuid not null references draft_catalog_submissions(id)
requested_by_user_id uuid not null references profiles(id)
parent_job_id uuid null references catalog_research_jobs(id)
mode text not null -- standard | extended | retry
status text not null
stage text null
input_snapshot jsonb not null
input_fingerprint text not null
prompt_version text not null
schema_version text not null
model_policy_version text not null
soft_budget_rub numeric(10,2) not null default 20
hard_budget_rub numeric(10,2) not null default 100
reserved_cost_rub numeric(10,2) not null default 0
actual_cost_usd numeric(12,6) not null default 0
actual_cost_rub numeric(10,2) not null default 0
fx_usd_rub numeric(10,4) not null
attempt_count integer not null default 0
error_code text null
error_safe_message text null
started_at timestamptz null
completed_at timestamptz null
created_at timestamptz not null
updated_at timestamptz not null
```

Ограничение: не более одного активного standard job на submission/input fingerprint.

### 14.2. `catalog_research_sources`

```sql
id uuid primary key
job_id uuid not null references catalog_research_jobs(id) on delete cascade
source_type text not null
source_tier text not null
url text null
resolved_url text null
domain text null
title text null
language text null
is_official boolean not null default false
content_fingerprint text null
metadata jsonb not null default '{}'
retrieved_at timestamptz null
created_at timestamptz not null
```

Не хранить полные копии страниц без необходимости. Достаточно нормализованных facts и коротких evidence excerpts.

### 14.3. `catalog_research_evidence`

```sql
id uuid primary key
job_id uuid not null references catalog_research_jobs(id) on delete cascade
source_id uuid null references catalog_research_sources(id) on delete cascade
field_path text not null
excerpt text null
normalized_value jsonb null
evidence_kind text not null
strength numeric(5,4) not null
metadata jsonb not null default '{}'
created_at timestamptz not null
```

### 14.4. `catalog_research_suggestions`

```sql
id uuid primary key
job_id uuid not null references catalog_research_jobs(id) on delete cascade
entity_type text not null -- wine | winery | region | grape | media
field_path text not null
suggested_value jsonb null
display_value text null
confidence numeric(5,4) not null
status text not null -- proposed | applied | edited | rejected | stale
conflict_payload jsonb not null default '[]'
validation_payload jsonb not null default '{}'
applied_by_user_id uuid null references profiles(id)
applied_at timestamptz null
moderator_value jsonb null
created_at timestamptz not null
updated_at timestamptz not null
```

### 14.5. `catalog_research_reports`

```sql
id uuid primary key
job_id uuid not null unique references catalog_research_jobs(id) on delete cascade
submission_id uuid not null references draft_catalog_submissions(id)
version integer not null
identification_payload jsonb not null
duplicate_candidates jsonb not null default '[]'
verified_fact_pack jsonb not null default '{}'
wine_payload jsonb not null default '{}'
winery_payload jsonb not null default '{}'
editorial_payload jsonb not null default '{}'
media_payload jsonb not null default '[]'
warnings jsonb not null default '[]'
quality_score numeric(5,4) null
created_at timestamptz not null
```

### 14.6. `catalog_research_usage`

```sql
id uuid primary key
job_id uuid not null references catalog_research_jobs(id) on delete cascade
step text not null
provider text not null
model text null
request_id text null
input_tokens integer null
cached_input_tokens integer null
output_tokens integer null
tool_calls integer not null default 0
cost_usd numeric(12,6) not null
cost_rub numeric(10,2) not null
latency_ms integer null
created_at timestamptz not null
```

### 14.7. Reusable winery knowledge

После подтверждения модератором можно обновлять отдельную curated-таблицу `catalog_research_knowledge`:

- entity type/id;
- normalized facts;
- approved source references;
- approved description facts, но не обязательно сам текст;
- moderator id;
- validity/review timestamps;
- knowledge version.

Неподтверждённый AI report не попадает в reusable trusted knowledge.

### 14.8. Нормализованный каталог конкурсов и наград

Research report сначала хранит награды в JSON и suggestions. После подтверждения модератором данные переносятся в нормализованные сущности:

```sql
award_competitions (
  id, canonical_name, organizer_name, website_url,
  description, country_code, created_at, updated_at
)

award_competition_editions (
  id, competition_id, edition_year, starts_at, ends_at,
  official_results_url, created_at, updated_at,
  unique (competition_id, edition_year)
)

award_definitions (
  id, competition_id, code, display_name_ru, original_name,
  level_rank, description, created_at, updated_at,
  unique (competition_id, code)
)

wine_awards (
  id, wine_id, wine_vintage_id null, competition_edition_id,
  award_definition_id null, category_name null,
  score null, score_scale null, wine_name_as_awarded,
  source_url, evidence_text, verification_status,
  verified_by_user_id, verified_at, created_at, updated_at
)
```

Справочник конкурса объясняет, кто его проводит, как устроены уровни и что означает медаль. `wine_awards` хранит факт результата. Связь с `wine_vintage_id` предпочтительна; до реализации vintage entity допускается research-only винтаж без публичного переноса награды на все выпуски вина.

Canonical writes выполняются только модератором. Для названий конкурсов и наград потребуется alias layer, чтобы сокращения, оригинальные и локализованные названия не создавали дубли.

## 15. Пример контрактов

### 15.1. Создание job

`POST /functions/v1/catalog-research-start`

```json
{
  "submission_id": "uuid",
  "mode": "standard",
  "soft_budget_rub": 20,
  "hard_budget_rub": 100
}
```

Ответ:

```json
{
  "job_id": "uuid",
  "status": "queued",
  "soft_budget_rub": 20,
  "hard_budget_rub": 100,
  "reused_report_id": null
}
```

### 15.2. Получение состояния

`GET /functions/v1/catalog-research-status?job_id=...`

Или предпочтительно подписка Supabase Realtime/read-only provider на `catalog_research_jobs` и report tables.

### 15.3. Apply suggestion

Apply является UI-операцией над workbench state. Серверная запись статуса suggestion используется для аналитики и выполняется отдельным idempotent RPC:

`admin_mark_catalog_research_suggestion(suggestion_id, action, moderator_value)`.

RPC не пишет в canonical entities.

### 15.4. Финализация

В существующий `normalizationPayload` добавить:

```json
{
  "research": {
    "report_id": "uuid",
    "job_id": "uuid",
    "prompt_version": "catalog-research-v1",
    "accepted_suggestion_ids": ["uuid"],
    "edited_suggestion_ids": ["uuid"],
    "rejected_suggestion_ids": ["uuid"],
    "actual_cost_rub": 8.4
  }
}
```

Контракт существующего RPC не требуется менять: данные могут войти в `p_normalization_payload`.

## 16. Backend architecture

### 16.1. Компоненты

1. Supabase Edge Function `catalog-research-start`:
   - проверка JWT/admin capability;
   - проверка submission;
   - deduplication/reuse;
   - budget reservation;
   - создание job.
2. Worker:
   - lease job;
   - orchestration steps;
   - provider calls;
   - retries;
   - schema validation;
   - сохранение report.
3. Edge Function/RPC для status/feedback.
4. Flutter repository/provider и UI panel.
5. Scheduled cleanup временных assets и зависших leases.

### 16.1.1. Provider gateway

Worker не должен зависеть от SDK или API одного поставщика. Между orchestration и внешними моделями вводится интерфейс `AiProviderGateway`:

- `listCapabilities()`;
- `createStructuredResponse()`;
- `analyzeImage()`;
- `estimateCost()`;
- `healthCheck()`;
- `getRetentionProfile()`;
- `getUsage()`;
- `cancel()` при поддержке.

Первой production-реализацией должен быть прямой OpenAI-compatible adapter к Cloud.ru Evolution Foundation Models. Модель выбирается серверной конфигурацией, а не Flutter-клиентом. OmniRoute реализуется только как опциональный experimental adapter и не находится в критическом пути production.

OmniRoute рассматривается как маршрутизатор, а не как источник юридических прав на использование upstream-модели. Для каждого provider connection отдельно проверяются:

- условия коммерческого и автоматизированного server-side использования;
- разрешённые продукты/сценарии OAuth subscription;
- rate limits и fair-use;
- data retention upstream-провайдера;
- доступность image input, tools и structured output;
- стабильность refresh token без интерактивного входа;
- фактический usage/cost accounting.

Бесплатное соединение, предназначенное только для IDE/CLI/личного coding assistant, не используется для production-модерации WinePool без явного разрешения условий сервиса.

Если после эксперимента будет принято отдельное решение запускать OmniRoute на VPS, обязательно:

- контейнер или systemd service без публичного dashboard;
- bind API/dashboard только на `127.0.0.1` либо private Docker network;
- собственный scoped API key между worker и router;
- отключение `Unprotected` режима;
- credentials/SQLite вне git с минимальными filesystem permissions;
- health check, автоматический restart и алерт на expired OAuth;
- запрет логирования изображений и полных prompts;
- pin версии образа/пакета и контролируемое обновление;
- прямой fallback при недоступности router.

Если выбранная Kiro-модель предоставляет только chat endpoint, vision extraction выполняется отдельным OCR/vision provider. Наличие названия Claude-модели в router не считается доказательством поддержки image input или полного Anthropic API contract.

### 16.2. Почему не выполнять весь workflow одной Edge Function

Исследование может занимать 30–180 секунд, включать повторные внешние запросы и требовать resumability. Worker с persisted state обеспечивает:

- отсутствие зависимости от короткого HTTP timeout;
- безопасные retry;
- cost accounting после каждого шага;
- отмену и продолжение;
- наблюдаемость.

Допустимый MVP-компромисс — background worker в существующем серверном окружении. Новую тяжёлую платформу очередей внедрять не обязательно: lease можно реализовать через Postgres RPC с `FOR UPDATE SKIP LOCKED`.

### 16.3. Idempotency

Ключ: `submission_id + input_fingerprint + mode + prompt_version + schema_version`.

Повторный start с тем же ключом:

- возвращает active job;
- либо готовый свежий report;
- не списывает бюджет повторно.

### 16.4. Retry policy

- network/429/5xx: до 3 попыток с exponential backoff;
- invalid structured output: одна repair attempt дешёвой моделью;
- blocked source: не ретраить тот же URL многократно;
- budget exhausted: сохранить partial report;
- permanent validation failure: `failed_final` с безопасным сообщением.

## 17. Model policy и контроль стоимости

### 17.0. Управляемая конфигурация без релиза

Все денежные и операционные лимиты должны храниться централизованно на сервере и изменяться администратором без новой сборки Flutter и без перезапуска worker.

Рекомендуется таблица `catalog_research_settings` с singleton active-конфигурацией либо versioned rows:

```sql
id uuid primary key
version integer not null unique
is_active boolean not null default false
settings jsonb not null
changed_by_user_id uuid references profiles(id)
change_reason text not null
created_at timestamptz not null
```

Минимальные изменяемые параметры:

- `target_average_cost_rub`;
- `target_median_cost_rub`;
- `soft_budget_rub`;
- `confirmation_threshold_rub`;
- `hard_cap_rub`;
- `daily_budget_rub`;
- `monthly_budget_rub`;
- `max_web_search_calls`;
- `max_main_sources`;
- `max_retrieved_pages`;
- `safe_apply_confidence`;
- `report_freshness_days`;
- `standard_model`, `cheap_model`, `escalation_model`;
- `web_search_enabled`, `editorial_enabled`, `extended_mode_enabled`;
- `auto_continue_after_soft_budget`;
- `max_concurrent_jobs`.

Правила:

- job фиксирует snapshot версии настроек при старте; изменение конфигурации не меняет уже запущенную задачу;
- Flutter получает effective settings с сервера, но не принимает решения о бюджете;
- hard cap проверяется worker перед каждым платным вызовом;
- каждое изменение настроек имеет автора, время, старое/новое значение и причину;
- для критичных значений применяются DB constraints: положительные бюджеты и `soft <= confirmation <= hard`;
- предусмотрены `Сохранить`, `Восстановить рекомендуемые` и preview ожидаемого влияния;
- изменение model/provider secrets выполняется отдельно через secret storage, не через таблицу;
- при недоступности таблицы worker использует безопасную встроенную конфигурацию и запрещает extended mode.

В admin-зоне требуется экран `Настройки AI-исследования`, доступный только полному administrator, а не каждому reviewer.

### 17.1. Базовая политика

- mini multimodal model — vision, extraction, synthesis и основной editorial draft;
- nano/наиболее дешёвая подходящая модель — query normalization, простая классификация и schema repair;
- stronger model — запрещена в standard job по умолчанию либо доступна только при наличии бюджета и сложного конфликта;
- web search — только через ограниченный server-side tool;
- model names хранятся в конфигурации, не зашиваются в Flutter.

### 17.1.1. Рекомендуемый provider shortlist для российского production

Утверждённая база production — Cloud.ru Evolution Foundation Models как коммерческий OpenAI-compatible API, доступный из российской инфраструктуры. Актуальный каталог включает внешние Claude, GPT и Gemini, а также Qwen, DeepSeek, GigaChat и другие модели. Конкретная доступность и цена фиксируются capability snapshot перед пилотом.

Рекомендуемая стартовая маршрутизация:

- `vision_ocr`: Claude Haiku 4.5 Vision либо специализированный DeepSeek-OCR-2;
- `fact_extraction`: Claude Haiku 4.5;
- `source_comparison`: Claude Haiku 4.5, escalation в Claude Sonnet 4.5;
- `editorial_generation`: Claude Sonnet 4.5;
- `editorial_lint/schema_repair`: дешёвая модель уровня Haiku/GPT mini/Qwen;
- `web_search`: отдельный search adapter либо документированный web-search tool провайдера;
- `experimental_free`: Kiro Claude через OmniRoute только после terms/capability gate;
- `fallback`: минимум одна модель другого семейства и/или GigaChat API.

Почему Sonnet не используется на каждом шаге: качество финального анализа и текста сохраняется, но дорогой контекст страниц, OCR и механическое извлечение обрабатываются Haiku/специализированной моделью. Sonnet получает компактный verified fact pack.

Ориентировочная экономика по опубликованным Cloud.ru ставкам Claude 4.5, требующая проверки реальным usage:

- Haiku: 15K input + 2K output ≈ 4.9 RUB;
- Sonnet: 5K input + 1K output ≈ 5.9 RUB;
- три web-search вызова ≈ 5.9 RUB;
- суммарный ориентир ≈ 16.7 RUB без дополнительных retrieval/network расходов.

Это укладывается в target average 20 RUB при условии компактного fact pack и cache/reuse winery research.

Yandex AI Studio является не потребительской Алисой, а облачной платформой API с YandexGPT, Model Gallery, function tools, JSON schema и Search API. Она остаётся кандидатом для search/Russian text/fallback, но качество на WinePool golden set должно быть измерено до выбора основной editorial model.

GigaChat API поддерживает Vision, functions и structured data и рассматривается как российский fallback/benchmark. Дешёвые Qwen/DeepSeek/GPT-OSS через Cloud.ru используются только после golden eval: низкая цена не компенсирует ошибки идентификации и фактов.

Codex-модели не являются default для этого workflow: Codex оптимизирован прежде всего под программирование. Для винного research предпочтительнее general-purpose multimodal GPT/Claude/Gemini; Codex не даёт продуктового преимущества, соразмерного цене.

### 17.1.2. Golden model routing evaluation

До фиксации конкретных model ids прогнать одинаковый набор из 10–15 реальных бутылок минимум через:

- Claude Haiku 4.5;
- Claude Sonnet 4.5;
- GPT 5.4 Mini;
- одну Gemini Flash multimodal модель;
- GigaChat Pro/Max;
- одну дешёвую Qwen/DeepSeek/GPT-OSS модель для вспомогательных шагов.

Оценка:

| Метрика | Вес |
|---|---:|
| Правильная идентификация | 25% |
| Точность структурированных полей | 25% |
| Отсутствие неподтверждённых фактов | 20% |
| Качество использования источников | 10% |
| Соответствие WinePool editorial guide | 10% |
| Фактическая стоимость | 5% |
| Latency и стабильность | 5% |

Стоимость не является главным критерием: модель за 3 RUB, требующая десяти минут исправлений, считается хуже модели за 20 RUB с почти готовой карточкой.

Результатом eval является не обязательно одна победившая модель, а versioned routing policy по этапам. Она может изменяться через `catalog_research_settings` без релиза приложения.

### 17.1.3. Production и experimental контуры

Production:

- прямые официальные API credentials Cloud.ru;
- измеряемая стоимость каждого вызова;
- provider SLA/rate limits;
- гарантированный fallback между официально подключёнными моделями;
- реальные пользовательские данные разрешены только после retention review.

Experimental:

- OmniRoute/Kiro и другие subscription/free connections;
- synthetic или специально отобранные обезличенные тестовые данные;
- отсутствие гарантий доступности;
- результаты не используются для автоматического заполнения production workbench;
- соединение может быть отключено одной настройкой;
- переход в production возможен только после terms, retention и capability gate.

Бесплатный experimental маршрут полезен как benchmark и способ снизить стоимость разработки, но экономия нескольких рублей не должна создавать риск остановки модерации, истечения OAuth или нарушения условий upstream-сервиса.

### 17.2. Cost guard

Перед каждым внешним вызовом worker рассчитывает worst-case estimated cost.

Вызов разрешён только если:

`actual_cost_rub + reserved_next_call_rub <= hard_budget_rub`.

После soft budget следующий вызов разрешается только если orchestration policy считает его полезным для незакрытого критичного поля. Перед переходом через 60 RUB требуется подтверждение модератора. При достижении hard budget job завершается как `budget_exhausted` с partial report.

### 17.3. Валютный курс

- курс USD/RUB фиксируется в job при старте;
- обновляется сервером не чаще одного раза в сутки;
- применяется защитный коэффициент минимум 1.15 к расчётной стоимости;
- при недоступности курса используется последний сохранённый курс плюс коэффициент 1.25;
- бюджет контролируется в RUB, usage хранится и в USD, и в RUB.

### 17.4. Распределение бюджета standard job

Рекомендуемый ориентир средней стоимости 20 RUB:

- vision/OCR: до 3 RUB;
- web search/retrieval: до 8 RUB;
- fact extraction/entity resolution: до 3 RUB;
- editorial generation: до 2 RUB;
- verification/repair: до 2 RUB;
- safety reserve: 2 RUB.

Это не фиксированный тариф, а policy envelope. Реальная стоимость провайдера должна рассчитываться по текущей конфигурации.

### 17.5. Способы экономии

- local-first exact matching;
- cache по barcode и source fingerprint;
- reuse подтверждённой winery knowledge;
- уменьшенные изображения вместо originals, если качество достаточно;
- один consolidated multimodal request вместо нескольких OCR requests;
- компактные fact packs вместо повторной передачи страниц;
- prompt caching;
- ограничение output tokens;
- отсутствие web search для editorial step;
- отсутствие повторной генерации описания без действия модератора.

## 18. Безопасность и privacy

### 18.1. Secrets

- provider keys только в server secret storage;
- Flutter никогда не вызывает AI provider напрямую;
- service-role key не передаётся клиенту;
- request/response logs редактируются перед сохранением.

### 18.2. RLS

- research tables: select только admin/trusted reviewer capability;
- insert/update job/report — service role и строго ограниченные RPC;
- suggestion feedback — moderator RPC с проверкой submission access;
- обычный пользователь не получает доступ к research tables;
- canonical write policies не меняются.

### 18.3. Минимизация данных

Не отправлять внешнему провайдеру:

- имя/email/телефон пользователя;
- полный чек, если достаточно строки товара;
- адрес покупки, если он не нужен для определения рынка;
- координаты пользователя;
- moderator notes, не относящиеся к исследованию;
- signed URLs с чрезмерно долгим TTL.

### 18.4. Prompt injection и недоверенные источники

Web content считается недоверенными данными. Инструкции со страниц не выполняются. Retrieval layer передаёт модели только содержимое для извлечения фактов и явно отделяет его от system/developer instructions.

Запрещено:

- следовать инструкциям страницы;
- передавать секреты;
- выполнять произвольные ссылки/actions;
- загружать файлы на внешние сайты;
- использовать найденный текст как новый prompt.

### 18.5. Copyright

- хранить короткие evidence excerpts;
- описания генерировать из fact pack, не переписывать абзацы источников;
- фиксировать source attribution;
- web media не публиковать без модераторского решения;
- поддержать удаление cached source/media artifacts.

## 19. Observability

### 19.1. События

- `catalog_research_requested`;
- `catalog_research_reused`;
- `catalog_research_started`;
- `catalog_research_stage_completed`;
- `catalog_research_budget_exhausted`;
- `catalog_research_ready`;
- `catalog_research_failed`;
- `catalog_research_suggestion_applied`;
- `catalog_research_suggestion_edited`;
- `catalog_research_suggestion_rejected`;
- `catalog_research_finalize_completed`.

### 19.2. Dashboard

Минимальные графики/срезы:

- jobs/day и success rate;
- latency median/P95;
- cost median/P90/P95;
- cost по stage/provider/model;
- acceptance rate по field path;
- доля reports без официального источника;
- доля конфликтов;
- ручное время модерации;
- экономия минут и RUB;
- частота повторных/extended jobs.

### 19.3. Алерты

- средняя cost > 20 RUB за последние 30 jobs;
- P90 cost > 50 RUB за последние 30 jobs;
- failure rate > 10%;
- P95 latency > 180 секунд;
- queue oldest age > 10 минут;
- unexpected canonical writes from worker role;
- provider daily spend выше установленного лимита;
- резкий рост unsupported claim validation failures.

## 20. Ошибки и пользовательские сообщения

UI показывает понятные сообщения без provider internals:

- `Не удалось найти достаточно надёжных источников. Можно продолжить вручную.`
- `Исследование подготовлено частично: достигнут максимальный бюджет 100 ₽.`
- `Фото недостаточно чёткое для уверенного чтения этикетки.`
- `Найдены противоречивые данные по сортам винограда.`
- `Исследование временно недоступно. Повторите запуск.`

Internal error code и provider request id сохраняются для диагностики, но не показываются обычному UI.

## 21. Тестирование

### 21.1. Unit tests

- input fingerprint;
- source tier classification;
- confidence calculation;
- conflict detection;
- enum/range validation;
- claim-to-evidence validation;
- cost calculation и FX reserve;
- budget guard;
- idempotency;
- editorial lint.

### 21.2. Integration tests

- создание job авторизованным moderator;
- отказ обычному пользователю;
- worker lease и retry;
- partial report при budget exhaustion;
- reuse finished report;
- feedback RPC;
- запись research metadata в normalization payload;
- подтверждение отсутствия canonical writes до finalize;
- cleanup временных assets.

### 21.3. Golden dataset

Собрать минимум 30 кейсов:

- точный barcode match;
- вино существующей winery;
- новая winery и новое wine;
- кириллица/латиница и разные транслитерации;
- нечёткая этикетка;
- отсутствующий vintage;
- разные винтажи одного wine;
- private label;
- игристое/brut/extra dry;
- organic/biodynamic claims;
- конфликт сортов;
- конфликт крепости;
- отсутствие официального сайта;
- российские и зарубежные вина;
- дубли в текущем каталоге.

Для каждого кейса вручную зафиксировать expected identity, допустимые источники и критичные поля.

### 21.4. Red-team cases

- prompt injection на странице источника;
- поддельный «официальный» домен;
- SEO-страница с выдуманными характеристиками;
- barcode другого рынка;
- фото не вина;
- этикетка с несколькими языками;
- источник смешивает несколько винтажей;
- PDF со скрытым/битым текстовым слоем;
- очень длинная страница;
- source redirects на нежелательный домен.

### 21.5. Первый фактический Cloud.ru capability baseline от 12.07.2026

Инфраструктура:

- Cloud.ru Evolution Foundation Models;
- отдельный project-level service account;
- роль `ml_inference.ai_marketplace.apikey-limit.user`;
- service-scoped API key;
- endpoint `https://foundation-models.api.cloud.ru/v1`;
- OpenAI-compatible `/models` и `/chat/completions`;
- доступно 70 моделей на момент проверки;
- подтверждено получение `usage` и соблюдение strict JSON Schema.

#### Synthetic structured-output smoke test

На строке `Château Test, Bordeaux, 2021, 13.5% vol.`:

- `ai-sage/GigaChat3-10B-A1.8B` вернул корректный JSON, 1 078 токенов, ориентировочно 0.013 RUB;
- `openai/gpt-5.4-mini` вернул корректный JSON, 126 токенов, ориентировочно 0.044 RUB;
- обе модели подтвердили совместимость API и Structured Output;
- различия token accounting между провайдерами требуют опоры на фактический usage, а не только цену за 1M токенов.

#### Vision case A: хорошо читаемая лицевая этикетка

Кейс: российское вино Fanagoria «Авторское», декоративная кириллица, три сорта и винтаж 2025.

`openai/gpt-5.4-mini`:

- правильно распознал producer, wine name, Cabernet/Saperavi/Krasnostop, dry red и vintage;
- latency 2.7 s;
- 1 187 total tokens;
- ориентировочная стоимость 0.26 RUB.

`anthropic/claude-haiku-4.5`:

- правильно распознал producer, dry red и vintage;
- ошибся в слове `Авторское` и не извлёк сорта;
- latency 9.4 s;
- 1 596 total tokens;
- ориентировочная стоимость 0.41 RUB.

#### Vision case B: сложная пара front/back

Кейс: Château Tamagne, изогнутая этикетка, мелкий вертикальный текст, два изображения, blend Красностоп анапский/Анчелотта, production date вместо vintage, IGI и barcode.

`openai/gpt-5.4-mini`:

- правильно определил brand, wine name, оба сорта, red/dry, Russia, IGI `Кубань. Таманский полуостров`, ООО `Кубань-Вино`, 0.75 L и production date `06.05.2026`;
- корректно оставил vintage `null`;
- не извлёк alcohol level;
- допустил ошибку barcode: выдал 14-значный кандидат вместо визуального EAN-13 `4630037251203`;
- latency 8.0 s;
- 4 821 total tokens;
- ориентировочная стоимость 0.83 RUB.

`anthropic/claude-haiku-4.5`:

- правильно определил brand, dry, Russia/Taman, 0.75 L, production date и отсутствие vintage;
- ошибочно определил color как white;
- не извлёк grapes и manufacturer;
- выдал неверный barcode и не отметил ошибки в `uncertain_fields`;
- latency 14.6 s;
- 3 748 total tokens;
- ориентировочная стоимость 0.83 RUB.

#### Предварительный вывод

- GPT-5.4 Mini — текущий основной кандидат `vision_extraction` и `structured_fact_extraction`;
- Claude Haiku не исключается по двум кейсам, но не является default до расширенного golden eval;
- Claude Sonnet остаётся кандидатом для difficult verification и editorial generation;
- стоимость Vision менее 1 RUB на бутылку подтверждает достижимость target average 20 RUB полного workflow;
- barcode, alcohol, dates, volumes и числовые доли нельзя принимать только по генеративной Vision;
- barcode должен извлекаться специализированным decoder, проверяться EAN/GTIN checksum и сравниваться с OCR/Vision;
- Yandex OCR сохраняется как детерминированный первый слой;
- production route должен сравнивать OCR, barcode decoder и Vision, а не заменять один инструмент другим;
- полноразмерные изображения увеличивают input usage: добавить preprocessing/crop/resize и измерить его эффект.

Оба реальных изображения включаются в golden dataset с разрешением только для внутреннего тестирования WinePool и без публикации.

## 22. Acceptance criteria MVP

MVP считается готовым, если:

1. Moderator может запустить standard research из normalization workbench.
2. Job выполняется асинхронно и переживает закрытие страницы.
3. Повторный одинаковый запрос не создаёт дублирующие расходы.
4. Report содержит идентификацию, sources, suggestions, confidence, conflicts и usage.
5. Существенные factual suggestions имеют evidence.
6. Можно применить одно поле и все безопасные поля.
7. Ручные изменения workbench не затираются массовым apply.
8. Description создаётся только из verified fact pack и проходит editorial lint.
9. Ни один AI-компонент не может самостоятельно записать canonical wine/winery.
10. Finalize продолжает выполняться существующим RPC.
11. В normalization payload сохраняется report lineage.
12. Web media не публикуются автоматически.
13. Budget guard использует soft budget 20 RUB и абсолютный hard cap 100 RUB.
14. Продолжение исследования после 60 RUB требует явного подтверждения.
15. В admin UI видны фактическая стоимость и длительность.
16. RLS закрывает research data от обычных пользователей.
17. Реализованы usage logging и основные алерты.
18. Пройден пилот на 30 реальных кейсах.
19. Медианное активное время модератора не превышает 10 минут.
20. Средняя стоимость выборки не превышает 20 RUB, P90 не превышает 50 RUB, ни один job не превышает 100 RUB.

## 23. План реализации

### 23.0. Рекомендуемый следующий инкремент

Статус на 12.07.2026: первый исполняемый срез создан в `tool/catalog_research_eval/`. Реализованы прямой Cloud.ru adapter, versioned strict schema `label_extraction.v1`, versioned registry моделей/цен, фактический usage/cost report, GTIN checksum, golden manifest и локальные JSON/Markdown-отчёты без production writes. Первый smoke-прогон `openai/gpt-5.4-mini` на реальной этикетке LETO занял около 3 секунд и 0,31 RUB. Следующее расширение harness — Yandex OCR/barcode decoder fusion, image preprocessing и пакет из 10–15 golden cases; после стабилизации schema тот же adapter переносится в asynchronous worker.

Второй исполняемый срез от 12.07.2026 добавил полный evaluation-контур `Vision -> web search/source acquisition -> read-only catalog lookup -> moderation report`: Yandex Search API adapter, SSRF/redirect/content guards, moderator-seeded sources, PostgreSQL read-only duplicate lookup, strict `research_report.v1`, конфликты, source tiers, редакционное описание и совокупный budget accounting. На кейсе «Ведерниковъ Резерв Ркацители» полный проход занял около 6,4 секунды и 1,48 RUB. До production-включения поиска требуется отдельная роль/ключ Yandex Search API; текущий OCR key получает 403. Текущий pooler URL для роли `codex_readonly` также не принимает custom DB role, поэтому catalog gate корректно возвращает `needs_review`, пока не будет настроен совместимый read-only endpoint либо server-side RPC.

Обновление 13.07.2026: отдельный `YANDEX_SEARCH_API_KEY` со scope `yc.search-api.execute` и ролью сервисного аккаунта `search-api.webSearch.user` успешно прошёл прямой и end-to-end тест. Production-кандидат больше не использует публичный search fallback. Source selector ограничен одной страницей на домен, чтобы несколько URL одного магазина не считались независимыми подтверждениями. Автоматический прогон того же golden case занял около 7,3 секунды и 2,44 RUB с Vision, двумя Yandex search calls, загрузкой источников и editorial report.

В тот же день на production self-host Supabase применена миграция `20260713_add_catalog_research_duplicate_rpc.sql`. RPC `admin_catalog_research_find_duplicate_candidates` выполняет только чтение, доступна service role либо catalog admin, учитывает exact barcode/name/aliases, winery aliases, line, color/type/sugar и пересечение сортов, возвращает score/reasons и не меняет canonical data. Обычный authenticated-вызов без admin-роли получает отказ. Golden case после добавления вина вернул exact barcode candidate со score `1.0`; полный автоматический контур выбрал `exact_match` и правильный `wine_id` за 7,1 секунды и 2,76 RUB. Production transport — PostgREST/service role; SSH transport разрешён только evaluation harness для локального аудита self-host базы.

Второй foreign-wine golden case `Korenika & Moškon Festival Red` подтвердил необходимость передавать winery hint из submission отдельным полем, а не ожидать его от минималистичной лицевой этикетки; CLI получил `--winery`. RPC не нашла существующей карточки. Официальная страница маркирует текущий продукт как NV, тогда как retail-источники описывают 2022/2023 и расходятся по крепости и купажу. Без контрэтикетки безопасный report оставляет vintage, alcohol и grapes пустыми, фиксирует конфликт и не смешивает variant-specific facts. Полный calibrated pass стоил 1,31 RUB и занял 6,9 секунды. Также выявлен обязательный editorial language lint: schema явно требует русскоязычное описание, поскольку одной system-инструкции недостаточно для стабильного результата на иностранных источниках.

Не начинать сразу с большой AI-панели в Flutter. Первый code slice должен создать повторяемый и измеряемый backend/evaluation фундамент:

1. `tool/catalog_research_eval/` — CLI capability/golden harness без production writes.
2. Versioned JSON Schema для `label_extraction` и `research_report`.
3. Provider adapter для Cloud.ru с model registry и usage/cost accounting.
4. Image preprocessing: orientation, resize, label crop candidates без генеративного изменения содержимого.
5. Adapter существующего Yandex OCR.
6. Barcode decoder + EAN/GTIN checksum validator.
7. Fusion evaluator, сравнивающий OCR/barcode/Vision и создающий conflicts.
8. Golden cases manifest без хранения секретов и персональных данных.
9. HTML/Markdown comparison report: accuracy, field-level score, latency, tokens и RUB.
10. Прогон минимум 10–15 бутылок до выбора production routing policy.

Definition of Done этого инкремента:

- одна команда прогоняет выбранный кейс по выбранным моделям;
- секрет читается только из `.deploy/ai-research.local.env`;
- исходные изображения не попадают в logs/repo без явного добавления в approved dataset;
- strict JSON валидируется локально;
- invalid JSON и provider errors не ломают весь прогон;
- стоимость рассчитывается по versioned price registry и фактическому usage;
- barcode проходит checksum;
- отчёт показывает expected vs actual по каждому полю;
- можно сменить модель конфигурацией;
- hard budget тестового запуска задаётся параметром;
- отсутствуют любые записи в canonical WinePool tables.

После этого инкремента реализуются `catalog_research_jobs` и worker orchestration с уже проверенными контрактами. Flutter UI начинается только после стабилизации report schema.

### Этап 0. Подготовка и измерение baseline — 1–2 дня

- выбрать 30 реальных заявок;
- вручную замерить 10 контрольных обработок;
- утвердить editorial guide;
- зафиксировать текущие provider prices и FX policy;
- подготовить golden expected facts.

Результат: baseline и тестовая выборка.

### Этап 1. Data/backend skeleton — 2–4 дня

- миграции research tables и RLS;
- start/status/feedback RPC/functions;
- job lease;
- usage/cost service;
- idempotency и budget guard;
- mock worker без реального AI.

Результат: безопасный асинхронный контур.

### Этап 2. Research worker — 4–7 дней

- local-first catalog search;
- vision extraction;
- web search/retrieval;
- source ranking;
- fact extraction;
- entity resolution;
- confidence/conflict engine;
- structured report validation;
- retries и partial reports.

Результат: технический report по заявке.

### Этап 3. Editorial и media candidates — 2–4 дня

- verified fact pack;
- WinePool editorial generator/linter;
- short/long descriptions;
- пользовательский photo preprocessing preview;
- web media candidates без публикации.

Результат: готовые тексты и медиа-предложения.

### Этап 4. Flutter workbench UI — 4–6 дней

- repository/models/providers;
- job progress;
- report panel;
- evidence/conflict views;
- apply one/apply safe;
- duplicate candidates;
- cost display;
- extended research confirmation;
- localization RU и fallback EN при необходимости.

Результат: полный moderator workflow.

### Этап 5. Pilot и calibration — 3–5 дней

- прогон 30 кейсов;
- сравнение с ручным эталоном;
- настройка confidence thresholds;
- снижение лишних search calls;
- корректировка prompts/editorial lint;
- проверка затрат и времени;
- go/no-go review.

Итого ориентир MVP: 3–5 рабочих недель одного разработчика с учётом QA, либо быстрее при параллельной backend/UI работе.

## 24. Порядок выпуска

1. Dev environment, только test submissions.
2. Feature flag `catalog_research_copilot_enabled` для одного администратора.
3. Shadow pilot: report строится, но apply отключён.
4. Pilot с apply на 30 кейсах.
5. Проверка KPI, стоимости и ошибок.
6. Включение всем moderator/admin пользователям.
7. Отдельное решение о background prefetch для очереди только после доказанной экономики.

Kill switches:

- глобально отключить start;
- отключить web search, оставив local/vision;
- отключить editorial generation;
- запретить extended mode;
- установить daily provider spend cap.

## 25. Решения, которые нужно принять перед кодированием

1. Выбрать AI/search provider и проверить возможность оплаты/доступность из production-инфраструктуры.
2. Утвердить способ запуска worker: существующий сервер, отдельный lightweight service либо scheduled invocation.
3. Утвердить provider data retention policy для изображений этикеток.
4. Утвердить editorial guide на 10 хороших и 10 плохих примерах WinePool.
5. Определить, нужен ли `vintage` как новое canonical поле Wine или пока остаётся в name/research metadata.
6. Определить правила юридического подтверждения web media.
7. Установить дневной и месячный бюджет пилота.

Рекомендуемые стартовые настройки:

- target average: 20 RUB;
- target median: 15 RUB;
- soft budget: 20 RUB;
- moderator confirmation threshold: 60 RUB;
- absolute hard cap: 100 RUB;
- daily pilot cap: 500 RUB;
- monthly pilot cap: 5 000 RUB;
- report freshness: 30 дней;
- maximum main sources: 3;
- maximum web searches: 6;
- safe apply threshold: 0.90;
- default feature flag: off.

### 25.1. Решения, утверждённые 12.07.2026

- лимиты должны управляться серверной versioned-конфигурацией без релиза приложения;
- target average: 20 RUB;
- soft budget: 20 RUB;
- moderator confirmation threshold: 60 RUB;
- absolute hard cap: 100 RUB;
- daily pilot cap: 500 RUB;
- monthly pilot cap: 5 000 RUB;
- maximum main sources: 3;
- maximum web searches: 6;
- safe apply threshold: 0.90;
- report freshness: 30 дней;
- worker рекомендуется разместить отдельным lightweight systemd-сервисом на существующем WinePool VPS;
- OpenAI API оплачивается отдельно от подписки ChatGPT;
- для API использовать отдельный OpenAI Project, отдельный restricted API key и отдельные spend limits;
- provider retention baseline: не передавать персональные данные, использовать `/v1/responses` с `store=false`, не создавать provider Files без необходимости, хранить оригиналы и отчёты только в WinePool;
- vintage не возвращается как одно поле в `wines`; проектируется дочерняя сущность vintage profile с переопределяемыми характеристиками.
- production AI routing строится напрямую на Cloud.ru Evolution Foundation Models;
- конкретные модели выбираются по golden eval, а не по бренду или только цене;
- OmniRoute допускается как опциональный experimental OpenAI-compatible gateway, но не входит в обязательный production path;
- Kiro OAuth/free connection допускается для локального технического spike, но не утверждается как production provider до проверки условий server-side/commercial use и capability tests;
- production routing обязан иметь платный/официальный fallback и не зависеть от одной OAuth-сессии;
- автоматически найденные и обработанные media остаются только предложениями; публикация возможна исключительно после просмотра/редактирования и подтверждения модератором;
- консервативная media policy, retention policy и рекомендованные бюджеты утверждены;
- editorial guide формируется анализом текущих catalog descriptions и затем проходит продуктовую калибровку владельцем WinePool;
- реализация `wine_vintages` утверждена как отдельный связанный этап.

### 25.1.1. Capability gate для каждой модели OmniRoute

Перед добавлением модели в production registry выполнить одинаковый набор тестов и сохранить результат:

1. OpenAI-compatible authentication и `/models`.
2. Обычный chat request.
3. Строгий JSON по WinePool schema, включая обработку malformed output.
4. Image input на front/back label; если не поддержан — capability `vision=false`.
5. Tool/function calling; если отсутствует, web search оркестрирует worker.
6. Русский editorial sample.
7. Максимальный контекст и output limit.
8. Timeout, 429 и retry behavior.
9. Usage fields и возможность рассчитать RUB cost.
10. OAuth refresh после перезапуска и через 24 часа.
11. Retention/terms review.
12. Golden comparison минимум на 10 бутылках.

Registry хранит не только model id, но и capability flags: `text`, `vision`, `json_schema`, `tools`, `streaming`, `usage`, `commercial_server_use_verified`.

### 25.2. Рекомендованная модель винтажей

`wines` остаётся идентичностью продукта без конкретного года. `offers.vintage`, `user_storage.vintage`, `user_tastings.vintage` и `reviews.vintage` продолжают описывать конкретную бутылку/предложение/опыт.

Добавляется `wine_vintages`:

```sql
id uuid primary key
wine_id uuid not null references wines(id) on delete cascade
vintage integer null
is_non_vintage boolean not null default false
description text null
alcohol_level numeric null
grape_composition jsonb null
serving_temperature text null
characteristics jsonb not null default '{}'
source_payload jsonb not null default '{}'
quality_status text not null default 'limited'
created_at timestamptz not null
updated_at timestamptz not null
unique (wine_id, vintage)
```

Ограничение должно гарантировать ровно один из вариантов: указан `vintage` либо `is_non_vintage=true`.

Правила наследования:

- общие стабильные свойства остаются в `wines`;
- vintage profile хранит только отличия конкретного урожая;
- если override отсутствует, UI использует базовое значение wine;
- grape composition с процентами хранится в нормализованной дочерней таблице на следующем этапе либо временно в валидируемом JSONB;
- offer по `(wine_id, vintage)` может ссылаться на соответствующий profile;
- отсутствие profile не блокирует существующие offers/cellar flows;
- миграция является additive и не требует немедленного backfill всех вин;
- AI создаёт только предложение vintage profile, а модератор подтверждает его;
- отдельная публичная визуализация различий винтажей является post-MVP.

### 25.3. Manual baseline case: Fidora Prosecco DOC Spumante Brut

Контрольный реальный проход 12.07.2026:

- старт: 18:10;
- карточка функционально готова примерно к 18:40;
- полный цикл с новой винодельней и вином: 25–30 минут;
- GPT-5.4 Mini vision extraction: 5.2 секунды, 1 526 tokens (922 input + 604 output);
- стоимость vision extraction по тарифу Cloud.ru на момент теста: около 0.67 RUB (0.136 input + 0.534 output), без учёта web search/orchestration;
- первичная AI extraction + web verification package: около 6 минут в текущем ручном orchestration;
- оставшееся время: проверка/перенос полей, создание winery, поиск и извлечение логотипа, выбор фонового изображения, подготовка bottle cutout, загрузка media и уточняющие проверки.

Дополнительные findings кейса: forced `tannins=1` из-за отсутствия nullable UI; смешение Pet-Nat/organic/biodynamic/natural; отсутствие structured Demeter/Vegan и production method; необходимость download original/processed media. Связанные решения вынесены в `wine_production_attributes_certifications_tz_2026_07_12.md`.

Для пилота сравнивать не только model latency, а moderator wall-clock time от открытия submission до успешной финализации. Целевой выигрыш достигается только если автоматизированы winery fact pack и media preparation, а не один текстовый research.

### 25.4. Manual baseline case: LETO Muscat 2023

Контрольный реальный проход 12.07.2026:

- старт: 19:04;
- завершение: около 19:28;
- полный цикл с новой винодельней и вином: 24 минуты;
- GPT-5.4 Mini vision extraction: 6.7 секунды, 1 937 tokens (1 068 input + 869 output), около 0.93 RUB;
- дополнительные операции: official product/winery verification, описание, winery contacts, подтверждение tourism availability, geocoding длинного дорожного адреса, сравнение двух coordinate candidates, media preparation и ручной перенос данных.

Finding: существующий `geocode-receipt` пригоден как общий address preview service, но для winery/visitor location нужен map confirmation и draggable marker; автоматический результат нельзя публиковать туристам без визуальной проверки.

### 25.5. Production Copilot baseline: Tosca Cerrada Palomino Fino En Rama

Первый полный прогон встроенной кнопки в production выполнен 13.07.2026 на заявке `Tosca Cerrada Palomino Fino En Rama` / `Akilia`.

Технический результат:

- полный путь `admin UI -> protected job RPC -> Edge Function -> Cloud.ru -> Yandex Search/source fetch -> report table -> admin UI` работоспособен;
- стоимость двух Cloud.ru model calls составила `1.96 RUB` и совпала с биллингом Cloud.ru;
- отображаемая стоимость пока не включает Yandex Search API, HTTP acquisition и инфраструктурную стоимость worker;
- реальная `total_cost_rub` должна складываться из `model_cost_rub + search_cost_rub + other_provider_cost_rub`; UI обязан показывать разбиение и итог;
- отчёт сохранился как `research-report.v2`, секции раскрываются в workbench;
- source pack включил PDF и вторичные страницы, но несколько запросов привели к одному домену Wine-Searcher.

Качественный результат недостаточен для цели `скопировать/применить без переписывания`:

- публичное описание содержит служебные фразы: упоминание заявки, объёма, отсутствия данных и необходимости проверки; такие сведения должны находиться только в moderator notes;
- описание винодельни сгенерировано как отчёт о неполноте (`контакты не подтверждены`), а не как готовый публичный текст; при недостатке фактов `winery.description` должен быть `null`, а причины — в `warnings/completeness`;
- отсутствуют подтверждённые website, visitor/production address, coordinates, phone, e-mail и winemaker;
- вкусовой профиль вернулся полностью пустым; UI не должен показывать визуально пустую раскрытую секцию: для каждой оси нужно отображать значение либо человекочитаемый статус `Нет данных` / `Не подтверждено`, а при полном отсутствии — общий явный статус;
- `awards.status=not_searched` честен, но следующий pipeline обязан действительно выполнить отдельный поиск наград;
- production methods/approaches/features содержат англоязычные фразы и смешивают исходный факт, классификацию и описание; contract должен возвращать canonical code + русскую display label + evidence;
- enum values `white`, `still`, `dry` нельзя показывать модератору как готовый текст: UI локализует их и сопоставляет с canonical enum формы;
- источники показываются как голые URL и дублируют домены; UI должен показывать title/domain/tier/official badge/supports fields, а orchestrator — ограничивать доминирование одного агрегатора;
- report не содержит field-level evidence, поэтому модератор не может быстро понять, какой источник подтверждает конкретное значение;
- заполненный report пока нельзя безопасно применить к форме: отсутствуют canonical IDs, suggestion records, validation against enums/references и состояния apply.

#### Обязательное разделение payload

Следующая версия контракта разделяет три слоя:

1. `catalog_values` — значения, готовые к сопоставлению и переносу в поля формы; никаких пояснений внутри значений.
2. `public_editorial` — только готовые публичные описания вина и винодельни; при недостатке фактов значение `null`.
3. `moderator_notes` — пробелы, конфликты, причины `null`, ограничения и вопросы для проверки.

Пояснение возле поля показывается через info icon/tooltip и никогда не копируется вместе со значением. Каждое suggestion содержит `value`, `display_value_ru`, `canonical_id/code`, `confidence`, `status`, `evidence_ids` и `moderator_note`.

#### Приоритет следующего инкремента

P0 — до появления Apply:

1. deterministic validation публичных описаний: запрет служебных оборотов и minimum editorial quality gate;
2. field-level evidence и явные статусы для каждого обязательного поля;
3. локализация enums/attribute codes и раздельные canonical/display values;
4. улучшение query planning: официальный домен производителя, contact/about pages, address/geocoding и отдельный award query;
5. source diversity: не более двух результатов одного secondary domain, приоритет официального производителя и документов;
6. полный cost accounting с Yandex Search calls и итоговой стоимостью;
7. contract/golden tests на текущем кейсе, включая запрет текста о неполноте в public descriptions.

P1 — Apply layer:

1. одиночное `Применить`;
2. checkbox + `Применить выбранное`;
3. `Применить уверенные` только для validated suggestions;
4. `Применить всё` с review dialog и исключением descriptions/media/awards/conflicts по умолчанию;
5. защита ручных изменений и статусы `предложено / применено / изменено / отклонено`.

Acceptance следующего прогона: модератор видит для каждого обязательного поля либо готовое локализованное значение, либо явный статус; публичные описания можно вставить без удаления внутренних комментариев; источники привязаны к полям; стоимость включает model и search components.

### 25.6. Реализация quality increment `research-report.v3`

14.07.2026 реализован и развёрнут P0-инкремент после production baseline:

- строгая JSON Schema `research-report.v3` разделяет `catalog_values`, `public_editorial` и `moderator_notes`;
- каждое каталожное предложение содержит clean value, русское display value, canonical code/ID, confidence, status, field-level `evidence_ids` и отдельную moderator note;
- для вкусового профиля, контактов и остальных полей UI показывает значение либо явный человекочитаемый статус, а не пустой раскрытый раздел;
- production attributes возвращаются структурированно и отображаются по-русски; награды имеют отдельный structured result;
- deterministic validator запрещает `exact_match` без реального candidate ID, нормализует основные enum labels и скрывает публичные описания, не прошедшие editorial quality gate;
- query plan включает product/technical sheet, официальный contact/about, локализованный запрос винодела/адреса и отдельный award query;
- source acquisition ограничивает один домен двумя результатами; отчёт хранит source ID, title, domain, tier, official flag и supported fields;
- стоимость job разбита на `model_cost_rub`, `search_cost_rub`, `other_provider_cost_rub`, `model_calls`, `search_calls`; `actual_cost_rub` содержит итог;
- дневной Yandex Search sync request учитывается как `0.488 RUB`, ночной как `0.366 RUB` на успешный API-вызов по тарифу на дату реализации;
- админка показывает итог и раздельно Cloud.ru/Yandex Search с количеством вызовов;
- миграция breakdown-полей применена на production VPS, Edge Function v3 и web admin развёрнуты.

Проверено автоматически: `flutter analyze`, release web build, `deno check`, SQL `BEGIN/ROLLBACK`, наличие production columns, загрузка Edge Function и unauthenticated endpoint smoke. Полноценный quality smoke с реальной заявкой требует авторизованного повторного исследования в админке; до него фактическое качество нового model output не считается подтверждённым.

Следующий слой после успешного smoke: Apply UI с выбором полей, защитой ручных изменений и обязательной валидацией canonical references. Media preparation, geocoding confirmation и автоматическое применение наград остаются за пределами этого инкремента.

### 25.7. Реализация безопасного Apply-слоя

14.07.2026 реализован первый Apply-инкремент:

- у применимых AI-полей есть индивидуальная галочка и отдельное копирование значения;
- `Выбрать подтверждённые` выбирает только непустые значения со статусом `confirmed` и уверенностью не ниже 85%;
- пустые, конфликтующие и неподтверждённые значения автоматически не выбираются;
- координаты считаются применимыми только при наличии обеих частей; массовый выбор требует подтверждения обеих координат;
- перед применением показывается review dialog `текущее значение -> предложение AI`;
- `Применить выбранное` меняет только локальное состояние workbench и не вызывает DB write/finalization;
- страна, регион, сорта и поля новой винодельни заполняются в текущем workbench;
- параметры вина сохраняются как local overrides и передаются initial values в отдельный редактор canonical wine;
- сохранение винодельни, вина и финальное решение модерации остаются отдельными явными действиями;
- награды, media, неподтверждённые coordinates, production attributes без canonical target и отсутствующие значения в Apply не входят.

Следующий acceptance smoke должен проверить реальную заявку по маршруту: выбрать подтверждённые -> review -> применить -> создать/выбрать винодельню -> открыть редактор вина -> убедиться, что initial values перенесены -> сохранить вручную.

### 25.8. Source-selection regression: Akilia

Контрольный v3-прогон ошибочно вернул `not_found` для website/address/phone/email Akilia, хотя официальный `akiliawines.com/contactar.html` и реестр D.O. Bierzo содержат сайт, адрес, телефон, e-mail и Mario Rovira как contact/technical director. Root cause: результаты пяти поисковых intents объединялись в один список, после чего первые product/technical результаты исчерпывали общий лимит fetch до contact results.

Исправление 14.07.2026:

- contact intents выполняются первыми;
- каждому intent выделяется квота до двух уникальных страниц;
- сохраняется ограничение не более двух страниц одного домена;
- source pack расширен до десяти страниц и содержит intent каждой страницы;
- prompt запрещает считать контактный адрес основного хозяйства местом производства конкретного регионального проекта и запрещает tourism flag без прямого подтверждения визитов/дегустаций.

Для кейса Tosca Cerrada контакты Akilia в Ponferrada допустимы как общие контакты хозяйства/бренда, но не как автоматически подтверждённый production или visitor address вина из Cádiz. Для проверки нового source selection требуется повторное исследование, поскольку старый сохранённый report не обогащается задним числом.

### 25.9. Entity-resolution regression: Tosca Cerrada roles

Повторный прогон после source quota fix вернул canonical winery `Delgato Bulet, S.L.` и снова не нашёл контакты. Production audit показал, что модель использовала товарную страницу и техкарту, но смешала роли и исказила название `Delgado Zuleta`. Техкарта при этом явно указывает `Producer: Akilia`, Mario Rovira как автора проекта и Bodegas Delgado Zuleta как площадку ферментации/выдержки. Исходная заявка также содержала Akilia и согласованные country/region/product fields.

Root cause: контакты искались до того, как product sources раскрыли второго участника; финальный model call одновременно разрешал identity и писал отчёт, поэтому поздно выявленная сущность уже не получала собственного contact search. Дополнительно отсутствовало жёсткое правило не заменять producer/project на production facility или bottler.

Исправление:

- между первичным поиском и contact enrichment добавлен отдельный structured identity-resolution model call;
- resolver получает всю заявку, этикеточное извлечение и search snippets, а не одно название;
- роли разделены на `brand_or_project`, `winemaker`, `production_winery`, `bottler`, `unknown`;
- исходная винодельня считается сильным кандидатом и не заменяется без прямого противоречащего доказательства;
- для найденных identity subjects выполняются дополнительные локализованные contact queries;
- source pack допускает до 14 intent-balanced страниц;
- финальный prompt требует оставлять brand/project/author canonical winery, а площадку производства и bottler — в production attributes/moderator notes;
- стоимость теперь честно учитывает третий model call и дополнительные successful Search API calls.

Ожидаемый результат для Tosca Cerrada: Akilia/Mario Rovira остаётся автором/проектом карточки; Delgado Zuleta отображается как производственная площадка/партнёр, без искажения названия; контакты Akilia могут быть предложены как общие контакты, но адрес Ponferrada не публикуется как место производства вина из Cádiz или tourism point без дополнительного подтверждения.

Продолжение 13.09.2026. Роль `production_winery` получает место в каталоге: у вина появляется необязательное хозяйство-производитель. Результат разрешения ролей сохраняется в отчёт (`identity_roles`), а разбор заявки предлагает найденное хозяйство одним нажатием. Правило выше не меняется: бренд/проект остаётся винодельней карточки, bottler хозяйством не предлагается. ТЗ: `docs/wine_production_winery_tz_2026_09_13.md` §7.4–7.5.

### 25.10. Official-site contact traversal

Следующий контрольный прогон корректно определил Akilia и официальный домен, но сохранил `not_found` для адреса, телефона и e-mail. В отчёт попала страница `/en/wines.html`; её footer уже содержал e-mail, а меню — отдельную страницу контактов. Однако research worker очищал HTML до plain text, теряя значения `mailto:`/`tel:`, и не переходил по внутренним ссылкам найденного официального сайта.

Исправление 14.07.2026:

- для страниц contact intents worker извлекает внутренние ссылки `contact/contactar/contacto/contacts/contatti/kontakt/about/visit/degust`;
- дополнительно проверяется корень того же HTTPS-домена;
- обход ограничен тем же hostname и шестью дополнительными страницами на один research job;
- `mailto:` и `tel:` извлекаются из HTML до очистки и явно добавляются в evidence text;
- страницы обхода получают отдельный intent `official_site_contact_page` и попадают в проверяемый source pack;
- обход не создаёт новых Yandex Search API calls и увеличивает только сетевую работу worker.

Модель не должна угадывать контакты или пытаться найти их по косвенным агрегаторам, если официальный сайт уже определён: получение footer/contact page является детерминированным этапом до финального synthesis.

### 25.11. Progress UX

После запуска research клиент больше не блокируется в ожидании полного ответа Edge Function: создание job и длительное выполнение разделены. Workbench сразу показывает текущую задачу, опрашивает её состояние каждые четыре секунды и отображает пятиэтапный progress bar: очередь, анализ заявки, поиск источников, проверка сведений, подготовка отчёта. Worker явно записывает стадию `writing`, поэтому длительный финальный model call также виден модератору. Повторный запуск остаётся недоступен до терминального состояния текущего job.

### 25.12. Год основания винодельни

`wineries.founded_year` добавлен в полный AI-контракт как отдельное integer suggestion-поле с confidence, status и evidence. Если официальный источник прямо указывает год основания в истории или описании хозяйства, Copilot выносит его в отдельную строку `Год основания`; поле участвует в индивидуальном выборе, review диалоге и безопасном заполнении существующего контроллера создания винодельни. Год основания запрещено выводить из года запуска отдельного проекта, публикации страницы или конкурса и запрещено смешивать с винтажом вина.

### 25.13. Display name и линейка вина

Apply для собственных имён больше не использует технически нормализованную строку: `wine.canonical_name`, `wine.line_name` и `winery.canonical_name` переносятся в формы из `display_value_ru` с fallback на `value`. Финальный research prompt требует сохранять официальную орфографию и регистр в обоих полях.

`catalog_values.wine.line_name` выведен в раздел характеристик вина, получил confidence/evidence, индивидуальную галочку, review и Apply. Подтверждённое AI-название передаётся в существующий picker линейки редактора вина как предложение: модератор может выбрать совпавшую canonical line или создать новую. Линейка остаётся `null`, если источник не выделяет отдельную серию; винодельня, название вина, сорт, апелласьон и метод производства не считаются линейкой.

### 25.14. Область исследования

Research job поддерживает три пользовательских scope:

- `wine` — вино, дубль, характеристики, профиль, производство и награды; без contact queries и отдельного producer identity model call;
- `winery` — винодельня, история, год основания, винодел, контакты и расположение; без duplicate RPC, product/technical/awards queries и producer identity model call;
- `full` — полный существующий конвейер.

Старые `standard/extended` job сохранены в DB constraint и интерпретируются worker как `full`. Перед запуском и повторным запуском workbench показывает выбор области. Если при открытии панели уже выбрана canonical winery, UI по умолчанию предлагает `Только вино`; иначе — `Полное исследование`. Частичный отчёт скрывает не относящиеся к выбранной области секции. Worker сохраняет scope в `catalog_research_jobs.mode`, меняет набор Search API запросов, лимит source pack и final prompt, поэтому режим влияет на фактические расходы, а не только на отображение.

### 25.15. Гипотезы без фронтальной этикетки и catalog candidates

Контрольный wine-only кейс по штрихкоду `4630037251203` показал регрессию: worker нашёл точный verified/public кандидат внутреннего каталога, но модель оставила чековую аббревиатуру canonical name и спрятала Красностоп/Анчелотта в moderator note из-за отсутствия фронтальной этикетки.

Исправление:

- `null + confidence 0` означает только отсутствие содержательной гипотезы;
- гипотеза из согласующихся web-источников остаётся в поле со статусом `probable/unconfirmed`, ненулевой уверенностью и пояснением;
- точное совпадение barcode с внутренним каталогом обрабатывается детерминированно после model synthesis: выставляются `exact_match`, `duplicate_candidate_id`, canonical wine name, winery, grapes и доступные enum-характеристики кандидата;
- значения из exact barcode candidate остаются предложениями для решения модератора и не финализируют заявку автоматически;
- в режиме `wine` сохраняется хотя бы canonical winery name из заявки/этикетки/internal candidate, хотя контакты и история не исследуются;
- vision extraction явно проверяет ориентации 0/90/180/270 для вертикально снятых боковых и контрэтикеток;
- duplicate candidates сохраняют `catalog_quality_status` и `visibility_scope`, показывают их в UI и открывают реальную карточку WinePool по `wine_id`.

Блок `Возможные совпадения в каталоге` является внутренним поиском WinePool, а не внешним агрегатором. Он может видеть административно доступные записи шире обычного пользовательского поиска; поэтому статус видимости обязан отображаться явно.

### 25.16. Редакционный контракт названия и публичного описания

- Исходная строка чека используется из уже существующих данных заявки и не создаётся как ещё одно каталожное поле.
- `canonical_name` содержит чистое официальное собственное название вина либо короткую редакционную гипотезу с пониженной уверенностью. В него не входят объём, крепость, тара, цвет, сахар, тип, ЗГУ/ЗНМП, география, слова «российское вино» и чековые сокращения, если они не являются частью официального имени.
- Название винодельни хранится отдельно и не дублируется в названии вина без основания. `line_name` заполняется только для отдельной серии производителя.
- Публичное описание адресовано любителю вина и не содержит сведений о заявке, чеке, совпадениях, источниках, уверенности, модерации и подтверждении. Объём, штрихкод и прочие технические характеристики не дублируются в тексте.
- Сомнения, противоречия и происхождение выводов помещаются только в `moderator_notes` и основания по полям.
- После основного прохода quality gate проверяет название и описание. При чековом названии, служебных оборотах, слишком коротком или отсутствующем описании запускается короткий редакторский проход без повторного веб-поиска. Непрошедшее повторную проверку описание не предлагается к публикации.
- В интерфейсе исследования начальный режим — `wine`; `full` и `winery` модератор выбирает явно.

## 26. Definition of Done

- код и миграции прошли review;
- все новые таблицы имеют RLS и комментарии;
- provider keys отсутствуют в клиенте и git;
- unit/integration/golden tests зелёные;
- audit lineage сохраняется до финального decision payload;
- документация runbook и rollback готова;
- настроены cost/latency/error dashboards;
- проведён пилот и сформирован отчёт по 30 кейсам;
- подтверждены KPI времени, стоимости и качества;
- product owner подтвердил редакционный стиль;
- canonical governance не ослаблена.

## 27. Итоговый продуктовый принцип

AI в этом контуре не решает, что является истиной. Он сокращает стоимость поиска и ручного переноса информации, показывает модератору структурированную гипотезу и её доказательства. Истиной для каталога становится только подтверждённое решение модератора, прошедшее существующую атомарную финализацию WinePool.

Связанный naming/location backlog вынесен в `catalog_names_aliases_winery_locations_tz_2026_07_12.md`: AI Copilot обязан следовать canonical/display/alias policy и раздельно предлагать legal, production и visitor locations без автоматической tourism publication.
