# H1-02B public operator page handoff

Date: 01.09.2026

Status: implementation complete; local responsive acceptance and production
data-contract smoke passed.

## Delivered

- canonical route `/tourism/o/{operator_slug}` in the tourism web entrypoint;
- versioned `get_public_tourism_operator_page_v1` repository/provider/model;
- responsive full-page composition: operator hero, anchor navigation, curated
  tours, story/feature media, trust facts, lead process, editorial gallery,
  trust/legal, FAQ, final CTA and footer;
- referral propagation from operator page to exact-tour route;
- copy-to-clipboard share action using the canonical URL without tracking query;
- controlled media for the operator and both current pilot tours;
- strict media-slot contracts preventing tour or winery imagery from becoming
  operator hero/feature/gallery by fallback;
- shared `TourismEditorialGallery` with desktop master/detail and mobile
  previous/next + counter behaviour.

## Behaviour decisions

- two current published tours are rendered in deterministic RPC order;
- `Все туры оператора` remains hidden while all inventory fits on the page;
- top-level catalog labels are intentionally non-navigating until their truthful
  destination routes exist;
- direct partner contacts are absent before a WinePool lead;
- arbitrary photo content never influences entity classification or relations.

## Verification

- focused Flutter analysis: passed;
- domain/model tests and existing published-tour contract tests: passed;
- mobile 390x844: no horizontal overflow; long Inkerman schedule wraps cleanly;
- desktop 1440 CSS px: hero, cards, story, process, gallery, FAQ and CTA inspected;
- mobile gallery advanced from `1 / 2` to `2 / 2`;
- FAQ expansion worked;
- flagship CTA opened
  `/tourism/o/yalta-excursions/t/yalta-massandra-tasting?source=qa&ref=qa-h1-02b`;
- fresh browser warning/error log: empty;
- production RPC smoke: operator, both exact tours, both wineries and legacy
  wine embed returned HTTP 200;
- visual comparison:
  `docs/assets/h1_02b_qa/operator-reference-vs-implementation.png`.

## Production data work

- `202609010050_enforce_public_profile_media_slots.sql` deployed;
- `202609010060_seed_h1_02_showcase_media.sql` deployed;
- 16 controlled WebP objects uploaded and publicly reachable;
- backup before the media deployment:
  `/root/winepool-backups/h1_02_showcase_media_20260901T0725MSK`.

The Flutter web artifact itself was not published by this task. The local
verified preview remains the implementation evidence; web deployment should be
performed by the normal release workflow.

## Next slice

H1-02C: build `/wineries/{winery_slug}` with the same shell and gallery while
keeping `direct_tours` and verified `visiting_tours` visibly distinct. The
controlled `massandra` and `inkerman` profiles are ready for that work.
