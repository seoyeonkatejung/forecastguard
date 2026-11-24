<!--
Sync Impact Report (temporary review material; remove before committing)
Version change: unratified scaffold → 1.0.0 (initial adoption)
Modified principles: placeholder slots replaced with:
  I. Statistical Correctness
  II. Reproducible Experiments
  III. Prevention of Future-Data Leakage
  IV. Explicit Evaluation Metrics
  V. Readable Python Code
  VI. Manageable Graduate Course Scope
Added sections: Experiment Requirements; Development Workflow and Review
Removed sections: none; generic section placeholders replaced with concrete sections.
Follow-up TODOs: none.
Dependent templates and commands: unchanged; read this constitution at runtime.
-->
# ForecastGuard Constitution

## Core Principles

### I. Statistical Correctness

Every experiment MUST define its target, prediction horizon, unit of analysis, sampling frequency,
and statistical assumptions. Model comparisons MUST use the same eligible observations, forecast
origins, horizons, and information availability. At least one appropriate simple forecasting
baseline MUST be evaluated under the same protocol. Missing values, outliers, transformations,
and exclusions MUST have documented handling rules. Statistical tests and uncertainty estimates,
when used, MUST account for temporal dependence and state their assumptions. Reports MUST
separate observed performance from claims of significance or causality and MUST NOT make
unsupported claims.

### II. Reproducible Experiments

Each reported result MUST identify a code revision or source snapshot, dependency versions,
dataset version or checksum, data acquisition instructions, split definitions, configuration,
and applicable random seeds. A documented command sequence MUST recreate preprocessing, training,
evaluation, and summaries from a clean environment. Runs MUST save effective configurations and
metrics. Reproduction MUST NOT depend on manual notebook state. Remaining nondeterminism MUST
have a documented cause and expected effect; numerical tolerances MUST be stated when exact
reproduction is not expected.

### III. Prevention of Future-Data Leakage

Predictions MUST use only information available at their forecast origin, accounting for publication
delays and dataset revisions. Training, validation, and test windows MUST be chronological; random
temporal splits MUST NOT be used for forecasting evaluation. Each fold MUST fit imputers, scalers,
encoders, feature selection, and other learned preprocessing solely on its training data. Rolling
features MUST exclude unavailable future observations. Training labels MUST be observable by the
forecast origin; gaps or purging MUST be applied when overlapping horizons would violate this rule.
Tuning and model selection MUST use training and validation data only. The final test window MUST
remain untouched until the evaluation protocol and model choices are fixed. Walk-forward updates
MUST follow a predefined policy and use only information available at each update. Automated checks
MUST verify temporal ordering, feature availability, and preprocessing boundaries.

### IV. Explicit Evaluation Metrics

Before model selection, experiments MUST declare a primary metric and any secondary metrics with
formulas, units, optimization direction, horizons, aggregation weights, and handling of missing
predictions, invalid values, and zero denominators. Metric choice MUST be justified for the target
and research question. Reports MUST include scored observation counts, evaluation dates, baseline
performance, and results by horizon when multiple horizons are used. Aggregation across series or
folds MUST be explicit and consistent across models. Undefined scores MUST be reported with their
cause. Changes to metrics or evaluation windows MUST be recorded and applied to all compared
models; a test set used to guide such changes MUST be disclosed as no longer untouched.

### V. Readable Python Code

Python code MUST use descriptive names, consistent formatting, explicit inputs and outputs, and
small functions with one clear purpose. Public functions MUST document behavior and provide type
hints; non-obvious statistical or temporal logic MUST explain its assumptions. Core preprocessing,
forecasting, and scoring logic MUST live in importable modules; notebooks MAY support exploration
and presentation. Focused automated tests MUST verify metrics against known examples, temporal
boundaries, and other logic whose failure could invalidate conclusions. Dependencies and
abstractions MUST have a concrete project need. Data integrity and evaluation errors MUST fail
visibly rather than yield plausible but invalid results.

### VI. Manageable Graduate Course Scope

The plan MUST define one primary research question, a bounded dataset and model set, required
deliverables, available compute, and milestones that fit the course deadline. A reproducible
end-to-end baseline MUST precede optional model complexity. Each addition MUST identify its
contribution to the research question and its time or compute cost. Additions exceeding the budget
MUST replace existing work or be deferred. Production infrastructure, extensive model searches,
and additional datasets MUST remain out of scope unless required to answer the primary question
within the documented budget. Completion MUST be judged by correct, reproducible evaluation and
clear limitations, regardless of whether a complex model wins.

## Experiment Requirements

Each experiment MUST have a concise protocol recording the research question, dataset provenance
and access requirements, target and horizons, timestamp and availability semantics, split dates,
preprocessing, baseline and candidate models, tuning budget, metrics, and compute budget.
Time zones, observation frequency, and assumptions about delayed or revised data MUST be explicit.
When historical availability cannot be reconstructed, the limitation MUST be reported and claims
MUST be restricted accordingly.

Results MUST preserve configurations, metrics, and prediction-level outputs needed to audit scores,
subject to dataset access restrictions. Restricted data MUST NOT be committed; documentation MUST
explain authorized acquisition or reconstruction. Reports MUST include failed or inconclusive
comparisons relevant to the research question and disclose departures from the planned protocol.

## Development Workflow and Review

Specifications and plans MUST map intended work to these principles and identify scope exclusions.
Implementation MUST proceed through data validation, a simple baseline, reproducible evaluation,
and then justified extensions. Before final test evaluation, the project MUST record the selected
model configuration, metric definitions, split boundaries, and any walk-forward update policy.

Changes affecting results MUST pass focused correctness and leakage checks and update the
experiment record when assumptions or behavior change. Before a milestone or final submission,
the author or reviewer MUST verify compliance, reproduce the documented workflow from a clean
environment, and confirm that reported metrics match saved outputs. Unresolved violations MUST
block presenting affected results as valid. Optional enhancements MUST be deferred when they
threaten required validation or the course deadline.

## Governance

This constitution takes precedence over conflicting project plans and implementation practices.
Amendments MUST record the change, rationale, affected principles, and any migration or
re-evaluation required. The project maintainer MUST approve and record amendments before dependent
work adopts them. Scope reductions MUST NOT waive statistical correctness, reproducibility,
leakage prevention, or honest evaluation.

Versions MUST follow semantic versioning: MAJOR for incompatible principle removals or
redefinitions, MINOR for new principles or materially expanded obligations, and PATCH for
clarifications preserving existing obligations. Amendments MUST update the version and last
amended date while preserving the original ratification date. Compliance MUST be reviewed during
specification, planning, change review, and final reporting. Conflicts MUST be resolved through a
compliant implementation change or an explicit constitution amendment.

**Version**: 1.0.0 | **Ratified**: 2026-10-26 | **Last Amended**: 2026-10-26
