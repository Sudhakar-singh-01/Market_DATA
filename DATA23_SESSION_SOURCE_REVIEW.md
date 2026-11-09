# Reviewed NSE session rules and disputed index tail

Full package: `7.8_market_data_v4_4_0_data23_session_evidence.zip`.
Baseline: `7.8_market_data_v4_4_0_runtime_report_paths.zip`.
Prepared9 November2026, Asia/Kolkata. Only items2–3; UI/next step held.

## Content reviewed from uploaded documents

These are scoped claims from the actual supplied PDFs. The agent did not directly
download/authenticate the exchange archive or establish that no later circular
supersedes them. Document review does not independently verify broker field units,
holidays, special sessions or feed identity.

| Document | Reviewed content | SHA256 |
| --- | --- | --- |
| NSE/FAOP/74467,29 May2026,2 pages | p1 normal equity-derivatives open09:15,close15:40; p2 effective3 August2026 | 6ff897abbe384e65426b5a53922e6a46690c315a0d80d1e923a09dd82d1b20d3 |
| NSE/CMTR/73362,18 March2026,4 pages | p1 CAS initially applies to stocks with derivative contracts; p4 section6.5 distinguishes actual index and indicative index close during CAS | c7da4fb3a74954903d9ac2106e16f22679654405774df7abe602f369fd1e8377 |
| NSE/FAOP/75472,30 July2026,5 pages | p1 live effective date3 August2026; p2 15:40 table explicitly describes1 August mock Session3, not a daily-hours proof by itself | 226a935c47d3baa819a26a729fe2a58bfcf315ab6ca81111df9fd10eff070c78 |

FAOP74467 was listed on NSE with a ZIP attachment; the guessed direct .pdf URL
returned404. Use the circular listing/actual attachment, not that guessed URL.

### Supported normal F&O rule

FAOP74467 sectionA explicitly states normal market09:15–15:40, effective
2026-08-03 on p2. This supports the existing385 canonical1m period-start slots
09:15–15:39 under the implementation's close-exclusive convention. Pre-open is
listed separately09:00–09:08 with random closure and is not part of this normal
385-minute candle scope. No FNO session shortening is justified by the375-minute
CASH index report.

SectionD specifies the closing-price VWAP window15:10–15:40. Preserve the existing
derived daily-bar contract: the daily OHLC close comes from the canonical bars;
it is not automatically the exchange's official closing/settlement-price VWAP.
No last-bar close was replaced with an invented official closing price.

### CASH / index rules still requiring care

CMTR73362 is a CAS operational/price-band SOP, not a complete daily CASH/index
session timetable. Section6.5 says the Exchange disseminates both actual index
values and indicative index close values during CAS, using different constituent
price rules. These are not interchangeable series. The reviewed pages do not
establish which stream FYERS NSE:NIFTY50-INDEX/NSE:NIFTYBANK-INDEX represents or
the final publication cutoff for each series.

The documents do not authorize extending every CASH instrument to15:40, filling
missing spot prices, replacing actual with indicative values or treating every
stock as CAS-eligible. FAOP74467 says its FAQ is attached as an annexure; that
separate attachment was not among the supplied2-page FAOP PDF. The earlier
CMTR72394 and relevant FAQ may contain remaining detailed CASH/index modalities;
they have not been reviewed here.

## Correction to the interpretation of the current index inventory

The existing selected CASH workflow planned only09:15 through15:29. Its375 index
bars and strict SQL/Parquet checks certify that configured regular scope. The
provider was never asked for15:30–15:39 by that fetch plan. Therefore missing
stored index pairing in those ten minutes is not proof that FYERS lacks them.

The current underlying check correctly reports ten unpaired FNO timestamps and
keeps full alignment false. That flag is preserved. The previous phrase that
regular index data is ready remains scoped to the configured session; it must
not be read as a complete official CAS/index-publication certificate.

## New read-only two-call provider observation

`controller.index_tail_probe_cli` requests only15:30:00–15:39:59 IST for one closed
historical date on/after3 August2026. It resolves only configured active NSE INDEX
mappings for NIFTY/BANKNIFTY, uses the existing authenticated FYERS client, shared
low-priority account limiter and response classifier, and makes one history SDK
call per selected index. No application retries, SQL/Parquet reads/writes, imports,
jobs, socket subscription, calendar extension or full-day/history scan occur.
Existing account quota/lock state can be updated by the limiter.

The report records requested epoch bounds, unique in-window minute timestamps,
duplicates, malformed numeric/OHLCV shapes and returned out-of-window rows.
At most2 mappings and2,000 response rows per request are processed. Provider
responses which ignore the requested range, contain duplicates or invalid rows
are flagged for review even if some tail timestamps exist. Authentication,
rate-limit, network failure or interruption stops later attempts and retains
partial observations. Raw provider messages, prices, secrets and credentials are
not written to the report. Unknown actual/indicative identity and units remain
unverified. This is not a SQL/Parquet value certificate or independent quote truth.

### Windows CMD

Use updated code with the existing profile and a valid FYERS token. No previous
JSON report input is needed, and the market need not be open for this historical
probe. Reports still follow TIOS_HOME; absolute paths are supported.

```bat
set "TIOS_HOME=F:\TIOS_Runtime"
.venv\Scripts\python.exe -m controller.index_tail_probe_cli --date 2026-11-05 --underlyings NIFTY BANKNIFTY --output "F:\TIOS_Runtime\index_tail_20261105_v1.json"
```

Admin password is prompted. Send the resulting JSON; do not run a normal fetch or
reset to force the tail into storage. Choose a new output filename if it exists.

| Per-index status | Meaning |
| --- | --- |
| ALL_REQUESTED_TAIL_SLOTS_OBSERVED | Ten unique shape-valid timestamps were returned; stream identity/price truth/units still unverified |
| PARTIAL_TAIL_OBSERVATIONS | Fewer than ten unique valid requested timestamps returned |
| PROVIDER_NO_DATA_IN_TAIL_REQUEST | Provider reports empty history for this bounded request; not proof of no index publication |
| RANGE_OR_ROW_REVIEW_REQUIRED | Invalid, duplicate or out-of-window response rows require review |
| PROVIDER_AUTH_REQUIRED / PROVIDER_RATE_LIMITED | Stop and resolve token/quota first |
| PARTIAL / NOT_ATTEMPTED | Later checks did not complete |

Exit2 is expected even when observations are collected successfully. There is no
full-PASS claim, no inference that all ten minutes are mandatory, and no change to
the provider's series mapping. Native broker results must precede any scoped
storage/session-policy proposal.

## Session audit evidence field

Existing `diagnostics_cli session-units` full JSON adds reviewed_exchange_source.
For FNO, it records the reviewed normal-hours/effective-date rule and whether the
requested ordinary configured hours match. Changed hours, pre-effective ranges
or special scopes do not inherit that match. CASH records the actual/indicative
distinction while keeping full CASH/index bounds unresolved. Existing general
external_evidence_verified/unit_scale_verified flags and issue meanings remain;
the broader calendar/source/unit scope is not silently promoted to PASS.

## Changes and tests

Added model/exchange_session_evidence.py, controller/index_tail_probe.py,
controller/index_tail_probe_cli.py, tests/test_index_tail_probe.py, this guide and
DATA23_SESSION_VALIDATION.json. Modified model/session_unit_audit.py additively,
tests/test_session_unit_audit.py, tools/market_logic_check.py and release/start/
handoff documentation. Configured hours, storage, existing reader and public
aggregation behavior are unchanged. No dependency, UI or next-step work.

Baseline relevant suite:52 passed. Focused new/existing suite:57 passed. Final affected fixture suites: **344 passed,70 optional MySQL-fixture skips,
0 failed** (Operations237/43, Jobs107/27). Results are recorded in
DATA23_SESSION_VALIDATION.json; runs
overlap and are not summed. Native provider requests were not executed in cloud.

Status: normal FNO timing source review complete for these supplied circulars;
CASH/index tail availability/mapping, full index timetable, provider units,
empty/sparse scope interpretations and actual live evidence remain pending.
User's earlier six empty historical option rechecks all returned providerNO_DATA;
no reason exists to repeat those six requests for this index-tail task.
