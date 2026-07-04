# E2E 全功能验证清单（ralph-loop 进度表）

> 这是 ralph-loop 的进度真相表。每轮迭代：选一个 `⬜ TODO` 用例 → 执行 → 回填状态/证据 → 保存。
> **状态值**：`⬜ TODO`(未测) · `🔄 DOING`(进行中) · `✅ PASS` · `❌ FAIL` · `⛔ BLOCKED`(环境阻塞,如H5未启动) · `🙋 MANUAL`(需人工,小程序专有,Playwright不可测)
> 失败/异常的细节写进 `E2E_FINDINGS.md`，本表「证据/备注」列只填一句话 + FINDINGS 编号(如 `见 F-003`)。
> **完成判定**：本表不存在任何 `⬜ TODO` 或 `🔄 DOING` 时，验证完成。

图例：🟢可自动化 🟡部分可测 🔴不可自动化(转人工)

> **⚠️ 本会话环境状态（2026-06-27 iter1 记录）**
> 1. **Playwright MCP 未在本会话加载**（ToolSearch 查无任何 browser_* 工具，仅 pencil/cloudflare/figma 等）。`.playwright-mcp/` 内是历史会话产物。→ 所有依赖浏览器 UI 的 🟢/🟡 用例本会话无法实测。
> 2. **后端登录强制图片验证码**：`POST api.php?s=user/login` 仅带 accounts/pwd/type 返回 `{code:-10,"图片验证码不能为空"}`，故 curl 旁路登录也走不通（admin 登录同理）。
> 服务器本身可达：H5 localhost:8083 → 200，admin xunxiang.ysfaedu.com → 200。
> **结论**：本会话只能推进「源码白盒佐证」类用例（🔴 MANUAL 及前端是否实现的判定）。需浏览器实测的用例在 Playwright MCP 重新加载前标 ⛔ BLOCKED。

---

## 一、用户端（H5，连接 xunxiang 后端）

| ID | 模块 | 功能点 | 可测性 | 测试要点 | 状态 | 证据/备注 |
|---|---|---|---|---|---|---|
| U-01 | 登录账户 | 微信一键登录(昵称/头像/手机号) | 🔴 | 小程序专有,H5用账号登录代替 | 🙋 MANUAL | 源码佐证:已实现。login.vue MP-WEIXIN 分支 uni.getUserProfile 取昵称/头像→user_auth_login;getPhoneNumber 按钮(行71)→confirm_phone_number_event 取手机号。需真机微信端人工确认 |
| U-02 | 登录账户 | H5 账号登录(gpj) | 🟢 | 用 gpj/gpj123456 登录,进入个人中心确认登录态 | ⛔ BLOCKED | 源码已实现用户名/密码登录(login.vue type=username+accounts+pwd+图形验证码,POST user/login);接口存活,无验证码返回code:-10。本会话无Playwright且登录强制图形验证码,无法实测登录态 |
| U-03 | 登录账户 | 手机号绑定 | 🟡 | 进入绑定页,验证表单与提交;短信验证码环节人工辅助 | ⛔ BLOCKED | 源码已实现(user/appmobilebind+onekeyusermobilebind+短信验证码发送 loginverifysend等)。需短信验证码,本会话无法实测 |
| U-04 | 礼品券兑换 | 兑换码输入(16位,支持扫码) | 🟢 | 找到兑换入口,输入框可输16位;扫码按钮存在即可(H5扫码🟡) | ⛔ BLOCKED | 源码已实现:giftcard/form.vue 卡密输入框+扫码(uni.scanCode,#ifndef H5 故H5无扫码符合预期),POST giftcard/index/exchange(端点存活返回-400需登录)。备注:输入框无16位长度/格式校验 见 F-001。需登录态实测 |
| U-05 | 礼品券兑换 | 兑换码校验(存在/未过期/未使用) | 🟢 | 输错误码→提示无效;输已用码→提示已用;输有效码→成功 | ⛔ BLOCKED | 源码:校验由后端 giftcard/index/exchange 驱动,前端按 code==0/!=0 展示 success_msg/error_msg(form.vue:136-162);端点存活。各类错误提示需登录态实测 |
| U-06 | 礼品券兑换 | 余额展示(剩余金额) | 🟢 | 兑换成功后我的礼品券/钱包显示对应余额 | ⛔ BLOCKED | 源码:giftcard/index 兑换记录展示 secret_value_text;data_type==0 跳钱包,wallet/user 显示 `user_wallet.normal_money` 余额(user.vue:26);端点存活。需登录实测到账金额 |
| U-07 | 礼品券兑换 | 兑换记录(历史+状态) | 🟢 | 兑换记录列表展示历史券及状态 | ⛔ BLOCKED | 源码:giftcard/index data_list 展示 exchange_time/secret_key/secret_value + use_data(关联订单号/商品/使用时间)即历史+状态(index.vue:1-58);端点存活。需登录实测 |
| U-08 | 商品浏览 | 商品列表(图/名/价) | 🟢 | 首页/列表页商品卡片三要素齐全 | ✅ PASS | 接口实测:`POST search/datalist` code:0,商品含 images(图)+title(名)+price/min_price(价)三要素;源码 goods-category.vue 渲染确认。⚠️无Playwright,UI视觉未截图 |
| U-09 | 商品浏览 | 商品详情(详情图/规格) | 🟢 | 进详情页,详情图与规格说明展示 | ✅ PASS | 接口实测:`POST goods/detail` code:0,goods 含 photo(相册/详情图)+spec_base/specifications/is_exist_many_spec(规格)+content_web(图文);源码 goods-detail.vue 渲染确认。⚠️UI视觉未截图 |
| U-10 | 商品浏览 | 搜索/分类筛选 | 🟢 | 关键词搜索有结果;分类切换列表变化 | ✅ PASS | 接口实测:`goods/category` 返回分类树 code:0;`search/index`+`search/datalist` 关键词检索 code:0 有结果;源码 goods-category/goods-search 渲染确认。⚠️UI视觉未截图 |
| U-11 | 下单支付 | 用礼品券下单(券抵扣,免微信支付) | 🟢 | 选商品→结算→用券余额抵扣→无需微信支付即下单 | ⛔ BLOCKED | 源码:buy.vue plugins_points_data 积分/兑换区(170-215);`total_price>0` 才显示支付方式,全额抵扣(total_price=0)时不显示支付=免微信支付。buy/index 端点存活(-400)。需登录实测 |
| U-12 | 下单支付 | 收货信息填写(姓名/手机/地址) | 🟢 | 下单时可填/选收货信息,校验必填 | ⛔ BLOCKED | 源码:buy.vue 地址选择(name/tel/省市区/详址,18-36)+自提联系人(58/62),提交 address_id;新增地址走 useraddress/save。端点存活。需登录实测 |
| U-13 | 下单支付 | 订单生成(唯一单号,待发货) | 🟢 | 下单成功生成订单号,初始状态=待发货 | ⛔ BLOCKED | 源码:buy→ordergoodsform/goods/save 下单链路+component-payment;order/index 端点存活。订单号唯一性与初始状态需登录实测 |
| U-14 | 下单支付 | 订单列表(按状态分类) | 🟢 | 订单列表有待发货/已发货/已完成分类筛选 | ⛔ BLOCKED | 源码:user-order.vue nav_status_list 分类tab(全部/待付款1/待发货2/待收货3/已完成4/售后5,6),order/index 按 status 拉取;端点存活(-400)。需登录实测 |
| U-15 | 下单支付 | 订单详情(商品/物流/状态) | 🟢 | 详情展示商品、物流信息、当前状态 | ⛔ BLOCKED | 源码:user-order-detail.vue order/detail(561)展示商品/状态/操作(取消collect等);端点认证。需登录实测物流字段 |
| U-16 | 消息通知 | 订阅消息授权(下单后请求) | 🔴 | 微信订阅消息,小程序专有 | 🙋 MANUAL | 源码佐证:订阅授权基本未落地——全工程仅 exchange-success.vue 一处 requestSubscribeMessage 且 tmplIds:[] 为空,该页又无任何代码跳转(orphaned)。下单/支付主流程无订阅授权调用。见 F-002。微信弹窗需真机 |
| U-17 | 消息通知 | 发货提醒(服务通知) | 🔴 | 微信服务通知,小程序专有 | 🙋 MANUAL | 属后端发货时下发微信服务通知/订阅消息,前端无对应代码(符合架构)。需后端模板配置+真机核查,且前置依赖 U-16 订阅授权(当前未落地,见 F-002) |
| U-18 | 个人中心 | 我的订单 | 🟢 | 同 U-14/U-15 入口可达 | ⛔ BLOCKED | 源码:user.vue 订单入口(59 跳 user-order)+状态快捷(待付款/待发货/待收货/售后 142-145)。入口确认,数据同 U-14 需登录。备注:已兑换礼劵数恒0 见 F-003 |
| U-19 | 个人中心 | 我的礼品券(可用/已用/过期) | 🟢 | 三类券分类展示正确 | ⛔ BLOCKED | 源码:coupon/user.vue 三类tab not_use(可用)/already_use(已用)/already_expire(已过期)分类渲染确认;端点认证。需登录实测分类数据 |
| U-20 | 个人中心 | 收货地址管理(增/改/删) | 🟢 | 新增、编辑、删除地址全流程 | ⛔ BLOCKED | 源码:user-address.vue 全CRUD——列表index(173)/新增改save(377)/删除delete(343)/编辑(422);端点存活(-400)。需登录实测增改删 |
| U-21 | 个人中心 | 客服入口(电话/微信客服) | 🟡 | 入口存在;微信客服跳转🔴,电话拨号🟡 | ❌ FAIL | 个人中心客服菜单存在但 service_event 仅 toast 占位(user.vue:286-289 TODO),未接电话/微信客服/online-service。工程内有 online-service 组件(open-type=contact)+FAQ页客服回退,能力存在仅此入口未接。见 F-004 |
| U-22 | 个人中心 | 常见问题 FAQ | 🟢 | FAQ 帮助页可打开并有内容 | ✅ PASS | 接口实测:article/index code:0 返回6分类(帮助中心等);article/datalist id=7 code:0 返回Q&A(首条"如何搜索"含富文本content)。faq.vue 已建+入口已通(faq_event)+mp-html内联渲染答案。⚠️UI未截图 |

## 二、供货商端（独立小程序 — Playwright 不可直测，转人工 + 后台侧面验证）

| ID | 模块 | 功能点 | 可测性 | 验证方式 | 状态 | 证据/备注 |
|---|---|---|---|---|---|---|
| S-01 | 登录权限 | 手机号+验证码登录 | 🔴 | 人工:小程序登录;后台供货商管理确认账号存在 | 🙋 MANUAL | 供货商端=独立小程序,不在本工程源码;后台「供货商管理」核账需 admin 登录(本会话有验证码+无Playwright)。需人工 |
| S-02 | 登录权限 | 订阅消息授权(新订单/超时) | 🔴 | 人工核查 | 🙋 MANUAL | 微信订阅消息,供货商小程序专有,需真机人工 |
| S-03 | 登录权限 | 仅查看自己的订单 | 🟡 | 人工:小程序;后台「供货商订单统计」侧面佐证数据隔离 | 🙋 MANUAL | 数据隔离需后台「供货商订单统计」核查;本会话无法登录后台佐证。需人工 |
| S-04 | 订单处理 | 待发货订单列表(时间倒序) | 🟡 | 后台核查该供货商待发货订单数据是否正确分配 | 🙋 MANUAL | 需后台核订单分配;后端有 supplier_id 关联(goods.supplier_id 已见)但需登录核数据。需人工 |
| S-05 | 订单处理 | 订单详情(收货信息/商品明细) | 🟡 | 后台订单详情数据完整性佐证 | 🙋 MANUAL | 需后台订单详情佐证;本会话无法登录。需人工 |
| S-06 | 订单处理 | 确认发货(物流公司/单号,状态→已发货) | 🟡 | 人工:小程序发货;后台确认状态变更落库 | 🙋 MANUAL | 供货商小程序发货+后台核状态落库,本会话均不可达。需人工 |
| S-07 | 订单处理 | 已发货订单列表 | 🟡 | 后台已发货订单佐证 | 🙋 MANUAL | 需后台已发货订单佐证;本会话无法登录。需人工 |
| S-08 | 订单处理 | 订单搜索/筛选(单号/日期) | 🔴 | 人工核查 | 🙋 MANUAL | 供货商小程序专有,需人工 |
| S-09 | 消息通知 | 新订单提醒 | 🔴 | 人工核查;后台「推送记录」佐证是否发出 | 🙋 MANUAL | 需后台「推送记录」佐证;本会话无法登录后台。需人工 |
| S-10 | 消息通知 | 超时提醒 | 🔴 | 人工核查;后台推送记录佐证 | 🙋 MANUAL | 需后台推送记录佐证+发货时限配置(见 B-20);本会话无法登录。需人工 |
| S-11 | 个人设置 | 修改手机号 | 🔴 | 人工核查 | 🙋 MANUAL | 供货商小程序专有,需人工 |
| S-12 | 个人设置 | 重新授权消息 | 🔴 | 人工核查 | 🙋 MANUAL | 微信订阅消息重授权,供货商小程序专有,需人工 |

## 三、平台运营后台（PC — 全程 Playwright 可测）

| ID | 模块 | 功能点 | 可测性 | 测试要点 | 状态 | 证据/备注 |
|---|---|---|---|---|---|---|
| B-00 | 登录 | admin 登录(xxlysc) | 🟢 | 登录后进入仪表盘 | ⛔ BLOCKED | admin登录页含验证码(已实测页面有 verify/验证码字段);无Playwright无法读图登录。所有B用例前置受阻 |
| B-01 | 仪表盘 | 数据概览(今日订单/待发货/超时) | 🟢 | 仪表盘显示三项关键指标 | ⛔ BLOCKED | admin为独立PHP项目(源码不在本仓)+登录验证码+无Playwright,无法实测 |
| B-02 | 商品管理 | 商品列表(查/搜/编辑/上下架) | 🟢 | 列表可搜索、编辑、切换上下架 | ⛔ BLOCKED | 同上。侧证:用户端 search/datalist 实测有商品数据,商品模块存在 |
| B-03 | 商品管理 | 新增商品(名/价/库存/详情图/规格) | 🟢 | 新增 `E2E_` 商品,字段齐全可保存 | ⛔ BLOCKED | admin不可达,无法新增。需Playwright+读图登录 |
| B-04 | 商品管理 | 关联供货商(必选一个发货方) | 🟢 | 新增/编辑商品时必须选供货商,缺省应报错 | ⛔ BLOCKED | admin不可达。侧证:goods.supplier_id 字段存在(用户端 goods/detail 实测),供货商关联数据模型已有 |
| B-05 | 商品管理 | 商品分类管理(增/改) | 🟢 | 创建/编辑分类 | ⛔ BLOCKED | admin不可达。侧证:goods/category 实测返回分类树 |
| B-06 | 供货商管理 | 供货商列表(查/搜/启停) | 🟢 | 列表可搜索、启用/禁用 | ⛔ BLOCKED | admin不可达,无法实测 |
| B-07 | 供货商管理 | 新增供货商(名/联系人/手机/openid) | 🟢 | 新增 `E2E_` 供货商,含 openid 字段 | ⛔ BLOCKED | admin不可达,无法实测 |
| B-08 | 供货商管理 | 供货商订单统计(订单量/超时率) | 🟢 | 统计页展示每供货商订单量与超时率 | ⛔ BLOCKED | admin不可达,无法实测(S-03/04/07 数据隔离佐证亦依赖此页) |
| B-09 | 订单管理 | 全平台订单列表(按供货商/状态/时间筛选) | 🟢 | 三种筛选均生效 | ⛔ BLOCKED | admin不可达,无法实测 |
| B-10 | 订单管理 | 订单详情(完整信息) | 🟢 | 详情字段完整 | ⛔ BLOCKED | admin不可达,无法实测 |
| B-11 | 订单管理 | 超时订单高亮(超时限标红) | 🟢 | 超时订单视觉高亮/标红 | ⛔ BLOCKED | admin不可达,无法实测 |
| B-12 | 订单管理 | 导出订单报表(Excel) | 🟢 | 导出按钮生成 Excel 并可下载 | ⛔ BLOCKED | admin不可达,无法实测 |
| B-13 | 订单管理 | 手动干预(强制发货/取消,需权限) | 🟢 | 在 E2E_ 订单上测强制发货/取消;无权限应拦截 | ⛔ BLOCKED | admin不可达,无法实测 |
| B-14 | 卡券管理 | 卡券生成(批量唯一码+Excel导出/单张/批次) | 🟢 | 批量生成 `E2E_` 批次,码唯一,可导出Excel;单张生成 | ⛔ BLOCKED | admin不可达。侧证:giftcard 插件端点存活(用户端兑换走 giftcard/index/exchange),卡券模块存在 |
| B-15 | 卡券管理 | 卡券列表(全部/详情/导出) | 🟢 | 列表、详情、导出券数据 | ⛔ BLOCKED | admin不可达,无法实测 |
| B-16 | 卡券管理 | 卡券监控(使用统计/将过期提醒/批次报表) | 🟢 | 待确认:统计、过期提醒、批次报表是否存在(二期?) | ⛔ BLOCKED | admin不可达,无法确认是否存在(需登录后台核实是否二期功能) |
| B-17 | 卡券管理 | 卡券操作(作废/补发/延长有效期) | 🟢 | 在 E2E_ 券上测作废、补发、延长 | ⛔ BLOCKED | admin不可达,无法实测 |
| B-18 | 卡券管理 | 卡券日志(谁/何时/作废补发延长) | 🟢 | 操作后日志记录操作人、时间、动作、券 | ⛔ BLOCKED | admin不可达,无法实测 |
| B-19 | 消息记录 | 推送记录(用户/供货商推送历史) | 🟢 | 推送历史列表可查 | ⛔ BLOCKED | admin不可达(S-09/10 推送佐证依赖此页),无法实测 |
| B-20 | 系统设置 | 发货时限设置(全局超时,如24h) | 🟢 | 可配置并保存超时时长 | ⛔ BLOCKED | admin不可达,无法实测 |
| B-21 | 系统设置 | 运营人员账号(创建/禁用) | 🟢 | 创建 `E2E_` 运营账号并禁用 | ⛔ BLOCKED | admin不可达,无法实测 |
| B-22 | 系统设置 | 操作日志(发货干预/商品上下架等) | 🟢 | 关键操作在日志中可追溯 | ⛔ BLOCKED | admin不可达,无法实测 |

---

## 端到端贯通用例（跨端,优先在 H5+后台间打通一条主链路）
| ID | 场景 | 状态 | 证据/备注 |
|---|---|---|---|
| E2E-1 | 后台生成E2E券 → H5用gpj兑换 → 余额到账 → H5用券下单(待发货) → 后台订单列表可见且关联供货商 → 后台手动/供货商发货 → H5订单变已发货 | ⛔ BLOCKED | 横跨 admin(验证码登录,B-00 受阻)+ H5(验证码登录,U-02 受阻)两端,本会话两端皆不可登录,无法端到端实跑。各环节已分别源码佐证就绪:giftcard 兑换(U-04~07 端点存活)、buy/order 下单(U-11~13 端点存活)、supplier_id 关联存在;仅缺登录态串联实测 |

## 统计
- 总用例：22(用户端) + 12(供货商端) + 23(后台) + 1(贯通) = **58**
- 进度由 ralph-loop 自动回填,本节每轮可更新计数：**PASS 4 / FAIL 1 / BLOCKED 38 / MANUAL 15 / TODO 0（共58,全部非TODO,验证轮完成）**（iter11：E2E-1 BLOCKED）
