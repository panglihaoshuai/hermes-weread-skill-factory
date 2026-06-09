# Evidence Ledger 模板（证据账本）

每个 Level 2 或 Level 3 的 book-skill 都必须生成 `references/evidence-ledger.md`。

## 模板

```markdown
# Evidence Ledger — 《书名》

## 基本信息
- 书名：
- 作者：
- 来源：WeRead / Paper / Kindle / PDF / Other
- 微信读书 book_id：
- 阅读来源：

## Gate 信息
- Gate 路径：WeRead Evidence / Private Reading Viva
- Gate 分数：X/20 或 X/30
- Gate 判定：Level 0/1/2/3

## 个人证据
- 用户划线数量：
- 用户批注数量：
- 章节覆盖情况：
- 用户最认可的观点：
- 用户不同意或保留的观点：
- 用户提供的真实应用场景：

## 外部辅助材料
- 热门划线数量：
- 公开点评参考：

## 蒸馏决策
- 为什么这本书值得生成 skill：
- 为什么这本书可能不适合生成 skill：
- 过度套用风险：
- 与已有 skill 重复风险：
```

## 证据等级（用于标记蒸馏结论来源）

| 等级 | 来源 | 说明 |
|------|------|------|
| A | 个人划线 + 批注 | 最高价值 |
| B | 个人划线（无批注） | 高价值 |
| C | 热门划线（>1000人） | 仅辅助 |
| D | 热门划线（<1000人） | 仅辅助 |
| E | 书籍简介/目录 | 仅元数据 |
| F | 推导/综合 | 必须标注 |

核心模型必须有 A 或 B 级证据支撑。C/D/E 级只能辅助。
