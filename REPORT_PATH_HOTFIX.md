# Persistent report paths — 9 November 2026

Full package: `7.8_market_data_v4_4_0_runtime_report_paths.zip`.
Baseline: `7.8_market_data_v4_4_0_data23_recheck.zip`.

## Problem and correction

The recent item2–3 CLIs passed relative report filenames directly to Path, so
they depended on the current code folder even with TIOS_HOME configured. A new
checkout could not find earlier value reports, and generated diagnostics/plans
were left beside the source. The generic FileNotFoundError message did not
identify the failed path. The user's reported error alone does not prove which
input/configuration file was missing; the new diagnostic identifies it.

The new report_path helper anchors relative paths to TIOS_HOME when set. Thus,
with TIOS_HOME=F:\TIOS_Runtime, a bare report.json means
F:\TIOS_Runtime\report.json. Nested paths such as reports\check.json also remain
under the profile. Output parents are created after authentication (or in the
already-supported explicit offline diagnostic mode), before running checks.

This applies to:

- data23_probe_cli --reports and --output;
- diagnostics_cli inventory/session-units/fno-values --output;
- underlying_cli preview/check --output and run --plan.

Explicit absolute paths retain their meaning. Without TIOS_HOME, existing
current-working-directory report semantics are preserved. A missing configured
profile fails closed; no report falls back to a similarly named code-folder file.
Relative paths escaping the profile are rejected; use an explicit absolute path
for an intentional external input/export. Existing outputs remain protected by
exclusive create; old reports are not moved, overwritten, deleted or regenerated.

The empty-scope CLI reports FILE_NOT_FOUND, missing_path and a stage such as
input_reports, admin_authentication, configuration_and_provider_recheck or
write_report. It does not expose passwords, token values or raw provider errors.
Underlying/diagnostic CLIs also expose the missing filesystem path on this error.

## Windows: one-time report placement

Use the updated source with the existing profile and interpreter. Only the two
existing value JSON reports need to be available at stable paths; do not copy
blank configuration or reset data.

```bat
set "TIOS_HOME=F:\TIOS_Runtime"
dir "F:\TIOS_Runtime\nifty_values_20261105_v1.json"
dir "F:\TIOS_Runtime\banknifty_values_20261105_v1.json"
```

If they are still in the current code folder, these commands copy them only when
the target is absent; /-Y also prevents a racing existing target being silently
overwritten. If the source reports are elsewhere, use their exact old paths.

```bat
if not exist "F:\TIOS_Runtime\nifty_values_20261105_v1.json" copy /-Y "nifty_values_20261105_v1.json" "F:\TIOS_Runtime\nifty_values_20261105_v1.json"
if not exist "F:\TIOS_Runtime\banknifty_values_20261105_v1.json" copy /-Y "banknifty_values_20261105_v1.json" "F:\TIOS_Runtime\banknifty_values_20261105_v1.json"
```

Do not regenerate already-reviewed reports just because their source paths moved.
An existing target should be checked for correct report contents/date, not silently
replaced. Explicit full input paths can also read the old files without copying.

Then run the same scoped provider recheck with fully explicit paths:

```bat
.venv\Scripts\python.exe -m controller.data23_probe_cli --reports "F:\TIOS_Runtime\nifty_values_20261105_v1.json" "F:\TIOS_Runtime\banknifty_values_20261105_v1.json" --output "F:\TIOS_Runtime\data23_empty_recheck_20261109.json"
```

With this update, the previous bare-filename command is equivalent when TIOS_HOME
is set. Explicit full paths also work with the preceding package, if the files
already exist at those paths. A new output filename is required if a prior report
exists. The recheck's exit2 is intentional; successful observation collection
still does not certify all session/unit/live/coverage gates.

Future value reports and underlying plans now persist across code upgrades:

```bat
.venv\Scripts\python.exe -m controller.diagnostics_cli fno-values --underlyings BANKNIFTY --from 2026-11-05 --to 2026-11-05 --output reports\banknifty_values_new.json
```

That example writes F:\TIOS_Runtime\reports\banknifty_values_new.json. It is only a
path example; the prior verified value audit need not be repeated. For a plan
stored by underlying_cli, open it with its runtime path in Notepad as well.

## Scope, tests and state

Modified model/config/runtime_paths.py and the three item2–3 CLIs above; added
tests/test_report_paths.py and this guide/validation. Updated test manifest,
start/release/handoff and the affected guide examples. This is not a global
rewrite of unrelated backup/reset/workflow CLI path rules. No credential changes,
new dependencies, SQL/data operations, schema/session/unit changes or UI work.

Baseline relevant tests:81 passed. Focused after new path regressions:91 passed.
Final affected disposable suites: **326 passed,70 optional MySQL-fixture skips,
0 failed** (Operations219/43, Jobs107/27). Machine evidence is in
REPORT_PATH_VALIDATION.json; overlapping runs are not added together.
Meaningful tests cover code-folder changes, same-name stale local input, exact
missing-path diagnostics before DB/provider access, nested output parents,
absolute overrides, no-profile compatibility, relative escape rejection, output
preservation and no directory creation after failed authentication.

Native Windows production paths were not inspected from cloud. The user must
place/select the existing reports and send the resulting empty-scope JSON.
Items2–3 remain PARTIAL; source/unit access and live/coverage evidence pending.
The official-host network draft remains saved, not applied/published. Next step
and UI remain held.
