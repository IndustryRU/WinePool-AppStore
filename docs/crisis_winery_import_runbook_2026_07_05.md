# Импорт списка виноделен в кризисный pilot

Дата: 05.07.2026

## Зачем

У нас может быть список до 150 виноделен. Не нужно вручную переносить их в рабочую очередь. Сначала импортируем список в target tracker, затем выбираем тех, у кого есть быстрый контакт, дегустации и шанс принять группу.

## Формат файла

Использовать шаблон:

```powershell
docs\crisis_winery_import_template_2026_07_05.csv
```

Колонки:

- `name` - название для outreach.
- `organization` - юридическое/полное название, если отличается.
- `city` - Ялта, Бахчисарай, Севастополь, Балаклава, Судак и т.д.
- `source_url` - сайт, карточка, соцсеть или источник.
- `contact_channel` - `phone`, `email`, `telegram`, `site_form`, `phone_email`.
- `contact_value` - телефон, почта, Telegram или ссылка.
- `has_tasting` - `yes`, `no`, `unknown`.
- `accepts_groups` - `yes`, `no`, `unknown`.
- `scale` - `small`, `medium`, `large`, `unknown`.
- `priority` - можно оставить пустым; импортёр рассчитает сам.
- `notes` - что важно знать перед контактом.

## Команды

Сначала сухая проверка:

```powershell
pwsh -NoProfile -File scripts\import_winery_targets.ps1 -SourcePath path\to\wineries.csv -DryRun
```

Добавить только в target tracker:

```powershell
pwsh -NoProfile -File scripts\import_winery_targets.ps1 -SourcePath path\to\wineries.csv
```

Добавить также в operational outreach tracker:

```powershell
pwsh -NoProfile -File scripts\import_winery_targets.ps1 -SourcePath path\to\wineries.csv -AddToOutreach
```

Ограничить количество новых outreach-строк:

```powershell
pwsh -NoProfile -File scripts\import_winery_targets.ps1 -SourcePath path\to\wineries.csv -AddToOutreach -MaxOutreach 25
```

## Правило приоритета

Высокий приоритет:

- есть контакт;
- есть дегустации;
- принимает группы или это вероятно;
- малое или среднее хозяйство.

Средний приоритет:

- есть контакт и дегустации, но крупный объект;
- есть дегустации, но контакт слабый;
- есть контакт, но дегустации нужно подтвердить.

Низкий приоритет:

- нет контакта;
- нет признака дегустаций;
- нужен отдельный ресерч.

## Важно

Импорт не меняет статусы существующих строк. Новые строки создаются со статусом `new`. Панель и скрипты теперь показывают не только фиксированный список `fw`, но и новые импортированные строки после базового блока.
