# Data item 2 — session/calendar and numeric-unit audit

Prepared 7 November 2026, Asia/Kolkata.
Full package: `7.8_market_data_v4_4_0_data_phase2_session_units.zip`.
Baseline: `7.8_market_data_v4_4_0_data_phase1_inventory.zip`, SHA-256
`d24a5c3c55f16ba9a5496e0b7680c213c13e3b20a383c01dfc728f7e78f60bdc`.

## Scope and changes

The user's data plan item 2 is implemented as `diagnostics_cli session-units`:

- Report configured session intervals, close-exclusive boundaries, minute counts,
  local calendar verification flags, applicable specials and effective-range gaps.
- Offline mode can review future calendar dates without DB/auth/provider calls.
- Online mode inspects only selected contracts/symbols, requested closed dates,
  and stored SQL 1m numeric/timestamp fields. No Parquet/history rescan.
- Count invalid/null/present price/volume/OI values, minute alignment, timestamps
  outside configured sessions, missing lot/tick metadata and rows after expiry.
- Report comparisons with CURRENT lot/tick metadata without claiming historical
  effectiveness or inferring lots versus underlying quantities from divisibility.
- Explicitly document source/storage timestamp conventions, provider-native
  volume/OI scale, provisional live volume, nullable unknown fields and unconfirmed
  IV/Greek units. Index volume is not labeled traded-share quantity.
- Retain partial results under read bounds/errors or a changing calendar snapshot.

Numeric bugs found during inspection were corrected for FUTURE normalization:
`model/fno/chain.py` no longer uses `int(float(value))`; large integral values keep
precision, fractions/booleans/nonfinite quantities become unknown rather than
truncated/zero. Signed OI-change remains signed. Nonfinite optional decimals become
null; valid negative theta/delta remain allowed. `model/fno/normalizer.py` rejects
boolean historical OI instead of accepting True as 1. Existing stored rows are not
rewritten and already-coerced original feed types cannot be recovered from SQL.

## Evidence findings and limits

The existing app cites CMTR71775 (Cash holidays), FAOP71777 (FNO holidays) and
FAOP75472 (FNO timing effective 2026-08-03). Existing configuration uses FNO
09:15–15:40, last start 15:39, 385 minutes. This update DOES NOT assume that timing
is wrong or replace it with older market hours. A fresh attempt to retrieve the
cited timing PDF received HTTP/proxy 403; its contents were not independently
verified in this environment. Existing comments/review notes are retained as
citations, not new independent confirmation.

Configured `verified_years`/`verified_from` flags are declarations. The report
distinguishes them from `external_evidence_verified=false`. Unknown special-session
times, including the bundled 2026-11-08 Muhurat entry, remain unconfirmed. Inspect
the ACTIVE runtime calendar; local reviewed overrides may differ from bundled
defaults. Calendar snapshots are hashed and checked for changes during audit.

Historical volume/OI are stored in provider-native magnitude. A multiple of lot
size is NOT proof that the field is lots or underlying units; zero is also a
mathematical multiple and is not evidence. Current lot/tick rules may differ from
historical rules. Off-current-tick-grid observations need review, not automatic
rounding or rewriting. Provider source documentation or independently reconciled
samples with explicit units/effective dates are still needed for unit confirmation.

Chain IV has an existing probe note declaring percent (15.96 would be 0.1596 as a
fraction). Storage does not automatically rescale it, and this audit does not
confirm new chain/Greeks availability. Greeks' time/normalization basis remains
unconfirmed. Live FNO candles currently leave OI null. Chain collection, broader
OI availability and live freshness are later data-plan items, not started here.

The audit does not establish OHLCV ordering/parity, source correctness, exchange
session truth, complete history or trading readiness. It does not fetch, migrate,
repair, reset, modify the runtime profile or change the UI.

Bounds: 31 inclusive days, 50 selected targets (existing inventory selector), 500
target/day scopes, 20000 SQL rows/day and 200000 total SQL rows plus at most one
sentinel. FNO underlying selection uses data/claims in the requested date range,
not the entire contract master. Reuse reports; no full historical scan is required.

## Windows commands

Use the updated project interpreter/dependencies and existing permanent profile:

```bat
set "TIOS_HOME=F:\TIOS_Runtime"
.venv\Scripts\python.exe -m controller.diagnostics_cli session-units --segment CASH --symbols SBIN RELIANCE HDFCBANK ICICIBANK NIFTY50 NIFTYBANK --from 2026-11-05 --to 2026-11-05 --output cash_session_units_20261105.json
.venv\Scripts\python.exe -m controller.diagnostics_cli session-units --segment FNO --underlyings NIFTY --from 2026-11-05 --to 2026-11-05 --output fno_session_units_20261105.json
```

Admin authentication precedes online catalog access. Choose NEW output filenames;
existing reports are never overwritten. Completed historical dates are required
online; unavailable CASH data is reported rather than fetched.

Optional configuration-only review including the upcoming special-session date:

```bat
.venv\Scripts\python.exe -m controller.diagnostics_cli session-units --segment FNO --from 2026-11-05 --to 2026-11-09 --offline --output fno_calendar_review_20261105_09.json
```

Offline needs no admin password or database. Omit target selectors in offline mode.
It can inspect future calendar declarations but does not prove actual market hours.

Expected status: `AUDIT_FINDINGS`, exit 2, `complete=true` when inspection succeeds.
This means the audit ran, with explicit pending source/unit evidence and possibly
stored-data findings. It is not a failed test. External/unit proof is never set
true simply to turn the command green. `PARTIAL`/exit2 means some inspection was
incomplete; `INVALID_SCOPE`/exit2 means dates/selectors/output path were invalid.
`UNAVAILABLE`/exit1 indicates auth/configuration/setup/report-file failure.

An expected native report does not automatically fill the five missing CASH
symbols or change sparse option histories. Send the JSON reports for review.
Do not reset/refetch/rescale merely because the audit reports findings.

## Tests actually run

Baseline calendar/diagnostics/history-normalizer suite: 26 passed. Final focused
audit/diagnostics/history-normalizer suite: 28 passed. Affected isolated groups:
History 106 passed/29 skipped; Operations 179 passed/43 skipped; total 285 passed,
72 MySQL-fixture skips on Linux Python 3.12.14. Runs overlap; do not sum them.

Tests cover offline no-auth/no-DB, unconfirmed specials, split sessions, effective
dates, read-only SQL, unknown versus zero OI, lot-divisibility limitations, current
tick review without writes, invalid numeric/timestamp shapes, bounds, changed
calendar, exclusive output and parser precision/nonfinite regressions. An offline
FNO calendar smoke check completed with the expected findings. Native Windows,
real source-document confirmation and production reports remain pending.

Optional isolated native check:

```bat
tools\market_logic_check.cmd --groups history operations
```

## Changed files and status

Added: `model/session_unit_audit.py`, `tests/test_session_unit_audit.py`, this guide,
`docs/DATA_PHASE2_VALIDATION.json`.
Changed: `controller/diagnostics_cli.py`, `model/fno/chain.py`,
`model/fno/normalizer.py`, `tests/fno/test_fno_chain.py`,
`tests/fno/test_fno_normalizer.py`, `tools/market_logic_check.py`,
`RELEASE_NOTES.md`, `START_HERE.md`, `docs/CLAUDE_HANDOFF.md`.

Implementation complete; authoritative session/unit confirmation and native
acceptance remain partial/pending. Item 3 and UI work have not been started.
