# servicenow-ai-readiness-diagnostic — Risk Report

**Product:** AI Readiness Diagnostic (x_snc_ard)
**Author:** Vladimir Kapustin
**Date:** 2026-06-03

---

## Risk Catalog

### R01 — P0 Critical: CMDB plugin deactivated

| Attribute | Value |
|-----------|-------|
| Severity | P0 — Critical |
| Component | CMDBHealthAnalyzer |
| Trigger | `com.snc.cmdb` plugin not installed or deactivated |
| Impact | CMDB health metrics missing from dashboard, readiness score degraded by 35%, AISimulator cross-reference disabled |
| Likelihood | Low (CMDB is core platform) |
| Detection | Startup check in `CMDBHealthAnalyzer.initialize()` |
| Mitigation | Skip CMDB analysis, set cmdb_health=0 in dashboard, log warning, schedule runs only after CMDB re-activated |
| Recovery Time | Instant — re-activate plugin, re-run diagnostic |

### R02 — P0 Critical: Workflow Engine plugin deactivated

| Attribute | Value |
|-----------|-------|
| Severity | P0 — Critical |
| Component | ProcessDebtScanner |
| Trigger | `com.glide.workflow` plugin deactivated |
| Impact | Workflow scanning disabled (40% of readiness score), table `wf_workflow` inaccessible |
| Likelihood | Very Low |
| Detection | `GlideRecord("wf_workflow").canRead()` check |
| Mitigation | Skip workflow scanning, log critical event, mark diagnostic as partial |
| Recovery Time | < 5 min — re-activate plugin |

### R03 — P1 High: Flow Designer plugin deactivated

| Attribute | Value |
|-----------|-------|
| Severity | P1 — High |
| Component | ProcessDebtScanner._scanFlows |
| Trigger | `com.glide.flow_designer` plugin deactivated |
| Impact | `sys_flow` and `sys_hub_flow` scanning disabled (~15% of artifacts missed) |
| Likelihood | Low |
| Detection | Table access check before scan |
| Mitigation | Skip flow tables, continue with wf_workflow only, log warning |
| Recovery Time | < 5 min |

### R04 — P1 High: KB plugin deactivated

| Attribute | Value |
|-----------|-------|
| Severity | P1 — High |
| Component | KBAnalyzer |
| Trigger | `com.glide.knowledge` plugin deactivated |
| Impact | KB readiness score = 0, dashboard knowledge readiness = 0%, overall readiness degraded by 25% |
| Likelihood | Low-Medium (optional in some deployments) |
| Detection | `KBAnalyzer.analyze()` gracefully returns {total_articles:0, knowledge_readiness:0} |
| Mitigation | Zero-KB score accepted as valid state, dashboard shows "KB not configured" |
| Recovery Time | < 10 min |

### R05 — P1 High: Large instance — scan timeout

| Attribute | Value |
|-----------|-------|
| Severity | P1 — High |
| Component | ProcessDebtScanner.fullScan |
| Trigger | Instance with >10K workflows or CIs |
| Impact | Scan exceeds scheduled job timeout, partial results |
| Likelihood | Medium (enterprise instances often have 5-20K workflows) |
| Detection | `setLimit(500)` on workflow scan, `setLimit(200)` on orphan check — caps exist but may still be slow |
| Mitigation | Chunk scanning by type (wfs first, flows second), reduce limits on large instances |
| Recovery Time | Per-run — next invocation continues from last checkpoint |

### R06 — P1 High: Concurrent diagnostic runs

| Attribute | Value |
|-----------|-------|
| Severity | P1 — High |
| Component | REST API POST /diagnose |
| Trigger | Two API calls or a cron + manual overlap |
| Impact | Duplicate diag_run records, conflicting AISimulator results referencing different diag_runs |
| Likelihood | Medium |
| Detection | Check for RUNNING diag_run status before creating new |
| Mitigation | Add `status=RUNNING` check, reject if active run exists, return existing run ID |
| Recovery Time | Auto — second invocation returns first run's ID |

### R07 — P2 Medium: Orphan check limited to 200 CIs

| Attribute | Value |
|-----------|-------|
| Severity | P2 — Medium |
| Component | CMDBHealthAnalyzer._measureOrphans |
| Trigger | Instance has >200 CIs matching the class filter |
| Impact | Orphan percentage sample-based, not population-exact |
| Likelihood | High (most enterprise instances have thousands of CIs) |
| Detection | Compare `setLimit(200)` count to `total_cis` |
| Mitigation | Accept as sampling estimate, label as "sample-based" in dashboard |
| Recovery Time | Not recoverable — design limitation |

### R08 — P2 Medium: Staleness uses sys_updated_on heuristic

| Attribute | Value |
|-----------|-------|
| Severity | P2 — Medium |
| Component | CMDBHealthAnalyzer._measureStaleness |
| Trigger | CIs updated by automation scripts (not human) resetting staleness artificially |
| Impact | Staleness underreported — automated discovery updates timestamps without human review |
| Likelihood | High (Discovery/IRE run frequently in managed instances) |
| Detection | None — no way to distinguish auto-update from human update |
| Mitigation | Document as known limitation, recommend separate audit for auto-maintained CIs |
| Recovery Time | N/A — requires Discovery integration |

### R09 — P2 Medium: issues_json field size limits

| Attribute | Value |
|-----------|-------|
| Severity | P2 — Medium |
| Component | x_snc_ard_workflow_health.issues_json |
| Trigger | Workflow with >50 issues generating large JSON payload |
| Impact | issues_json exceeds field limit (default 4000 chars) |
| Likelihood | Low (most workflows have <10 issues) |
| Detection | JSON.stringify length check before insert |
| Mitigation | Truncate issues array to top 20 by weight before serialization |
| Recovery Time | Auto on next scan |

### R10 — P2 Medium: No incremental/delta scan

| Attribute | Value |
|-----------|-------|
| Severity | P2 — Medium |
| Component | All scanners |
| Trigger | Every diagnostic is a full scan |
| Impact | Redundant processing of unchanged workflows/CI/KB, wasted compute |
| Likelihood | Certain (every invocation is full scan) |
| Detection | N/A — by design |
| Mitigation | Roadmap v1.1: add delta scanning comparing sys_updated_on to last scan |
| Recovery Time | v1.1 release |

### R11 — P3 Low: README has license mismatch (MIT badge vs AGPL-3.0)

| Attribute | Value |
|-----------|-------|
| Severity | P3 — Low |
| Component | README.md |
| Trigger | README header shows MIT badge, body says AGPL-3.0 |
| Impact | Confusion for downstream users, no legal risk (LICENSE file is authoritative) |
| Likelihood | Certain (current state) |
| Detection | G7 quality gate |
| Mitigation | Fix badge to AGPLv3 |
| Recovery Time | 1 commit |

### R12 — P3 Low: Missing .gitignore

| Attribute | Value |
|-----------|-------|
| Severity | P3 — Low |
| Component | Repository root |
| Trigger | No .gitignore present |
| Impact | Accidental check-in of __pycache__, .pyc, reports/ |
| Likelihood | Certain |
| Detection | G6 quality gate |
| Mitigation | Create .gitignore with standard ServiceNow exclusions |
| Recovery Time | 1 commit |

---

## Risk Summary

| Severity | Count | Status |
|----------|-------|--------|
| P0 — Critical | 2 | R01, R02 — mitigated by plugin checks |
| P1 — High | 4 | R03-R06 — design-level mitigations, R06 needs active-run check |
| P2 — Medium | 4 | R07-R10 — accepted design limitations, roadmap items |
| P3 — Low | 2 | R11-R12 — fixable in this pipeline run |
| **Total** | **12** | |
