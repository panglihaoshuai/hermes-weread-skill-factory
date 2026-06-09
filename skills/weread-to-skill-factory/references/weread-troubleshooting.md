# WeRead API 故障诊断

## 错误模式：公开接口正常，用户接口返回「用户不存在」

**症状**：
- `/store/search`（无需登录）→ ✅ 正常
- `/user/notebooks`、`/shelf/sync`、`/book/bookmarklist`（需要用户身份）→ `errcode: -2010, errmsg: 用户不存在`

**诊断**：API key 本身没过期，但**用户身份绑定失效**。key 还能搜书，但读不了你的书架和笔记。

**原因**：微信读书的 API key 绑定可能因账号状态变化、长时间未使用、或服务端策略调整而失效。key 格式仍为 `wrk-xxx`（28 字符），服务端接受它，但不再关联用户身份。

**修复**：重新扫码授权，刷新绑定。

### Re-auth 流程（使用 agent-weread-skill）

```bash
# 1. 确保 qrcode 包已安装
pip3 install qrcode

# 2. 运行扫码登录脚本
cd ~/.hermes/skills/agent-weread-skill
python3 scripts/weread_auth.py --qr
```

脚本会输出：
- QR 码图片路径（用 `open` 命令打开）
- 直接 URL（微信打开即可确认）
- ASCII QR 码（终端渲染，可能因宽度问题无法扫码）

**三种扫码方式**：
1. `open <png路径>` — 用系统看图器打开 QR 图片，微信扫码
2. 复制 URL `https://weread.qq.com/web/confirm?pf=2&uid=...` — 微信内打开直接确认
3. 终端 ASCII QR — 需要终端足够宽

**注意**：headless browser 打开 `https://weread.qq.com/r/weread-skills` 可能被反爬拦截，QR 码不加载。优先用 `weread_auth.py --qr` 脚本。

### 验证修复

```bash
# 登录态接口应返回正常数据
curl -s -X POST "https://i.weread.qq.com/api/agent/gateway" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $WEREAD_API_KEY" \
  -d '{"api_name": "/shelf/sync", "skill_version": "1.0.3"}'
```

## 错误模式：所有接口返回错误

**症状**：包括 `/store/search` 在内所有接口都报错（非 -2010）。

**诊断**：API key 本身可能已失效或格式错误。

**修复**：访问 https://weread.qq.com/r/weread-skills 重新获取 key。
