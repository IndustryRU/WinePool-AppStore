# H0 weekly operating report — 24–30.08.2026

Дата запуска: 30.08.2026. Источник: production server facts и
`supabase/reports/h0_weekly_operating_report.sql`. ПДн и пользовательские строки
в отчёт не выгружались.

## Consumer cohort

| Cohort | Actor | Registrations | Activation D0 / 7d | Strong 7d | Meaningful return | Qualifying actions | Sample |
|---|---|---:|---:|---:|---:|---:|---|
| 28.08.2026 | external | 1 | 1 / 1 | 0 | 0 | 2 | low sample |

D7/D30 retention ещё не созрел и остаётся `NULL`, а не нулём. Один внешний
пользователь активировался через server facts: дегустация и catalog
contribution. Выводы о conversion/retention по одному пользователю запрещены.

## Tourism attribution

| Segment | Leads | Contacted | Confirmed | Completed | No-show | Sample |
|---|---:|---:|---:|---:|---:|---|
| commercial | 0 | 0 | 0 | 0 | 0 | low sample |
| QA/test | 3 | 2 | 2 | 1 | 1 | low sample |

Одна QA-заявка остаётся `new` дольше 60 минут. Это ожидаемый тестовый хвост, но
он должен быть закрыт перед следующей SLA-сверкой. QA/test не попадает в
commercial KPI.

## Operating conclusion

- anomaly: первоначальный Sunday-запуск ошибочно выбрал будущий понедельник из
  shell-выражения `monday this week`; расчёт исправлен на PostgreSQL
  `date_trunc('week', current_date)` и повторный production-run воспроизвёл
  фактическую неделю;
- top funnel loss: коммерческого трафика и активных production referral-точек
  пока нет, поэтому conversion не оценивается;
- one product action: в H1 активировать подготовленные точки по одной после
  проверки ответственного, ссылки/QR и канала ответа;
- data quality: raw/clean и test/commercial разделяются, незрелые retention
  окна возвращают `NULL`, все наблюдаемые cohorts помечены `low_sample`.

