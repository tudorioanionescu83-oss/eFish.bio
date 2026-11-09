# Copernicus Marine updater (public GitHub Actions runner)

This repository is public and is separate from the private `efish-app` application. The standalone updater in `scripts/update_constanta_marine.py` is copied from the private project without stored credentials. It downloads Copernicus Marine Black Sea model subsets (surface temperature, currents, salinity and waves), then upserts into `marine_station_data` in the **cote-dunare** Supabase project (`apnvxfaelgjthxuztqln`).

## Activation — do not send credentials through chat, email or a commit

1. Open **Settings → Secrets and variables → Actions → Secrets**:
   https://github.com/tudorioanionescu83-oss/eFish.bio/settings/secrets/actions
2. Create all **three repository secrets**, entering their exact values privately in GitHub:
   - `COPERNICUSMARINE_SERVICE_USERNAME`
   - `COPERNICUSMARINE_SERVICE_PASSWORD`
   - `SUPABASE_SERVICE_ROLE_KEY` — **from Supabase project cote-dunare**, not efish-app.
3. Under **Variables**, add `MARINE_IMPORT_ENABLED` with value `true`. Until then, both scheduled and manual jobs will be skipped. This is intentional to prevent spurious failures before credentials exist.
4. Open **Actions → Copernicus Marine - Public Runner → Run workflow** on the main branch and check the job status and the last imported timestamp:
   https://github.com/tudorioanionescu83-oss/eFish.bio/actions/workflows/constanta-marine-public.yml
5. Confirm that the data in `marine_station_data` was updated (including meaningful non-null marine fields) **before** removing the schedule from the private `efish-app` workflow.

The workflow schedules at 00:00, 06:00, 12:00 and 18:00 **UTC** (same as the original), and uses the free standard `ubuntu-latest` public-repository runner.

## Security and operating constraints

- Public source code, workflow definitions and Actions log output can be inspected by anyone. **Never** commit keys, passwords, `.env` files or downloaded datasets. Secrets are set only via the GitHub UI.
- The Supabase service-role key is powerful. Prefer replacing it later with a narrow-scope ingestion endpoint and rotating credentials if there is any sign of exposure.
- This workflow uses read-only `GITHUB_TOKEN` permissions and has no pull-request trigger. It does not upload diagnostics artifacts.
- The import script is unchanged: it queries 7 days of history and 5 days of forecast, then upserts with conflict key `(station_id,timestamp)`. Running both the old and new jobs simultaneously can create overlapping writes; turn off the old schedule after verification.
- Public repositories with no repository activity for **60 days** can have scheduled workflows disabled by GitHub. Keep a separate stale-data monitor and check that scheduled runs remain enabled; a scheduled run alone is not a substitute for a data-freshness check.
- GitHub workflows are not guaranteed to execute exactly at the scheduled minute.
- The new public workflow does **not** update the private application source or deploy eFish; it only updates the Supabase table used by the app.

## Relevant paths

- Workflow: `.github/workflows/constanta-marine-public.yml`
- Python script: `scripts/update_constanta_marine.py`
- Old private workflow: `efish-app/.github/workflows/constanta-marine-update.yml` (leave unchanged until successful test)
