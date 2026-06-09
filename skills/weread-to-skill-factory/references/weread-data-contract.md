# 微信读书 API 数据契约

## 统一入口

```
POST https://i.weread.qq.com/api/agent/gateway
Header: Authorization: Bearer $WEREAD_API_KEY
每次请求必须带 "skill_version": "1.0.3"
```

⚠️ **参数名是 `api_name`**，不是 `action`。写错会返回 errcode -2010「用户不存在」（容易误判为 key 失效）。

**参数名**：接口名用 `"api_name"`（不是 `"action"`）。业务参数平铺在 body 顶层，不要包在 `params` 里。

```json
{"api_name": "/store/search", "keyword": "三体", "count": 10, "skill_version": "1.0.3"}
```

## 接口清单

| 接口 | 用途 | 需登录 | smoke test | Gate 权重 |
|------|------|--------|-----------|----------|
| `/store/search` | 搜索书籍 | 否 | ✅ 返回6条 | 辅助 |
| `/user/notebooks` | 笔记本概览（有笔记的书+数量） | 是 | ✅ 返回5本 | **候选发现入口** |
| `/book/info` | 书籍详情 | 否 | ✅ 返回title/author | 辅助 |
| `/book/chapterinfo` | 章节目录 | 否 | ✅ 返回109章 | 辅助 |
| `/shelf/sync` | 书架列表 | 是 | ✅ 返回books数组 | 辅助 |
| `/book/bookmarklist` | 个人划线 | 是 | ✅ 返回chapters+updated | 核心 |
| `/review/list/mine` | 个人批注/想法 | 是 | ✅ 返回reviews+totalCount | 核心 |
| `/book/bestbookmarks` | 热门划线Top20 | 是 | ✅ 返回20条items | 仅辅助 |
| `/book/readreviews` | 划线下评论 | 是 | ✅ 返回reviews数据 | 仅辅助 |
| `/book/getprogress` | 阅读进度 | 是 | ⚠️ 未测试 | 核心（UNKNOWN） |
| `/user/notebooks` | 笔记本概览 | 是 | ⚠️ 未测试 | 辅助（UNKNOWN） |
| `/book/recommend` | 推荐 | 是 | ⚠️ 未测试 | 仅辅助 |
| `/book/similar` | 相似书籍 | 是 | ⚠️ 未测试 | 仅辅助 |
| `/book/underlines` | 划线热度统计 | 是 | ⚠️ 未测试 | 仅辅助 |

### UNKNOWN API 处理政策

标记为 ⚠️ 未测试 的接口：
- 不得作为硬 Gate 条件
- API 不可用或未测试时记 UNKNOWN，不得记 0
- 阅读完成度评分依赖 `/book/getprogress`，如果该 API 不可用，此维度记 UNKNOWN（不计入总分），并通知用户
- UNKNOWN 不等于 FAIL

## 不可用接口

| 需求 | 状态 |
|------|------|
| 正文/章节正文 | ❌ 无专用 API |
| 书签内容导出 | ❌ 只有数量统计 |

## 关键字段

### /book/bookmarklist 回包

```json
{
  "updated": [
    {
      "bookmarkId": "...",
      "bookId": "...",
      "chapterUid": 1,
      "markText": "划线原文",
      "createTime": 1748563200,
      "range": "900-2004"
    }
  ],
  "chapters": [
    {"chapterUid": 1, "chapterIdx": 0, "title": "章节标题"}
  ]
}
```

### /review/list/mine 回包

⚠️ **参数注意**：请求参数名是 `bookid`（小写 i），不是 `bookId`。写错会返回 errcode -2003。

```json
{
  "reviews": [
    {
      "review": {
        "reviewId": "...",
        "content": "批注内容",
        "createTime": 1748563200,
        "chapterName": "章节名",
        "star": 5
      }
    }
  ],
  "totalCount": 10,
  "hasMore": 1,
  "synckey": 12345
}
```

### /book/bestbookmarks 回包

```json
{
  "totalCount": 20,
  "items": [
    {
      "bookmarkId": "...",
      "chapterUid": 1,
      "range": "900-2004",
      "markText": "热门划线原文",
      "totalCount": 58500
    }
  ],
  "chapters": [...]
}
```

## API 故障诊断

| errcode | errmsg | 含义 | 处理 |
|---------|--------|------|------|
| -2010 | 用户不存在 | API key 用户绑定失效 | 见下方「-2010 细分诊断」 |
| -2012 | 无权限 | 需要登录但未登录 | 确认 key 有登录态 scope |
| -2014 | 参数错误 | api_name 或字段名拼写错误 | 检查 api_name 路径和参数名 |

**-2010 细分诊断**：`-2010` 不一定是 key 过期。有两种情况：

| 现象 | 诊断 | 修复 |
|------|------|------|
| 所有接口（含 `/store/search`）都报 -2010 | key 本身已失效 | 重新获取 key（访问 weread.qq.com/r/weread-skills） |
| `/store/search` 正常，但 `/shelf/sync`、`/book/bookmarklist` 等用户接口报 -2010 | key 有效但**用户身份绑定失效** | 重新扫码授权（用 `agent-weread-skill` 的 `weread_auth.py --qr`） |

验证方法：先调 `/store/search`（无需登录），如果正常则 key 没问题，需要 re-auth 刷新绑定。
详见 `references/weread-troubleshooting.md`。

## 分页规则

- `/user/notebooks`：游标分页，`lastSort` 取上一页最后一项的 `sort`
- `/review/list/mine`：游标分页，`synckey` 取上一页返回的 `synckey`
- 循环直到 `hasMore=0`
- **所有业务参数平铺在 body 顶层**，不要包在 `params` 里

## 深度链接

```
打开书籍: weread://reading?bId={bookId}
跳转章节: weread://reading?bId={bookId}&chapterUid={chapterUid}
跳转划线: weread://bestbookmark?bookId={bookId}&chapterUid={chapterUid}&rangeStart={start}&rangeEnd={end}
```
