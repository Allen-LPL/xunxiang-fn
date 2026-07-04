# AGENT_NOTES.md — 走查备忘

## 项目事实（首轮确认）

- **无 npm 构建**：本项目用 HBuilderX 开发/打包，没有 `npm run build`。验证手段以**静态 grep 断言**为主（禁止色计数=0、token 引用正确），必要时肉眼 diff 复核。根目录 package.json 只属于视频播放组件。
- **git 仓库刚初始化、无任何 commit**：第 1 轮已做 baseline 提交（全量），此后每轮 diff 才有意义。
- **排除目录**：`uni_modules/`（第三方插件，禁止手改）、`unpackage/`（编译产物）、`*.old.*`（遗留死码，见下）。
- **`.old.*` 死码清单**：index、user、user-integral、user-order、paytips、user-address、goods-detail、plugins/points/scan 各有 .old.vue/.old.css 备份，grep 违规时须排除，不修改不删除（P2-3 统一确认引用情况）。
- `pages/diy/init_data.js` 被 gitignore（本地 DIY 种子数据）。

## 关键映射（页面 ↔ 样式文件）

- 页面样式多为独立 `<页面名>.css` 与 .vue 同目录（非 scoped style），改样式优先改 .css 文件。
- 全局样式：`common/css/theme.css`（8 主题类，theme-red 为默认）、`page.css`、`business.css`、`lib.css`。
- `common/js/common/common.js` 内有 DIY 样式默认值（js 字符串形式的色值），属"设计 token"性质，可按规范改色值，**不动逻辑**。
- `components/cart/cart.vue:398` 的 `#E64340` 是 js 对象里的样式值（backgroundColor），改色值字符串不算改逻辑。
- tabbar 在 pages.json 中全部 transparent + text 为空 → 说明是**自绘 tabbar**（App.vue 有 app_tabbar_pages()），P1-1 时先找到自绘组件再动手。

## 扫描基线数据（2026-07-04 第 1 轮）

- 微信红系（#E64340/#dd514c）：5 处（2 处在 .old 死码中，实际 3 处）
- 金色渐变：7 处活代码（goods-search/faq/paytips×2/exchange-success×2/user）
- theme.css 渐变：8 处
- 冷灰 #ddd：26 行，分布 12 文件（含插件）
- 渐变总量（含插件/diy）：112 处 —— 只清"金渐变+theme.css+核心链路"，插件排 P2
- box-shadow：84 处（核心链路优先，浮层允许 ≤0.06）
- 米色底 #F5E4C6/#FAF9F6 系：0 处 ✅
- 宋体 font-family：0 处 → 排印体系完全未建，P1-2 做基建
- 大圆角大量存在（20rpx/32rpx/50rpx/胶囊 1000px 等），按"核心页面逐页收敛，存量胶囊可平涂保形"处理

## 验证命令（每轮复用）

```bash
# 禁止色断言（应为 0，排除 .old 与 uni_modules）
grep -rniE '#e64340|#e54d42|#dd514c' pages components common App.vue --include='*' | grep -v '\.old\.' | wc -l
grep -rnE 'gradient[^;]*(#[fF][fF][dD]|#[dD]4[aA][fF]|#[cC]9[aA]|#[eE]8[cC][dD]|#[eE][cC][dD]7)' pages components common | grep -v '\.old\.' | wc -l
grep -rniE '#dddddd|#ddd\b' common components/cart pages/user-address pages/plugins/scanpay | grep -v '\.old\.' | wc -l
```

## 下轮注意

- 下一项任务：**P0-4 theme.css 渐变清零（8 处）**，含 :99 红渐变 #ff0036→#ffdbe2；theme-red 主题色值向印章红 #9E2B22 收敛（只改色值不动类结构）。
- token 文件已就绪：`common/css/longzhuge.css`，页面可直接用 `var(--seal-red)` 等变量与 `.lz-*` 工具类（nvue 除外）。
- `.lz-price` 用 ::before/::after 生成 `¥:` 与 ` 元`，模板侧只放数字即可；小程序 wxss 支持伪元素 content。
- **首轮扫描的正则太窄**：金渐变实际 26 处（估 7）。以后每类断言都要用宽正则 + `grep -o 'linear-gradient(...' | sort | uniq -c` 人工过目兜底。

## 遗留观察（走查对应页面时处理，勿忘）

- 微信绿 #1AAD19：`common/css/page.css` .bg-green、`components/cart/cart.vue:392` 滑动"移入收藏"按钮 —— 属促销感/微信系色，P1-13 购物车走查时统一处理。
- `pages/goods-comment/goods-comment.css` 进度条还有荧光蓝 #0e90d2/#3bb4f2、橙 #F37B1D —— 冷色/荧光违规，需在商品评论相关走查时收敛为墨黑/驼金系（可并入 P1-7 或单列）。
- **金渐变平涂后的耦合金光投影未动**（属 P1-17 投影轮）：faq:256、exchange-success:53、paytips:60、scan:92,116 等 rgba(196,158,60,.3) 金色 glow。
- **米色底残留**：goods-detail.css:4 `#f5f1ea` 页底、faq.css `#f6f1e8` 系 —— 首轮米色 grep 漏检（正则只查了 5 个色值）。各页 P1 走查时改 --paper 白 / #FBFAF7。
- 非金色渐变残留：paytips:63 粉（失败图标，P1-8）、faq:117-123 绿/青/蓝图标块（P1-10）、goods-detail:174 橙红促销条（P1-7）、goods-category:138 米色淡出条（P1-12）、插件与 DIY 内大量彩渐变（P2-1/P2-2）。
- 金渐变文字曾用 background-clip:text（index.css member-name、scan.css hero title），现改平涂 #B99359，clip 机制保留未动——视觉为纯色金字，后续走查若统一字色可顺手简化。
