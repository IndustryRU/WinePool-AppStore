# H1-01 — bounded public tourism data audit and Package A handoff

Дата: 31.08.2026
Статус: Package A deployed and smoke-verified in production; native public
catalog/operator compatibility adapter accepted on a physical Android device
Граница: без ПДн, UUID, contact destinations и visual implementation.

## Current facts

### Local (после adapter pass)

- Canonical exact-tour route `/tourism/o/{operator_slug}/t/{tour_slug}` uses
  `get_published_tour_v1(text,text)` and the immutable current publication.
- `main_tourism_web.dart` has exact-tour routes only; operator and catalog web
  routes are not implemented.
- Native public `fetchOrganization(s)`, `fetchExperiences` and
  `fetchExperiencesForOrganization` use the bounded Package A RPCs. Slug-based
  legacy detail resolves the catalog card to the exact current publication.
- Public cards carry slug identities only and navigate by
  `/tourism/o/{operator_slug}/t/{tour_slug}`. Existing `ref`, `referral` and
  `source` query values survive the catalog-to-tour hop.
- Existing lead/trip records still address an experience by UUID. Their
  compatibility branch temporarily uses the legacy published-row hydrate;
  admin authoring continues on its dedicated raw/admin paths.
- Published Tour v1 is already a safe source for current tour cards, but its
  exact-tour payload is intentionally wider than a catalog card.

### Production (initial read-only audit)

- Organizations: 2 active rows, no paused/archived rows. Only 1 active row has
  a public slug; that row is also verified and has story, website, logo and
  hero readiness.
- Tours: 2 `published`, both have a current immutable publication under an
  active operator.
- Approved public organization coverage among active rows: city/region 2/2;
  slug, verified, story, website, logo and hero 1/2; public phone/messenger
  0/2. No contact value was read or printed.
- `get_published_tour_v1(text,text)` is stable/security-definer and executable
  by anon, authenticated and service role; admin publish/preview are not anon.
- The current publication operator snapshot contains only slug, name, type,
  verified, story, logo and website. Both tour snapshots have the full accepted
  Published Tour v1 shape.
- Referral registry: 5 commercial points are complete but inactive; 2 test
  points are active. Location, material, partner and experience references are
  present for every point. Responsible/SLA/placement verification fields do
  not exist yet, so physical readiness is not proven.
- Membership aggregate: 2 active members across owner/agent roles; active
  defaults and notification recipients exist. No user identity was read.
- Lead aggregate is non-empty, including commercial new/confirmed and test
  new/completed/no-show facts. No lead row or contact data was read.

## Anonymous raw-read closure

The legacy gap found by the initial audit is closed in production. Migration
`202608311845_close_h1_anonymous_raw_tourism_reads.sql` removes both the anon
table grants and anon membership in the public-read policies for:

- `tourism_organizations`;
- `tourism_experiences`, stops, pickup points and vehicles;
- `tourism_media_assets`;
- `tourism_home_hero_images`.

Authenticated compatibility/admin grants remain unchanged. Anonymous clients
can reach current content only through bounded security-definer RPCs. Public
Storage object reads remain intentionally available because the returned
approved image URLs must render, but metadata tables are no longer readable.

## Public DTO allowlist

| DTO | Allowed fields | Explicitly excluded |
|---|---|---|
| Operator | schema version; slug; public name; type; verified; approved story; city; region; logo URL/alt; hero media URL/alt; current tour cards | IDs; website, phone, email, messenger and every direct booking channel; notification routes; members; responsible; business link; delivery policy; drafts |
| Operator tour card | canonical path; publication revision/time; slug; editorial title/subtitle/value; destination/city/region; kind/verified; duration; schedule; price currency/summary; cover source/alt | description body; pickup details; vehicles; route internals; price components; lead config; publication IDs/hash |
| Catalog card | same bounded tour card plus operator slug/name/type/verified/logo URL | story/contact; raw description in response; internal IDs; vehicles/pickups; draft fields |

Search may inspect the current snapshot title, public value/description,
city/region and operator name, but it never echoes the raw query.

## Package A contracts

```text
get_public_tourism_operator_v1(p_operator_slug text) -> jsonb|null
list_public_tourism_catalog_v1(
  p_query text = null,
  p_region text = null,
  p_cursor text = null,
  p_limit integer = 12
) -> jsonb
record_tourism_acquisition_event_v1(
  p_event_type text,
  p_session_id text,
  p_operator_slug text = null,
  p_tour_slug text = null,
  p_referral_code text = null,
  p_referral_source text = null,
  p_canonical_path text = '/tourism',
  p_build_mode text = 'release',
  p_is_test boolean = false
) -> table(recorded boolean, occurred_at timestamptz)
```

Operator returns null for missing, inactive or zero-current-tour operators.
Catalog accepts 1–24 rows and uses an opaque keyset cursor ordered by current
publication time, operator slug and tour slug. Invalid limit/query/region/cursor
raises SQLSTATE `22023`; an empty result is a successful empty page.

Acquisition accepts only four milestones, hashes the opaque session, derives
actor type server-side, resolves only current public targets, normalizes active
referrals from the registry, deduplicates an entity/path milestone and limits a
session to 12 new events per 10 minutes. Canonical paths cannot contain query or
fragment data. There is no payload argument, so email, phone, comment, claim
secret and raw URL have no storage path. Lead submit/outcome remain lead/event
facts.

The same Package A migration narrows the anonymous exact-tour response by
removing legacy `operator.website_url` at read time. Immutable stored
publication rows are preserved; the optional Dart field remains compatible and
parses as absent.

## RLS and grants target matrix

| Surface | anon | authenticated | service/admin |
|---|---|---|---|
| Three public Package A RPCs | execute | execute | execute |
| Acquisition table/sequence | none | none | owner/service SQL only |
| Publications | no raw rows | catalog admin policy only | owner/service |
| Members, leads, lead events | existing RLS only | own/member/admin policies | owner/service |
| Referral registry/outcomes | none | bounded admin RPC only | owner/service |
| Legacy public raw tables | temporarily compatible | temporarily compatible/admin RLS | owner/service |

## Compatibility consumers after raw-read closure

- Completed: `fetchOrganization`, `fetchOrganizations`, `fetchExperiences` and
  `fetchExperiencesForOrganization` now consume Package A.
- Completed: slug-based `fetchExperience` resolves to the exact publication;
  list cards use canonical operator/tour routes.
- Completed: public organization DTOs do not populate website, phone or
  messenger, so the existing conditional external-site CTA is absent without a
  visual redesign.
- Completed: UUID-based lead/trip compatibility calls
  `resolve_public_tourism_route_v1(text)` and then the exact current publication;
  it no longer hydrates anonymous raw rows.
- Completed: `fetchHomeHeroImages` calls
  `get_public_tourism_home_hero_images_v1()` and receives only mode plus image
  source.
- Admin editor methods share the same repository but must remain on admin raw
  policies or dedicated admin projections; they must not be routed through the
  public DTO.

`fetchMyLatestLeadForExperience` still resolves a UUID through authenticated
RLS after a user check. This is not an anonymous path and remains part of the
explicit authenticated compatibility boundary.

## Migration, rollback and verification plan

1. Backup only schemas/functions/grants for touched objects and the new empty
   acquisition table contract before production application.
2. Apply `202608311700_add_h1_public_tourism_contracts.sql` only.
3. Run the H1 SQL transaction contract and anon RPC smoke; do not create a lead.
4. Disable rollback: revoke execute on the three functions. Preserve recorded
   events; do not drop data. Code rollback returns clients to the accepted exact
   tour and legacy catalog until the adapter gate.
5. Raw-read revokes are intentionally deferred until compatibility consumers
   have migrated and their contract test changes from documented gap to denial.

Implemented SQL test coverage: function grants, acquisition raw denial,
publication/operator pause filtering, strict DTO leakage checks, search/region,
stable cursor, limit 24, referral normalization, event idempotency, unknown
referral rejection and raw-query path rejection. Remaining planned coverage at
the adapter/closure gate: legacy raw-table denial and production PostgREST
smoke; load/rate-limit boundary gets a dedicated test before traffic scaling.

Before deployment on 31.08.2026 the complete migration plus contract test was
executed against the production PostgreSQL version inside one outer transaction
and ended with `ROLLBACK`. All DDL and both test blocks passed.

Production deployment then used only
`202608311700_add_h1_public_tourism_contracts.sql` (SHA-256
`E671D2EE0BD72099230D9F9A98270139D9850A4AE09432B742C5F3E45359441D`). The
scoped server backup is
`/root/winepool-backups/h1_package_a_20260831T162327_976904c8`; permissions are
700/600 and its checksum manifest passed. Production has no common
`supabase_migrations.schema_migrations` registry, so no synthetic registry row
was inserted.

One bounded anonymous smoke proved:

- operator/catalog/exact-tour/acquisition RPC execution is granted;
- both operator and catalog return the 2 current tours;
- exact-tour output has no `operator.website_url`;
- raw anon select/insert on the acquisition table is denied;
- one privacy-safe `is_test=true`, `build_mode=test` `/tourism` event was
  persisted and its repeated key deduplicated to one row.

The anonymous closure deployment used only
`202608311845_close_h1_anonymous_raw_tourism_reads.sql` (SHA-256
`DF6EF33F01782C9E6DB909CFECCEC0BBB527B80BB4941E91ECF662E664E435F6`). Its
scoped backup is
`/root/winepool-backups/h1_anon_closure_20260831T170001_8cff5419`; directory/file
permissions are 700/600 and all checksums pass. Transactional preflight ended
in `ROLLBACK`; the migration then committed and the independent SQL contract
passed.

PostgREST smoke after schema-cache reload returned HTTP 200 for hero resolver,
UUID/slug route resolver and exact tour, with one active hero mode and no exact
website. Anonymous raw reads of experiences and home heroes returned HTTP 401;
the SQL contract proves denial across all seven revoked tables.

### Physical-device acceptance — 31.08.2026

A signed release APK containing the anonymous-closure adapter was installed as
an in-place update on a Redmi Note 8 Pro running Android 11. The QA artifact
used `versionName=1.1.0` and temporary device-only `versionCode=2013`; this did
not change the repository version metadata.

Authenticated smoke confirmed that the home Tourism entry, `/tourism` catalog
and the exact “Дегустация вин в Массандре” page load current images and tour
data after anonymous raw reads were revoked. No Flutter, PostgREST, HTTP 401/403
or fatal application error was observed in the bounded log scan.

The owner then completed guest acceptance on the same physical device and
confirmed:

- the Tourism catalog and multiple tour cards load without errors or hangs;
- the exact tour shows its public program, schedule, duration, transfer, price
  and lead CTA;
- operator phone, email, website, messenger and other direct-contact bypasses
  are not exposed;
- tapping “Оставить заявку” while signed out opens the expected bottom sheet
  offering registration or sign-in and explains why an account is useful.

The account was signed out only for this guest check; application data and the
account itself were not deleted. This closes the native guest rollout gate for
the tested release adapter. Older native builds remain incompatible with the
revoked anonymous raw reads and must not be used as acceptance evidence.

## Owner decisions

Accepted 31.08.2026:

1. Public identity/reputation remains visible, but website, phone, email,
   messenger and other direct booking channels are excluded before a WinePool
   lead is recorded. Contact is disclosed only for fulfillment of that lead.
2. Historical monetization review restores both fixed qualified-lead fee and
   completed-visit success fee as H1-06 candidates. Exact basis/rate is not yet
   accepted; first agreed facts may be `waived`.
3. The catalog/operator card allowlist above is accepted.
4. Canonical routing is `/tourism` → operator → exact tour, preserving
   `ref`, `referral` and `source`.
5. Publication-time ordering is accepted for the current two-tour pilot.

No referral point was activated. No layout or visual design was changed; only
the data source and route destination of existing controls changed.
