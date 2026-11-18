# QA Report: Final Sprint, Applied Data Science

| | |
|---|---|
| **Section** | Shared VM (team ___) / Local VM |
| **Test author(s)** | ______________________ (Shared VM: QA) |
| **Tester** | ______________________ (Local VM: you are author and tester) |
| **Commit tested** | `git rev-parse --short HEAD` = ________ |
| **Dates** | Cases written: ________ Cases run: ________ |

## How to use this report

1. **Write the test cases first,** in Week 11, before the work they test exists. A test written after the fact tends to test what was built instead of what was asked for.
2. **Write the expected result before you run anything.** If the actual result surprises you, that is a finding, not a typo to fix.
3. **Paste real output or point to an evidence file.** "Works" is not a result. A command and its output is.
4. **Cover every success criterion with at least one case,** and include at least one negative test (something that must fail or be blocked) and the rebuild proof of life.
5. **Re-run every case in Week 13** on the rebuilt VM and update the status. A case that passed only on the old VM is a rebuild gap.

If your team replaced a criterion through [MAKE-IT-YOURS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/MAKE-IT-YOURS.md), test the replacement instead, and link the approved design brief.

---

## Test Cases

Copy the block for each new case. Six slots are a starting point, not a limit.

<details>
<summary>Example of a filled-in case (delete when you start)</summary>

**TC-00: Pipeline regenerates outputs from scratch**

| Field | Value |
|---|---|
| Criterion | 2 |
| Steps | Delete `capstone/data-science/output/`, run the one pipeline command, list the folder |
| Expected (written before running) | All three CSVs and three PNGs come back; row counts match `data_quality.csv` |
| Actual | See `capstone/data-science/evidence/<file>`; summary: ... |
| Status | Pass |
| Run on | Original VM, YYYY-MM-DD; rebuilt VM, YYYY-MM-DD |

</details>

### TC-01: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-02: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-03: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-04: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-05: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-06: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

---

## Success Criteria Verification

Evidence paths are relative to `capstone/data-science/` and come from [EVIDENCE-CHECKLIST](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/EVIDENCE-CHECKLIST.md). Change a path if your file is named differently.

| Criterion | Evidence | Test case(s) | Met? |
|---|---|---|---|
| 1. 10,000+ rows over 90+ days, a category for every row and a resolution time for every closed row | `output/data_quality.csv`, `DATA.md` | | [ ] Yes [ ] No |
| 2. Three analysis CSVs from one command (weekly volume, resolution median and p90 per category, category ranking) | `output/*.csv` | | [ ] Yes [ ] No |
| 3. Three PNG charts with title, labeled axes, and units | `output/*.png` | | [ ] Yes [ ] No |
| 4. One statistical check with a number and a plain-English sentence | `report.md` or `output/stats.txt` | | [ ] Yes [ ] No |
| 5. Staffing recommendation in `report.md` (two pages or less) with stated assumptions | `report.md` | | [ ] Yes [ ] No |

---

## Rebuild Verification (Week 13)

| Check | Result |
|---|---|
| Rebuild date and VM (new VirtualBox VM, or Shared VM queue slot) | |
| Started from commit | |
| Proof of life, Shared VM: One command regenerates every CSV and PNG; row counts match `data_quality.csv` | [ ] Pass [ ] Fail |
| Proof of life, Local VM: Same as the Shared VM | [ ] Pass [ ] Fail |
| Every app change came back (schema, routes, services, images) | [ ] Yes [ ] No |
| Gaps logged in `capstone/data-science/REBUILD-GAPS.md` and fixed by pull request | ___ gaps found, ___ fixed |
| All test cases above re-run on the rebuilt VM | [ ] Yes [ ] No |

---

## Deliverables Verification

- [ ] Implementation committed under `capstone/data-science/` (merged by pull request)
- [ ] `capstone/data-science/RUNBOOK.md`: operate, check, and recover
- [ ] `capstone/data-science/TOOL-VERSIONS.txt` (and `requirements.txt` for Python packages)
- [ ] `capstone/data-science/ARCHITECTURE.md` with a diagram and a "What we changed and why" section
- [ ] "How to run this" section in the top-level README
- [ ] `capstone/data-science/REBUILD-GAPS.md` from the Week 13 rebuild
- [ ] This QA report, with every case run and signed
- [ ] Shared VM: `docs/final-sprint-retrospective.md`. Local VM: the Individual Reflection section
- [ ] `capstone/data-science/DESIGN-BRIEF.md`, only if a criterion was replaced, with the approval noted
- [ ] Discussion answers in the Google Doc
- [ ] Final presentation and Week 14 rebuild presentation prepared (live on the Shared VM, recorded on the Local VM)
- [ ] Track-specific: `data_quality.csv`, three analysis CSVs, and the PNG charts under `capstone/data-science/output/`
- [ ] Track-specific: `report.md` written for Dana and Priya
- [ ] Track-specific: `DATA.md` describing how categories and resolution times were obtained, and their limits
- [ ] Track-specific: The script(s) that regenerate every output from one command

---

## Rework Required

| # | Failed case or criterion | What is wrong | Fix and owner | Re-test date | Re-test result |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |

---

## Sign-Off

**Overall status:** [ ] All criteria met [ ] Met with documented exceptions [ ] Rework required

**Exceptions** (criteria not met, and why):

```

```

| Role | Name | Date |
|---|---|---|
| Test author | | |
| Tester (Local VM: same person) | | |

**Notes:** observations about technical quality and, on the Shared VM, team coordination.
