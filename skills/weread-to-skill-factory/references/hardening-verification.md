# Hardening Verification（加固验证）

Post-install 之后的独立验证流程，用于将 weakly verified 的 book-skill 推进到 HARD_VERIFIED。

## 适用场景

- forensic audit 发现 smoke test 无独立证据
- eval 只有静态分析，无真实 runtime
- final report 存在自证循环风险
- 需要从 weakly verified 升级到 hard verified

## 验证文件结构

在 book-skill 目录下创建 `verification/` 子目录，包含 4 个文件：

```
verification/
├── smoke-test-log.md          # 独立 smoke test 证据
├── runtime-eval-log.md        # runtime 可用性与执行状态
├── final-hardening-report.md  # 最终 hardening 判定
└── verification-index.md      # 索引
```

## 核心原则

1. **不伪造 runtime eval** — 没有真实运行就写 RUNTIME_UNAVAILABLE
2. **不把静态分析冒充 runtime** — STATIC_FILE_AND_CONTENT_CHECK ≠ runtime
3. **UNKNOWN 不得写 PASS** — 证据不足就诚实标注
4. **无自证循环** — 验证文件独立于被验证对象

## Runtime 调用方法

真实调用 book-skill 的方法：

```bash
hermes chat -q "<测试输入>" -s <skill-name> --cli
```

- `-s` 参数预加载指定 skill
- `--cli` 非交互模式
- 输出包含 skill 诊断命中信息

**注意**：`hermes skills inspect` 对 books 类别技能可能返回 "No skill named"，但这不代表 skill 不可用，只是 inspect 不支持该类别。

## 判定规则

| 条件 | 判定 |
|------|------|
| runtime eval 真实执行成功 + smoke test 独立完整 | HARD_VERIFIED |
| runtime unavailable + smoke test 独立完整 + 静态 eval 完整 + scope guard 完整 | HARD_VERIFIED_WITH_RUNTIME_UNAVAILABLE |
| smoke test 仍只在 final report 中 | STILL_WEAK |
| 关键文件缺失 / DRAFT 残留 / scope 缺失 / UNKNOWN 写成 PASS | FAIL |

## Forensic Audit 检查清单（15 项）

1. verification/smoke-test-log.md 存在
2. smoke-test-log.md 有独立文件结构检查
3. smoke-test-log.md 有内容检查
4. smoke-test-log.md 明确 Method: STATIC_FILE_AND_CONTENT_CHECK
5. runtime-eval-log.md 存在
6. runtime-eval-log.md 明确 Runtime Availability
7. 如果 runtime unavailable，是否明确写 RUNTIME_UNAVAILABLE
8. final-hardening-report.md 存在
9. final-hardening-report.md 判定是否合理
10. verification-index.md 存在
11. SKILL.md 引用了 verification 文件
12. 没有扩大 scope
13. 没有 DRAFT 残留
14. 没有 UNKNOWN 写成 PASS
15. 没有把 static eval 冒充 runtime eval

## Trigger 测试标准（5 项）

| 测试 | 输入 | 预期 |
|------|------|------|
| 触发 1 | 与 skill 核心场景匹配的请求 | skill 诊断命中，给出协议响应 |
| 触发 2 | 另一个核心场景 | skill 诊断命中 |
| 拒绝 1 | 明确超出 scope 的请求 | skill 明确拒绝，说明不适用 |
| 拒绝 2 | 政治/敏感判断请求 | 拒绝或系统拦截 |
| 不触发 | 与 skill 无关的请求 | 正常回答，不触发 skill |
