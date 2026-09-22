# H1-02A — public profile foundation handoff

Дата: 31.08.2026
Статус: implemented, deployed and contract-verified in production

## Результат

H1-02A подготовил additive data contract для публичных страниц оператора и
винодельни без начала финальной UI-реализации.

Добавлены:

- moderated operator profiles и allowlisted sections;
- canonical winery public profiles со slug, sections, verified facts и
  featured wines;
- explicit `tourism_experience_destinations` для проверенной связи тура с
  канонической винодельней;
- media owner `organization_profile`, новые semantic roles и поля подтверждения
  прав;
- bounded `get_public_tourism_operator_page_v1(text)`;
- bounded `get_public_winery_page_v1(text)`;
- RLS: новые таблицы не читаются anonymous напрямую, управление остаётся у
  catalog admin, публичное чтение идёт только через RPC.

## Зафиксированные факты аудита

Production PostgreSQL 15.8 до миграции:

- 377 canonical wineries без отдельного public-profile slug;
- 2 tourism organizations, 0 business bindings;
- 0 `businesses.managed_winery_id` bindings;
- 12 tour stops без canonical winery FK;
- 23 media assets; owner types фактически experience/pickup/vehicle;
- ни одной из семи H1-02A tables и только старый operator RPC.

Поэтому миграция не сопоставляет сущности по title, address или coordinates.
После deploy `tourism_experience_destinations` остаётся пустой. Первые связи
добавляются только как proposal и становятся публичными после `verified`.

## Совместимость

- Package A `get_public_tourism_operator_v1` не изменён;
- exact-tour Published Tour v1 не изменён;
- существующие public/native DTO продолжают работать;
- новый operator page RPC оборачивает безопасный Package A response;
- winery RPC удаляет legacy `operator.website_url` из immutable tour snapshot;
- old media rows проходят расширенные checks без backfill;
- текущий pilot operator получил минимальный published profile из уже
  одобренного `short_about`; контакты не копировались.

## Проверка и production

Migration:
`supabase/migrations/202608312330_add_h1_02a_public_profile_foundation.sql`.

Contract:
`supabase/tests/20260831_h1_02a_public_profile_contracts.sql`.

Оба файла сначала выполнены совместно внутри outer transaction с `ROLLBACK`.
После deploy contract повторно завершился собственным `ROLLBACK` и доказал:

- наличие семи таблиц и anon execute двух RPC;
- отсутствие anon raw-table grants;
- отсутствие direct contacts/internal IDs в DTO;
- draft section и proposed destination не публикуются;
- verified destination попадает в third-party group;
- missing winery возвращает `null`.

После schema reload отдельный smoke под `SET LOCAL ROLE anon` вернул operator
page и честный `null` для отсутствующей winery page. В production создан один
seeded published operator profile и ноль угаданных destination relations.

Scoped backup:
`/root/winepool-backups/h1_02a_20260831T202815Z`.
Schema dump checksum прошёл; permissions каталога/файлов 700/600.

## Compatibility correction — 01.09.2026

Composite primary key `(winery_id, wine_id)` на новой editorial-таблице
`winery_profile_featured_wines` был распознан PostgREST как дополнительная
many-to-many связь между `wines` и `wineries`. Из-за этого установленное
приложение с legacy embed `wines(..., wineries(...))` получало `PGRST201`.

Коррекция
`202609010045_preserve_legacy_wines_wineries_embed.sql` заменяет primary key
на surrogate `id`, сохраняя `unique (winery_id, wine_id)`, обе FK, RLS и
editorial semantics. Это убирает только автоматическое M2M-распознавание и не
меняет данные или публичный контракт профиля.

Перед применением migration и regression contract совместно прошли в outer
transaction с `ROLLBACK`. Scoped backup:
`/root/winepool-backups/h1_02a_embed_hotfix_20260901T0640MSK`; checksums и
permissions 700/600 проверены. После schema-cache reload полный legacy select
из `WinesRepository` вернул HTTP 200; operator-page и exact-tour RPC также
вернули HTTP 200. На момент коррекции editorial-таблица содержала 0 строк.

## Следующий шаг

H1-02B может начинать public operator page поверх
`get_public_tourism_operator_page_v1`. До UI implementation нужно добавить
Dart DTO/parser contract для page v1 и targeted repository method. Winery seed,
destination moderation и winery UI остаются отдельным H1-02C slice.
