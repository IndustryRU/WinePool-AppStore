# WinePool analytics event dictionary v1

Дата контракта: 25.08.2026. Semantic boundary: события после H0.0. В каждом
client event фасад автоматически добавляет `build_mode` и `actor_type`.

Общие запрещённые параметры: email, UUID пользователя, телефон, имя, OCR/text,
barcode, fiscal FN/FD/FP, review text, tourism contact/comment. Raw включает все
actor types; clean допускает только `external` + `release`. Client retention —
согласно AppMetrica; server facts — согласно production DB retention.

| Name | Owner / trigger | Scope | Allowed params | Idempotency | Source of truth | Raw / clean | Rollout |
|---|---|---|---|---|---|---|---|
| `onboarding_completed` | onboarding / finish or skip | client | `skipped` | once per local onboarding | client | yes / diagnostic only | H0.0 |
| `registration_wall_shown` | auth / wall visible | client | `kind` | per display | client | yes / yes | legacy |
| `registration_wall_action` | auth / CTA tap | client | `kind`, `action` | per tap | client | yes / yes | legacy |
| `registration_started` | auth / submit begins | client | `entry` | once per submit attempt | client | planned / planned | H0.2 audit |
| `registration_completed` | auth / account created | server | none | auth user PK | `auth.users.created_at` | report only | H0.3 |
| `catalog_search_started` | catalog / explicit search | client | normalized `entry` | per search attempt | client | planned / planned | H0.2 audit |
| `catalog_result_opened` | catalog / result opened | client | `entry`, `result` | per open | client | planned / planned | H0.2 audit |
| `add_bottle_opened` | cellar / hub opened | client | `entry` | per open | client | yes / yes | legacy |
| `add_bottle_path_selected` | cellar / path selected | client | `path` | per selection | client | yes / yes | legacy |
| `add_bottle_match_result` | matcher / result resolved | client | `attempt_id`, `path`, `result`, `candidates` | once per attempt | client diagnostic | yes / yes | H0.0 |
| `add_bottle_candidate_selected` | matcher / candidate chosen | client | `attempt_id`, `path`, `rank`; canonical wine ID temporarily legacy | per candidate/rank | client diagnostic | yes / yes | legacy |
| `add_bottle_candidate_rejected` | matcher / candidates rejected | client | `attempt_id`, `path`, `had_candidates` | once per attempt | client diagnostic | yes / yes | H0.0 |
| `add_bottle_draft_created` | draft / persisted | client+server | `attempt_id`, `entry_method`, boolean flags | once per attempt | draft submission/server row | yes / server preferred | H0.0 |
| `wine_added` | cellar / storage persisted | client+server | none | server PK for KPI | `user_storage.created_at` | yes / server fact | H0.0 |
| `receipt_scanned` | receipts / receipt saved | client+server | none | server PK for KPI | `user_receipts.created_at` | yes / server fact | H0.0 |
| `tasting_logged` | cellar / tasting saved | client+server | none | server PK for KPI | `user_tastings.created_at` | yes / server fact | H0.0 |
| `review_created` | cellar / review saved | server | none | review PK | `reviews.created_at` | report only | H0.3 |
| `catalog_proposal_submitted` | catalog / submission persisted | server | none | submission PK | `draft_catalog_submissions.submitted_at` | report only | H0.3 |
| `shop_place_opened` | map / place details opened | client | `entry`, normalized `result` | per open | client | planned / planned | H0.2 audit |
| `route_opened` | map / route intent | client | `entry` | per tap | client | planned / planned | H0.2 audit |
| `tourism_public_open` | tourism / direct public route opened | client | normalized `referral_code` | per detail mount | client | yes / external release | H0.4 |
| `tourism_experience_view` | tourism / published experience rendered | client | normalized `referral_code` | per detail mount | client | yes / external release | H0.4 |
| `tourism_lead_started` | tourism / request sheet opened | client | normalized `referral_code` | per sheet open | client | yes / external release | H0.4 |
| `tourism_lead_submitted` | tourism / server insert succeeded | client+server | normalized `referral_code` | lead PK | `tourism_leads.created_at` | yes / server preferred | H0.4 |

Add-bottle invariant: one opaque `attempt_id` links path → one match result →
optional selection/rejection → draft or storage outcome. `abandoned` is derived
from the attempt cohort and is not emitted on screen close.

## Server-fact metric contract

- Activation v1: at least one qualifying fact at or after registration and
  before midnight at the start of registration date + 7 days.
- D0 activation: qualifying fact on the registration calendar date.
- Strong activation: qualifying facts on two distinct calendar dates inside
  that seven-day window.
- Meaningful return: a qualifying fact on a later calendar date.
- D1/D7/D30: exact calendar-day qualifying return; `NULL` until the cohort has
  reached the relevant age.
- Qualifying facts: storage, tasting, saved receipt, review, submitted catalog
  draft. Views, onboarding, search and moderation work are excluded.
- `external` is clean. `owner`, `qa`, `automated` are retained separately.
- Cohorts below five registrations carry `low_sample=true`; percentages must be
  computed only together with their numerator and denominator.
