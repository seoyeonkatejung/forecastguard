# Implementation Plan: ForecastGuard Forecasting and Inventory Decisions

**Branch**: No git branch created; resolver feature identifier: `001-forecast-inventory`
| **Date**: 2026-10-26 | **Updated**: 2026-11-23 | **Spec**: [spec.md](spec.md)

**Input**: `specs/001-forecast-inventory/spec.md`, interview decisions, and constitution v1.0.0.
**Status**: Design only. No application code, data profiling, dependency installation, or execution
of statistical models has occurred. Numerical defaults below are planning decisions to freeze before
final testing, rather than additional user-confirmed interview answers.

## Summary

Build a local Streamlit demonstration for UK product-unit sales using Online Retail II. Use DuckDB
SQL to validate and aggregate transactions, compare seven forecasting structures per selected
product, construct an empirical distribution of seven-day sales, monitor signed errors, and solve
an independent single-period newsvendor problem. Retrospective inventory evaluation uses held-out
sales under an explicit demand-proxy assumption. Neither better forecasts nor inventory savings
are guaranteed. Keep the implementation within approximately 20–24 hours through on-demand product
analysis, cached outputs, one simple interface, and no deployment at the checkpoint.

## Technical Context

**Language/Version**: Python 3.12 proposed baseline; confirm compatible packages at implementation
start, then record the exact patch version and lock dependency versions. No tested environment is
claimed by this plan.

**Primary Dependencies**: Streamlit, DuckDB, Plotly, statsmodels, pandas, NumPy, openpyxl, pytest.
Use the standard library for configuration, hashing, logging, and command parsing. No extra
forecasting framework, solver, API server, or monitoring package is required.

**Storage**: Per-workspace file-backed DuckDB database for prepared product/date tables, a managed
source snapshot, versioned JSON working state, and immutable CSV/JSON analysis bundles. Do not commit customer identifiers,
raw workbook, environments, or caches; include download instructions, attribution, and checksums.

**Testing**: pytest with small known-answer fixtures and one complete integration example; manual
local-app checks. Numerical tests matter more than broad interface snapshot testing.

**Target Platform**: Student laptop; GitHub source repository when initialized by the implementer.
Local execution first. Streamlit Community Cloud (`streamlit.app`) is a later proposed target.

**Project Type**: Single Python package and thin Streamlit web interface, with a small analysis CLI.

**Performance Goals**: Reuse prepared results so the manager can complete the demonstrated workflow
within five minutes. Measure uncached one-product runtime and cache-hit latency on the actual laptop;
there is no promised all-catalog runtime. Fit only selected products and three prepared examples.

**Constraints**: Checkpoint November 23, 2026; 5–6 hours per week. Do not replace model search with
one fixed model to save time. Defer optional presentation work before reducing correctness checks.

**Scale/Scope**: One public workbook (UCI reports approximately 1.07 million transaction rows), UK
only, any product selectable, supported products analyzed individually. One weekly horizon, seven
candidates, one uncertainty method, one monitor, and two inventory policies. Source details are
from [UCI](https://archive.ics.uci.edu/dataset/502/online%2Bretail%2Bii); the workbook is not yet audited.

## Constitution Check

*Gate evaluated before research and again after design: PASS at design level; implementation must
supply the evidence described below.*

| Principle | Design evidence and implementation gate |
| --- | --- |
| Statistical correctness | Fixed weekly target, baseline, explicit assumptions; known-answer checks and failure accounting. |
| Reproducibility | Source checksum, locked environment, protocol hash, saved support and predictions; clean rerun required. |
| No future leakage | Four chronological blocks, past-only fitting and eligibility, completed-error bank; mutation tests required. |
| Explicit metrics | Formulas, units, equal week weights, common comparison weeks, failure counts; no success threshold favoring a method. |
| Readable Python | Small typed pure functions, SQL separation, thin UI; focused numerical tests. |
| Course scope | Original 20-hour core plus buffer; new persistence/docs work estimated separately at 3–5 hours; on-demand search and explicit deferrals. |

No constitution exceptions are requested. Poor coverage or unfavorable policy results are findings;
incorrect calculations, leakage, or omitted failures block valid reporting.

## Project Structure

### Documentation (this feature)

```text
SPEC.md                                 # Self-contained coding-agent handoff at repository root
specs/001-forecast-inventory/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── checklists/requirements.md
└── contracts/app-workflow.md
```

`tasks.md` is intentionally left to a later task-generation command.

### Source Code (repository root)

Proposed paths only; this planning operation does not create them:

```text
app.py
pyproject.toml
requirements.lock
config/protocol.json
src/forecastguard/
├── data.py
├── forecasting.py
├── evaluation.py
├── uncertainty.py
├── inventory.py
├── monitoring.py
├── workspace.py                       # Save/open, compatibility checks, managed paths
└── cli.py
sql/aggregate_sales.sql
tests/
data/raw/                             # Excluded from version control
workspaces/default/                   # Local only; exclude from version control
├── workspace.json
├── sales.duckdb
├── raw/                              # Managed original source snapshot
└── runs/                             # Completed versioned analysis bundles
README.md                            # Clone/install/run entry point (to be implemented)
docs/                                # Local setup, workspace lifecycle, troubleshooting
artifacts/                            # Optional generated exports
```

**Structure Decision**: One package with one responsibility per module; no separate frontend/backend,
database server, deployment configuration, or abstractions without a concrete use.

## Data Preparation and Availability

1. Obtain the original workbook from UCI with CC BY 4.0 attribution. Enumerate sheet names, headers,
   row counts, timestamp ranges, and potential cross-sheet overlaps. Save a SHA-256 checksum and
   audit summary. Do not assume sheet names or duplicate counts have already been verified.
2. Read sheets with pandas and an explicit openpyxl engine. Normalize documented aliases, including
   `Invoice`/`InvoiceNo`, `Price`/`UnitPrice`, and `Customer ID`/`CustomerID`. Ambiguous aliases are an
   intake error. Read invoice and stock codes as strings, not numerical quantities.
3. Required usable fields: invoice identifier, product code, transaction timestamp, quantity, and
   country. Price is retained for diagnostics, not inferred inventory penalties. Customer ID and
   description are optional; missing values there do not remove otherwise valid sales.
4. Via DuckDB SQL, normalize country whitespace/case and keep `United Kingdom`; reject unusable
   required fields, nonfinite/noninteger quantities, and invalid timestamps with counted reasons.
   Exclude cancellation invoices (case-insensitive leading `C`) and nonpositive quantities from
   positive-unit sales, preserving separate exclusion summaries. Do not retroactively erase an
   earlier sale because a later return exists.
5. Flag suspected non-merchandise codes for a documented explicit mapping, frozen after intake
   review. Unsupported codes remain selectable with an explanation. Keep large legitimate orders;
   no full-history statistical outlier removal. Do not automatically deduplicate indistinguishable
   invoice/product lines without evidence of source duplication. Resolve overlapping sheet records
   once in the documented intake audit; ambiguous duplication blocks an affected aggregate.
6. Aggregate positive units by exact product code and UK local civil date with DuckDB. Preserve
   daily sums and row/exclusion counts. Do not fill before the first observed qualifying sale.
   Fill later absent dates with zero only within the assumed complete recording period, using
   the agreed offered-and-available assumption. Never infer discontinuation from the last sale
   found in the full dataset.
7. Treat timestamps as local UK civil timestamps for day grouping, not UTC; timezone provenance
   and historical publication delays are unavailable. Use only transactions dated before the
   Monday origin. This is an explicit availability assumption, not a verified reporting history.
8. Remove partial edge weeks from outcomes. Calendar completeness does not prove complete
   transaction capture. Display that limitation and freeze dataset coverage after the audit.

## Forecast Protocol and Search

### Target and chronological blocks

Let `o` be Monday 00:00 local time. Fit through the previous Sunday; predict the seven dates
`o,...,o+6`. Let `Y_o` be their total positive sales. Each week receives equal evaluation weight.
No random temporal splits, overlap between target weeks, or daily-origin pseudo-replication.

The default protocol uses the following complete weeks, derived from UCI's documented coverage.
Confirm them in the intake audit, then freeze them before examining test outcomes:

| Block | Dates inclusive | Purpose |
| --- | --- | --- |
| Initial fitting, 26 weeks | 2009-12-07–2010-06-06 | Fit initial models; select demonstration examples using this block only. |
| Rolling calibration, 26 weeks | 2010-06-07–2010-12-05 | Generate truly out-of-sample weekly errors for every candidate. |
| Rolling selection, 26 weeks | 2010-12-06–2011-06-05 | Score candidates and freeze one winning structure per product. |
| Final test, 26 weeks | 2011-06-06–2011-12-04 | Evaluate selected pipeline and seasonal naive without reselection. |

Do not use partial 2009-12-01–06 or 2011-12-05–09 as complete scored weeks. The final demonstration
forecast origin can be 2011-12-05 using observations through 2011-12-04; its partly observed future
week remains unscored and those partial observations cannot enter its decision. Label this a
historical demonstration, not a forecast for the current calendar year. Actionable selected-model
forecasts and inventory replay are restricted to origins on or after 2011-06-06, after selection
completes. Earlier calibration/selection charts are retrospective candidate diagnostics; they cannot
use the eventual winner as if it had already been chosen at those earlier origins.

### Eligibility (explicit design defaults)

At an origin, point forecasting requires at least 26 complete weeks of product history, positive
sales in at least 13 of the preceding 26 weeks, and no run of eight zero-sales weeks ending at the
cutoff. Apply these tests to past observations only. These cutoffs bound sparse/inactive products;
they do not establish statistical validity. Products lacking the fixed protocol's history cannot
receive a validation-selected model and must show why. Inventory and interval output additionally
require at least 26 completed valid errors for the method in use and a finite predictive support.
A 26-week calibration block does not ensure 26 usable errors for every product; late introductions,
inactivity, and fit failures can delay support. Never pad the error bank or borrow future errors.

Choose three demonstration products from initial-block characteristics only: rank products meeting
eligibility by number of positive-sales weeks, then total initial units, then product code. Do not
replace examples because test performance is poor. Report later eligibility loss or fit failure.
For a final checkpoint, aim to complete all planned validation weeks and at least 20 eligible test
weeks per demonstration product; if this is not achieved, report the shortfall, not a changed test.

### Seven candidates and fitting

- `SN7`: repeat the preceding seven daily observations.
- `ES_N_N`, `ES_A_N`, `ES_Ad_N`: no, additive, or damped additive trend; no seasonality.
- `ES_N_A7`, `ES_A_A7`, `ES_Ad_A7`: the same trend choices with additive period-seven seasonality.

Use `statsmodels.tsa.holtwinters.ExponentialSmoothing`, estimated initialization, `missing='raise'`,
no Box–Cox transform, and optimized parameters. Damping is enabled only for additive trend.
Use one documented optimizer and initialization policy for all origins, such as L-BFGS-B with
`use_brute=False`; lock the actual library version and record warnings/convergence status.
[Official constructor and supported options](https://www.statsmodels.org/stable/generated/statsmodels.tsa.holtwinters.ExponentialSmoothing.html),
[fit options](https://www.statsmodels.org/stable/generated/statsmodels.tsa.holtwinters.ExponentialSmoothing.fit.html).

Create fresh fit objects per origin with expanding past-only history. Project negative daily point
forecasts to zero before summing; use this identical postprocessing in calibration, selection,
test, and the app. Record projection counts. The resulting weekly model point forecast `f_o`
remains real-valued; it is distinct from the mean of the empirical predictive distribution.

Select minimum selection-block MAE of weekly totals. A candidate must have valid forecasts for
all planned eligible selection origins; numerical failures disqualify it rather than reducing its
scored sample. If there are fewer than 20 eligible selection origins, do not declare a winner.
Report counts and failures for every candidate. Compare successful candidates on exactly the same
origins. Ties within absolute `1e-8` MAE use fixed order:
`SN7, ES_N_N, ES_A_N, ES_Ad_N, ES_N_A7, ES_A_A7, ES_Ad_A7`.
No final-test outcomes, CRPS, or inventory costs enter selection.

Freeze structure before test; refit its numerical parameters weekly using expanding history.
Earlier completed test weeks may enter subsequent fits and error banks under this declared policy;
this is prequential evaluation, not an embargo on all test-era updating. A failed test fit yields
no selected-method forecast or recommendation for that week; it does not trigger an unrecorded
reselection. Report planned, eligible, scored, and failed week counts and paired comparison weeks.
Always retain the separately evaluated seasonal-naive baseline. Rolling-origin guidance:
[Forecasting: Principles and Practice](https://otexts.com/fpp3/tscv.html).

## Predictive Demand Distribution and Validation

For each model structure, store `e_j = Y_j - f_j` from prior out-of-sample complete weeks. Begin
with calibration-block errors; append selection and earlier completed test errors under the same
fit policy. At origin `o`, retain the most recent 52 valid completed weekly errors, never the current
week. Require at least 26. Keep errors uncentered so historical bias is not silently removed.

For `m` available errors define equally weighted nonnegative integer support:

```text
x_j = max(0, floor(f_o + e_j + 0.5)),   probability = 1/m for each entry
```

Duplicate support values retain their accumulated probability. This is a simple empirical additive
weekly-error approximation. It assumes recent aggregate errors transfer to the next week; it is
not an ETS state-space predictive distribution or a reconstruction of latent lost demand.
Aggregate weekly errors incorporate realized within-week effects without assuming independent
daily errors. Do not resample independent daily residuals, add daily interval endpoints, or infer
a full distribution from two interval bounds. Display weekly intervals only; daily plots show
point forecasts without fabricated daily uncertainty bands.

Use the entire empirical support for expectations and scores; no Monte Carlo draws are required.
Define `Q(p)` as the smallest support value whose empirical cumulative probability is at least p.
Central 80% interval: `[Q(.10), Q(.90)]`; central 95%: `[Q(.025), Q(.975)]`. Include endpoints when
scoring coverage. Show the support mean as predictive mean and `f_o` separately as model forecast.
Validate after rounding/clipping, which changes the distribution. Report support size, distinct
values, proportion clipped to zero, and quantile resolution `1/m`. Warn when an inventory critical
ratio falls in the outer `1/m` tails or selects an extreme support value; no tail beyond the
observed residual support is modeled.

For `n` common scored weeks, use these definitions:

```text
MAE = mean(abs(Y_o - f_o))                         [weekly units; lower is better]
RMSE = sqrt(mean((Y_o - f_o)^2))                  [weekly units; lower is better]
coverage(level) = mean(L_o <= Y_o <= U_o)         [fraction; compare with stated level]
mean_width(level) = mean(U_o - L_o)              [weekly units; coverage/width trade-off]
CRPS_o = mean_j(abs(x_j - Y_o))
         - sum_j sum_k abs(x_j - x_k)/(2*m*m)    [weekly units; lower is better]
CRPS = mean(CRPS_o)
```

Use all pair terms for exact empirical-distribution CRPS. Do not use the finite-sample correction
for a different latent-distribution estimand. The scoring-rule basis is
[Gneiting and Raftery (2007)](https://doi.org/10.1198/016214506000001437).
Show baseline and selected-method scores on common scored weeks, dates, and counts; never replace
missing forecasts with zero. Within-product weeks have equal weights. If showing a multi-product
summary, macro-average product scores with equal product weights, show product counts, and keep
per-product results visible. Undefined scores are null with a reason. Do not use MAPE on zero sales.

Report validation and final-test distribution diagnostics separately. Small samples do not justify
claims of nominal calibration or reliable extreme quantiles. There is no required coverage band or
required method improvement for a successful project: numerical correctness and honest reporting
are mandatory; poor calibration is a substantive result.

## Inventory Optimization and Policy Simulation

Let `c_o > 0` and `c_u > 0` be finite pounds per leftover and unmet unit. They are net penalties,
not selling prices, purchase costs, or measured business losses. Quantity means stock available at
the start of the period; starting inventory, ordering lead time, and pipeline stock are not modeled.
Each product is independent, with integer `q >= 0` and no shared budget or capacity limit.

For the empirical support calculate:

```text
L(q) = mean_j max(q - x_j, 0)               [expected leftover units]
U(q) = mean_j max(x_j - q, 0)               [expected unmet units]
C(q) = c_o * L(q) + c_u * U(q)             [expected pounds]
alpha = c_u / (c_u + c_o)
q_star = smallest integer q with F(q) >= alpha
```

Compute the critical ratio stably for very different positive costs; reject nonfinite resulting
arithmetic with an explanation. For this discrete linear objective,
`C(q+1)-C(q) = (c_o+c_u)*F(q)-c_u`, so the smallest quantile minimizer is exact for the supplied
empirical model, including ties. No arbitrary upper capacity or general solver is necessary.
This analytical solution does not prove the forecast distribution represents actual demand.
[Single-period inventory reference](https://rossetti.github.io/RossettiInventoryBook/07-Chapter7.html).

The benchmark is `q_B = floor(mean(previous eight complete weekly sales totals) + 0.5)`.
Use the same half-up rounding convention throughout. Costs do not affect `q_B`.
Show expected cost/leftovers/unmet units for both quantities under the same predictive support.
The optimizer cannot be expected to lose to the benchmark on its own modeled objective; explicitly
separate that mechanical result from the historical policy comparison.

For each eligible held-out origin, choose quantities using earlier data, then substitute observed
`Y_o` for demand and compute both simulated realized costs, leftovers, and unmet units. Report
paired per-week results, means, cumulative costs, and `cost_benchmark - cost_optimized`, including
negative values. Use precisely the same weeks and costs for both policies; show withholding counts.
Default illustrative costs are `c_o=1, c_u=3`, labeled hypothetical and editable. Cost changes
recompute policy arithmetic against cached historical predictive supports, not forecasts or winners.

Historical replay is the principal simulation. Do not describe it as observed lost demand,
verified business savings, a causal effect, or proof of optimal real-world replenishment.
No significance test for cost improvement is needed at checkpoint; correlated weekly outcomes
and small samples make unsupported precision inappropriate.

## Monitoring

Use signed weekly errors `e=actual-model forecast`. Positive errors mean underprediction. Monitor
the frozen selected structure over test weeks; never use current unfinished-week outcomes.
Use its calibration-plus-selection errors as the pre-test reference, with mean `mu_ref`. Show that
mean separately so a persistent pre-existing bias is not hidden by centering.

Define `z_0=0` and `z_k=.2*(e_k-mu_ref)+.8*z_(k-1)`. Replay the reference errors chronologically,
then take the empirical 95th percentile (inverse-CDF convention) of their absolute centered EWMA
values as the fixed review threshold `H`. Start test monitoring from the final reference state;
alert when `abs(z_k)>H`. Freeze `mu_ref`, `H`, lambda, and selected structure throughout test.
With fewer than 26 reference errors, withhold monitoring. If `H=0`, label the reference degenerate;
any nonzero deviation is a review signal, without a nominal false-alarm claim.

This deliberately simple, empirically calibrated band is exploratory. Serial dependence, selection,
and small reference samples mean 95th-percentile calibration is not a guaranteed 5% future alert
rate. Signed EWMA primarily detects shifts in directional bias; interval/CRPS diagnostics address
other uncertainty failures. No automatic retraining or parameter changes follow alerts.
[NIST EWMA background](https://www.itl.nist.gov/div898/handbook/pmc/section3/pmc314.htm).

## Application and Reproducibility Design

One page with product selection and explicit Run analysis action, then four views: sales/forecast,
validation/test comparison, monitoring, and inventory. Display origin/cutoff, assumptions, and status
before results. Product choices cover the catalog; unavailable analyses explain the failed gate.
Use Plotly for sales, daily point forecasts, weekly intervals, errors, and policy-cost comparisons.

Read and aggregate data once; cache serializable results, not mutable fit objects. Product-analysis
cache keys include source checksum, protocol/preprocessing versions, product, dates, candidate
settings, and source revision. A separate cost form reruns only inventory calculations.
[Streamlit caching](https://docs.streamlit.io/develop/concepts/architecture/caching),
[forms](https://docs.streamlit.io/develop/concepts/architecture/forms).

Each run records code revision or source hash, Python/package versions, raw checksum, SQL and
protocol hashes, normalized schema, exclusions, candidates and failed fits, origins, cutoff dates,
daily/weekly predictions, actuals only when observed, error support with source origins, metrics,
monitor reference and alerts, and inventory costs/decisions. Seeds default to 615 wherever
randomness is introduced; the primary empirical computations are deterministic. Exact decisions
and candidate selection must reproduce; floating metrics use `atol=1e-8, rtol=1e-6` in the locked
reference environment. Record rather than hide cross-platform numerical variation.

## Local Workspace Persistence and User Setup

**Added 2026-11-23**: This is a scope addition to the October plan. The dated checkpoint schedule
below is historical; it does not establish that the checkpoint or implementation is complete.
Budget approximately 3–5 additional implementation/verification hours for minimal persistence and
plain documentation, to be confirmed during implementation; MkDocs presentation is optional.

Use a local workspace directory, default `workspaces/default`, selected explicitly by name/path.
Each user clone can create its own workspace; no database server or account is required. Persist
prepared sales in `sales.duckdb`, not just a DataFrame or a temporary SQL relation. An explicit
file-backed DuckDB connection supports reopening data across processes
([DuckDB persistent storage](https://duckdb.org/docs/2.0/clients/python/overview#persistent-storage)).
Keep JSON/CSV result bundles and a small `workspace.json` manifest alongside it. Streamlit session
state may hold live widget values, but durable recovery must read the workspace: it resets on a
browser connection reset ([Streamlit session-state limitations](https://docs.streamlit.io/develop/api-reference/caching-and-state/st.session_state)).

- First preparation copies the chosen workbook into workspace-managed storage, hashes/audits it,
  and commits prepared tables plus provenance. Later startup reads the workspace database and
  manifest; the external source file does not need to be selected again. This currently concerns
  the built-in dataset only, not a new CSV upload flow.
- Provide **Save workspace** and **Open workspace** actions and show workspace name, location,
  last successful save, and unsaved-changes indicator. Save valid working choices explicitly and
  after successfully applied analysis/cost submissions. Do not silently persist invalid inputs.
- Save product, Monday origin, costs, active view, protocol snapshot/hash, dataset checksum,
  schema version, analysis-version/source hash, dependency-lock hash, and completed run references.
  Rehydrate settings before rendering their widgets. Loaded run outputs must still be filtered by
  the restored origin's information cutoff; a later week's error bank cannot serve an earlier week.
- Reuse compatible completed bundles before fitting models. Changing only costs recomputes inventory
  arithmetic; changing dataset/protocol/model code invalidates affected numerical results. Loading
  stale results shows an explanation, not a current recommendation. Match all cache keys from the
  reproducibility design. In-memory cache clearing must not delete a workspace.
- Commit database preparation in a transaction with immutable snapshot identifiers; retain tables
  needed by the previous manifest instead of replacing its dataset in place. Write completed bundles to a new immutable run
  directory, then atomically replace the manifest as the last step. Until this pointer update
  succeeds, the previous complete workspace remains authoritative. Save UI state through the same
  atomic manifest replacement. No need for serialized live model objects or resumable partial fits.
- Reopening resolves relative paths within the workspace, validates schema/hashes/bundle completion,
  and reports missing or damaged components. Preserve prepared data when only a result is stale;
  offer explicit recomputation. An unsupported schema is a recoverable error with upgrade/restore
  instructions, not an automatic destructive reset. Handle a busy local database with an instruction
  to close the other writing process; concurrent multi-user writing is outside this design.
- For backup, stop the app and close database connections, then copy the entire workspace, including
  the source snapshot, database, manifest, and run bundles. Reopen a restored copy at a different
  path using relative references. Deleting a workspace requires an explicit user action; do not
  delete it through a generic clear-cache action. Keep workspaces/raw data out of Git.

Provide a root README linking `docs/local-setup.md`, `docs/workspaces.md`, and
`docs/troubleshooting.md`. Cover clone URL (once the GitHub repository exists), supported Python and
exact dependency installation, OS-specific environment activation, initial dataset acquisition and
attribution, database creation/path, workspace selection, app launch, save/restart/reopen, backup,
restore, explicit reset, stale results, schema errors, and file locks. Commands must use the actual
implemented CLI and pass one clean-clone smoke check. Do not publish invented working setup claims.
Plain Markdown is sufficient initially. Optionally render the same docs through MkDocs after the
workflow works; keep a separate docs dependency group and avoid a second copy of the instructions.

Future validated CSV imports should create a new immutable dataset snapshot inside a chosen
workspace and invalidate related analyses. Hosted persistence would require a separate design for
user identity, isolation, and durable storage; the local design makes no cross-device promise.

## Correctness and Acceptance Evidence

- Data: hand-computed UK aggregate, returns separate, no dependence on customer IDs, exclusion
  reconciliation, and overlap/duplicate handling. First-sale and partial-week boundaries explicit.
- Leakage: modify/append future transactions and future errors; earlier eligibility, forecasts,
  banks, quantities, and monitor states must stay identical. Also assert each bank source week ends
  before the decision origin and no final-test values enter structure selection.
- Models: all seven structures instantiated correctly; seasonal-naive toy example; daily totals
  sum correctly; deterministic ties and all-fold failure rules; identical postprocessing everywhere.
- Metrics: hand-computed MAE, RMSE, coverage and width; degenerate-distribution CRPS equals absolute
  error; a two-point distribution verifies all CRPS pair terms. Empty samples produce unavailable.
- Inventory: exhaustive integer comparison over `0..max(support)` on small distributions; exact
  critical-ratio ties, all-zero demand, high/low ratios, duplicate support, and expected-cost identity.
  Match expected costs within `1e-8`. A larger cost input need not improve true outcomes.
- Simulation: both policies use identical held-out outcomes, eight prior weeks only, no resubstitution
  outcome in its own uncertainty support, and signed savings retained.
- EWMA: manual recurrence and frozen reference; no unfinished-week update; synthetic sustained
  shifts trigger review and future errors do not change earlier states.
- End to end: one clean rerun and three training-selected examples; unsupported product and invalid
  cost UI flows; saved reports match displayed metrics. Record runtime on the actual laptop.

- Persistence: save, refresh, stop/start, and reopen with original source moved; restore product,
  origin, costs, active view, prepared sales and compatible result references without another fit.
  Exercise changed-data/protocol invalidation, an earlier-origin restore, two workspaces, interrupted
  save, damaged manifest/bundle, and backup relocation; preserve the last complete save.
- Documentation: follow clone/install/prepare/run/save/reopen from a clean environment and record
  versions and command outcomes. Only mark published commands working after this walkthrough.

These are planned checks, not test results from an implemented application.

## Milestones, Budget, and Risks

| Dates (2026) | Core hours | Deliverable and gate |
| --- | --- | --- |
| Oct 26–Nov 1 | 5 | Intake audit, frozen protocol, SQL daily aggregates, seasonal-naive path and boundary checks. |
| Nov 2–8 | 5 | Six smoothing structures, rolling model search, model-specific errors, MAE/RMSE and fit-failure report. |
| Nov 9–15 | 5 | Integer empirical support, intervals/CRPS, exact optimizer, historical benchmark simulation and numerical fixtures. |
| Nov 16–22 | 5 | EWMA, thin interface and plots, three examples, clean rerun, checkpoint narrative and limitations. |
| Nov 23 | 0 scheduled | Demonstrate local workflow; retain up to four additional hours from 6-hour weeks as buffer. |

The 20-hour core is ambitious, not an assurance. Integrate numerical checks with each stage. If
behind, remove visual polish, optional exports, and extra example products before any agreed model
search, evaluation, uncertainty, optimizer, monitoring, or correctness requirement. If core gates
still cannot be met, report the specific incomplete scope rather than quietly claiming completion.

| Risk | Mitigation |
| --- | --- |
| Workbook quality or sheet overlap | Audit first; preserve counts, inspect ambiguous duplication; do not assume a cleaned source. |
| Late/sparse or discontinued products | Past-only eligibility, visible unsupported status, any-product selection retained. |
| Slow repeated fits | On-demand one-product work and cache; no all-catalog fitting. Profile before changing search. |
| Nonconvergence | Explicit failure records and all-fold candidate qualification; no silent omission or replacement. |
| Only 26–52 tail observations | Show support resolution and extreme-cost warnings; report poor coverage honestly. |
| Nonstationary errors or wholesale orders | Show calibration/test diagnostics and EWMA limitations; no guarantee of reliable demand tails. |
| Sales censored by unavailable inventory | Disclose availability assumption; simulate only; do not estimate verified lost demand. |
| Selection optimism and limited test size | Separate test, frozen structure, no outcome-driven retuning or significance claims. |
| Time budget | Keep local, one horizon, seven candidates, one monitor, one optimizer; defer hosting/CSV/Tableau. |

## Complexity Tracking

No constitution violations or extra infrastructure. Empirical expectations and the analytical
integer quantile solution keep optimization smaller than a solver-based implementation.
