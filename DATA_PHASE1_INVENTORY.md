# Data item 1 — bounded read-only inventory

Latest full package: `7.8_market_data_v4_4_0_data_phase1_inventory.zip`.
Baseline: `7.8_market_data_v4_4_0_windows_jobs_hotfix.zip`, SHA-256
`d313bbc460b5f49a2b5cc5bbcc919482503f8b31391184c6834e3ca2474e97da`.

## What was added

An admin-authenticated, headless `inventory` command in the existing diagnostics
CLI. This implements ONLY item 1 of the user's data plan. It does not start UI
work, change exchange sessions, refill history or add live/Greeks collection.

For an explicit closed historical date range and targets, it reports:

- Registered symbols/contracts and their first/last SQL timestamps WITHIN scope.
- SQL/Parquet row counts, CLOSED/OPEN counts and duplicate timestamp counts.
- Observed base days and expected-session days without candles.
- Unobserved configured-calendar slots, with bounded timestamp samples.
- Physical Parquet presence/read errors and timestamp/status key agreement.
- FNO manifest status and overlapping checkpoint statuses (including NO_DATA).
- Complete/partial inspection status, issue counts and explicit resource limits.

Selecting FNO underlyings considers only contracts with candles or checkpoint
claims intersecting the requested dates. It does not inventory all 9551 contract
master records as though they have data. Explicit IDs can inspect an empty contract.
Shared FNO Parquet files are projected/read once and reused for selected contracts.
Owned read handles close even on errors. Unsafe roots, links and partition paths
are rejected. Report errors never include database URLs/passwords.

No provider client is constructed, jobs are not submitted, SQL is not mutated and
market files are not rewritten. An optional new JSON OUTPUT report is written only
at the requested path. Existing report files are never overwritten.

## Evidence boundaries

This inventory inspects identity/timestamps/status, NOT OHLCV/OI values, manifest
checksums, indicator readiness, exchange session correctness or trading readiness.
`values_verified=false` stays explicit even when counts/keys match. Use existing
strict reads/verification commands to check value parity. A VERIFIED manifest
status is recorded metadata, not a fresh checksum certificate.

Unobserved option slots do not prove missing trades. Both stores can agree on an
empty scope without having usable candles; this is FINDINGS, not success. No day
is inferred to be a holiday merely because candles are absent. The expected slots
come from the CURRENT configured calendar; data-plan item 2 still needs its own
calendar/unit review. Concurrent reads are not an atomic SQL/Parquet snapshot.

`complete=true` means the requested inspection completed, NOT that the dataset has
complete history. First/last timestamps are not all-time data-depth claims.
Counts summed over all selected timeframes are not unique base-minute counts.

## Limits

31 inclusive calendar days, 50 targets, 1000 target/day/timeframe partitions.
SQL: at most 20000 rows per partition and 500000 per store per run (a single extra
sentinel can be read to detect exceeding the bound). Parquet: 200000 physical rows
per file and 500000 total projected rows. FNO checkpoint metadata is bounded to
10000 records. Exceeded read bounds or a changing/unreadable file yield PARTIAL,
never a false pass. Excess target/date scope is rejected as INVALID_SCOPE.

A large shared file can exceed a file bound even for one selected contract. This
phase deliberately reports that limitation rather than silently scanning it all.
There is no recursive all-history filesystem scan and no data cache persisted
between runs. Reuse each report; rerun only after data/scope changes warrant it.

## Windows commands — current dataset

From the extracted updated project with dependencies installed:

```bat
set "TIOS_HOME=F:\TIOS_Runtime"
.venv\Scripts\python.exe -m controller.diagnostics_cli inventory --segment CASH --symbols SBIN RELIANCE HDFCBANK ICICIBANK NIFTY50 NIFTYBANK --from 2026-11-05 --to 2026-11-05 --output cash_inventory_20261105.json
.venv\Scripts\python.exe -m controller.diagnostics_cli inventory --segment FNO --underlyings NIFTY --from 2026-11-05 --to 2026-11-05 --output fno_inventory_20261105.json
```

Admin password is prompted. Use your registered CASH internal names/FYERS symbols
and actual selected underlyings. The named CASH list reflects the shipped YAML;
adjust it if your local instrument configuration differs. Output filenames must
be new; choose a suffix for a deliberate later run.

Based on the user's existing results, FINDINGS is expected for unfilled CASH
symbols/indexes and unbuilt FNO derived scopes. Contract 5608 should remain visible
with NO_DATA metadata. Do not infer data loss from those expected inventory gaps.
There is no automatic refetch/aggregation/repair.

A smaller previously checked scope:

```bat
.venv\Scripts\python.exe -m controller.diagnostics_cli inventory --segment FNO --contract-ids 10 5569 5570 --from 2026-11-05 --to 2026-11-05 --output selected_fno_inventory.json
```

That scope is expected to have matching observed keys/counts if unchanged. For
base-data inventory only, add `--timeframes 1m`. Omitting --output prints full JSON
to the console. With --output, the console prints a compact summary and path.

Status/exit codes:

| Status | Code | Meaning |
| --- | ---: | --- |
| SCOPED_INVENTORY_PASS | 0 | Requested counts/keys checks completed without findings; values still unverified |
| FINDINGS | 2 | Inspection completed, but data/calendar/manifest/key/coverage findings exist |
| PARTIAL | 2 | Some inspection could not complete; preserve the partial report |
| INVALID_SCOPE | 2 | Invalid dates/selection/bounds or an existing/non-JSON output path |
| UNAVAILABLE | 1 | Authentication/configuration or report-file/DB setup unavailable |

Do not reset or refill solely because exit code is 2. Read the report's issues.

## Code/tests and continuation

- Added `model/data_inventory.py` and `tests/test_data_inventory.py`.
- Extended `controller/diagnostics_cli.py`; existing commands/contracts preserved.
- Added inventory tests to `tools/market_logic_check.py` Operations group.
- Updated `RELEASE_NOTES.md`, `START_HERE.md`, `docs/CLAUDE_HANDOFF.md`.
- Added this guide and `docs/DATA_PHASE1_VALIDATION.json`.

Tests cover disposable SQLite/real Parquet, read-only SQL/source bytes, empty days,
shared-file reuse, NO_DATA target selection, unknown calendars, exact configured
timeframe counts, corrupt/changing files, bounds, links, auth and exclusive output.
Baseline related tests: 23 passed. Final focused inventory/diagnostics/scope suite:
40 passed. Operations group: 162 passed, 43 MySQL-fixture skips on Linux Python
3.12.14. These runs overlap; do not add counts together.
See the adjacent validation JSON for actual runs; native Windows acceptance and
user-production inventory have not been performed in the cloud.

Optional native isolated code check:

```bat
tools\market_logic_check.cmd --groups operations
```

Status: item 1 implemented with isolated regression coverage; native report outputs
pending. Item 2 and later data items have NOT been started. UI unchanged.
