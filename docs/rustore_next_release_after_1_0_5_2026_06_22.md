# RuStore Next Release After 1.0.5

Дата подготовки: 22.06.2026

Статус с 25.08.2026: historical release-candidate memo; версии 1.0.6 и 1.1.0 опубликованы. Не использовать для подготовки следующего релиза.

## Статус

Release candidate документ для следующего публичного обновления после
опубликованной `1.0.5+6`.

Текущий код уже имеет:

```text
pubspec.yaml: 1.0.6+7
```

Перед сборкой нужно принять финальный `versionName`. Если это обычный публичный
инкремент после `1.0.5`, рекомендуемый вариант:

```text
versionName: 1.0.6
APP_BUILD_NUMBER: 7
Ожидаемый RuStore build для split APK: 1.0.6(2007)
```

Фактически подготовлено 22.06.2026:

```text
pubspec.yaml: 1.0.6+7
Сборка: flutter build apk --release --target-platform android-arm64 --split-per-abi --dart-define=APP_VERSION_NAME=1.0.6 --dart-define=APP_BUILD_NUMBER=7
APK: build/app/outputs/flutter-apk/winepool-android-arm64-v1.0.6.apk
Size: 116746205 bytes (111.3 MB)
SHA256: 0B6D650CDCFF38619ED77E5FCC99290DD0B2E8C5E3340EE2A4792A32BBC725BD
```

## Что Входит После 1.0.5

### Локализация

- Подключены новые языки: беларуская (`be`) и узбекский латиницей (`uz`).
- Созданы `lib/l10n/app_be.arb` и `lib/l10n/app_uz.arb`.
- Оба ARB содержат полный набор `app_ru.arb`: `3355` текстовых ключей и `790`
  metadata-записей.
- Переведены также admin/workbench/suspect/seller ключи.
- `flutter gen-l10n` создаёт `app_localizations_be.dart` и
  `app_localizations_uz.dart`; `supportedLocales` включает `be`, `en`, `ru`,
  `uz`.
- Первый запуск без сохранённого языка использует системную локаль.

### Мультивалютность И Чеки Узбекистана

- Добавлены базовые валюты `RUB`, `BYN`, `USD`, `UZS`.
- Выбор валюты в профиле стал единым для display currency и ручного ввода.
- Добавлен контекст валюты ручной цены.
- Символы валют централизованы в shared widgets.
- Реализованы чеки Узбекистана и парсер ответа UZ-провайдера.
- Добавлена ежедневная синхронизация курсов и base-price слой для аналитики.
- Карта покупок и карта конкретного вина используют base-цены и не учитывают
  строки без явных wine positions.

### Профиль И Update Notice

- Профиль реструктурирован: коллекция, инструменты, открытия, настройки,
  аккаунт и версия приложения разнесены по понятным секциям.
- Язык и валюта находятся в секции настроек.
- Language/currency picker стали scrollable bottom sheets на 70% высоты, чтобы
  не переполняться при новых языках.
- Баннер обновления и карточка версии локализованы.
- Release notes в БД поддерживают RU/EN/BE/UZ-поля с fallback на RU.

### Стабильность

- Исправлен краш, который мог возникать у части пользователей в активном
  сценарии. В release notes оставить это видимым пунктом, чтобы пользователи,
  столкнувшиеся с падением, понимали причину обновиться.

### iOS Groundwork

- Bundle id приведён к `ru.winepool.app`.
- iOS deployment target поднят до `16.0`.
- Убран конфликт `google_mlkit_barcode_scanning`; QR остаётся через
  `mobile_scanner`.
- `ebs_plugin` отключён для iOS, Android сохранён.
- Обновлён `yandex_mapkit`.
- Добавлена alpha-free iOS иконка и permission descriptions.

### Лендинг

- Обновлены файлы лендинга и SEO-страниц.
- Текущий вариант не считать опубликованным: в commit message отмечено, что
  лендинг требует доработки и не выложен на хостинг.

## RuStore Что Нового

Короткая версия:

```text
Добавили белорусский и узбекский интерфейс, поддержку чеков Узбекистана, мультивалютные цены, улучшили настройки профиля и исправили краш у части пользователей.
```

Подробная версия:

```text
WinePool стал удобнее для пользователей в разных странах: добавлены белорусский и узбекский интерфейсы, а приложение теперь может работать с чеками Узбекистана и валютой UZS.

Цены стали аккуратнее в мультивалютном сценарии: покупки из чеков и ручного ввода хранят исходную валюту, а каталог, карта и аналитика считают агрегаты через базовую цену.

В профиле обновили настройки: язык и валюта собраны в одном месте, выбор валюты стал единым для отображения и ручного ввода, а длинные списки языков и валют больше не переполняют экран.

Продолжили локализацию интерфейса: переведены оставшиеся экраны, админские разделы, история чеков, туризм, карта, каталог, погребок и уведомления об обновлениях.

Также исправили краш, который мог возникать у части пользователей в активном сценарии.
```

## In-App Release Notes

```text
summary:
Белорусский и узбекский интерфейс, чеки Узбекистана, мультивалютные цены, обновлённые настройки профиля и исправление краша.

highlights:
- Белорусская и узбекская локализация интерфейса
- Поддержка чеков Узбекистана и валюты UZS
- Мультивалютные цены RUB/BYN/USD/UZS и корректные агрегаты на карте
- Обновлённый профиль: язык и валюта в настройках
- Плашка обновления и история версий выбирают текст по языку интерфейса
- Исправлен краш, который мог возникать у части пользователей
```

## Pre-RC QA

- `flutter gen-l10n`
- `flutter analyze lib/l10n/`
- переключение `ru/en/be/uz` в профиле;
- проверка language/currency picker на маленьком экране;
- UZ-чек: распознавание, валюта `UZS`, сохранение цены;
- BYN/UZS покупки на карте и карте конкретного вина;
- in-app update banner на RU/EN;
- iOS sanity build, если этот build идёт в App Store/TestFlight track.

## Post-Publish SQL Шаблон

Выполнять только после публикации APK в RuStore и после выбора финального
`versionName`.

```sql
-- Заменить 1.0.6 / 7, если перед сборкой будет принято другое имя версии.
WITH release_row AS (
  INSERT INTO public.app_releases (
    platform,
    version_name,
    build_number,
    summary,
    highlights,
    release_notes,
    summary_en,
    highlights_en,
    release_notes_en,
    summary_be,
    highlights_be,
    release_notes_be,
    summary_uz,
    highlights_uz,
    release_notes_uz,
    store_url,
    is_active,
    published_at
  )
  VALUES (
    'android',
    '1.0.6',
    7,
    'Белорусский и узбекский интерфейс, чеки Узбекистана, мультивалютные цены, обновлённые настройки профиля и исправление краша.',
    '[
      "Белорусская и узбекская локализация интерфейса",
      "Поддержка чеков Узбекистана и валюты UZS",
      "Мультивалютные цены RUB/BYN/USD/UZS и корректные агрегаты на карте",
      "Обновлённый профиль: язык и валюта в настройках",
      "Плашка обновления и история версий выбирают текст по языку интерфейса",
      "Исправлен краш, который мог возникать у части пользователей"
    ]'::jsonb,
    '[
      "Добавлены беларуская и узбекская локализации интерфейса.",
      "Подключены чеки Узбекистана и валюта UZS.",
      "Покупки из чеков и ручного ввода хранят исходную валюту и корректно участвуют в агрегатах через base-price слой.",
      "В профиле язык и валюта собраны в настройках; выбор валюты стал единым для отображения и ручного ввода.",
      "Плашка обновления и история версий выбирают текст по языку интерфейса.",
      "Исправлен краш, который мог возникать у части пользователей."
    ]'::jsonb,
    'Belarusian and Uzbek UI, Uzbekistan receipts, multi-currency prices, updated profile settings, and a crash fix.',
    '[
      "Belarusian and Uzbek interface localization",
      "Uzbekistan receipts and UZS currency support",
      "Multi-currency prices with correct map aggregates",
      "Updated profile settings for language and currency",
      "Localized update notice and release history",
      "Fixed a crash that could affect some users"
    ]'::jsonb,
    '[
      "Added Belarusian and Uzbek interface localization.",
      "Added Uzbekistan receipt parsing and UZS currency support.",
      "Receipt and manual prices keep their source currency and participate in analytics through base prices.",
      "Language and currency settings are now grouped in profile settings.",
      "Update notices and release history choose text by the current interface language.",
      "Fixed a crash that could affect some users."
    ]'::jsonb,
    'Беларускі і ўзбекскі інтэрфейс, чэкі Узбекістана, мультывалютныя цэны, абноўленыя налады профілю і выпраўленне збою.',
    '[
      "Беларуская і ўзбекская лакалізацыя інтэрфейсу",
      "Падтрымка чэкаў Узбекістана і валюты UZS",
      "Мультывалютныя цэны з карэктнымі агрэгатамі на карце",
      "Абноўленыя налады профілю для мовы і валюты",
      "Лакалізаванае паведамленне пра абнаўленне і гісторыя версій",
      "Выпраўлены збой, які мог узнікаць у часткі карыстальнікаў"
    ]'::jsonb,
    '[
      "Дададзена беларуская і ўзбекская лакалізацыя інтэрфейсу.",
      "Дададзена распазнаванне чэкаў Узбекістана і падтрымка валюты UZS.",
      "Цэны з чэкаў і ручнога ўводу захоўваюць зыходную валюту і ўдзельнічаюць у аналітыцы праз базавыя цэны.",
      "Налады мовы і валюты цяпер сабраныя ў профілі.",
      "Паведамленні пра абнаўленне і гісторыя версій выбіраюць тэкст паводле мовы інтэрфейсу.",
      "Выпраўлены збой, які мог узнікаць у часткі карыстальнікаў."
    ]'::jsonb,
    'Belaruscha va o''zbekcha interfeys, O''zbekiston cheklari, ko''p valyutali narxlar, yangilangan profil sozlamalari va nosozlik tuzatildi.',
    '[
      "Belaruscha va o''zbekcha interfeys lokalizatsiyasi",
      "O''zbekiston cheklari va UZS valyutasini qo''llab-quvvatlash",
      "Xaritadagi agregatlar to''g''ri hisoblanadigan ko''p valyutali narxlar",
      "Til va valyuta uchun yangilangan profil sozlamalari",
      "Lokalizatsiya qilingan yangilanish xabari va versiyalar tarixi",
      "Ayrim foydalanuvchilarda yuz berishi mumkin bo''lgan nosozlik tuzatildi"
    ]'::jsonb,
    '[
      "Belaruscha va o''zbekcha interfeys lokalizatsiyasi qo''shildi.",
      "O''zbekiston cheklarining tanilishi va UZS valyutasi qo''llab-quvvatlandi.",
      "Chek va qo''lda kiritilgan narxlar asl valyutasini saqlaydi va bazaviy narxlar orqali analitikada qatnashadi.",
      "Til va valyuta sozlamalari profilda bir joyga yig''ildi.",
      "Yangilanish xabarlari va versiyalar tarixi joriy interfeys tiliga qarab matn tanlaydi.",
      "Ayrim foydalanuvchilarda yuz berishi mumkin bo''lgan nosozlik tuzatildi."
    ]'::jsonb,
    'https://www.rustore.ru/catalog/app/ru.winepool.app',
    true,
    now()
  )
  ON CONFLICT (platform, build_number) DO UPDATE
  SET
    version_name = EXCLUDED.version_name,
    summary = EXCLUDED.summary,
    highlights = EXCLUDED.highlights,
    release_notes = EXCLUDED.release_notes,
    summary_en = EXCLUDED.summary_en,
    highlights_en = EXCLUDED.highlights_en,
    release_notes_en = EXCLUDED.release_notes_en,
    summary_be = EXCLUDED.summary_be,
    highlights_be = EXCLUDED.highlights_be,
    release_notes_be = EXCLUDED.release_notes_be,
    summary_uz = EXCLUDED.summary_uz,
    highlights_uz = EXCLUDED.highlights_uz,
    release_notes_uz = EXCLUDED.release_notes_uz,
    store_url = EXCLUDED.store_url,
    is_active = true,
    published_at = COALESCE(public.app_releases.published_at, EXCLUDED.published_at),
    updated_at = now()
  RETURNING id
)
UPDATE public.app_release_config
SET
  latest_release_id = release_row.id,
  latest_version_name = '1.0.6',
  latest_build_number = 7,
  min_supported_build_number = 1,
  store_url = 'https://www.rustore.ru/catalog/app/ru.winepool.app',
  title = 'Доступно обновление WinePool',
  message = 'Добавили белорусский и узбекский интерфейс, чеки Узбекистана, мультивалютные цены и исправление краша.',
  summary = 'Белорусский и узбекский интерфейс, чеки Узбекистана, мультивалютные цены, обновлённые настройки профиля и исправление краша.',
  highlights = '[
    "Белорусская и узбекская локализация интерфейса",
    "Поддержка чеков Узбекистана и валюты UZS",
    "Мультивалютные цены RUB/BYN/USD/UZS и корректные агрегаты на карте",
    "Обновлённый профиль: язык и валюта в настройках",
    "Плашка обновления и история версий выбирают текст по языку интерфейса",
    "Исправлен краш, который мог возникать у части пользователей"
  ]'::jsonb,
  release_notes = '[
    "Добавлены беларуская и узбекская локализации интерфейса.",
    "Подключены чеки Узбекистана и валюта UZS.",
    "Покупки из чеков и ручного ввода хранят исходную валюту и корректно участвуют в агрегатах через base-price слой.",
    "В профиле язык и валюта собраны в настройках; выбор валюты стал единым для отображения и ручного ввода.",
    "Плашка обновления и история версий выбирают текст по языку интерфейса.",
    "Исправлен краш, который мог возникать у части пользователей."
  ]'::jsonb,
  summary_en = 'Belarusian and Uzbek UI, Uzbekistan receipts, multi-currency prices, updated profile settings, and a crash fix.',
  highlights_en = '[
    "Belarusian and Uzbek interface localization",
    "Uzbekistan receipts and UZS currency support",
    "Multi-currency prices with correct map aggregates",
    "Updated profile settings for language and currency",
    "Localized update notice and release history",
    "Fixed a crash that could affect some users"
  ]'::jsonb,
  release_notes_en = '[
    "Added Belarusian and Uzbek interface localization.",
    "Added Uzbekistan receipt parsing and UZS currency support.",
    "Receipt and manual prices keep their source currency and participate in analytics through base prices.",
    "Language and currency settings are now grouped in profile settings.",
    "Update notices and release history choose text by the current interface language.",
    "Fixed a crash that could affect some users."
  ]'::jsonb,
  summary_be = 'Беларускі і ўзбекскі інтэрфейс, чэкі Узбекістана, мультывалютныя цэны, абноўленыя налады профілю і выпраўленне збою.',
  highlights_be = '[
    "Беларуская і ўзбекская лакалізацыя інтэрфейсу",
    "Падтрымка чэкаў Узбекістана і валюты UZS",
    "Мультывалютныя цэны з карэктнымі агрэгатамі на карце",
    "Абноўленыя налады профілю для мовы і валюты",
    "Лакалізаванае паведамленне пра абнаўленне і гісторыя версій",
    "Выпраўлены збой, які мог узнікаць у часткі карыстальнікаў"
  ]'::jsonb,
  release_notes_be = '[
    "Дададзена беларуская і ўзбекская лакалізацыя інтэрфейсу.",
    "Дададзена распазнаванне чэкаў Узбекістана і падтрымка валюты UZS.",
    "Цэны з чэкаў і ручнога ўводу захоўваюць зыходную валюту і ўдзельнічаюць у аналітыцы праз базавыя цэны.",
    "Налады мовы і валюты цяпер сабраныя ў профілі.",
    "Паведамленні пра абнаўленне і гісторыя версій выбіраюць тэкст паводле мовы інтэрфейсу.",
    "Выпраўлены збой, які мог узнікаць у часткі карыстальнікаў."
  ]'::jsonb,
  summary_uz = 'Belaruscha va o''zbekcha interfeys, O''zbekiston cheklari, ko''p valyutali narxlar, yangilangan profil sozlamalari va nosozlik tuzatildi.',
  highlights_uz = '[
    "Belaruscha va o''zbekcha interfeys lokalizatsiyasi",
    "O''zbekiston cheklari va UZS valyutasini qo''llab-quvvatlash",
    "Xaritadagi agregatlar to''g''ri hisoblanadigan ko''p valyutali narxlar",
    "Til va valyuta uchun yangilangan profil sozlamalari",
    "Lokalizatsiya qilingan yangilanish xabari va versiyalar tarixi",
    "Ayrim foydalanuvchilarda yuz berishi mumkin bo''lgan nosozlik tuzatildi"
  ]'::jsonb,
  release_notes_uz = '[
    "Belaruscha va o''zbekcha interfeys lokalizatsiyasi qo''shildi.",
    "O''zbekiston cheklarining tanilishi va UZS valyutasi qo''llab-quvvatlandi.",
    "Chek va qo''lda kiritilgan narxlar asl valyutasini saqlaydi va bazaviy narxlar orqali analitikada qatnashadi.",
    "Til va valyuta sozlamalari profilda bir joyga yig''ildi.",
    "Yangilanish xabarlari va versiyalar tarixi joriy interfeys tiliga qarab matn tanlaydi.",
    "Ayrim foydalanuvchilarda yuz berishi mumkin bo''lgan nosozlik tuzatildi."
  ]'::jsonb,
  is_active = true,
  updated_at = now()
FROM release_row
WHERE platform = 'android';
```
