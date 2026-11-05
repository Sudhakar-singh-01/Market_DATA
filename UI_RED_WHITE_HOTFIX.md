# Live UI fixes and red/white theme

The supplied Windows run passed 203 UI tests, but live interaction exposed a CSV
export widget ID collision and a Windows sharing violation during FNO aggregation.

## Changes

- Preview, job-list, verification-history and finding CSV exports now have stable
  context-specific keys. Preview and a populated auto-refresh job fragment can
  render together without duplicate download widgets. A real Streamlit widget
  regression reproduced the original exception before the fix.
- Streamlit and the shell now use a white background with deep-red accents and a
  pale-red sidebar. Task cards retain the top-right multi-task layout. Desktop
  and small-window screenshots show explicitly labeled sample tasks, not live
  account operations.
- FNO Parquet reads own and close their file handles. Candidate writes use an
  exclusive writable stream, flush and fsync before validation/publication.
- Publication retries the same candidate at most five times, only for Windows
  sharing/lock errors 32/33 (delays 0.1, 0.2, 0.4 and 0.8 seconds). Access-denied
  and other permission errors fail immediately. Repeated failures preserve the
  previous target and restore its previous VERIFIED manifest when appropriate.
  A locked temporary candidate that cannot be removed remains diagnostic evidence;
  its cleanup error does not mask the original publication failure.

## Existing failed FNO job

This code update does not repair job 12 or its data. The supplied aggregation
failed while publishing a derived 5m partition; earlier SQL work may have committed.
SQL and Parquet do not form one atomic transaction. The process holding the file
cannot be identified from this traceback alone. Persistent external reader locks
may still cause a safe failure after the bounded retry budget.

Inspect the existing job and exact affected scope before any explicit resume.
Do not reset databases, delete runtime state or blindly repeat historical recovery.
The supplied stale-job recovery messages and interruption during shutdown are
lifecycle observations, not reasons to reset the job tables.

## Commands

Stop/restart the dashboard after updating code/theme so Streamlit reloads the
configuration. Preserve the existing TIOS_HOME; use a fresh extracted code folder.

```bat
.venv\Scripts\python.exe -m tools.ui_rebuild_check
.venv\Scripts\python.exe -m streamlit run ui/dashboard.py --server.address 127.0.0.1
```

The acceptance runner includes the duplicate-widget and Windows-lock simulations.
Expected: final JSON PASSED, with explicit optional DB/symlink skips. Native
Windows execution of this corrected package remains pending.

## Recorded verification

- UI acceptance: 215 passed, 13 optional skips; final PASSED.
- Core 100 passed, API 42 passed, operations 404 passed / 64 skips; final PASSED.
- Duplicate-widget and transient Windows-lock regressions failed before the fix.
- Persistent lock, non-sharing permission failure and owned-handle closure are
  covered by additional provider-free regressions.
- Local Chromium desktop/small-window sample previews inspected.

These are Linux/Python 3.12 results and simulated Windows locking. Native
Windows/browser verification and production scope recovery are separate.
