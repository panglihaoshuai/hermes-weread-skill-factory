# Runtime Eval Log

---

## Immutable Evidence

* Book Skill: [skill 名称]
* Path: [路径]
* Runtime Method: [方法]
* Runtime Availability: [RUNTIME_AVAILABLE/RUNTIME_UNAVAILABLE]
* External API Called: [API]
* Files Modified: [修改文件]

---

## Runtime Capability Check

| Candidate Method | Evidence | Safe to Run | Result | Notes |
|------------------|----------|-------------|--------|-------|
| [方法 1] | [证据] | [✓/✗] | [RUNTIME_AVAILABLE/RUNTIME_UNAVAILABLE] | [备注] |
| [方法 2] | [证据] | [✓/✗] | [RUNTIME_AVAILABLE/RUNTIME_UNAVAILABLE] | [备注] |

---

## Runtime Eval Execution

如果 RUNTIME_AVAILABLE：

### Test 1: [测试名称]

* Input: [输入]
* Expected: [预期]
* Actual Output: [实际输出]
* Runtime Command: [命令]
* Evidence: [证据]
* Result: [PASS/FAIL]

### Test 2: [测试名称]

* Input: [输入]
* Expected: [预期]
* Actual Output: [实际输出]
* Runtime Command: [命令]
* Evidence: [证据]
* Result: [PASS/FAIL]

---

如果 RUNTIME_UNAVAILABLE：

Runtime eval was not executed because no reliable local skill invocation method was found.

* Runtime Status: RUNTIME_UNAVAILABLE
* Static Eval Status: STATIC_EVAL_ONLY
* Blocking: NO, because static eval and smoke test provide file-level hard verification, but runtime behavior remains unverified.

---

## Runtime Eval Summary

* Total Tests: [数量]
* PASS: [数量]
* FAIL: [数量]
* UNKNOWN: [数量]
* Runtime Status: [状态]
* Static Eval Status: [状态]
* Blocking: [YES/NO]
