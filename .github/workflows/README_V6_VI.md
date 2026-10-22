# HỆ THỐNG 10 KÊNH V6 — POSTGRES + QUALITY GATE

## V6 changes
- Single n8n workflow, one Manual Trigger, 10 independent content branches and 10 independent schedules.
- One manual selector supports `all` or a single channel ID.
- Each branch reads its own content history from PostgreSQL (not static data).
- Fixed five branches that previously passed raw Pexels API responses directly into dispatch validation; they now build a full render payload with title/script/video URLs/channel/source metadata.
- Before GitHub dispatch, a PostgreSQL `INSERT ... ON CONFLICT DO NOTHING RETURNING` reserves the job. Duplicate `job_id`, script hash, or topic fingerprint returns no row and therefore stops that branch before dispatch.
- GitHub dispatch status update is guarded so a fast callback that already marked a job `published` is not overwritten.
- Callback updates PostgreSQL by both `job_id` and `channel`; unknown callbacks are recorded in `callback_orphans_10ch`.
- Dispatch validation checks channel/title/script/video URL, generates deterministic script/topic fingerprints, and adds source quality metadata. High-risk factual channels fail closed unless the content carries `source_verification=VERIFIED`, at least 2 source URLs and at least one allowlisted source domain. This validates metadata/domain only; it does NOT fetch and independently verify the page content.
- Workflow remains inactive after import.

## Install in this order
1. Create/use a PostgreSQL database. Run `registry_postgres_schema_v6.sql` once.
2. Import `n8n_10_kenh_V6_POSTGRES.json` into n8n.
3. Create a PostgreSQL credential in n8n and assign the same credential to every node named `DB READ HISTORY - ...`, `DB RESERVE JOB - ...`, `DB MARK DISPATCHED - ...`, and `DB UPDATE CALLBACK + ORPHAN AUDIT`.
4. Confirm the webhook `WEBHOOK - Ket qua dang Facebook` still has its Header Auth credential. Ensure GitHub Actions sends the same `job_id` and exact `channel` in callback.
5. Check all existing OpenAI, Pexels and GitHub credentials. Confirm the repo `huth-1981/video-render`, workflow `render.yml`, branch `main`, and expected dispatch input names.
6. Keep workflow inactive. Set `TEST_CHANNEL` to one channel ID and use a separate GitHub test branch/repository or disable Facebook publishing in the renderer. Do not test against a live Page unless you intentionally want a public post.
7. Verify one run end-to-end: PostgreSQL row created before dispatch; GitHub action accepted the exact job_id; output video/audio inspected; callback updates the same row; permalink points to the expected post; repeated callback does not create another job. Repeat for all 10 channels.
8. Only after all tests pass, set `TEST_CHANNEL='all'` and activate schedules.

## Channel IDs
`chuyen_tham_60s`, `sao_lai_the`, `song_khoe_sau_50`, `the_gioi_hoang_da_60s`, `loi_phat_day_1_phut`, `tien_trong_tui`, `viet_nam_nhin_tu_tren`, `su_ke_60s`, `the_gioi_ky_dieu`, `phat_minh_doi_doi`.

## Remaining operational caveats
- No live API request was executed in this build environment. The workflow needs your credentials and actual PostgreSQL connection before it can run.
- The SQL reservation prevents exact duplicate scripts and repeated topic fingerprints within a channel; semantic near-duplicate detection still relies on the channel's existing quality gate and is not a mathematical guarantee.
- The source gate validates source metadata and allowlisted hostnames, but no automated crawler/fact-checker can prove truth from a URL string alone. For health, finance, history, science and religious/historical claims, require a human or dedicated source-fetch/verification step before enabling public auto-posting.
- Pexels API result metadata is not equivalent to visual understanding. For location-specific clips, the existing Vietnam location matching gate remains; other channels should be reviewed visually during rollout.
- Retry safety: inspect `content_jobs_reconciliation_10ch` before manually retrying a failed/stale job. Do not blindly re-dispatch; GitHub might have completed even if a callback was lost.

## Existing schedule (Asia/Ho_Chi_Minh)
01:00 Lời Phật Dạy 1 Phút; 01:30 Sao Lại Thế?; 02:00 Chuyện Thấm 60s; 02:30 Việt Nam Nhìn Từ Trên; 03:00 Sống Khỏe Sau 50; 03:30 Thế Giới Hoang Dã 60s; 04:00 Tiền Trong Túi; 04:30 Sử Kể Trong 60 Giây; 05:00 Thế Giới Kỳ Diệu; 05:30 Những Phát Minh Đổi Đời.
