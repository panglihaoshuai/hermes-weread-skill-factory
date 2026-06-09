# Post-install Smoke Test Log

---

## Immutable Evidence

* Date: [日期]
* Book Skill: [skill 名称]
* Path: [路径]
* Install Status: [状态]
* Scope: [覆盖范围]
* Files Modified During Test: [修改文件]
* Method: STATIC_FILE_AND_CONTENT_CHECK

---

## File Structure Smoke Test

| File | Exists | Evidence | Result |
|------|--------|----------|--------|
| SKILL.md | [✓/✗] | [证据] | [PASS/FAIL] |
| references/book-map.md | [✓/✗] | [证据] | [PASS/FAIL] |
| references/distilled-framework.md | [✓/✗] | [证据] | [PASS/FAIL] |
| references/decision-rules.md | [✓/✗] | [证据] | [PASS/FAIL] |
| references/anti-patterns.md | [✓/✗] | [证据] | [PASS/FAIL] |
| references/diagnostic-questions.md | [✓/✗] | [证据] | [PASS/FAIL] |
| references/action-protocols.md | [✓/✗] | [证据] | [PASS/FAIL] |
| references/user-highlights.md | [✓/✗] | [证据] | [PASS/FAIL] |
| references/user-notes.md | [✓/✗] | [证据] | [PASS/FAIL] |
| references/evidence-ledger.md | [✓/✗] | [证据] | [PASS/FAIL] |
| references/limits.md | [✓/✗] | [证据] | [PASS/FAIL] |
| evals/trigger-tests.md | [✓/✗] | [证据] | [PASS/FAIL] |
| evals/application-tests.md | [✓/✗] | [证据] | [PASS/FAIL] |
| evals/adversarial-tests.md | [✓/✗] | [证据] | [PASS/FAIL] |

---

## Content Smoke Test

| Check | File | Evidence Snippet | Observed | Result |
|-------|------|------------------|----------|--------|
| SKILL.md 无 DRAFT | SKILL.md | [证据片段] | [观察] | [PASS/FAIL] |
| SKILL.md 有正式状态 | SKILL.md | [证据片段] | [观察] | [PASS/FAIL] |
| SKILL.md 明确 scope-limited | SKILL.md | [证据片段] | [观察] | [PASS/FAIL] |
| limits.md 存在 | references/limits.md | [证据片段] | [观察] | [PASS/FAIL] |
| evidence-ledger.md 存在 | references/evidence-ledger.md | [证据片段] | [观察] | [PASS/FAIL] |
| user-highlights.md 有划线 | references/user-highlights.md | [证据片段] | [观察] | [PASS/FAIL] |
| user-notes.md 有批注 | references/user-notes.md | [证据片段] | [观察] | [PASS/FAIL] |
| 第 7 章以后 UNKNOWN | references/evidence-ledger.md | [证据片段] | [观察] | [PASS/FAIL] |
| 不用于严肃历史定论 | SKILL.md | [证据片段] | [观察] | [PASS/FAIL] |
| 不用于政治立场判断 | SKILL.md | [证据片段] | [观察] | [PASS/FAIL] |
| 不用于学术引用 | SKILL.md | [证据片段] | [观察] | [PASS/FAIL] |
| 证据不足输出 UNKNOWN | references/limits.md | [证据片段] | [观察] | [PASS/FAIL] |

---

## Trigger Static Smoke Test

### Test A: [测试名称]

* Test Name: [测试名称]
* Input: [输入]
* Expected: [预期]
* Observed: [观察]
* Method: STATIC_RULE_CHECK
* Evidence: [证据]
* Result: [PASS/FAIL]

### Test B: [测试名称]

* Test Name: [测试名称]
* Input: [输入]
* Expected: [预期]
* Observed: [观察]
* Method: STATIC_RULE_CHECK
* Evidence: [证据]
* Result: [PASS/FAIL]

---

## Smoke Test Summary

* PASS: [数量]
* FAIL: [数量]
* UNKNOWN: [数量]
* WEAK_PASS: [数量]
* Blocking Issues: [问题]
* Judgment: [STATIC_SMOKE_PASS/STATIC_SMOKE_WEAK/STATIC_SMOKE_FAIL]
