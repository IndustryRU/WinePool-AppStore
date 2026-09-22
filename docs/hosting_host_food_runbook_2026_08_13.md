# WinePool: актуальная инфраструктура Host-Food

Дата проверки: 13.08.2026  
Статус: **источник истины для сайта, FTP и VPS**

## 1. Где сейчас находится WinePool

Рабочая инфраструктура WinePool размещена у провайдера **Host-Food** (`host-food.ru`):

- публичный сайт `winepool.ru` обслуживается с веб-хостинга Host-Food;
- файлы сайта загружаются по FTP в `/www/winepool.ru`;
- VPS приложения и self-hosted Supabase также находятся у Host-Food, но это отдельный контур с отдельными реквизитами и процедурами;
- DNS домена делегирован на `dns0.host-food.ru` и `dns1.host-food.ru`;
- SOA: `srv11.host-food.ru`;
- на дату проверки `winepool.ru` резолвится в `91.227.16.28`;
- FTP-профиль использует `srv11.host-food.ru:21` и корень `/www/winepool.ru`;
- `srv11.host-food.ru` резолвится в `91.227.16.8`; прежнее значение `91.227.16.11` не является рабочим FTP-адресом этого аккаунта.

**Timeweb не является текущим хостингом сайта или VPS.** Упоминания Timeweb в старых планах относятся к историческому этапу выбора/регистрации домена и не должны использоваться для деплоя. Названия `HostHub` / `Host-Foot` в переписке следует трактовать как оговорки: корректное имя провайдера — **Host-Food**.

## 2. Какой контур использовать

### Публичный сайт и юридические страницы

- локальный источник файлов: `landing/`;
- профиль подключения: `.deploy/winepool_ftp.env` (локальный секрет, не коммитить);
- шаблон профиля: `.deploy/winepool_ftp.env.example`;
- скрипт: `scripts/deploy_crisis_landing_ftp.ps1`;
- удалённый корень: `/www/winepool.ru`;
- основные страницы: `/`, `/privacy`, `/terms`, `/support-winepool`.

### Приложение, API, база данных и admin web

Это VPS-контур Host-Food. Для него нельзя использовать FTP-инструкцию сайта. Применять соответствующие VPS/SSH runbook и `scripts/deploy_admin_web.ps1` только для admin web.

## 3. Безопасная проверка перед изменениями

1. Проверить рабочее дерево: `git status --short`.
2. Не считать локальный файл актуальным только по дате — сначала сравнить его с опубликованной страницей.
3. Для публичной проверки открыть HTTPS-страницу и сверить дату/контрольную сумму.
4. Для проверки инфраструктуры:

```powershell
Resolve-DnsName winepool.ru -Type A
Resolve-DnsName winepool.ru -Type NS
Resolve-DnsName winepool.ru -Type SOA
Test-NetConnection srv11.host-food.ru -Port 21
```

5. Не выводить `FTP_USER` и `FTP_PASS` в терминал, документацию или чат.

До обновления политики 13.08.2026 опубликованный `privacy.html` отвечал по HTTPS, содержал дату **15 июля 2026 г.** и совпадал по SHA-256 с прежней локальной копией. После изменения юридических страниц локальную и опубликованную версии необходимо сверять заново; эта историческая проверка не подтверждает актуальность последующих редакций.

## 4. Деплой сайта

### Настройка подключения (один раз на рабочем компьютере)

Скопировать `.deploy/winepool_ftp.env.example` в `.deploy/winepool_ftp.env` и заполнить реквизиты Host-Food. Файл с секретами исключён из git. Ожидаемая несекретная часть:

```text
FTP_HOST=srv11.host-food.ru
FTP_PORT=21
FTP_REMOTE_ROOT=/www/winepool.ru
FTP_SSL=false
FTP_PASSIVE=true
DEPLOY_DOMAIN=https://winepool.ru
```

`FTP_USER` и `FTP_PASS` брать из панели Host-Food или существующего локального профиля; не публиковать.

### Точечная загрузка

Сначала точечный dry run:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File scripts\deploy_crisis_landing_ftp.ps1 -Only privacy.html -DryRun
```

После проверки изменений и явного запроса владельца — точечная загрузка:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File scripts\deploy_crisis_landing_ftp.ps1 -Only privacy.html
```

Для нескольких юридических страниц передать массив через PowerShell-команду. Полный деплой не запускать, если в `landing/` есть несвязанные или непроверенные изменения.

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command "& 'scripts\deploy_crisis_landing_ftp.ps1' -Only @('privacy.html','terms.html','support-winepool.html')"
```

Если публичную страницу нужно удалить, передать её относительный путь явно. Скрипт сначала попытается сохранить серверную копию в backup, затем удалит только указанный файл:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command "& 'scripts\deploy_crisis_landing_ftp.ps1' -Only @('index.html','.htaccess','sitemap.xml') -Delete @('old-page.html')"
```

### Полная загрузка

```powershell
# Сначала только проверка пакета
powershell -NoProfile -ExecutionPolicy Bypass -File scripts\deploy_crisis_landing_ftp.ps1 -DryRun

# Затем, только по явному запросу владельца
powershell -NoProfile -ExecutionPolicy Bypass -File scripts\deploy_crisis_landing_ftp.ps1
```

Скрипт собирает разрешённый набор из `landing/`, создаёт каталоги, пытается сохранить заменяемые файлы в backup и загружает новую версию. Для ручной передачи можно собрать ZIP командой `scripts/package_crisis_landing.ps1`, но рабочий серверный корень всё равно `/www/winepool.ru`, а не `public_html`.

Скрипт перед перезаписью пытается сохранить серверную копию в `build/ftp-backups/`. После загрузки обязательно проверить HTTPS-страницу, дату, текст и ссылки.

Перед публикацией форм, аналитики или юридических страниц выполнить `docs/landing_152fz_compliance_checklist_2026_07_15.md`. После неудачной выкладки вернуть файлы из последнего `build/ftp-backups/` и повторно проверить `/`, `/privacy`, `/terms` и формы.

## 5. Особенность FTP Host-Food

13.08.2026 проверено успешное подключение к FTP по имени `srv11.host-food.ru`. Подключение к ранее сохранённому адресу `91.227.16.11` разрывалось до авторизации, поэтому этот IP не использовать. Для диагностики сначала выполнять read-only подключение и просмотр `/www/winepool.ru`; скрипт деплоя предназначен для изменения сервера и не должен запускаться только ради проверки реквизитов.

## 6. Что считать устаревшим

- указания загружать `winepool.ru` в `public_html` на Timeweb;
- планы перенести прокси на «Timeweb VPS»;
- предположение, что регистратор домена автоматически является текущим веб-хостингом;
- смешивание FTP веб-сайта и SSH/VPS приложения.

Исторические документы могут сохранять старые решения как контекст, но должны явно ссылаться на этот runbook как на текущий источник истины.
