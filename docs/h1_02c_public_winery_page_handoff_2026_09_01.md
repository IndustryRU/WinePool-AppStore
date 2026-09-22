# H1-02C public winery page handoff

Date: 01.09.2026

Status: implementation complete; local responsive acceptance and production
data-contract smoke passed.

## Delivered

- canonical `/wineries/{winery_slug}` route in the tourism web entrypoint;
- typed `PublicWineryPageV1` model, repository method and Riverpod provider for
  `get_public_winery_page_v1`;
- responsive winery composition: own hero, anchor navigation, visit options,
  story/feature media, practical facts, optional key wines, shared gallery,
  trust/FAQ, final CTA and footer;
- explicit separation of programs operated by the winery from verified visits
  run by a third-party operator;
- operator identity, departure/transfer, duration, schedule and price remain
  visible on every third-party visit card;
- exact-tour navigation preserves the `source` and `ref` acquisition chain;
- single-image galleries no longer render misleading previous/next controls;
- one shared safe transfer label formatter is used by winery, operator and
  exact-tour pages.

## Current production truth

Both pilot profiles are intentionally sparse and render without invented
content:

| Winery | Direct programs | Verified visiting tours | Responsible operator |
|---|---:|---:|---|
| `massandra` | 0 | 1 | `yalta-excursions` / «Едувялту» |
| `inkerman` | 0 | 1 | `yalta-excursions` / «Едувялту» |

Therefore both pages honestly state that a direct WinePool program from the
winery is not currently published. Empty facts, wines or editorial modules are
hidden rather than filled with placeholders.

## Public contract boundary

Migration `202609010070_bound_public_winery_visit_cards.sql` narrows each
visiting item to the fields required for a public selection card. It excludes
contacts, vehicles, pickup points, full programs, requirements, payment and
cancellation internals, moderation data and media rights internals.

The migration and its denylist contract were transactionally preflighted,
deployed to production and smoke-tested through PostgREST. Pre-deployment
backup:

`/root/winepool-backups/h1_02c_bounded_winery_cards_20260901T142959MSK`

## Verification

- SQL bounded-card contract: passed before and after production deployment;
- production RPC smoke for Massandra, Inkerman, operator and exact tours: HTTP
  200;
- focused model and widget tests: passed;
- direct/self-arrival and third-party/operator/transfer labels covered by a
  widget test;
- desktop Massandra and Inkerman pages inspected with their own entity media
  and correct tour media;
- mobile 390x844: no horizontal overflow and one-image gallery controls are
  truthful;
- standard 800 px widget viewport: top navigation overflow found and fixed;
- browser warning/error log after the narrowed production DTO: empty;
- QA captures: `docs/assets/h1_02c_qa/`.

The Flutter web artifact is not published by this slice. Production data and
RPC changes are live; the UI remains a locally verified release candidate.

## Next slice

H1-02D is the structured, moderated authoring path for operator and winery
profiles: approved text sections, media slots, alt/rights metadata, preview,
review, publish and pause. Visual polish of H1-02C may proceed independently as
long as the direct-versus-third-party semantics stay unchanged.
