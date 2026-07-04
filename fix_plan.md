# fix_plan.md — 龙珠阁 UI 走查任务清单

> 规范依据：`docs/design.longzhuge.md`（权威版）。每轮只做一项，完成后勾选。
> 扫描范围：`pages/`、`components/`、`common/`、`App.vue`。**排除 `uni_modules/`（第三方，不改）与 `*.old.*`（遗留死码）。**
> 生成时间：2026-07-04（Ralph 第 1 轮全量扫描）

---

## P0 — 基建与禁止项清零（阻塞性）

- [x] **P0-1 建立龙珠阁设计 token 文件** `common/css/longzhuge.css`：§1.1 全局 token（--paper/--ink/--ink-soft/--camel-gold/--imperial-gold/--seal-red）+ §2 六系列主题类（`.series-qinghua` 等，含暗场底色/主题色变量），在 `App.vue` 全局样式引入。仅新增文件，不动业务。
- [x] **P0-2 清除微信红系（5 处）**：`common/css/page.css:596`（#E64340 !important）、`components/cart/cart.vue:398`（js 内联 backgroundColor '#E64340'→ 改为印章红色值字符串，属样式值可改）、`pages/goods-comment/goods-comment.css:35`（#dd514c）→ 统一印章红 `#9E2B22`。（goods-detail.old.* 属死码不改）
- [x] **P0-3 清除金色渐变 → 驼金平涂 #BE8F5B（实际 26 处，首轮正则漏检浅金导航系）**：CTA/图标/徽章→#BE8F5B 平涂；金渐变文字/下划线→#B99359 平涂；浅金/米金导航条与头部大底→纯白 #FFFFFF。覆盖 13 文件（index、user、paytips、exchange-success、faq、goods-search、goods-detail、user-order、user-address、user-integral、points/scan、points/exchange-goods、antifakecode）。
- [x] **P0-4 theme.css 渐变清零（8 处）**：全部 gradient 改各主题主色平涂；theme-red 整块收敛为印章红体系（#ff0036→#9E2B22、浅色→#eddcda 系、pair 黄→御金 #B99359）；App.vue get_theme_color() 的 red/red_light 同步。
- [x] **P0-5 冷灰 #DDD 描边 → 淡暖灰 #E8E4DC**：`common/css/lib.css`、`common/css/page.css`、`common/js/common/common.js`（DIY 样式默认值中的色值字符串）、`components/cart/cart.vue`、`pages/user-address/user-address.css`、`pages/plugins/scanpay/index/index.vue`。（video/ask/live/weixinliveplayer 等非核心插件延后到 P2-1）

## P1 — 核心链路逐页走查（首页→商品→兑换→个人中心）

- [x] **P1-1 全局导航与 tabbar**：pages.json 全部导航已纯白+黑字（审计确认，0 违规）；nav-back 自绘返回条白底 ✅；tabbar=component-diy-footer（后端 DIY 配置驱动配色），前端兜底默认色改墨黑 #1a1a1a/御金选中 #b99359；footer 无描边无投影 ✅。
- [x] **P1-2 排印基建**：longzhuge.css 已含 .lz-serif/.lz-title/.lz-section-title(「」)/.lz-price(¥: 元)/.lz-price-num/.lz-num-hero/.lz-series-name + ≥14px 注释；本轮补 .lz-nav-title（宋体居中导航标题）与 .lz-desc（说明文字层级，与主标题 ≥1.8:1）。
- [x] **P1-3 pages/index（首页/积分商城）走查**：米色页底→纯白；会员卡去金描边/去投影/去圆角、底色 #fbfaf7 弱分组；搜索框去投影（存量胶囊平涂保形）；商品卡去圆角去投影；积分数字宋体+印章红；兑换按钮印章红底白字直角；「热门推荐」宋体+「」；亮金 #d5b345/#c8a04a 系全部收敛为御金 #b99359（含 tabbar SVG 图标 base64 重编码）；banner 通栏出血 ✅。
- [x] **P1-4 pages/user（个人中心）走查**：米色页底 #f3eee3→纯白；三张卡（积分/订单/菜单）去圆角去投影、#fbfaf7 弱分组；导航标题宋体墨黑；积分数字宋体；签到按钮金光投影→1px 御金描边透明底；订单角标 #e1574b→印章红；分割线全部 #f1eee8；「」card-title；杂金（#b8923a/#d4b86a/#d5b345 含 SVG fill）→御金 #b99359。
- [x] **P1-5 pages/plugins/points/index（积分插件首页）走查**：分享按钮荧光橙渐变 #FF9747→#FF6E01 → 驼金平涂 #be8f5b（半胶囊存量保形）；分割线 #eee→#f1eee8；页面无投影；其余样式走全局 theme 类（已印章红化）；积分数字为行内小字非 hero，宋体规则不适用。注：真正的"兑换中心"大数字页是 points/scan（P1-6 覆盖）。
- [x] **P1-6 points/exchange-goods + points/scan（兑换中心）走查**：scan 页 5 处金光投影清零、hero 标题 clip 金字→宋体 44rpx 墨黑（御金下划线保留为唯一装饰）、礼券卡/记录卡去圆角去投影 #fbfaf7、输入行加 #e8e4dc 下划线、usable 金 accent 条删除（禁止项）、状态 chip 米金/冷灰→白底+印章红/ink-soft/浅印章红文字、「近期记录」宋体+「」；exchange-goods 页米金渐变底→纯白、标题宋体墨黑、卡片直角 #fbfaf7、积分 tag 米金→白底印章红。
- [ ] **P1-7 pages/goods-detail（商品详情）走查**：T3 映射——顶部沉浸图+白场参数区；价格印章红 + `¥:xxxx 元` 格式；去投影去描边。
- [ ] **P1-8 pages/exchange-success + paytips 走查**：P0-3 后残留的金色面积≤5% 核查、圆角、按钮形制。
- [ ] **P1-9 pages/user-integral（积分明细）走查**：大数字宋体、去框、分割线 ≤1px #F1EEE8。
- [ ] **P1-10 pages/faq 走查**：去框化、section 标题「」、装饰元素≤1/屏。
- [ ] **P1-11 pages/login 走查**：输入框下划线式或 #E8E4DC 描边、主按钮墨黑底白字、去圆角胶囊渐变。
- [ ] **P1-12 pages/goods-category + goods-search 走查**：tab/筛选样式统一、卡片去框、每屏≤2.5 卡片密度（列表间距）。
- [ ] **P1-13 pages/cart + components/cart 走查**：结算按钮墨黑/印章红、价格格式、去圆角胶囊。
- [ ] **P1-14 pages/user-order + user-order-detail 走查**：订单卡片去框（留白分组）、状态色收敛（禁荧光/冷色）、按钮形制统一。
- [ ] **P1-15 pages/user-address(+save) 走查**：输入框形制、去 #ddd（P0-5 已清主文件）、按钮统一。
- [ ] **P1-16 components/ 共享业务组件走查**（goods-buy、payment、search、share-popup 等）：按钮/弹层形制统一，弹层投影 opacity≤0.06。
- [ ] **P1-17 投影清理（核心链路）**：上述已走查页面外，grep box-shadow 复核核心链路残留，页面内投影清零（浮层除外 ≤0.06）。
- [ ] **P1-18 全量复核轮**：§5 清单逐项 grep 断言（禁止色=0、金渐变=0、核心页圆角≤2px 或存量平涂胶囊）+ 抽查 diff，确认无新增违规。

## P2 — 非核心插件与资源（不阻塞完成承诺）

- [ ] **P2-1 非核心插件禁止项清扫**：`pages/plugins/`（coin/wallet/coupon/signin/ask/video/live/weixinliveplayer/binding/magic/distribution 等）内 #ddd、渐变按红/金规则批量替换（每轮 1-2 个插件）。
- [ ] **P2-2 diy 组件（tabs-carousel/tabs-magic 等）渐变与圆角收敛**：注意 DIY 为服务端配置驱动，仅改组件默认样式不改配置解析逻辑。
- [ ] **P2-3 `.old.*` 遗留文件处理**：确认无引用后在 AGENT_NOTES 标记为死码（不删除、不修改，避免误伤）。
- [ ] **P2-4 运营 banner/静态图资源走查**：`static/` 内与新体系冲突的促销感图片列「需重制」清单。
- [ ] **P2-5 系列主题机制落地**（data-series 换肤示例应用到系列落地页/详情页，视后端字段支持情况，纯样式层）。

---

## 完成条件
P0 + P1 全部勾选，且 P1-18 复核无新增违规 → 终检报告 → 输出完成承诺。
