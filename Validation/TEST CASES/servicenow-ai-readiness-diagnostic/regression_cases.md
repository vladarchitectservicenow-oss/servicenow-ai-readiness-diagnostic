# Regression Cases: servicenow-ai-readiness-diagnostic

**Author:** Vladimir Kapustin | **License:** AGPL-3.0-only
**Scope:** x_snc_ard

---

## Purpose

Ensure that changes to the AI Readiness Diagnostic codebase do not break existing functionality. All regression cases must pass after any code modification.

---

## Case Catalog

### R01 — Idempotent Diagnostic Execution

| Attribute | Value |
|-----------|-------|
| Trigger | Run `POST /diagnose` twice sequentially |
| Expected Behavior | Both runs produce valid diag_run records with independent data |
| Verification | Two diag_run records exist with different sys_ids, both have status COMPLETED |
| Risk if Broken | Duplicate records overwrite each other, loss of historical data |

### R02 — Workflow Health Score Consistency

| Attribute | Value |
|-----------|-------|
| Trigger | Run diagnostic on same instance twice without changes to workflows |
| Expected Behavior | health_score and issue_count are identical (±0) for the same workflow between runs |
| Verification | Compare x_snc_ard_workflow_health records for same workflow_sys_id across two diag_runs |
| Risk if Broken | Score drift causes false-positive alerts, roadmap recommendations change without cause |

### R03 — CMDB Metrics Stability

| Attribute | Value |
|-----------|-------|
| Trigger | Run CMDBHealthAnalyzer.analyze("cmdb_ci") twice on same CI state |
| Expected Behavior | completeness_pct, orphan_pct, stale_pct, ai_impact_score identical across runs |
| Verification | Diff x_snc_ard_cmdb_health records across two diag_runs for same ci_class |
| Risk if Broken | Non-deterministic scoring undermines trust in AI readiness assessment |

### R04 — REST API Response Schema Compatibility

| Attribute | Value |
|-----------|-------|
| Trigger | Call GET /dashboard, GET /simulate, GET /diagnose/{id} after code update |
| Expected Behavior | Response JSON schema unchanged — same keys, same types, same structure |
| Verification | Validate against stored schema snapshot from previous version |
| Risk if Broken | CI/CD integrations break, dashboards fail to render |

### R05 — KB Analysis Non-Negative Returns

| Attribute | Value |
|-----------|-------|
| Trigger | KBAnalyzer.analyze() on any instance state |
| Expected Behavior | All percentage values are 0-100, knowledge_readiness is 0-100, total_articles ≥ 0 |
| Verification | Assert all numeric fields in valid range, no negative values |
| Risk if Broken | Negative readiness scores cause UI rendering errors |

### R06 — Roadmap Generator Output Completeness

| Attribute | Value |
|-----------|-------|
| Trigger | RoadmapGenerator.generate() after full diagnostic |
| Expected Behavior | quick_wins + phase_1_30d + phase_2_90d + phase_3_180d items sum to total_items |
| Verification | Sum of all 4 array lengths === total_items |
| Risk if Broken | Missing items from roadmap, silent data loss |

### R07 — Simulation No Cross-Contamination

| Attribute | Value |
|-----------|-------|
| Trigger | Run AISimulator.simulateAll() with diagRunId1, then with diagRunId2 |
| Expected Behavior | Simulation results for run2 only reference run2's workflow_health records |
| Verification | All x_snc_ard_simulation_result.diag_run values match the passed diagRunId |
| Risk if Broken | Simulation results from wrong diagnostic run, incorrect AI readiness assessment |

### R08 — Scheduled Job Completion Status

| Attribute | Value |
|-----------|-------|
| Trigger | Execute weekly_health_scan.js via Background Script |
| Expected Behavior | x_snc_ard_diag_run record created with status COMPLETED, completed_at IS NOT NULL |
| Verification | Query x_snc_ard_diag_run WHERE sys_created_on > gs.minutesAgo(5), assert status = COMPLETED |
| Risk if Broken | Scheduled job silently fails, no alerts, stale AI readiness data |

### R09 — REST API Error Handling Format

| Attribute | Value |
|-----------|-------|
| Trigger | Call GET /diagnose/nonexistent_id, GET /simulate with no diagnostics, POST with malformed body |
| Expected Behavior | All errors return consistent JSON format: { error: "description" } with appropriate HTTP status |
| Verification | 404 returns { error: "Not found" }, 404 for no diagnostic returns { error: "No completed diagnostic" } |
| Risk if Broken | Consumers can't parse error responses, retry storms |

---

## Execution

```bash
# Run regression suite after any src/ change
# 1. Deploy updated code to PDI
# 2. Run full diagnostic: POST /api/x_snc_ard/v1/diagnose
# 3. Wait for COMPLETED status
# 4. Verify all R01-R09 checks manually or via Background Script
```

## Pass Criteria

- All 9 regression cases must pass
- Any failure blocks merge/promotion
