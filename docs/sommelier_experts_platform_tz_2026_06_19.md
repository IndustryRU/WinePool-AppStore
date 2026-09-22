# WinePool - ТЗ: Живые сомелье, экспертные профили и консультационные запросы

Дата: 19.06.2026

Статус: продуктово-техническое ТЗ для обсуждения и последующей реализации.

## 1. Короткая формула

WinePool не должен пытаться заменить сомелье искусственным интеллектом. AI помогает распознавать чеки, сопоставлять вина, строить аналитику и подсказывать пользователю путь. Живой сомелье дает то, что нельзя полностью автоматизировать: дегустационный опыт, личный вкус, контекст винтажа, доверие и человеческую речь.

Смысловое ядро направления:

> WinePool строится разработчиком, но должен обрести вкус вместе с сомелье.

Основатель проекта не должен притворяться винным авторитетом. Его сильная сторона - продукт, код, данные, пользовательские сценарии и инфраструктура. Сильная сторона сомелье - вкус, профессиональная память, язык вина и доверие. Направление `Живые сомелье` должно строиться как совместное создание культуры продукта, а не как найм обезличенных консультантов.

Новый слой WinePool:

```text
Пользователь -> вино / чек / погребок / событие -> запрос сомелье -> консультация -> отзыв / дегустация / повторный вход в WinePool
```

Это не алкомаркет, не продажа вина и не встроенный платежный маркетплейс на первом этапе. Это экспертно-образовательный слой поверх винного дневника, каталога, отзывов, карты покупок, партнерской ленты и WinePool Tourism.

## 2. Главные продуктовые решения

### 2.1. Сомелье - не отдельная auth-role

Не добавлять `sommelier` в `profiles.role` и не расширять auth-роли до `buyer/seller/sommelier/administrator`.

Оставить текущую модель:

- `buyer`
- `seller`
- `administrator`

Сомелье сделать capability layer через отдельную таблицу `expert_profiles`, связанную с `profiles.id`.

Почему:

- один человек может быть обычным пользователем, сомелье и владельцем бизнеса одновременно;
- это не ломает текущую role/business architecture;
- проще RLS: базовый аккаунт остается `profile`, профессиональная публичная витрина живет отдельно;
- можно модерировать экспертный статус без смены роли пользователя;
- можно скрыть или приостановить профиль сомелье без блокировки личного аккаунта.

Рекомендуемый термин в интерфейсе: `Живые сомелье` или `Эксперты WinePool`. Внутреннее доменное имя: `experts`.

### 2.2. Сомелье - персональный профессиональный профиль, а не business type

Не добавлять `sommelier` в `business_type` на MVP.

`businesses.type` должен оставаться для коммерческих организаций и точек:

- `winery`
- `retail`
- `horeca`
- позже возможно `tourism_operator`

Сомелье в MVP - это индивидуальный экспертный профиль поверх `profiles`. Если позже появятся агентства сомелье, винные школы или команды экспертов, их лучше вести через отдельную организационную модель или `businesses` после отдельного legal/product review.

### 2.3. Деньги на MVP идут напрямую сомелье

WinePool на MVP не принимает оплату от пользователя за консультацию и не перечисляет деньги сомелье.

MVP-модель:

- WinePool показывает экспертный профиль и помогает создать запрос;
- сомелье принимает или отклоняет запрос;
- условия, стоимость и способ оплаты фиксируются как договоренность между пользователем и сомелье;
- оплата происходит напрямую сомелье вне WinePool;
- сомелье самостоятельно выдает чек, если применимо;
- WinePool не хранит платежные данные, не проводит транзакции, не удерживает комиссию.

Это снижает риски:

- не становиться платежным агентом;
- не строить фискализацию на первом этапе;
- не отвечать за налоги сомелье;
- не смешивать консультационную услугу с продажей алкоголя;
- не превращать приложение в marketplace услуг раньше юридической вычитки.

## 3. Цели

### 3.1. Для пользователя

Пользователь должен:

- найти живого сомелье по стилю, специализации и опыту;
- открыть публичный профиль эксперта;
- увидеть, чем эксперт полезен: подбор, разбор бутылки, погребок, мероприятие, подарок, винный тур;
- отправить запрос из контекста вина, чека, погребка, отзыва или свободной формы;
- приложить фото, ссылку на wine_id, заметку, бюджет, повод, сроки;
- получить ответ, уточнения, итоговую рекомендацию;
- договориться об оплате напрямую с сомелье, если консультация платная;
- оставить оценку консультации;
- сохранить результат в погребок или заметки WinePool.

### 3.2. Для сомелье

Сомелье должен:

- подать заявку на экспертный профиль;
- заполнить публичную витрину: опыт, специализации, язык, город, форматы работы, ссылки;
- пройти ручную модерацию WinePool;
- публиковать экспертные заметки и подборки;
- принимать входящие запросы;
- задавать уточняющие вопросы;
- указывать формат и ориентир стоимости;
- отмечать запросы как принятые, выполненные, отклоненные;
- получать прямой контакт пользователя только в рамках принятого запроса;
- вести историю консультаций внутри кабинета.

### 3.3. Для WinePool

WinePool получает:

- новый trust layer против обезличенных AI-рекомендаций;
- рост регистраций и возвратов через экспертные запросы;
- больше качественных отзывов, дегустационных заметок и данных погребка;
- партнерский аргумент для виноделен, туроператоров и медиа;
- будущую B2B-монетизацию через экспертные материалы, отчеты и партнерские пакеты;
- возможность позже включить платные SaaS-инструменты для экспертов без комиссии с каждой консультации.

## 4. Non-goals MVP

На первом срезе не делаем:

- прием платежей внутри WinePool;
- escrow / безопасную сделку;
- автоматические выплаты сомелье;
- встроенную фискализацию;
- публичный маркетплейс скидок на алкоголь;
- кнопку `Купить вино`;
- встроенную продажу алкогольной продукции;
- рекламные кампании вин без legal/ORD review;
- гарантию качества консультации со стороны WinePool;
- live-video внутри приложения;
- сложный календарь занятости и слот-букинг;
- публичные групповые сборы денег.

## 5. Юридическая и налоговая рамка

Этот раздел не является юридическим заключением. Перед публичным запуском платных консультаций нужна вычитка юристом по IT, рекламе алкоголя, налогам и персональным данным.

### 5.1. Консультация, а не продажа алкоголя

В интерфейсе и документах использовать формулировки:

- `консультация`;
- `экспертный разбор`;
- `подбор под задачу`;
- `образовательный материал`;
- `дегустационная заметка`;
- `частное мнение сомелье`.

Избегать:

- `купите это вино`;
- `закажите алкоголь`;
- `доставка вина`;
- `скидка на бутылку`;
- `лучшее вино для здоровья`;
- `лечебный эффект`;
- прямых побуждений к покупке конкретной алкогольной продукции.

### 5.2. Возрастной доступ

Все поверхности слоя сомелье показываются только после age gate 18+.

Публичные web-страницы экспертов, если появятся, также должны иметь 18+ copy и юридический футер WinePool.

### 5.3. Деньги и налоги

MVP-правило:

```text
WinePool не принимает оплату за консультацию.
Пользователь и сомелье договариваются напрямую.
Сомелье самостоятельно отвечает за налоги, чеки и статус исполнителя.
```

Для российских индивидуальных экспертов базовый рекомендуемый путь - самозанятость / НПД, если конкретная деятельность и лимиты подходят человеку. По официальным материалам ФНС режим НПД предусматривает ставки 4% для доходов от физлиц и 6% для доходов от юрлиц/ИП, а чеки можно формировать через приложение или веб-кабинет `Мой налог`.

В MVP WinePool может показывать нейтральный текст:

> Оплата и чек обсуждаются напрямую с сомелье. WinePool не принимает платежи за консультации и не является стороной расчета.

Не показывать:

- номер карты сомелье публично;
- платежную форму WinePool;
- кнопку `Оплатить в WinePool`;
- подтверждение платежа как юридически значимый факт.

Допустимый MVP-компромисс:

- поле `payment_note` в приватной переписке, которое заполняет сомелье вручную;
- статус `terms_agreed`, а не `paid`;
- чек-лист для сомелье `Я понимаю, что сам отвечаю за налоги и чеки`.

### 5.4. Реклама и партнерские материалы

Если экспертный материал оплачивает винодельня, магазин, дистрибьютор или туроператор, такой материал может потребовать:

- маркировки как партнерский / рекламный;
- проверки по ФЗ о рекламе;
- учета интернет-рекламы через ОРД/ЕРИР, если формат будет квалифицирован как реклама;
- отдельного договора и хранения первичных документов.

MVP-правило:

- экспертные профили и частные консультации - не рекламная поверхность;
- платные размещения виноделен с участием сомелье - только после legal review;
- в B2B-пакетах использовать wording `редакционный материал`, `экспертный разбор`, `партнерский материал` только после согласования с юристом.

## 6. Роли и доступ

### 6.1. Guest

Гость после age gate может:

- видеть список опубликованных сомелье;
- открыть публичный профиль эксперта;
- читать опубликованные экспертные материалы;
- видеть CTA `Войдите, чтобы задать вопрос сомелье`.

Гость не может:

- отправлять приватный запрос;
- видеть контакты сомелье, скрытые от публичного профиля;
- участвовать в приватной переписке;
- оставлять оценку консультации.

### 6.2. Authenticated buyer

Пользователь может:

- отправить запрос сомелье;
- приложить вино, чек, погребок, фото или свободный текст;
- видеть свои запросы;
- переписываться по своим запросам;
- закрыть запрос;
- оценить консультацию;
- пожаловаться на консультацию.

### 6.3. Expert

Эксперт - это authenticated user с approved/published `expert_profiles`.

Может:

- редактировать черновик своего экспертного профиля;
- отправлять профиль на модерацию;
- управлять услугами и ориентировочными ценами;
- принимать и отклонять запросы;
- вести переписку по принятым запросам;
- публиковать ответы, если запрос стал публичным;
- видеть историю своих консультаций.

Не может:

- видеть чужие запросы;
- менять public verification status;
- модерировать отзывы о себе;
- публиковать рекламные материалы без модерации;
- выдавать себя за представителя WinePool без специального статуса.

### 6.4. Administrator / moderator

Admin может:

- видеть все экспертные профили;
- approve/reject/pause expert profile;
- проверять документы/ссылки/сертификаты;
- скрывать спорные материалы;
- рассматривать жалобы;
- блокировать прием новых запросов конкретным экспертом;
- видеть audit log.

Moderator может:

- проверять профиль и материалы;
- отправлять на доработку;
- скрывать контент;
- не должен менять финансовые/partner flags без admin capability.

## 7. Доменные сущности

### 7.1. `expert_profiles`

1:1 профессиональный профиль поверх `profiles`.

Поля:

```text
id uuid primary key
user_id uuid references profiles(id) unique not null
display_name text not null
slug text unique
headline text
bio text
city text
country_code text
languages text[] default '{ru}'
avatar_url text
cover_image_url text
experience_years int
specializations text[]              -- champagne, russian_wine, burgundy, horeca, pairing, cellar, tourism
credentials text                    -- свободное описание дипломов/школ/конкурсов
certificate_urls text[]
external_links jsonb                -- telegram, website, youtube, vk, etc.
contact_policy text                 -- in_app_first, external_after_acceptance
consultation_policy text            -- как работает эксперт, сроки, границы
legal_status text                   -- unknown, self_employed_declared, ip_declared, company_declared
tax_disclaimer_accepted_at timestamptz
verification_status text            -- draft, submitted, approved, rejected, paused
publication_status text             -- hidden, published
is_featured boolean default false
accepts_private_requests boolean default false
accepts_group_requests boolean default false
avg_response_hours int
rating_avg numeric
rating_count int default 0
completed_requests_count int default 0
created_at timestamptz
updated_at timestamptz
submitted_at timestamptz
reviewed_at timestamptz
reviewed_by uuid references profiles(id)
review_note text
```

Индексы:

```text
expert_profiles(user_id)
expert_profiles(slug)
expert_profiles(verification_status, publication_status, is_featured)
expert_profiles using gin(specializations)
```

### 7.2. `expert_services`

Услуги/форматы, которые эксперт готов принимать.

Поля:

```text
id uuid primary key
expert_profile_id uuid references expert_profiles(id) on delete cascade
service_type text not null          -- bottle_review, cellar_review, pairing, gift_selection, event_selection, tourism_advice, live_tasting, corporate
title text not null
description text
duration_minutes int
price_mode text                     -- free, fixed, from_price, by_agreement, external
price_amount numeric
price_currency text default 'RUB'
delivery_format text                -- text, voice, video_external, offline, group
is_active boolean default true
sort_order int default 100
created_at timestamptz
updated_at timestamptz
```

MVP допускает только справочную цену. Цена не является офертой WinePool.

### 7.3. `expert_requests`

Основная таблица запросов к сомелье.

Поля:

```text
id uuid primary key
requester_user_id uuid references profiles(id) not null
expert_profile_id uuid references expert_profiles(id)
service_id uuid references expert_services(id)
request_kind text not null          -- private, group_interest, editorial_question
topic text not null
question text not null
context_type text                   -- wine, cellar, receipt_item, tasting, review, tourism, free
context_wine_id uuid references wines(id)
context_user_storage_id uuid
context_tasting_id uuid
context_review_id uuid
context_receipt_item_id uuid
context_tourism_experience_id uuid
budget_min numeric
budget_max numeric
budget_currency text default 'RUB'
occasion text
deadline_at timestamptz
visibility text default 'private'   -- private, anonymized_public, public_after_moderation
status text not null default 'submitted'
terms_summary text
external_payment_note text          -- приватная заметка эксперта, не платежная инструкция платформы
accepted_at timestamptz
answered_at timestamptz
completed_at timestamptz
cancelled_at timestamptz
created_at timestamptz
updated_at timestamptz
```

Статусы:

```text
draft
submitted
needs_clarification
accepted
declined_by_expert
cancelled_by_user
terms_agreed
in_progress
answered
completed
closed
reported
hidden_by_admin
```

Ключевое MVP-правило: не использовать `paid` как системный статус. Деньги остаются вне WinePool.

### 7.4. `expert_request_messages`

Переписка по запросу.

Поля:

```text
id uuid primary key
request_id uuid references expert_requests(id) on delete cascade
sender_user_id uuid references profiles(id) not null
message_type text default 'text'    -- text, system, answer, clarification, external_contact_shared
body text
attachment_urls text[]
is_final_answer boolean default false
created_at timestamptz
edited_at timestamptz
deleted_at timestamptz
```

Для MVP достаточно текстовых сообщений и вложений-картинок через Storage после отдельной реализации bucket/policies.

### 7.5. `expert_request_events`

Audit/history.

Поля:

```text
id uuid primary key
request_id uuid references expert_requests(id) on delete cascade
actor_user_id uuid references profiles(id)
event_type text not null
old_status text
new_status text
payload jsonb default '{}'
created_at timestamptz
```

События:

```text
submitted
expert_assigned
expert_accepted
expert_declined
clarification_requested
terms_updated
work_started
final_answer_sent
completed
cancelled
reported
admin_hidden
```

### 7.6. `expert_reviews`

Оценки консультаций.

Поля:

```text
id uuid primary key
request_id uuid references expert_requests(id) unique not null
expert_profile_id uuid references expert_profiles(id) not null
user_id uuid references profiles(id) not null
rating int check (rating between 1 and 5)
comment text
is_public boolean default false
moderation_status text default 'pending' -- pending, visible, hidden
created_at timestamptz
updated_at timestamptz
```

Публично показывать только модерированные отзывы, без деталей запроса.

### 7.7. `expert_articles`

Легкий слой экспертного самовыражения.

Поля:

```text
id uuid primary key
expert_profile_id uuid references expert_profiles(id) not null
title text not null
slug text unique
summary text
body text
cover_image_url text
related_wine_id uuid references wines(id)
related_winery_id uuid references wineries(id)
related_tourism_experience_id uuid
publication_status text default 'draft' -- draft, submitted, published, hidden
partner_disclosure text
published_at timestamptz
created_at timestamptz
updated_at timestamptz
```

MVP-альтернатива: не делать `expert_articles`, а использовать существующую `partner_media_feed`. Но отдельная таблица понадобится, если хотим нативные материалы внутри WinePool, связанные с wine_id/winery_id.

### 7.8. `expert_group_questions`

Post-MVP слой для группового спроса без сбора денег.

Идея: пользователи голосуют за тему или присоединяются к вопросу. Эксперт видит спрос и может провести публичный разбор.

Поля:

```text
id uuid primary key
created_by_user_id uuid references profiles(id)
expert_profile_id uuid references expert_profiles(id)
topic text not null
description text
context_wine_id uuid references wines(id)
interest_count int default 0
status text default 'open'          -- open, selected_by_expert, answered, closed, hidden
created_at timestamptz
updated_at timestamptz
```

`expert_group_question_members`:

```text
question_id uuid
user_id uuid
joined_at timestamptz
note text
primary key(question_id, user_id)
```

Никаких pooled payments на MVP.

## 8. RLS и приватность

### 8.1. Public expert profile

Публично отдавать не саму таблицу целиком, а view/RPC:

`public_expert_profiles`

Поля:

- id;
- slug;
- display_name;
- headline;
- bio;
- avatar_url;
- cover_image_url;
- city;
- languages;
- specializations;
- experience_years;
- external public links;
- rating_avg;
- rating_count;
- completed_requests_count;
- accepts_private_requests;
- accepts_group_requests.

Не отдавать публично:

- `user_id`, если не нужен;
- contact private fields;
- legal/tax fields;
- review notes;
- admin status history;
- приватные ссылки и документы.

### 8.2. Expert own profile

Эксперт может:

- читать свой полный `expert_profiles`;
- обновлять свой черновик, если статус не `paused`/`hidden_by_admin`;
- создавать/редактировать свои `expert_services`;
- не может сам менять `verification_status = approved`.

### 8.3. Requests

`expert_requests` видят:

- requester;
- assigned expert;
- admin/moderator;
- group/public summary через отдельный view.

Messages видят только:

- requester;
- assigned expert;
- admin/moderator при жалобе или moderation view.

### 8.4. Data retention

Рекомендация:

- приватные сообщения хранить до удаления аккаунта или запроса удаления;
- публичные материалы после удаления аккаунта скрывать или переводить в anonymized state по политике;
- жалобы и audit log хранить отдельно в рамках legal retention после вычитки юристом.

## 9. Пользовательские сценарии

### 9.1. Пользователь задает вопрос из карточки вина

Вход:

- карточка вина `/wine/:id`;
- кнопка `Спросить сомелье`;
- список подходящих экспертов.

Flow:

1. Пользователь выбирает эксперта.
2. Выбирает формат: `Разбор бутылки`, `С чем сочетать`, `Стоит ли хранить`, `Что попробовать рядом`.
3. Заполняет вопрос и сроки.
4. Видит copy: `Оплата, если потребуется, обсуждается напрямую с сомелье`.
5. Отправляет запрос.
6. Сомелье получает notification.
7. Сомелье принимает, просит уточнение или отклоняет.
8. Пользователь получает ответ.
9. По завершении может сохранить заметку в погребок или оставить оценку консультации.

### 9.2. Запрос из чека или ручной бутылки

Вход:

- receipt scan result;
- pending draft wine;
- user_storage item.

Полезные CTA:

- `Разобрать эту бутылку с сомелье`;
- `Понять, когда открыть`;
- `Подобрать пару к ужину`;
- `Сравнить с похожими винами`.

Контекст автоматически прикладывает:

- wine_id, если есть match;
- raw receipt item только если это собственный чек пользователя;
- vintage;
- price;
- shop name только для личного контекста;
- пользовательскую заметку.

### 9.3. Разбор погребка

Вход:

- `Мой погребок`;
- `Аналитика`;
- профиль сомелье.

Flow:

1. Пользователь выбирает `Разбор погребка`.
2. WinePool показывает, какие данные будут переданы:
   - список выбранных бутылок;
   - оценки;
   - заметки, если пользователь явно разрешил;
   - бюджет/цель.
3. Пользователь может снять галочки с приватных заметок.
4. Сомелье получает структурированный snapshot.
5. Итог может сохраниться как `cellar_expert_note`.

MVP можно начать без snapshot-таблицы: передавать только выбранные `user_storage_id[]` в `expert_requests.payload`.

### 9.4. Сомелье принимает запрос

Экран `Кабинет сомелье`:

- вкладки `Новые`, `В работе`, `Завершенные`, `Профиль`, `Услуги`;
- карточка запроса: тема, контекст, дедлайн, формат, пользовательский комментарий;
- действия:
  - `Принять`;
  - `Уточнить`;
  - `Отклонить`;
  - `Предложить условия`;
  - `Отправить итоговый ответ`.

В `Предложить условия`:

- краткое описание результата;
- ориентир стоимости;
- срок;
- способ связи/оплаты вне WinePool как текст;
- чекбокс `Я самостоятельно отвечаю за расчеты и налоги`.

### 9.5. Групповой запрос

MVP-light:

- пользователи голосуют за тему;
- сомелье видит `N интересуются`;
- сомелье может опубликовать публичный ответ;
- WinePool уведомляет участников.

Без денег внутри приложения.

Post-MVP:

- открытые групповые дегустации;
- external registration link;
- integration with WinePool Tourism, если это офлайн-мероприятие;
- платный билет только после legal review.

### 9.6. Экспертное самовыражение

Поверхности:

- профиль сомелье;
- экспертные заметки;
- подборки;
- участие в партнерской ленте;
- `Совет сомелье` на карточке вина;
- `Сомелье месяца`;
- публичные ответы на групповые вопросы.

Важно: экспертный контент должен иметь moderation layer, если он выходит в публичную ленту, главную или карточки вин.

## 10. Flutter-архитектура

Новый модуль:

```text
lib/features/experts/
  domain/
    expert_profile.dart
    expert_service.dart
    expert_request.dart
    expert_message.dart
    expert_review.dart
  data/
    experts_repository.dart
    expert_requests_repository.dart
  application/
    experts_providers.dart
    expert_requests_controller.dart
    expert_profile_controller.dart
  presentation/
    experts_list_screen.dart
    expert_profile_screen.dart
    expert_request_sheet.dart
    my_expert_requests_screen.dart
    expert_console_screen.dart
    expert_profile_editor_screen.dart
    expert_request_details_screen.dart
```

### 10.1. Routes

```text
/experts
/experts/:expertIdOrSlug
/experts/:expertIdOrSlug/request
/my-expert-requests
/expert-console
/expert-console/profile
/expert-console/requests/:requestId
/admin/experts
```

### 10.2. Entry points

Buyer:

- главный экран: блок `Живые сомелье`;
- профиль: `Мои консультации`;
- карточка вина: `Спросить сомелье`;
- погребок: `Разобрать погребок`;
- receipt result: `Разобрать покупку`;
- tourism: `Спросить о маршруте`.

Expert:

- профиль: `Стать сомелье`;
- после approval: `Кабинет сомелье`;
- notifications: новые запросы, сообщения, статусы.

Admin:

- admin dashboard: `Сомелье`;
- очередь заявок;
- опубликованные эксперты;
- жалобы.

### 10.3. Локализация

Все строки сразу в `.arb`:

- RU обязательный;
- EN заготовить, если страница эксперта потом будет публичной.

## 11. Backend / SQL порядок реализации

### Phase 1. Public expert profiles

Миграция:

- enums/check constraints;
- `expert_profiles`;
- `expert_services`;
- public view/RPC;
- RLS;
- storage bucket для avatar/cover/certificates, если не используем существующий механизм.

Flutter:

- список экспертов;
- профиль эксперта;
- заявка на статус сомелье;
- admin approve/reject.

DoD:

- пользователь может подать заявку;
- admin может одобрить;
- опубликованный эксперт виден в `/experts`;
- приватные поля не видны публично.

### Phase 2. Private requests without payments

Миграция:

- `expert_requests`;
- `expert_request_messages`;
- `expert_request_events`;
- notifications.

Flutter:

- request sheet;
- buyer request list;
- expert console request list;
- request details/messages;
- status transitions.

DoD:

- buyer отправляет запрос;
- expert принимает/отклоняет/уточняет;
- buyer и expert видят переписку;
- admin не нужен для обычного flow;
- платежей внутри WinePool нет.

### Phase 3. Ratings, complaints and moderation

Миграция:

- `expert_reviews`;
- `expert_complaints` или reuse moderation/report table, если появится общий механизм.

Flutter:

- оценка консультации;
- жалоба;
- admin moderation view.

DoD:

- рейтинг считается только по completed requests;
- публичные отзывы проходят moderation;
- жалоба открывает admin visibility к конкретному request.

### Phase 4. Expert content and wine-card overlays

Миграция:

- `expert_articles` или расширение partner media linking.

Flutter:

- экспертные заметки;
- блок `Совет сомелье` на wine detail;
- блок на главной;
- deep-link из wine card/article.

DoD:

- эксперт может отправить материал на модерацию;
- admin публикует;
- материал связан с wine/winery/tourism context.

### Phase 5. Group questions

Миграция:

- `expert_group_questions`;
- `expert_group_question_members`.

Flutter:

- список тем;
- `Мне тоже интересно`;
- public answer.

DoD:

- пользователь присоединяется к теме;
- эксперт публикует ответ;
- участники получают notification;
- денег внутри WinePool нет.

### Phase 6. Monetization after legal review

Варианты:

1. Платный SaaS-пакет для сомелье:
   - расширенный профиль;
   - больше входящих заявок;
   - аналитика;
   - управление услугами;
   - без комиссии с консультации.

2. B2B-пакеты для виноделен:
   - экспертный разбор линейки;
   - редакционный материал;
   - tourism route with sommelier;
   - аналитика интереса пользователей.

3. Premium для пользователей:
   - экспертные подборки;
   - расширенные публичные разборы;
   - закрытые эфиры через external platform;
   - priority request, если юридически корректно.

Любые платные размещения и рекламные форматы - только после legal/ORD decision.

## 12. Notifications

Типы:

```text
expert_profile_submitted_admin
expert_profile_approved
expert_profile_rejected
expert_request_created_expert
expert_request_message_user
expert_request_message_expert
expert_request_status_changed
expert_request_final_answer
expert_request_completed_review_prompt
expert_group_question_answered
expert_complaint_admin
```

Deep-links:

```text
/expert-console/requests/:requestId
/my-expert-requests?requestId=...
/experts/:slug
/admin/experts?profileId=...
```

## 13. Admin moderation

Admin screen `/admin/experts`:

Вкладки:

- `Заявки`;
- `Опубликованные`;
- `Приостановленные`;
- `Жалобы`;
- `Материалы`.

Профильная заявка показывает:

- имя;
- bio/headline;
- опыт;
- специализации;
- сертификаты/ссылки;
- external links;
- legal disclaimer accepted;
- публичный preview;
- action: approve/reject/request changes/pause.

Модерационные причины:

- insufficient credentials;
- suspicious identity;
- unsafe alcohol claims;
- direct sales language;
- misleading commercial promise;
- offensive content;
- legal/tax missing acknowledgement;
- duplicate expert profile.

## 14. Safety copy

Базовый copy на request sheet:

> Сомелье отвечает как независимый эксперт. WinePool помогает передать запрос и сохранить историю, но не принимает оплату и не продает алкоголь. Если консультация платная, условия и чек обсуждаются напрямую с сомелье.

Copy для эксперта при принятии запроса:

> Я понимаю, что самостоятельно отвечаю за расчеты с клиентом, налоги, чеки и корректность своих рекомендаций. WinePool не является стороной оплаты и не принимает деньги за эту консультацию.

Copy на публичном профиле:

> Материалы эксперта носят информационный и образовательный характер. Чрезмерное употребление алкоголя вредит здоровью. 18+.

## 15. Аналитика

Events:

```text
expert_list_opened
expert_profile_opened
expert_request_started
expert_request_submitted
expert_request_accepted
expert_request_declined
expert_request_answered
expert_request_completed
expert_review_submitted
expert_article_opened
expert_group_interest_joined
```

Метрики:

- expert profile views;
- request conversion by entry point;
- request acceptance rate;
- median first response time;
- completion rate;
- rating average;
- repeat requester rate;
- wine/card contexts that generate requests;
- B2B leads influenced by expert content.

Важно: B2B-аналитика должна быть агрегированной и обезличенной.

## 16. Acceptance criteria

### Public profile

- Пользователь после age gate видит список опубликованных сомелье.
- Публичный профиль не раскрывает приватные legal/admin fields.
- Эксперт не появляется публично без approval.
- Admin может снять профиль с публикации.

### Expert request

- Auth user может отправить private request.
- Guest видит registration wall.
- Expert видит только свои запросы.
- User видит только свои запросы.
- Messages недоступны посторонним пользователям.
- Нет системного статуса `paid` и нет платежной формы WinePool.
- Финальный ответ можно пометить и сохранить в истории.

### Legal/product safety

- Нет CTA `Купить алкоголь`, `Заказать вино`, `Доставка вина`.
- Все экспертные поверхности имеют 18+ context.
- Платные/партнерские материалы не публикуются без moderation flag.
- При отправке условий эксперт подтверждает tax/payment responsibility.

### Admin

- Admin видит очередь экспертных заявок.
- Admin может approve/reject/pause.
- Жалоба открывает admin доступ к request details.
- Все status changes пишутся в events.

## 17. Test plan

Unit tests:

- expert status transitions;
- request status transitions;
- RLS helper policies, если есть SQL tests;
- public profile serialization;
- safety copy validator for forbidden CTA words.

Repository/RPC tests:

- user creates expert profile draft;
- user cannot approve own expert profile;
- admin approves profile;
- public sees only published profile;
- requester creates request;
- random user cannot read request;
- assigned expert can read request;
- assigned expert can answer;
- non-assigned expert cannot answer;
- requester can review completed request once.

Widget tests:

- experts list;
- expert profile;
- request sheet from wine detail;
- my expert requests;
- expert console list/detail;
- admin expert review card.

Manual QA:

- fresh install guest -> experts -> auth wall;
- buyer sends question from wine card;
- buyer sends question from cellar item;
- expert accepts and answers;
- expert declines;
- admin pauses expert profile;
- complaint flow;
- weak Android long message;
- offline/network error during message send;
- localization RU/EN placeholders.

## 18. Recommended first slice

Не начинать с платежей. Первый полезный и безопасный slice:

1. `expert_profiles` + `expert_services`.
2. Admin approval.
3. Public `/experts`.
4. Expert profile page.
5. `Спросить сомелье` из карточки вина.
6. Private request + messages.
7. Expert console.
8. Notifications.
9. Rating after completion.

Этот срез уже даст рынку понятный сигнал:

> WinePool - место, где AI помогает, но человеческая винная экспертиза остается в центре.

После 20-50 реальных запросов можно решать, нужны ли:

- групповые вопросы;
- external event integrations;
- платный SaaS-профиль сомелье;
- B2B-пакеты с винодельнями;
- интеграция с НПД/оператором электронной площадки;
- встроенные платежи.

## 19. Источники для legal/tax checkpoint

- ФНС, Налог на профессиональный доход: https://npd.nalog.ru/
- ФНС, ставки НПД 4%/6%: https://www.nalog.gov.ru/rn16/npd/
- ФНС, приложение и веб-кабинет `Мой налог`: https://npd.nalog.ru/app/ и https://npd.nalog.ru/web-app/
- ФНС, проверка статуса плательщика НПД: https://npd.nalog.ru/check-status/
- ФНС, операторы электронных площадок НПД: https://npd.nalog.ru/aggregators/
- Официальный портал правовой информации, 422-ФЗ о НПД: https://publication.pravo.gov.ru/document/view/0001201811270056
- Официальный портал правовой информации, 38-ФЗ `О рекламе`: https://pravo.gov.ru/proxy/ips/?docbody=&nd=102105292
- Официальный портал правовой информации, 171-ФЗ об обороте алкогольной продукции: https://pravo.gov.ru/proxy/ips/?docbody=&nd=102038309
- Официальный портал правовой информации, правила дистанционной продажи товаров: https://pravo.gov.ru/proxy/ips/?docbody=&nd=102116923
- Роскомнадзор, ЕРИР и учет интернет-рекламы: https://rkn.gov.ru/activity/register-ord/
