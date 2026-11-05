# New TIOS UI and multiple-task center

Run from the extracted project root:

```bat
.venv\Scripts\python.exe -m streamlit run ui/dashboard.py --server.address 127.0.0.1
```

The old numeric sidebar/page routing is replaced by nine workspaces: Overview,
Historical Data, Live Market, Data Explorer, Coverage & Quality, Jobs, Instruments,
Storage & Recovery, Settings & Diagnostics. Default styling is dark, with an
explicit role display and one shell-owned logout. Original guarded domain controls
are reused inside named views. Their previews, confirmations and service checks
remain authoritative. See `UI_FEATURE_MAP.md` for every old-to-new entry and FNO
subsection; the module inventory is checked automatically.

## Task center — right side, at the top

On desktop the persistent right panel refreshes every five seconds. It lists
every observed task card independently; it never replaces tasks with a hidden
"+N" summary. Long lists scroll. Smaller windows place the panel above the main
workspace so it does not obscure controls.

Sources:

- CASH persisted PENDING/RUNNING ingestion jobs, including workflows, fetch,
  aggregation and verification; no historical table scan.
- FNO persisted PENDING/RUNNING jobs of every job_type, including live, chain,
  contract synchronization, historical, aggregation and verification.
- Legacy process-local background jobs, CASH live controller and auto gap scanner.
- Foreground UI activity contexts: guarded tasks and tracked spinners, including
  previews, backup/export, recovery checks, schema operations and cleanup.

Each source is read-only. Monitoring never starts a worker, resumes a job, contacts
a broker or creates a schema. Persisted rows use keyset pagination of 200 rows to
retrieve all active entries without an arbitrary display cutoff. Task identifiers
include source and segment, so matching numeric IDs are not collapsed.

Available counters and heartbeat are shown; there is no invented overall percent.
A persisted RUNNING row is reported as recorded, not proof that its worker is
alive. Unavailable sources are explicit and the panel says its list may be
incomplete. Raw params, result blobs, exception bodies and credentials are not
selected for the panel. Every rendered string is HTML-escaped.

Foreground operations are visible immediately before their blocking call; their
registry entry is removed on success or exception. The same browser session may
not poll while a synchronous operation blocks, so its initial card can remain
until the operation ends. Other sessions can observe the registry on refresh.
Completed short tasks disappear from this in-progress panel; durable history is
in Jobs. Process-local tasks do not survive a server restart. External commands
are visible only when they record work in the configured job tables; arbitrary
unregistered scripts/machine processes cannot be inferred from database status.

## Data reader

Data Explorer defaults to a new explicit stored-candle reader. Select CASH/FNO,
symbol, timeframe including 3m, dates, quality and session. The first and next
pages are loaded only on a click, 200 rows per page. Changing the scope discards
old results/cursors. The displayed JSON export retains quality metadata. Verified
reads preserve SQL/Parquet checks and sparse-coverage meaning; raw results do not
receive a quality certificate. Table/catalog, contract and chain tools remain
available in the workspace selector.

## Acceptance check

```bat
.venv\Scripts\python.exe -m tools.ui_rebuild_check
```

The script runs task-center, workspace, widget, permission, isolated SQL/Parquet,
FNO isolation, recovery/cleanup and migrated-view tests using a temporary profile.
Real MySQL fixtures are explicitly skipped before setup. Expected: no failures
and final JSON `"status": "PASSED"`; skips vary with Windows symlink privileges.

For visual acceptance, sign in, navigate every workspace, and inspect the panel
with your existing jobs. No new production fetch/cleanup is required. AppTest
checks reruns and controls but does not certify browser pixel layout on Windows.
Old unextracted copies of `ui/pages/` must not be mixed into this new package:
extract into a fresh code directory and retain the existing TIOS_HOME profile.
Do not reinitialize the profile or rerun historical repairs.

## Validation evidence

- Existing UI/role/isolation regression: 203 passed, 4 optional skips.
- Final task/workspace/workflow/runner safeguards: 40 passed.
- Existing provider-free acceptance runner: core 100, API 42, operations 400 passed
  and 64 skips; final PASSED on Linux/Python 3.12.
- Task federation exercised 906 concurrent observed task cards across sources,
  cursor paging, namespaced IDs, escaped content, source outages and exception
  cleanup. This is correctness evidence, not a production load benchmark.

Native Windows Python 3.13/browser acceptance of the rebuilt UI remains pending.

## Browser preview evidence

Local Chromium screenshots are included in `screenshots/ui_desktop.png` and
`screenshots/ui_small_window.png`. They show the real shell rendered with three
explicitly labeled sample tasks; no production jobs were created. The preview
checked the top-right desktop panel and its inline small-window layout. It is
not a native Windows visual acceptance result.

The isolated one-command UI runner also completed: 203 passed, 4 optional skips,
final PASSED. A final feature-routing/task subset completed 14 passed.
