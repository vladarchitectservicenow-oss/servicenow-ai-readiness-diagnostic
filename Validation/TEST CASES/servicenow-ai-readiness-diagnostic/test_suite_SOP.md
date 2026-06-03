# Test Suite SOP: servicenow-ai-readiness-diagnostic

**Author:** Vladimir Kapustin | **License:** AGPL-3.0-only
**Scope:** x_snc_ard | **Test Framework:** ServiceNow PDI + Node.js Unit Mocks

---

## Overview

This SOP defines the complete test strategy for AI Readiness Diagnostic. All tests must pass before marking a build as production-ready. Minimum 10/10 scenarios PASS required.

## Test Environment

- **PDI:** devNNNNN.service-now.com (Zurich+)
- **Background Script:** `/sys.scripts.do`
- **Unit Tests:** Node.js with mocked GlideRecord/GlideAggregate/GlideDateTime

---

## Scenario Catalog

### T01 — ProcessDebtScanner: Full Scan Pipeline

| Attribute | Value |
|-----------|-------|
| Priority | P0 |
| Module | ProcessDebtScanner |
| Description | Execute fullScan() and verify all three scan phases complete |
| Input | Instance with ≥1 active wf_workflow, ≥1 active sys_flow |
| Expected Output | { diag_run_id: string, workflows_scanned: ≥1, broken_count: integer } |
| Validation | diag_run_id !== null, workflows_scanned > 0, broken_count >= 0 |
| Negative Case | Instance with zero workflows → workflows_scanned = 0, broken_count = 0 |
| Test Script | `new x_snc_ard.ProcessDebtScanner().fullScan()` |

### T02 — CMDBHealthAnalyzer: Completeness Calculation

| Attribute | Value |
|-----------|-------|
| Priority | P0 |
| Module | CMDBHealthAnalyzer |
| Description | Verify completeness_pct is calculated correctly for cmdb_ci class |
| Input | Standard PDI with default cmdb_ci records |
| Expected Output | { ci_class: "cmdb_ci", total_cis: ≥1, completeness_pct: 0-100, ai_impact_score: 0-100 } |
| Validation | completeness_pct is integer 0-100, total_cis matches GlideAggregate COUNT |
| Negative Case | Non-existent CI class → total_cis = 0, all percentages = 0, ai_impact_score = 0 |
| Test Script | `new x_snc_ard.CMDBHealthAnalyzer(null).analyze("cmdb_ci")` |

### T03 — CMDBHealthAnalyzer: Orphan Detection

| Attribute | Value |
|-----------|-------|
| Priority | P0 |
| Module | CMDBHealthAnalyzer |
| Description | Verify orphan_pct identifies CIs with no relationships |
| Input | Instance with cmdb_ci records, some having cmdb_rel_ci relationships |
| Expected Output | orphan_pct integer 0-100, derived from /cmdb_rel_ci negative check |
| Validation | orphan_pct > 0 when isolated CIs exist, orphan_pct = 0 when all CIs have relationships |
| Negative Case | Empty cmdb_rel_ci → orphan_pct near 100% |
| Test Script | `new x_snc_ard.CMDBHealthAnalyzer(null).analyze("cmdb_ci")` |

### T04 — KBAnalyzer: Full KB Analysis

| Attribute | Value |
|-----------|-------|
| Priority | P0 |
| Module | KBAnalyzer |
| Description | Verify KB health analysis returns correct readiness score |
| Input | Instance with kb_knowledge articles |
| Expected Output | { total_articles: ≥0, empty_pct: 0-100, outdated_pct: 0-100, unattached_pct: 0-100, knowledge_readiness: 0-100 } |
| Validation | knowledge_readiness = 100 - empty_pct*0.4 - outdated_pct*0.35 - unattached_pct*0.25, clamped to 0-100 |
| Negative Case | Zero articles → total_articles = 0, knowledge_readiness = 0 |
| Test Script | `new x_snc_ard.KBAnalyzer(null).analyze()` |

### T05 — REST API: POST /diagnose Endpoint

| Attribute | Value |
|-----------|-------|
| Priority | P0 |
| Module | diagnostic_api.js |
| Description | Verify POST /api/x_snc_ard/v1/diagnose returns 200 with valid response |
| Input | Authenticated as snc_ard.admin |
| Expected Output | HTTP 200, JSON { diag_run_id, status: "COMPLETED", workflows_scanned, broken_count } |
| Validation | response status = 200, diag_run_id is valid sys_id, status = "COMPLETED" |
| Negative Case | Unauthenticated → HTTP 401 |
| Test Method | REST API Explorer or curl |

### T06 — REST API: GET /dashboard Endpoint

| Attribute | Value |
|-----------|-------|
| Priority | P0 |
| Module | diagnostic_api.js |
| Description | Verify dashboard returns aggregated readiness score |
| Input | Authenticated as snc_ard.viewer, at least one completed diagnostic |
| Expected Output | { readiness_score: 0-100, workflow_health: 0-100, cmdb_health: 0-100, kb_health: 0-100 } |
| Validation | readiness_score = workflow_health*0.4 + cmdb_health*0.35 + kb_health*0.25 (rounded) |
| Negative Case | No completed diagnostic → readiness_score = 0, message: "No diagnostic yet" |
| Test Method | REST API Explorer or curl |

### T07 — REST API: GET /simulate Endpoint

| Attribute | Value |
|-----------|-------|
| Priority | P1 |
| Module | diagnostic_api.js |
| Description | Verify simulation returns PASS/WARNING/FAIL breakdown |
| Input | Authenticated, latest completed diagnostic |
| Expected Output | { pass: N, warning: M, fail: K, total: N+M+K, results: [...] } |
| Validation | pass + warning + fail = total, each result has result and gap fields |
| Negative Case | No completed diagnostic → HTTP 404, error: "No completed diagnostic" |
| Test Method | REST API Explorer or curl |

### T08 — AISimulator: Cross-Reference Logic

| Attribute | Value |
|-----------|-------|
| Priority | P1 |
| Module | AISimulator |
| Description | Verify AISimulator correctly adjusts results based on CMDB and KB health |
| Input | Completed diagnostic with workflow_health, cmdb_health, kb_health records |
| Expected Output | Counts of PASS/WARNING/FAIL match expected for the given input state |
| Validation | When CMDB completeness < 50, non-FAIL workflows upgraded to WARNING with LOW_CMDB_COMPLETENESS gap |
| Test Script | `new x_snc_ard.AISimulator(diagRunId).simulateAll()` |

### T09 — RoadmapGenerator: Priority Phasing

| Attribute | Value |
|-----------|-------|
| Priority | P1 |
| Module | RoadmapGenerator |
| Description | Verify roadmap generates correct 30/90/180-day phases |
| Input | Completed diagnostic with workflow_health records |
| Expected Output | { quick_wins: [...], phase_1_30d: [...], phase_2_90d: [...], phase_3_180d: [...], total_items, total_estimated_hours } |
| Validation | Items sorted by priority DESC, phase assignment based on priority threshold (≥30 → 30d, ≥15 → 90d, else → 180d) |
| Test Script | `new x_snc_ard.RoadmapGenerator(diagRunId).generate()` |

### T10 — Scheduled Job: Weekly Health Scan Execution

| Attribute | Value |
|-----------|-------|
| Priority | P1 |
| Module | weekly_health_scan.js |
| Description | Verify scheduled job runs full pipeline without errors |
| Input | Execute as Background Script with gs.info interception |
| Expected Output | gs.info log with "ARD Weekly Scan: N workflows, M broken" |
| Validation | No exceptions thrown, diag_run created with status COMPLETED |
| Test Method | Background Script execution |

### T11 — Edge: Concurrent Diagnostic Prevention

| Attribute | Value |
|-----------|-------|
| Priority | P2 |
| Module | diagnostic_api.js |
| Description | Verify system rejects duplicate POST /diagnose while run is active |
| Input | POST /diagnose while previous run has status RUNNING |
| Expected Output | HTTP 409 or return existing diag_run_id |
| Current Behavior | No guard currently (see risk R06) — test for future implementation |
| Test Method | Sequential POST calls with <1s gap |

### T12 — Edge: Empty Instance Graceful Degradation

| Attribute | Value |
|-----------|-------|
| Priority | P2 |
| Module | All scanners |
| Description | Verify full pipeline handles zero workflows, zero CIs, zero KB articles |
| Input | Clean PDI with no data in scanned tables |
| Expected Output | workflows_scanned=0, broken_count=0, total_cis=0, completeness_pct=0, total_articles=0, knowledge_readiness=0 |
| Validation | No exceptions, no NaN, no null dereference, dashboard returns readiness_score=0 |

---

## Execution

```bash
# PDI execution
# Navigate to /sys.scripts.do, paste Background Script below

// Run full pipeline
var scanner = new x_snc_ard.ProcessDebtScanner();
var result = scanner.fullScan();
gs.info("Scanner: " + JSON.stringify(result));

var cmdb = new x_snc_ard.CMDBHealthAnalyzer(scanner.diagRunId);
var cmdbResult = cmdb.analyze("cmdb_ci");
gs.info("CMDB: " + JSON.stringify(cmdbResult));

var kb = new x_snc_ard.KBAnalyzer(scanner.diagRunId);
var kbResult = kb.analyze();
gs.info("KB: " + JSON.stringify(kbResult));

var sim = new x_snc_ard.AISimulator(scanner.diagRunId);
var simResult = sim.simulateAll();
gs.info("Simulation: " + JSON.stringify(simResult));

var roadmap = new x_snc_ard.RoadmapGenerator(scanner.diagRunId);
var rmResult = roadmap.generate();
gs.info("Roadmap: " + JSON.stringify(rmResult));
```

## Pass Criteria

- Minimum 10/12 scenarios must PASS
- T01-T05 (P0) must ALL pass — no exceptions
- T06-T10 (P1) must ALL pass
- T11-T12 (P2) may have documented exceptions (e.g., concurrent prevention not yet implemented)
