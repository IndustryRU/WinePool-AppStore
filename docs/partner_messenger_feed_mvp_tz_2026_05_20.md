# Partner Media Feed MVP

Дата: 2026-05-20
Последнее обновление: 2026-05-21

## 0. Статус Реализации На 21.05.2026

MVP винной ленты реализован и считается функционально закрытым для релизного кандидата.

Закрытые коммиты:

- `46afb52 Add partner media feed backend sync` — backend foundation, Worker, sync job.
- `6871519 Add partner media feed UI` — экран ленты, профиль, локализация, image cache bucket.
- `a13e0df Add home partner media preview` — блок на главной, deep-link к публикации, UI-polish.

Фактическое состояние:

- 3 стартовых источника заведены в `partner_media_sources`.
- Sync на VPS активен через `winepool-partner-media-sync.timer`, периодичность около 1 часа.
- Хранится максимум 10 видимых публикаций на канал.
- Картинки скачиваются server-side и кэшируются в `storage.partner_media_images`; приложение получает URL с `https://api.winepool.ru/storage/...`, а не с внешнего CDN/Worker.
- В профиле есть вход в `Винную ленту` и карточки каналов.
- На главной есть горизонтальный блок `Винная лента` до 10 последних публикаций.
- Тап по карточке на главной открывает `/partner-feed?postId=...` и прокручивает ленту к конкретной публикации.
- Экран `/partner-feed` показывает публикации всех активных источников, полный текст внутри приложения, изображения, дату, источник.
- `Читать` раскрывает полный текст с плавной анимацией и fade в свернутом состоянии; `Свернуть` закрывает.
- `Канал` ведет именно на внешний канал (`external_url`), не на отдельный пост.
- Для последней карточки добавлен нижний отступ, чтобы bottom navigation не перекрывал кнопки.

Осталось перед релизом: только smoke QA на release APK без VPN и проверка сценариев guest/authenticated.

## 1. Цель

Запустить в WinePool партнерский контент из внешних медиа-каналов:

- дать партнерам видимость внутри приложения;
- добавить живой винный контент, который возвращает пользователей в приложение;
- сохранить переходы во внешний канал, чтобы партнеры получали подписчиков и трафик;
- не зависеть от доступности конкретного мессенджера на клиентских устройствах и текущем VPS;
- заложить модель, которую позже можно расширить на другие мессенджеры и площадки.

## 2. Проверенный технический вывод

Текущий VPS `91.227.18.214` без обходного маршрута не подключается к первому целевому источнику напрямую, хотя обычный HTTPS с VPS работает.

Cloudflare Worker успешно получает данные из первого целевого источника. Вывод: для MVP используем схему:

```text
WinePool backend -> Cloudflare Worker -> external media source
```

Приложение не обращается к внешнему источнику напрямую. Flutter читает только подготовленные данные из Supabase.

## 3. MVP Scope

В MVP делаем:

- список партнерских медиа-источников в базе;
- загрузку последних публикаций через Cloudflare Worker;
- хранение ограниченного числа публикаций в Supabase;
- server-side кэширование изображений в собственный Supabase Storage;
- блок партнерских каналов в профиле;
- компактный блок последних публикаций на главной;
- отдельный экран партнерской ленты;
- чтение полного текста публикации внутри приложения;
- переход из карточки канала во внешний источник.

В MVP не делаем:

- полноценную админку управления партнерами;
- видео и тяжелые медиа;
- скачивание и хранение всех типов медиа у себя, кроме MVP-кэша изображений;
- комментарии, реакции и просмотры внешней площадки;
- персональные рекомендации;
- оплату/биллинг партнерских размещений;
- глубокую модерацию каждой публикации перед показом.

## 4. Пользовательские сценарии

### 4.1. Пользователь видит партнерские каналы

В разделе `Профиль` пользователь видит блок `Винные медиа` или `Партнерские каналы`.

Карточка канала показывает:

- название;
- внешний handle или короткий адрес;
- краткое описание;
- логотип или инициалы;
- кнопку `Открыть канал`.

### 4.2. Пользователь читает партнерскую ленту

Из профиля или отдельной точки входа пользователь открывает экран `Винная лента`.

Экран показывает последние публикации партнерских каналов:

- источник;
- дату публикации;
- свернутый полный текст на 5 строк;
- изображение из собственного Storage, если оно доступно;
- кнопку `Читать` / `Свернуть` для раскрытия полного текста;
- кнопку `Канал` для перехода во внешний канал.

### 4.3. Пользователь видит живой контент на главной

На главной странице после верхнего action-блока показывается компактный горизонтальный блок `Винная лента`:

- до 10 последних публикаций;
- миниатюра изображения, если есть;
- название источника;
- 4 строки текста;
- тап по карточке открывает полную ленту и прокручивает к конкретной публикации;
- кнопка `Все` открывает ленту с начала.

### 4.4. Партнер получает пользу

Каждая публикация и карточка канала явно атрибутированы:

- название канала;
- ссылка на канал;
- CTA на подписку/переход во внешний источник.

Ссылка на оригинальную публикацию сохраняется в `external_post_url`, но отдельная кнопка на нее в MVP не выводится, чтобы не перегружать карточку. При необходимости ее можно добавить post-MVP как вторичное действие.

## 5. Архитектура

```text
External media page
  -> Cloudflare Worker fetch/parser
  -> WinePool sync job
  -> Supabase Storage image cache
  -> Supabase tables
  -> Flutter app
```

Конкретная площадка должна быть внутренней деталью адаптера. В продуктовых текстах используем нейтральные слова: `медиа`, `канал`, `партнерская лента`, `внешний источник`.

## 6. Данные

### 6.1. Таблица `partner_media_sources`

Назначение: источник правды по партнерским каналам.

Поля:

- `id uuid primary key`;
- `source_type text not null`;
- `external_handle text not null`;
- `external_url text not null`;
- `title text not null`;
- `summary text`;
- `description text`;
- `avatar_url text`;
- `logo_asset text`;
- `is_active boolean not null default true`;
- `is_featured boolean not null default false`;
- `display_order int not null default 100`;
- `posts_limit int not null default 10`;
- `last_sync_at timestamptz`;
- `last_sync_status text`;
- `last_sync_error text`;
- `created_at timestamptz not null default now()`;
- `updated_at timestamptz not null default now()`;
- `unique(source_type, external_handle)`.

`source_type` нужен для будущего расширения на несколько площадок. В UI его не показываем.

### 6.2. Таблица `partner_media_posts`

Назначение: последние публикации, готовые для показа в приложении.

Поля:

- `id uuid primary key`;
- `source_id uuid not null references partner_media_sources(id) on delete cascade`;
- `external_post_id text not null`;
- `external_post_url text not null`;
- `published_at timestamptz`;
- `text text`;
- `text_preview text`;
- `image_url text`;
- `media_type text`;
- `is_visible boolean not null default true`;
- `raw_payload jsonb`;
- `created_at timestamptz not null default now()`;
- `updated_at timestamptz not null default now()`;
- `unique(source_id, external_post_id)`.

Индексы:

- `partner_media_posts(source_id, published_at desc)`;
- `partner_media_posts(is_visible, published_at desc)`;

### 6.3. Ограничение роста базы

Для MVP:

- хранить максимум `posts_limit`, по умолчанию `10`, видимых публикаций на канал;
- после каждой синхронизации удалять лишние публикации сверх лимита;
- дедупликация по `source_id + external_post_id`.

Важно: текущая MVP-реализация уже ограничивает рост таблицы публикаций, но после добавления Storage-кэша изображений нужен отдельный retention-предохранитель для медиа-файлов.

Рекомендуемая post-MVP модель:

- разделить `visible_limit` и `retention_limit`;
- `visible_limit`: сколько публикаций показываем в UI, например 10;
- `retention_limit`: сколько последних публикаций физически храним на источник, например 30;
- публикации между 11 и 30 можно хранить как архив/буфер для будущего экрана канала или аналитики;
- публикации старше 30 удалять из `partner_media_posts`;
- при физическом удалении поста удалять связанные файлы из `partner_media_images`, если они больше нигде не используются;
- значение `retention_limit` лучше сделать настройкой источника или глобальной env-настройкой sync job.

Варианты на будущее:

- не удалять физически, а переводить в `is_visible = false`;
- хранить архив 30-60 дней для аналитики;
- добавить ручную модерацию/скрытие.

## 7. Cloudflare Worker

Worker отвечает за доступ к внешнему медиа-источнику и первичный парсинг публичной web-страницы канала.

Endpoint MVP:

```text
GET /source/:sourceType/:handle
```

или:

```text
GET /?source=messenger&handle=example_channel
```

Ответ:

```json
{
  "sourceType": "messenger",
  "handle": "example_channel",
  "fetchedAt": "2026-05-20T13:05:14.098Z",
  "posts": [
    {
      "externalPostId": "1234",
      "url": "https://example.com/channel/1234",
      "publishedAt": "2026-05-20T10:00:00.000Z",
      "text": "Post text",
      "textPreview": "Post text",
      "imageUrl": "https://example.com/image.jpg",
      "mediaType": "photo"
    }
  ]
}
```

Worker должен:

- валидировать `source` и `handle`;
- ходить только в разрешенные внешние источники;
- возвращать не больше 20 последних найденных публикаций;
- ставить `Cache-Control` на 5-15 минут;
- возвращать структурированную ошибку при недоступности источника;
- не требовать секретов для MVP, если URL не публичен в приложении.

Для production лучше добавить простой shared secret между sync job и Worker.

### 7.1. Media Proxy

Worker также содержит endpoint `/media?url=...` для server-side скачивания изображений из разрешенных CDN.

Назначение:

- sync job получает реальное изображение через Worker;
- проверяет `Content-Type`;
- загружает файл в `partner_media_images`;
- записывает в `partner_media_posts.image_url` публичный URL `https://api.winepool.ru/storage/v1/object/public/partner_media_images/...`.

Клиентское приложение не должно ходить к `workers.dev` или внешнему CDN за картинками. Это важно для работы без VPN на мобильном интернете в РФ.

## 8. Sync Job

Задача: регулярно подтягивать публикации активных каналов.

Варианты запуска:

- cron на текущем VPS;
- Supabase Edge Function schedule, если доступно;
- отдельный Node/Deno script, вызываемый по расписанию.

Для MVP предпочтительно:

```text
VPS cron -> script -> Cloudflare Worker -> Supabase
```

Периодичность:

- каждые 30-60 минут для MVP;
- чаще не нужно, чтобы не упираться в лимиты и не выглядеть как агрессивный scraping.

Sync алгоритм:

1. Получить `partner_media_sources where is_active = true`.
2. Для каждого источника вызвать Cloudflare Worker.
3. Upsert публикации по `source_id + external_post_id`.
4. Скачать изображения через Worker media proxy и сохранить в `partner_media_images`.
5. Записать в `image_url` публичный storage URL на `api.winepool.ru`.
6. Обновить `last_sync_at`, `last_sync_status`, `last_sync_error`.
7. Удалить публикации сверх `posts_limit` для каждого источника.

Поведение при ошибке источника:

- старые посты не удаляются;
- источник получает `last_sync_status = error`;
- приложение продолжает показывать сохраненные публикации из базы.

Retention-предохранитель для следующей итерации:

1. Получить все посты источника, отсортированные по `published_at desc nullslast, created_at desc`.
2. Оставить первые `retention_limit`, рекомендуемо 30.
3. Для хвоста старше лимита собрать `image_url` из bucket `partner_media_images`.
4. Удалить строки хвоста из `partner_media_posts`.
5. Удалить связанные Storage objects, если URL указывает на `partner_media_images`.
6. Логи sync должны явно показывать, сколько удалено строк и media objects.

## 9. Flutter UI

### 9.1. Профиль

Текущий hardcoded-блок `_wineMediaChannels` заменить на данные из Supabase.

MVP поведение:

- если Supabase вернул каналы, показывать их;
- если загрузка не удалась, можно временно показать текущий hardcoded fallback;
- карточки визуально оставить близкими к текущим, чтобы не раздувать фронтенд-объем.

### 9.2. Экран `Винная лента`

Новый экран:

```text
/partner-feed
```

Содержимое:

- AppBar `Винная лента`;
- список публикаций всех активных партнеров;
- сортировка по `published_at desc`;
- карточка публикации с источником, датой, preview и CTA;
- empty state, если публикаций нет;
- error state с возможностью обновить.

Точка входа:

- кнопка/плитка в профиле: `Винная лента`;
- блок `Винная лента` на главной с deep-link к конкретному посту.

### 9.3. Карточка публикации

Показываем:

- avatar/initials канала;
- title канала;
- дату;
- текст: 5 строк по умолчанию, полный текст при раскрытии;
- изображение, если есть;
- `Читать` / `Свернуть`;
- `Канал`.

Ограничения:

- длинный текст сворачивать до разумного preview;
- раскрытие и закрытие делать плавно, без резкого скачка карточки;
- не показывать пустые публикации без текста и без изображения;
- открывать внешние ссылки через `url_launcher`.

### 9.4. Главная

На главной показывается горизонтальный блок:

- заголовок `Винная лента`;
- до 10 последних публикаций;
- карточка шириной около 280 px;
- изображение 88x114 или fallback-иконка;
- источник и текст;
- тап по карточке ведет на `/partner-feed?postId=<post.id>`;
- лента выполняет двухфазный автоскролл: примерный scroll к индексу, затем `Scrollable.ensureVisible`, когда карточка построилась ленивым `ListView`.

Если feed не загрузился, блок на главной скрывается и не ломает основной сценарий.

## 10. Безопасность и ограничения

- Не выполнять произвольные URL через Worker.
- Не принимать полный URL от клиента, только `source` и `handle`.
- Нормализовать handle: буквы, цифры, `_`, длина до 64.
- Не хранить секреты в Flutter.
- На публичные таблицы дать только `select`.
- Запись в таблицы выполнять только service-role скриптом или защищенной серверной функцией.
- Учитывать, что HTML внешней площадки не является стабильным API и может измениться.

## 11. Acceptance Criteria

MVP считается готовым, если:

- в Supabase есть 3 стартовых партнерских источника:
  - `takoe_vino`;
  - `RamonSosnovskiy`;
- `tvoi_somele`;
- sync успешно подтягивает публикации через Cloudflare Worker;
- в базе хранится не больше 10 публикаций на канал;
- изображения публикаций кэшируются в `partner_media_images` и отдаются с `api.winepool.ru`;
- профиль показывает партнерские каналы из Supabase;
- экран `Винная лента` показывает последние публикации;
- главная показывает до 10 последних публикаций;
- тап по карточке на главной ведет к конкретному посту в полной ленте;
- `Читать` раскрывает полный текст внутри приложения;
- `Канал` открывает внешний канал;
- приложение не падает при недоступности Worker/источника;
- `flutter analyze` проходит без новых ошибок.

## 12. План запуска MVP

### Этап 1. Backend foundation

- Создать Supabase migration для `partner_media_sources` и `partner_media_posts`.
- Добавить seed для трех текущих каналов.
- Настроить RLS на публичное чтение активных источников и видимых публикаций.

### Этап 2. Cloudflare Worker

- Перенести тестовый Worker в нормальную директорию проекта под нейтральным именем.
- Реализовать parser первого мессенджер-адаптера.
- Добавить валидацию `source` и `handle`.
- Задеплоить Worker под нейтральным именем.

### Этап 2.1. Cloudflare Worker deploy через API token

Если `npx wrangler deploy` падает с `Authentication error [code: 10000]`,
нужно выпустить новый Cloudflare API token и передать его Wrangler локально.

Token создавать в Cloudflare Dashboard:

- `User API Tokens` -> `Create Token`;
- template `Edit Cloudflare Workers` или `Custom token`;
- минимальные права:
  - `Account` -> `Workers Scripts` -> `Edit`;
  - `Account` -> `Account Settings` -> `Read`;
- `Account Resources`: `Include` -> конкретный аккаунт WinePool;
- `Client IP Address Filtering`: оставить пустым;
- TTL можно поставить короткий, например 1 день или 1 неделю.

Token не вставлять в чат и не коммитить. Для локального Codex/Wrangler
deploy можно создать временный файл в корне проекта:

```text
.env.cloudflare.local
```

Содержимое:

```env
CLOUDFLARE_API_TOKEN=<token>
CLOUDFLARE_ACCOUNT_ID=3cc65dbc10fa170bfcd9c56c5056528d
```

Файл должен быть в `.gitignore`.

Команда deploy:

```powershell
$envFile = 'R:\Flutter\Project\winepool_final\.env.cloudflare.local'
$vars = @{}
Get-Content $envFile | ForEach-Object {
  if ($_ -match '^\s*([^#=]+)=(.*)$') {
    $vars[$matches[1].Trim()] = $matches[2].Trim()
  }
}
$env:CLOUDFLARE_API_TOKEN = $vars['CLOUDFLARE_API_TOKEN']
$env:CLOUDFLARE_ACCOUNT_ID = $vars['CLOUDFLARE_ACCOUNT_ID']
npx wrangler deploy
```

Рабочая директория:

```text
workers/partner-media-fetcher
```

После deploy проверить Worker:

```powershell
curl.exe -s "https://winepool-partner-media-fetcher.winepool-probe.workers.dev/?source=messenger&handle=winepool&limit=5" -o "$env:TEMP\winepool_worker_payload.json"
```

Затем перезапустить live sync на VPS:

```powershell
ssh -i "$env:USERPROFILE\.ssh\winepool_vps" root@91.227.18.214 "systemctl start winepool-partner-media-sync.service && systemctl status winepool-partner-media-sync.service --no-pager -n 50"
```

Важно: `winepool-partner-media-sync.service` — `oneshot`; после успешного
запуска `systemctl status` может вернуть non-zero из-за состояния
`inactive (dead)`, но в логе должно быть `code=exited, status=0/SUCCESS`.

### Этап 3. Sync

- Добавить sync script.
- Подключить Supabase service role через переменные окружения на VPS.
- Настроить cron каждые 30-60 минут.
- Проверить, что лимит 10 публикаций на канал соблюдается.
- Создать bucket `partner_media_images` и кэшировать изображения в Storage.

### Этап 4. Flutter app

- Добавить модели `PartnerMediaSource`, `PartnerMediaPost`.
- Добавить providers для каналов и ленты.
- Заменить hardcoded media channels в профиле на Supabase-backed данные.
- Добавить экран `Винная лента` и route `/partner-feed`.
- Добавить блок последних публикаций на главную.
- Добавить deep-link `/partner-feed?postId=...`.
- Добавить раскрытие полного текста внутри приложения.
- Добавить переход `Канал` во внешний источник.

### Этап 5. QA

- Проверить fresh install / guest user / authorized user.
- Проверить отсутствие публикаций.
- Проверить недоступность Worker.
- Проверить длинные тексты и изображения.
- Проверить блок на главной и переход к конкретному посту.
- Проверить последний пост в ленте: bottom navigation не перекрывает кнопки.
- Проверить Android release без VPN на мобильном интернете.
- Прогнать `dart format`, `flutter analyze`, web/android smoke test.

## 13. Будущее после MVP

- Админка управления каналами.
- Ручное скрытие публикаций.
- Закрепленные партнерские публикации.
- UTM/analytics tracking переходов.
- Экран отдельного канала внутри приложения.
- Фильтрация публикаций по ключевым словам.
- AI-summary или ручные редакционные подборки.
- Поддержка дополнительных мессенджеров и медиа-площадок.
- Видео: `video_url`, `thumbnail_url`, storage limits, player, poster image, no autoplay.
- Ссылка на оригинальный пост как вторичное действие, если партнерам это будет нужно.
