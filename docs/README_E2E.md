# 电商项目 E2E 全功能验证（ralph-loop 模式）· 使用说明

用 Claude CLI 的 ralph-loop 自动把全部功能走查一遍，产出给后端的缺陷报告。

## 文件构成
| 文件 | 作用 |
|---|---|
| `E2E_CONFIG.md` | 地址 / 账号 / 环境约束 / 可测性规则（事实来源） |
| `E2E_CHECKLIST.md` | 58 个用例的进度真相表，ralph-loop 每轮回填 |
| `E2E_FINDINGS.md` | 缺陷报告（给后端工程师），逐条累加 |
| `E2E_RALPH_PROMPT.md` | 喂给 /ralph-loop 的任务提示词 |

## 前置条件（必须先做）
1. **本工程目录下有 Playwright MCP**（`.playwright-mcp/` 已存在，说明已配置）。务必在**本工程目录**启动 claude，否则没有浏览器工具。
2. **H5 用例需要先启动 H5 dev server**：HBuilderX → 打开本工程 → 运行 → 运行到浏览器（Chrome）。记下端口，更新 `E2E_CONFIG.md` 第 1 表。
   - 不启动也能跑：所有后台(B-XX)用例可直接做，H5 用例会被标 `⛔ BLOCKED`，等启动后重跑。
3. 确认这是**测试/预发环境，允许写操作**（已确认）。

## 启动方式

在工程目录 `shopxo-uniapp-master` 下打开 claude，执行（提示词取自 `E2E_RALPH_PROMPT.md` 全文）：

```
/ralph-loop 按 docs/E2E_RALPH_PROMPT.md 的全部要求执行本轮 E2E 验证：读 docs 下 CONFIG/CHECKLIST/FINDINGS 三份文件，选下一个 TODO 用例执行，用 Playwright MCP 操作，回填 CHECKLIST 与 FINDINGS 并保存。全部用例非 TODO 时输出 <promise>E2E验证完成</promise>。 --completion-promise "E2E验证完成" --max-iterations 80
```

> `--max-iterations 80`：58 个用例、每轮 1~3 个，约需 30~60 轮，留余量。可按需调大。
> 中途想停：`/cancel-ralph`。

## 运行中你能看到什么
- 每轮：选一个用例 → 操作浏览器 → 把结果写进 CHECKLIST、缺陷写进 FINDINGS → 保存 → 自动进入下一轮。
- 进度随时可看：打开 `E2E_CHECKLIST.md`（状态列）和 `E2E_FINDINGS.md`（缺陷数）。

## 结束后产物
- `E2E_CHECKLIST.md`：每个功能点的 PASS/FAIL/BLOCKED/MANUAL 终态。
- `E2E_FINDINGS.md`：完整缺陷清单 + 「功能是否实现」逐项结论 + 给后端的优先修复清单。
- `.playwright-mcp/`：失败用例的截图与日志证据。

## 覆盖范围与硬约束（重要）
- 🟢 **运营后台(PC)**：Playwright 全覆盖。
- 🟢 **用户端 H5**：Playwright 覆盖（H5 镜像小程序用户端业务功能）。
- 🔴 **小程序专有能力**（微信一键登录、微信订阅消息/服务通知、微信客服）：Playwright **测不了**，标 `🙋 MANUAL`，需人工在微信内核查；能用后台「推送记录」等侧面佐证的会佐证。
- 🔴 **供货商端（独立小程序）**：Playwright 不可直测，主要通过**运营后台的订单/供货商模块侧面验证**数据正确性，发货等动作需人工在小程序内核查。
  - 若需供货商端也自动化，要单独引入微信官方 `miniprogram-automator` + 微信开发者工具 + 小程序源码（当前未做）。
