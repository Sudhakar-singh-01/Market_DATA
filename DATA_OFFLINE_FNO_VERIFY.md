# Offline historical F&O value verification

Prepared 7 November 2026 (Asia/Kolkata). UI remains on hold.
Full package: `7.8_market_data_v4_4_0_data_offline_fno_verify.zip`.
Baseline: `7.8_market_data_v4_4_0_data_phase3_underlying.zip`.

The user authorized remaining non-live verification and the next steps that do
not require an open market. This update adds a bounded historical FNO value audit
over existing strict readers, rather than interpreting successful ingestion or
timestamp alignment as value/storage verification. It needs no API server, broker
token, network connection, live ticks, reset, migration or active market.

## New command

`controller.diagnostics_cli fno-values` checks selected registered contract IDs or
contracts with stored candles/checkpoint claims for a selected underlying/date.
It never treats the entire active contract master as an expected history universe.

- Reuses existing FNO strict verification: SQL/Parquet keys and values, manifest
  VERIFIED status, checksum/row count, duplicates, candle identity/status/session,
  OHLCV/optional numeric validity, canonical minute coverage and, if requested,
  aggregation equality against canonical 1m data.
- Reports null/present historical OI, IV, delta, gamma, theta and vega counts from
  verified stored rows. Zero OI is present. Unknown fields remain null. This is
  availability evidence, not unit confirmation or inferred live OI.
- Complete futures require all declared session minutes. Sparse options can have
  matching verified values while `complete_minute_coverage=false`. Empty sessions
  with NO_DATA claims are not certified as no trading or as complete price history.
- One immutable in-memory cache per audit reads/hashes each valid shared Parquet
  file once, with subsequent version/manifest checks. Rejected unchanged files are
  not repeatedly checksummed for each contract. The cache is not reused by later
  CLI invocations or consumer API requests.
- At the end, recheck cached source versions/manifests and calendar metadata;
  detected changes invalidate the entire audit's successful checks. This is not a
  transactional SQL/Parquet snapshot. Concurrent SQL changes after a particular
  read are not excluded; results are sequential point-in-time evidence.
- Admin authentication precedes catalog access. JSON output uses exclusive create;
  existing files are preserved. Diagnostics use allowlisted reasons, not raw
  provider/database exception strings.

Bounds: 31 historical calendar days, 50 target contracts (existing selector), 500
contract/day/timeframe checks, 100 shared files, 200,000 rows per Parquet file,
500,000 total scanned Parquet rows, and 500,000 SQL rows plus at most one existing
20,000-row contract-partition sentinel. Read-bound exhaustion is PARTIAL, not PASS.
NIFTY and BANKNIFTY should be checked separately because their combined current
selection can exceed 50 targets. This does not certify contracts outside the
explicit/data-and-claim selection.

## Windows CMD — current production data, read-only

Use the updated source with the existing interpreter, dependencies and profile.
No new dependency, profile copying, schema change or market-open wait is needed.

```bat
set "TIOS_HOME=F:\TIOS_Runtime"
.venv\Scripts\python.exe -m controller.diagnostics_cli fno-values --underlyings BANKNIFTY --from 2026-11-05 --to 2026-11-05 --output banknifty_values_20261105_v1.json
.venv\Scripts\python.exe -m controller.diagnostics_cli fno-values --underlyings NIFTY --from 2026-11-05 --to 2026-11-05 --output nifty_values_20261105_v1.json
```

Each command prompts for the admin password and defaults to canonical 1m only.
Choose a new filename on subsequent checks. Send both JSON reports for review.
Do not reset/refetch because a valid sparse/NO_DATA scope prevents an overall PASS.

Expected status meanings:

| Output | Meaning / exit |
| --- | --- |
| FNO_VALUES_CHECK_PASS | All selected contract/day/timeframe values matched and checked, exit 0 |
| VERIFIED_SPARSE per check | Stored option values matched; minute coverage remains SPARSE_UNCONFIRMED |
| FINDINGS | One or more selected scopes could not be certified, exit 2 |
| NO_CANDLES_OBSERVED reason | Empty session, including NO_DATA claims; not proof of zero trading |
| PARTIAL | Inspection incomplete, source/calendar changed or read budget exhausted, exit 2 |
| INVALID_SCOPE | Invalid selectors/dates/output or excessive target/check scope, exit 2 |
| UNAVAILABLE | Authentication/configuration/bootstrap/report failure, exit 1 |

The user's last BANKNIFTY harvest reported 43 contracts, 12,277 stored candles,
38 verified ranges and 5 NO_DATA; NIFTY previously reported 43 contracts, 15,342
stored candles, 42 verified ranges and 1 NO_DATA. These are ingestion reports,
not this new value audit. NO_DATA claim scopes stay visible, so FINDINGS with some
NO_CANDLES_OBSERVED reasons may be appropriate. Actual counts must come from the
new native JSON reports, not this expectation.

Optional derived verification is for an explicit scope whose derived data exists;
it does not create missing aggregation. For example, previously aggregated IDs:

```bat
.venv\Scripts\python.exe -m controller.diagnostics_cli fno-values --contract-ids 10 5569 5570 --from 2026-11-05 --to 2026-11-05 --timeframes 1m 3m 5m 15m 30m 1h 1d --output fno_selected_derived_values_20261105_v1.json
```

Those IDs are from the user's current database; do not assume they identify the
same symbols after another reset/master rebuild. No repeat of this already-checked
scope is necessary solely to collect duplicate evidence.

## Items 2 and 3: verified evidence and remaining gates

The latest uploaded Windows index report was reviewed: both NIFTY and BANKNIFTY
have 7/7 verified timeframes, 621 rows per index, 1,242 total. Both now have 385
distinct FNO SQL timestamps; exactly 10 (15:30–15:39 IST) lack index pairing and
all are outside the configured CASH session. Regular-session timestamps pair.
This is a timestamp union, not all-contract/FNO-value/live synchronization proof.

Official NSE session-source and FYERS unit evidence remain pending. The current
cloud policy rejected official NSE/FYERS document requests with proxy CONNECT 403.
No bypass, arbitrary unit conversion, calendar extension, synthetic index price or
false verified flag was introduced. Source content or permitted official access
is required to close those gates. Existing 375-CASH/385-FNO declarations remain.

Live reception/freshness/reconnect, real-time index/FNO synchronization and live
OI/chain availability remain for a confirmed open session. A future session date
is not certified merely because it is a Monday; review exchange/special calendars.

## Changed files and validation

Added `model/fno/value_audit.py`, `tests/test_fno_value_audit.py`, this guide and
`docs/DATA_OFFLINE_FNO_VALIDATION.json`. Modified
`model/fno/read_verification.py` only to accept an opt-in per-audit cache; existing
consumer calls keep uncached behavior. Updated `controller/diagnostics_cli.py`,
`tools/market_logic_check.py`, `START_HERE.md`, `RELEASE_NOTES.md` and the handoff.
No application SQL writes, candle changes, fetching, unit rescaling or UI work.

Baseline focused session/underlying/calendar tests: 55 passed. New audit tests:
17 passed. The initial combined sandbox run had two existing socket guard tests
fail with PermissionError before their network guard could run; unchanged guard
assertions passed outside sandbox. An initial broader outside-sandbox regression
run passed 102 tests. Final isolated acceptance: Operations196 passed/43 skipped, API100 passed,
Jobs107 passed/27 skipped, Aggregation77 passed/14 skipped: **480 passed,
84 skipped, 0 failed**. Results are recorded in the validation JSON; overlapping runs must not be added together. Production/MySQL
and native Windows results for this new command are pending.

Status: next offline audit implementation complete; production value results,
source/unit evidence and live verification remain explicitly pending. Items 2–3
are not silently marked fully complete.
