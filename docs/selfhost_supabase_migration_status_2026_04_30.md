# Self-host Supabase migration status

Date: 2026-04-30

## Current state

WinePool has been moved from cloud Supabase access to a self-hosted Supabase stack on the Russian VPS.

- Public API domain: `https://api.winepool.ru`
- DNS is delegated to host-food name servers.
- `api.winepool.ru` resolves to the VPS IP `91.227.18.214`.
- Supabase is served through nginx -> Kong on localhost.
- Direct external access to Supabase/Postgres ports is not required; public traffic should go through 80/443 only.
- Receipt processing runs on the same VPS behind nginx at `https://api.winepool.ru/receipt/process-receipt-qr`.
- Receipt purchase place geocoding runs as the self-host Edge Function `geocode-receipt`.

## Verified checks

- `https://api.winepool.ru/auth/v1/health` works with the self-host anon key.
- `https://api.winepool.ru/rest/v1/wines?select=id&limit=1` returns data through the self-host REST API.
- `https://api.winepool.ru/receipt/health` returns `200 OK`.
- `https://api.winepool.ru/functions/v1/geocode-receipt` returns coordinates for receipt addresses through Yandex Geocoder.
- Public storage image URLs served by self-host storage return `200 OK`.
- Mobile internet without VPN is stable in initial manual testing.

## Data migrated

Final self-host counts after import:

```text
public policies      102
auth.users           12
auth.identities      12
storage.objects      1081
public.wines         1161
public.user_receipts 9
```

Storage files copied to the file backend:

```text
1081 files
273M
```

Existing auth sessions were intentionally not migrated. Users are expected to sign in again.

## Application changes

- Supabase initialization now uses `AppSupabaseConfig`.
- Default Supabase URL is `https://api.winepool.ru`.
- Default anon key is the self-host anon key.
- `proxyStorageUrl()` rewrites legacy cloud Supabase storage origins to the configured self-host origin.
- Receipt QR processing defaults to `https://api.winepool.ru/receipt/process-receipt-qr`.
- `delete-user` edge function no longer hardcodes the old cloud Supabase URL or service role key; it reads environment variables.
- `geocode-receipt` is deployed to the VPS and uses Yandex Geocoder as the primary provider. Photon remains as an emergency fallback.
- Legacy cloud Supabase storage URLs in `wines.image_url` and `draft_wines.photo_url` were normalized in the self-host database to `https://api.winepool.ru`.
- Storage files were linked into the self-host file backend layout `volumes/storage/stub/stub/<bucket>/<object>/<version>` and xattrs were restored from `storage.objects.metadata`.

## VPS services

Useful checks:

```bash
ssh -i ~/.ssh/winepool_vps root@91.227.18.214

cd /opt/supabase/docker
docker compose ps
docker compose logs auth --tail=100
docker compose logs storage --tail=100

systemctl status winepool-receipts
journalctl -u winepool-receipts --no-pager -n 100

curl https://api.winepool.ru/receipt/health
```

Receipt service files:

```text
/opt/winepool-receipts
/etc/systemd/system/winepool-receipts.service
/etc/winepool-receipts.env
```

## Recommended follow-up

1. Build and install a fresh APK using the self-host defaults.
2. Test a complete mobile flow without VPN: login, catalog, image loading, wine details, cellar, receipt scan, receipt save.
3. Audit missing catalog images. Some catalog image records still point to legacy cloud Supabase URLs; the app rewrites them at runtime, but the database should eventually be normalized to `https://api.winepool.ru`.
4. Add a diagnostic script that checks `wines.image_url`, winery logos/banners, and draft photos against `storage.objects` and the file backend.
5. Create a VPS backup:
   - local DB dump after migration,
   - `/opt/supabase/docker/volumes/storage`,
   - nginx config,
   - receipt service unit/config,
   - Supabase `.env` stored securely outside git.
6. Add basic monitoring for:
   - Supabase auth health,
   - receipt health,
   - `geocode-receipt` health via a known test address,
   - disk usage,
   - `docker compose ps`,
   - `winepool-receipts.service`.
7. After the event and at least two stable weeks, rotate exposed credentials and remove old cloud dependencies:
   - prod Supabase DB password,
   - prod Supabase S3 keys,
   - old service role keys,
   - VPS root password,
   - disable SSH password auth.
8. Keep the old cloud Supabase project alive for rollback for at least two weeks.

## Known caveats

- If catalog images disappear again after a data import, check for old `wnzewejxrhjnjvyrshzp.supabase.co` URLs in public text columns and normalize them to `https://api.winepool.ru`.
- If storage object URLs return `500 ENOENT` or `ENODATA`, verify the file backend layout and xattrs `user.supabase.content-type` / `user.supabase.cache-control`.
- The self-host DB runs PostgreSQL 15 while the source cloud dump came from PostgreSQL 17. The import succeeded for the application-critical data, but internal Supabase auth/storage schema differences produced expected owner/schema warnings during import.
- `auth.sessions` were not migrated by design.
- The Yandex Geocoder key was refreshed on the VPS. After changing `.env`, the `functions` container must be recreated with `docker compose up -d --force-recreate functions`; a simple restart keeps the old environment.
- The original migration handoff document contains live secrets and should not be committed to git as-is.

## Session 2.5 schema update — 2026-07-30

Applied `202607300100_add_showcase_flag_to_vintage_images.sql` to the self-hosted
production database through `supabase-db` with `ON_ERROR_STOP=1`.

Post-apply verification:

- `public.wine_vintage_images.is_showcase` exists, is `NOT NULL`, default `false`;
- `wine_vintage_images_showcase_role_check` exists;
- `admin_set_wine_vintage_image_showcase(uuid, boolean)` exists;
- `admin_upsert_wine_vintage_image_v2(...)` exists;
- existing showcase row count remained `0`, so no legacy evidence image was
  silently promoted to public hero content.
