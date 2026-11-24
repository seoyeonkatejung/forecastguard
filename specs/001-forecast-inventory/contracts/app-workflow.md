# Application and Analysis Contracts

**Status**: Planned local interfaces. No external API, authentication, or upload endpoint at checkpoint.

## Manager workflow

1. Choose any product code from the built-in catalog and a Monday origin from the fixed test period,
   or the final historical demonstration origin. Show date range and local cutoff explicitly.
2. Submit **Run analysis**. Show progress for uncached fits, with no invented latency promise.
3. Show one of: history only with support reasons; point forecast without distribution; complete
   results; or explicit failure. Never retain an earlier product's recommendation as current output.
4. Forecast view: historical sales, seven daily point predictions, weekly total, weekly 80%/95%
   intervals when supported, empirical predictive mean, sample count, and limitations.
5. Evaluation view: all seven candidate validation statuses/scores; frozen winner; separate test
   metrics for winner and seasonal naive, common dates/counts, and failures. Validation charts for
   the eventual winner are post-selection diagnostics, not unbiased final evaluation.
6. Monitoring view: signed errors, reference bias, centered EWMA and fixed review limits; explain
   positive errors as underprediction and the alert's limited interpretation.
7. Inventory form: overage and underage penalties in £/unit, finite and strictly positive; default
   illustrative 1 and 3. Submit **Compare policies** without rerunning forecasts.
8. Inventory output: origin/week, integer quantities for both policies, expected cost in £, expected
   leftover and unmet units, and separately historical paired simulated outcomes. State that q is
   stock available at the period start, with no replenishment. Show unsupported-tail warnings.

Persistent inventory label:

> Simulation using observed sales as a demand proxy. Stockouts and lost demand are unobserved.
> Results depend on availability and predictive-distribution assumptions; they are not verified
> business savings.

Cost validation errors identify the field and required condition. Data, eligibility, and fit failures
name their cause, rather than displaying an empty plot or zero metric. Partial weeks are marked
unscored. Historical forecasts are not labeled current-year forecasts.

## Workspace workflow

- On startup, open the configured local workspace or offer **Create/Open workspace**. After first
  preparation, an existing workspace needs no repeat source selection. Show saved timestamp/location.
- **Save workspace** persists valid product, origin, costs, active view, and completed-run references.
  Applied analysis/cost submissions also save; unsubmitted edits show an unsaved indicator.
- Browser refresh or app restart restores the last complete save. A cost edit does not rerun model
  search. Loading an earlier origin cannot use later errors from any saved bundle.
- **Open workspace** switches data and choices together; it cannot mix results from another workspace.
- If saved results are incompatible or incomplete, withhold current recommendations and explain
  recovery/recomputation. Failure during saving leaves the preceding complete save usable.
- Document offline backup/restore of the whole closed workspace. Clearing transient caches preserves
  it. No cloud synchronization, shared account, or CSV import is added by these actions.

## Planned CLI

These entry points are implementation requirements for a reproducible workflow, not commands that
exist in the repository yet:

- `python -m forecastguard.cli prepare --source PATH`: audit workbook, normalize and aggregate,
  write catalog/audit/checksum artifacts; fail if schema or overlap remains ambiguous.
- `python -m forecastguard.cli analyze --product CODE --overage-cost VALUE --underage-cost VALUE`:
  run frozen protocol for one code; record calibration and selection before generating final-test
  results, save the complete artifact chain; return an explained unsupported status if ineligible.
- `python -m forecastguard.cli analyze --demo --overage-cost 1 --underage-cost 3`: choose three
  examples using initial-block ranking only and analyze without test-driven replacement.

All commands accept `--workspace PATH`, defaulting to `workspaces/default`. `prepare` copies the
source snapshot and creates the file-backed `sales.duckdb`; a repeated prepare with the same checksum
and protocol reuses compatible prepared data. A changed source creates a new snapshot and invalidates
affected results; never silently combine datasets. `analyze` opens the prepared workspace without
another `--source` argument. The app reads `FORECASTGUARD_WORKSPACE` (same default) and exposes the
save/open workflow. These are future implementation contracts.

Default protocol path: `config/protocol.json`. Make source/protocol overrides explicit and record
all effective values. Invalid inputs or data failures return nonzero exit status and a useful error.
An expected unsupported product returns a recorded unsupported status, without misleading numerical
outputs. The same product analysis functions serve the CLI and interface.

## Artifact contract

An analysis bundle contains a manifest, protocol snapshot, candidate table, daily/weekly forecasts,
model-selection record, empirical supports with their source origins, metric summaries, monitoring
reference/records, and inventory decisions/outcomes. CSV is appropriate for flat tables; JSON for
manifest, configuration and ragged supports. Dates use ISO format; null means unavailable with a
reason. Units and metric definitions accompany exports. Do not publish invoice-level customer data.

The containing workspace also includes its managed source, `sales.duckdb`, and atomically saved
`workspace.json`, with the fields and compatibility rules in the data model. Analysis bundles under
`runs/` are complete and immutable before the manifest references them.

Scope has no CSV-import contract yet. A future upload feature must define its own schema and
validation before implementation; do not silently expose arbitrary workbook or query execution.
