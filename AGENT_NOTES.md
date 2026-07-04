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

- 下一项任务：**P1-14 pages/user-order + user-order-detail 走查**（订单卡片去框、状态色收敛、按钮形制统一）。
- 微信绿已全项目清零 ✅（"遗留观察"第 1 条完成）；"成功/正向"语义色定为千里江山青绿 #2f4a4a。
- **坑：部分 css 文件是 CRLF 行尾**（goods-category.css 等，且内部混合）。python open().read()/write() 会静默转 LF 造成全文件 diff——改用 bytes 模式或改完后恢复 CRLF。sed 不受影响。
- FAQ 分类图标色板定案：青绿 #2f4a4a / 灰绿 #547070 / 青花蓝 #3a3f8f（规范明文例外）/ 驼金 #be8f5b——多分类图标需要区分色时用这套。
- goods-comment 进度条 4 色已收敛（含 #5eb95e 荧光绿→千里江山青绿 #2f4a4a），该页其余样式待全量复核轮抽查。
- points/index 是 ShopXO 原生积分页（分享/明细），主要靠全局 theme 类；自定义"兑换中心"界面在 points/scan（礼券码输入+大标题）。
- SVG base64 图标重上色套路：python 解码→replace fill→重编码（见第 10 轮），比手工 sed 安全，能同时断言所有 fill 合规。
- **走查套路已成型**（P1-3 首页为模板）：① grep box-shadow/border-radius/border/色值盘点 → ② 米色底→白、卡片去框直角、数字宋体、价格/积分→印章红、CTA→印章红或墨黑直角、亮金→御金、section 标题宋体+「」伪元素 → ③ 断言：禁止色 0、投影仅浮层 ≤0.06、容器圆角 0（圆形头像/图标 50% 与存量胶囊除外）。
- 页面直接写 font-family 栈而非 .lz-* 类（模板不加类、纯 css 实现），diff 更小；「」用 ::before/::after 加在既有标题类上。
- **宋体依赖系统字体栈**：未打包字体文件（小程序包体积/不新增资源依赖）。iOS/Mac 有 Songti SC；部分 Android 无衬线中文时回退 serif→黑体，属可接受降级。若品牌方后续要求强一致，需在 H5 侧 @font-face 引入 Noto Serif SC（另立任务）。
- P0 已全部完成 ✅（token / 微信红 / 金渐变 / theme.css / 冷灰）。
- **tabbar 架构**：pages.json tabBar 全 transparent（占位），真实渲染链 = components/common/common.vue → component-diy-footer（components/diy-footer/diy-footer.vue），配色/图标来自后端 `get_config('app_tabbar')` DIY 配置。前端只有兜底默认色（已改墨黑/御金）。**要让 tabbar 真正龙珠阁化需在后端 DIY 后台配置图标与色值**——列为运营侧事项，不阻塞前端走查。
- diy-footer.vue:120 存在既有表达式优先级问题（'color:'+x || fallback 恒取左侧），兜底色实际难触达——属业务逻辑，铁律禁改，仅记录。
- `common/css/page.css:368` `.sales-price` 价格色是橙红 #E22C08 —— 「价格=印章红」规则，全站价格组件用它，改动影响面大，纳入 P1 走查（建议单独在 P1-7 商品详情轮或价格统一时处理）。
- `pages/plugins/scanpay/index/index.vue:34` 选中态 #E83B11 荧光橙红未动（本轮只清 #ddd），归 P2-1。
- theme-red 印章红色阶已定：主 #9e2b22 / 浅 #eddcda / 中浅 #c98f89 / 更浅 #f4eae8 / 最浅 #faf5f4 / pair #b99359——各页走查时红色系一律对齐这套色阶。
- 其余 7 个主题（yellow/black/green/orange/blue/brown/purple）只做了去渐变，色值未龙珠阁化——default_theme 为 red，非默认主题不在本次规范范围（如需全部收敛另立任务）。
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
