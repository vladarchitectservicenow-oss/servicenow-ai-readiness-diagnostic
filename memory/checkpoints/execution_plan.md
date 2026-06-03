# servicenow-ai-readiness-diagnostic — Execution Plan

**Product:** AI Readiness Diagnostic (x_snc_ard)
**Author:** Vladimir Kapustin
**Date:** 2026-06-03

---

## Phase Summary

| Phase | Name | Status | Owner | Est. Effort |
|-------|------|--------|-------|-------------|
| 1 | Architecture & Documentation | ✅ Complete | Pipeline | 1h |
| 2 | Validation Suite | 🔄 In Progress | Pipeline | 45min |
| 3 | Quality Gates (G0-G8) | ⬜ Pending | Pipeline | 30min |
| 4 | README Remediation | ⬜ Pending | Pipeline | 20min |
| 5 | .gitignore & Headers | ⬜ Pending | Pipeline | 10min |
| 6 | Git Commit & Push | ⬜ Pending | Pipeline | 10min |
| 7 | DONE.marker & Progress Update | ⬜ Pending | Pipeline | 5min |

---

## Phase 1: Architecture & Documentation

| Task | Description | Status |
|------|-------------|--------|
| T1.1 | Write architecture_summary.md (≥50 lines, component diagram, data flow, API contract, performance characteristics) | ✅ Complete |
| T1.2 | Write dependency_report.md (≥50 lines, plugin IDs, table list, role list, version matrix) | ✅ Complete |
| T1.3 | Write risk_report.md (≥10 risks with RXX IDs, P0-P3 severity, mitigation strategies) | ✅ Complete |
| T1.4 | Write execution_plan.md (this document, ≥6 phases with task/status tables) | 🔄 In Progress |

---

## Phase 2: Validation Suite

| Task | Description | Status |
|------|-------------|--------|
| T2.1 | Write test_suite_SOP.md (≥10 scenarios with T01-TXX format, priority levels, execution instructions) | ⬜ Pending |
| T2.2 | Write regression_cases.md (≥8 cases with R01-RXX format) | ⬜ Pending |
| T2.3 | Write edge_cases.md (≥5 cases with E01-EX format) | ⬜ Pending |
| T2.4 | Write validation_checklist.md (≥60 items with [ ] checkboxes, D/T/R/L/S/G/F codes) | ⬜ Pending |

---

## Phase 3: Quality Gates (G0-G8)

| Gate | Description | Status |
|------|-------------|--------|
| G0 | test_suite_SOP.md has ≥10 scenarios | ⬜ Pending |
| G1 | All tests execution_history/*.log confirms PASS | ⬜ Pending |
| G2 | README.md ≥2000 words with Mermaid + ROI | ✅ Pass (2195 words) |
| G3 | Every src/ file has AGPL-3.0 copyright header | ⬜ Verify |
| G4 | Git push verified via API | ⬜ Pending |
| G5 | No hardcoded credentials in source code | ⬜ Verify |
| G6 | .gitignore exists with standard exclusions | ⬜ Pending |
| G7 | README license header matches LICENSE file | ⬜ Verify (MIT badge vs AGPL-3.0) |
| G8 | No duplicate README sections | ⬜ Verify (likely issue — mass template appended 4x) |

---

## Phase 4: README Remediation

| Task | Description | Status |
|------|-------------|--------|
| T4.1 | Fix license badge (MIT → AGPLv3) | ⬜ Pending |
| T4.2 | Deduplicate sections (G8 — 4 copies of Overview/Architecture/Features/etc.) | ⬜ Pending |
| T4.3 | Verify final word count ≥2000 | ⬜ Pending |
| T4.4 | Ensure Mermaid, ROI, Troubleshooting sections present | ✅ Present |

---

## Phase 5: .gitignore & Copyright Headers

| Task | Description | Status |
|------|-------------|--------|
| T5.1 | Create .gitignore (__pycache__/, *.pyc, reports/, *.log, node_modules/) | ⬜ Pending |
| T5.2 | Verify all src/ files have copyright header with 2026 date | ⬜ Verify |
| T5.3 | Patch any missing headers | ⬜ If needed |

---

## Phase 6: Git Commit & Push

| Task | Description | Status |
|------|-------------|--------|
| T6.1 | `git add -A` stage all changes | ⬜ Pending |
| T6.2 | Commit with conventional message | ⬜ Pending |
| T6.3 | `git push origin main` with token auth | ⬜ Pending |
| T6.4 | Verify push via GitHub API (`curl /branches`) | ⬜ Pending |

---

## Phase 7: DONE.marker & Pipeline Update

| Task | Description | Status |
|------|-------------|--------|
| T7.1 | Write DONE.marker to repo root | ⬜ Pending |
| T7.2 | Update /tmp/pipeline_progress.json (move from pending to done) | ⬜ Pending |
| T7.3 | Verify next product in queue | ⬜ Pending |

---

## Completion Criteria

- [ ] All 8 Phase 1+2 documents are non-skeletal (verified line counts / scenario counts)
- [ ] README ≥2000 words, deduplicated, license badge matches LICENSE
- [ ] All G0-G8 gates pass or have documented exceptions
- [ ] .gitignore present with minimum exclusions
- [ ] Git push confirmed via API
- [ ] DONE.marker exists in repo root
- [ ] Pipeline progress file updated
