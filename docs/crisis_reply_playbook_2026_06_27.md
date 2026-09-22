# WinePool crisis reply playbook

Дата: 27.06.2026  
Цель: быстро превращать ответы в деньги, заявки, интро или понятный следующий шаг.

## 1. Главное правило

Не спорить и не объяснять WinePool целиком. Каждый ответ должен вести к одному из трех действий:

- созвон/переписка по paid-work задаче;
- заявка туриста с датой, людьми и точкой старта;
- партнер, который готов быстро подтверждать заявки.

## 2. Если ответили по paid-work

### "Что можешь сделать?"

> Быстрее всего могу закрыть короткую задачу на 1-5 дней: форма заявок, лендинг, Flutter-экран, Supabase/Postgres/RLS, мини-админка, QR-воронка или интеграция. Лучше начать с того, что прямо влияет на заявки/оплаты. Что сейчас больше всего мешает?

Статус:

```powershell
pwsh -NoProfile -File scripts\update_outreach_tracker.ps1 -Id fw11 -Status replied -NotesAppend "спросил, что могу сделать"
```

### "Сколько стоит?"

> Для короткой задачи могу предложить понятный диапазон: 5000-15000 рублей за результат, если объем укладывается в 1-5 дней. Сначала быстро фиксируем задачу и критерий готовности, потом цену. Что нужно получить на выходе?

Статус:

```powershell
pwsh -NoProfile -File scripts\update_outreach_tracker.ps1 -Id fw11 -Status paid_work_lead -MoneySignal paid_work_lead -NotesAppend "спросил цену"
```

### "Есть задача, но позже"

> Ок, давай не потеряем. Я могу коротко посмотреть задачу сейчас и сказать, как ее лучше резать на маленький результат. Даже если делать позже, будет понятно, сколько это займет и сколько стоит.

Статус:

```powershell
pwsh -NoProfile -File scripts\update_outreach_tracker.ps1 -Id fw11 -Status replied -NotesAppend "задача позже, нужен follow-up"
```

## 3. Если ответил турист или знакомый

### "Нужен маршрут"

> Супер. Чтобы подобрать живой вариант, нужно 4 вещи: дата, сколько человек, откуда стартуете и нужен ли трансфер. Еще важно: хотите дегустацию, экскурсию по винодельне или спокойный маршрут на полдня?

Статус:

```powershell
pwsh -NoProfile -File scripts\update_outreach_tracker.ps1 -Id fw07 -Status money_signal -MoneySignal route_request -NotesAppend "есть запрос на маршрут"
```

### "Могу спросить знакомых"

> Спасибо, это сейчас очень полезно. Можно просто переслать ссылку: https://winepool.ru/routes-crimea. Нам важны реальные запросы: дата, люди, старт, транспорт. Мы вручную проверим актуальный вариант.

Статус:

```powershell
pwsh -NoProfile -File scripts\update_outreach_tracker.ps1 -Id fw07 -Status support_signal -MoneySignal intro -NotesAppend "готов спросить знакомых"
```

## 4. Если ответил партнер/оператор

### "Как это работает?"

> На пилоте без предоплаты: мы делаем страницу/QR/ссылку, принимаем запрос гостя и передаем живую заявку. Если заявка превращается в оплаченного гостя, обсуждаем небольшую оплату с результата. Если заявок нет, вы ничего не платите. Нам нужен человек, который быстро подтверждает дату, доступность и условия.

Статус:

```powershell
pwsh -NoProfile -File scripts\update_outreach_tracker.ps1 -Id fw03 -Status replied -NotesAppend "спросили как работает"
```

### "Куда отправлять заявки?"

> Отлично. Давайте зафиксируем рабочий контакт: имя, телефон/мессенджер, часы связи и какие маршруты/даты сейчас реально подтверждать. После этого можем принимать первые тестовые заявки.

Статус:

```powershell
pwsh -NoProfile -File scripts\update_outreach_tracker.ps1 -Id fw03 -Status pilot_candidate -MoneySignal success_fee_possible -NotesAppend "готовы принимать заявки"
```

### "У нас сейчас тяжело, денег нет"

> Понимаю. Поэтому мы и предлагаем без предоплаты. Мы тоже в Крыму и видим ситуацию. Наша логика простая: если WinePool не приводит гостя, вы ничего не платите. Если приводит, появляется честный разговор про оплату с результата.

Статус:

```powershell
pwsh -NoProfile -File scripts\update_outreach_tracker.ps1 -Id fw03 -Status replied -NotesAppend "денег нет, объяснен формат без предоплаты"
```

## 5. Если сказали "не сейчас"

> Понял. Тогда не дергаю. Если ситуация изменится или появятся гости/задача, вот ссылка: https://winepool.ru/routes-crimea. Я через пару недель аккуратно напомню, если будет уместно.

Статус:

```powershell
pwsh -NoProfile -File scripts\update_outreach_tracker.ps1 -Id fw03 -Status no_now -NotesAppend "не сейчас"
```

## 6. Вечерняя сводка

Посмотреть факты за день:

```powershell
pwsh -NoProfile -File scripts\summarize_crisis_outreach.ps1
```

Записать факты в daily metrics:

```powershell
pwsh -NoProfile -File scripts\summarize_crisis_outreach.ps1 -UpdateMetrics -NotesAppend "вечерняя фиксация"
```

Если появились деньги или подтвержденная сумма:

```powershell
pwsh -NoProfile -File scripts\summarize_crisis_outreach.ps1 -UpdateMetrics -PaidWorkRub 10000 -NotesAppend "есть paid-work задача на 10000"
```

## 7. Быстрое paid-work предложение

Если человек готов обсуждать задачу, не писать цену с нуля. Использовать один из пакетов:

```powershell
pwsh -NoProfile -File scripts\show_paid_work_offer.ps1 -Package audit
pwsh -NoProfile -File scripts\show_paid_work_offer.ps1 -Package task
pwsh -NoProfile -File scripts\show_paid_work_offer.ps1 -Package mvp
```

С проблемой и сроком:

```powershell
pwsh -NoProfile -File scripts\show_paid_work_offer.ps1 -Package task -Problem "лендинг и форма заявки" -Deadline "2 дня"
```

Подробная рамка пакетов: `docs/crisis_paid_work_offer_menu_2026_06_27.md`.
