# Backend commands — CASH and F&O

Run from the extracted `Market_DATA` directory in Windows CMD. Use the existing
Python environment and permanent profile. No UI process is needed. This guide
describes command effects; it does not run ingestion, repairs, migrations or reset.

```bat
set "TIOS_HOME=F:\TIOS_Runtime"
```

Keep your existing profile `.env`, YAML files, instruments and runtime quota state.
Do not initialize the profile again. A fresh code folder needs its own `.venv` or
your existing compatible interpreter; do not overwrite the profile with defaults.

The CMD launcher requires `.venv\Scripts\python.exe` inside this code folder.
If that file is missing, create an environment from your installed Python 3.13
and install all three dependency sets before the first run:

```bat
py -3.13 -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt -r requirements-api.txt -r requirements-client.txt
```

This installs dependencies and needs package-network access. Skip it when the
existing environment is already ready. Alternatively, use your compatible
interpreter's full quoted path with `-m tools.market_logic_check`; the CMD launcher
itself does not search other environments. Do not copy a `.venv` between machines.

**1. One-command code verification (start here)**

```bat
tools\market_logic_check.cmd
```

Expected: seven groups, final `"status": "PASSED"`, exit code 0. It creates
`validation_reports\backend-<unique-id>\summary.json`, plus one `.log` and JUnit
`.xml` per group. Failures do not hide the other independent groups. A group that
times out returns 124 in the JSON; the command returns 1 if any group is incomplete
or failed. Default timeout is 600 seconds per group; use `--timeout 1200` if needed.

The runner copies source into a temporary directory, excludes installation `.env`
and runtime data, removes broker/database credentials from the child environment,
blocks real network connections in pytest, and skips MySQL-backed fixtures before
setup. Tests use mocks and disposable SQLite/Parquet files. `PASSED` certifies
these tests, not your production data or live broker connection. Skips remain
visible. Native MySQL and Windows acceptance are separate evidence.

List exact test files without executing them:

```bat
tools\market_logic_check.cmd --list
```

Rerun only an affected group when needed:

```bat
tools\market_logic_check.cmd --groups live
tools\market_logic_check.cmd --groups history
tools\market_logic_check.cmd --groups aggregation
tools\market_logic_check.cmd --groups jobs
tools\market_logic_check.cmd --groups api
tools\market_logic_check.cmd --groups storage
tools\market_logic_check.cmd --groups operations
```

**2. Configuration, databases and storage checks**

```bat
.venv\Scripts\python.exe -m controller.diagnostics_cli storage --offline
.venv\Scripts\python.exe -m controller.storage_cli schema --segment CASH
.venv\Scripts\python.exe -m controller.storage_cli schema --segment FNO
.venv\Scripts\python.exe -m controller.diagnostics_cli storage
```

Offline returns `CONFIGURATION_ONLY`, with DB/manifest checks skipped. Schema
commands prompt for admin authentication and report missing tables, suggested DDL
and compatibility. They never apply DDL: exit 0 means compatible under the
documented comparison, 2 means findings. Storage checks manifest path safety and
file existence; `PASS` here does not establish SQL/Parquet value parity.

Optional dependency health checks:

```bat
.venv\Scripts\python.exe main.py health --skip-fyers
.venv\Scripts\python.exe main.py health
```

Health checks create/remove small writability probe files; the second command also
contacts FYERS for authentication. `HEALTHY` means those dependencies were reachable,
not that market candles are complete.

**3. Choose a small, real data scope**

The following example date is a closed trading day. Change it to a date you intend
to inspect. CASH commands using `--symbols` take configured internal names;
API/SDK commands take exact FYERS symbols. Discover the current FNO contract symbol
and ID from the contract master before using them; examples are not live symbols.

```bat
set "TRADE_DATE=2026-10-19"
```

**4. Stored-data GET API, SDK, instruments and chain**

Start the API in a separate CMD, with the same `TIOS_HOME`:

```bat
.venv\Scripts\python.exe -m api --port 8000
```

Expected: listening on `http://127.0.0.1:8000`. From another CMD, `curl.exe -u admin`
prompts for the admin password; do not put passwords into command lines:

```bat
curl.exe --fail-with-body -u admin "http://127.0.0.1:8000/api/v1/instruments?segment=CASH&limit=20"
curl.exe --fail-with-body -u admin "http://127.0.0.1:8000/api/v1/instruments?segment=FNO&active_only=true&limit=20"
curl.exe --fail-with-body -u admin "http://127.0.0.1:8000/api/v1/coverage?segment=CASH&symbol=NSE:SBIN-EQ&from=%TRADE_DATE%&to=%TRADE_DATE%"
curl.exe --fail-with-body -u admin "http://127.0.0.1:8000/api/v1/fno/option-chain?underlying=NIFTY&from=%TRADE_DATE%&to=%TRADE_DATE%&limit=20"
set "MARKET_DATA_API_URL=http://127.0.0.1:8000"
set "MARKET_DATA_API_USER=admin"
.venv\Scripts\python.exe -m client --segment CASH --symbol "NSE:SBIN-EQ" --timeframe 1m --from-date %TRADE_DATE% --to-date %TRADE_DATE% --quality verified
```

These requests read existing data only. An empty chain response is not evidence
of successful capture. The SDK requests strict SQL/Parquet verification. HTTP 409
means `DATA_NOT_VERIFIED`; inspect the findings instead of treating it as an API
failure or automatically refetching. For sparse options, `--allow-sparse` explicitly
permits incomplete coverage; it never turns it into complete coverage.

One password, bounded CASH checks across every timeframe and pagination:

```bat
.venv\Scripts\python.exe -m controller.read_probe_cli --date %TRADE_DATE% --cash-symbol "NSE:SBIN-EQ"
```

To include FNO, set `FNO_SYMBOL` to the exact stored contract symbol from the API
response, then run:

```bat
.venv\Scripts\python.exe -m controller.read_probe_cli --date %TRADE_DATE% --cash-symbol "NSE:SBIN-EQ" --fno-symbol "%FNO_SYMBOL%" --underlying NIFTY
```

Expected: `SCOPED_READ_CHECK_PASS`, or `FINDINGS` with the failing checks. Reports
are under the profile's `recovery\read_checks`. Missing data stays a finding.
The probe does not fetch, repair, delete or rewrite candles. Optional scopes that
you omit are not certified. A normal complete session has CASH 375 base minutes
and FNO 385; special sessions use the configured intervals. Matching counts alone
do not establish correctness.

**5. Historical fetching (writes market data and consumes provider quota)**

Preview missing CASH data without submitting work:

```bat
.venv\Scripts\python.exe -m controller.diagnostics_cli fetch-preview --segment CASH --from %TRADE_DATE% --to %TRADE_DATE% --symbols SBIN
```

After reviewing the scope, choose ONE CASH fetching path:

```bat
.venv\Scripts\python.exe main.py smart-fetch --from %TRADE_DATE% --to %TRADE_DATE% --symbols SBIN --run
```

Smart Fetch requests missing closed minutes and stops on storage conflicts. The
legacy full selected-range path also derives configured timeframes:

```bat
.venv\Scripts\python.exe main.py historical --from %TRADE_DATE% --to %TRADE_DATE% --symbols SBIN
```

FNO selection uses the configured active universe, ATM/strike window and provider
entitlements. Review those settings first; this can select multiple contracts:

```bat
.venv\Scripts\python.exe main.py fno-historical --underlyings NIFTY --expiries 1 --from %TRADE_DATE% --to %TRADE_DATE%
```

The command prints its estimate and job ID and runs only that job. `--submit-only`
queues it without fetching history, but plan preparation can authenticate/contact
the provider for spot quotes. Empty/malformed responses, unavailable expired
history and incomplete quality evidence must not be interpreted as complete data.
Historical checkpoints record ingestion progress; strict reads/workflow READY
provide the stronger completeness checks.

**6. Aggregation from stored canonical 1m (writes derived data)**

```bat
.venv\Scripts\python.exe main.py pipeline-aggregate --from %TRADE_DATE% --to %TRADE_DATE% --symbols SBIN --timeframes 3m 5m 15m 30m 1h 1d --run
```

For one exact FNO contract, set `FNO_ID` to its actual numeric master ID:

```bat
.venv\Scripts\python.exe main.py fno-aggregate --from %TRADE_DATE% --to %TRADE_DATE% --contract-ids %FNO_ID% --timeframes 3m 5m 15m 30m 1h 1d
```

These commands use stored base candles. Complete closed buckets produce bars;
missing/invalid/forming source minutes produce PARTIAL findings. FNO replaces the
selected derived partitions under the existing lock policy, so stop overlapping
writers. SQL and Parquet commits remain independent. Inspect `current_write` or
`storage_failures` if interrupted; no automatic rollback is claimed.

**7. Detailed quality verification (market data read; job/findings metadata written)**

```bat
.venv\Scripts\python.exe main.py verify --from %TRADE_DATE% --to %TRADE_DATE% --symbols SBIN --run
.venv\Scripts\python.exe main.py fno-verify --from %TRADE_DATE% --to %TRADE_DATE% --contract-ids %FNO_ID%
.venv\Scripts\python.exe main.py fno-reconcile --from %TRADE_DATE% --to %TRADE_DATE%
```

The first two persist their diagnostic jobs/reports, not candle repairs. Broad
diagnostics may legitimately return PARTIAL/NOT_VERIFIED for sparse options or
unobserved live telemetry. `fno-reconcile` without `--repair` diagnoses only; its
date scope covers FNO partitions, not just the selected ID. Use the GET probe for
checks that must avoid all SQL writes.

Set `JOB_ID` to the actual printed ID, then inspect/export:

```bat
.venv\Scripts\python.exe -m controller.diagnostics_cli job --segment CASH --id %JOB_ID%
.venv\Scripts\python.exe -m controller.diagnostics_cli job --segment FNO --id %JOB_ID%
.venv\Scripts\python.exe main.py quality-export --job %JOB_ID% --output cash_quality_report.json
```

Use the matching segment command. Job-query exit 0 means the record was read;
inspect its `state`. CASH `quality-export` is for VERIFY jobs and replaces the
chosen output report; choose a fresh filename if retaining an older report.

**8. Live CASH/FNO and option-chain capture (provider access + writes)**

Run one selected live stream at a time under the existing deployment's socket
ownership policy. Stop with Ctrl+C before switching CASH/FNO:

```bat
.venv\Scripts\python.exe main.py live --symbols SBIN
.venv\Scripts\python.exe main.py fno-live --underlyings NIFTY --expiries 1 --max-subscriptions 20
.venv\Scripts\python.exe main.py fno-chain-snapshot --underlyings NIFTY
```

The live commands block while running; the chain command captures one batch.
Manual idle connections are allowed off-market; an idle connection is not proof
that ticks arrived or candles persisted. Inspect actual tick/candle log times and
the stored latest values while the market is sending data:

```bat
curl.exe --fail-with-body -u admin "http://127.0.0.1:8000/api/v1/candles/latest?segment=CASH&symbol=NSE:SBIN-EQ&timeframe=1m&limit=5"
curl.exe --fail-with-body -u admin "http://127.0.0.1:8000/api/v1/candles/latest?segment=FNO&symbol=%FNO_SYMBOL%&timeframe=1m&limit=5"
```

Latest reads show SQL observations, not cross-store certification. Follow with
scoped verification of completed data. Live volume after a mid-session join or
reconnect is provisional; authoritative history can supersede it. CASH closes a
bucket on the next minute's tick or stop; no wall-clock flush timer was added.
Real provider reception, entitlement, native Windows file sharing and MySQL
behaviour still need an operator run; fixture tests cannot certify them.

**9. Jobs, worker lifecycle and scheduling**

```bat
.venv\Scripts\python.exe main.py pipeline-jobs
.venv\Scripts\python.exe main.py fno-jobs --list
```

These list jobs. The following are operational actions, not read-only checks:

```bat
.venv\Scripts\python.exe main.py pipeline-jobs --cancel %JOB_ID%
.venv\Scripts\python.exe main.py fno-jobs --cancel %JOB_ID%
.venv\Scripts\python.exe main.py pipeline-jobs --resume %JOB_ID%
.venv\Scripts\python.exe main.py fno-jobs --resume %JOB_ID%
```

Resume creates a new linked job for FAILED/PARTIAL/INTERRUPTED work. CANCELLED is
terminal. Review retained progress and storage findings first. Single submission
commands run only their new job; queue workers below intentionally run all eligible
pending jobs, so inspect the queue before starting one:

```bat
.venv\Scripts\python.exe main.py pipeline-worker --once
.venv\Scripts\python.exe main.py fno-jobs --run
```

Continuous workers/schedulers, for deliberate unattended operation:

```bat
.venv\Scripts\python.exe main.py pipeline-worker --schedule-eod --symbols SBIN
.venv\Scripts\python.exe main.py fno-jobs --worker
.venv\Scripts\python.exe main.py schedule
```

Choose the appropriate host, not all three. `schedule` manages CASH market-session
live/EOD activity; pipeline EOD enqueues durable jobs. Do not run a second owner for
the same stream. Boundary-based cancellation cannot interrupt every blocking
broker call immediately.

**10. Unified FETCH → AGGREGATE → VERIFY workflow**

Create a new JSON parameter file with the intended closed date(s) and actual
targets. CASH example:

```json
{"from":"2026-10-19","to":"2026-10-19","symbols":["SBIN"]}
```

Save it as `backend_cash_params.json`, then:

```bat
.venv\Scripts\python.exe -m controller.workflow_cli preview --segment CASH --params backend_cash_params.json --timeframes 1m 3m 5m 15m 30m 1h 1d --output backend_cash_plan.json
.venv\Scripts\python.exe -m controller.workflow_cli run --preview backend_cash_plan.json --confirm
```

Run only after reviewing the plan. FNO uses the same commands with `--segment FNO`
and the selection JSON described in `docs/UNIFIED_WORKFLOW.md`. READY requires
SUCCESS + `ready=true` with strict complete evidence. It is valid at verification
time; later writes may change the data. Sparse option scopes may remain blocked.

**11. Instrument masters and exports**

Master sync changes instrument metadata; FNO may download the master:

```bat
.venv\Scripts\python.exe main.py sync-instruments
.venv\Scripts\python.exe main.py fno-sync-contracts
```

Stored CASH CSV export:

```bat
.venv\Scripts\python.exe main.py export --symbol SBIN --timeframe 1m --from %TRADE_DATE% --to %TRADE_DATE% --output cash_1m_export.csv
```

This reads Parquet and writes the requested CSV; use a new filename. For strict
verification and a no-overwrite JSONL export, use the SDK's `--output` option.
Provider capability diagnostics are available as `main.py fno-probe`; it performs
real provider calls and is not part of the offline acceptance runner. Rate-limit
and websocket-limit stress probes remain explicit opt-ins.

**12. Backup, recovery and scoped cleanup preview**

New headless backup commands reuse the existing guarded BackupService. Stop all
SQL/Parquet writers and schema tools before creating the plan/backup and keep them
stopped until creation completes. Run one segment at a time:

```bat
.venv\Scripts\python.exe -m controller.storage_cli backup-preview --segment CASH --output cash_backup_plan.json
.venv\Scripts\python.exe -m controller.storage_cli backup-create --plan cash_backup_plan.json --writers-stopped
.venv\Scripts\python.exe -m controller.storage_cli backup-preview --segment FNO --output fno_backup_plan.json
.venv\Scripts\python.exe -m controller.storage_cli backup-create --plan fno_backup_plan.json --writers-stopped
.venv\Scripts\python.exe -m controller.storage_cli backups
```

`backup-preview` creates a new plan file only; creation validates it against the
current scope. Use a new filename for a new preview. Creation returns a backup ID,
checksum and `CHECKSUM_VERIFIED`, with `restore_tested=false`. This does not certify
restoration or data parity. Set `BACKUP_ID` to a returned ID to recheck its archive:

```bat
.venv\Scripts\python.exe -m controller.storage_cli backup-verify --id %BACKUP_ID%
.venv\Scripts\python.exe -m controller.recovery_cli --list-staged
.venv\Scripts\python.exe -m controller.storage_cli cleanup-preview --segment CASH --symbol "NSE:SBIN-EQ" --from %TRADE_DATE% --to %TRADE_DATE% --timeframe 1m
```

Cleanup preview never deletes. Recovery verification requires a separately staged
archive and isolated restored database; follow `docs/RECOVERY_VERIFICATION.md`.
Existing scoped recovery/repair CLIs remain documented in `CASH_RESTORE.md`,
`COMBINED_CASH_RECOVERY.md`, `CASH_PROVIDER_REPAIR.md`, `CASH_DERIVED_REBUILD.md` and
`CASH_FINAL_CHECK.md`. No automatic reset, recovery apply, retention deletion or
full-history rescan is part of this backend verification workflow.

**Exit codes and interpreting results**

After any command, run this as a separate CMD line:

```bat
echo %ERRORLEVEL%
```

Historical/aggregation execution: 0 successful, 1 failed, 2 partial/empty/pending.
FNO duplicate submissions retain exit 3. A queue-only command returning 0 means
submission succeeded, not job completion. General parser/validation errors can
also return 2; read the message. Workflow run returns 0 only for READY, otherwise
2 for a terminal unready result. Diagnostic GET/job-query success is separate
from quality/job state. Warnings and skipped tests are never counted as passed.

When a group fails, retain `summary.json` and that group's `.log`; the JSON alone
cannot explain an assertion failure. Stop at the reported scope, inspect completed
work, and verify SQL/Parquet before resuming a partial write. Resetting all data is
not a prerequisite for any code change in this package.
