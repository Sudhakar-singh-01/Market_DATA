# Backend logic and CLI update

Prepared 6 November 2026. Scope: existing CASH/F&O market-data logic and commands.
UI work is paused. This update includes corrections, regression coverage and
verification commands; it does not certify an operator's existing production data.

Full source package: `7.8_market_data_v4_4_0_backend_logic_cli.zip`.
Compare against `7.8_market_data_v4_4_0_guarded_reset_tool.zip`, SHA-256
`247f8dff41d5774fdbe003e3c403e0616a4206159f6af79d54db5b9cd3830262`.
The local source checkout has no baseline commit, so review used file hashes and
file-by-file diffs against that ZIP, not an empty `git diff`.

## What changed and why

### Live CASH and F&O

- Parse cumulative volume without losing integer precision. Missing volume does
  not become a false zero baseline followed by the entire day's volume. Reject
  boolean numeric fields and ignore stale ticks within a minute.
- CASH carries cumulative-volume deltas only across consecutive minutes in the
  same day. Gaps and reconnects clear the baseline; missed trading activity is not
  assigned to the next single candle. Existing quiet-option behavior is retained
  in F&O; sparse live ticks cannot prove complete minute-level volume.
- Reconnect history accepts aligned, closed, current-session-date minutes only.
  It never imports the still-forming minute. CASH removes an in-progress bucket
  superseded by history and sends canonical accepted SQL values to aggregation.
- F&O reconnect failures retain provider/normalization/storage findings. SQL
  success followed by Parquet failure remains visible as a partial storage write.

### Historical fetch and gaps

- Enforce positive integral history chunk sizes. A provider's successful status
  with missing/malformed candle payload is an error; an actual empty list remains
  `NO_DATA`. Invalid expired-contract rows are rejected without crashing the whole
  normalizer.
- Reject candles outside the requested range, wrong-day F&O rows, unaligned CASH
  timestamps and forming CASH minutes. Do not checkpoint rejected wrong-day data.
- Preserve completed raw work when subsequent aggregation fails. Keep per-symbol
  progress separate; derived validation/conflicts produce partial outcomes.
- F&O history attaches recovery state to storage exceptions and job results:
  write started/completed, store-specific completion, affected scope and next
  action. Exception types are preserved. A failed request can still have committed
  SQL work; `FAILED` alone must not be read as “nothing was written.”
- Gap counts and tail cutoffs use actual session minutes, including split and
  special sessions, within the existing bounded history request window.

### Aggregation and verification

- CASH generation stays within the selected symbol/date scope and requires the
  applicable calendar verification. Reject non-finite, fractional or boolean
  volumes instead of silently aggregating invalid values.
- F&O aggregation/verification use session intervals rather than one continuous
  wall-clock span. Exclude breaks, combine daily bars from actual session minutes,
  and respect holidays/special sessions. Validate volume and open interest.
- Count wholly missing requested trading days as incomplete. Unknown calendars
  cannot produce verified checkpoints. Skip dates beyond contract expiry.
- Default contract selection is bounded to data/claims in the requested range;
  explicitly selected empty contracts remain inspectable. Do not scan the entire
  contract master as a substitute for a requested data scope.
- Persist independent SQL/Parquet write progress during F&O aggregation. Owned
  Parquet read handles are closed on success and exceptions, including strict
  read verification, to reduce Windows sharing failures.

### Jobs and CLI outcomes

- Synchronous `--run` claims the submitted job ID; it cannot silently consume an
  older unrelated queue item. Explicit queue workers retain queue-draining behavior.
- Terminal job states cannot be overwritten by late progress/finish calls.
  Stale-job recovery rechecks status and heartbeat in the update, so a fresh
  heartbeat or concurrent completion is not lost.
- Preserve durable progress on exceptions/interruption. F&O workers detect loss
  of running ownership where the job-store interface supports state lookup.
- CLI success/failure reflects work results: `0` for successful work, `1` for
  failures, `2` for partial/no-data/not-completed outcomes in the updated batch and
  synchronous job paths. Submission and preview commands have their own documented
  semantics; use the runbook's status table.

## Added features

1. `tools\market_logic_check.cmd`: seven independent backend test groups, isolated
   temporary source/runtime, credentials removed from child environments, blocked
   real network access and MySQL fixture skips before setup. Each group gets a log,
   JUnit file and summary counts. Failures do not suppress other groups. Timeouts
   and malformed/missing test reports cannot become a false pass. The Windows
   asyncio wakeup socket pair is allowed narrowly; arbitrary loopback DB/network
   connections remain blocked.
2. `controller.storage_cli`: authenticated schema compatibility/DDL preview,
   backup listing, backup plan creation, explicitly acknowledged backup creation,
   backup checksum verification and cleanup preview. Schema/cleanup commands do
   not apply changes. Backup creation requires writers to be stopped by the
   operator; the CLI does not stop them. Checksum verification does not certify a
   successful database restore.
3. `BACKEND_CLI_RUNBOOK.md`: Windows CMD commands and expected outcomes for live,
   history, aggregation, calendars/quality, jobs/workflows, GET API/SDK, option
   chains, instruments, diagnostics, storage, backup and recovery inspection.

## Tests actually run

Baseline/early regression failures were investigated and corrected, including
incomplete controller implementations and fixture clocks/state. Existing live
test assertions were preserved: the F&O backfill fixture now advances its clock
so seeded historical minutes are closed; CASH controller stubs initialize the
new baseline fields.

Release checks ran on Linux, Python 3.12.14. One complete seven-group run passed
with **884 passed, 190 skipped**. After the final bounded-contract selection and
Windows socket-pair regression adjustments, the affected aggregation and
operations groups were rerun successfully. Latest results per distinct group:

| Group | Passed | Skipped | Latest evidence |
| --- | ---: | ---: | --- |
| Live | 53 | 25 | Full run |
| History | 105 | 29 | Full run |
| Aggregation | 77 | 14 | Final affected-group run |
| Jobs | 88 | 27 | Full run |
| API/SDK/consumer | 100 | 0 | Full run |
| Storage/recovery | 321 | 52 | Full run |
| Operations/CLI | 141 | 43 | Final affected-group run |
| **Latest distinct total** | **885** | **190** | No repeated tests added twice |

All 30 selected CLI `--help` smoke checks returned exit code 0. The exact commands
and machine-readable run provenance are in `BACKEND_VALIDATION.json`. These help
checks validate command availability, not execution against the user's services.

The restricted host sandbox initially prevented the socket pair required by the
in-process HTTP tests. Release checks used a permitted host execution with the
runner's own real-network guard still active. Interrupted diagnostic runs are not
counted as passes. The Windows socket-pair fallback was exercised on Linux, not
on a native Windows machine.

Of the 190 skips, 185 need MySQL-backed fixtures; 5 are existing segment-specific
exclusions (FNO-only conventions/manifests). Tests exercised disposable
SQLite/Parquet and mocked providers; no production DB, broker or market dataset
was accessed. UI-specific test groups were excluded because UI work is paused.

## Remaining risks and limitations

- Native Windows Python 3.13.1, MySQL 8.0.39 and real FYERS sessions remain to be
  checked on the user's installation. Dependency deprecation warnings are not
  resolved by weakening tests or changing unrelated dependencies.
- Existing inconsistent candles, failed jobs or checkpoints are not automatically
  repaired. Start with a bounded read/verification scope and inspect findings.
- SQL and Parquet have separate commits; cross-store operations are **not atomic**.
  A failure can leave completed SQL and incomplete Parquet work. Keep the reported
  scope/progress, inspect both stores and use the documented recovery procedure.
  Do not blindly resume a different job or reset both databases.
- Live tick gaps/sparse options and provider retention/expired-contract support
  still limit available data. No fabricated bars or false complete-coverage claims
  were added. Strict calendar verification is still required for certification.
- GET verification is a point-in-time observation. Concurrent writers can change
  data later. Backups need stopped writers; checksum verification is not a restore
  drill. These operational requirements remain explicit.
- The delivery branch distributes full ZIPs and docs. Application source has not
  been merged into GitHub `main`. No UI files or production runtime configuration
  were changed by this backend update.

## Run next on Windows

Extract into a fresh folder and retain the existing permanent profile, config and
data. Use a compatible installed environment with project dependencies. From the
folder containing `main.py`:

```bat
set "TIOS_HOME=F:\TIOS_Runtime"
tools\market_logic_check.cmd
```

Expected: seven groups finish, final JSON has `"status": "PASSED"`, exit code 0.
Skips remain visible. Logs and `summary.json` go under `validation_reports`.
This command does not need the production DB or broker. Full feature commands,
their effects and expected results are in `BACKEND_CLI_RUNBOOK.md`.

**Status:** implementation and isolated regression validation complete for this
backend update; native Windows/MySQL/FYERS and production-data acceptance pending.
No new phase, UI rebuild, reset, migration or production repair was started.

## Changed files

The complete filename inventory follows; additions and modifications are relative
to the guarded-reset source package. No baseline source files were deleted.

<!-- FILE_INVENTORY -->

### Modified (31)

- `.gitignore`
- `RELEASE_NOTES.md`
- `START_HERE.md`
- `controller/fno_aggregate.py`
- `controller/fno_cli.py`
- `controller/fno_historical.py`
- `controller/fno_jobs.py`
- `controller/fno_live.py`
- `controller/fno_verify.py`
- `controller/gapfill_controller.py`
- `controller/historical_controller.py`
- `controller/live_controller.py`
- `controller/pipeline_cli.py`
- `controller/timeframe_controller.py`
- `controller/verification_controller.py`
- `docs/CLAUDE_HANDOFF.md`
- `main.py`
- `model/fno/aggregation.py`
- `model/fno/history_client.py`
- `model/fno/job_store.py`
- `model/fno/normalizer.py`
- `model/fno/read_verification.py`
- `model/fyers/historical.py`
- `model/processing/candle_builder.py`
- `model/processing/gap_detector.py`
- `model/processing/normalizer.py`
- `model/processing/timeframe_aggregator.py`
- `model/quality_store.py`
- `model/storage/mysql.py`
- `tests/fno/test_fno_live.py`
- `tests/test_idle_live_policy.py`

### Added (16)

- `controller/storage_cli.py`
- `docs/BACKEND_CLI_RUNBOOK.md`
- `docs/BACKEND_LOGIC_REPORT.md`
- `docs/BACKEND_VALIDATION.json`
- `tests/fno/test_fno_ingestion_regressions.py`
- `tests/test_aggregation_boundaries.py`
- `tests/test_backend_cli_outcomes.py`
- `tests/test_backend_read_handles.py`
- `tests/test_cash_history_regressions.py`
- `tests/test_cash_live_regressions.py`
- `tests/test_job_state_regressions.py`
- `tests/test_market_logic_check.py`
- `tests/test_storage_cli.py`
- `tools/market_logic_check.cmd`
- `tools/market_logic_check.py`
- `tools/market_logic_guard.py`
