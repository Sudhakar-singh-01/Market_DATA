# UI replacement feature map

Current routing inventory: `ui/feature_map.py`. The old `ui/pages/` navigation and
`ui/home.py` are removed. Named, role-gated domain views preserve existing controls
and service calls inside the new nine-workspace shell. This is component reuse,
not a backend rewrite. Tests now reference the new view paths; their permission,
confirmation, failure and data-isolation assertions remain in place.

| Previous entry | New workspace / tool | Guarded view | Role |
| --- | --- | --- | --- |
| home.py | Overview — System health, Cash counts and verification evidence | `ui/views/overview_health.py` | user |
| 2_Live_Market.py | Live Market — Cash live feed | `ui/views/live_market.py` | user |
| 3_Data_Explorer.py | Data Explorer — SQL tables and candle export | `ui/views/database_explorer.py` | user |
| 4_Historical_Data.py | Historical Data — Cash Smart Fetch and gap fill | `ui/views/historical_cash.py` | user |
| 5_Aggregation.py | Historical Data — Cash aggregation | `ui/views/aggregation_cash.py` | user |
| 6_Data_Flow.py | Settings & Diagnostics — Pipeline diagram | `ui/views/data_flow.py` | user |
| 7_Data_Quality.py | Coverage & Quality — Cash verification and telemetry | `ui/views/quality_cash.py` | user |
| 8_Storage.py | Storage & Recovery — Storage files and retention | `ui/views/storage_browser.py` | user |
| 9_Logs.py | Settings & Diagnostics — Application logs | `ui/views/logs.py` | user |
| 10_API_Monitor.py | Settings & Diagnostics — Python API inspector | `ui/views/api_monitor.py` | user |
| 11_Coverage_Heatmap.py | Coverage & Quality — Cash coverage and gap preview | `ui/views/coverage_cash.py` | user |
| 12_Master_Tables.py | Instruments — Instruments, timeframes and calendar | `ui/views/master_tables.py` | user |
| 1_Admin.py | Settings & Diagnostics — Configuration and advanced CRUD | `ui/views/configuration.py` | admin |
| 20_Admin_Database.py | Storage & Recovery — Database administration | `ui/views/database_admin.py` | admin |
| 21_Admin_Files.py | Storage & Recovery — Approved file browser | `ui/views/files_admin.py` | admin |
| 22_Admin_Jobs.py | Jobs — Cash ingestion jobs | `ui/views/jobs_cash.py` | admin |
| 23_Admin_Checkpoints.py | Jobs — Cash checkpoints | `ui/views/checkpoints_cash.py` | admin |
| 24_Admin_Bulk_Delete.py | Storage & Recovery — Bulk deletion | `ui/views/bulk_delete.py` | admin |
| 25_Admin_Operations_Center.py | Jobs — Operations controls | `ui/views/operations.py` | admin |
| 26_Admin_Audit_Log.py | Settings & Diagnostics — Audit history | `ui/views/audit_log.py` | admin |
| 27_Admin_Security.py | Settings & Diagnostics — Security information | `ui/views/security.py` | admin |
| 28_Admin_Data_Operations.py | Storage & Recovery — Validation and cleanup tools | `ui/views/data_operations.py` | admin |
| 29_Admin_Database_Catalog.py | Storage & Recovery — All databases and tables | `ui/views/database_catalog.py` | admin |
| 30_Admin_Schema_Manager.py | Storage & Recovery — Schema preview and migrations | `ui/views/schema_manager.py` | admin |
| 30_FNO.py | FNO tools distributed across the nine workspaces (see below) — FNO overview, contracts, chain, history, live, aggregation, storage, checkpoints, jobs, quality and settings | `ui/views/fno_management.py` | user |
| 31_Admin_Backup_Recovery.py | Storage & Recovery — Backup, checksum and recovery staging | `ui/views/backup_recovery.py` | admin |
| 32_Admin_Unified_Workflow.py | Historical Data / Jobs — Reviewed unified workflow | `ui/views/workflow.py` | admin |
| 33_Admin_Scoped_Cleanup.py | Storage & Recovery — Scoped cleanup and recovery journal | `ui/views/scoped_cleanup.py` | admin |
| 34_Admin_Recovery_Drill.py | Storage & Recovery — Isolated recovery verification | `ui/views/recovery_verification.py` | admin |

## FNO subsection parity

The standalone FNO rendering wrapper remains as a reusable/tested domain entry;
it is not a second sidebar or old numeric route. The new workspaces select the
same section handlers directly from `ui/fno/page.py`.

| Workspace | FNO sections |
| --- | --- |
| Overview | Overview |
| Historical Data | Historical Fetch, Aggregation |
| Live Market | Live Data, Option Chain |
| Data Explorer | Futures, Options Contracts, Option Chain |
| Coverage & Quality | Data Quality |
| Jobs | Jobs and Logs, Checkpoints |
| Instruments | Futures, Options Contracts |
| Storage & Recovery | Storage Manager |
| Settings & Diagnostics | Settings |

## Backend ownership retained

| Capability | Existing implementation |
| --- | --- |
| Reviewed historical workflow / job commands | `controller/market_data_service.py`, `controller/unified_workflow.py`, `ui/workflow_service.py` |
| Cash fetch, cleaning, validation, aggregation | `controller/pipeline_service.py`, `controller/historical_controller.py`, `controller/timeframe_controller.py` |
| FNO commands and lifecycle | `ui/fno/service.py`, `controller/fno_jobs.py`, existing FNO controllers |
| Stored data / verified reads | `model/market_read_service.py`, `model/database_catalog.py` |
| Backup / schema / cleanup / recovery | Existing `model/backup_service.py`, `model/schema_management.py`, `model/cleanup_service.py`, recovery verifier |
| Credentials / permissions / runtime | Existing `ui/auth.py`, password hashing and `model/config/runtime_paths.py` |
| Multiple-task center | New `ui/task_center.py`, `ui/task_panel.py`; existing queue tables, local managers and UI activity contexts |

## Deferred scope

Event Bus, production indicator engine, trading/strategy/backtesting systems are
not added. Existing SMA remains a historical consumer example. Schema/data roots
and worker semantics are unchanged. No historical recovery or full-data scan is
performed by the new shell.
