# E2E 缺陷报告（给后端工程师）

> 每发现一个问题追加一条。ralph-loop 每轮把新问题写到这里，并在 CHECKLIST 对应用例备注里引用编号(如 `见 F-003`)。
> 严重级：🔴 Blocker(主流程不可用) · 🟠 Major(功能缺失/错误) · 🟡 Minor(体验/边界) · 🔵 Question(待产品/后端确认)

## 模板（复制使用）
```
### F-XXX  [严重级]  [端] 一句话标题
- 关联用例: U-/S-/B-XX
- 复现步骤:
  1.
  2.
- 期望:
- 实际:
- 定位(前端/后端/数据/环境): 
- 证据: .playwright-mcp/xxx.png ; 控制台报错: ... ; 接口: POST ...php?s=... 返回 {code:..., msg:...}
- 建议:
```

---

## 缺陷列表

<!-- ralph-loop 在此下方追加 F-001, F-002 ... -->

### F-001  🟡 Minor  [H5/用户端] 礼品券兑换码输入框无 16 位长度/格式校验  ✅ 已修复(2026-06-29) ⚠️页面适用性订正(2026-07-01)
- **适用性订正(Playwright实测)**: 改造版"兑换中心"实际入口是 `pages/plugins/points/scan/scan.vue`(走 points/scan 接口),**没有任何地方链接到 `giftcard/form`**(见 F-006)。故 F-001 所修的 giftcard 表单页在当前改造版中**未被接入**。好消息:真正在用的 scan.vue 输入框**本就带 `maxlength="16"`**(scan.vue:16),`confirm_event` 去空格+非空校验,实测"输入码→确认兑换→后端返回请先登录→结果态渲染"链路正常(截图 v-04-scan-confirm.png)。F-001 的修复仍对 giftcard 页有效(若将来接入),但当前主流程不依赖它。
- 状态: **已修复**。`pages/plugins/giftcard/form/form.vue`:①输入框加 `maxlength="19"`;②`secret_key_event` 经新增 `secret_key_format()` 实时格式化(大写、仅留字母数字、截断16位、每4位插 `-` → `XXXX-XXXX-XXXX-XXXX`);③扫码结果同样格式化;④`form_submit` 规范化后校验「非空 + 必须16位」,不足提示 `secret_len_error`(请输入16位兑换码),通过后以含 `-` 大写格式提交,匹配后端存储格式(实测样本均为 `XXXX-XXXX-XXXX-XXXX`,后端 `$max=16` 固定生成、无导入)。i18n 已补 zh/en。node --check 语法通过。**前端真机/H5 渲染验证待 HBuilderX 运行。**
- 关联用例: U-04
- 复现步骤(源码白盒,本会话无 Playwright):
  1. 打开 `pages/plugins/giftcard/form/form.vue`。
  2. 卡密输入框(行14)`<input type="text">` 无 `maxlength`，提交校验(行121-123)仅判断"非空"。
- 期望: 按需求"兑换码(16位)"，前端宜限制/提示 16 位长度或格式。
- 实际: 任意长度/字符均可提交，长度与格式校验完全交给后端。
- 定位(前端): `pages/plugins/giftcard/form/form.vue:14, 121-123`。
- 证据: 源码;后端端点存活 `POST api.php?s=plugins/index&pluginsname=giftcard&...&pluginsaction=exchange` 无 token 返回 `{code:-400,"登录失效，请重新登录"}`。
- 建议: 若卡密固定 16 位，前端加 `maxlength="16"` 与格式校验，减少无效请求；否则属可接受设计,本条可降级为 Question。

### F-002  🟠 Major  [H5/小程序] 下单后订阅消息授权未落地(模板ID为空 + 授权页未被跳转)
- 关联用例: U-16(及其前置 U-17 发货服务通知)
- 复现步骤(源码白盒):
  1. 全工程搜索 `requestSubscribeMessage` 仅命中 `pages/exchange-success/exchange-success.vue`。
  2. 该页 `auth_yes_event()`(行61-64)调用 `uni.requestSubscribeMessage({ tmplIds: [] })`——**模板ID数组为空**(行60 已标 TODO)。
  3. 全工程搜索跳转到 `exchange-success` 的代码:无(仅 docs 提及)，即**下单/兑换成功后没有任何地方跳到该授权页**。
  4. 下单主流程(buy.vue/component-payment/order)中无任何 `requestSubscribeMessage` 调用。
- 期望: 用户下单/兑换成功后弹出微信订阅消息授权(物流状态通知)，授权后后端方可推送发货提醒(U-17)。
- 实际: 授权弹窗既无有效模板ID、其所在页面也从未被进入 → 订阅授权实际不会发生，发货服务通知链路缺前置授权。
- 定位(前端+后端协同): 前端 `pages/exchange-success/exchange-success.vue:60-64`(tmplIds 空)；下单成功回调未接入跳转；后端需下发「物流状态通知」订阅消息模板ID。
- 证据: 源码;`pages/exchange-success/对接说明.md` 已自述 TODO#1(模板ID)与 TODO#2(接入跳转)。
- 建议: 后端提供模板ID(随下单/兑换成功接口下发)，前端在下单成功回调 `redirectTo` 到 exchange-success 并填入 tmplIds;或在 component-payment 成功回调统一接入订阅授权。

### F-003  🟡 Minor  [H5/用户端] 个人中心「已兑换礼劵数量」字段后端未返回,恒显示 0  ✅ 已修复并登录实测+DB对账通过(2026-07-01)
- **登录实测(2026-07-01)**: Playwright 用 gpj 登录后打开会员中心,显示「已兑换礼券 **3张**」「积分余额 **499,700**」(截图 v-11-user-loggedin.png);DB 对账 `plugins_giftcard_card_secret` user_id=1 且 is_exchange=1 = **3**、user.integral = **499700**,**完全一致**。F-003 端到端确认修复生效。
- 状态: **已修复并验证**。后端 `app/api/controller/User.php::Center` 现返回 `exchange_coupon_count`(来源:新增 `app/plugins/giftcard/service/CardSecretService.php::UserExchangeTotal($user_id)`,统计 `PluginsGiftcardCardSecret` 中 `user_id=当前用户 且 is_exchange=1` 的条数;经 `CallPluginsServiceMethod` 安全调用,giftcard 插件未启用时回退 0)。接口实测 `POST api.php?s=user/center` 已含 `"exchange_coupon_count":0`(未登录)。前端 `pages/user/user.vue` 已消费该字段,TODO 注释已清。
- 关联用例: U-18(个人中心)
- 复现步骤(源码白盒):
  1. `pages/user/user.vue:218-219` `exchange_coupon_count: data.exchange_coupon_count || 0`，注释明确标 TODO「待后端补充」。
  2. 页面 39-40 行展示「已兑换礼劵 {{exchange_coupon_count}} 张」。
- 期望: 个人中心展示用户真实「已兑换礼劵」数量。
- 实际: 后端 user/center 类接口未返回 `exchange_coupon_count`，前端兜底为 0，用户恒看到 0 张。
- 定位(后端): 用户中心接口需补充 `exchange_coupon_count` 字段。
- 证据: 源码 `pages/user/user.vue:39-40,218-219`。
- 建议: 后端在个人中心数据接口补 `exchange_coupon_count`(已兑换礼劵/卡密张数)。

### F-004  🟡 Minor  [H5/用户端] 个人中心「客服」入口为占位 stub,点击仅 toast 无实际功能  ✅ 已修复并Playwright实测通过(2026-07-01)
- **实测验证(2026-07-01)**: Playwright 打开 `http://localhost:8082/#/pages/user/user`,点击「客服入口」,控制台输出 `[INFO] Launched external handler for 'tel:021-88888888'.` —— 成功唤起配置的客服电话。F-004 端到端通过(截图 v-06-user-center.png)。
- 状态: **已修复**。`pages/user/user.vue::service_event()` 重写为与 `components/online-service` 一致的客服优先级解析:客服系统(`plugins_base.chat`) → 自定义客服(按端) → 企业微信客服(`#ifdef MP-WEIXIN` openCustomerServiceChat / `#ifdef APP` plus.share / `#ifdef H5` url_open) → 电话客服(`call_tel`),均未配置时 toast `service_not_configured`(客服未配置)。本店后台配置实测命中**电话 021-88888888**(`common_app_is_online_service=1`、自定义/企业微信为空),故点击将拨号。i18n 已补 zh/en。node --check 语法通过。**真机/H5 验证待 HBuilderX 运行。** 注:微信小程序「原生在线客服」(open-type=contact)须为按钮、无法 JS 触发,本店未配置该模式故不受影响;若后续仅配置该模式,需将菜单项改为 `<button open-type="contact">`。
- 关联用例: U-21
- 复现步骤(源码白盒):
  1. `pages/user/user.vue:286-289` `service_event()` 仅 `uni.showToast({title:'客服'})`，注释标 TODO「接入项目在线客服」。
  2. 个人中心 86-91 行「客服」菜单 @tap=service_event。点击无拨号/无微信客服/无在线客服。
- 期望: 客服入口可拨打电话(H5/App)或调起微信客服(小程序 open-type=contact)或打开在线客服。
- 实际: 仅弹一个 toast，功能缺失。
- 定位(前端): `pages/user/user.vue:286-289`。注:工程已有 `components/online-service/online-service.vue`(含 `open-type="contact"` 微信客服按钮)及 FAQ 页客服分类回退,**能力存在,仅个人中心此入口未接线**。
- 证据: 源码;`pages/faq/对接说明.md` TODO#2 也描述了客服接入方案。
- 建议: 将 service_event 接 `components/online-service`(读 `config.common_app_is_online_service` 等),或按平台 `#ifdef MP-WEIXIN` 用 `open-type=contact`、H5/App 用 `makePhoneCall`/跳客服文章分类。

### F-005  🟠 Major  [H5/首页积分商城] 商品卡「积分: N」显示的是"购买赠送积分"而非"兑换所需积分",且多数为 0  ✅ 已修复并实测(2026-07-01)
- **修复(2026-07-01)**: 根因是前端未读正确字段。后端 `goods_exchange_data`/`goods/detail` **实际有返回 `plugins_points_exchange_integral`(兑换所需积分)**(实测:七匹狼=100/PRADA=200/华为=100),但前端 `goods_points_view` 错用 `exchange_integral||give_integral||price` 回退链。已改为读 `item.plugins_points_exchange_integral`,两处修复:`pages/index/index.vue:306`、`pages/goods-detail/goods-detail.vue:133`。Playwright 实测:首页 100/200/100(截图 v-14)、详情页"积分：100"(原0),均正确。模型确认=固定积分价(华为带 exchange_price=1600 为"100积分+¥1600"混合兑换,当前仅展示积分数,混合价展示可后续增强)。**无需产品决策,纯前端字段修复。**
- 关联用例: U-08(首页商品浏览) / 积分商城核心展示
- 关联用例: U-08(首页商品浏览) / 积分商城核心展示
- 发现方式: Playwright 实测 `http://localhost:8082/#/` 首页,截图 `v-01-home.png` —— 七匹狼/PRADA 显示「积分:0」,仅华为畅享「积分:10」。
- 复现/定位:
  1. 首页 `pages/index/index.vue:306` `goods_points_view(item) = item.exchange_integral || item.give_integral || item.price || 0`。
  2. 接口 `plugins/index&pluginsname=points&...action=index` 的 `base.goods_exchange_data` **未返回 `exchange_integral` 字段**,只有 `price / give_integral / min_price / original_price`(已实测:七匹狼 give_integral='0'、华为 give_integral='10')。
  3. `exchange_integral` 为 undefined → 回退到 `give_integral`(JS 中 `'0'` 为真值字符串直接返回)→ 卡片显示的是**购买赠送积分**,不是兑换所需积分;未配赠送积分的商品恒显示「积分:0」。
- 期望: 积分商城每个商品展示「兑换所需积分」(用户用多少积分可兑换/抵扣)。
- 实际: 展示的是"购买该商品赠送的积分"(give_integral),语义相反,且大多数=0,首页看起来"0积分可兑换",严重误导。
- 定位(前端+后端+产品):
  - 后端 points 插件用的是**积分抵扣模型**(base 里 `is_integral_deduction / deduction_price / order_max_integral / order_price_max_rate`),并非"固定积分价"模型,故没有单品 `exchange_integral`。
  - 需产品确认积分商城的目标模型:①固定积分价(需后端为商品新增"兑换所需积分"字段并在 goods_exchange_data 返回) or ②按价格×抵扣率折算积分(前端按 base 的 deduction 规则计算展示)。
- 证据: 截图 `.playwright-mcp`/`v-01-home.png`;接口实测 goods_exchange_data 字段无 exchange_integral;源码 `pages/index/index.vue:306`。
- 建议: 先定模型。若②,前端 `goods_points_view` 改为按 `price` 与 `base.order_price_max_rate/deduction_price` 折算;若①,后端补 `exchange_integral` 字段。当前"回退 give_integral"的实现无论如何都是错的,应优先纠正。

### F-006  🟠 Major  [H5/兑换中心] 改造版兑换只走 points/scan(仅发积分),支持"实物/余额/优惠券"的 giftcard 插件未接入  ✅ 已实现并实测(2026-07-01)
- **已实现(2026-07-01, 产品确认"兑换券要能兑实物")**: 打通"兑换券兑实物"全链路,Playwright + API + DB 三重实测通过。
  - 后端: ①`CardSecretExchange` 成功返回 `data_type`(供前端路由);②新增 `CardSecretService::UserExchangeGoodsList` + `api/Index::ExchangeGoods`(返回用户已兑换未领取的实物商品,含图/名/价/url)。均 php -l 通过,备份 *.bak.20260701。
  - 前端: ①`scan.vue::confirm_event` 改为"先试 giftcard 兑换(覆盖余额/券/积分/实物)→ 成功按 data_type 路由(3=实物→跳我的兑换商品页;其它→提示刷新)→ 非礼品卡回退 points/scan 积分码";②输入框 maxlength 16→19(适配 giftcard 19位卡密);③新建 `pages/plugins/points/exchange-goods/exchange-goods.vue`(金色主题,列出实物商品,去下单跳详情),pages.json 注册路由。
  - 实测(自建 E2E 实物卡绑定商品109七匹狼): 兑换中心输码→确认兑换→giftcard兑换(data_type=3)→跳"我的兑换商品"列出七匹狼→点"去下单"→商品详情(id=109)。DB 核对卡密 user_id=1/is_exchange=1 正确。E2E 数据已全部清理(0残留,gpj积分/已兑换数回基线)。截图 v-15~v-17。
  - **完整结账实测(2026-07-01, 已补验)**: 详情"立即兑换"→选数量→结算页,结算页正确显示**「礼品卡商品兑换 －¥100.00」→ 合计¥0**(券全额抵扣,截图 v-19);提交订单→支付成功(¥0直接完成,v-20)。DB 核对:订单生成(id=8,total/pay=0.00,状态2待发货)、**券消耗 use_status=1 且 use_data 记录 order_id/goods/use_time**、库存 88887→88886。E2E 订单及全部关联行(order/detail/address/status_history/inventory_log)+E2E卡已清理,库存/订单数全回基线(0残留)。**"实物券→兑换→领取→下单→券全额抵扣→券消耗→库存扣减"全链路端到端闭环验证通过。**
- 关联用例: U-04~U-07(礼品券兑换) / 产品需求"兑换券兑换成功后发放实物或积分"
- 发现方式: Playwright 实测 + 前后端源码交叉验证(2026-07-01)。
- 事实:
  1. 改造版"兑换中心"页 `pages/plugins/points/scan/scan.vue` 提交到 `points/scan`(`ScanQrcodeService::UserScanQrcodeData`)——该后端逻辑是"兑换码=`PluginsPointsScanQrcode.qrcode` → 给用户 `inc('integral')` **加积分**",**只发积分**。
  2. 支持"余额/优惠券/积分/**实物商品**"四种发放的是**另一套** `giftcard` 插件(`CardSecretService::CardSecretExchange`,data_type 0/1/2/3),但**全工程无任何入口链接到 giftcard 前端页**(pages.json 注册了分包但无跳转)。
- 影响: 若产品预期"兑换券可兑换实物",**当前改造版前端不满足**——用户端只有"扫码/输入码领积分"这一条路。后端具备实物兑换能力,但前端没接。
- **后端兑换能力已实测(2026-07-01)**: 自建 E2E 积分卡(data_type=2,面值100)→ 用 gpj token 调真实兑换 API `giftcard/index/exchange` → 返回`兑换成功`,gpj 积分 **499700→499800(+100)**、已兑换数 **3→4**、卡密置 is_exchange=1。**证明 giftcard 兑换→积分到账后端链路完全正常**(实物 data_type=3 走同一 CardSecretExchange,逻辑一致)。测试数据已全部清理还原(积分回 499700、已兑换回 3、E2E卡密0残留)。**结论:后端 OK,缺的是前端入口。**
- 定位(前端架构 + 产品): 需产品明确"兑换券"到底走哪套:
  - 走 points/scan → 只能发积分(现状);
  - 需发实物/优惠券/余额 → 前端需接 giftcard 插件的兑换页(`giftcard/index/exchange`)或把 giftcard 能力并入兑换中心。
- 证据: scan.vue:224-246(提交 points/scan);后端 `ScanQrcodeService.php:53-89`(qrcode→inc integral);`giftcard/service/CardSecretService.php`(data_type 实物分支);giftcard 无前端入口(grep 无引用)。
- 建议: 产品先定"兑换券"的发放范围。若含实物,前端需接 giftcard 兑换链路(兑换页已存在,只差入口与 UI 融合)。

### F-007  🟡 Minor  [H5/全局] 页面主题风格不统一:积分商城页金色主题,登录/搜索页仍是默认红色商城主题  ✅ 已修复并实测(2026-07-02)
- **修复(2026-07-02)**:
  1. **全局主题**: `App.vue` `default_theme: 'red' → 'yellow'`(主色 #f6c133 金黄,与积分商城金色系一致)。实测登录页(确认登录金色按钮/tab金色下划线/协议金色链接,v-24)、搜索页(加购按钮变金,v-25)统一金色调。注:已有用户本地缓存 `theme=red` 的,清缓存/重装后生效。
  2. **LOGO**: 从 `static/icon/logo_graphic_only_preserve.svg`(内嵌156×172 PNG)提取源图,PIL 生成 4 尺寸透明底 PNG(logo-square-200/400、logo-h60 200×60、logo-h90 300×90,存 `static/icon/`),上传服务器 `/static/upload/images/common/2026/07/02/`,更新后端 6 个配置(home_site_logo→h60, _wap→h90, _square→square-200, _app→square-200, admin_logo→h60, admin_login_logo→square-400)并清 runtime/cache。实测:H5登录页显示金铜色新logo(v-23)、后台登录页HTML已引用 square-400。原值(2019占位图)已记录可回滚。
- **遗留(语义层,非主题)**: 搜索页仍展示"¥价格+加购"而非"积分+兑换"——需接 points 插件专用搜索接口(GoodsSearchList,仅返回可兑换商品)做语义改造,属产品级功能项,不在 F-007 主题范畴。
- 关联用例: U-02(登录) / U-10(搜索) / 全局视觉一致性
- 发现方式: Playwright 走查(2026-07-01)。
- 事实: 积分商城核心页(首页/兑换中心/FAQ/个人中心)是**金色积分主题**;但 `pages/login/login`(确认登录红按钮,截图 v-09-login.png)、`pages/goods-search/goods-search`(¥价格+购物车+加购,截图 v-05-search.png)仍是 ShopXO **默认红色商城 UI**,未积分商城化。
- 影响: 用户从积分商城点进搜索/登录时视觉断裂;搜索页展示"¥价格+加购物车"而非"积分+兑换",与积分商城定位不符。
- 定位(前端): 这些页沿用原版 ShopXO 组件/主题,未纳入 index-points 改造范围。
- 证据: 截图 v-05-search.png(¥价+购物车)、v-09-login.png(红主题)。
- 建议: 按积分商城范围补齐搜索/登录等页的主题与"积分+兑换"语义(搜索结果卡应显示兑换积分与兑换按钮,而非¥价与加购)。另:登录页 LOGO 未配置(显示占位"LOGO"),需后台上传。

### F-008  🟡 Minor  [H5/兑换中心] 「近期记录」为硬编码假数据,非用户真实兑换记录  ✅ 已修复并实测(2026-07-01)
- **修复(2026-07-01)**: 后端新增 `CardSecretService::UserExchangeRecordList($user_id)` + `api/Index::ExchangeRecord`(查用户已兑换卡密→映射 state/title/date_text/status_text,实物按 use_status 区分"立即使用/已领取");前端 `scan.vue` record_list 改空数组、新增 `get_record()` 在 onShow 及兑换成功后拉取,删除硬编码假数据。Playwright 实测(gpj):近期记录显示 3 条**真实**记录(实物商品兑换2026.07.01已领取 / 余额兑换¥10 2026.06.01已兑换 / 实物商品兑换2026.06.01已领取),与 API/DB 一致(截图 v-21)。空态"暂无兑换记录"已存在。php -l + node --check 均通过,备份 *.bak.20260701。
- 关联用例: U-07(兑换记录)
- 发现方式: 走查 + 源码(2026-07-01)。
- 事实: `pages/plugins/points/scan/scan.vue:116-120` 的 `record_list` 是写死的 3 条假数据(立即使用/已兑换/已过期),`get_data` 并未用接口数据填充它(注释也自述"scan 接口暂未返回记录列表,以下为预览假数据")。用户无论是否登录、有无真实记录,都恒显示这 3 条假记录。
- 期望: 展示用户真实兑换记录(或无记录时空态)。
- 实际: 恒显示 3 条假记录,误导。
- 定位(前端+后端): 后端需提供用户兑换记录列表接口(giftcard 已有 `index/index` 数据列表可复用,或 points 侧补充),前端改由接口填充 record_list;"查看更多"当前也是 toast 占位。
- 建议: 接 giftcard `index/index`(用户已兑换卡密列表)或后端补记录接口,替换假数据。

---

## 汇总（每轮更新）
| 严重级 | 发现 | 已修复 | 待处理 |
|---|---|---|---|
| 🔴 Blocker | 0 | 0 | 0 |
| 🟠 Major | 3 | 2 (F-005, F-006) | 1 (F-002) |
| 🟡 Minor | 5 | 5 (F-001/F-003/F-004/F-007/F-008) | 0 |
| 🔵 Question | 0 | 0 | 0 |

> 修复记录(2026-06-29): F-003(后端 user/center 补 exchange_coupon_count,已实测)、F-001(前端兑换码16位校验+格式化)、F-004(前端客服入口接线)。
> 待处理: **F-002**(订阅消息授权,需微信平台模板ID)、**F-005**(首页积分展示语义错误,Playwright实测发现,需先定积分商城模型)。

## 「功能是否实现」结论速览（验证完成后填）
> 对照需求逐项给后端一个明确结论：✅已实现 / ⚠️部分实现 / ❌未实现 / ❓未能验证(小程序专有等)
> 说明:2026-07-01 起 **Playwright 已可用 + H5 dev server 在 localhost:8082 + 用户登录已实测可用(home_user_login_img_verify_state=0,无图形验证码,gpj 账号直接登录成功)**,已对 H5 用户端做真机走查(截图 v-01~v-12)。后台/供货商端仍为独立工程,未实测。

### 用户端（H5, Playwright 实测 @ localhost:8082）
- 登录(微信一键 U-01 / 账号 U-02 / 手机绑定 U-03)：✅账号登录 **实测通过**(gpj/gpj123456 UI 登录成功→跳首页,登录后会员中心显示真实积分/订单);登录页渲染正常(截图 v-09,红主题见 F-007)。微信一键属小程序专有(H5不适用)。
- 礼品券兑换(U-04 输入/U-05 校验/U-06 余额/U-07 记录)：✅前端实现完整(giftcard/form+index,卡密输入/扫码#ifndef H5/后端校验/余额跳钱包/记录列表),giftcard 端点全部存活。⚠️未登录实测。**🟡 F-001 ✅已修复**:兑换码已加16位长度/格式校验+自动格式化。
- 商品浏览(U-08 列表/U-09 详情/U-10 搜索分类)：✅**Playwright UI 实测通过**(首页积分商城网格 v-01、商品详情富文本 v-02、搜索页排序+商品 v-05)。**🟠 F-005 ✅已修复**(首页/详情改读 plugins_points_exchange_integral,实测显示 100/200/100)。
- 下单支付(U-11 用券免支付/U-12 收货/U-13 订单生成/U-14 列表/U-15 详情)：✅前端实现完整(buy.vue 积分兑换区+total_price=0免支付、地址+自提、order 链路、订单状态tab、详情),buy/order/useraddress 端点存活。⚠️未登录实测。
- 消息通知(U-16 订阅授权/U-17 发货通知)：❌**订阅授权未落地**(全工程仅 exchange-success 一处 requestSubscribeMessage,tmplIds 空且该页无人跳转;下单主流程无授权)——**🟠 F-002**。U-17 发货服务通知属后端推送,前端无代码,且前置依赖 U-16。
- 个人中心(U-18 订单/U-19 礼品券/U-20 地址/U-21 客服/U-22 FAQ)：✅**登录实测通过**(v-11)——会员中心显示 gpj、积分余额499,700、我的订单角标(待发货4/待收货1)、地址/客服/FAQ 菜单齐全;**兑换中心近期记录**3条(立即使用/已兑换/已过期)正常(v-12);**FAQ ✅实测通过**(分类+手风琴展开富文本 v-07/v-08);**客服入口 ✅实测通过**(点击唤起 tel:021-88888888)——**🟡 F-004 ✅**;**「已兑换礼劵数」✅登录实测+DB对账通过**(显示3张=DB3条)——**🟡 F-003 ✅**。

### 供货商端（独立小程序,源码不在本仓）：❓全部未能验证
- S-01~S-12 全部 ❓未能验证:供货商端为独立小程序工程(不在本仓,无法源码佐证);其数据隔离/订单/推送的「后台侧面佐证」需 admin 登录(验证码阻塞)。后端确有供货商概念(goods.supplier_id 实测存在)。需人工真机 + 后台核查。

### 平台运营后台（独立 PHP 项目,源码不在本仓）：❓全部未能验证
- B-00~B-22 全部 ❓未能验证:admin 为独立 PHP 项目(源码不在本仓);登录页含验证码(已实测),无 Playwright 无法读图登录。侧证:商品(search/datalist)、分类(goods/category)、供货商关联(supplier_id)、卡券(giftcard 端点)等模块数据在用户端可见,说明对应后端模块存在;但后台 UI 的增删改查/导出/统计/日志等均未实测。

### 贯通 E2E-1：❓未能端到端实跑
- 横跨 admin + H5 两端登录,本会话皆受验证码阻塞。各环节(兑换/下单/关联供货商)已分别源码+端点佐证就绪,缺登录态串联实测。

### 一句话总结
核心**用户端前端实现完整度高**:商品浏览已实测可用、礼品券兑换与下单链路代码完整且后端端点全部存活;**真实缺陷集中在消息通知(F-002 订阅授权未落地,Major,待处理)与个人中心收尾项(F-001/F-003/F-004,Minor,均已于2026-06-29修复)**;**供货商端与运营后台因环境(独立工程+验证码登录+无Playwright)本会话无法实测**,需补 Playwright/人工真机/可登录后台后复测。

---

## 走查复核 W-001~W-008（工作人员反馈，2026-07-02，全部修复并 Playwright 实测）

| # | 反馈问题 | 结论/修复 | 实测证据 |
|---|---|---|---|
| W-1 | 搜索跳转的商品列表还是老样式 | ✅重写 `pages/goods-search`：数据源改为 points 兑换商品接口（原通用搜索返回全站28个商品、积分=0，语义就不对），金色卡片=首页同款，关键词前端过滤 | v-28：搜"华为"只剩1条 |
| W-2 | 礼券代码兑换提示"没有相关扫码" | ✅无效码兜底提示改为「兑换码无效或已被使用，请核对后重试」（scan.vue code_fallback 标记） | v-29 |
| W-3 | 近期记录"立即使用"点击无反应 | ✅后端 UserExchangeRecordList 补 `data_type`；前端按类型路由：实物可用→我的兑换商品、实物已领→提示、积分→积分明细、余额→钱包、优惠券→我的优惠券 | 实测：已领取实物→toast、余额→跳我的钱包 |
| W-4 | 首页 全部/我能兑换/积分/分类 切换无效果 | ✅前端实现：我能兑换=所需积分≤我的积分（未登录先登录）；积分=升/降序切换；分类筛选=ActionSheet 选分类过滤（后端 points BaseService 为兑换商品附加 category_names） | 实测：升100,100,200/降200,100,100；选"手机"只剩华为；我能兑换3/3 |
| W-5 | 签到是按钮还是状态？点了没反应 | 产品答复：是**签到入口按钮**。后端**未安装 signin 插件**，故原跳转路径不存在。已改为：插件未装→提示「签到功能暂未开通」；装上后自动跳 `/pages/plugins/signin/user/user` 签到页 | 实测 toast 生效 |
| W-6 | FAQ 帮助中心/店主之家顶部被盖住 | ✅根因：`.faq-hero` 是定位元素、绘制层高于上移(-20rpx)的分类卡，卡片顶部10px被盖。`.faq-cards` 加 `position:relative;z-index:1` | v-30 对比 v-27：卡片圆角顶完整 |
| W-7 | FAQ"联系客服"跳到常见问题列表 | ✅service_event 改为与个人中心一致的四级链路（在线客服→自定义→企业微信→电话）。当前配置命中电话 021-88888888 | 实测不再跳文章列表 |
| W-8 | 详情"立即兑换"弹层还有加入购物车 | 产品答复：**应去掉**（导航已无购物车）。已在 goods-detail 传入弹层前过滤 cart 按钮，购买按钮文案统一"立即兑换" | v-34：弹层仅一个金色"立即兑换" |

> 后端改动（均有 .bak.20260702 备份）：`plugins/giftcard/service/CardSecretService.php`（记录补 data_type）、`plugins/points/service/BaseService.php`（兑换商品附加 category_names），php -l 通过、cache 已清、接口实测返回正常。
> 前端改动：goods-search.vue/css（重写）、index.vue（筛选Tab）、scan.vue（W-2/W-3）、user.vue（签到）、faq.vue（客服链路）、faq.css（W-6）、goods-detail.vue（W-8）、locale/zh+en.json（signin_not_open/category_*/search_* 等 key）。

> 补充（2026-07-02 二轮）：① 兑换中心"查看更多"已接分页（后端 ExchangeRecord 支持 page/limit，前端追加加载，末页提示"没有更多记录了"；实测 limit=2 时 2→3 条追加成功）；② 业务无钱包/优惠券概念，记录点击引导修正：实物→我的兑换商品/已领取提示，其余积分类→积分明细（实测跳转正确）。giftcard api/Index.php、CardSecretService.php 均有 .bak.20260702 备份。

> 补充（2026-07-02 三轮，选品流程简化）：商品管理编辑页「积分商城兑换」卡片新增**首页显示**下拉（goods 表新列 plugins_points_is_home_show，Hook 保存链路与兑换积分同机制）；首页数据源改为「勾选首页显示的可兑换商品」，全部未勾选时回退旧应用配置选品（兼容）；新增 points/goodslist 接口（分页+关键词，返回全部 integral>0 商品+category_names），前端 goods-search 改接该接口（触底翻页+服务端搜索）。实测：勾选 id=32 后首页仅剩该商品、还原后回退 6 个配置商品；列表页展示全量 10 个、搜"餐具"精确 4 条。备份：points 插件 Hook.php/api/Index.php/BaseService.php(.bak.20260702b)/goods_edit.html 均有 .bak.20260702。
