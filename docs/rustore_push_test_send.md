# RuStore push: test send

Инструкция для ручной проверки Android push-уведомления WinePool.

## Что нужно

- `RUSTORE_PROJECT_ID` из RuStore Console.
- `RUSTORE_SERVICE_TOKEN` из раздела сервисных токенов RuStore.
- `RUSTORE_DEVICE_TOKEN` из таблицы `public.user_push_tokens` для нужного устройства.
- На телефоне должна быть установлена release-сборка с тем же package name и SHA-256, который указан в RuStore project.
- После переустановки приложения его нужно открыть хотя бы один раз и войти в аккаунт, чтобы token заново зарегистрировался.

## Где лежат ключи

`RUSTORE_PROJECT_ID` и `RUSTORE_SERVICE_TOKEN` — в `.env` в корне проекта (не
коммитится). Токен устройства — из базы на VPS:
`select token from public.user_push_tokens order by updated_at desc limit 1;`

## Отправка в оба канала (RuStore + FCM)

С 21.09.2026 приложение принимает push по двум каналам. Основная проверка —
универсальный API:

```powershell
# в окружении: RUSTORE_PROJECT_ID, RUSTORE_SERVICE_TOKEN, FCM_SERVICE_ACCOUNT_JSON (из .env)
python scripts/send_universal_push.py --rustore-token <rustore> --fcm-token <fcm>
python scripts/send_universal_push.py --only fcm --fcm-token <fcm>   # один канал
```

Токены — `select provider, token from public.user_push_tokens ...` на VPS.
Успех — `200 {"status":"OK"}`. Уведомление должно быть одно.

## Отправка только в RuStore (старый скрипт)

Из корня проекта:

```powershell
$env:RUSTORE_PROJECT_ID = "..."
$env:RUSTORE_SERVICE_TOKEN = "..."
$env:RUSTORE_DEVICE_TOKEN = "..."

.\scripts\send_rustore_catalog_push.ps1
```

Успешный ответ RuStore для `messages:send` обычно выглядит как пустой JSON:

```json
{}
```

Это значит, что запрос принят. Фактическую доставку на Android проверяем на устройстве и, при необходимости, через:

```powershell
adb logcat -d | Select-String -Pattern "WinePoolRustorePush|PushDelivery|Rustore"
adb shell dumpsys notification --noredact | Select-String -Pattern "WinePool|Новинка|Q Бордо" -Context 2,2
```

Важно: для WinePool-теста скрипт отправляет `data`-only push. Если добавить в payload блок
`notification`, RuStore SDK может показать ещё одно системное уведомление на своём default channel,
а затем приложение покажет наше кастомное уведомление вторым экземпляром.

## Частые причины, почему не пришло

- Приложение переустановлено, но ещё не открывалось после установки.
- Взяли старый `RUSTORE_DEVICE_TOKEN`.
- Уведомления для WinePool отключены в системных настройках Android.
- Токен RuStore был пересоздан или отключён в консоли.
- Телефон/транспорт RuStore временно не доставил сообщение, хотя API принял запрос.
