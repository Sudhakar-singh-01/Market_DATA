> **Latest — 9 November 2026:** `7.8_market_data_v4_4_0_runtime_report_paths.zip`;
> read REPORT_PATH_HOTFIX.md. User reported masked FileNotFoundError and requested
> persistent reports under F:\TIOS_Runtime. Relative item2–3 report/plan paths now
> use TIOS_HOME; explicit absolute paths and legacy no-profile CWD behavior kept.
> No silent code-folder fallback or report migration/overwrite; old value reports
> must be placed under runtime once or passed by explicit absolute path. Probe
> shows FILE_NOT_FOUND + exact missing_path/stage. Other selected CLIs also show
> missing paths. Path tests cover stale copies/code upgrades/auth/parent creation.
> No DB/data/credential/session changes or next step/UI work. Native probe pending.

> **Earlier — 9 November 2026:** `7.8_market_data_v4_4_0_data23_recheck.zip`;
> read DATA23_COMPLETION.md. User says finish items2–3 before proceeding. New
> data23_probe_cli rechecks ONLY up to10 report-bound empty scopes, one history
> call/date each; current exact master identity required, populated/expired scopes
> skipped. No data import, retries, job/checkpoint writes, live socket or full-pass
> claim. Uploaded value reports confirmed27,619 matching rows,80 nonempty scopes,
> 6emptyIDs5608/458/468/470/472/476; allOI present, candleIV/Greeks null. Input scope
> extraction tested against uploads. Native recheck pending. Source-access draft
> successfully saved for nsearchives.nseindia.com/myapi.fyers.in; requires_publish,
> NOT applied/published. Prior no-automatic-publish instruction preserved. Official
> source/session/unit and live proof pending; UI/next step held.

> **Earlier — 7 November 2026:** `7.8_market_data_v4_4_0_data_offline_fno_verify.zip`;
> read `DATA_OFFLINE_FNO_VERIFY.md`. User authorized non-live verification and next
> offline work. New read-only diagnostics `fno-values` checks bounded historical
> selected values/manifest/coverage/optional availability with opt-in per-run
> shared-file cache; consumer API requests remain uncached. Native command results
> are pending. User Windows job11 and fresh report verified both indexes 14/14,
> 1,242 rows. BANKNIFTY job6 harvested12,277 rows,38 verified scopes,5NO_DATA.
> Both underlying timestamp unions have385 slots,375 paired,10 extra15:30–15:39.
> Source/unit evidence still blocked by official-host proxy403; live/UI work held.
> Do not call items2–3 fully complete or repeat successful index fetching/reset.

> **Earlier — 7 November 2026:** package
> `7.8_market_data_v4_4_0_data_phase3_underlying.zip`; read `DATA_PHASE3_UNDERLYING.md`.
> User selected data item 3. New underlying CLI resolves configured FNO spot symbols
> to exact active CASH indices and delegates collection to the existing reviewed
> CASH workflow. It runs only the submitted job ID. Check is read-only strict
> regular-session CASH values, with separate FNO SQL timestamp union comparison.
> Configured 385 FNO vs375 CASH minutes can leave10 unpaired slots; no price carry
> forward, calendar extension or false full alignment. Native collection pending.
> Item4/UI not started. Phase2 native audit completed without numeric/timestamp
> errors: CASH375 rows, FNO15342 rows; all FNO rows had non-null valid OI, current
> lot65 and tick0.05/0.10 observations. No source/unit proof inferred. Five CASH
> symbols/indexes still empty and contract5608 NO_DATA until scoped collection.

> **Latest — 7 November 2026:** package
> `7.8_market_data_v4_4_0_data_phase2_session_units.zip`; read
> `DATA_PHASE2_SESSION_UNITS.md`. User explicitly selected data item 2. New offline
> calendar and bounded SQL numeric/unit audit never equates local verified flags,
> existing source citations or lot divisibility with independent proof. Existing
> FAOP75472 timing citation means do NOT assume 385 minutes is wrong. A fresh
> circular request failed with proxy/HTTP 403; independent confirmation pending.
> No calendar/data/schema changes or lot/IV rescaling applied. Numeric parser fixes
> preserve large integer quantities, reject fractional/bool/nonfinite quantities,
> reject nonfinite optional decimals and boolean historical OI. Native phase-2
> reports pending. Stop before item 3/UI work unless requested.
> User's phase-1 Windows inventory completed: CASH only SBIN populated (621 rows),
> five selected CASH symbols/indexes empty. NIFTY 43 contracts, 15342 base +759
> derived rows, 37 fully observed grids, 5 sparse puts and 1 NO_DATA. Derived data
> only on future 10/options5569/5570. No timestamp/status mismatches or read errors;
> values were not re-certified by inventory.

> **Latest — 6 November 2026:** package
> `7.8_market_data_v4_4_0_data_phase1_inventory.zip`; read `DATA_PHASE1_INVENTORY.md`.
> The user requested adding pending data features one by one. Item 1 adds bounded
> read-only counts/date/key inventory to diagnostics_cli, not full OHLCV parity or
> a new fetch/recovery workflow. UI remains paused. Do not start item 2 automatically.
> User evidence since the hotfix: Jobs now passes on Windows (89/27 skipped),
> reset returned EMPTY_SCHEMA_READY with verified retained backups; CASH/FNO
> recreated schemas compatible, masters synced. SBIN 2026-11-05 fetched 375 base
> bars and all 6 derived TFs matched. Broad CASH VERIFY is PARTIAL for checkpoint,
> legacy-job evidence and live telemetry, not candle mismatch. NIFTY history:
> 43 requests, 15342 stored, 42 verified scopes, 1 NO_DATA (5608,
> NSE:NIFTY26N0925250PE). Futures ID 10 full verify: 38 PASS. Options 5569/5570:
> 72 PASS plus 2 blanket option minute-coverage NOT_VERIFIED labels despite 385
> observed / zero missing each. Date-scoped FNO reconciliation passed. These are
> bounded observations; do not certify all dates/contracts or post-reset live.

> **Latest update — 6 November 2026:** package
> `7.8_market_data_v4_4_0_windows_jobs_hotfix.zip`; read `WINDOWS_JOBS_HOTFIX.md`.
> User's updated Windows output: Storage 322 passed/61 skipped, Operations 144
> passed/44 skipped; CASH failure no longer occurs. Jobs failed during isolated
> subprocess startup because its minimal env omitted SYSTEMROOT (WinError 10106).
> Fix is test-only: allow OS bootstrapping variables, retain stripped app credentials
> and existing isolation assertions. Native Windows Jobs rerun pending. Reset
> preview succeeded; apply stopped BEFORE execution because user typed ACCEPT
> rather than ACCEPT SQL DATA LOSS. Do not claim production was reset.

> **Latest update — 6 November 2026:** package
> `7.8_market_data_v4_4_0_recovery_windows_reset.zip`; read `RECOVERY_WINDOWS_RESET.md`.
> CASH repair backup path was 263 characters. A simulated MAX_PATH limit reproduces
> the same failure; extended Windows operation paths are now used. Native rerun
> remains pending. Backend failures now print tracebacks in console. Original
> Jobs failure remains unknown; the user cannot locate the old log. The user has
> explicitly authorized clearing production SQL/Parquet; only commands are provided
> here because the cloud cannot access the user's Windows installation. No reset
> has been run. Preserve backups/archives and do not claim old expired F&O history
> can necessarily be fetched again.

> **Latest follow-up — 6 November 2026:** read `WINDOWS_SCHEMA_CLI_HOTFIX.md`.
> Latest package is `7.8_market_data_v4_4_0_schema_cli_hotfix.zip` on the same
> delivery branch. It supersedes the backend ZIP below. API execution and CASH
> scoped strict reads now have user-supplied Windows evidence. Two Windows test
> failures still lack tracebacks; F&O parity and genuine missing CASH audit columns
> remain pending. No production migration/repair was run by this update.

> **Current continuation update — 6 November 2026:** UI work is paused by the user.
> Backend logic fixes, regression tests, an isolated validation runner and a storage
> CLI are now included. Read `BACKEND_LOGIC_REPORT.md`, `BACKEND_CLI_RUNBOOK.md` and
> `BACKEND_VALIDATION.json` first. These supersede the artifact/status statements
> below for this release; earlier sections remain architectural history.
>
> Latest full source package:
> https://github.com/Sudhakar-singh-01/Market_DATA/raw/refs/heads/codex-download-20261030/7.8_market_data_v4_4_0_backend_logic_cli.zip
>
> The adjacent `.zip.sha256` file records its checksum. The delivery branch holds
> artifacts; no application-source merge into `main` is implied. Baseline for this
> update is the guarded-reset ZIP, SHA-256
> `247f8dff41d5774fdbe003e3c403e0616a4206159f6af79d54db5b9cd3830262`.
> Latest per-group evidence: 885 passed and 190 skips (185 MySQL-backed and 5
> segment-specific) on Linux
> Python 3.12.14. Native Windows/MySQL/FYERS behavior still needs the bounded checks
> in the runbook. No production data was reset, ingested or repaired in this work.

> Addition after the red/white handoff: the user requested a reset file.
> A guarded optional reset utility now exists; see `docs/GUARDED_RESET.md`.
> Only creation/testing of the utility was performed. No production reset was
> run. Default `tools/reset_all.cmd` only previews; apply is separately authenticated
> and explicitly confirmed, creates fresh backups, and archives Parquet before
> dropping/recreating empty schemas. Do not treat its presence as permission for
> an agent to reset production or as evidence that reset fixes UI/file-lock bugs.

# TIOS / FYERS Market Data — continuation handoff for Claude

Prepared 5 November 2026 (Asia/Kolkata). This is a continuation guide, not an
instruction to run recovery or write to production. User decisions below override
older architecture documents when they conflict. Read the latest code alongside
this file; historical release notes are evidence for their stated scopes only.

## 1. Start here: authoritative artifact and repository state

Repository identity: `Sudhakar-singh-01/Market_DATA`.

Latest code package:

https://github.com/Sudhakar-singh-01/Market_DATA/raw/refs/heads/codex-download-20261030/7.8_market_data_v4_4_0_ui_red_white_hotfix.zip

SHA-256:

`52043cc6e9f1f251c7f1d1b8c78f37cdd9db2f7162a58da31773bdcf23d7e3c5`

The package contains the full `Market_DATA/` source, tests, safe configuration
examples, documentation and sample UI screenshots. It excludes actual `.env`,
virtual environments, caches, runtime logs, databases and market-data directories.
It includes all earlier audit/UI fixes, not just the last diff.

Artifact publication branch: `codex-download-20261030`. Latest code-artifact
publication commit: `a59834f`. That branch is a DOWNLOAD/DELIVERY branch holding
ZIPs and standalone preview/docs, not the normal application source tree.
Do not assume GitHub `main` has received the code changes from these ZIPs.

In the current cloud workspace, actual source is `/workspace/Market_DATA`.
Its local `work` branch has no commits and source files are untracked. The
artifact-publishing clone is `/tmp/market-hotfix-download`. Therefore a blank
`git diff` is NOT evidence of no changes. Compare source against the previous ZIP
or inspect files explicitly. Do not run `git clean`, reset, overwrite a user's
checkout, or invent source commit IDs. If continuing in a new environment, extract
the authoritative ZIP into a fresh directory and inspect its Git state first.

Current standalone links:

- Feature mapping: https://github.com/Sudhakar-singh-01/Market_DATA/blob/codex-download-20261030/UI_REBUILD_FEATURE_MAP.md
- Red/white preview: https://github.com/Sudhakar-singh-01/Market_DATA/blob/codex-download-20261030/UI_REBUILD_PREVIEW.png
- Hotfix limitations: https://github.com/Sudhakar-singh-01/Market_DATA/blob/codex-download-20261030/UI_RED_WHITE_HOTFIX.md

Preview screenshots contain explicitly labeled SAMPLE tasks. They do not show
real account jobs and do not certify Windows execution.

## 2. Product, user and scope

This is an existing personal, single-user, local Windows market-data application.
Enhance it; do not rebuild the backend from scratch.

Implemented product scope:

- CASH equities/indices and FNO futures/options market-data ingestion.
- Historical and live paths, normalization, cleaning, validation and aggregation.
- Separate MySQL + Parquet stores, checkpoints, persisted jobs, gap planning.
- Stored-data GET API, Python SDK, strict consumer adapter and SMA replay example.
- Database/schema/catalog inspection and guarded administrative operations.
- Backup, isolated recovery staging/verification and scoped cleanup/recovery tools.
- Streamlit UI, recently reorganized into nine workspaces with a multi-task panel.

Explicitly deferred: Event Bus, full production indicator engine, strategies,
backtesting/forward testing, paper/live trading and real-money orders. Existing
SMA is a historical integration example, not proof of a completed trading engine.
There is no automatically authorized Phase 11 or other trading-platform expansion.

The user prefers concrete changes and downloadable full ZIPs, one scope at a time.
They do not always have access to the Windows computer; run meaningful tests in
cloud where possible, consolidate necessary Windows commands, and do not request
repeated full runs without a new change or unresolved failure. Explain actual
limitations plainly. Do not fabricate diagnoses from summary JSON alone.

## 3. Operator environment and non-negotiable rules

Known operator environment:

| Item | Value |
| --- | --- |
| OS / Python | Windows, Python 3.13.1 observed in supplied outputs |
| Database | MySQL 8.0.39 reported by user |
| CASH DB | `market_data` |
| FNO DB | `fno_market_data` |
| Permanent profile | `TIOS_HOME=F:\TIOS_Runtime` |
| CASH Parquet | `F:\TIOS_Runtime\data\cash` |
| FNO Parquet | `F:\TIOS_Runtime\data\fno` |
| Code | Frequently extracted into new folders; never use folder changes to reset state |

User Windows drives are not accessible from cloud. No production DB connection,
provider request, migration or historical repair was run by this agent while
implementing the audit/UI changes. Operator logs do show their own live usage.

Rules to preserve:

1. Never reset/empty production databases, delete runtime state, or run destructive
   migrations to make tests pass. Preserve existing good data and public contracts.
2. Keep CASH/FNO databases, roots, identities and checkpoints separate. Respect
   nested-root ownership guards as well as explicitly separated roots.
3. Never overwrite existing profile config, credentials or instrument customizations
   with package defaults. Never print or package secrets.
4. Reuse FYERS authentication, quota/retry and socket ownership modules. Do not add
   a parallel broker client or implicit authentication flow.
5. MySQL and Parquet are NOT one atomic transaction. Report completed/failed/pending
   work and recovery state accurately. MySQL DDL is not transactionally rolled back.
6. Do not weaken tests, fabricate candles, fill gaps with zeros, or label sparse
   options complete. No successful count alone certifies parity/quality.
7. No unnecessary repeated full-history scans or replay of prior production repairs.
8. Windows commands must be individual, single-line CMD commands. Paths with spaces
   must be quoted. Do not paste activation scripts onto pytest command lines.
9. When production actions would be required, prepare the scoped plan/evidence
   before requesting confirmation. Implementation authorization is not permission
   to apply repairs to production.

## 4. Architecture and important entry points

Python + Streamlit + FastAPI + SQLAlchemy/PyMySQL + pandas/PyArrow/Parquet.
YAML configuration, plain SQL schema/migrations; there is no separate SPA frontend.
Core requirements are in `requirements.txt`; API/client requirements are split into
`requirements-api.txt` and `requirements-client.txt`. Optional DB tests require a
disposable MySQL-protocol server. Do not blindly upgrade a working FYERS SDK.

Main entry points:

| Entry | Responsibility |
| --- | --- |
| `main.py` | Existing ingestion, workers, migration/verification/FNO CLI commands |
| `ui/dashboard.py` | New authenticated Streamlit workspace shell |
| `python -m api` | Loopback authenticated GET API |
| `controller/workflow_cli.py` | Admin-reviewed workflow preview/run and wait |
| `controller/read_probe_cli.py` | Scoped stored-data GET integration probe |
| `python -m client` | SDK export CLI |
| `tools/phase9_validation.py` | Provider-free core/API/operations acceptance |
| `tools/ui_rebuild_check.py` | Isolated UI + recent live-regression acceptance |

Backend ownership:

- `controller/market_data_service.py`: command facade over existing CASH/FNO services.
- `controller/pipeline_service.py`: CASH preview, submissions, operations and workers.
- `controller/unified_workflow.py`: frozen-scope FETCH/AGGREGATE/VERIFY/READY orchestration.
- `controller/fno_jobs.py`, `model/fno/job_store.py`: persisted FNO queue, claims,
  heartbeats, cancellation, recovery and outcomes.
- `model/quality_store.py`: CASH job/report metadata and existing explicit migration.
- `controller/historical_controller.py`, `model/fyers/historical.py`: CASH history.
- `controller/live_controller.py`, `model/processing/candle_builder.py`: CASH live.
- `controller/fno_historical.py`, `controller/fno_live.py`, `controller/fno_chain.py`:
  FNO history/live/chain paths.
- `controller/timeframe_controller.py`, `model/processing/timeframe_aggregator.py`:
  CASH aggregation and strict completeness.
- `controller/fno_aggregate.py`, `model/fno/aggregation.py`: FNO derived rebuild.
- `model/storage/mysql.py`, `model/storage/parquet.py`, `controller/sinks.py`:
  CASH storage and cross-store fan-out state.
- `model/fno/repository.py`, `model/fno/release3_repository.py`,
  `model/fno/parquet_store.py`: independent FNO SQL/Parquet and manifests.
- `model/quality.py`, `model/verified_reads.py`,
  `controller/verification_controller.py`, `model/fno/read_verification.py`:
  validation, parity, session and completeness evidence.
- `model/config/market_calendar.py`, `model/timeframes.py`: calendar/timeframe rules.
- `model/config/runtime_paths.py`, `model/storage_scope.py`: stable runtime/root policy.

## 5. Runtime configuration and storage contracts

`TIOS_HOME` is opt-in and points at an already initialized persistent profile:
`config/`, `instruments/`, `data/cash`, `data/fno`, `runtime/`, `logs/`, `backups/`,
`recovery/`, and default exports. Absolute overrides remain absolute; relative
mutable paths anchor to the selected profile, or the installed code root when no
profile is configured. They do not follow the launcher's current working directory.
A missing configured profile/required config fails closed; it is not replaced with
an empty environment. Do not rerun `runtime_cli init` on the existing profile.

FYERS credentials, DB credentials and password hashes stay in the user's existing
profile/configuration. Packages include `config/.env.example`, not real `.env`.
Review operator YAML updates (including 3m) rather than overwriting customizations.
See `docs/PERMANENT_RUNTIME.md`, `docs/AUDIT_PHASE_3_RUNTIME_PATHS.md`.

CASH Parquet: `raw/1m/<symbol>/<date>.parquet` and
`processed/<tf>/<symbol>/<date>.parquet`.
FNO candle partitions: `candles/<tf>/underlying=<U>/expiry=<date>/date=<date>.parquet`,
containing multiple contract IDs. FNO chain snapshots use their own partitions and
batch/contract identities. FNO manifest status/checksums are part of verified reads.

SQL stores IST wall-clock timestamps; public timestamps include `+05:30`.
Decimal SQL values serialize as strings to preserve precision. Optional fields
remain null. Do not invent OI/Greeks: live FYERS samples did not supply OI; historical
OI and stored chain Greeks are separate sources. Earlier provider observations and
entitlements are scoped, not universal guarantees.

Canonical provider data is 1m; derived timeframes are 3m, 5m, 15m, 30m, 1h, 1d.
CASH and FNO session/business rules remain distinct. Configured normal CASH session
has 375 minutes; FNO has 385 minutes (09:15–15:39 starts, close 15:40). Verified
special/split sessions override normal expectations. CASH buckets restart per
session interval; daily candles combine valid intervals without manufacturing
break minutes. FNO's final clipped 3m bucket can be shorter. Full normal sessions
produce 125 CASH and 129 FNO 3m bars.

Manual historical requests can run off-market/weekends/holidays; execution time
and requested trading date are different concepts. Manual live connection off-market
is allowed by the later idle-live policy. Empty/status-only feed messages must not
create market-data rows/files. Unknown calendar evidence is not invented. Later
`docs/IDLE_LIVE_POLICY.md` overrides older session-gating descriptions.

Historical authoritative CLOSED data may supersede provisional LIVE data under
existing merge policy. Conflicting same-authority CLOSED history is not silently
replaced. Strict derived aggregation requires valid unique CLOSED source minutes;
incomplete inputs do not become fabricated bars. SQL-accepted canonical rows feed
the second sink; partial failures expose recovery state.

## 6. Jobs and workflows

Persisted states: PENDING, RUNNING, SUCCESS, PARTIAL, FAILED, NO_DATA, CANCELLED,
INTERRUPTED. Jobs deduplicate active scopes, publish counters/heartbeats, support
boundary-based cancellation and recover stale RUNNING records. Do not infer a
worker is alive from the stored state alone. Shutdown cannot forcibly cancel all
blocking provider I/O.

Unified workflow freezes exact targets/dates/timeframes in a reviewed preview:
FETCH -> optional AGGREGATE -> strict VERIFY -> READY.

- Both SQL and Parquet required.
- Closed historical dates, maximum 31 inclusive days, 200 targets, 5000 target-days.
- CASH internal-name/FYERS identity and FNO contract-ID/symbol mappings rechecked.
- FNO selection/ATM scope is frozen; preview can request a spot quote explicitly.
- READY requires SUCCESS, ready=true, complete verification and nonempty usable data.
- A holiday-only empty scope is NO_DATA, not a false READY.
- Sparse option coverage can block workflow readiness even if returned values match.
- READY is point-in-time job evidence; consumers must still use verified reads.
- FAILED/PARTIAL/INTERRUPTED can explicitly resume as new linked jobs; CANCELLED
  workflows require a fresh reviewed run. Never blindly replay uncertain submissions.
- UI workers are process-local; closing a tab does not necessarily stop them, but
  stopping the host does. Headless workflow CLI waits and returns a meaningful exit.

CASH/FNO data sockets share account ownership rules. Reconnect re-subscribes current
symbols before gap handling. Reconnect history excludes the still-forming reconnect
minute. A genuinely stuck provider close retains ownership and blocks overlap.
Existing provider SDK reconnection is reused, not replaced with a second socket loop.

## 7. GET API, SDK and consumers

`api/app.py` authenticates existing user/admin password hashes via HTTP Basic.
Default deployment is loopback; remote TLS/authentication deployment is separate.
The API is GET-only and never triggers fetch, migration, repair or job submission.
The command facade's role argument is an internal trusted caller contract, not a
remote authentication substitute.

Endpoints under `/api/v1`:

- instruments; candles; candles/latest; coverage; fno/option-chain; jobs/{job_id}.
- Admin: quality; databases; database table lists, schemas, rows and counts.

`model/read_contracts.py`: request validation, JSON/Decimal conversion, scope-bound
cursors and result envelope. `model/market_read_service.py`: read behavior.
`model/database_catalog.py`: registered DBs, exact discovered table names,
parameterized scalar equality filters and bounded stable browsing; no arbitrary SQL.

- Raw reads are NOT_VERIFIED. Server default is raw; SDK historical default is verified.
- Verified reads require historical closed scope and strict SQL/Parquet/calendar
  evidence; derived values are recomputed/checked from canonical minutes.
- Verified CASH `session=regular` checks full-store parity and integrity BEFORE
  filtering valid stored off-session rows. Exclusion is a response view, not deletion.
- Raw/regular CASH filters SQL before pagination and reports observed exclusions.
  FNO `session=regular` is not implemented/certified; FNO all-session rules remain.
- Page max 1000 candles; latest max 100. Raw date scope max 3661 inclusive days,
  verified/regular max 31, coverage max 366. Admin offset is bounded.
- Candle cursors are timestamp keysets; other entity pages use ID keysets. Queries
  are a live view, not cross-request snapshots. Verified pages intentionally recheck
  their scope; there is no unsafe verification cache introduced for speed.
- SDK bounds retries, response bytes/pages, validates scope/order/overlap/cycles,
  checks requested row limits and optionally enforces latest-data freshness.
- `client/consumer.py`: bounded immutable Decimal batches; incomplete or malformed
  inputs fail closed. `indicators/sma.py` and `examples/indicator_check.py` demonstrate
  historical SMA replay with explicit warm-up and candle-close availability.
- `client/read_probe.py`: bounded scoped GET probe across all seven CASH timeframes,
  pagination, optional FNO contract and stored option-chain sample. No broker calls.

## 8. Maintenance and prior recovery tools

Existing guarded features include schema compatibility/preview/apply with journals,
backup inventory and checksums, recovery staging to a new location, manual import
into isolated `tios_recovery_*` DBs, exact schema/typed-row-multiset verification,
and reviewed scoped cleanup retaining originals/recovery journals.

Backup verification checks archive inventory/checksums, not market-data validity or
successful restore. Recovery staging does not perform a live SQL import/cutover.
MySQL schema changes and SQL/Parquet publication are not claimed atomic.

CASH recovery modules/CLIs already exist for legacy inventory, validation preview,
conflict audit, SQL-exact restore, provider candidate staging/repair, session gaps,
regular derived rebuilds, apply journals and final checks. See `model/cash_*.py`,
`controller/cash_*_cli.py` and their matching docs. These are potentially mutating
operator tools, not a list to run during onboarding.

Prior source reports mention successful scoped production repair/restoration and
derived rebuilding. Treat those as historical evidence, not authorization to repeat
repairs or current full-store certification. Existing delete-first CASH rebuild
entry point remains disabled; do not re-enable it casually.

## 9. Audit phases and implementation history

The numbered audit sequence was completed incrementally; each phase is limited to
its documented scope. It is not a certificate that every possible issue is fixed.

| Phase | Concrete result | Primary docs |
| --- | --- | --- |
| 0 | Baseline and operator evidence; test/platform limits recorded | AUDIT_PHASE_0_BASELINE.md |
| 1 | Durable FNO workflow in-flight progress/result JSON and heartbeat while RUNNING | AUDIT_PHASE_1_DURABLE_JOBS.md |
| 2 | Structured completed/failed/pending sink and partition recovery state; raw parity before derived rebuild; FNO safe-stop checkpoint boundary | AUDIT_PHASE_2_STORAGE_CONSISTENCY.md |
| 3 | Profile/project-anchored runtime, logging, exports and health paths | AUDIT_PHASE_3_RUNTIME_PATHS.md |
| 4 | 3m support across CASH/FNO aggregation, contracts, clients, workflow, CLI, cleanup/rebuild scopes | AUDIT_PHASE_4_CALENDAR_3M.md |
| 5 | Strict backup manifests, link rejection and extracted staging re-verification | AUDIT_PHASE_5_DATABASE_RECOVERY.md |
| 6 | Idempotent socket stop, stuck-close ownership, restart blocking, re-subscription and reconnect-minute exclusion | AUDIT_PHASE_6_LIVE_RECONNECT.md |
| 7 | Bounded service iterator/cursor-cycle checks, SDK row-limit checks, single-clock latest metadata | AUDIT_PHASE_7_GET_API.md |
| 8 | Canonical server timeframe definitions; compatibility aliases retained | AUDIT_PHASE_8_SAFE_REFACTORING.md |
| 9 | Isolated Windows acceptance runner, real platform fixes and confirmed Windows PASSED | AUDIT_PHASE_9_WINDOWS_E2E.md |
| 10 initial | Workflow input/status/recovery guidance and navigation improvements | AUDIT_PHASE_10_UI.md |
| UI replacement | User-approved nine-workspace shell, named views, new reader, all-source task center, full feature mapping | UI_REBUILD.md, UI_FEATURE_MAP.md |
| Latest hotfix | Red/white theme, live CSV collision regression, FNO Windows publication handling | UI_RED_WHITE_HOTFIX.md |

Important Phase 9 corrections:

- Read-probe integration fixture omitted real 3m bars while the probe already checked
  3m; positional assertions were stale. Fixed fixture and checked results by name.
  Earlier chat claims of a production JSON/result-format defect were incorrect.
- Windows cleanup fsync required writable handles; fixed actual service code.
- Real symlink fixtures skip only Windows privilege error 1314. Independent guards
  and unrelated overlap/mixed-partition tests remain active.
- Source-inspection tests explicitly read UTF-8 instead of the Windows default codepage.
- ZIP reader normalization can hide raw backslashes/NUL spellings. Verify original
  `ZipInfo.orig_filename` and reject normalization before manifest checks; do not
  weaken the unsafe-name tests.

## 10. Current UI and proper feature mapping

Latest user instruction: replace old UI, map useful features correctly, and show
ALL observed in-progress tasks together in a panel at the top right. User then
explicitly requested RED AND WHITE, superseding the initial dark theme.

Current theme: `.streamlit/config.toml` uses light base, white background, deep-red
`#b91c1c` accents, pale-red `#fff1f2` secondary surfaces and dark text. `ui/shell.py`
uses matching CSS. Do not change only one and leave mismatched dark/light overrides.

Old `ui/pages/` numeric routes and `ui/home.py` were removed. There is no old sidebar
alongside the new one. Existing useful screens/controls were migrated to named
`ui/views/*.py` and embedded in the new workspaces; they were not all rewritten
from scratch. Backend code and confirmation/role gates were preserved.

Nine workspaces:

1. Overview: optional system/CASH evidence and FNO summary, without automatic full scans.
2. Historical Data: reviewed unified workflow, CASH smart fetch/gap fill/aggregation,
   FNO historical/aggregation sections.
3. Live Market: CASH live engine and FNO live/chain controls.
4. Data Explorer: new bounded verified/raw candle reader; retained SQL tables, FNO
   contracts/chain and complete advanced FNO tools.
5. Coverage & Quality: CASH findings, coverage/gaps, telemetry and FNO quality.
6. Jobs: unified workflow monitor shortcut, CASH ingestion/checkpoints/operations,
   FNO jobs/logs/checkpoints.
7. Instruments: masters/timeframes/calendar and FNO futures/options.
8. Storage & Recovery: storage/files/catalog, DB/schema, backup/staging, cleanup,
   isolated recovery verification and retained advanced tools.
9. Settings & Diagnostics: configuration/advanced CRUD, logs/API inspector,
   data-flow information, audit/security and FNO settings.

`ui/feature_map.py` is the machine-readable inventory of all 29 retained views,
roles and FNO section destinations. `docs/UI_FEATURE_MAP.md` contains every old
entry -> new workspace -> new path plus backend ownership. Tests assert exact
view inventory coverage, all nine valid workspaces, admin access and parity across
all 12 FNO subsections. Update mapping and tests when adding/removing views.

New UI code:

- `ui/dashboard.py`: authenticated entry, new navigation, shell-owned logout.
- `ui/shell.py`: layout/theme and persistent task panel attachment.
- `ui/workspaces.py`: lazy workspace routing and explicit stored-candle reader.
- `ui/feature_map.py`: inventory and role-filtered tools.
- `ui/task_center.py`: provider-free task federation and foreground activity registry.
- `ui/task_panel.py`: escaped task cards, 5-second fragment refresh, tracked spinners.
- `ui/views/`: named guarded domain views; old relative depth preserved.
- `ui/ux_helpers.py`: old one-job/live pills suppressed in new shell; task_guard
  integrates with the shared task center rather than a duplicate overlay.

Reader: explicit first/next click, 200 rows/page, current scope bound to stored
cursor/results, changed scope discards old page, JSON export retains quality.
Do not bypass `MarketReadService` strict verification for UI convenience.

## 11. Task center semantics and limits

Every observed task gets its own card. Desktop panel stays at top right; long
lists scroll. At smaller widths it becomes inline above main content to avoid
covering controls. It does not collapse multiple tasks into an opaque +N summary.

Sources: CASH/FNO persisted PENDING/RUNNING queue rows of all job types; legacy
local JobManager jobs; CASH LiveRunManager; automatic gap scanner; tracked
foreground UI scopes (preview/backup/recovery/schema/cleanup/read operations).

Persisted queries are SELECT-only, keyset-paged by 200, without arbitrary active-row
cutoff. They deliberately omit params/results/error blobs. IDs are source/segment
namespaced so CASH/FNO/local job #1 remain distinct. Available counters, source,
heartbeat and cancellation flag are shown. Source outages are explicit; no failed
read silently becomes zero tasks. Strings are HTML-escaped. No invented overall %.

Limits: stored RUNNING is not proof of live worker health; refresh is a live view,
not a snapshot. Same-session polling can pause during synchronous work, although
its initial task card is drawn before blocking. Local activity does not survive
host restart. Completed short tasks disappear; durable history remains in Jobs.
External commands appear only if they record into the configured job tables.
Arbitrary unregistered machine processes cannot be inferred. Do not claim universal
OS/process monitoring. A test federates 906 observed task cards; that is a
correctness check, not a production-scale performance benchmark.

## 12. Latest live failures and fixes — read before continuing

The operator's original rebuilt-UI Windows tests PASSED (203/4 skipped), but their
subsequent live use exposed two additional contextual failures.

A. Duplicate CSV widgets

`ui/pipeline_page.py` rendered smart-fetch preview rows plus a populated job
fragment; both called `ui.quality_page.records()` with identical download metadata.
Streamlit raised StreamlitDuplicateElementId. Current code passes stable,
context-specific CSV keys for preview/job/history/finding sections. The real
widget regression reproduced the exact exception before the fix and now passes.
Do not use random keys as a workaround or assume a previously empty job fixture
covers populated preview+job rendering.

B. FNO Windows sharing violation — existing job is unresolved

Observed: FNO job #12, AGGREGATION, failed at os.replace while publishing:

`candles/5m/underlying=BANKNIFTY/expiry=2026-10-27/date=2026-10-22.parquet`

Runtime root reported by the operator: `F:\TIOS_Runtime\data\fno`.
Windows error 32 reported a file in use. The traceback alone does not identify the
process holding the file or prove that only this partition needs recovery. Earlier
job units may have completed and SQL work for this unit may already have committed.
Do not claim the data is repaired by installing new code.

Current `model/fno/parquet_store.py` changes:

- `_read_frame`: owned stream closes after pandas/PyArrow reads.
- Candidate writes own an exclusive writable stream, flush/fsync, close, then validate.
- `_replace_candidate`: same candidate, at most 5 attempts, ONLY winerror 32/33;
  delays 0.1/0.2/0.4/0.8 seconds. Other permission errors fail immediately.
- If publication never happened, previous target stays and previous VERIFIED
  manifest is restored when appropriate. Failure after replace is not falsely
  rolled back or marked verified.
- A failed temporary unlink is logged without masking the publication failure;
  retained temp files remain diagnostic evidence.

Tests cover transient retry, persistent lock/old-file+manifest preservation,
non-sharing immediate failure and owned-handle closure. Persistent external locks
can still exhaust retries. Do not disable file validation, force delete the target,
or suppress job failure to make this green.

User logs also showed stale jobs 9/10/11 marked INTERRUPTED, job 13 RECONCILE PARTIAL,
and job 14 VERIFY INTERRUPTED during shutdown. These are observations, not reasons
to reset all jobs. Review exact findings/scope before proposing any operator action.
Index zero-volume warnings in the live log are not proof of data corruption.

## 13. Test evidence: separate platforms, scopes and dates

Never add overlapping suite counts together or treat skipped tests as passes.

User-supplied native Windows evidence:

- Earlier baseline: core 98 passed; original API pair 40 passed.
- Completed Phase 9 corrected runner: core 100, API 42, operations 389 passed /
  74 skipped -> total 531 passed, 74 skipped, zero failures, final PASSED.
- Initial Phase 10 focused UI/workflow suites: 44 passed.
- Rebuilt UI before red/white hotfix: 203 passed, 4 skipped; final PASSED.
- That same old UI later hit the live bugs above; the passing tests did not prove
  those missing contexts were safe.

Latest red/white hotfix cloud evidence (Linux, Python 3.12.14):

- `python -m tools.ui_rebuild_check`: 215 passed, 13 optional skips; final PASSED.
- `python -m tools.phase9_validation`: core 100 passed; API 42 passed;
  operations 404 passed, 64 skipped; final PASSED.
- Local Chromium desktop and small-window previews inspected with labeled sample
  tasks. These are not Windows/real-account screenshots.
- Latest corrected hotfix has NOT yet received native Windows confirmation.

Earlier phases also have recorded disposable-MariaDB runs (e.g. Phase 2 parity
and lifecycle/path suites). They were isolated scratch databases. Do not extrapolate
those old runs into current MySQL 8.0/live-provider acceptance.

Test runner isolation:

`tools/phase9_validation.py` creates a temporary TIOS_HOME, copies non-secret
settings/instrument declarations, redirects temporary storage, strips DB/FYERS/
password and inherited storage/runtime overrides, and runs fail-fast groups with
the current interpreter. `tools/phase9_pytest_guard.py` explicitly skips FNO MySQL
fixtures before setup. `tools/ui_rebuild_check.py` reuses this isolation with its
own UI-specific manifest including the new live regressions and Parquet tests.

Direct FNO integration tests use `FNO_TEST_DB_HOST/PORT/USER/PASSWORD` and scratch
`market_data_test` / `fno_market_data_test`. Fixtures can create/migrate/delete
scratch objects. NEVER point them at production or provide production credentials
just to eliminate skips.

Cloud sandbox restrictions can block sockets/cross-thread TestClient wakeups.
Readiness/network policy and escalation may be needed. Standard HTTP tests later
ran successfully outside those restrictions; do not permanently claim API tests
cannot run anywhere in cloud. Use bounded diagnosis; do not weaken assertions or
hang forever with repeated polling. Environments previously failed entirely;
if a file cannot be read, say so rather than invent its contents.

## 14. Safe commands for a fresh code folder (Windows CMD)

Use the user's existing configured environment where appropriate. If creating a
new environment, run each command separately from the extracted project root:

```bat
py -3.13 -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt -r requirements-api.txt -r requirements-client.txt pytest
.venv\Scripts\python.exe -m tools.ui_rebuild_check
.venv\Scripts\python.exe -m tools.phase9_validation
.venv\Scripts\python.exe -m streamlit run ui/dashboard.py --server.address 127.0.0.1
.venv\Scripts\python.exe -m api --port 8000
```

Dashboard/API commands are long-running separate processes; use separate CMD
windows. Test commands are provider-free fixture checks under the runners.
Expected acceptance result is final PASSED with no failures; optional skip counts
vary with fixture availability and Windows symlink privileges. Dependency deprecation
warnings are not automatic test failures and do not justify uncontrolled upgrades.
Restart Streamlit after theme/config changes so it loads `.streamlit/config.toml`.
Do not merge old `ui/pages/` files back into the new UI. Preserve existing TIOS_HOME
rather than copying blank/default credentials into it.

## 15. Recommended continuation order

1. Read this guide, `docs/UI_REBUILD.md`, `docs/UI_FEATURE_MAP.md`,
   `docs/UI_RED_WHITE_HOTFIX.md`, and inspect the latest artifact's actual code.
2. Confirm the operator is using the red/white hotfix ZIP and get the native Windows
   acceptance result without unnecessarily rerunning all old phases.
3. If UI issues remain, obtain exact traceback + interaction context; preserve
   top-right simultaneous tasks and all mapped features. Add realistic regressions.
4. For failed FNO job #12, inspect existing persisted job/result and affected
   SQL/Parquet/manifest scope in an authorized read-only manner. Prepare a narrow
   recovery/resume proposal. Do not automatically execute it or scan all history.
5. Review any further UI changes with the user. Scope a new feature explicitly;
   do not start Event Bus/trading engines because old architecture docs mention them.

Implementation workflow expected by the user: inspect affected callers, baseline
relevant tests, small reversible changes, meaningful regression coverage, changed-
file review, full safe package + exact Windows commands, actual results and clear
remaining risks. Report complete/partial/blocked honestly. Do not ask the user for
permission that is already present, but do not expand into production recovery.

## 16. Copy/paste starter instruction for the next Claude session

Read CLAUDE_HANDOFF.md and extract the latest verified red/white hotfix ZIP. Treat
that artifact as the current source, not an assumed GitHub main checkout. Inspect
current code and preserve existing runtime/profile/data. First summarize what is
verified and what remains pending. Continue only the user's next authorized task.
The latest code fixes the live CSV collision and hardens Windows Parquet publication;
existing failed FNO job #12/data is not repaired. Never reset databases, overwrite
profile configuration, rerun historical recovery, weaken tests or claim cross-store
atomicity. Keep all nine workspaces and simultaneous top-right task visibility.
