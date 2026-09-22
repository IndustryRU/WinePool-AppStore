# H1-02 controlled showcase media manifest

Date: 01.09.2026

Mode: generated with OpenAI ImageGen, then converted to project-local WebP
derivatives.

Purpose: remove arbitrary cross-entity imagery from the two controlled pilot
tours and make owner/slot behaviour testable.

## Ownership rule

- operator page uses only `organization_profile` media;
- a tour and its search card use only that `experience` cover/gallery;
- a winery page uses only `winery_profile` hero/feature/gallery;
- a verified destination relation may aggregate a tour onto a winery page but
  never transfers ownership of its media;
- visual similarity, filename, caption and landmark recognition are never used
  to infer a domain relation.

## Final project assets

| Owner | Slot | Project file |
|---|---|---|
| operator `yalta-excursions` | hero | `docs/assets/h1_02_showcase_media/operator/hero.webp` |
| operator `yalta-excursions` | feature | `docs/assets/h1_02_showcase_media/operator/feature.webp` |
| operator `yalta-excursions` | gallery | `docs/assets/h1_02_showcase_media/operator/gallery-transport.webp` |
| operator `yalta-excursions` | gallery | `docs/assets/h1_02_showcase_media/operator/gallery-guide.webp` |
| tour `yalta-massandra-tasting` | cover | `docs/assets/h1_02_showcase_media/tours/massandra/cover.webp` |
| tour `yalta-massandra-tasting` | gallery | `docs/assets/h1_02_showcase_media/tours/massandra/gallery-tasting.webp` |
| tour `yalta-massandra-tasting` | gallery | `docs/assets/h1_02_showcase_media/tours/massandra/gallery-arrival.webp` |
| tour `inkermans-secrets` | cover | `docs/assets/h1_02_showcase_media/tours/inkerman/cover.webp` |
| tour `inkermans-secrets` | gallery | `docs/assets/h1_02_showcase_media/tours/inkerman/gallery-cellar.webp` |
| tour `inkermans-secrets` | gallery | `docs/assets/h1_02_showcase_media/tours/inkerman/gallery-tasting.webp` |
| winery `massandra` | hero | `docs/assets/h1_02_showcase_media/wineries/massandra/hero.webp` |
| winery `massandra` | feature | `docs/assets/h1_02_showcase_media/wineries/massandra/feature.webp` |
| winery `massandra` | gallery | `docs/assets/h1_02_showcase_media/wineries/massandra/gallery-terroir.webp` |
| winery `inkerman` | hero | `docs/assets/h1_02_showcase_media/wineries/inkerman/hero.webp` |
| winery `inkerman` | feature | `docs/assets/h1_02_showcase_media/wineries/inkerman/feature.webp` |
| winery `inkerman` | gallery | `docs/assets/h1_02_showcase_media/wineries/inkerman/gallery-cellar.webp` |

Production storage prefix:
`tourism-media/h1-02/showcase/v1/`. All 16 public objects returned HTTP 200
after upload. The database seed is
`supabase/migrations/202609010060_seed_h1_02_showcase_media.sql`.

## Final prompt set

All prompts requested premium editorial travel photography, natural light,
realistic people and architecture, restrained burgundy/gold-compatible colour,
no typography, no logos, no watermark and no unsafe alcohol consumption.

1. Operator hero — panoramic Crimean/Yalta coast at golden hour, premium dark
   passenger minibus on a mountain road, generous copy space, cinematic 16:9.
2. Operator feature — local female guide welcoming a small adult group beside
   the minibus, South Coast background, candid editorial photograph.
3. Operator gallery/transport — guests boarding the minibus at a scenic coastal
   stop, calm organised departure, wide travel photograph.
4. Operator gallery/guide — guide explaining the route to adults at a coastal
   viewpoint, map in hand, documentary travel photograph.
5. Massandra tour cover — historic stone barrel cellar, adult visitors walking
   with a guide, warm practical lighting, wide composition.
6. Massandra tour tasting — elegant supervised wine tasting for adults in a
   heritage cellar, bottles and glasses as context, no consumption close-up.
7. Massandra tour arrival — adults arriving at a historic Crimean wine estate,
   heritage architecture and landscaped grounds, editorial travel scene.
8. Inkerman tour cover — dramatic limestone wine galleries, barrels and adult
   visitors walking through the tunnel, cool mineral texture.
9. Inkerman gallery/cellar — guide-led adult group inside long limestone
   galleries, strong depth and authentic working-cellar atmosphere.
10. Inkerman gallery/tasting — calm supervised tasting for adults in a limestone
    room, refined documentary composition.
11. Massandra winery hero — broad historic winery estate on the Crimean coast,
    heritage architecture, gardens and mountains, premium wide establishing shot.
12. Massandra winery feature — cellar master in a historic barrel cellar,
    knowledgeable and approachable, editorial portrait with environmental context.
13. Massandra winery gallery — terraced vineyards and coastal terroir at warm
    light, realistic agricultural landscape.
14. Inkerman winery hero — monumental entrance to limestone wine cellars,
    restrained industrial heritage, wide establishing shot.
15. Inkerman winery feature — winemaker in limestone galleries, barrels and
    mineral walls, editorial environmental portrait.
16. Inkerman winery gallery — deep bottle archive within limestone tunnels,
    atmospheric but realistic cellar lighting.

Generated originals are retained under
`C:/Users/sirsa/.codex/generated_images/01a05715-a0d3-73c3-8f30-0748795d332f/`.
The WebP derivatives total 3,395,532 bytes; heroes/covers are 1672x941 and
supporting images are 1536x1024.

## Production evidence and recovery

- transactional preflight for migrations `0050` and `0060`: passed and rolled
  back before deployment;
- scoped backup:
  `/root/winepool-backups/h1_02_showcase_media_20260901T0725MSK`;
- production contract:
  `supabase/tests/20260901_h1_02_showcase_media_contracts.sql` passed;
- operator, both exact tours, both winery RPCs and the legacy wine embed
  returned HTTP 200 after deployment;
- the two tours were republished so their current immutable revisions contain
  the controlled covers and galleries.

These assets are illustrative pilot content, explicitly marked with ImageGen
provenance and confirmed rights metadata. Partner-provided verified media can
replace them later without changing routing or ownership logic.
