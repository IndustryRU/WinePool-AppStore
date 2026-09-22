# Техническое задание: пользовательское AI-исследование вина и вклад в каталог WinePool

Дата: 14.07.2026  
Статус: product/technical feature source of truth; с 25.08.2026 массовый пользовательский rollout отложен в H6 активного [post-1.1.0 roadmap](/R:/Flutter/Project/winepool_final/docs/post_release_1_1_0_execution_roadmap_2026_08_25.md). Это не запрещает стабилизацию уже работающего internal/admin research.
Рабочее название функции: `Умный поиск WinePool` / `Исследовать вино`

Связанные документы:

- `docs/add_bottle_universal_flow_tz_2026_06_12.md` — существующий пользовательский flow добавления бутылки;
- `docs/ai_catalog_research_copilot_tz_2026_07_12.md` — действующий серверный AI research pipeline и модераторский отчёт;
- `docs/catalog_normalization_moderation_tz_2026_05_11.md` — нормализация и финальное решение модератора;
- `docs/draft_catalog_moderation_abuse_controls_tz_2026_04_21.md` — ограничения на заявки и контрибьюторов;
- `docs/catalog_names_aliases_winery_locations_tz_2026_07_12.md` — canonical/display/alias policy;
- `docs/mvp_catalog_contribution_gamification_2026_05_09.md` — опыт и вознаграждение за вклад.

---

## 1. Проблема и продуктовая возможность

Сейчас пользователь уже передаёт почти все исходные данные, необходимые AI Copilot:

- строку из чека либо введённое название;
- штрихкод;
- фотографию бутылки, фронтальной или задней этикетки;
- винодельню, страну, регион и характеристики, если они ему известны.

После отправки заявки модератор повторно открывает те же данные и вручную запускает исследование стоимостью ориентировочно 5–15 RUB. Исследование готовит структурированный отчёт, после чего модератор проверяет значения и создаёт canonical catalog entity.

Требуется перенести запуск того же исследования на пользовательскую сторону без передачи пользователю прав модератора:

1. пользователь не находит бутылку в каталоге;
2. WinePool выполняет дешёвый поиск и дедупликацию;
3. если совпадения нет, пользователь запускает AI-исследование;
4. получает понятную предварительную карточку;
5. подтверждает или дополняет результат;
6. отправляет результат как вклад в каталог;
7. модератор получает уже исследованную структурированную заявку;
8. после одобрения личная бутылка связывается с canonical wine.

Это не автономное добавление вина. AI создаёт исследовательский черновик, пользователь подтверждает намерение внести вклад, а каталог изменяет только существующий moderation workflow.

---

## 2. Цели

### 2.1. Пользовательские

- находить известное вино за 2–10 секунд по названию, штрихкоду или этикетке;
- получить полезный результат даже для отсутствующего в каталоге вина;
- сохранить бутылку в погребок сразу со статусом `На проверке`;
- видеть понятный прогресс исследования и вернуться к готовому результату позднее;
- участвовать в развитии каталога без необходимости самостоятельно заполнять сложную карточку.

### 2.2. Операционные

- сократить активное время модерации новой бутылки до 3–7 минут;
- не выполнять повторное платное исследование уже исследованной бутылки;
- повысить долю заявок с фронтальной/задней этикеткой и штрихкодом;
- превратить каждое подтверждённое исследование в переиспользуемый актив каталога.

### 2.3. Экономические

- целевая средняя переменная стоимость нового wine-only research: до 10 RUB;
- допустимый soft cap одного пользовательского исследования: 20 RUB;
- hard cap без дополнительного решения системы/администратора: 30 RUB;
- исключить AI-расход для найденных canonical wines;
- обеспечить глобальные дневные/месячные лимиты и per-user quota;
- измерить повторное использование результатов и фактическую стоимость одной новой опубликованной позиции.

---

## 3. Не входит в первый релиз

- автоматическая публикация AI-карточки без модератора;
- открытый пользовательский доступ к admin report, полным evidence и служебным заметкам;
- безлимитный AI-поиск;
- обязательная платная подписка до подтверждения продуктовой ценности;
- полноценное исследование винодельни по умолчанию;
- генерация или автоматическая публикация web-изображений;
- AI-рекомендации по здоровью, цене или инвестиционной ценности вина;
- автоматическое юридическое подтверждение сертификаций и наград.

---

## 4. Термины и сущности

- **Catalog search** — бесплатный поиск по canonical catalog и alias-слоям.
- **Bottle fingerprint** — нормализованный набор сигналов: barcode, OCR tokens, winery, name, vintage, country, image hash и другие признаки.
- **Discovery session** — пользовательская сессия распознавания бутылки до отправки вклада.
- **Research job** — существующий серверный AI job, запущенный с `audience=user` и scope `wine`.
- **Research cache** — переиспользуемый результат для одинаковой или достаточно похожей бутылки.
- **User preview** — безопасная сокращённая версия отчёта без внутренних служебных данных.
- **Contribution** — явное подтверждение пользователем отправки результата в каталог.
- **Moderation submission** — существующая заявка, дополненная ссылкой на research job/report.
- **AI credit** — право на один новый платный wine-only research; cache hit кредит не расходует.

---

## 5. Основной продуктовый принцип

```text
Сначала бесплатно ищем знания WinePool.
Платный AI вызываем только для неизвестной бутылки.
Результат AI не публикуется автоматически.
Подтверждённое исследование пополняет каталог и больше не оплачивается повторно.
```

---

## 6. Поисковая лестница

Backend обязан последовательно пройти дешёвые стадии и остановиться на первой достаточной находке.

### Stage 0. Нормализация ввода

- очистить OCR/чековые сокращения;
- извлечь barcode/GTIN;
- выделить winery, probable wine name, line, vintage, country, color/type/sugar;
- повернуть изображение для OCR в ориентациях 0/90/180/270;
- вычислить perceptual image hash;
- сформировать bottle fingerprint.

### Stage 1. Exact catalog lookup

- exact barcode;
- canonical wine ID, ранее выбранный пользователем;
- exact normalized alias;
- ранее подтверждённый fingerprint.

При exact match AI не вызывается. Пользователю сразу показывается canonical card.

### Stage 2. Ranked internal search

- wine aliases и localized names;
- winery aliases;
- wine line aliases;
- OCR/name token coverage;
- совместимость country/region/color/type/sugar/vintage;
- image similarity при наличии индекса изображений.

Результаты делятся на:

- `high >= 0.85` — показать главный кандидат и действие `Это моё вино`;
- `medium 0.55–0.84` — показать до пяти кандидатов;
- `low < 0.55` — не объявлять совпадением, но сохранить как evidence.

### Stage 3. Research cache lookup

До нового AI-вызова искать:

- completed research по exact barcode;
- completed research по fingerprint;
- незавершённые/готовые jobs того же пользователя;
- активные contributions других пользователей;
- недавно отклонённые результаты, чтобы не повторять заведомо неверный путь.

Cache hit не расходует AI credit и не создаёт новый model/search usage. Создаётся связь с существующим immutable report либо безопасный snapshot его пользовательской проекции.

### Stage 4. AI research

Только если stages 1–3 не дали достаточного результата. По умолчанию scope `wine`. Full winery research не запускается из стандартного flow.

---

## 7. Пользовательские сценарии

### 7.1. Штрихкод найден

1. Пользователь сканирует штрихкод.
2. Backend возвращает canonical wine.
3. Открывается компактная карточка.
4. Действия: `Добавить в погребок`, `Открыть карточку`.
5. Стоимость AI: 0.

### 7.2. Этикетка совпала с каталогом

1. OCR и internal ranking возвращают кандидатов.
2. Пользователь выбирает совпадение либо `Это не оно`.
3. Выбор сохраняется как label/alias learning signal после необходимых governance-проверок.
4. AI не вызывается.

### 7.3. Вино не найдено — бесплатный вклад

1. Экран объясняет: `Мы не нашли эту бутылку в WinePool. Исследовать и отправить в каталог?`.
2. Перед запуском показываются требования к данным и оставшиеся бесплатные исследования.
3. Research выполняется асинхронно.
4. Пользователь видит progress stages и может закрыть экран.
5. После завершения приходит in-app/push notification.
6. Показывается user preview.
7. Пользователь подтверждает/исправляет простые поля и нажимает `Подтвердить и отправить в каталог`.
8. Создаётся или обогащается draft/submission.
9. Бутылка появляется в погребке как `На проверке`.

### 7.4. Исследование готово, вклад не отправлен

- результат хранится как private discovery session в пределах retention policy;
- модерационная заявка автоматически не создаётся;
- WinePool может переиспользовать обезличенный fingerprint/cache только в пределах принятой privacy policy;
- пользователю отправляется одно мягкое напоминание;
- после retention срока приватные изображения и незавершённый preview удаляются.

### 7.5. Повторное исследование той же бутылки

- если есть completed compatible report — показать его без списания кредита;
- если job выполняется — открыть текущий progress;
- если report устарел или конфликтует с новыми evidence — предложить обновление, но новый платный запуск требует policy decision;
- concurrent duplicate jobs запрещены idempotency key.

### 7.6. Личное исследование без вклада

Post-pilot monetization:

- пользователь явно выбирает `Только для себя`;
- расходуется платный credit;
- результат не попадает в модерацию;
- canonical catalog не меняется;
- такой режим не получает бесплатную contribution-субсидию.

---

## 8. UX пользовательского preview

Пользователь не видит административный отчёт целиком.

### 8.1. Показывать

- предполагаемое чистое название;
- винодельню;
- страну/регион;
- цвет/type/sugar;
- сорта с нейтральной пометкой, если они требуют проверки;
- публичное редакционное описание;
- изображение пользователя;
- найденного canonical candidate;
- простую общую формулировку `Найдено уверенно / Нужна ваша помощь / Данных недостаточно`;
- вопросы, на которые пользователь может ответить по бутылке.

### 8.2. Не показывать

- raw moderator notes;
- внутренние prompt/model/provider данные;
- стоимость model/search calls;
- service URLs и непроверенные source snippets;
- внутренние catalog visibility statuses;
- технические decision codes;
- юридические/операционные комментарии модератора;
- возможность применить поля напрямую в canonical catalog.

### 8.3. Уточняющие вопросы

Вместо длинной формы показывать не более трёх вопросов за шаг:

- `Название совпадает с этикеткой?`;
- `Какой год указан на бутылке?`;
- `Сфотографировать контрэтикетку для состава?`;
- `Штрихкод читается верно?`;
- `Это одна из найденных карточек?`.

Ответ пользователя сохраняется отдельно от AI suggestion и виден модератору как user confirmation, но не становится истиной автоматически.

---

## 9. Состояния discovery session

```text
collecting_input
matching_catalog
candidate_review
research_offer
research_queued
research_running
research_ready
user_review
contribution_submitted
linked_to_submission
approved
needs_clarification
rejected
expired
failed
```

Переход `research_ready -> contribution_submitted` возможен только после явного действия пользователя.

---

## 10. Data model

Названия предварительные; при реализации сначала провести аудит существующих `draft_wines`, `draft_catalog_submissions`, `catalog_research_*`.

### 10.1. `wine_discovery_sessions`

```sql
id uuid primary key
user_id uuid not null
status text not null
entry_method text not null -- barcode | label | receipt | name | combined
input_snapshot jsonb not null
fingerprint_hash text null
barcode text null
image_phash text null
matched_wine_id uuid null
matched_research_job_id uuid null
draft_wine_id uuid null
submission_id uuid null
credit_source text null -- contribution_free | subscription | purchased | cache
credit_ledger_id uuid null
user_preview jsonb null
user_confirmations jsonb not null default '{}'
created_at timestamptz not null
updated_at timestamptz not null
expires_at timestamptz null
```

### 10.2. Расширение `catalog_research_jobs`

```sql
audience text not null default 'moderator' -- moderator | user
discovery_session_id uuid null
cache_key text null
reused_from_job_id uuid null
requested_by_context text null -- admin_workbench | user_contribution | private_research
```

Job пользователя всегда создаётся сервером. Клиент не может передавать произвольный provider/model/budget.

### 10.3. `ai_credit_ledger`

```sql
id uuid primary key
user_id uuid not null
delta integer not null
reason text not null
reference_type text null
reference_id uuid null
idempotency_key text not null unique
expires_at timestamptz null
created_at timestamptz not null
```

Баланс вычисляется по ledger либо поддерживается транзакционно materialized balance. Нельзя хранить только изменяемое число без истории.

### 10.4. Research cache

На MVP допускается использовать completed `catalog_research_jobs/reports` с индексируемыми `cache_key`, barcode и fingerprint. Отдельная таблица нужна, если появятся versioning/TTL/merge сложнее одного report.

Cache key не должен строиться только по нормализованной строке: одинаковые названия разных производителей не являются одинаковой бутылкой.

---

## 11. API и RPC

### `POST /wine-discovery/match`

Вход: entry method, barcode, OCR text, draft fields, media references.  
Выход: exact match, ranked candidates, cache hit либо research offer.

### `POST /wine-discovery/start-research`

Сервер атомарно:

1. проверяет auth, limits, abuse status и входные данные;
2. повторяет catalog/cache lookup;
3. резервирует credit;
4. создаёт idempotent research job scope `wine`;
5. связывает job с discovery session;
6. возвращает job/session status.

### `GET /wine-discovery/:id`

Возвращает только user-safe projection. RLS: владелец сессии либо admin.

### `POST /wine-discovery/:id/confirm`

Записывает подтверждения/исправления пользователя. Не изменяет research report.

### `POST /wine-discovery/:id/submit-contribution`

Транзакционно:

- проверяет состояние и required evidence;
- создаёт/обогащает draft wine;
- создаёт moderation submission;
- прикрепляет immutable research reference;
- создаёт user storage pending entry по текущим правилам;
- начисляет/резервирует XP без двойного начисления;
- меняет session status.

### `POST /wine-discovery/:id/cancel`

Отменяет только пользовательскую сессию. Уже понесённый provider cost не возвращается; unused reserved credit возвращается транзакционно.

---

## 12. Переиспользование admin AI Copilot

Не создавать второй независимый research pipeline.

Переиспользуются:

- image/OCR extraction;
- internal duplicate RPC;
- Yandex Search/source acquisition;
- Cloud.ru model adapter;
- strict report schema;
- naming/editorial quality gates;
- cost accounting;
- progress stages;
- retry/idempotency;
- admin workbench preview и apply после submission.

Добавляются:

- user-safe projection report;
- audience/context policy;
- discovery/cache layer;
- credit/entitlement gate;
- contribution handoff;
- пользовательские уведомления.

Модератор открывает ту же заявку и видит полный report, user confirmations и изменения пользователя относительно AI suggestions. Повторное исследование по умолчанию не запускается.

---

## 13. Кредиты и пилотная экономика

### 13.1. Рекомендуемый пилот

- 2 contribution credits после регистрации;
- затем 1 credit каждые 30 дней активному пользователю;
- cache hit и catalog match бесплатны и не расходуют credit;
- credit списывается только после фактического старта платного provider stage;
- failed job по технической причине возвращает credit;
- rejected contribution не возвращает уже потраченный credit, но не блокирует добросовестного пользователя автоматически;
- лимит пилота: не более 3 новых AI jobs на пользователя в сутки;
- глобальный daily/monthly RUB budget с автоматическим stop.

### 13.2. Contribution и private режим

- `contribution_free`: субсидируется WinePool и обязательно предлагает отправку в каталог;
- `subscription/purchased`: может остаться приватным;
- full winery research оценивается несколькими credits и отсутствует в MVP.

### 13.3. Будущие тарифы — гипотеза, не обязательство

- Free: каталог, barcode, internal label match, ограниченные contribution credits;
- Plus: месячный пакет wine research + retention-функции;
- credit pack: дополнительные исследования без подписки;
- extended winery report: отдельная стоимость/несколько credits.

Цены утверждаются только после пилота и фактической unit economics.

---

## 14. Anti-abuse и бюджетная безопасность

- только authenticated user может запускать provider research;
- CAPTCHA/device attestation для аномальной активности;
- rate limit по user/device/IP/fingerprint;
- один активный job на один fingerprint/user;
- server-side idempotency key;
- минимальное качество входа: barcode либо изображение достаточного качества либо содержательное название + winery;
- запрет свободного prompt и произвольных URL;
- provider keys только на сервере;
- content-type/size/dimension limits изображений;
- malware/storage guards;
- daily/monthly/user budget checks до job и между стадиями;
- circuit breaker при росте средней стоимости/error rate;
- contributor reputation влияет на quota, но не на каталожную истину;
- repeat offenders обрабатываются существующим anti-abuse governance.

---

## 15. Privacy, retention и права на изображения

- перед первым research пользователь принимает понятное уведомление об обработке фотографии внешними AI/search providers;
- не отправлять provider сведения о пользователе, магазине, цене, координатах и полном чеке, если они не нужны для идентификации вина;
- crop этикетки предпочтительнее полного пользовательского кадра;
- удалить EXIF и лишние метаданные;
- private незавершённые изображения имеют ограниченный TTL;
- contribution media публикуются только после существующего moderation/media review;
- web media никогда не публикуются автоматически;
- user-safe preview не раскрывает внутренние источники, содержащие потенциально чувствительные URL/token parameters.

---

## 16. Уведомления

- `Исследование началось` — только UI, без push;
- `Мы определили вино — проверьте результат`;
- `Нужна фотография контрэтикетки`;
- `Вино отправлено на проверку`;
- `Вино добавлено в каталог WinePool`;
- `Нужно уточнение модератора`;
- `Исследование не удалось, кредит возвращён`.

Уведомления idempotent, deep link ведёт в discovery/submission details.

---

## 17. Аналитика

События:

```text
wine_search_started
wine_search_catalog_match
wine_search_candidates_shown
wine_search_not_found
wine_research_offered
wine_research_started
wine_research_cache_hit
wine_research_ready
wine_research_preview_opened
wine_research_field_confirmed
wine_research_contribution_submitted
wine_research_abandoned
wine_research_failed
wine_research_moderation_approved
wine_research_moderation_rejected
wine_research_paywall_shown
wine_research_credit_purchased
```

Не отправлять в AppMetrica raw OCR, barcode, названия, e-mail, URLs и изображения.

Основные метрики:

- catalog match rate по barcode/label/name;
- доля cache hit;
- AI start rate после not found;
- research ready -> contribution submitted;
- contribution -> approved;
- средняя/P95 стоимость нового job;
- стоимость одной approved canonical position;
- moderator minutes на заявку с/без user research;
- повторное использование одного research;
- D1/D7/D30 retention пользователей, применивших smart discovery;
- conversion free credits -> paid product после запуска монетизации.

---

## 18. Ошибки и сообщения

- Нет сети: сохранить ввод локально и предложить повторить позже.
- Недостаточно данных: запросить фронтальную/заднюю этикетку, не списывать credit.
- Catalog candidate найден поздно: остановить provider pipeline по возможности, показать candidate.
- Budget exhausted: `Исследования временно недоступны. Обычный поиск продолжает работать`.
- Provider timeout: retry policy; после terminal failure вернуть credit.
- Conflict: показать нейтрально `Найдено несколько вариантов`, не выдавать гипотезу за факт.
- App closed: job продолжает выполняться, результат доступен по notification/history.
- Moderation rejected: объяснить пользовательскую причину безопасным текстом, не показывать внутреннюю заметку.

---

## 19. Feature flags и настройки

```text
user_wine_discovery_enabled
user_ai_research_enabled
user_ai_research_pilot_user_ids
free_contribution_credits_on_signup
free_contribution_credits_periodic
per_user_daily_job_limit
global_daily_budget_rub
global_monthly_budget_rub
user_research_soft_cap_rub
user_research_hard_cap_rub
research_cache_enabled
private_research_enabled
```

Настройки меняются без релиза приложения. Финансовые настройки доступны только administrator.

---

## 20. Этапы реализации

### Этап 0. Аудит и контракт

- аудит текущего catalog search, alias RPC, add-bottle и draft submission;
- зафиксировать fingerprint v1 и threshold evaluation set;
- определить user-safe report schema;
- утвердить retention/consent;
- подготовить feature flags.

### Этап 1. Быстрый поиск

- единая строка поиска;
- exact barcode lookup;
- scanner entry;
- alias-aware ranked catalog search;
- переход в карточку/добавление в погребок;
- аналитика search funnel.

### Этап 2. Discovery session и cache

- таблица/RLS/RPC;
- fingerprint;
- cache/idempotency;
- progress/history;
- user-safe preview без запуска monetization.

### Этап 3. Закрытый AI pilot

- whitelist 10–30 пользователей;
- два contribution credits;
- запуск существующего worker scope `wine`;
- preview, confirmations, submit contribution;
- handoff в admin workbench;
- бюджеты и dashboards.

### Этап 4. Оптимизация модерации

- diff AI/user/moderator;
- запрос уточнения/контрэтикетки;
- не запускать duplicate moderator research;
- измерить wall-clock и approval quality;
- cache approved knowledge.

### Этап 5. Ограниченный публичный запуск

- authenticated users;
- anti-abuse и global budget;
- уведомления;
- support/runbook;
- A/B количества free credits.

### Этап 6. Монетизация

- entitlements и credit packs;
- subscription experiments;
- private mode;
- extended winery report;
- store billing/refunds/reconciliation.

---

## 21. Критерии приёмки MVP

1. Известный barcode открывает canonical wine без AI job и без кредита.
2. Известная этикетка предлагает корректные catalog candidates.
3. Перед AI server повторно проверяет catalog и cache.
4. Два конкурентных запроса одной бутылки не создают два платных jobs.
5. Пользователь видит progress и может закрыть приложение.
6. User preview не содержит moderator notes, provider metadata и служебную лексику.
7. Без явного `Подтвердить и отправить в каталог` moderation submission не создаётся.
8. После отправки moderator видит полный report, исходные evidence и user confirmations.
9. Moderator approval использует существующую атомарную финализацию.
10. Pending bottle корректно превращается в canonical storage item.
11. Provider failure возвращает credit ровно один раз.
12. Cache hit не списывает credit.
13. Global budget stop не ломает бесплатный catalog search.
14. Обычный пользователь не может читать чужие sessions/reports.
15. AI не создаёт и не изменяет canonical catalog напрямую.
16. Все события стоимости и воронки записываются без персональных/raw wine данных в аналитике.

---

## 22. QA-матрица

- barcode: exact / absent / invalid / duplicate;
- label: Cyrillic / Latin / curved / rotated / glare / low resolution;
- input: только чек / только название / только фото / combined;
- catalog: public / hidden / merged / deleted / duplicate alias;
- cache: same user / other user / running / expired / rejected;
- job: success / timeout / partial / provider 4xx/5xx / budget stop;
- user: guest / authenticated / no credits / abuse restricted / pilot;
- lifecycle: close app / retry / double tap / notification deep link;
- moderation: approve / clarify / reject / merge with existing;
- security: RLS, forged job ID, arbitrary URL, oversized media, replayed idempotency key;
- accounting: reserve / consume / refund / cache no-charge / concurrent requests.

---

## 23. Rollout guardrails

Публичное расширение запрещено, пока не выполнены одновременно:

- не менее 50 pilot researches;
- не менее 70% research-ready результатов отправляются как contribution либо дают пользователю полезный catalog match;
- не менее 60% submitted contributions одобряются без полного повторного исследования;
- средняя стоимость нового research <= 10 RUB либо утверждена новая экономика;
- duplicate paid job rate < 1%;
- P95 cost <= hard cap;
- нет RLS/privacy инцидентов;
- moderator wall-clock статистически ниже baseline;
- global budget/circuit breaker проверены тестом.

---

## 24. Рекомендуемый первый вертикальный срез

Не начинать с подписок. Первый срез:

1. exact barcode + alias-aware catalog match;
2. `wine_discovery_sessions`;
3. server cache/idempotency;
4. закрытый запуск существующего wine-only research;
5. user-safe preview;
6. кнопка `Подтвердить и отправить в каталог`;
7. research reference в существующей заявке;
8. два бесплатных pilot credits и глобальный бюджет;
9. метрики стоимости, approval и времени модерации.

Этот срез проверяет главную гипотезу: пользователь получает мгновенную ценность, WinePool оплачивает примерно ту же работу, которую всё равно выполнил бы модератор, а заявка приходит существенно лучше подготовленной.

---

## 25. Итоговый принцип

WinePool не продаёт пользователю доступ к конкретной модели. WinePool даёт возможность распознать неизвестную бутылку и внести проверяемый вклад в общий каталог.

```text
Пользователь предоставляет бутылку и подтверждает результат.
AI структурирует исследование.
Модератор принимает каталожное решение.
Подтверждённое знание переиспользуется бесплатно для всех следующих пользователей.
```
