# Windows Jobs test environment correction

Latest full package: `7.8_market_data_v4_4_0_windows_jobs_hotfix.zip`.
Baseline is `7.8_market_data_v4_4_0_recovery_windows_reset.zip`, SHA-256
`848ddb3547517b3062d7d0875d4c108af7cc55c6970acef2e0720b6c7a2acf5c`.

## User-supplied Windows results

Python 3.13.1, approximately 6 minutes 3 seconds:

| Group | Passed | Skipped | Failed |
| --- | ---: | ---: | ---: |
| Jobs | 87 | 27 | 1 |
| Storage | 322 | 61 | 0 |
| Operations | 144 | 44 | 0 |
| Total | 553 | 132 | 1 |

The previously failing CASH recovery/rebuild test now passes on Windows. The
remaining failed test is:
`tests/fno/test_fno_cli.py::test_main_registers_all_release1_modes_and_fno_modes_skip_subsystem_a_startup`.

## Cause and change

That test deliberately launched main.py with only a POSIX PATH and HOME to prove
that FNO refuses to start without explicit database isolation/credentials. On
Windows, omitting SYSTEMROOT prevents socket/asyncio native imports from loading
their service providers: `_overlapped` raises WinError 10106 before the CLI can
print its expected configuration error. This is a test-process bootstrap failure,
not evidence that production market-data jobs or storage need resetting.

The minimal test environment now retains only PATH, SYSTEMROOT, WINDIR, TEMP and
TMP from the host, with case-normalized names. HOME and USERPROFILE point at the
test's disposable directory. DB/FNO/FYERS/TIOS credentials/settings and PYTHONPATH
remain excluded; dotenv loading stays disabled. Existing assertions still require
both isolation refusal and missing FNO credential diagnostics. A regression checks
OS bootstrap preservation and credential exclusion. Failure assertions now include
stderr so native import failures are visible.

Production CLI/job logic, reset logic, UI and market data are unchanged.

## Tests actually run

- Baseline CLI file: 1 passed, 16 MySQL-backed skips on Linux.
- Final focused isolation/environment checks: 2 passed, 16 deselected.
- Final isolated Jobs group: results recorded in WINDOWS_JOBS_VALIDATION.json.
- Native Windows verification of this test correction remains pending. The user's
  supplied Storage/Operations passes are distinct Windows evidence, not new cloud
  reruns. No production database/provider was accessed in these checks.

## Run next

From the updated project folder with dependencies installed, only rerun Jobs:

```bat
tools\market_logic_check.cmd --groups jobs
```

Expected final status PASSED. Storage/Operations already passed on Windows and
their code did not change; no repeat full suite is required for this test fix.

## Reset status and correct acknowledgment

The user's preview succeeded: CASH market_data contains 8 tables and 2706 selected
Parquet files; FNO fno_market_data contains 10 tables and 73 selected Parquet files.
Those are preview inventory counts, not proof of data correctness or backup success.

The apply attempt stopped at `SQL data-loss acknowledgment not provided` because
the entered phrase was `ACCEPT`. Reset execution/backup creation had not started.

The user already authorized cleaning both active stores. After stopping the API,
dashboard, live feeds, workers/schedulers and external readers/writers, make a
fresh plan so changing runtime inventory does not invalidate the old one:

```bat
set "TIOS_HOME=F:\TIOS_Runtime"
.venv\Scripts\python.exe -m controller.reset_cli preview --output reset_clean_plan_v2.json
notepad reset_clean_plan_v2.json
.venv\Scripts\python.exe -m controller.reset_cli apply --plan reset_clean_plan_v2.json --writers-stopped
```

Use a different fresh filename if v2 already exists. Verify actual database names,
hosts and roots in the plan. At the prompts enter the complete phrases:

```text
ACCEPT SQL DATA LOSS
RESET market_data + fno_market_data
```

Use the actual plan's second phrase if database names differ. Expected success:
EMPTY_SCHEMA_READY. The utility verifies fresh backups for both segments, archives
active Parquet, drops/recreates the reviewed tables and retains recovery journals.
Archived copies/backups remain. Provider-limited expired history may be impossible
to refetch, so preserve them. Configuration and credentials are retained.

MySQL DDL and file moves are not atomic. RECOVERY_NEEDED requires inspecting
completed phases/backups while writers remain stopped; never blindly retry or
delete the marker. Complete bounded refill commands and post-reset verification
are in RECOVERY_WINDOWS_RESET.md. The cloud has not executed a production reset.

## Changed files and status

- `tests/fno/test_fno_cli.py`: OS-safe isolated environment and regression coverage.
- `RELEASE_NOTES.md`, `START_HERE.md`, `docs/CLAUDE_HANDOFF.md`: current status.
- Added `docs/WINDOWS_JOBS_HOTFIX.md` and `docs/WINDOWS_JOBS_VALIDATION.json`.

Code correction complete; native Jobs acceptance and user-local reset/refetch
pending. No source files deleted, no UI changes, prior backend fixes included.
