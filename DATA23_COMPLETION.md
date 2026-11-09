# Items 2–3: scoped completion recheck — 9 November 2026

Full source package: `7.8_market_data_v4_4_0_data23_recheck.zip`.
Baseline: `7.8_market_data_v4_4_0_data_offline_fno_verify.zip`.
User instruction: finish items 2 and 3 before starting another step. UI held.

## What is already established

Supplied Windows reports for 2026-11-05 were reviewed, not remotely rerun:

- Both index series: 14/14 strict checks, 1,242 rows across seven timeframes.
- NIFTY: 15,342 value-checked candle rows; 37 complete grids, 5 sparse matching
  option scopes, 1 empty scope (ID5608).
- BANKNIFTY: 12,277 value-checked rows; 12 complete grids, 26 sparse matching
  option scopes, 5 empty scopes (IDs458,468,470,472,476).
- All 27,619 verified historical rows had OI present; candle IV/delta/gamma/theta/
  vega were null. Presence does not prove units or independent source accuracy.
- Both underlying FNO timestamp unions had385 slots,375 paired to verified indexes,
  and10 unpaired15:30–15:39 IST slots outside the configured CASH session.

These results are not full source/session/unit/live certification. Sparse options
do not automatically imply missing trades; empty SQL does not prove no trading.
Do not repeat successful collection, reset databases, delete Parquet, rescale
quantities or change session hours merely to force a PASS.

## New bounded provider recheck

`controller.data23_probe_cli` consumes the two existing `fno-values` reports and
selects ONLY checks with NOT_VERIFIED / NO_CANDLES_OBSERVED / canonical1m. Supplied
reports were parsed in cloud and selected exactly the six IDs above. Existing
populated/sparse scopes are not fetched again.

Before any provider request, resolve every ID from the current registered master
and require the exact reported symbol and a valid contract/date. A reset/master
rebuild cannot silently redirect old IDs. If SQL now contains any 1m rows, skip
the request and ask for a fresh value check. If currently expired, request separate
expired-endpoint review; do not label the old date irrecoverable or consume the
limited expired API budget automatically.

The existing FYERS history payload, response classifier, normalizer and account
limiter are reused. Each eligible still-empty unexpired scope gets one history
SDK call for exactly its supplied historical date. No application retry loop.
At most10 scopes,4 report files,500 checks/report,1MiB/report and2,000 response
rows/scope. The history_call_attempts counter records SDK call attempts, not an
independent audit of lower-level network transmissions. Broker requests consume
quota; the existing shared limiter can update its runtime quota/lock files.

Authentication/rate-limit/network failures or interruption stop later attempts;
partial observations and unattempted scopes are retained in the JSON. Raw broker
messages/responses/credentials are not copied into it. Report hashes bind the
input evidence. Report output is exclusive-create; existing files are preserved.

The command does not write candles, metadata jobs/checkpoints/findings or manifests,
does not repair data, place orders or open a live subscription. Returned history
is summarized only; newly available rows require explicit scoped recovery.

## Windows command

Use the updated code and existing profile/interpreter. Both earlier JSON reports
must be in the current folder (or provide their full paths). Do not copy blank
configuration defaults over the permanent profile. A current FYERS token is needed
only if a provider call is required; the normal admin password is still prompted.

```bat
set "TIOS_HOME=F:\TIOS_Runtime"
.venv\Scripts\python.exe -m controller.data23_probe_cli --reports nifty_values_20261105_v1.json banknifty_values_20261105_v1.json --output data23_empty_recheck_20261109.json
```

Choose a fresh output filename if this one exists. Send the resulting JSON. Do not
run the broad `fno-probe` merely to recheck these six scopes: its existing probe
also tests30/60/100/200-day spans and different capabilities, which are unnecessary.

Expected outcomes:

| Scope status | Interpretation |
| --- | --- |
| PROVIDER_NO_DATA_ON_RECHECK | Fresh history request returned no data; cause and zero trading remain unproven |
| DATA_NOW_AVAILABLE_REVIEW_REQUIRED | Usable rows appeared; counts shown, nothing imported |
| PROVIDER_ROWS_NOT_USABLE | Response has no usable rows in the requested date; inspect summary |
| STORED_ROWS_NOW_PRESENT_REVERIFY_VALUES | No provider call; current SQL needs a fresh strict value audit |
| EXPIRED_API_REVIEW_REQUIRED | No standard/expired fallback request; inspect current entitlement separately |
| PROVIDER_AUTH_REQUIRED / PROVIDER_RATE_LIMITED | Stop and resolve credentials/quota before another attempt |
| PARTIAL / NOT_ATTEMPTED | Prior observations retained; later scopes were not completed |

The command returns exit2 even after successful observation collection, because
it does not close every unit/session/coverage gate. It never prints that items2–3
fully passed. Output items_2_3_fully_verified=false is intentional.

## Calendar, source access and live verification

On9 November, the bundled calendar declares a normal session. This is a local
declaration, not an independently fetched exchange notice. The report records
its calendar hash and configured CASH/FNO session status at the observation start,
with independently_confirmed=false. Do not equate Monday or current clock time
with actual exchange/feed readiness, nor infer holiday/special timings.

Cloud runtime has no user's Windows database connection or configured FYERS secret.
No native provider/live recheck was executed here. Official source access was
previously rejected by proxy CONNECT403. On9 November a draft was successfully
saved adding only nsearchives.nseindia.com and myapi.fyers.in to custom allowed
domains, preserving presets. The tool returned requires_publish=true. It is NOT
applied or published; the user previously prohibited automatic publication.
Review/save/publish that draft in environment settings, or provide the relevant
official source files, before cloud can review the session/unit documents. No
network bypass or placeholder source verification was performed.

Remaining source references: CMTR71775, FAOP71777 and FAOP75472 from NSE, plus FYERS
History volume/OI definitions and relevant IV/Greeks bases. Retrieving a document
alone would not confirm its contents, effective date or applicability.

This recheck does not verify live reception/freshness/reconnect, synchronized
quotes, candle persistence or live OI. Existing live pipelines and diagnostics must
be exercised in the user's authorized runtime during a confirmed open session.
Do not open an additional FYERS data socket alongside its existing owner; the
transport enforces account ownership. No live capture was automatically started.

## Changed files and tests

Added controller/data23_probe.py, controller/data23_probe_cli.py,
tests/test_data23_probe.py, this guide and DATA23_COMPLETION_VALIDATION.json.
Updated tools/market_logic_check.py to include new provider-free regressions,
START_HERE.md, RELEASE_NOTES.md and CLAUDE_HANDOFF.md. No schema/config/default
session/normalizer/storage/public read contracts were changed.

Baseline affected tests:62 passed. Focused new/existing tests:58 passed. Initial
new-report serialization tests exposed calendar YAML date objects in diagnostic
output; fixed the production result to expose a hash and JSON-safe configured
session summary instead of dumping raw runtime calendar metadata. Assertions were
not weakened. Final isolated Operations/History: **314 passed,72 optional MySQL-fixture skips,0 failed**.
Final new-probe suite: **13 passed**, including an additional shared-limiter CLI
factory regression. Runs overlap; do not sum them. Machine evidence is in
DATA23_COMPLETION_VALIDATION.json.
Native Windows/provider/live and independent official-source results remain pending.

Status: completion tooling implemented; items2–3 remain PARTIAL until the actual
provider/source/alignment/live evidence supports closure. No next step started.
