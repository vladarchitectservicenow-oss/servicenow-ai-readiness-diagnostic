# Validation Checklist: servicenow-ai-readiness-diagnostic

**Author:** Vladimir Kapustin | **License:** AGPL-3.0-only
**Scope:** x_snc_ard | **Target:** Production-ready

---

## Legend

| Code | Category | Description |
|------|----------|-------------|
| D | Documentation | Architecture, specs, marketing assets |
| T | Test Coverage | Test suite completeness and pass rate |
| R | Runtime | PDI execution and runtime behavior |
| L | Legal | License, copyright, compliance |
| S | Security | Vulnerability scanning, credential hygiene |
| G | Git | Source control, push verification |
| F | Functional | Feature correctness and business logic |

---

## Documentation (D)

- [ ] D01 — architecture_summary.md exists and has ≥50 lines
- [ ] D02 — architecture_summary.md includes component diagram (text or Mermaid)
- [ ] D03 — architecture_summary.md includes data flow diagram
- [ ] D04 — architecture_summary.md includes API contract with request/response schemas
- [ ] D05 — architecture_summary.md includes performance characteristics
- [ ] D06 — dependency_report.md exists and has ≥50 lines
- [ ] D07 — dependency_report.md lists all required plugins with IDs
- [ ] D08 — dependency_report.md lists all accessed core tables
- [ ] D09 — dependency_report.md defines required system roles
- [ ] D10 — dependency_report.md includes version compatibility matrix
- [ ] D11 — risk_report.md exists and has ≥10 risks with RXX identifiers
- [ ] D12 — risk_report.md classifies risks as P0/P1/P2/P3
- [ ] D13 — risk_report.md includes mitigation strategy for each risk
- [ ] D14 — execution_plan.md exists and has ≥6 phases
- [ ] D15 — execution_plan.md includes task/owner/status tables
- [ ] D16 — README.md has product tagline and elevator pitch
- [ ] D17 — README.md has Ideal Customer Profile section
- [ ] D18 — README.md has Competitive Landscape section
- [ ] D19 — README.md has Monetization section
- [ ] D20 — PRD.md exists and describes product requirements

## Test Coverage (T)

- [ ] T01 — test_suite_SOP.md exists with ≥10 scenarios (T01-TXX format)
- [ ] T02 — All P0 test scenarios (T01-T05) pass on PDI
- [ ] T03 — All P1 test scenarios (T06-T10) pass on PDI
- [ ] T04 — test_suite_SOP.md includes negative cases for each scenario
- [ ] T05 — regression_cases.md exists with ≥8 cases (R01-RXX format)
- [ ] T06 — All regression cases verified on latest PDI run
- [ ] T07 — edge_cases.md exists with ≥5 cases (E01-EX format)
- [ ] T08 — E01 (empty instance) verified — no NaN/null/division by zero
- [ ] T09 — validation_checklist.md exists (this document) with ≥60 items
- [ ] T10 — Weekly scheduled job execution produces gs.info log output
- [ ] T11 — REST API POST /diagnose returns 200 with valid schema
- [ ] T12 — REST API GET /dashboard returns aggregated readiness score
- [ ] T13 — REST API GET /simulate returns PASS/WARNING/FAIL breakdown
- [ ] T14 — REST API GET /diagnose/{id} returns status for specific run
- [ ] T15 — REST API returns 404 with JSON error for nonexistent resource
- [ ] T16 — REST API returns 500 with JSON error for server failures
- [ ] T17 — Node.js unit test mocks for GlideRecord work correctly
- [ ] T18 — Node.js unit test mocks for GlideAggregate work correctly
- [ ] T19 — All unit tests pass if tests/ directory exists

## Runtime (R)

- [ ] R01 — ProcessDebtScanner.fullScan() completes without uncaught exceptions
- [ ] R02 — CMDBHealthAnalyzer.analyze("cmdb_ci") returns valid object for populated instance
- [ ] R03 — CMDBHealthAnalyzer.analyze() handles non-existent CI class gracefully
- [ ] R04 — KBAnalyzer.analyze() returns valid object with total_articles=0 for empty KB
- [ ] R05 — AISimulator.simulateAll() produces valid PASS/WARNING/FAIL counts
- [ ] R06 — RoadmapGenerator.generate() returns items sorted by priority DESC
- [ ] R07 — RoadmapGenerator total_items matches sum of phase arrays
- [ ] R08 — Weekly scheduled job does not exceed transaction timeout
- [ ] R09 — No gs.info spam — log output is informational not verbose
- [ ] R10 — GlideRecord.setLimit() respected — no full-table scans beyond limits

## Legal (L)

- [ ] L01 — LICENSE file exists at repository root
- [ ] L02 — LICENSE contains full AGPL-3.0 text (not SPDX tag alone)
- [ ] L03 — LICENSE includes 'Copyright (C) 2026 Vladimir Kapustin'
- [ ] L04 — All src/ files have copyright header with 'Vladimir Kapustin' (full name, not abbreviated)
- [ ] L05 — All src/ files have SPDX identifier 'AGPL-3.0' on separate line
- [ ] L06 — README license badge matches LICENSE (G7 gate)
- [ ] L07 — No 'Vladimir K.' or 'V.K.' abbreviations anywhere
- [ ] L08 — 'Copyright (C)' uppercase C, not lowercase (c)

## Security (S)

- [ ] S01 — No hardcoded passwords or API tokens in source code (G5 gate)
- [ ] S02 — All credential references use process.env or system properties
- [ ] S03 — REST API endpoints require authentication (ACL enforced)
- [ ] S04 — REST API write endpoints require snc_ard.admin role
- [ ] S05 — REST API read endpoints require minimum snc_ard.viewer role
- [ ] S06 — No PII stored in diagnostic results or logs
- [ ] S07 — HTTPS enforced for all API communication
- [ ] S08 — issues_json does not expose sensitive field values

## Git (G)

- [ ] G01 — .gitignore exists (G6 gate)
- [ ] G02 — .gitignore excludes __pycache__/ and *.pyc
- [ ] G03 — .gitignore excludes reports/ directory
- [ ] G04 — .gitignore excludes *.log files
- [ ] G05 — .gitignore excludes node_modules/
- [ ] G06 — git remote origin points to vladarchitectservicenow-oss
- [ ] G07 — git push to main successful (G4 gate)
- [ ] G08 — GitHub API confirms branch exists after push
- [ ] G09 — DONE.marker exists in repo root
- [ ] G10 — Commit message follows conventional format

## Functional (F)

- [ ] F01 — CMDB completeness_pct correct: only CIs with ALL required fields counted as complete
- [ ] F02 — Orphan detection correct: only CIs with zero cmdb_rel_ci relationships counted
- [ ] F03 — Staleness threshold correct: sys_updated_on < 1 year ago
- [ ] F04 — KB outdated threshold correct: sys_updated_on < 2 years ago
- [ ] F05 — ProcessDebtScanner detects all 7 issue types from DETECTORS array
- [ ] F06 — Workflow health_score starts at 100 and decrements by issue weight
- [ ] F07 — Health score never goes below 0
- [ ] F08 — Roadmap priority formula correct: AI_impact*0.4 + blocking*3 + fixEase
- [ ] F09 — Quick wins correctly identified: fix_ease ≥10 AND issue_count ≥3
- [ ] F10 — Dashboard readiness_score formula: WH*0.4 + CMDB*0.35 + KB*0.25
- [ ] F11 — AISimulator correctly upgrades WARNING when CMDB completeness <50
- [ ] F12 — AISimulator correctly upgrades WARNING when orphan_pct >30
- [ ] F13 — AISimulator correctly upgrades WARNING when KB readiness <50
- [ ] F14 — Diagnostic run transitions from RUNNING → COMPLETED with timestamps
- [ ] F15 — No duplicate README sections (G8 gate — section heading uniqueness)

---

## Summary

| Category | Items | Required Minimum |
|----------|-------|-----------------|
| Documentation (D) | 20 | 20 |
| Test Coverage (T) | 19 | 15 |
| Runtime (R) | 10 | 9 |
| Legal (L) | 8 | 8 |
| Security (S) | 8 | 8 |
| Git (G) | 10 | 9 |
| Functional (F) | 15 | 13 |
| **Total** | **90** | **82** |

---

## Pass Criteria

- All D items must pass (documentation is structural)
- All L items must pass (license is non-negotiable)
- All S items must pass (security is mandatory)
- At least 82/90 total items must pass
- Any failure in D/L/S blocks deployment
