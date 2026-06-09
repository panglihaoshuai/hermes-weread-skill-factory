# Post-install Smoke Test Log

**⚠️ 本文件是虚构示例，不包含真实书籍内容。**

---

## Immutable Evidence

* Date: 2026-06-09
* Book Skill: sample-scope-limited-book-skill
* Path: examples/sample-scope-limited-book-skill/
* Install Status: INSTALLED
* Scope: WeRead 前 2 章 + Guided Viva（前 6 章理解）
* Files Modified During Test: 无（仅读取）
* Method: STATIC_FILE_AND_CONTENT_CHECK

---

## File Structure Smoke Test

| File | Exists | Evidence | Result |
|------|--------|----------|--------|
| SKILL.md | ✓ | 文件存在 | PASS |
| references/book-map.md | ✓ | 文件存在 | PASS |
| references/distilled-framework.md | ✓ | 文件存在 | PASS |
| references/decision-rules.md | ✓ | 文件存在 | PASS |
| references/anti-patterns.md | ✓ | 文件存在 | PASS |
| references/diagnostic-questions.md | ✓ | 文件存在 | PASS |
| references/action-protocols.md | ✓ | 文件存在 | PASS |
| references/user-highlights.md | ✓ | 文件存在 | PASS |
| references/user-notes.md | ✓ | 文件存在 | PASS |
| references/evidence-ledger.md | ✓ | 文件存在 | PASS |
| references/limits.md | ✓ | 文件存在 | PASS |
| evals/trigger-tests.md | ✓ | 文件存在 | PASS |
| evals/application-tests.md | ✓ | 文件存在 | PASS |
| evals/adversarial-tests.md | ✓ | 文件存在 | PASS |

---

## Content Smoke Test

| Check | File | Evidence Snippet | Observed | Result |
|-------|------|------------------|----------|--------|
| SKILL.md 无 DRAFT | SKILL.md | "status: INSTALLED" | 无 DRAFT 关键字 | PASS |
| SKILL.md 有正式状态 | SKILL.md | "status: INSTALLED" | 正式状态 | PASS |
| SKILL.md 明确 scope-limited | SKILL.md | "scope-limited: true" | 明确声明 | PASS |
| limits.md 存在 | references/limits.md | 文件存在 | ✓ | PASS |
| evidence-ledger.md 存在 | references/evidence-ledger.md | 文件存在 | ✓ | PASS |
| user-highlights.md 有 2 条划线 | references/user-highlights.md | "总划线数: 2 条（虚构）" | 2 条划线 | PASS |
| user-notes.md 有 4 条批注 | references/user-notes.md | "总批注数: 4 条（虚构）" | 4 条批注 | PASS |
| 第 7 章以后 UNKNOWN | references/evidence-ledger.md | "第7章以后（chapterUid=7+） | UNKNOWN" | 明确标注 | PASS |
| 不用于严肃学术研究 | SKILL.md | "不用于严肃学术研究" | 明确声明 | PASS |
| 不用于正式引用 | SKILL.md | "不用于正式引用" | 明确声明 | PASS |
| 不用于完整主题研究 | SKILL.md | "不用于完整主题研究" | 明确声明 | PASS |
| 证据不足输出 UNKNOWN | references/limits.md | "证据不足输出 UNKNOWN" | 明确声明 | PASS |

---

## Smoke Test Summary

* PASS: 26
* FAIL: 0
* UNKNOWN: 0
* WEAK_PASS: 0
* Blocking Issues: 无
* Judgment: STATIC_SMOKE_PASS
