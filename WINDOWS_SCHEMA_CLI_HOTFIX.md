# Windows output follow-up — 6 November 2026

Package: `7.8_market_data_v4_4_0_schema_cli_hotfix.zip`.
Baseline: `7.8_market_data_v4_4_0_backend_logic_cli.zip`, SHA-256
`00dbd137e8a272fbd2b1f74efb8afcb0b93acce7bd6593023533450c54bfec14`.

## Confirmed user evidence

The API is running on 127.0.0.1:8000. Instrument reads and seven CASH timeframe
requests returned HTTP 200. The earlier read probe verified SBIN on 2026-10-19:
375/125/75/25/13/7/1 rows for 1m/3m/5m/15m/30m/1h/1d, plus pagination.
CASH and F&O live logs show received data and persisted candles. These observations
establish their recorded scope, not all-contract or all-history correctness.

The user's isolated Windows Python 3.13.1 check reports 873 passed, 200 skipped,
two failed: one Jobs and one Storage test. The uploaded console output contains
summary counts but not the failed test names or assertions. Those failures remain
unresolved and are not claimed fixed by this package.

## Changes

- `model/schema_compatibility.py`: ignore reflected string charset/collation
  decorators under the comparator's existing documented scope. String length,
  numeric precision, unsignedness, nullability and missing columns remain checked.
  A restricted CHECK parser recognizes utf8mb4 literal introducers and redundant
  logical grouping emitted by MySQL. It preserves operator precedence, literal
  case, predicates and IN members. Unknown grammar falls back to strict comparison.
  This changes read-only comparison; no DDL or data changes are executed.
- `controller/storage_cli.py`: after admin authentication, validate cleanup dates
  before catalog/database access. Show safe `ReadRequestError` explanations such
  as `Use ISO dates: YYYY-MM-DD`; retain generic messages for other errors.
- `tests/test_schema_compatibility.py`: regressions for real reflected collation,
  changed string length/missing columns, MySQL CHECK formatting, changed logical
  grouping, weakened predicates, changed literal case and unknown charsets.
- `tests/test_storage_cli.py`: invalid/unexpanded and reversed dates cannot access
  the database; the public reason is reported.
- `docs/BACKEND_CLI_RUNBOOK.md`: literal date examples avoid undefined CMD variables;
  existing API reuse/restart guidance. `RELEASE_NOTES.md` and `docs/CLAUDE_HANDOFF.md`
  identify this follow-up. This report and `docs/WINDOWS_HOTFIX_VALIDATION.json`
  record evidence and outstanding scope.

## Validation actually run

- Before edits: schema/CLI focused suite, 26 passed.
- After edits: the same focused suite plus regressions, 36 passed.
- Isolated affected groups on Linux Python 3.12.14: storage 329 passed / 52
  skipped; operations 143 passed / 43 skipped. Total 472 passed, 95 skipped.
  These are separate overlapping checks, not numbers to add together.
- Regression checks preserve real drift/missing-column detection and reject
  weakened CHECKs, changed grouping/literal case and unknown character sets.

No native Windows or live MySQL test was run in this cloud update. User-provided
API/live/read results are recorded above separately from these fixture tests.

## Remaining findings

1. CASH audit_log is missing ip_address and session_id according to the user's
   schema preview. Normalization does not hide missing columns. No migration applied.
2. F&O reconciliation reported SQL=385/Parquet=0 for contracts 2, 1108, 1109, 1110
   on 2026-10-19. Inspect roots/partitions and recovery evidence before any scoped
   repair. No automatic repair or data reset is included.
3. Contract ID 123 was copied from an example; the selected contract/day had zero
   candles. Use an actual master ID with known data rather than repeating that ID.
4. CASH VERIFY job 339 was PARTIAL (23 PASS, 1 WARNING, 1 NOT_VERIFIED). Its check
   details need inspection; a successful strict candle read does not clear every
   independent diagnostic/telemetry check.
5. Jobs/Storage Windows test failures need their existing logs. Extra platform
   skips are not passes. SQL/Parquet writes remain separate and are not atomic.

## Windows commands

From the updated project directory with dependencies installed, run only affected
groups first (provider-free fixtures; no production DB/provider calls):

```bat
tools\market_logic_check.cmd --groups storage operations
```

Expected: both groups pass, with platform-dependent skips shown separately. This
also reruns the Storage group whose Windows assertion is still unknown; attach
its log if it still fails. Do not treat a local Linux pass as Windows acceptance.

Keep the old report directory. From the original installation folder, gather the
two existing failure logs without rerunning tests:

```bat
set "REPORT=validation_reports\backend-7c138f8983554c9b8b9bf6d7e1b83c4b"
type "%REPORT%\jobs.log" "%REPORT%\storage.log" > backend_failures.txt
```

Inspect schema (admin authentication; previews only):

```bat
set "TIOS_HOME=F:\TIOS_Runtime"
.venv\Scripts\python.exe -m controller.storage_cli schema --segment CASH
.venv\Scripts\python.exe -m controller.storage_cli schema --segment FNO
```

Expected: formatting-only differences disappear. Real missing/incompatible objects
still produce findings and exit code 2. No DDL runs.

Use the existing API for reads. After deliberately stopping the old API with
Ctrl+C, restart once using the updated project's interpreter:

```bat
.venv\Scripts\python.exe -m api --port 8000
```

From another CMD:

```bat
.venv\Scripts\python.exe -m controller.read_probe_cli --date 2026-10-19 --cash-symbol "NSE:SBIN-EQ"
```

Expected for the previously verified unchanged scope: SCOPED_READ_CHECK_PASS.
All feature commands/effects are in BACKEND_CLI_RUNBOOK.md. Do not reset data to
address a formatting comparison or a missing CMD variable.

Status: schema/CLI corrections implemented; native acceptance, two Windows test
failures and production data/schema remediation remain partial/pending.

## Complete file inventory

Modified: `RELEASE_NOTES.md`, `START_HERE.md`, `controller/storage_cli.py`,
`docs/BACKEND_CLI_RUNBOOK.md`, `docs/CLAUDE_HANDOFF.md`,
`model/schema_compatibility.py`, `tests/test_schema_compatibility.py`,
`tests/test_storage_cli.py`.

Added: `docs/WINDOWS_SCHEMA_CLI_HOTFIX.md`,
`docs/WINDOWS_HOTFIX_VALIDATION.json`. No files deleted; UI unchanged.
