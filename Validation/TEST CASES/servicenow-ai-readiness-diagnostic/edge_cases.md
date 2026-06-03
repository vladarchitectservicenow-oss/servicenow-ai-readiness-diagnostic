# Edge Cases: servicenow-ai-readiness-diagnostic

**Author:** Vladimir Kapustin | **License:** AGPL-3.0-only
**Scope:** x_snc_ard

---

## Purpose

Document edge cases that may not be covered by standard test scenarios or regression cases. These represent boundary conditions, unusual states, and failure modes.

---

## Case Catalog

### E01 — Empty Instance: Zero Workflows, Zero CIs, Zero KB Articles

| Attribute | Value |
|-----------|-------|
| Trigger | Full diagnostic on a freshly provisioned PDI with no data |
| Expected Behavior | workflows_scanned=0, broken_count=0, total_cis=0, completeness_pct=0, total_articles=0, knowledge_readiness=0 |
| Edge Concern | Division by zero in percentage calculations, null dereference |
| Current Handling | KBAnalyzer returns early with readiness=0 if total_articles=0; CMDBHealthAnalyzer guards all calculations behind total_cis===0 check |
| Verification | Run full diagnostic on clean PDI, inspect all returned objects for NaN/null |

### E02 — 50K+ Records: Maximum Scan Boundary

| Attribute | Value |
|-----------|-------|
| Trigger | Instance with >500 workflows, >200 CIs of a single class |
| Expected Behavior | Scan caps at setLimit values (500 workflows, 200 CIs for orphan check), processes without timeout |
| Edge Concern | GlideRecord query timeout, transaction limits, excessive log output |
| Current Handling | setLimit(500) on workflow scan, setLimit(200) on orphan check — sample-based reporting |
| Verification | Test on instance with known large table counts, verify no timeout errors |

### E03 — Null/Undefined System Properties

| Attribute | Value |
|-----------|-------|
| Trigger | Required system properties (if any config-driven) missing or null |
| Expected Behavior | Scanners use hardcoded defaults, no property-dependent failure |
| Edge Concern | Null property causes undefined behavior in initialized() or parameter reading |
| Current Handling | All scanners use direct API calls with no config property dependencies currently |
| Verification | Run diagnostic after deleting all x_snc_ard.* system properties |

### E04 — Corrupted issues_json Field

| Attribute | Value |
|-----------|-------|
| Trigger | x_snc_ard_workflow_health.issues_json contains malformed JSON |
| Expected Behavior | RoadmapGenerator handles JSON.parse failure gracefully with empty array fallback |
| Edge Concern | JSON.parse throws uncaught exception, roadmap generation fails entirely |
| Current Handling | RoadmapGenerator uses `JSON.parse(whGr.getValue("issues_json") || "[]")` — empty string defaults to "[]" |
| Verification | Manually insert corrupted JSON value, run RoadmapGenerator.generate(), verify no crash |

### E05 — Unicode/Special Characters in Field Names

| Attribute | Value |
|-----------|-------|
| Trigger | Workflow names or CI names contain Unicode (Japanese, Chinese, emoji), <, >, &, ", ' |
| Expected Behavior | Values stored and retrieved correctly in JSON, no XSS injection risk |
| Edge Concern | JSON.stringify breaks on special chars, HTML rendering of reports shows raw HTML |
| Current Handling | ServiceNow GlideRecord.getValue() handles Unicode, JSON.stringify escapes quotes |
| Verification | Create CI/workflow with name containing `テスト CI <script>alert(1)</script>`, run scan, verify stored correctly |

### E06 — Concurrent GlideRecord Writes

| Attribute | Value |
|-----------|-------|
| Trigger | Two diagnostic runs started within milliseconds (scheduled job + manual API call) |
| Expected Behavior | Each creates independent diag_run record, no duplicate workflow_health for same workflow in same run |
| Edge Concern | Race condition on x_snc_ard_diag_run insert, shared state in scanner instance |
| Current Handling | No concurrency guard currently — each scanner instance creates its own diag_run via ProcessDebtScanner._createDiagRun |
| Verification | Trigger two Background Script executions simultaneously, verify two distinct diag_runs created |

### E07 — Timeout During Scan

| Attribute | Value |
|-----------|-------|
| Trigger | ProcessDebtScanner.fullScan() exceeds transaction timeout (default 60s) |
| Expected Behavior | Transaction rolls back partial diag_run record, no orphaned records |
| Edge Concern | Partially written workflow_health records with no parent diag_run |
| Current Handling | All writes happen within same transaction — GlideRecord insert in scope means transaction-level rollback |
| Verification | Trigger timeout by scanning very large instance, verify no orphaned x_snc_ard_* records |

### E08 — Deleted Workflow Between Scan and Simulation

| Attribute | Value |
|-----------|-------|
| Trigger | Workflow deleted after ProcessDebtScanner runs but before AISimulator.simulateAll() runs |
| Expected Behavior | Simulation skips deleted workflow, does not throw NullPointerException |
| Edge Concern | workflow_health record references deleted workflow_sys_id, getUniqueValue() returns null on GlideRecord.get() |
| Current Handling | AISimulator uses issue data from workflow_health record (already serialized) and makes only read-access GlideRecord calls for CMDB/KB cross-reference; deleted workflow_sys_id would cause no issue since issues_json is self-contained |
| Verification | Delete workflow mid-pipeline (between scan and sim), verify no error |

---

## Execution

```bash
# Test edge cases in order:
# 1. Fresh PDI - test E01
# 2. Data-loaded PDI - test E02, E04, E05, E06, E08
# 3. Config manipulation - test E03, E07
```

## Pass Criteria

- Minimum 5/8 edge cases must pass
- E01 (empty instance) is mandatory — must pass
- E02-E08 may have documented limitations for future versions
