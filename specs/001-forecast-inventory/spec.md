# Feature Specification: ForecastGuard Forecasting and Inventory Decisions

**Feature Branch**: None created; feature directory is independent of a branch.

**Created**: 2026-10-26

**Last Updated**: 2026-11-23 — workspace persistence, local setup, and Monte Carlo experiments.

**Status**: Interview complete; specified and technically planned; implementation not started.

**Input**: BIOSTAT 615 Track A interactive graduate project for a UK online-retail inventory
manager. Forecast product sales, assess reliability, and simulate independent seven-day inventory
choices. Original checkpoint: November 23, 2026, with approximately 20–24 planned development hours.
The November 23 persistence/setup addition does not reschedule that checkpoint or certify delivery.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Inspect Sales and Compare Forecasts (Priority: P1)

A manager selects any product from the built-in Online Retail II catalog, sees its UK sales history
and eligibility, and compares forecasting methods for the next complete Monday–Sunday period.

**Why this priority**: A valid forecast and understandable evaluation are prerequisites for a
credible inventory decision.

**Independent Test**: Using a documented eligible example, complete selection and view weekly
forecasts and the comparison report without entering inventory costs.

**Acceptance Scenarios**:

1. **Given** an eligible product, **When** the manager requests a forecast, **Then** the app shows
   the seven-day decision dates, a weekly point forecast, uncertainty if supported, and the selected
   method with its validation performance relative to seasonal naive.
2. **Given** an unsupported product, **When** it is selected, **Then** its history and specific
   eligibility reasons remain visible, and unsupported predictions or recommendations are withheld.
3. **Given** seven search candidates, **When** comparison completes, **Then** the report lists all
   candidates, valid scores or failure reasons, scored weeks, and the validation-based selection;
   final test results never determine the winner.

### User Story 2 - Choose a Cost-Aware Stocking Quantity (Priority: P2)

A manager enters leftover and unmet-demand costs and compares a recommended weekly stocking
quantity with a historical-average policy. This journey can be checked against a known demand
distribution independently of the forecasting interface.

**Why this priority**: It connects forecast uncertainty to the project's inventory decision.

**Independent Test**: Supply a known discrete demand distribution and two positive costs; verify
quantity, expected cost, expected leftovers, and expected unmet demand against hand calculations.

**Acceptance Scenarios**:

1. **Given** a supported predictive distribution and valid costs, **When** optimization runs,
   **Then** it returns a nonnegative integer quantity minimizing expected linear overage and
   underage cost, plus the three requested expectations in their correct units.
2. **Given** held-out weeks, **When** policies are compared, **Then** both quantities were chosen
   from earlier information, face the same weekly sales proxy, and show paired simulated costs,
   leftovers, and unmet units, including negative cost improvements when applicable.
3. **Given** zero, negative, missing, or nonfinite costs, **When** submitted, **Then** the app
   identifies the invalid input and withholds optimization.
4. **Given** insufficient forecast-error history, **When** inventory results are requested,
   **Then** the app explains the limitation and does not substitute a prediction interval for a
   demand distribution.

### User Story 3 - Review Forecast Reliability (Priority: P3)

A manager reviews forecast errors and monitoring alerts to identify sustained directional changes.
This journey can be exercised independently with a fixed chronological error sequence.

**Why this priority**: Forecasts need ongoing review; monitoring is narrower than automatic proof
that a forecast has become unreliable.

**Independent Test**: Replay completed weekly errors through a frozen monitoring configuration;
verify updates and alerts without running model search or inventory optimization.

**Acceptance Scenarios**:

1. **Given** an unfinished week, **When** viewing monitoring, **Then** its outcome does not update
   the error monitor or enter uncertainty calibration.
2. **Given** completed weekly errors, **When** replayed, **Then** the app shows an EWMA of signed
   errors, its historically calibrated review limits, and dated alerts.
3. **Given** a monitoring alert or weak interval coverage, **When** displayed, **Then** the app
   explains what was detected and does not claim a cause, guarantee future failure, or automatically
   change the selected model.

### User Story 4 - Save and Resume a Local Workspace (Priority: P2)

A manager prepares the dataset once, saves their working context, and returns later without
reloading the source data or repeating completed compatible analyses. A new user can follow the
repository documentation to run the app and its local data store on their own computer.

**Why this priority**: Repeated data loading and lost settings make the application difficult to use
across work sessions; a documented setup makes the project usable beyond its author's laptop.

**Independent Test**: Prepare a workspace, select a product/week and costs, complete an analysis,
save, close the browser and stop the app, then restart and reopen the workspace with the original
input file moved elsewhere. Verify the restored settings and completed outputs.

**Acceptance Scenarios**:

1. **Given** a saved workspace, **When** the app restarts and it is reopened, **Then** prepared data,
   product, decision week, valid cost inputs, active view, and compatible completed results return
   without reselecting the source file or rerunning completed model search.
2. **Given** changed data or analysis settings, **When** a workspace is opened, **Then** incompatible
   saved results are identified as stale and recomputed before being presented as current.
3. **Given** a missing or damaged workspace component, **When** reopening, **Then** the app explains
   what can be recovered and preserves the last complete save rather than silently starting over.
4. **Given** a fresh repository clone, **When** a user follows the local setup guide, **Then** they
   can create their own local data store, run the app, save work, and resume after a process restart.

### User Story 5 - Examine Monte Carlo Estimation and Optimization (Priority: P3)

A student or project reviewer selects a supported product/week and compares randomly simulated
inventory outcomes with exact calculations, demonstrating numerical accuracy, computational
variability, and simulation-based optimization.

**Why this priority**: This adds an explicit statistical computing experiment with a known reference
answer, using the demand distribution already needed for inventory decisions.

**Independent Test**: Use a known finite demand distribution and fixed costs; repeat seeded random
sampling at several sample sizes and check cost estimates, standard errors, and sampled-objective
optimization against hand calculations and exact expected costs.

**Acceptance Scenarios**:

1. **Given** a supported saved distribution and fixed quantities, **When** the experiment runs,
   **Then** it estimates cost, leftovers, and unmet demand for both inventory policies at sample
   sizes 100, 1,000, and 10,000, using the same demand draws for the two policies within each run.
2. **Given** independent repeated runs, **When** summarized, **Then** the report shows estimates,
   Monte Carlo standard errors, signed/absolute error against exact expectations, and variability
   across runs. It does not promise that every individual estimate improves as sample size grows.
3. **Given** a simulated average-cost objective, **When** minimized over feasible candidate integer
   quantities, **Then** its selected quantity is compared with the exact quantile solution using
   exact-model cost regret, without replacing the app's primary recommendation.
4. **Given** identical saved inputs, random-generator configuration, and locked environment,
   **When** the experiment is rerun or reopened, **Then** its random samples/results reproduce;
   changing the experiment seed does not change the fitted forecast or exact recommendation.

### Edge Cases

- Missing required fields; unparseable dates; noninteger quantities; cancellation invoices;
  returns; non-merchandise product codes; missing customer IDs or descriptions; duplicate-looking
  transaction lines; incomplete edge weeks; and inconsistent records across source sheets.
- New products, all-zero histories, prolonged inactivity, sparse sales, insufficient calibration
  errors, constant errors, failed fits, and nonfinite or negative model outputs.
- Cost ratios implying extreme quantiles, tied optimal quantities, zero-demand distributions,
  empirical tail limits, and forecasts whose uncertainty samples require truncation at zero.
- No scored weeks or incomplete candidate coverage: report unavailability and reasons, never a
  fabricated zero score or a silently reduced evaluation set.

- Browser refresh, stopped app process, interrupted save, missing source file after preparation,
  moved workspace, incompatible saved-result versions, and two workspaces with different inputs.

- Monte Carlo repetitions with rare support values not sampled, zero estimated standard error,
  tied sample-optimal quantities, invalid sample settings, and reproducibility after workspace reload.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST use built-in UCI Online Retail II and restrict analysis to UK orders.
- **FR-002**: Users MUST be able to select any catalog product; eligibility MUST be assessed using
  only information available by the selected historical decision date.
- **FR-003**: Sales targets MUST be daily positive product-unit sales. Cancellations and returns
  MUST be excluded from this target and summarized separately; they MUST NOT become negative demand.
- **FR-004**: Data validation MUST report exclusions and aggregation checks. Absent transaction
  dates after a product's first recorded sale MAY become zero only within an assumed complete
  recording period, with the offered-and-available assumption prominently disclosed.
- **FR-005**: Inventory decisions MUST occur before Monday for that Monday–Sunday, with no
  within-period replenishment; incomplete weeks MUST NOT be evaluated as complete weeks.
- **FR-006**: The system MUST compare seven candidates per eligible product: seasonal naive and
  six exponential-smoothing structures formed by no/additive/damped-additive trend crossed with
  no/additive weekly seasonality. Parameters MUST be estimated using past observations only.
- **FR-007**: Candidate selection MUST minimize rolling-origin MAE of seven-day totals on a
  predefined validation period. Candidate failures and deterministic tie handling MUST be recorded.
- **FR-008**: Final performance MUST use a separate chronological test period. Model structure
  selection MUST be frozen before this period; permitted parameter refitting and completed-error
  updates MUST follow a recorded walk-forward policy. Selected-model decisions MUST start only
  after selection completes; earlier candidate comparisons MUST be labeled retrospective.
- **FR-009**: Predictive distributions MUST use model-specific historical errors of seven-day
  total forecasts, observed before each origin. The transformation to nonnegative integer demand
  and any residual-window rules MUST be explicit and reproducible.
- **FR-010**: The app MUST show weekly 80% and 95% prediction intervals and validate the resulting
  distribution using interval coverage, interval width, and CRPS. It MUST show calibration size
  and limitations; insufficient support MUST prevent inventory recommendations.
- **FR-011**: The app MUST report MAE as primary and RMSE as secondary forecast accuracy measures,
  in weekly units, with dates and observation counts. Model comparisons MUST use common weeks;
  undefined values MUST have reasons. No performance improvement is required for project success.
- **FR-012**: Monitoring MUST use EWMA of completed signed weekly forecast errors with limits
  determined from earlier data. Alerts MUST be labeled review signals for bias changes, rather
  than a comprehensive reliability test or an automatic retraining trigger.
- **FR-013**: Costs MUST be finite, strictly positive pounds per leftover unit and pounds per unmet
  unit. They represent net decision penalties, not quantities inferred from selling prices.
- **FR-014**: For each product independently, the app MUST choose integer q >= 0 minimizing
  E[overage_cost * max(q - demand, 0) + underage_cost * max(demand - q, 0)]. It MUST show q,
  expected cost in pounds, expected leftover units, and expected unmet units from the full
  predictive distribution. Ties MUST select the smallest minimizing quantity.
- **FR-015**: The benchmark MUST stock the rounded mean of the preceding eight complete weekly
  sales totals, ignoring cost inputs. Rounding MUST be explicit and consistent.
- **FR-016**: Main policy evaluation MUST be a retrospective simulation against held-out weekly
  sales as a demand proxy. Both policies MUST face the same observations and costs. Before-decision
  expected costs and retrospective simulated costs MUST be clearly distinguished.
- **FR-017**: The system MUST disclose missing stockout information, the unidentifiability of lost
  demand, and the sales-as-demand assumption. Inventory comparisons MUST NOT be presented as
  verified savings or evidence of actual unmet demand.
- **FR-018**: Reports MUST preserve provenance, data version, exclusions, model choices, splits,
  predictions, predictive support, costs, seeds where applicable, and metric definitions so the
  analysis can be reproduced and audited.
- **FR-019**: The checkpoint MUST demonstrate a local end-to-end workflow by November 23, 2026,
  including model search, evaluation, uncertainty, EWMA, and minimal inventory comparison.
- **FR-020**: CSV uploads, public hosting, Tableau, CUSUM, annual seasonality, holiday/price
  predictors, shared budgets, capacity constraints, and multi-period replenishment MUST be deferred.
- **FR-021**: Statistical and optimization correctness MUST be checked against known examples,
  temporal boundary checks, future-data perturbations, and exhaustive small-distribution inventory
  searches before results are presented as valid.

- **FR-022**: Users MUST be able to save and reopen a named local workspace across browser and app
  restarts. The workspace MUST retain prepared data, source provenance, protocol, selected product,
  decision week, valid costs, active view, and references to completed analyses. Reopening MUST NOT
  require reselecting the original source file when the managed workspace is intact.
- **FR-023**: Saved results MUST be checked for compatible dataset, protocol, analysis version, and
  forecast origin before reuse. Reopening MUST preserve temporal cutoffs, withhold stale outputs,
  report missing/corrupt components, and preserve the last complete save on an interrupted write.
  Restoring completed work does not require resuming a partially executed model fit.
- **FR-024**: Repository documentation MUST cover cloning, the supported environment and dependency
  installation, first-time data preparation, local database creation/location, app launch, workspace
  save/reopen, backup/restore, safe reset, and common errors. The documented fresh-install and
  restart workflow MUST be checked on a clean environment before being described as working.

- **FR-025**: The system MUST provide an on-demand Monte Carlo experiment for one supported product
  and seven-day decision period. It MUST sample independently with replacement from that origin's
  saved predictive distribution, estimate both fixed policies' expected cost/leftover/unmet units,
  and use identical demand draws for paired policy comparisons. The default experiment MUST compare
  sample sizes 100, 1,000, and 10,000 across 100 independently seeded repetitions.
- **FR-026**: The experiment MUST report exact reference expectations, Monte Carlo estimates and
  standard errors, paired cost-difference estimates, repeated-run variability, and a convergence
  display. It MUST also minimize a sampled average-cost objective over a bounded finite set of
  nonnegative integer quantities and assess the resulting quantity under the exact distribution.
  Exact quantile optimization MUST remain the primary stocking recommendation.
- **FR-027**: Monte Carlo configuration, random-generator identity, seeds/streams, support version,
  costs, quantities, and results MUST be reproducible and saved with the workspace. Computation
  MUST respect forecast cutoffs and MUST NOT refit or select forecasting models. Reports MUST
  distinguish Monte Carlo numerical error from demand/forecast uncertainty and MUST NOT present
  within-model simulation gains as evidence of better held-out outcomes or verified savings.

### Key Entities *(include if feature involves data)*

- **Workspace**: Named local collection of prepared data, saved choices, protocol, and compatible
  completed results, with version and last-successful-save information.
- **Dataset snapshot**: Source, checksum, coverage, exclusions, and completeness assumptions.
- **Product sales history**: Product identity, date, positive units, and eligibility status.
- **Forecast origin**: Monday decision time, information cutoff, and seven-day target period.
- **Candidate result**: Model structure, fit status, daily predictions, weekly total, and score.
- **Predictive demand distribution**: Model-specific completed error history and integer demand
  support, with calibration size, intervals, and validation scores.
- **Inventory decision**: Product, origin, costs, recommended and benchmark quantities, and expected
  outcomes.
- **Policy evaluation**: Matched historical week, observed-sales proxy, and both simulated outcomes.
- **Monitoring record**: Completed error, EWMA state, frozen limits, and review alert.
- **Monte Carlo experiment**: Saved predictive support, fixed policies/costs, sample-size and seed
  configuration, repeated estimates, standard errors, sampled optima, and exact-model regret.
- **Experiment record**: Dataset and source versions, protocol, configuration, outputs, and limitations.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: By November 23, a manager can complete selection, forecast review, cost entry, and
  policy comparison for each of three eligible demonstration products in at most five minutes
  after their analyses are prepared, without editing program files.
- **SC-002**: Every supported demonstration product has all seven candidate outcomes recorded,
  with validation scores or explicit failure reasons, and a separate final-test report.
- **SC-003**: Each final-test report contains scored dates and counts, MAE, RMSE, 80%/95% coverage
  and mean width, and CRPS; every unsupported or undefined result has an explicit reason.
- **SC-004**: Every demonstrated inventory decision exposes a nonnegative integer quantity and
  all three expectations; known finite-distribution checks match exhaustive optima and expected
  costs within an absolute numerical tolerance of 0.00000001.
- **SC-005**: Every scored policy comparison uses identical held-out weeks and cost inputs and
  displays the benchmark quantity, recommended quantity, costs, leftovers, and unmet units.
- **SC-006**: Future-outcome changes leave earlier decisions unchanged in all temporal checks;
  no completed-week outcome enters its own forecast, uncertainty estimate, or inventory choice.
- **SC-007**: A clean-environment rerun reproduces selected methods and integer decisions exactly
  and numerical summaries within the recorded tolerances on the recorded reference environment.
- **SC-008**: Demonstration scenarios cover one eligible product, one unsupported product,
  invalid costs, and an error-monitor sequence with a sustained shift; assumptions and simulation
  labels are visible in every inventory report.

- **SC-009**: A browser refresh followed by a full app stop/start restores all saved working choices
  and completed compatible outputs without another source-file selection or model search; the
  reopened workspace remains usable when the original external source path is unavailable.
- **SC-010**: One clean-clone walkthrough completes setup, local database preparation, analysis,
  save, restart, and reopen using only the repository instructions. Two distinct workspaces restore
  their own data/settings, and stale-result and interrupted-save scenarios pass their checks.

- **SC-011**: At least one supported product/week has an experiment covering all three sample sizes
  and 100 repetitions, with exact references, Monte Carlo errors, standard errors, paired cost
  differences, and a convergence display. No monotone improvement or policy victory is required.
- **SC-012**: Fixed-draw numerical fixtures reproduce means and standard errors; small-distribution
  sampled optima match exhaustive integer search. Their exact expected-cost regret is nonnegative
  within absolute tolerance 0.00000001. Identical saved random configuration reproduces the
  experiment in the locked environment, and a new seed leaves primary exact results unchanged.

## Assumptions

- The intended user manages UK online-order inventory for independent products; no existing stock,
  lead-time pipeline, cross-product coupling, or replenishment is represented in this first version.
  The quantity is stock available at the start of the seven-day decision period.
- Observed positive sales proxy demand only under adequate availability and complete recording.
  Product offering periods, stockouts, and publication delays cannot be established from these
  transaction fields. This is a methodological demonstration, not an operational recommendation.
- Historical transaction timestamps are treated as UK local civil dates, available by the following
  Monday. Missing or revised reporting cannot be reconstructed and remains a disclosed limitation.
- Public dataset access and attribution are required. No private customer data, accounts,
  authentication, or live business integrations are needed; customer identifiers are not retained
  in analysis outputs.
- Approximately 5–6 hours per week are available. Three training-selected examples support thorough
  demonstration, while all products remain selectable and eligible products can run on demand.
- Numeric eligibility thresholds, split boundaries, tie rules, and computational settings in the
  technical plan are explicit design defaults, not claims that the raw data have already been
  inspected. They MUST be frozen before final-test results are inspected.
- Persistence is local to a user's own machine/workspace. Accounts, cross-device synchronization,
  shared multi-user workspaces, and hosted storage are outside this addition. CSV uploads remain
  deferred; future validated imports must use the same managed-workspace lifecycle.
- Monte Carlo is a bounded statistical computing experiment using the existing empirical weekly
  distribution. It adds no demand observations or tail support. More draws reduce numerical
  sampling error, not uncertainty caused by an incorrect demand model. Historical held-out sales
  replay remains the principal check of policy performance. No new forecasting model is introduced.
- There are no additional course-rubric constraints. The checkpoint is a working local project;
  public deployment and optional visual dashboards follow only if time remains afterward.
