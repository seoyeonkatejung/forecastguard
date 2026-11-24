# Research and Design Decisions

**Date**: 2026-10-26. Research supports a design, not a tested implementation. The original workbook
has not been downloaded, profiled, or used to choose products. Two focused research reviews covered
statistical design and official stack documentation; their findings are consolidated below.

## Dataset and intake

**Decision**: Online Retail II, UK positive-unit sales only; preserve cancellation/exclusion counts.
**Rationale**: Transaction data exercises validation and aggregation, while roughly two years allow
chronological calibration, selection, and testing. UCI documents missing values, wholesale customers,
and no inventory availability fields. Workbook schemas and sheet overlap require intake inspection.
**Alternatives considered**: M5 provides longer daily histories, but the user chose Online Retail II;
Online Retail I offers less history. CSV upload is deferred.
**Source**: [UCI dataset and license](https://archive.ics.uci.edu/dataset/502/online%2Bretail%2Bii).

## Model search and time ordering

**Decision**: Seven candidates; daily fits, weekly totals; separate calibration/selection/test.
**Rationale**: Bounded structure search includes trend and seasonality as requested. Fresh fits,
convergence recording, and common validation origins prevent easier failed-fold scoring. Parameters
can update prequentially without reselecting the structure on test performance.
**Alternatives considered**: A fixed smoothing model was explicitly rejected by the user. AIC-only
selection, random splits, annual seasonal search, and larger model families are excluded.
**Sources**: [Statsmodels constructor](https://www.statsmodels.org/stable/generated/statsmodels.tsa.holtwinters.ExponentialSmoothing.html),
[fit](https://www.statsmodels.org/stable/generated/statsmodels.tsa.holtwinters.ExponentialSmoothing.fit.html),
[pitfalls](https://www.statsmodels.org/stable/pitfalls.html),
[rolling-origin evaluation](https://otexts.com/fpp3/tscv.html).

## Predictive distribution and scoring

**Decision**: Use model-specific completed seven-day errors to construct an equally weighted,
nonnegative integer empirical distribution. Use its entire finite support for expectations and CRPS.
**Rationale**: Inventory costs require more information than interval endpoints. Weekly errors avoid
an unsupported independence assumption across days; their transferability remains an explicit model
assumption. Clipping/rounding must be validated after transformation. Exact finite-support arithmetic
needs no pseudo-random draws for primary results; the added Monte Carlo experiment supplies a
separate numerical comparison. Limited tail support must remain visible.
**Alternatives considered**: Parametric count models, recursive ETS innovation bootstrap, daily error
resampling, and treating prediction intervals as distributions. These add assumptions or effort;
the chosen aggregate-error approximation is not claimed equivalent to an ETS state-space model.
**Sources**: [Forecast uncertainty background](https://otexts.com/fpp3/prediction-intervals.html),
[proper scoring rules and CRPS](https://doi.org/10.1198/016214506000001437).
**Design-specific choices**: 26-error minimum, 52-error window, half-up rounding, and eligibility
cutoffs are project defaults, not guarantees supplied by those sources.

## Newsvendor and policy evaluation

**Decision**: Smallest empirical critical-ratio quantile; exact expectations; held-out historical
sales replay as the main simulation, compared with the previous eight-week mean policy.
**Rationale**: The discrete cost increment proves the quantile rule and supplies a direct numerical
oracle. Replay checks predictive errors against later outcomes rather than only the optimizer's own
objective. Observed sales still require an availability assumption to proxy demand.
**Alternatives considered**: A generic optimizer, arbitrary stock cap, shared budgets, and repeated
replenishment are unnecessary or explicitly deferred. Monte Carlo comparison using only fitted
forecast draws would not establish generalization.
**Source**: [Single-period inventory and demand-versus-sales discussion](https://rossetti.github.io/RossettiInventoryBook/07-Chapter7.html).

## Monitoring

**Decision**: Signed-error EWMA, lambda 0.2, frozen pre-test reference, empirical review threshold.
**Rationale**: One transparent monitor fits the time budget. Reference centering measures bias shifts;
reference bias itself stays visible. The review band is exploratory, without a false-alarm guarantee.
**Alternatives considered**: CUSUM, normal-theory limits without checking assumptions, and automatic
retraining. Dependence and small samples preclude casually importing idealized guarantees.
**Sources**: [NIST EWMA](https://www.itl.nist.gov/div898/handbook/pmc/section3/pmc314.htm),
[NIST on autocorrelated monitoring](https://www.nist.gov/publications/statistical-process-monitoring-autocorrelated-data).

## Minimal application stack

**Decision**: Pandas/openpyxl intake, DuckDB SQL aggregation, typed Python numerical modules,
Streamlit forms and serializable result caching, Plotly charts, pytest correctness fixtures.
**Rationale**: One local process avoids infrastructure. Data/protocol/product cache keys and a separate
cost form prevent model refits on cost changes. Daily point forecasts and weekly intervals remain
distinct in charts.
**Alternatives considered**: Separate web API, database server, optional Excel extension, additional
forecasting framework, and immediate public deployment. None is needed for the checkpoint.
**Sources**: [Pandas Excel intake](https://pandas.pydata.org/docs/reference/api/pandas.read_excel.html),
[DuckDB querying pandas](https://duckdb.org/docs/stable/guides/python/sql_on_pandas),
[Streamlit caching](https://docs.streamlit.io/develop/concepts/architecture/caching),
[forms](https://docs.streamlit.io/develop/concepts/architecture/forms),
[Plotly interval-band building blocks](https://plotly.com/python/filled-area-plots/).

## Local persistence and documentation (added 2026-11-23)

**Decision**: Named local workspace with a file-backed DuckDB database, managed source snapshot,
JSON working-state manifest, and reusable completed analysis bundles. Document clone/setup, local
database creation, save/reopen, backup/restore, and recovery. MkDocs is optional presentation.
**Rationale**: A user must be able to return after a browser or process restart without repeated
source loading. Working choices and version-compatible analysis results need durable storage;
transient UI state/cache alone is insufficient. A local workspace adds no account or database server.
**Alternatives considered**: Browser session only, requiring source selection each visit, immediate
cloud accounts/sync, and mandatory docs-site tooling. They either fail durable resume or expand scope.
**Sources**: [DuckDB file-backed storage](https://duckdb.org/docs/2.0/clients/python/overview#persistent-storage),
[Streamlit session-state lifetime](https://docs.streamlit.io/develop/api-reference/caching-and-state/st.session_state).
**Scope**: Built-in data first remains; future CSV import must reuse this lifecycle. The original
checkpoint schedule is not reset by this November 23 design addition. Local workflow estimates and
clean-environment validation remain to be checked during implementation.

## Monte Carlo statistical computing experiment (added 2026-11-23)

**Decision**: Sample the saved empirical seven-day distribution, using a recorded generator and
independent repetition streams; compare sizes 100/1,000/10,000 over 100 repetitions. Estimate both
fixed inventory policies with paired draws, report Monte Carlo errors against exact expectations,
and compare sampled-objective quantities using exact-model regret. Keep exact optimization primary.
**Rationale**: Existing exact finite-support results provide a numerical reference for expectation
estimation and simulation-based optimization. The experiment adds no forecasting family or new
claim about business savings. More draws address computational error, not an inadequate demand model.
**Alternatives considered**: Replacing exact expectations with noisy estimates, simulating all
products/weeks by default, choosing a favorable seed, or treating simulated within-model superiority
as evidence of generalization. Those choices add expense or misleading interpretation.
**Sources**: [NumPy Generator](https://numpy.org/doc/stable/reference/random/generator.html),
[sampling with replacement and weights](https://numpy.org/doc/stable/reference/random/generated/numpy.random.Generator.choice.html).
**Scope**: R=100, paired draws, nested sizes within each repetition, support-candidate enumeration,
and the 2–4 hour implementation estimate are explicit design defaults. Actual runtime and numerical
results require implementation. Historical held-out policy replay remains the principal comparison.

## Resolved uncertainties and remaining evidence

All decisions needed to plan are resolved. Exact source headers, duplicate counts, product eligibility,
package compatibility, fit convergence, runtime, and predictive performance require implementation
and intake evidence. They are not unresolved scope questions and are not represented as measured
facts. Freeze any audit-driven protocol correction before viewing final-test model performance.
