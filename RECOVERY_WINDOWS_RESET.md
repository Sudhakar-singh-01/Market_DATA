# CASH recovery Windows follow-up and authorized reset sequence

Package: `7.8_market_data_v4_4_0_recovery_windows_reset.zip`.
Baseline: `7.8_market_data_v4_4_0_schema_cli_hotfix.zip`, SHA-256
`5931918fbdd0754f0e17f943bf6b057ec372161debba361829d81cf8d589f66e`.

## Fix and evidence

The user's failing CASH coverage CLI test produced PLAN_CREATED, then
REVIEW_REQUIRED and exit 2 instead of rebuilding. Its generated backup filename
is 263 characters long on that Windows profile. On Linux the test passes normally;
injecting a 260-character open limit reproduces the same failure at the backup
stage (target_before.parquet, before SQL commit). This is concrete evidence for a
MAX_PATH failure mode; the original Windows journal was not supplied, so a native
rerun is still needed to confirm that no additional failure remains.

CASH provider repair and derived rebuild now convert already validated operation
directories to extended Windows paths. Full SHA-256 operation identities, backup
filenames, exclusive writes, root/link checks and independent SQL/Parquet recovery
evidence are retained. No registry change or data reset is needed for this fix.

CASH gap CLI now prints failed outcomes including stage, OS error and possible
SQL commit state. The isolated backend checker now prints bounded failed-test
names/tracebacks directly in the console; full logs remain in validation_reports.
This avoids having to locate old logs to report a future failure.

The original Jobs failure is not identified because its log is unavailable.
No speculative Jobs logic changes were made, and a Linux pass is not represented
as a fix of the unknown Windows assertion.

## Tests actually run

- Original failing test baseline on Linux: 1 passed.
- Injected 260-character open limit: reproduced failure in backup before SQL commit.
- Recovery/reset focused regression suite after edits: 100 passed.
- Isolated Jobs/Storage/Operations: 88/331/145 passed, respectively; 564 total
  passed, 122 skipped, no failures. Linux Python 3.12.14 only.
- Skips remain exclusions, not passes. Counts from overlapping runs are not added.
- Native Windows long-path I/O and the unknown Windows Jobs assertion need a rerun.

## Check code first — no production data access

Use the updated source folder with the project dependencies installed:

```bat
tools\market_logic_check.cmd --groups jobs storage operations
```

Expected: all three groups PASSED, or a failed test's name/traceback printed
directly after the group result. Send that failure text if Jobs still fails.
These tests use disposable fixtures; resetting production cannot fix them.

## Clean the production stores on Windows

The user has explicitly authorized cleaning both SQL stores and active Parquet.
The cloud cannot access that local installation; these commands must run there.
The existing reset utility empties the active stores while retaining recoverable
backups and moving active Parquet into an archive. It does not permanently purge
the archived copies. Do not add manual del/rmdir commands to this sequence.

Expired F&O/provider-limited history may not be fetchable again. Keep the backups
and archives even if the plan is to fetch fresh history. Checksum validation is
not a restore drill. Config/profile/credentials/quota state are retained.

1. Stop the API with Ctrl+C. Stop dashboards, live feeds, schedulers, workers and
   any external users of both databases/data folders. Keep them stopped until the
   reset returns EMPTY_SCHEMA_READY. The tool does not stop processes for you.
2. From the updated project directory, create a new preview (admin password):

```bat
set "TIOS_HOME=F:\TIOS_Runtime"
.venv\Scripts\python.exe -m controller.reset_cli preview --output reset_clean_plan.json
notepad reset_clean_plan.json
```

Expected PREVIEW_ONLY. Review database host/port, CASH/FNO database names and
Parquet roots. This resets ALL base tables in the reviewed databases, including
jobs, checkpoints, masters, manifests and audit data, then recreates repository
tables. Custom tables are backed up/dropped but not recreated. Use a fresh plan
filename if one already exists. Do not use an old preview after changing code.

3. Apply the reviewed reset:

```bat
.venv\Scripts\python.exe -m controller.reset_cli apply --plan reset_clean_plan.json --writers-stopped
```

The existing CLI asks for admin authentication and these exact acknowledgments:

```text
ACCEPT SQL DATA LOSS
RESET market_data + fno_market_data
```

Use the second phrase printed by your actual plan if database names differ.
The operation first creates and checksum-verifies BOTH backups, archives owned
Parquet, then drops/recreates the reviewed tables. Expected EMPTY_SCHEMA_READY.
No provider calls/history fetch happen automatically.

If status is RECOVERY_NEEDED, keep everything stopped. File moves and MySQL DDL
are not atomic: earlier file moves/table drops may already be complete. Inspect
the returned journal/backups/completed phases. Do not delete a recovery marker or
rerun the reset blindly. Source file locks and backup preflight failures can block
reset too. Both roots/archive must be on the same filesystem.

4. After successful EMPTY_SCHEMA_READY, inspect recreated schema and sync masters:

```bat
.venv\Scripts\python.exe -m controller.storage_cli schema --segment CASH
.venv\Scripts\python.exe -m controller.storage_cli schema --segment FNO
.venv\Scripts\python.exe main.py sync-instruments
.venv\Scripts\python.exe main.py fno-sync-contracts
```

Expected schema compatibility (inspect any findings) and instrument sync counts.
These commands alone do not create candles.

5. Fetch/aggregate/verify one bounded CASH scope before a wider refill:

```bat
.venv\Scripts\python.exe main.py smart-fetch --from 2026-11-05 --to 2026-11-05 --symbols SBIN --run
.venv\Scripts\python.exe main.py pipeline-aggregate --from 2026-11-05 --to 2026-11-05 --symbols SBIN --timeframes 3m 5m 15m 30m 1h 1d --run
.venv\Scripts\python.exe main.py verify --from 2026-11-05 --to 2026-11-05 --symbols SBIN --run
```

Use an intended completed trading date covered by the verified calendar/provider.
Expected successful jobs for complete available data; PARTIAL/NO_DATA/unknown
calendar stays a finding. Do not treat a reset as proof of data correctness.

F&O bounded fetch, after reviewing configured selection:

```bat
.venv\Scripts\python.exe main.py fno-historical --underlyings NIFTY --expiries 1 --from 2026-11-05 --to 2026-11-05
```

Then select actual master IDs with stored candles for aggregation/verification,
using BACKEND_CLI_RUNBOOK.md. Do not reuse the old placeholder contract ID 123.
Restart the API only after successful schema setup, using the updated interpreter:

```bat
.venv\Scripts\python.exe -m api --port 8000
```

## Changed files

- Added `model/storage/windows_paths.py`.
- Changed `model/cash_repair.py`, `model/cash_rebuild_apply.py`,
  `controller/cash_gap_cli.py`, `tools/market_logic_check.py`.
- Added regression coverage in `tests/test_cash_recovery.py` and
  `tests/test_market_logic_check.py`.
- Updated `RELEASE_NOTES.md`, `START_HERE.md`, `docs/CLAUDE_HANDOFF.md`.
- Added this report and `docs/RECOVERY_RESET_VALIDATION.json`.

Status: CASH Windows-path correction implemented; native Windows acceptance and
unknown Jobs assertion pending. Production reset/refetch is not executed by the
cloud. UI unchanged. Prior fixes are included in this full package.
