# Локализация WinePool — Архитектура и статус

Последнее обновление: 22.06.2026

## Стек и генерация

- **Механизм**: Flutter `flutter_localizations` + `flutter gen-l10n` из `.arb`-файлов
- **Локали**: `ru` (Russian), `en` (English), `be` (Беларуская), `uz` (O'zbekcha); все `.arb` всегда в синхроне по ключам
- **ARB-файлы**: `lib/l10n/app_ru.arb`, `lib/l10n/app_en.arb`, `lib/l10n/app_be.arb`, `lib/l10n/app_uz.arb`
- **Генерированные файлы** (не редактировать вручную):
  - `lib/l10n/app_localizations.dart`
  - `lib/l10n/app_localizations_en.dart`
  - `lib/l10n/app_localizations_ru.dart`
  - `lib/l10n/app_localizations_be.dart`
  - `lib/l10n/app_localizations_uz.dart`
- **Extension**: `context.l10n` → `lib/core/localization/app_localizations_x.dart`
- **Команда**: `flutter gen-l10n` (настройки в `l10n.yaml`)

## Правила написания l10n-кода

### Синхронные методы (build, State)
```dart
// Безопасно — использовать context.l10n напрямую
Text(context.l10n.someKey)
```

### Асинхронные методы (с await)
```dart
// ОБЯЗАТЕЛЬНО кешировать l10n ДО первого await
Future<void> _doSomething(BuildContext context) async {
  final l10n = context.l10n;  // ← здесь, до await
  await someAsyncCall();
  if (context.mounted) {
    showSnackBar(l10n.someKey); // ← l10n.xxx, не context.l10n.xxx
  }
}
```

### Top-level функции, которым нужен l10n
```dart
// Добавить AppLocalizations как первый параметр
String _myHelper(AppLocalizations l10n, OtherArg arg) { ... }

// Call site:
_myHelper(context.l10n, arg)
```

### Параметризованные строки
```arb
"welcomeUser": "Hello, {name}!",
"@welcomeUser": {
  "placeholders": {
    "name": {"type": "String"}
  }
}
```
```dart
context.l10n.welcomeUser('Alice')
```

## Что уже локализовано (статус на 22.06.2026)

### Полностью готово — non-admin экраны

| Файл | Примечание |
|---|---|
| `auth/` — login, register, reset_password | |
| `cellar/` — my_cellar_screen | |
| `notifications/` — notifications_screen | |
| `search/` — search_results_screen | |
| `wines/` — wine_details, wine_compare_prep, add_edit_wine, receipt_details, winery_details | wine_lines UI + duplicate reasons |
| `receipt/` — receipt_qr_scanner_screen | |
| `draft/` — draft_wine_details_screen | |
| `catalog/` — catalog_screen, category_screen, widgets/ | filter chips, line filter |
| `cellar/` — my_cellar_screen | |
| `onboarding/` — onboarding_screen | |
| `add_bottle/` — hub, barcode_scan, manual_draft | |
| `profile/` — profile_screen | XP-события через action-код маппинг |
| `winery_tourism/` — tourism_list, tourism_experience_details, tourism_operator | заголовок, фильтр-чипы, дисклеймеры |
| `admin/` — reviews, draft_submission_details, dashboard, workbench | |

### На паузе (admin-экраны)
- `admin_tourism_experiences_screen.dart` — ~400 строк, сложная CMS
- Отложено до решения о переводе

### Новые языки

На 22.06.2026 созданы и подключены:

- `app_be.arb` (Беларуская), `@@locale = be`;
- `app_uz.arb` (O'zbekcha, latin), `@@locale = uz`.

Оба файла содержат полный набор ключей шаблона `app_ru.arb`: **3355 текстовых ключей** + **790 metadata-записей**. После `flutter gen-l10n` обе локали входят в `AppLocalizations.supportedLocales`.

Проверка:

```powershell
node -e "const fs=require('fs'); for (const f of ['lib/l10n/app_ru.arb','lib/l10n/app_en.arb','lib/l10n/app_be.arb','lib/l10n/app_uz.arb']) { const d=JSON.parse(fs.readFileSync(f,'utf8')); console.log(f, Object.keys(d).filter(k=>!k.startsWith('@')).length, d['@@locale']); }"
flutter gen-l10n
flutter analyze lib/l10n/
```

## Что НЕЛЬЗЯ локализовать (важные исключения)

| Тип строки | Причина |
|---|---|
| OCR/regex-паттерны для распознавания | Матчатся с входящим текстом, менять нельзя |
| Строки-ключи map/switch, не отображаемые пользователю | Логика приложения, не UI |
| Значения фильтров, используемые в сравнении И отображении | Изменение сломает фильтрацию |
| Значения, сохраняемые в БД как данные (например `qualityReason:`) | Хранятся в БД, не должны зависеть от языка |
| Строки в `kDebugMode`-блоках | Debug-only, не для пользователя |
| `'Не указано'` в `_FieldRow.build` | Особый случай: кешировать через `final notSpecified = context.l10n.commonNotSpecified;` и использовать для присвоения И сравнения |
| Символ `₽` и виджет `CurrencyText` | Принадлежит multi-currency PR, не трогать |

## Locale-aware changelog обновлений

### Архитектура (реализовано 21.06.2026)

Контент changelog хранится в Supabase в двух таблицах:
- `app_release_config` — конфиг текущего релиза
- `app_releases` — история релизов

**Миграция** `supabase/migrations/20260621_add_bilingual_release_content.sql` добавляет в обе таблицы:
- `summary_en TEXT` — краткое описание на EN (fallback → RU `summary`)
- `highlights_en JSONB` — ключевые улучшения на EN (fallback → RU `highlights`)
- `release_notes_en JSONB` — детальные заметки на EN (fallback → RU `release_notes`)

**Миграция** `supabase/migrations/20260622_add_be_uz_release_content.sql` добавляет:
- `summary_be`, `highlights_be`, `release_notes_be`
- `summary_uz`, `highlights_uz`, `release_notes_uz`

Production self-host: применена 22.06.2026, после ALTER отправлен
`NOTIFY pgrst, 'reload schema'`. Текущая опубликованная строка `1.0.5+6`
оставляет BE/UZ-поля пустыми (`summary_* = null`, JSONB-поля `[]`); их
заполняем post-publish SQL для следующей версии.

**Применить на VPS:**
```powershell
# На VPS через Docker psql
ssh root@91.227.18.214 "docker exec -i supabase-db psql -U supabase_admin -d postgres -f /path/to/migration.sql"
# Или вставить SQL вручную через psql
```

**Логика выбора языка в UI** (`_AppVersionCard`, `_ReleaseNotesBlock`):
```dart
final languageCode = Localizations.localeOf(context).languageCode;
// languageCode == 'ru' → RU-контент
// languageCode == 'en' → EN-контент (fallback → RU)
// languageCode == 'be' → BE-контент (fallback → RU)
// languageCode == 'uz' → UZ-контент (fallback → RU)
```

**Методы модели:**
- `AppUpdateInfo.rawSummaryFor(languageCode)` — locale-aware summary
- `AppReleaseNote.notesFor(languageCode)` — locale-aware список заметок
- `AppUpdateInfo.homeHighlightsFor(languageCode)` — locale-aware highlights для баннера на главной

### Как заполнять при выпуске новой версии

При добавлении записи в `app_release_config` или `app_releases`:
1. Заполнить `summary`, `highlights`, `release_notes` на **русском**
2. Заполнить `summary_en`, `highlights_en`, `release_notes_en` на **английском**
3. Заполнить `summary_be`, `highlights_be`, `release_notes_be` на **белорусском**
4. Заполнить `summary_uz`, `highlights_uz`, `release_notes_uz` на **узбекском**
5. Если поле конкретного языка пустое — пользователь увидит RU-текст (безопасный fallback)

---

## Добавление нового языка — полный чеклист

> Пример: добавляем Белорусский (`be`). Везде заменяй `be`/`belarusian`/`Беларуская` на нужный язык.

### Шаг 0 — Снять MVP-лок

В `lib/core/localization/locale_controller.dart` изменить:
```dart
const _lockRussianLocaleForMvp = false;  // было true
```
**Это нужно сделать один раз** — после снятия лока пользователи смогут переключать язык в профиле.

---

### Шаг 1 — Создать ARB-файл с переводом

Создать `lib/l10n/app_be.arb`. Структура:
```json
{
  "@@locale": "be",
  "appTitle": "WinePool",
  "languageSystem": "Сістэмная мова",
  ...
}
```

**Откуда брать ключи:** скопировать `app_ru.arb` целиком, заменить `"@@locale": "ru"` на `"@@locale": "be"`, перевести все значения. Мета-записи `@key` копируются без изменений.

**Количество ключей:** на 22.06.2026 — **3355 текстовых ключей** (проверить: `node -e "const d=JSON.parse(require('fs').readFileSync('lib/l10n/app_ru.arb','utf8')); console.log(Object.keys(d).filter(k=>!k.startsWith('@')).length)"`).

Разбивка по группам:
| Группа (префиксы ARB) | Ключей | Приоритет |
|---|---|---|
| `admin`, `workbench`, `suspect`, `seller` | ~823 | Командные/admin-поверхности, тоже переводим перед релизом |
| `receipt` | ~454 | Высокий |
| `tourism` | ~359 | Высокий |
| `draft` | ~293 | Высокий |
| `wine` | ~190 | Высокий |
| `add`, `profile`, `cellar`, `catalog` | ~525 | Высокий |
| Прочее (auth, map, winery, common...) | ~686 | Высокий |

**Стратегия:** переводить все ключи, включая admin/workbench/suspect/seller. Если нужно временно ускорить промежуточный build, admin-группы можно оставить русским fallback, но перед релизной сборкой их нужно закрыть.

После создания файла:
```bash
flutter gen-l10n
```
Это автоматически создаст `lib/l10n/app_localizations_be.dart`. В `AppLocalizations.supportedLocales` `be` появится само.

---

### Шаг 2 — Добавить язык в контроллер

В `lib/core/localization/locale_controller.dart`:

```dart
enum AppLanguage {
  system(null),
  russian('ru'),
  english('en'),
  belarusian('be'),  // ← добавить
  // uzbek('uz'),    // аналогично для других
}
```

В `fromStoredCode`:
```dart
case 'be':
  return AppLanguage.belarusian;
```

В `fromLocale`:
```dart
case 'be':
  return AppLanguage.belarusian;
```

---

### Шаг 3 — Добавить название языка в ARB

В **оба** `app_en.arb` и `app_ru.arb` (и все остальные ARB) добавить ключ:
```json
"languageBelarusian": "Беларуская",
```

В `app_be.arb`:
```json
"languageBelarusian": "Беларуская",
```

---

### Шаг 4 — Добавить в UI языкового пикера

В `lib/features/profile/presentation/profile_screen.dart` найти все switch по `AppLanguage` и добавить ветку:
```dart
// Два места — _langLabel и метод в _LanguagePickerSheet:
AppLanguage.belarusian => l10n.languageBelarusian,
```

---

### Шаг 5 — Заполнить DB-контент на новом языке (опционально)

Не всё в приложении берётся из ARB. Часть контента хранится в БД в виде Russian-текста:

| Контент | Таблица | Решение |
|---|---|---|
| Релизные заметки (changelog) | `app_release_config`, `app_releases` | Уже есть `summary/highlights/release_notes` для RU и суффиксы `_en`, `_be`, `_uz`; новые релизы заполнять на всех 4 языках |
| Описания туров, операторов | `tourism_experiences`, `tourism_organizations` | Добавить поля `title_be`, `description_be` — крупная работа, можно пропустить |
| Имена вин, виноделен | `wines`, `wineries` | Не локализуется — собственные имена |
| XP-события (профиль) | `user_experience_events` | Уже решено через маппинг `action`-кода → ARB ключ. **Для нового языка ничего не нужно** — добавь перевод ключей `xpEvent*` в `app_be.arb`. |
| Atlas-контент | `wine_atlas_entries` | Добавить поля если нужно |

**Вывод:** для CIS-языков (белорусский, узбекский) DB-контент на русском — приемлемый fallback. Для не-CIS языков нужна полноценная DB-локализация.

---

### Шаг 6 — Проверка качества перевода

Чеклист перед релизом:
- [ ] Все `3355` текстовых ключей синхронизированы во всех ARB
- [ ] Параметризованные строки сохранены: `{name}`, `{count}`, `{error}` — не переведены, не удалены
- [ ] `flutter gen-l10n` без ошибок
- [ ] `flutter analyze` без ошибок
- [ ] Ручная проверка ключевых экранов:
  - Онбординг (4 страницы)
  - Профиль → Настройки → Язык (видит новый язык в списке)
  - Главный экран
  - Каталог + фильтры
  - Страница вина
  - Туризм: список, тур, оператор
  - XP-события в профиле (блок "Вклад")
- [ ] Числа и даты форматируются правильно (Dart `intl` использует locale)
- [ ] Длинные переводы не переполняют UI (особенно кнопки и чипы)

---

### Быстрая справка: что НЕ надо трогать при добавлении языка

- `main.dart` — `supportedLocales` берётся из `AppLocalizations.supportedLocales` автоматически
- `l10n.yaml` — не нужно добавлять `supported-locales`, файл обнаруживается автоматически
- `GlobalMaterialLocalizations.delegate` и другие delegates — уже подключены в `main.dart`, работают для всех локалей из flutter_localizations пакета (проверь что нужная локаль поддерживается)

---

### Порядок операций (итого)

```
1. locale_controller.dart  → _lockRussianLocaleForMvp = false
2. app_be.arb              → создать, перевести полный набор ключей из app_ru.arb
3. flutter gen-l10n        → запустить
4. locale_controller.dart  → добавить AppLanguage.belarusian + switch-ветки
5. app_en.arb + app_ru.arb → добавить languageBelarusian ключ
6. app_be.arb              → добавить languageBelarusian ключ
7. flutter gen-l10n        → запустить ещё раз
8. profile_screen.dart     → добавить ветку в _langLabel и _LanguagePickerSheet
9. flutter analyze         → проверить
10. Ручное тестирование на устройстве
```

---

## Пикер валюты (profile_screen.dart)

Список поддерживаемых валют определён в `display_currency_controller.dart`:
- `SupportedCurrency.label` и `.description` хранятся на русском (const)
- Локализованные строки берутся в UI через switch по `currency.code`:
  ```dart
  final (label, description) = switch (currency.code) {
    'BYN' => (l10n.currencyBynLabel, l10n.currencyBynDescription),
    'USD' => (l10n.currencyUsdLabel, l10n.currencyUsdDescription),
    _ => (l10n.currencyRubLabel, l10n.currencyRubDescription),
  };
  ```
- При добавлении новой валюты: добавить ключи `currency{Code}Label` / `currency{Code}Description` в оба ARB, добавить ветку в switch.

## Структура профиля (обновлено 21.06.2026)

### Авторизованный пользователь
1. Карточка пользователя
2. Вклад в каталог
3. МОЯ КОЛЛЕКЦИЯ — Погребок, Избранные, Моя карта (3 пункта)
4. ИНСТРУМЕНТЫ — Сканер, OCR, Черновики, История чеков, Уведомления
5. ОТКРЫТИЯ — Атлас, WinePool туризм, Заявки на туры
6. АДМИНИСТРИРОВАНИЕ (условно)
7. МОЯ ВИНОДЕЛЬНЯ (условно)
8. ПАРТНЁРСКИЕ КАНАЛЫ О ВИНЕ
9. НАСТРОЙКИ — Валюта (единый параметр) + Язык приложения
10. АККАУНТ
11. Версия приложения
12. Выйти / Удалить аккаунт

### Гостевой режим
1. Гостевая карточка (CTA регистрации/входа)
2. ИНСТРУМЕНТЫ
3. ПАРТНЁРСКИЕ КАНАЛЫ О ВИНЕ
4. НАСТРОЙКИ — Валюта + Язык
5. АККАУНТ
6. Версия приложения

### Единая валюта
Выбор одной валюты обновляет одновременно `displayCurrency` и `manualEntryCurrency`.
Провайдеры (`displayCurrencyCodeProvider`, `manualEntryCurrencyCodeProvider`) остались без изменений — слияние только на уровне UI.

## Отложенная локализация (pending) — обновлено 25.07.2026

Часть админ/модерационных и caталожных экранов исторически содержит русский текст
напрямую, без ARB-ключей. Локализацию по ним нужно провести отдельным проходом
(сейчас не блокирует, но при мультиязычном релизе потребуется). Известные зоны:

- экраны модерации черновиков и предложений релизов
  (`admin_draft_catalog_submission_details_screen.dart`,
  `admin_catalog_resolution_workbench_screen.dart`), редактор вина
  (`add_edit_wine_screen.dart`), sheet предложения релиза
  (`wine_release_proposal_sheet.dart`), менеджер релизов
  (`admin_wine_release_manager.dart`);
- строки, добавленные в Сессии 2 (в т.ч. подэтап 3a: «Участников: N», «Вклад …»,
  честное сообщение о дополнении предложения; подэтап 3b: кандидаты полей; 16.2:
  переключатель «Детальный режим добавления» в профиле и контекстная подсказка в
  confirm sheet) — вшиты как RU. Профиль и add-bottle в остальном локализованы,
  поэтому эти строки при l10n-проходе вынести в ARB в первую очередь;
- при локализации: вынести строки в `app_ru.arb`/`app_en.arb` (+ be/uz по политике),
  прогнать `flutter gen-l10n`, заменить литералы на `context.l10n.*`.
