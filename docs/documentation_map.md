# Documentation Map

Последнее обновление: 13.09.2026

## Зачем Нужен Этот Файл

В `docs/` уже накопились несколько слоёв документов:

- живые source-of-truth по governance и implementation status
- фазовые roadmap-файлы
- feature-spec документы
- historical handoff/bootstrap файлы
- research-конспекты

Этот файл нужен как одна точка входа, чтобы быстро понимать:

- какой документ за что отвечает
- какой документ сейчас главный
- что уже исторический контекст, а не active source of truth
- где ещё остаются рассинхроны или doc-follow-ups

Если нужна текущая стратегия, приоритет H0–H6, gates и ближайший порядок после релиза 1.1.0, использовать **первым**:

- [post_release_1_1_0_execution_roadmap_2026_08_25.md](/R:/Flutter/Project/winepool_final/docs/post_release_1_1_0_execution_roadmap_2026_08_25.md) — active product and execution source of truth.

**Уточнение порядка от 12.09.2026.** Первый живой Tourism-пилот ведём через
винодельню как место для посещения: страница винодельни, её программы, заявки
и общий рабочий кабинет — один продуктовый пакет. Простая очередь, границы
первого пилота и следующий шаг зафиксированы в
[tourism_h1_compact_plan_2026_09_12.md](/R:/Flutter/Project/winepool_final/docs/tourism_h1_compact_plan_2026_09_12.md).
Этот документ уточняет порядок исполнения, но не отменяет исходные требования
к защите данных, заявкам и публичным страницам.

Если нужны исходные store-метрики и границы их интерпретации:

- [product_metrics_baseline_2026_08_25.md](/R:/Flutter/Project/winepool_final/docs/product_metrics_baseline_2026_08_25.md) — baseline App Store, RuStore и AppMetrica; содержит аудит событий/профилей, признаки загрязнения internal QA, ограничения crash/retention расчётов и список обязательных H0-поправок.
- [h0_tourism_web_first_lead_flow_tz_2026_08_26.md](/R:/Flutter/Project/winepool_final/docs/h0_tourism_web_first_lead_flow_tz_2026_08_26.md) — approved H0.5 contract: единая Flutter guest/auth form для browser/Android/iPhone, lightweight public web target, idempotent server create-lead contract, consent, success state и verified-link boundary.
- [h0_published_tour_contract_audit_2026_08_27.md](/R:/Flutter/Project/winepool_final/docs/h0_published_tour_contract_audit_2026_08_27.md) — implemented H0.5 contract-first gate: immutable public snapshot, CRM preview/publish, unified app/web adapter, production pilot normalization and smoke status.
- [h0_tourism_public_hosting_preview_runbook_2026_08_29.md](/R:/Flutter/Project/winepool_final/docs/h0_tourism_public_hosting_preview_runbook_2026_08_29.md) — Host-Food layout, reproducible slim Flutter build, scoped deploy/rollback и фактический production-like owner preview H0.5.
- [h0_weekly_operating_report_2026_08_24.md](/R:/Flutter/Project/winepool_final/docs/h0_weekly_operating_report_2026_08_24.md) — первый воспроизводимый production weekly baseline H0: external activation, test/commercial tourism split, low-sample и data-quality conclusion.
- [h1_tourism_revenue_loop_tz_2026_08_30.md](/R:/Flutter/Project/winepool_final/docs/h1_tourism_revenue_loop_tz_2026_08_30.md) — active implementation source of truth всего H1: public entry, lead operations/SLA, Trip Center и guest claim, outcome/revenue ledger, partner report, pilot launch kit, security, rollout и acceptance.
- [h1_tourism_revenue_loop_next_session_prompt_2026_08_31.md](/R:/Flutter/Project/winepool_final/docs/h1_tourism_revenue_loop_next_session_prompt_2026_08_31.md) — готовый handoff для H1-01: bounded public data audit, production-safe проверка, Package A contract, deliverables и запрет начинать новый UI без owner gate.
- [h1_tourism_public_operator_catalog_discovery_plan_2026_08_30.md](/R:/Flutter/Project/winepool_final/docs/h1_tourism_public_operator_catalog_discovery_plan_2026_08_30.md) — supporting plan H1.1b/H1.1c: operator page и curated catalog/basic search; advanced discovery вынесен в Tourism Growth backlog и не конфликтует с основным H2 Retail.

Если задача относится к текущему горизонту H0 — AppMetrica, clean/raw аудитории, activation/retention, first milestones, internal QA, weekly owner report или tourism referral attribution:

- [h0_measurement_activation_partner_attribution_tz_2026_08_25.md](/R:/Flutter/Project/winepool_final/docs/h0_measurement_activation_partner_attribution_tz_2026_08_25.md) — completed H0 implementation source and acceptance handoff: event/privacy contract, H0.0–H0.6, server facts, actor overrides, reports, QA и production closure.

Старые общие очереди сохранены только как исторический контекст:

- [prioritized_work_backlog.md](/R:/Flutter/Project/winepool_final/docs/prioritized_work_backlog.md) — historical/superseded с 25.08.2026;
- [next_iterations_execution_order.md](/R:/Flutter/Project/winepool_final/docs/next_iterations_execution_order.md) — historical/superseded с 25.08.2026.

Правило authority:

- post-1.1.0 roadmap определяет **когда и зачем** выполняется направление;
- feature-ТЗ определяет **как** его реализовать внутри разрешённого horizon;
- implementation status/handoff определяет **что фактически сделано**;
- research/transcript даёт идеи, но не является контрактом;
- release-план не меняет продуктовую приоритетность следующего этапа.

Если нужен исторический многоролевой срез продукта до релиза 1.1.0 (5 ролей и кросс-ролевая матрица):

- [expert_multi_role_analysis_2026_04_23.md](/R:/Flutter/Project/winepool_final/docs/expert_multi_role_analysis_2026_04_23.md) — стратегический research-input апреля; текущую очередь не определяет.

Если нужно вспомнить человеческую веху проекта после первой рабочей release APK-раздачи:

- [project_milestone_first_real_release_apk_2026_05_18.md](/R:/Flutter/Project/winepool_final/docs/project_milestone_first_real_release_apk_2026_05_18.md) — короткая запись о моменте, когда WinePool стал ощущаться не как прототип, а как живое приложение, которое хочется показывать людям.

Если задача про живых сомелье, экспертные профили, консультационные запросы, кабинет сомелье, прямые расчёты пользователя с экспертом вне WinePool или legal-safe модель экспертной монетизации:

- [public_wine_identity_profile_tz_2026_08_12.md](/R:/Flutter/Project/winepool_final/docs/public_wine_identity_profile_tz_2026_08_12.md) — source of truth для универсальной публичной винной визитки любого пользователя: preview по нажатию на шапку профиля, bio/роли/интересы, системный уровень и вклад, приватность, туристическая секция, квалификации, связь без дублирования с `expert_profiles`, schema/RLS/RPC, этапы и QA.
- [sommelier_experts_platform_tz_2026_06_19.md](/R:/Flutter/Project/winepool_final/docs/sommelier_experts_platform_tz_2026_06_19.md) — source of truth для слоя `Живые сомелье`: почему сомелье не становится отдельной auth-role, схема `expert_profiles`, запросы/сообщения/рейтинги, RLS, Flutter-модуль `features/experts`, прямые расчёты вне платформы на MVP, фазы внедрения и legal/tax guardrails.
- [sommelier_recruitment_pitch_2026_06_19.md](/R:/Flutter/Project/winepool_final/docs/sommelier_recruitment_pitch_2026_06_19.md) — рабочие тексты для привлечения профессиональных сомелье: короткое первое сообщение, основное письмо, лендинговый/PDF-блок, follow-up и акценты, которые важно не обещать.

Если задача про партнерскую `Винную ленту`, внешние медиа-каналы, Worker/sync, Storage-кэш изображений или блок последних публикаций на главной:

- [partner_messenger_feed_mvp_tz_2026_05_20.md](/R:/Flutter/Project/winepool_final/docs/partner_messenger_feed_mvp_tz_2026_05_20.md) — актуальный source of truth по реализованному MVP на 21.05.2026; включает runbook деплоя Cloudflare Worker через временный API token и `.env.cloudflare.local`.
- Реализация закрыта коммитами `46afb52`, `6871519`, `a13e0df`.
- Release QA по этой фиче зафиксирован в [mvp_release_readiness_2026_05_17.md](/R:/Flutter/Project/winepool_final/docs/mvp_release_readiness_2026_05_17.md).

Если задача про публичные отзывы пользователей, блок `Последние отзывы` на главной, модерацию отзывов или fullscreen-фото вина из карточки отзыва:

- [public_reviews_feed_mvp_tz_2026_05_25.md](/R:/Flutter/Project/winepool_final/docs/public_reviews_feed_mvp_tz_2026_05_25.md) — source of truth для следующего обновления WinePool в RuStore: MVP-срез, схема БД, RLS, админская модерация, карточка отзыва с фото вина и post-MVP развитие.

Если задача про карту конкретного вина, переход из отзыва по гео-иконке, `где покупали это вино`, личные/community-точки по одному `wine_id`, цены и винтажи на карте:

- [public_reviews_feed_mvp_tz_2026_05_25.md](/R:/Flutter/Project/winepool_final/docs/public_reviews_feed_mvp_tz_2026_05_25.md) — разделы `Контекст Покупки В Карточке Отзыва` и `Карта Конкретного Вина Из Карточки И Отзыва`.
- [community_map_detail_and_monetization_tz_2026_05_03.md](/R:/Flutter/Project/winepool_final/docs/community_map_detail_and_monetization_tz_2026_05_03.md) — раздел `Статус На 25.05.2026: Карта Конкретного Вина` и `Контекст Конкретного Вина`.

Если задача про винодельню как сущность — связь каталожной винодельни с туристическим кабинетом, публичная страница винодельни, вина и линейки на ней, владение карточкой:

- [winery_unified_model_2026_09_07.md](/R:/Flutter/Project/winepool_final/docs/winery_unified_model_2026_09_07.md) — **читать первым**: одна сведённая картина направления. Что решено тремя документами, что построено на бою с числами, четыре расхождения между решённым и построенным, целевая схема, порядок работ и открытые вопросы владельцу. Не отменяет источники, а показывает, какая их часть жива.
- [h1_02_public_operator_winery_profiles_tz_2026_08_31.md](/R:/Flutter/Project/winepool_final/docs/h1_02_public_operator_winery_profiles_tz_2026_08_31.md) — source of truth публичного профиля винодельни и его редактора; §8.2.1 фиксирует один тёмный editor shell, общий live preview, facts и восемь контролируемых атмосфер с макетами в репозитории; §8.2.2 — кликабельный прототип редактора (`design_assets/winery_profile_editor_2026_09_13/prototype/`), решения владельца 13.09.2026, токены пресетов, расхождения с ТЗ, сверка схемы и production-правило: сначала совместимая миграция, не перезаписывающая публичную обёртку бренда/хозяйства.
- [h1_02e_reviews_visits_and_visit_card_tz_2026_09_21.md](/R:/Flutter/Project/winepool_final/docs/h1_02e_reviews_visits_and_visit_card_tz_2026_09_21.md) — карточка «Готовы приехать?» вместо финального баннера, подборка отзывов о винах (выбор — выделение, не сокрытие), отзывы о посещениях: один вход после визита по токену заявки «Завершена», три витрины; факты оператора отложены до его редактора.
- [winepool_winery_closed_demo_tz_2026_09_12.md](/R:/Flutter/Project/winepool_final/docs/winepool_winery_closed_demo_tz_2026_09_12.md) — принятое описание закрытого образца WinePool Winery: что показываем потенциальному партнёру, как различаются собственные и операторские предложения, как защищён доступ и как образец превращается в живую страницу.
- [test_accounts.md](/R:/Flutter/Project/winepool_final/docs/test_accounts.md) — тестовые учётные записи и пароли для проверки Tourism CRM: четыре роли пилотной организации, владелец «Едувялту», учётная запись каталога. Точка правды по доступам.

Если задача про винодельни как публичный post-MVP слой, винный туризм, экскурсии, дегустации, расписания, заявки/бронирование, маршруты до виноделен, кабинет винодельни/туроператора или pilot в Ялте:

- [h1_tourism_revenue_loop_tz_2026_08_30.md](/R:/Flutter/Project/winepool_final/docs/h1_tourism_revenue_loop_tz_2026_08_30.md) — читать первым для всего H1 Tourism Revenue Loop.
- [h1_tourism_revenue_loop_next_session_prompt_2026_08_31.md](/R:/Flutter/Project/winepool_final/docs/h1_tourism_revenue_loop_next_session_prompt_2026_08_31.md) — использовать для старта следующей сессии с H1-01 без повторного чтения старых Tourism-документов.
- [h1_tourism_public_operator_catalog_discovery_plan_2026_08_30.md](/R:/Flutter/Project/winepool_final/docs/h1_tourism_public_operator_catalog_discovery_plan_2026_08_30.md) — supporting plan H1.1b/H1.1c; не использовать как разрешение менять web UI без совместного visual acceptance.
- [post_release_1_1_0_execution_roadmap_2026_08_25.md](/R:/Flutter/Project/winepool_final/docs/post_release_1_1_0_execution_roadmap_2026_08_25.md) — читать первым: H0 measurement/pilot readiness, затем H1 Tourism Revenue Loop и H3 Winery Unified Value Center.
- [tourism_release_1_0_5_tz_2026_06_06.md](/R:/Flutter/Project/winepool_final/docs/tourism_release_1_0_5_tz_2026_06_06.md) — релизное ТЗ WinePool Tourism 1.0.5: P0/P1 scope, `Центр поездки`, страница тура, QR/launch kit, QA, RuStore packaging и acceptance criteria.
- [tourism_release_1_0_5_next_session_prompt_2026_06_06.md](/R:/Flutter/Project/winepool_final/docs/tourism_release_1_0_5_next_session_prompt_2026_06_06.md) — подробный prompt/handoff для следующей Codex-сессии по реализации Tourism 1.0.5.
- [winery_tourism_platform_tz_2026_06_01.md](/R:/Flutter/Project/winepool_final/docs/winery_tourism_platform_tz_2026_06_01.md) — целевой source of truth для направления `winery tourism`: продуктовая рамка, юридические guardrails, роли, БД, RLS, Flutter-модули, admin/business surfaces, карта/маршруты, уведомления, аналитика, phases и recommended first slice.
- [tourism_next_work_plan_2026_06_06.md](/R:/Flutter/Project/winepool_final/docs/tourism_next_work_plan_2026_06_06.md) — historical feature backlog после редизайна `/tourism`; использовать только пункты, включённые в H0/H1.
- [hotel_partner_system_tz_2026_07_25.md](/R:/Flutter/Project/winepool_final/docs/hotel_partner_system_tz_2026_07_25.md) — historical research transcript, не ТЗ; отели рассматриваются как referral channel H1, hotel-specific schema проектируется только после pilot audit.

Если задача про универсальный flow `Добавить бутылку` — добавление вина в погребок без чека (штрихкод/этикетка/название), manual drafts, мгновенную видимость бутылки `На проверке` в погребке, rate limits manual intake или замену старого `/wine-label-ocr`:

- [catalog_search_copilot_four_session_execution_plan_2026_07_17.md](/R:/Flutter/Project/winepool_final/docs/catalog_search_copilot_four_session_execution_plan_2026_07_17.md) — historical feature execution plan: ранние сессии дали baseline релиза 1.1.0; оставшийся массовый AI fallback подчинён H6 и не является текущей общей очередью.
- [catalog_fast_search_session2_handoff_2026_07_25.md](/R:/Flutter/Project/winepool_final/docs/catalog_fast_search_session2_handoff_2026_07_25.md) — итоговый handoff функционально закрытой Session 2: контракты, миграции/backups, QA, production deploy и оставшийся versioned release record.
- [catalog_fast_search_session2_qa_matrix_2026_07_25.md](/R:/Flutter/Project/winepool_final/docs/catalog_fast_search_session2_qa_matrix_2026_07_25.md) — полностью пройденная ручная/автоматическая приёмочная матрица текста, GTIN, этикетки, add-bottle, гостя, идемпотентности и модерации.
- [release_experience_and_wine_card_roadmap_2026_07_24.md](/R:/Flutter/Project/winepool_final/docs/release_experience_and_wine_card_roadmap_2026_07_24.md) — historical roadmap релизного контура 1.1.0; сохраняется как архитектурный контекст карточки и offers→release/SKU.
- [session_2_5_release_experience_start_prompt_2026_07_29.md](/R:/Flutter/Project/winepool_final/docs/session_2_5_release_experience_start_prompt_2026_07_29.md) — подробный стартовый промпт отдельной Session 2.5: аудит, политика фотографий, релизы в поиске, карточка вина, QA и handoff.
- [session_2_5_wine_card_visual_master_2026_07_30.md](/R:/Flutter/Project/winepool_final/docs/session_2_5_wine_card_visual_master_2026_07_30.md) — актуальный визуальный эталон полной прокрутки карточки после аудита редакции 3: hero, релизы, факты, награды, особенности, Атлас, вкус 0–5, магазины, винодельня и отзывы.
- [session_2_5_wine_card_flutter_implementation_plan_2026_07_30.md](/R:/Flutter/Project/winepool_final/docs/session_2_5_wine_card_flutter_implementation_plan_2026_07_30.md) — поэтапный план Flutter-реализации карточки: release read model, декомпозиция монолита, runtime icons, responsive/accessibility, QA, границы Session 2.6 и commit checkpoints.
- [session_2_5_cellar_write_path_audit_2026_08_01.md](/R:/Flutter/Project/winepool_final/docs/session_2_5_cellar_write_path_audit_2026_08_01.md) — аудит всех путей записи в погребок: релизы, ручное добавление, чеки, заказы, idempotency, provenance и production-исправление фантомного дубля.
- [session_2_5_wine_knowledge_moderation_atlas_tz_2026_08_01.md](/R:/Flutter/Project/winepool_final/docs/session_2_5_wine_knowledge_moderation_atlas_tz_2026_08_01.md) — подробное ТЗ следующего среза наполнения карточки: AI research proposals, модераторское подтверждение и safe bulk apply, Atlas taxonomy/aliases, wine/release assignments, aromas/pairings, structured awards, audit, RLS, dual-read/dual-write, backfill и production QA.
- [session_2_6_release_aware_offers_tz_2026_08_07.md](/R:/Flutter/Project/winepool_final/docs/session_2_6_release_aware_offers_tz_2026_08_07.md) — **source of truth Session 2.6**: аудит живой базы (offers пуст — 2 тестовые строки, 0 заказов, 0 точек), целевая модель `wine_vintages → wine_skus → offers → locations`, обязательные выпуск и точка, разделение вывески и юрлица, сопоставление чековых адресов с точками по ИНН, расстояние до точки, закрытие дыры в RLS. Заменяет стартовый промпт в части модели и стратегии миграции.
- [session_2_6_release_aware_offers_start_prompt_2026_07_29.md](/R:/Flutter/Project/winepool_final/docs/session_2_6_release_aware_offers_start_prompt_2026_07_29.md) — исторический стартовый промпт Session 2.6. Его блок C (additive-миграция, backfill, legacy fallback, compatibility period) **не выполняется**: аудит 07.08.2026 показал, что мигрировать нечего. Читать вместе с ТЗ выше, а не вместо него.
- [atlas_knowledge_base_consolidated_analysis_2026_08_06.md](/R:/Flutter/Project/winepool_final/docs/atlas_knowledge_base_consolidated_analysis_2026_08_06.md) — сводный анализ Atlas как базы знаний: история замысла с апреля 2026, фактическое состояние (6 статей в коде, 130 терминов без статей), разрывы, коммерческий слой и решения владельца.
- [session_atlas_publishing_tz_2026_08_06.md](/R:/Flutter/Project/winepool_final/docs/session_atlas_publishing_tz_2026_08_06.md) — feature source of truth для публикационного контура (`content_pages` + секции), но не общий execution-order; работы выполняются в H3/H5 либо раньше узким срезом, если они блокируют H1 public/QR flow.
- [cellar_release_workflow_1_1_0_tz_2026_08_10.md](/R:/Flutter/Project/winepool_final/docs/cellar_release_workflow_1_1_0_tz_2026_08_10.md) — компактное релизное ТЗ для работы с релизами/NV в «Храню» и «Пробовал» до публикации 1.1.0.
- [cellar_filters_tz_2026_08_10.md](/R:/Flutter/Project/winepool_final/docs/cellar_filters_tz_2026_08_10.md) — feature-ТЗ фильтров погребка; текущая очередь подчинена gate H4 Cellar Pro, production defects исправляются независимо.
- [cellar_release_workflow_1_1_0_tz_2026_08_10.md](/R:/Flutter/Project/winepool_final/docs/cellar_release_workflow_1_1_0_tz_2026_08_10.md) — компактное релизное ТЗ работы с винтажем и релизом в погребке 1.1.0 с сохранением необязательного предложения нового релиза.
- [my_wine_shops_and_brand_proposals_1_1_0.md](/R:/Flutter/Project/winepool_final/docs/my_wine_shops_and_brand_proposals_1_1_0.md) — итоговый контракт «Моих винных магазинов», группировки покупок, маршрутов, обновления расстояний и пользовательских предложений брендов с модерацией и XP.
- [add_bottle_universal_flow_tz_2026_06_12.md](/R:/Flutter/Project/winepool_final/docs/add_bottle_universal_flow_tz_2026_06_12.md) — source of truth: продуктовая рамка, UX всех путей, миграции manual sources, расширение автодобавления `20260527`, изменения workbench, guest mode, метрики, критерии приёмки, QA-план и промпты реализации по проходам 0-5.
- [user_ai_wine_discovery_and_catalog_contribution_tz_2026_07_14.md](/R:/Flutter/Project/winepool_final/docs/user_ai_wine_discovery_and_catalog_contribution_tz_2026_07_14.md) — feature-ТЗ пользовательского AI-исследования; массовый rollout отложен в H6, точечные исправления существующего internal/admin research не блокируются.
- [label_ocr_recovery_notes_2026_06_14.md](/R:/Flutter/Project/winepool_final/docs/label_ocr_recovery_notes_2026_06_14.md) — что починено в распознавании этикеток (винодельня как строгий scope, Yandex-only OCR, фикс «Крым»→название) и оставшиеся риски. Baseline в коммите `6de0323`.
- [label_ocr_polish_next_session_prompt_2026_06_14.md](/R:/Flutter/Project/winepool_final/docs/label_ocr_polish_next_session_prompt_2026_06_14.md) — мастер-промпт и повестка для отдельной сессии по полировке распознавания этикеток и ручного добавления (чистка дублей винодельни ZB, label-title extractor, UX, память этикеток).

Если задача про качество каталога — объединение дублей вин, линейки вин (серии внутри винодельни), коллекции/ачивки или распознавание полей этикетки (винодельня/название/винтаж/линейка):

- [wine_merge_admin_implementation_2026_06_14.md](/R:/Flutter/Project/winepool_final/docs/wine_merge_admin_implementation_2026_06_14.md) — объединение дублей вин в админке (RPC + обратимый откат + UI), миграция `20260614_add_wine_merge_schema_and_rpc.sql`. Применено на live.
- [wine_lines_tz_2026_06_14.md](/R:/Flutter/Project/winepool_final/docs/wine_lines_tz_2026_06_14.md) — ТЗ линеек вин (Этап 1): модель `wine_lines + wines.line_id + wine_line_aliases`, пикер в редакторе, фильтр каталога, управление в карточке винодельни. Бэклог: визуальное выделение + уровень качества (Reserve/Grand Reserve как отдельный атрибут). Применено на live.
- [production_winery_public_page_handoff_2026_09_13.md](/R:/Flutter/Project/winepool_final/docs/handoffs/production_winery_public_page_handoff_2026_09_13.md) — передача Codex среза 3 ТЗ хозяйства-производителя: контракт `get_winery_production_links`, подключение к `get_public_winery_page_v1`, ограничения совместимости с 1.1.0, проверка, границы файлов.
- [push_notifications_guide.md](/R:/Flutter/Project/winepool_final/docs/push_notifications_guide.md) — **как добавить новое push-уведомление**: цепочка очередь → отправщик → универсальный API RuStore (RuStore + FCM) → нативный показ, контракт полей `data` (`type`, `title`, `body`, `link`, `channel`, `image_url`), когда нужна новая версия приложения, типы и примеры («Как прошёл вечер?», отзыв о вине всем).
- [push_notifications_handoff_2026_09_21.md](/R:/Flutter/Project/winepool_final/docs/handoffs/push_notifications_handoff_2026_09_21.md) — состояние push: этап 0 (Android) закрыт, FCM вторым каналом, результаты проверок на телефоне, план этапов 1–3.
- [wine_production_winery_tz_2026_09_13.md](/R:/Flutter/Project/winepool_final/docs/wine_production_winery_tz_2026_09_13.md) — ТЗ хозяйства-производителя у вина (13.09.2026): бренд остаётся винодельней вина, хозяйство — необязательная ссылка в `wine_production_partners`; перенос вина между винодельнями, объединение дублей, блоки «Хозяйства-производители» / «Производит для брендов», подсказка из исследования AI. Три среза; первый шаг — проверка PostgREST на совместимость с 1.1.0. Опирается на мультирегиональные винодельни (`wineries.geography_scope`, применено 13.09.2026).
- [catalog_names_aliases_winery_locations_tz_2026_07_12.md](/R:/Flutter/Project/winepool_final/docs/catalog_names_aliases_winery_locations_tz_2026_07_12.md) — связанное ТЗ по canonical/localized names, unified alias graph (включая wine lines), computed wine display titles и многоточечной географии winery для wiki/tourism без автоматической публикации legal/production адресов.
- [wine_production_attributes_certifications_tz_2026_07_12.md](/R:/Flutter/Project/winepool_final/docs/wine_production_attributes_certifications_tz_2026_07_12.md) — additive-рефакторинг Pet-Nat/organic/biodynamic/natural в методы, подходы, сертификации и особенности продукта (Demeter, Vegan) с legacy compatibility, фильтрами и UI badges.
- [wine_collections_concept_2026_06_14.md](/R:/Flutter/Project/winepool_final/docs/wine_collections_concept_2026_06_14.md) — концепт коллекций/ачивок (Этап 2, геймификация поверх линеек). Линейка ≠ коллекция. Отдельное ТЗ позже.
- [label_field_extraction_2026_06_14.md](/R:/Flutter/Project/winepool_final/docs/label_field_extraction_2026_06_14.md) — разбор этикетки по полям (винодельня/название/винтаж/тип/ABV, геометрия Yandex, лексикон линеек). Бэклог: поле ABV в форме черновика.
- [label_line_ocr_tz_2026_06_15.md](/R:/Flutter/Project/winepool_final/docs/label_line_ocr_tz_2026_06_15.md) — ТЗ + стартовый промпт следующей сессии: довести распознанную линейку до конца в потоке добавления по фото (prefill → форма → миграция `draft_wines` → модерация → `wines.line_id`), линейка как сигнал матчинга.

Если задача про `WinePool Cellar Pro` — учет личного запаса вина, приватные зоны хранения без координат, расположение бутылок внутри погребка, журнал движения, экспорт, платный Pro-слой погребка, QR/NFC-метки или tablet/desktop-инвентаризацию:

- [post_release_1_1_0_execution_roadmap_2026_08_25.md](/R:/Flutter/Project/winepool_final/docs/post_release_1_1_0_execution_roadmap_2026_08_25.md) — сначала проверить входной gate H4: активное ядро, размер коллекций, repeat use и готовность платить.
- [wine_cellar_pro_tz_2026_06_23.md](/R:/Flutter/Project/winepool_final/docs/wine_cellar_pro_tz_2026_06_23.md) — feature source of truth для будущей реализации: продуктовая рамка, UX, SQL-модель `user_cellars / cellar_locations / cellar_stock_lots / cellar_inventory_events`, RPC, entitlements/paywall, privacy, QA и этапы. Не является текущей очередью до прохождения H4 gate.
- Связанные входы: [add_bottle_universal_flow_tz_2026_06_12.md](/R:/Flutter/Project/winepool_final/docs/add_bottle_universal_flow_tz_2026_06_12.md) для попадания бутылки в погребок, [receipt_sprint_status.md](/R:/Flutter/Project/winepool_final/docs/receipt_sprint_status.md) для чекового ввода, [map_experience_roadmap_2026_04_25.md](/R:/Flutter/Project/winepool_final/docs/map_experience_roadmap_2026_04_25.md) для разделения `где куплено` и приватного `где лежит`, [seo_aso_growth_plan_2026_05_23.md](/R:/Flutter/Project/winepool_final/docs/seo_aso_growth_plan_2026_05_23.md) для поискового спроса.

Если задача про «Мои места» — личную базу магазинов пользователя из его чеков, переименование юр. названия в вывеску (Пятёрочка, Красное & Белое), быстрый выбор места при ручном добавлении вина:

- [my_places_user_shops_tz_2026_06_13.md](/R:/Flutter/Project/winepool_final/docs/my_places_user_shops_tz_2026_06_13.md) — ТЗ (реализация отложена; лёгкий пикер магазинов из чеков уже сделан в ручном добавлении). Полный раздел с переименованием и единым применением имени — отдельный слайс.

Если задача про массовый импорт каталога — новый крупный источник данных (Роскачество, фид ритейлера/импортёра, партнёрский каталог), staging-импорт через normalization-слой, замену старого CSV-импорта или судьбу PARSER_*-доков:

- [batch_catalog_import_staging_concept_2026_06_12.md](/R:/Flutter/Project/winepool_final/docs/batch_catalog_import_staging_concept_2026_06_12.md) — зафиксированное направление (НЕ активная задача): триггеры возврата, целевая архитектура «модерация пачкой» поверх alias/normalization-конвейера, неторгуемые принципы качества, открытые вопросы и промпт для будущей сессии. Полное ТЗ писать только при наступлении триггеров из раздела 2.

Если задача про **локализацию (l10n)** — Flutter ARB/gen-l10n, добавление новых ключей, правила локализации строк, новые языки, locale-aware changelog, структуру профиля, архитектуру валюты:

- [l10n_localization_architecture_2026_06_21.md](/R:/Flutter/Project/winepool_final/docs/l10n_localization_architecture_2026_06_21.md) — source of truth по локализации: стек, правила написания кода (async/sync/top-level), что локализовано (статус на 22.06.2026), подключённые локали `ru/en/be/uz`, что НЕЛЬЗЯ локализовывать, locale-aware changelog (миграции Supabase + модель + UI), пикер валюты, структура меню профиля.

Если задача про сайт/лендинг winepool.ru — структура, файлы, что куда деплоить, формы, 152-ФЗ, cookie, внешние ресурсы и pending-задачи перед публикацией:

- [hosting_host_food_runbook_2026_08_13.md](/R:/Flutter/Project/winepool_final/docs/hosting_host_food_runbook_2026_08_13.md) — **читать первым**: актуальный источник истины по Host-Food, разделение FTP сайта и VPS, проверенные DNS/IP, безопасная проверка и точечный деплой. Timeweb не является текущим хостингом.
- [landing/index.html](/R:/Flutter/Project/winepool_final/landing/index.html) — готовый лендинг (dark premium, age gate, phone mockup с реальным скриншотом)
- [landing/styles.css](/R:/Flutter/Project/winepool_final/landing/styles.css) — полная CSS-тема
- [landing/script.js](/R:/Flutter/Project/winepool_final/landing/script.js) — JS: sticky nav, FAQ, email form, age gate, scroll-reveal
- [landing_152fz_compliance_checklist_2026_07_15.md](/R:/Flutter/Project/winepool_final/docs/landing_152fz_compliance_checklist_2026_07_15.md) — обязательный чеклист перед добавлением страниц/форм: 152-ФЗ, согласия, cookie, Метрика после согласия, локальные шрифты, запрет внешних CDN/трекеров без анализа, РКН и постдеплойная проверка.
- [seo_aso_growth_plan_2026_05_23.md](/R:/Flutter/Project/winepool_final/docs/seo_aso_growth_plan_2026_05_23.md) — SEO/ASO-план после публикации в RuStore: Wordstat-кластеры, посадочные страницы, правила текстов и метрики.

Деплой: содержимое `landing/` → FTP-каталог `/www/winepool.ru` на **Host-Food**. Не использовать Timeweb и не смешивать FTP сайта с VPS-контуром. Перед работой читать [hosting_host_food_runbook_2026_08_13.md](/R:/Flutter/Project/winepool_final/docs/hosting_host_food_runbook_2026_08_13.md), а перед деплоем новых форм обязательно сверить [landing_152fz_compliance_checklist_2026_07_15.md](/R:/Flutter/Project/winepool_final/docs/landing_152fz_compliance_checklist_2026_07_15.md). Не подключать Google Fonts/GA/GTM/reCAPTCHA/внешние CDN без отдельного анализа трансграничной передачи и обновления документов.

Если нужно вернуться к полному функциональному коду до MVP-среза (cart, checkout, seller, admin в полном виде) — использовать git-тег `pre-mvp-cut` (24.04.2026). Команды и описание что внутри: раздел «Git-тег полного базлайна» в [mvp_launch_execution_plan_2026_04_23.md](/R:/Flutter/Project/winepool_final/docs/mvp_launch_execution_plan_2026_04_23.md).

Если задача про исторический MVP-релиз на рынок РФ, старые release candidates, pitch или исходные юридические решения:

- [next_rustore_update_changelog_2026_05_25.md](/R:/Flutter/Project/winepool_final/docs/next_rustore_update_changelog_2026_05_25.md) — historical changelog до уже опубликованного 1.1.0; не использовать как живой список следующего релиза.
- [rustore_next_release_after_1_0_5_2026_06_22.md](/R:/Flutter/Project/winepool_final/docs/rustore_next_release_after_1_0_5_2026_06_22.md) — historical pre-RC memo 1.0.6; не использовать для новой сборки.
- [next_rustore_update_feedback_plan_2026_05_27.md](/R:/Flutter/Project/winepool_final/docs/next_rustore_update_feedback_plan_2026_05_27.md) — рабочий triage-план после первых пользователей опубликованного приложения: onboarding без чека, автодобавление одобренного draft-вина в погребок, исправление user email-ссылки, обработка camera permission errors, P1/P2 идеи.
- [mvp_release_readiness_2026_05_17.md](/R:/Flutter/Project/winepool_final/docs/mvp_release_readiness_2026_05_17.md) — historical readiness и фактическая история первого RuStore-релиза
- [mvp_week_5_rustore_beta_runbook_2026_05_21.md](/R:/Flutter/Project/winepool_final/docs/mvp_week_5_rustore_beta_runbook_2026_05_21.md) — операционный Week 5 runbook: beta gates, feedback intake, P0/P1 triage, RuStore submission checklist и план по датам
- [mvp_week_5_beta_materials_2026_05_21.md](/R:/Flutter/Project/winepool_final/docs/mvp_week_5_beta_materials_2026_05_21.md) — готовые материалы closed beta: feedback form, tester list, invite/reminder тексты и daily triage
- [android_release_workflow_2026_05_21.md](/R:/Flutter/Project/winepool_final/docs/android_release_workflow_2026_05_21.md) — рабочий протокол Android APK-релизов: version bump, signed APK, VPS download link, `app_release_config`, ветки/теги и handoff для новой сессии
- [local_docker_postgrest_testing.md](/R:/Flutter/Project/winepool_final/docs/local_docker_postgrest_testing.md) — как проверить поведение PostgREST локально в Docker (новые связи таблиц, embed'ы, совместимость со старыми сборками): запуск Docker Desktop, скрипт-образец в `tool/`, чтение ответов, журнал проведённых проверок. На VPS такие проверки не проводятся.
- [powershell_ssh_sql_quoting_guide.md](/R:/Flutter/Project/winepool_final/docs/powershell_ssh_sql_quoting_guide.md) — памятка по кавычкам для PowerShell → SSH → Docker → psql; использовать перед любыми SQL-командами на VPS
- [mvp_launch_strategy_2026_04_23.md](/R:/Flutter/Project/winepool_final/docs/mvp_launch_strategy_2026_04_23.md)
- [mvp_launch_execution_plan_2026_04_23.md](/R:/Flutter/Project/winepool_final/docs/mvp_launch_execution_plan_2026_04_23.md)
- [winery_partnership_pitch_2026_04_24.md](/R:/Flutter/Project/winepool_final/docs/winery_partnership_pitch_2026_04_24.md)
- [legal_privacy_policy_draft_2026_04_24.md](/R:/Flutter/Project/winepool_final/docs/legal_privacy_policy_draft_2026_04_24.md)

## Общая Картина

### Текущая картина после 1.1.0

Общая очередь теперь одна:

1. H0 — measurement, activation, partner attribution и pilot readiness;
2. H1 — Tourism Revenue Loop;
3. H2 — Retail Availability and First Store;
4. H3 — Winery Unified Value Center;
5. H4 — Cellar Pro после входного product gate;
6. H5 — trust/content/growth scale;
7. H6 — отложенное расширение платформы.

Подробности и критерии перехода: [post_release_1_1_0_execution_roadmap_2026_08_25.md](/R:/Flutter/Project/winepool_final/docs/post_release_1_1_0_execution_roadmap_2026_08_25.md).

### Историческая тематическая карта

Сейчас папка `docs` описывает несколько больших линий работы:

1. `role/business/canonical governance`
2. `receipt / QR / draft / price flows`
3. `canonical winery alias / merge governance`
4. `partner media feed / live content`
5. `cellar pro / personal inventory / paid retention`

Эти линии уже больше не стоит читать как один линейный спринт.

Правильнее так:

- role/business документы задают platform governance baseline
- receipt документы задают consumer-flow implementation reality и buyer/admin split
- alias/merge документы задают отдельный canonical catalog governance slice
- partner media feed документ задает live-content слой для retention и партнерских каналов
- cellar pro документ задает post-MVP платный слой поверх уже работающих `add-bottle`, receipt, cellar и map данных

## Главные Source Of Truth По Темам

### 1. Role / Business / Governance

Целевая модель:

- [role_business_architecture_spec.md](/R:/Flutter/Project/winepool_final/docs/role_business_architecture_spec.md)

Живой roadmap по фазам:

- [role_business_architecture_implementation_plan.md](/R:/Flutter/Project/winepool_final/docs/role_business_architecture_implementation_plan.md)

Практическая матрица write surfaces:

- [role_business_write_surface_inventory.md](/R:/Flutter/Project/winepool_final/docs/role_business_write_surface_inventory.md)

Policy vocabulary / enforcement contract:

- [role_business_policy_draft.md](/R:/Flutter/Project/winepool_final/docs/role_business_policy_draft.md)

### 2. Receipt / QR / Draft / Price

Governance truth для buyer/admin границ:

- [receipt_buyer_admin_governance_spec.md](/R:/Flutter/Project/winepool_final/docs/receipt_buyer_admin_governance_spec.md)

Current moderation status-board:

- [receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md](/R:/Flutter/Project/winepool_final/docs/receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md)

Фактическое состояние receipt sprint и миграций:

- [receipt_sprint_status.md](/R:/Flutter/Project/winepool_final/docs/receipt_sprint_status.md)

Receipt price/geolocation/map MVP layer (25.04.2026) зафиксирован в [receipt_sprint_status.md](/R:/Flutter/Project/winepool_final/docs/receipt_sprint_status.md), [mvp_launch_execution_plan_2026_04_23.md](/R:/Flutter/Project/winepool_final/docs/mvp_launch_execution_plan_2026_04_23.md) и [map_experience_roadmap_2026_04_25.md](/R:/Flutter/Project/winepool_final/docs/map_experience_roadmap_2026_04_25.md): receipt-based средняя цена, freshness, `geocode-receipt`, Yandex MapKit preview, карта покупки, ручное `Определить` для старых чеков, `/my-wine-map`, drill-down `Вина из магазина`, связи `В каталоге`/`Черновик` и fullscreen-фото. Этот слой считается закрытым для MVP кроме QA/bugfix.

Roadmap map-experience слоя (Phase 1/1.1 closed for MVP, Phase 2+ post-MVP):

- [map_experience_roadmap_2026_04_25.md](/R:/Flutter/Project/winepool_final/docs/map_experience_roadmap_2026_04_25.md)

MVP-слой личного вклада и лёгкой геймификации (09.05.2026): профильная карточка `Вклад в каталог`, журнал `user_experience_events`, идемпотентное начисление опыта за чек и принятые черновики, ручной share наружу без внутренней социальной ленты:

- [mvp_catalog_contribution_gamification_2026_05_09.md](/R:/Flutter/Project/winepool_final/docs/mvp_catalog_contribution_gamification_2026_05_09.md)

Текущий follow-up backlog:

- [receipt_follow_up_backlog.md](/R:/Flutter/Project/winepool_final/docs/receipt_follow_up_backlog.md)

Calibration / bug bash:

- [receipt_bug_bash_calibration_checklist.md](/R:/Flutter/Project/winepool_final/docs/receipt_bug_bash_calibration_checklist.md)

Grape-detection status/spec:

- [receipt_grape_detection_spec.md](/R:/Flutter/Project/winepool_final/docs/receipt_grape_detection_spec.md)

Draft UX/implementation spec:

- [draft_wine_flow_spec.md](/R:/Flutter/Project/winepool_final/docs/draft_wine_flow_spec.md)

Active Track B redesign spec:

- [draft_wine_details_redesign_tz_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/draft_wine_details_redesign_tz_2026_04_21.md)

Track B implementation companion:

- [track_b_draft_details_evidence_implementation_plan_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/track_b_draft_details_evidence_implementation_plan_2026_04_21.md)

Operational admin guide for draft catalog moderation:

- [administrator_guide.md](/R:/Flutter/Project/winepool_final/docs/administrator_guide.md)

Минимальный рабочий стол модератора каталога после Share Loop / QA:

- [moderator_catalog_workbench_tz_2026_05_03.md](/R:/Flutter/Project/winepool_final/docs/moderator_catalog_workbench_tz_2026_05_03.md)

In-App + email уведомления для пользовательско-модераторского контура (развёрнуто 07.05.2026):

- [moderation_notifications_tz_2026_05_07.md](/R:/Flutter/Project/winepool_final/docs/moderation_notifications_tz_2026_05_07.md)

Архитектура: `app_notifications` таблица → Supabase Realtime для In-App → systemd timer `winepool-notifs` на VPS → `smtp.yandex.ru` для email. Конфиг: `/opt/winepool-notifs/.env`. Admin-письма → `support@winepool.ru`.

Рабочий промпт для следующего полного QA-прохода по QR → чек → черновик → модерация → каталог:

- [moderation_e2e_qa_prompt_2026_05_08.md](/R:/Flutter/Project/winepool_final/docs/moderation_e2e_qa_prompt_2026_05_08.md)

Веб-панель администратора (развёрнута 07.05.2026):

- URL: `https://admin.winepool.ru` (basic auth: `winepool` / см. `.htpasswd-admin` на VPS)
- Flutter Web build → `/var/www/admin-winepool/web/` на VPS
- nginx конфиг: `/etc/nginx/sites-available/admin-winepool`
- SSL: Let's Encrypt (`certbot`), автообновление активно
- Обновление: `flutter build web --release` → `scp build/web root@91.227.18.214:/var/www/admin-winepool/`

Session bootstrap for the next draft moderation work cycles:

- [draft_wine_session_bootstrap.md](/R:/Flutter/Project/winepool_final/docs/draft_wine_session_bootstrap.md)

Next governance layer for moderation abuse-controls:

- [draft_catalog_moderation_abuse_controls_tz_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/draft_catalog_moderation_abuse_controls_tz_2026_04_21.md)

### 3. Canonical Winery Alias / Merge

Alias layer:

- [winery_alias_layer_spec.md](/R:/Flutter/Project/winepool_final/docs/winery_alias_layer_spec.md)

Admin merge workflow:

- [winery_merge_admin_spec.md](/R:/Flutter/Project/winepool_final/docs/winery_merge_admin_spec.md)

Operational admin-facing context:

- [administrator_guide.md](/R:/Flutter/Project/winepool_final/docs/administrator_guide.md)

Важно:
на 21.04.2026 [administrator_guide.md](/R:/Flutter/Project/winepool_final/docs/administrator_guide.md) уже покрывает не только winery/business admin surfaces, но и текущий live-baseline по:

- `Очереди черновиков на проверке`
- экрану `Разбор черновика для каталога`
- evidence/photo/barcode review
- repeat submission / clarification loop

## Документы-Записи Уже Выполненных Slice-ов

Эти документы полезны, но их лучше читать как snapshot выполненных этапов, а не как главный roadmap:

- [phase3_phase4_hardening_plan.md](/R:/Flutter/Project/winepool_final/docs/phase3_phase4_hardening_plan.md)
- [draft_resolution_test_checkpoint.md](/R:/Flutter/Project/winepool_final/docs/draft_resolution_test_checkpoint.md)

Они отвечают скорее на вопрос:

- что именно хотели закрыть этим slice
- почему этот этап был следующим логичным шагом
- что считалось done на момент закрытия конкретного этапа

## Historical / Handoff / Research Context

Эти документы не нужно удалять, но и не стоит читать как текущий контракт:

Исторический handoff:

- [qr_receipt_draft_handoff_2026_03_24.md](/R:/Flutter/Project/winepool_final/docs/qr_receipt_draft_handoff_2026_03_24.md)

Research / product-thinking:

- [scanCheque.md](/R:/Flutter/Project/winepool_final/docs/scanCheque.md)

Дополнительные reference-файлы:

- [SELF_DEBUG_GUIDE.md](/R:/Flutter/Project/winepool_final/docs/SELF_DEBUG_GUIDE.md)
- [WinePoolDB.sql](/R:/Flutter/Project/winepool_final/docs/WinePoolDB.sql)
- [SELLERS_ARCHITECTURE_SPECS.md](/R:/Flutter/Project/winepool_final/docs/SELLERS_ARCHITECTURE_SPECS.md)

## Как Сейчас Читать Общую Программу

### Role/Business Track

По смыслу картина сейчас такая:

- `Phase 0`–`Phase 4` в основном закрыты как baseline
- `Phase 5` уже не “планируемый”, а largely landed:
  - alias CRUD
  - merge execution
  - rollback-safe unmerge
  - compare-prep merge entrypoint
  - merge history / audit foundation
- `Phase 6` тоже уже сдвинулся из идеи в работающий moderated baseline:
  - `winery_alias_add`
  - `winery_edit`
  - moderated `retail/horeca -> winery`

Значит, главный незакрытый слой здесь уже не “начать governance”, а:

- поддерживать doc sync
- пройти regression smoke
- удержать consistency между UI, policy и backend contract
- потом перейти к следующему аккуратному slice вроде wine-level duplicate merge

### Receipt Track

Receipt/QR направление тоже уже прошло стадию “собрать первый рабочий flow”.

Сейчас baseline уже включает:

- QR ingestion
- review/history/details
- draft queue/details/link/merge
- draft catalog moderation queue/details для администратора
- evidence/photo/barcode слой для moderator review
- resubmission loop после уточнений и после отклонения
- grape detection v2
- alias-aware parser improvements
- reserve tie-breaker calibration
- owner-scoped suspect price trust split (`owner_confirmed` / `owner_rejected` вместо direct `verified`)

Следующий слой здесь уже другой:

- `Track B` full redesign экрана `Черновик вина`
- anti-abuse governance для draft catalog moderation
- live calibration на реальных чеках
- CTA/UX polish
- raw receipt replay persistence
- системный audit alias coverage
- weak OCR / weak grape explanation polish

### Общий Мост Между Треками

Главное архитектурное правило, которое теперь объединяет обе линии:

- buyer receipt flow остаётся self-service consumer workspace
- business winery collaboration идёт через moderated proposals
- canonical catalog direct writes, alias CRUD и merge остаются privileged admin surfaces

То есть receipt track больше нельзя читать как автономный каталоговый workflow без связи с governance.
Но и role/business roadmap нельзя читать так, будто receipt flow это просто часть seller/admin консоли.

## Рекомендуемый Порядок Чтения По Сценариям

### Если задача про роли, права, RLS, capability и business model

1. [role_business_architecture_spec.md](/R:/Flutter/Project/winepool_final/docs/role_business_architecture_spec.md)
2. [role_business_architecture_implementation_plan.md](/R:/Flutter/Project/winepool_final/docs/role_business_architecture_implementation_plan.md)
3. [role_business_write_surface_inventory.md](/R:/Flutter/Project/winepool_final/docs/role_business_write_surface_inventory.md)
4. [role_business_policy_draft.md](/R:/Flutter/Project/winepool_final/docs/role_business_policy_draft.md)

### Если задача про receipt / QR / drafts / prices

1. [receipt_buyer_admin_governance_spec.md](/R:/Flutter/Project/winepool_final/docs/receipt_buyer_admin_governance_spec.md)
2. [receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md](/R:/Flutter/Project/winepool_final/docs/receipt_draft_proposal_moderation_implementation_handoff_2026_04_20.md)
3. [next_iterations_execution_order.md](/R:/Flutter/Project/winepool_final/docs/next_iterations_execution_order.md) — historical sequencing context, не текущая очередь
4. [draft_wine_session_bootstrap.md](/R:/Flutter/Project/winepool_final/docs/draft_wine_session_bootstrap.md)
5. [draft_wine_details_redesign_tz_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/draft_wine_details_redesign_tz_2026_04_21.md)
6. [track_b_draft_details_evidence_implementation_plan_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/track_b_draft_details_evidence_implementation_plan_2026_04_21.md)
7. [draft_catalog_moderation_abuse_controls_tz_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/draft_catalog_moderation_abuse_controls_tz_2026_04_21.md)
8. [receipt_sprint_status.md](/R:/Flutter/Project/winepool_final/docs/receipt_sprint_status.md)
9. [receipt_follow_up_backlog.md](/R:/Flutter/Project/winepool_final/docs/receipt_follow_up_backlog.md)
10. [receipt_bug_bash_calibration_checklist.md](/R:/Flutter/Project/winepool_final/docs/receipt_bug_bash_calibration_checklist.md)
11. [receipt_grape_detection_spec.md](/R:/Flutter/Project/winepool_final/docs/receipt_grape_detection_spec.md)
12. [draft_wine_flow_spec.md](/R:/Flutter/Project/winepool_final/docs/draft_wine_flow_spec.md)
13. [administrator_guide.md](/R:/Flutter/Project/winepool_final/docs/administrator_guide.md)

### Если задача про canonical winery aliases / duplicate / merge

1. [winery_alias_layer_spec.md](/R:/Flutter/Project/winepool_final/docs/winery_alias_layer_spec.md)
2. [winery_merge_admin_spec.md](/R:/Flutter/Project/winepool_final/docs/winery_merge_admin_spec.md)
3. [role_business_architecture_implementation_plan.md](/R:/Flutter/Project/winepool_final/docs/role_business_architecture_implementation_plan.md)
4. [administrator_guide.md](/R:/Flutter/Project/winepool_final/docs/administrator_guide.md)

## Historical Doc Backlog Snapshot

Список ниже зафиксирован 22.04.2026 и сохраняется как historical snapshot. Он не заменяет правила документации и ближайший порядок из post-1.1.0 roadmap:

- держать [role_business_architecture_implementation_plan.md](/R:/Flutter/Project/winepool_final/docs/role_business_architecture_implementation_plan.md) синхронизированным с уже landed `Phase 5`
- при изменениях alias/merge UI сразу синхронизировать:
  - [role_business_policy_draft.md](/R:/Flutter/Project/winepool_final/docs/role_business_policy_draft.md)
  - [role_business_write_surface_inventory.md](/R:/Flutter/Project/winepool_final/docs/role_business_write_surface_inventory.md)
  - [administrator_guide.md](/R:/Flutter/Project/winepool_final/docs/administrator_guide.md)
- при новых receipt slices сразу проверять, не расходятся ли:
  - [receipt_buyer_admin_governance_spec.md](/R:/Flutter/Project/winepool_final/docs/receipt_buyer_admin_governance_spec.md)
  - [receipt_sprint_status.md](/R:/Flutter/Project/winepool_final/docs/receipt_sprint_status.md)
  - [receipt_follow_up_backlog.md](/R:/Flutter/Project/winepool_final/docs/receipt_follow_up_backlog.md)
  - [administrator_guide.md](/R:/Flutter/Project/winepool_final/docs/administrator_guide.md)
- когда начнётся реализация anti-abuse слоя для moderation flow, синхронно обновлять:
  - [administrator_guide.md](/R:/Flutter/Project/winepool_final/docs/administrator_guide.md)
  - [draft_catalog_moderation_abuse_controls_tz_2026_04_21.md](/R:/Flutter/Project/winepool_final/docs/draft_catalog_moderation_abuse_controls_tz_2026_04_21.md)
- не превращать historical handoff/research файлы обратно в source of truth

## Короткий Итог

Если смотреть на `docs/` целиком, то общая картина сейчас такая:

- platform governance foundation уже в основном собрана
- canonical winery alias/merge governance уже перешла из planning в working admin baseline
- receipt sprint уже перешёл из build-out стадии в calibration/polish stage
- главная задача по документации теперь не “написать ещё один большой spec”, а удерживать синхрон между живыми truth-документами и историческим слоем
