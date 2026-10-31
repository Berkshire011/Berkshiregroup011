# Employment data preservation and Supabase migration runbook

## Status

Preparation only. This document does not run SQL, deploy Edge Functions, change frontend endpoints, or alter production.

**Source confirmed by the owner:** the current source is the Supabase project `dolzafpaqbhonbcrmhqv` (URL `https://dolzafpaqbhonbcrmhqv.supabase.co`). The destination project has not yet been identified. Migration remains blocked until the destination is confirmed, access to both projects is available through authorized secure tools, and a verified source backup is created. No source data has been accessed or copied by this preparation.

## Safety requirements

- Keep `main`, the live GitHub Pages site, and the current backend unchanged during preparation and testing.
- Never commit application exports, Social Security numbers, ID images, access tokens, database passwords, service-role keys, Turnstile secrets, or other credentials to GitHub.
- Do not use production applicant data for routine testing. Use synthetic test records.
- Keep the `employment-id-documents` bucket private. Never make it public.
- Restrict data transfer and backup access to authorized operators. Encrypt backups at rest and in transit; delete temporary copies securely after validation and the agreed retention period.
- Do not send applicant PII or ID documents through email, GitHub issues, pull requests, or chat.
- Do not delete or overwrite any source data as part of migration.

## Phase 1 — identify and freeze the source

Before copying data, an authorized project owner must confirm:

1. The exact target Supabase project and its project reference.
2. Whether the source contains application rows, ID files, or both; and whether there are other storage locations or email/form providers.
3. Whether writes can be paused briefly for a consistent final export, or whether a two-pass migration (initial copy, then final delta) is needed.
4. Who is authorized to access applicant records and who can approve the migration.

Record project names/references and counts only in a restricted operational record. Do not record secrets or applicant data in this repository.

## Phase 2 — back up and inventory

- Create a provider-supported, access-controlled backup of the source database and private document storage.
- Verify that the backup is readable before migrating.
- Record baseline counts: total application rows, counts by status, rows with missing/invalid document paths, and total object count in the source ID bucket. Counts must not include applicant names or identifiers.
- Preserve original application UUIDs, timestamps, statuses, notification status, and payload values where schema-compatible.
- Inventory document objects by their application association and side (front/back). Preserve original files byte-for-byte where possible; record SHA-256 checksums in a restricted migration log, not in public issues.
- Do not infer that a missing document is safe to ignore. Resolve it explicitly before cutover.

## Phase 3 — prepare the target without production cutover

The current repository defines `public.employment_applications` with JSONB `payload`, `front_storage_path`, and `back_storage_path`, plus the private `employment-id-documents` bucket.

Before importing:

- Confirm the target migration has been applied and the table/bucket exist.
- Confirm row-level security is enabled and there are no public/anon/authenticated policies exposing applications or ID objects.
- Confirm Edge Functions use server-side `SUPABASE_SERVICE_ROLE_KEY`; never put that key in browser code.
- Confirm the target administrator Auth user and `ADMIN_EMAILS` allowlist.
- Confirm function secrets such as `TURNSTILE_SECRET_KEY` are configured through the Supabase dashboard/CLI, not committed here.
- Test the schema and import procedure first with synthetic records in a non-production project.

## Phase 4 — copy records and documents

Use a controlled migration script or operator-run procedure from an authorized, secure environment. Do not put the export/import script's credentials in the repository.

1. Copy the source document objects to the target private bucket, preserving a deterministic path for each application and front/back file.
2. Verify every copied object exists, has the expected content type and size, and matches the source checksum.
3. Import application rows while preserving IDs, timestamps, statuses, notification status, payload, and the matching new storage paths.
4. Make the process restartable and idempotent; do not create duplicate rows on retry.
5. Validate row counts, status totals, document counts, object checksums, and a sample of records using authorized reviewers. Never include PII in logs.
6. Keep the source backend and backup intact until the migration has been accepted and the rollback window has expired.

If the source schema differs from the target, prepare and review an explicit field mapping before importing. Do not silently discard unknown fields or overwrite existing target records.

## Phase 5 — end-to-end test and cutover approval

On a preview/staging version only, verify:

- Valid applications with both ID documents are stored successfully.
- Missing or invalid Turnstile tokens, missing documents, unsupported file types, and files larger than 8 MiB are rejected.
- Authorized administrators can sign in, list applications, view details, open short-lived signed document links, and update statuses.
- Non-allowlisted users cannot access the admin Edge Function.
- The old and new row counts and document checksums reconcile.
- Application email configuration and sender identity are verified, if notifications are required.
- The separate visitor-relay backend and visitor scripts are unchanged.

Do not change production frontend URLs, deploy to the production project, merge the pull request, or disable the source backend until the owner explicitly approves the tested migration and rollback plan.

## Phase 6 — rollback and retention

- Keep the source backend available and unchanged through the agreed rollback period.
- Document how to restore the original frontend endpoint and source service without deleting either data set.
- If validation fails, stop cutover and investigate; do not remove source records or files.
- After owner acceptance and the retention period, handle source copies and temporary backups according to the organization's privacy and retention requirements.

## Current blocker

Repository inspection alone cannot identify which backend currently holds the live applicant rows and ID documents, nor verify a backup or source-to-target mapping. An authorized operator must identify the source project/provider and provide access through the provider's secure tools before an actual data transfer can be performed. No applicant records or files have been copied by this preparation.
