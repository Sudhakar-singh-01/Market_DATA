# Guarded all-store reset utility

This is a maintenance option, not a fix for a Streamlit widget collision or a
Windows file-sharing lock. Resetting makes the dataset EMPTY. It does not create
correct market data, repair code, or guarantee that FYERS can recover expired
FNO history. Existing data may be impossible to download again.

The user requested a reset file. The utility has been created; no production
preview, backup, reset, file move or migration has been executed by the agent.

## Default launcher — does not reset anything

From the project root in Windows CMD:

```bat
tools\reset_all.cmd
```

It authenticates the existing administrator and creates a new `reset_preview.json`.
It only reads registered database metadata and file inventory; it does not scan
all candle values, drop tables, fetch history or move files. Existing plans are
not overwritten. To choose another plan name:

```bat
.venv\Scripts\python.exe -m controller.reset_cli preview --output reset_preview_2.json
```

Review exact database host/port/names, ALL base tables to drop, selected Parquet
paths, recreation definitions and archive/journal location. Credentials and
profile configuration must stay in the existing TIOS_HOME.

## Explicit apply — destructive SQL operation

Only after reviewing the generated plan and deliberately deciding to lose the
active SQL data, stop ALL dashboards, live feeds, schedulers, CLI workers, schema
operations and external readers/writers. Stop them normally; this tool does not
kill processes or release someone else's file locks.

```bat
.venv\Scripts\python.exe -m controller.reset_cli apply --plan reset_preview.json --writers-stopped
```

The command authenticates the administrator and separately prompts for:

1. `ACCEPT SQL DATA LOSS`
2. The exact confirmation from the plan, normally
   `RESET market_data + fno_market_data` for the supplied installation.

This is a command to review, not an instruction to execute it automatically.
SQL passwords are not printed. No apply switch is forwarded by the CMD launcher.

Apply sequence:

1. Recheck code/plan, root identities and schema/file inventory.
2. Create fresh backups for BOTH segments using the existing BackupService.
   Both must be CHECKSUM_VERIFIED before any reset. These checks do not prove a
   successful restore or market-data parity; `restore_tested` remains false.
3. Acquire the existing database schema locks and revalidate inventory.
4. Archive BOTH segments' owned `.parquet` and Parquet `.tmp-*` candidates under
   `TIOS_HOME/recovery/reset_operations/<operation>/parquet/<segment>/...`.
   Nested FNO trees are excluded from CASH ownership. There is no permanent
   Parquet purge. The archive must be on the same filesystem as both roots.
5. Only after all selected files have moved, drop the reviewed base tables and
   recreate repository-owned EMPTY CASH/FNO schemas plus CASH job metadata.
   Custom tables are backed up/dropped but are NOT automatically recreated.
6. Retain backups, archived files and a flushed per-step `journal.jsonl`.

All SQL market-data rows, SQL instrument masters, checkpoints, jobs, manifests,
audit/error/quality records in the reviewed base tables are lost from the active
schemas. Database containers/users are retained. Original `.env`, settings,
instrument YAML, quota/runtime ledger, backups and existing recovery evidence are
not reset. Lock files and unrelated non-Parquet sidecars are not blindly erased.
Views, stored routines/events/triggers and non-InnoDB tables are refused by the
existing backup preflight; cross-database inbound foreign keys also block reset.
Such deployments need DBA review.

## Partial failures

SQL DDL and file moves are NOT atomic. A failure leaves a `RECOVERY_NEEDED` marker,
per-step intents/completions, verified backup IDs and completed phase list.
There is no automatic rollback or automatic resume. A previous marker blocks
another reset. Keep readers/writers stopped and inspect the exact journal and
archives; use the existing isolated recovery verification workflow for backup
review. Do not delete the marker merely to force a retry.

A file lock can prevent reset too. The locked-file regression demonstrates that
no SQL tables are dropped if the FNO archive move fails, although earlier file
moves can still require recovery. If DDL fails after earlier table drops, those
DDL steps may already have committed; inspect individual TABLE_DROPPED events.

After successful EMPTY_SCHEMA_READY, there are no usable candles. Review CASH
YAML-to-MySQL metadata sync, explicitly synchronize FNO contracts as appropriate,
and review a small fetch/aggregate/verify scope before using data again. There
is no automatic provider request and no automatic rebuild of historical datasets.

## Code and validation

- `model/reset_service.py`: plan, gates, backup-before-reset ordering and journals.
- `controller/reset_cli.py`: authenticated preview / separately confirmed apply.
- `tools/reset_all.cmd`: preview-only Windows launcher.
- `tests/test_reset_service.py`: disposable SQLite/files and doubles for MySQL
  backup/DDL boundaries. Covers missing acknowledgments, changed plans, backup
  failure, source drift, preserved files, partial DDL failure, FK-session cleanup,
  blocked retry, privilege checks and locked-file failure before any SQL drop.

Reset tests: 15 passed. Related reset/backup/schema/recovery/path regression ran
131 passed before adding the final locked-file case. No real MySQL reset or native
Windows reset has been executed. This utility is not certified for unreviewed
production use; its first operation should be preview and review.
