# Data item 3 — underlying-index collection and checks

Prepared 7 November 2026 (Asia/Kolkata).
Full package: `7.8_market_data_v4_4_0_data_phase3_underlying.zip`.
Baseline: `7.8_market_data_v4_4_0_data_phase2_session_units.zip`, SHA-256
`dccc75a3a431f10d9a90f007f64822a50a0836ec36e91419de10bbaf94e0511c`.

## What was added

`controller.underlying_cli` provides preview, run and check. It resolves selected
FNO underlying names through existing `spot_symbols` configuration to exactly one
active NSE INDEX in the CASH instrument registry. Shipped mappings:

| FNO underlying | CASH internal name | FYERS symbol |
| --- | --- | --- |
| NIFTY | NIFTY50 | NSE:NIFTY50-INDEX |
| BANKNIFTY | NIFTYBANK | NSE:NIFTYBANK-INDEX |

Unknown, missing, ambiguous, inactive or stock-instead-of-index mappings stop
before work is submitted. FINNIFTY/MIDCPNIFTY are not guessed or auto-added if their
index registry entries are absent. No new provider client/fetch engine was built.

Preview delegates to existing CASH workflow planning (no provider calls), freezes
the exact index targets/timeframes/dates and binds calendar/mapping evidence.
Run revalidates that reviewed scope and submits an existing durable WORKFLOW:
missing closed 1m FETCH, selected AGGREGATE, strict SQL/Parquet VERIFY, then READY.
Only its new job ID runs; unrelated queues are not drained/recovered. Prior job
progress and existing non-atomic SQL/Parquet failure semantics remain. READY needs
SUCCESS plus verified usable rows; it is not inferred from submission or raw counts.

Check requires no API server/provider. It reuses the strict regular-session CASH
reader and verifies each requested historical index/timeframe/day. SQL index
metadata must also match the configured active index identity. A bad/unverified
base prevents cascading derived reads. Zero index volume remains valid; traded
volume/turnover is not fabricated.

All commands authenticate admin before service/catalog access. Preview/check write
only a new requested JSON plan/report; they do not overwrite old reports. Run is
the only collection command and requires --confirm. Output never exposes raw DB
URLs, credentials or provider exception strings. A submitted job ID is printed
before processing so interrupted/partial work can be inspected.

Bounds: 31 closed historical calendar days, 20 explicitly selected underlyings,
1000 index/day/timeframe checks. Each strict day read is bounded to one API-service
page (1000 rows); larger scopes cannot silently certify pagination. Existing read
verifiers retain their own partition bounds. FNO timestamp observations are capped
at 20000 distinct timestamps per underlying/day. No all-history source-folder scan.

## Session alignment remains separate

After strict CASH 1m verification, check compares verified index timestamps to a
bounded UNION of CLOSED FNO SQL 1m timestamps for that underlying/date. It reports
unpaired samples and observations outside the configured CASH session. No index
price is forward-filled, no missing minute invented and no 15:30/15:40 policy
silently changed. This is timestamp evidence, not FNO value parity, simultaneous
live quotes, all-contract completeness, exchange-source confirmation or unit proof.

With current configuration (375 index regular minutes, 385 FNO minutes), NIFTY can
show 10 unpaired timestamps even when regular index data is fully verified. This
is a visible boundary requiring reference/session-policy review, not permission
to extend hours or refetch the same scope repeatedly. BANKNIFTY may have no FNO
observations because only NIFTY history was collected after reset.

`regular_index_data_ready` and `fno_timestamp_alignment_ready` are separate. Any
calendar change during check invalidates readiness. Old reports/jobs do not
certify data changed later. Primary session/unit evidence from item 2 remains
pending; this phase does not override it.

## Windows commands — one closed date first

Use updated code/dependencies with the existing profile. No reset/migration is
needed. The user already synced both masters; preview does not recreate them.

```bat
set "TIOS_HOME=F:\TIOS_Runtime"
.venv\Scripts\python.exe -m controller.underlying_cli preview --underlyings NIFTY BANKNIFTY --from 2026-11-05 --to 2026-11-05 --output underlying_plan_20261105.json
notepad underlying_plan_20261105.json
```

Expected PREVIEW_ONLY, correct index mappings, missing-range estimate and stages.
Review dates, targets and timeframe scope. Missing/bad registered metadata or
storage conflicts block work instead of silently replacing data. Use a fresh
filename if this one already exists.

Execute the reviewed plan (provider quota + selected CASH SQL/Parquet writes):

```bat
.venv\Scripts\python.exe -m controller.underlying_cli run --plan underlying_plan_20261105.json --confirm
```

Expected SUBMITTED with job ID, then UNDERLYING_WORKFLOW_READY and
regular_index_data_ready=true when complete. For two normal CASH index sessions,
base count is 750 combined; six derived timeframes add 492 rows (1242 across all
selected timeframes). Those are not 1242 unique base minutes. The broad historical
VERIFY command's live/checkpoint diagnostic labels do not replace strict workflow
readiness. If a stage blocks, retain the job ID and inspect it before any retry.

After successful workflow execution, inspect both index values and alignment:

```bat
.venv\Scripts\python.exe -m controller.underlying_cli check --underlyings NIFTY BANKNIFTY --from 2026-11-05 --to 2026-11-05 --output underlying_check_20261105.json
```

Expected regular_index_data_ready=true if unchanged. Under current FNO/index
windows, REGULAR_INDEX_READY_ALIGNMENT_PENDING and exit2 may be the correct
result. Read its alignment list; do not forward-fill or reset data to force green.
No FNO observations yields NO_FNO_OBSERVATIONS, not a false alignment certificate.

Run exits: 0 only for successful READY; 1 failed/unavailable command/job; 2
incomplete/unready job. Check: 0 only when regular index data and observed FNO
timestamps both pass their scoped checks; 2 findings or incomplete alignment.
INVALID_SCOPE is exit2; auth/config/setup errors remain sanitized exit1. Missing
--confirm is parser exit2. Reports/plans are never automatically overwritten.

Do not rerun a completed collection merely to generate another report. Use check
for fresh read evidence. No phase-4 history expansion, live subscriptions, option
chain capture, strategy/execution logic or UI changes were added.

## Changed files, tests and status

Added `controller/underlying_service.py`, `controller/underlying_cli.py`,
`tests/test_underlying_data.py`, this guide and `docs/DATA_PHASE3_VALIDATION.json`.
Updated `tools/market_logic_check.py`, `RELEASE_NOTES.md`, `START_HERE.md`,
`docs/CLAUDE_HANDOFF.md`. Existing generic workflow/provider/storage contracts
were reused without modification.

Baseline relevant suite: 41 passed. Focused underlying/workflow suite: 42 passed.
Final Jobs group: 107 passed, 27 MySQL-backed fixture skips. Final focused
underlying suite: 18 passed. All cloud checks used Linux Python 3.12.14; overlapping
runs are not added together. Machine-readable evidence is in the validation JSON.
Initial test-only failure attempted to mutate the frozen Instrument dataclass;
the fixture was corrected using dataclasses.replace, retaining the inactive-index
assertion. No production assertion was weakened and the original Instrument
contract was preserved.

Real SQLite/Parquet tests cover strict index values, later mutation, wrong master
metadata, zero volume, 385-vs-375 alignment, no forward fill, no SQL/source-file
writes during check, changed plans/mappings/calendar, rehashed stock-target escape,
scoped job execution, terminal readiness, authentication and exclusive reports.
The native Windows fetch/check has not run in the cloud; provider entitlements and
fresh token access are required for actual collection.

Implementation complete; native collection, reports and full FNO alignment remain
pending. Data item 4 and UI work have not been started.
