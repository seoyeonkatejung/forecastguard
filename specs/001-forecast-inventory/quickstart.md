# Quickstart Validation Guide

**Status**: Forward-looking guide for the implementation phase. The commands below cannot run until
the package, lock file, CLI, tests, and app are implemented. Documentation creation did not install
packages, fetch the workbook, fit models, run application tests, or launch Streamlit.

## Prerequisites

Use the implementation's recorded Python version (proposed 3.12), a compatible exact dependency lock,
and the original [UCI Online Retail II workbook](https://archive.ics.uci.edu/dataset/502/online%2Bretail%2Bii)
placed at `data/raw/online_retail_II.xlsx`. Record checksum and CC BY 4.0 attribution. Run from the
repository root. GitHub setup and public hosting are separate later operations.

## Clone and local database setup

The implementer must publish the actual GitHub clone URL in the root README; none is available yet.
Clone the repository, enter its directory, then follow the commands below. Use your own workspace
path to keep datasets and working choices separate from another local project. DuckDB is embedded:
preparation creates a local database file, with no separate database service to install or start.
Keep the workspace out of version control.

For Windows, document environment activation as `.venv\Scripts\Activate.ps1` in PowerShell; the
shell example below uses macOS/Linux activation. The implemented README must verify both the stated
platform prerequisites and the exact launch command, rather than assuming a globally installed app.

## Planned commands

```sh
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.lock
python -m pip install --no-deps -e .
python -m forecastguard.cli prepare --source data/raw/online_retail_II.xlsx --workspace workspaces/default
python -m pytest
python -m forecastguard.cli analyze --demo --overage-cost 1 --underage-cost 3 --workspace workspaces/default
FORECASTGUARD_WORKSPACE=workspaces/default python -m streamlit run app.py
```

The `prepare` audit must resolve schema and sheet-overlap questions and confirm/freeze the protocol
before performance evaluation. Tests include small fixtures that need no raw workbook, plus an
explicitly designated real-data integration run. `analyze` saves selection before final-test outputs;
repeated runs never silently choose a model using saved test scores.

## Resume, backup, and recovery

After initial preparation, the workspace contains `sales.duckdb`, `workspace.json`, `raw/`, and
`runs/`. Select a product/week, set costs, run analysis, choose a view, and select **Save workspace**.
Close the browser and stop the process. Restart with the same workspace; no repeat `prepare`, source
selection, or completed compatible model search should be necessary. Move the original external
workbook to verify that the managed snapshot/database are sufficient. A moved workspace must reopen
using relative paths, without changing forecast-origin restrictions.

To back up, stop the app/close database connections and copy the entire workspace. Restore to a
separate directory and launch against that path. For safe reset, first make a backup and explicitly
create a new empty workspace; a cache clear must not remove the old one. A stale analysis needs
recomputation, a missing raw source may need restoration for full re-preparation, and a damaged or
unsupported manifest must produce a recoverable error. Do not advise deleting the database as a
universal repair. For file-lock errors, close another process writing the same workspace.

The root README and `docs/local-setup.md`, `docs/workspaces.md`, and `docs/troubleshooting.md` are
planned implementation deliverables. They may later be presented with MkDocs, using the same
Markdown sources; MkDocs is optional and not required to run the numerical application.

## Expected outcomes

1. Inspect the preparation report: sheet ranges and row/exclusion counts reconcile; UK product/day
   totals are nonnegative; returns remain separate; assumptions and partial-week exclusions appear.
2. Inspect three initial-training-selected examples. Every candidate has a complete validation
   status, the selected structure follows MAE/tie rules, and test dates are later than selection.
3. Verify metrics against [plan.md](plan.md), including distribution support count, post-transform
   intervals/CRPS, common scored weeks, and visible poor results if any. No numerical outcome is
   presumed to improve on baseline.
4. In the app, select an example and review the Monday–Sunday forecast, reference-based EWMA, and
   editable costs. Change costs and verify the forecast/model selection stays unchanged.
5. Confirm the quantity and three expected outcomes match saved support arithmetic; compare
   retrospective paired costs against the eight-week mean policy. Simulation labels stay visible.
6. Select an unsupported catalog code and enter invalid costs: see clear reasons and no stale
   recommendation. Use known synthetic data to exercise zero demand, tied optima, and a bias shift.
7. Repeat the clean workflow with the recorded lock and raw checksum. Candidate selections and
   integer quantities match exactly; summaries match `atol=1e-8, rtol=1e-6` on the reference platform.
   Record first-run and cached runtime, machine details, and any deviations.

8. Follow the save/restart and backup/relocation scenarios above, including two distinct workspaces
   and a simulated interrupted save. Confirm settings, data and valid completed outputs survive,
   stale outputs are withheld, and recovery instructions preserve the last complete state.

See [data-model.md](data-model.md) and [app-workflow.md](contracts/app-workflow.md) for contracts.
Implementation evidence is required before marking any of these scenarios passed.
