# H0.5 — public Tourism preview на Host-Food

Статус: canonical exact-tour page активирована 30.08.2026; owner preview
сохранён как noindex QA surface. 07.09.2026 тем же способом выложены страницы
винодельни.

## URL и границы

- owner preview тура:
  `https://winepool.ru/tourism-preview/o/yalta-excursions/t/yalta-massandra-tasting/`;
- owner preview винодельни:
  `https://winepool.ru/tourism-preview/w/massandra/` и
  `https://winepool.ru/tourism-preview/w/inkerman/`;
- canonical URL винодельни: `https://winepool.ru/wineries/{slug}/` — маршрут
  живёт в публичной сборке `lib/main_tourism_web.dart`; в основном приложении
  адрес `/wineries` занят админским списком виноделен, поэтому дублировать его
  там нельзя;
- canonical URL после приёмки:
  `https://winepool.ru/tourism/o/yalta-excursions/t/yalta-massandra-tasting/`;
- Flutter runtime/assets: `https://winepool.ru/tourism/app/`;
- `tour.winepool.ru` не является canonical host и зарезервирован как будущий
  короткий QR redirect layer.

Preview закрыт от индексации одновременно HTML meta-тегом и HTTP-заголовком
`X-Robots-Tag: noindex, nofollow, noarchive`. Его canonical указывает на
production URL. Canonical shell индексируема и загружает тот же принятый
Flutter runtime; legacy RuStore-only CTA удалён вместе со старой shell.

## Архитектура

Route-specific HTML shell генерируется из production
`get_published_tour_v1` (тур) либо `get_public_winery_page_v1` (винодельня),
поэтому title, description, OG image и JSON-LD не дублируют CRM-данные вручную.
Генератор один на оба вида страницы: различаются только источник данных и тип
schema.org — `TouristTrip` против `Winery`. Shell загружает единый Flutter entrypoint.

У страницы `<base href="/">`, чтобы GoRouter видел canonical/preview path без
подмены URL. Bootstrap явно задаёт:

- `entrypointBaseUrl: /tourism/app/`;
- `assetBase: /tourism/app/`;
- `canvasKitBaseUrl: /tourism/app/canvaskit/`.

CanvasKit хранится на WinePool, внешний Google CDN не используется. Tourism
build включает только нужный logo/font/runtime набор; OCR tessdata, skwasm,
debug symbols и недостижимые package assets исключаются. Размер deployment
package после canonical shell и logo optimization — `19.57 MiB`, 34 файла.

## Воспроизведение

```powershell
dart run tool/generate_tourism_preview_shell.dart `
  --operator yalta-excursions `
  --tour yalta-massandra-tasting `
  --output landing/tourism-preview/o/yalta-excursions/t/yalta-massandra-tasting/index.html

dart run tool/generate_tourism_preview_shell.dart `
  --operator yalta-excursions `
  --tour yalta-massandra-tasting `
  --route-path /tourism/o/yalta-excursions/t/yalta-massandra-tasting/ `
  --indexable true `
  --output landing/tourism/o/yalta-excursions/t/yalta-massandra-tasting/index.html

dart run tool/generate_tourism_preview_shell.dart `
  --winery massandra `
  --output landing/tourism-preview/w/massandra/index.html

dart run tool/generate_tourism_preview_shell.dart `
  --winery inkerman `
  --output landing/tourism-preview/w/inkerman/index.html

powershell -NoProfile -ExecutionPolicy Bypass `
  -File tool/build_tourism_web.ps1

powershell -NoProfile -ExecutionPolicy Bypass `
  -File tool/package_tourism_preview.ps1
```

Локальная production-like проверка:

```powershell
python tool/serve_tourism_web.py `
  --directory build/tourism_preview_package/public_html `
  --port 8765
```

Открыть тот же route на `http://127.0.0.1:8765`.

## Scoped deploy и rollback

Перед работой использовать общий Host-Food runbook и локальный credential
profile; значения credential не выводить и не коммитить.

```powershell
powershell -NoProfile -ExecutionPolicy Bypass `
  -File scripts/deploy_crisis_landing_ftp.ps1 `
  -PackageDir build\tourism_preview_package `
  -SkipPackaging `
  -DryRun
```

После проверки списка убрать `-DryRun`. Допустимы только
`tourism/app/**` и `tourism-preview/**`; удалений быть не должно. Deploy
29.08.2026 сохранил backup-контроль в
`build/ftp-backups/winepool-crisis-landing-20260829_130130` (новые файлы ранее
на сервере отсутствовали). HTTPS trailing-slash rule затем обновлён с backup
`build/ftp-backups/winepool-crisis-landing-20260829_130746`. Для rollback
preview достаточно удалить только эти два новых поддерева отдельной
подтверждённой операцией; основной Tourism URL при этом не затрагивается.

Premium master-detail carousel transition задеплоен отдельным scoped update
`tourism/app/main.dart.js` с backup
`build/ftp-backups/winepool-crisis-landing-20260829_132509`; синхронная
`1200 ms` vertical preview tape справа — с backup
`build/ftp-backups/winepool-crisis-landing-20260829_133428`.
Mobile scenic-break typography/height fix задеплоен тем же scoped способом с
backup `build/ftp-backups/winepool-crisis-landing-20260829_172418`.

Consent gate задеплоен 29.08.2026. Backup runtime-файлов сохранён в
`build/ftp-backups/winepool-crisis-landing-20260829_175935`, backup прежней
политики — в `build/ftp-backups/winepool-crisis-landing-20260829_180026`.
FTP сообщил protocol violation при закрытии потока `main.dart.js`, поэтому
результат проверен независимо: production size и SHA-256 runtime совпадают с
локальной release-сборкой. `/privacy.html` содержит редакцию 29.08.2026.
Мягкий optional-app value block success state задеплоен отдельным runtime
update с backup
`build/ftp-backups/winepool-crisis-landing-20260829_181643`.

## Smoke 29.08.2026

- preview shell: `HTTP 200`, `Cache-Control: no-store`, HTTP noindex;
- CanvasKit wasm: `HTTP 200`, `Content-Type: application/wasm`, immutable cache;
- `main.dart.js`: `HTTP 200`, `application/javascript`;
- существующий canonical tour URL: `HTTP 200`, прежний файл не изменён;
- preview URL без завершающего slash: один `301` на HTTPS-вариант со slash;
- published preview визуально загрузился без console error;
- локально проверены desktop и phone `390x844`;
- targeted `flutter analyze` — без замечаний.

## Следующий gate

1. ✅ Owner visual acceptance завершён на desktop/tablet/mobile и реальном
   Android-устройстве.
2. ✅ Consent gate применён: unchecked checkbox, legal links, server rejection
   и immutable evidence; hidden preview обновлён.
3. ✅ Web guest submission с отдельными QA-данными: consent evidence и одно
   operator notification подтверждены, лишней заявки без checkbox нет.
4. ✅ Raw/clean attribution reconciliation: raw `1`, clean `0`, test delta `1`,
   low sample `true`.
5. ✅ Android App Links принят на устройстве; iOS Universal Links принят как
   contract/server verified по явному owner decision без Apple device.

## Verified links status — 30.08.2026

- Android App Links configuration implemented for canonical
  `https://winepool.ru/tourism/o/...` and release package `ru.winepool.app`.
- `/.well-known/assetlinks.json` deployed as one scoped FTP artifact; the file
  did not previously exist, so rollback is deletion of that exact path. Local
  deployment control: `build/ftp-backups/winepool-crisis-landing-20260830_141012`.
- Production smoke: HTTP `200`, `Content-Type: application/json`, one release
  certificate fingerprint and expected package name.
- iOS Associated Domains entitlement is implemented for `winepool.ru`.
  Publication of `/.well-known/apple-app-site-association` is intentionally
  blocked until the real Apple Team ID is supplied; the placeholder template
  must never be deployed.
- Native acceptance still requires an APK containing the new manifest and an
  iOS build signed by the Apple team. Existing store builds cannot acquire new
  link entitlements from a server-only change.

Android owner acceptance завершён 30.08.2026 на Android 11:

- release-signed test build `1.1.0 (2012)` установлен поверх RuStore build без
  очистки пользовательских данных;
- Android domain state для `winepool.ru` — `always`;
- canonical HTTPS URL с `ref=h0_app_links_20260830` сразу открыл WinePool на
  точной странице тура «Дегустация вин в Массандре», без browser/chooser;
- повторная Android сборка или дополнительный server smoke не требуются.

iOS AASA подготовлен 30.08.2026 для app ID
`6ZD23YR74Z.ru.winepool.app` и canonical component `/tourism/o/*/t/*`.
`tour.winepool.ru` и preview route в association не включены.
Production `/.well-known/apple-app-site-association` и scoped MIME rule
опубликованы с deployment control
`build/ftp-backups/winepool-crisis-landing-20260830_143413`; оба файла ранее
отсутствовали, поэтому rollback — удаление только этих двух путей. HTTP smoke:
прямой `200`, без redirect, `Content-Type: application/json`, app ID и component
совпадают с локальным contract.

Реального iPhone/iPad для acceptance нет. По решению владельца iOS-контур
принимается как `contract/server verified`; device smoke явно отложен и не
подменяется симулятором или утверждением о фактическом открытии приложения.

## Canonical activation and performance — 30.08.2026

- indexable canonical shell атомарно заменила legacy page; backup shell и
  runtime headers: `build/ftp-backups/winepool-crisis-landing-20260830_145812`;
- gzip rule для CanvasKit сохранён отдельным control backup
  `build/ftp-backups/winepool-crisis-landing-20260830_145933`;
- logo `1248x1291`, который показывается максимум около 180 logical px,
  уменьшен без изменения композиции до `512x530`: `1,628,561 -> 331,254`
  bytes; production backup
  `build/ftp-backups/winepool-crisis-landing-20260830_150557`;
- canonical отвечает `200`, содержит index/follow, canonical/OG/JSON-LD из
  Published Tour revision `3`, запускает `/tourism/app/flutter_bootstrap.js` и
  больше не содержит legacy RuStore CTA;
- reproducible cold mobile profile (`390x844`, DPR 2.75, 4 Mbps, 100 ms RTT)
  после оптимизации: `9,166,143` transferred bytes, Flutter first frame
  `17,845 ms`, `CLS=0`; Chromium LCP для CanvasKit не считается надёжным и не
  выдаётся за content LCP;
- CanvasKit transport уменьшен примерно с `7.05` до `2.84 MiB`; hosting nginx
  продолжает отдавать `main.dart.js` без gzip (`3.57 MB`). Это зафиксированный
  H1 performance backlog до масштабирования платного трафика, а не скрытая
  метрика. H0 pilot gate принят с учётом owner QA на реальном Android и
  отдельного dedicated bundle вместо полной admin-сборки.

Повторяемый synthetic замер: `node tool/measure_tourism_web.mjs <canonical-url>`.

Страницы винодельни выложены 07.09.2026 тем же scoped-деплоем; backup —
`build/ftp-backups/winepool-crisis-landing-20260907_195241`. На сервере
появились только два новых поддерева `tourism-preview/w/massandra` и
`tourism-preview/w/inkerman`, удалений не было. `X-Robots-Tag: noindex,
nofollow, noarchive` на них проверен запросом после деплоя.

## Кэш бандла: почему `.htaccess` до `main.dart.js` не доходит

07.09.2026 после выкладки страницы винодельни выяснилось, что в Яндекс
браузере она отвечала текстом ошибки GoRouter «Откройте точную ссылку на тур
WinePool», а в Chrome и Firefox открывалась. Чистый профиль Яндекс браузера
открывал её нормально — значит дело было не в браузере, а в его кэше.

Причина найдена по заголовкам ответа:

| Файл | Server | ETag | Cache-Control |
|---|---|---|---|
| `index.html` | nginx | `W/"1405-65ae774c0c6e6"` (Apache) | `no-store` |
| `canvaskit.wasm` | nginx | `"6b9e40-…-gzip"` (Apache) | `immutable` |
| `main.dart.js` | nginx | `"6a9eec09-37ed46"` (nginx) | **пусто** |
| `favicon.png` | nginx | `"6a9eebea-395"` (nginx) | **пусто** |

На Host-Food перед Apache стоит nginx, и `.js` с `.png` он отдаёт сам. До
Apache такой запрос не доходит, поэтому блок `<IfModule mod_headers.c>` в
`landing/tourism/app/.htaccess` на них не действует — правило
`^main\.dart\.js$` не применялось никогда. Без заголовка браузер выбирает
срок хранения сам, и после деплоя мог сколько угодно держать прежний бандл, в
котором нового маршрута ещё нет.

**Решение — версия в адресе, а не заголовок.** HTML shell отдаёт Apache с
`no-store`, значит она всегда свежая. `tool/package_tourism_preview.ps1`
берёт первые 12 символов SHA-256 от `main.dart.js` и проставляет их:

- в shell — `<script src="/tourism/app/flutter_bootstrap.js?v=…">`;
- в самом bootstrap — `"mainJsPath":"main.dart.js?v=…"`.

Каждая новая сборка даёт новый адрес, и промахнуться мимо кэша нельзя.
Версия печатается при упаковке строкой `Bundle version:` и её видно в
исходном коде выложенной страницы.

Правила `.htaccess` для `.js` и `.png` оставлены как есть: они безвредны и
заработают, если хостинг когда-нибудь перестанет отдавать статику мимо Apache.

## Заставка загрузки

Заставка (`#winepool-loader`: логотип, искра, фотография виноградников)
рисуется в `web/tourism_index.html`. Route-specific shell раньше показывала
вместо неё серую надпись — то есть на всех живых адресах заставки не было.
Теперь `tool/generate_tourism_preview_shell.dart` вырезает её из того же
`web/tourism_index.html`, поэтому правится она по-прежнему в одном месте.
