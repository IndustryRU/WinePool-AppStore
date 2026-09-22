# H1 Tourism — monetization and public contact decision

Дата: 31.08.2026
Статус: accepted owner decision; уточняет active H1 source of truth

## Решение

До создания WinePool lead публичные tourism surfaces показывают identity и
репутацию партнёра, но не дают direct booking channel:

- показываются name, type, verified marker, approved story, city/region,
  logo/hero и current published tours;
- не возвращаются и не отображаются website, phone, email, messenger или иной
  способ обратиться к партнёру в обход lead form;
- единственный pre-lead CTA ведёт в общую consent-backed WinePool form;
- оператор получает контакт для обработки созданной заявки;
- турист получает прямой контакт только для исполнения уже зафиксированной
  заявки: после confirmation либо раньше в адресном operator message;
- раскрытие контакта не удаляет immutable attribution и не освобождает партнёра
  от согласованного outcome/financial reporting.

Полностью скрывать бренд нельзя и не нужно: название можно найти независимо, а
анонимная карточка без identity снижает доверие. WinePool убирает собственные
zero-friction outbound links и защищает результат contract, attribution,
qualification/outcome audit и partner reconciliation.

## Что было принято раньше

Исторические документы не переписываются; их решения сопоставлены ниже.

| Документ | Ранее зафиксированная идея | Совместимость с текущим решением |
|---|---|---|
| `crisis_outreach_90min_session_2026_06_27.md` | оплата за квалифицированную заявку/группу или состоявшегося клиента | полностью сохраняется |
| `crisis_cash_survival_runbook_2026_06_27.md` | первые 3 заявки бесплатно; затем 500–1500 RUB за состоявшуюся заявку, 5–10% либо фикс после N гостей | числа являются старой гипотезой, не принятой ставкой; first-N waived сохраняется |
| `crisis_revenue_strategy_2026_06_27.md` | бесплатный вход, success fee только после доказанного результата | сохраняется как одна из двух допустимых баз |
| `winery_partnership_crimea_yalta_context_2026_06_27.md` | пилот без предоплаты, затем оплата с измеримого результата | сохраняется |
| `crisis_winery_operator_model_2026_07_05.md` | оператор платит за qualified lead/completed client, винодельня — за qualified group/request/visit; первый pilot не усложнять разделением денег | полностью сохраняется; один lead получает одну согласованную basis |
| `hotel_partner_system_tz_2026_07_25.md` | гипотеза 5–10% success fee с оператора и возможная выплата referral-отелю | процент и channel payout остаются непроверенной гипотезой, не текущей ставкой |
| `post_release_1_1_0_execution_roadmap_2026_08_25.md` | success fee или пилотный фикс после доказанного результата | расширено явным qualified-lead вариантом |
| `h1_tourism_revenue_loop_tz_2026_08_30.md` до этого решения | основной текст был сужен до completed success fee, хотя terms поддерживали fixed/manual/per-guest/percent | устранено расхождение |

Старый `winery_partnership_pitch_2026_04_24.md` обещал прямые ссылки и отсутствие
комиссии для каталога вина/прямой продажи бутылок. Это другой продуктовый
контур, не Tourism lead acquisition, и он не разрешает direct booking links на
H1 tourism surfaces.

## Поддерживаемые H1 модели

H1 должен технически и документально поддержать обе базы, не выбирая ставку
заранее:

1. `qualified_lead`: фикс за проверенную заявку/группу;
2. `completed_visit`: fixed, per-guest или percent success fee за
   подтверждённый визит;
3. `manual/waived/not_applicable`: явный pilot/legal fallback.

Первые согласованные `N` результатов могут иметь `waived`. Старые ориентиры
500–1500 RUB и 5–10% не являются оффером, договором или текущей ценой.

## Qualified lead для будущего terms gate

Qualified lead — не любой submit. Минимальный candidate:

- external и `is_test=false`;
- current published tour;
- consent-backed;
- имя, допустимый contact channel, дата/период и guests count;
- уникальный submission attempt и отсутствие доказанного дубля/spam;
- назначенная организация получила заявку.

Qualification становится billable только по approved versioned terms и
audited actor/time/reason fact. Partner может предложить rejection только с
bounded reason; disputed/self-serving rejection не превращается автоматически
в `waived` и требует WinePool review.

## Что ещё не решено

- qualified-lead fee или completed-visit success fee для первого платного
  цикла;
- точное `N` waived результатов;
- ставка/процент и attribution window;
- legal/tax/invoice flow до `invoiced/paid`;
- customer confirmation и dispute evidence UX.

## Текущие implementation gaps

- Package A применён в production; anonymous exact-tour RPC удаляет legacy
  optional `operator.website_url` на чтении, не переписывая snapshot;
- native public operator/catalog consumers переключены на bounded RPC, поэтому
  `TourismOperatorScreen` не получает website/phone/messenger и его условная
  кнопка внешнего сайта не отображается;
- UUID-based lead/trip compatibility и home hero media переключены на отдельные
  bounded RPC;
- anonymous raw-table grants и public-read RLS membership отозваны для семи
  tourism tables. Authenticated admin/compatibility access сохранён отдельно.

Контактная граница закрыта без перестройки operator UI: изменён источник данных,
а существующий условный CTA естественно отсутствует при `websiteUrl == null`.
Anonymous raw-read security gate также закрыт. Более строгий будущий hardening
может убрать общий authenticated published-read compatibility, но он не входит
в этот anonymous closure и потребует отдельной проверки user/admin workflows.

Эти решения принимаются в H1-06 после первых clean external facts. До этого
ledger использует `pending/waived`, а публичный UI не обещает оплату внутри
WinePool.
