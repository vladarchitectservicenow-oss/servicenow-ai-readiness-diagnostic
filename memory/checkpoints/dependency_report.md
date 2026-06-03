# servicenow-ai-readiness-diagnostic — Dependency Report

**Product:** AI Readiness Diagnostic (x_snc_ard)
**Author:** Vladimir Kapustin
**Date:** 2026-06-03
**Platform:** ServiceNow Zurich+

---

## 1. ServiceNow Platform Dependencies

### 1.1 Required Plugins

| Plugin ID | Plugin Name | Purpose | Criticality |
|-----------|-------------|---------|-------------|
| com.glide.workflow | Workflow Engine | Core workflow scanning (wf_workflow table) | P0 — App non-functional without it |
| com.glide.flow_designer | Flow Designer | Flow scanning (sys_flow, sys_hub_flow tables) | P0 — Flows comprise ~40% of scanned artifacts |
| com.glide.knowledge | Knowledge Management | KB article scanning (kb_knowledge table) | P1 — KBAnalyzer disabled without it |
| com.snc.cmdb | Configuration Management | CMDB CI scanning (cmdb_ci, cmdb_rel_ci) | P0 — CMDBHealthAnalyzer disabled without it |
| com.glideapp.itom.snac | ServiceNow Agent Client | Agent integration support | P2 — AISimulator cross-reference enhancement |

### 1.2 Core Tables (Platform)

| Table | Access Type | Purpose |
|-------|-------------|---------|
| `cmdb_ci` | Read | CI completeness, staleness, orphan analysis |
| `cmdb_rel_ci` | Read | Orphan detection via relationship absence |
| `wf_workflow` | Read | Legacy workflow scan |
| `sys_flow` | Read | Flow Designer flow scan |
| `sys_hub_flow` | Read | IntegrationHub flow scan |
| `sys_flow_step` | Read | Flow step count, error detection |
| `kb_knowledge` | Read | KB article metrics |
| `sys_user` | Read | Owner activity status check |

### 1.3 System Roles

| Role | Purpose | Access Level |
|------|---------|-------------|
| `snc_ard.admin` | Run diagnostics, view all results, schedule jobs | Full CRUD on x_snc_ard tables |
| `snc_ard.viewer` | View dashboards, query diagnostic results | Read-only on x_snc_ard tables |
| `snc_ard.integration` | REST API access for external CI/CD pipelines | API endpoints only |
| `admin` | Platform admin — full access | Implicit superuser |

## 2. Runtime Dependencies (Server-Side)

### 2.1 GlideRecord APIs

All scanners use the following GlideRecord patterns:
- `GlideRecord(table).addQuery().query()` — record iteration
- `GlideRecord(table).get(sys_id)` — single-record lookup
- `GlideAggregate(table).addAggregate("COUNT"/"AVG")` — aggregated metrics
- `gr.getUniqueValue()` — sys_id retrieval
- `gr.getValue(field)` — field value access
- `gr.getRowCount()` — result count without iteration
- `gr.setLimit(n)` — pagination cap
- `gr.addActiveQuery()` — filter to active records only

### 2.2 GlideDateTime

Used in `ProcessDebtScanner._finalizeDiagRun()` for completion timestamps and `CMDBHealthAnalyzer._measureStaleness()` for year-ago comparison via `gs.yearsAgo(1)`.

### 2.3 JSON

`JSON.parse()` and `JSON.stringify()` used for issues_json serialization in workflow health records.

### 2.4 gs Global

- `gs.info()` — scheduled job logging
- `gs.yearsAgo(n)` — staleness threshold (CMDB: 1 year, KB: 2 years)

## 3. REST API Dependencies

### 3.1 Scripted REST API Framework

- `sn_ws.RESTAPIRequest` / `sn_ws.RESTAPIResponse` — implicit via ServiceNow Scripted REST framework
- All 4 endpoints mapped to `x_snc_ard` scope
- No external REST calls — internal-only API

## 4. Scheduled Job Dependencies

| Dependency | Type | Version Required |
|------------|------|-----------------|
| sys_trigger | Scheduled Job Executor | Zurich+ |
| GlideSchedule | Job scheduling framework | Zurich+ |

## 5. Build & Test Dependencies

### 5.1 ServiceNow Studio

- Scoped application development via Studio or PDI
- sys_app.xml export for source control

### 5.2 Validation (External)

| Tool | Version | Purpose |
|------|---------|---------|
| Node.js | 20+ | Unit test execution with mock ServiceNow APIs |
| Python 3 | 3.10+ | Quality gate checker, GitHub API verification |
| pytest | 7+ | Test runner |
| Git | 2.40+ | Version control |

## 6. Version Compatibility Matrix

| Component | Zurich | Australia | Washington DC |
|-----------|--------|-----------|---------------|
| Workflow Engine | ✓ | ✓ | ✓ |
| Flow Designer | ✓ | ✓ | ✓ |
| CMDB | ✓ | ✓ | ✓ |
| Knowledge Management | ✓ | ✓ | ✓ |
| Scripted REST API | ✓ | ✓ | ✓ |
| GlideAggregate | ✓ | ✓ | ✓ |
| Scheduled Jobs | ✓ | ✓ | ✓ |

## 7. Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| Flow Designer plugin deactivated | P1 | Graceful skip of flow scanning, log warning |
| CMDB plugin deactivated | P0 | CMDBHealthAnalyzer disabled, surface in dashboard |
| KB plugin deactivated | P1 | KBAnalyzer disabled, score defaults to 0 |
| Large instance (>100K CIs) | P2 | setLimit(200) on orphan check, setLimit(500) on workflow scan |
| Concurrent diagnostic runs | P2 | diag_run isolation ensures idempotency |
