# Data Model

**Status**: Proposed schemas; no data tables or implementation created. Identifiers remain strings.
All local timestamps use the documented UK civil-date assumption. Every derived object carries a
source checksum and protocol hash directly or through its run identifier.

| Entity | Principal fields | Key and invariants |
| --- | --- | --- |
| Workspace | workspace_id, name, schema_version, last_successful_save, relative_database_path, snapshot_id, protocol_hash, selected_product, selected_origin, costs, active_view, completed_run_refs | Local managed directory; atomic versioned manifest; validated choices; relative paths; last complete save survives failures. |
| DatasetSnapshot | snapshot_id, source_url, sha256, acquired_at, sheet_audit, coverage_start/end, aliases, exclusion_counts, completeness_assumption | Immutable raw checksum; row counts reconcile with included/excluded categories. |
| NormalizedTransaction | source_sheet, source_row, invoice_id, product_id, timestamp_local, quantity, country, optional description/price | Source row unique within sheet; invoice/product are not a unique line key. Valid integer quantities; cancellation/return flags preserved during audit. |
| DailySales | snapshot_id, product_id, date, units, source_row_count, filled_zero | Unique product/date; nonnegative integer units; no pre-first-sale padding. |
| ProductCatalogEntry | product_id, display_description, merchandise_status, status_reason | All catalog choices remain selectable; display metadata must not enter model selection or eligibility. |
| Protocol | protocol_hash, split_dates, forecast_days, origin_weekday, candidates, fit_settings, eligibility_rules, residual_window/minimum, rounding, metrics, EWMA settings, tolerance | Immutable for final-test run; horizon=7 days, Monday origins, no overlapping target weeks. |
| EligibilityRecord | product_id, origin, cutoff, history_weeks, active_weeks, trailing_zero_weeks, point_supported, distribution_supported, reasons | Computed from data before origin; distinct point and distribution support. |
| CandidateForecast | run_id, product_id, model_id, origin, cutoff, block, fit_status, failure_reason, warning_codes, daily_values, weekly_point, projection_count | Unique product/model/origin; seven finite nonnegative postprocessed daily values; weekly_point equals sum. |
| CompletedError | product_id, model_id, origin, target_end, observed_at, weekly_actual, weekly_point, error | Error=actual-point; availability begins only after target week; never available at its own origin. |
| PredictiveSupport | product_id, model_id, origin, bank_source_origins, demand_values, weights, support_size, clipped_fraction, predictive_mean, interval_endpoints | At least26 errors; source weeks completed before origin; values nonnegative integers; weights sum to1; duplicate support permitted. |
| ModelSelection | product_id, protocol_hash, validation_origins, candidate_scores/statuses, selected_model_id, tie_rule, selected_at_cutoff | Only validation MAE chooses winner; structure fixed before test. |
| MetricSummary | product_id, method, block, scored_dates, planned/eligible/scored/failed_counts, MAE, RMSE, CRPS, coverage80/95, width80/95, unavailable_reasons | Common dates for comparisons; null with reason instead of undefined or fabricated zero. |
| MonitoringReference | product_id, selected_model_id, reference_origins, mu_ref, lambda, H, final_reference_state, degenerate | Derived before test; fixed while test errors update state. |
| MonitoringRecord | product_id, origin, completed_error, centered_error, previous_z, z, review_alert | No update before completion; recurrence matches frozen reference. |
| InventoryDecision | product_id, origin, cost_overage, cost_underage, alpha, q_opt, q_benchmark, expected_cost/leftover/unmet for both, support_reference, warnings | Costs finite>0; quantities integer>=0; benchmark depends only on eight past weekly totals. |
| PolicyOutcome | product_id, origin, weekly_sales_proxy, both_quantities, both_costs, both_leftovers, both_unmet, paired_savings | Same historical outcome and costs for both; savings=benchmark-optimized and may be negative. |
| RunManifest | run_id, source_revision/hash, python_version, dependency_lock_hash, protocol_hash, snapshot_id, seeds, generated_at, artifacts, runtime | Supports clean rerun; records failures and does not include customer identifiers. |

## Relationships and lifecycle

A workspace owns a snapshot, prepared database, saved working choices, and completed run references.
Opening it validates schema and result compatibility before rehydrating choices; no external source
reselection is needed. Dataset/protocol/code changes invalidate affected results without deleting
prepared data. Results always retain their original information cutoff.

A snapshot yields catalog choices and daily histories. A protocol and history produce per-origin
eligibility and candidate forecasts. Once the target week completes, a valid forecast gains an
observed total and a CompletedError. Only then may that error enter a later PredictiveSupport.
A product's validation forecasts produce a ModelSelection, after which its selected structure stays
fixed for test. Pre-test errors generate MonitoringReference; completed test errors update records.
PredictiveSupport plus entered costs yields InventoryDecision. A later completed sales week yields
PolicyOutcome. RunManifest ties the full chain together.

Unavailable is a first-class state: `history_only`, `point_only`, `ready`, or `failed`, with reasons.
A cost-only change creates different decisions/outcomes against existing supports, not a different
forecast search. Never retroactively overwrite a support with errors observed later.
