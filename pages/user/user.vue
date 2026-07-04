<template>
    <view :class="theme_view" class="user-points">
        <!-- 顶部金色区：导航 + 用户信息 -->
        <view class="header">
            <view :style="'height:' + status_bar_height + 'px;'"></view>
            <view class="nav-title-bar tc">
                <text class="nav-title">{{ $t('index-points.index-points.member_center') }}</text>
            </view>
            <view class="user-info flex-row align-c jc-sb">
                <view class="flex-row align-c flex-1">
                    <image class="avatar circle dis-block" :src="avatar" mode="aspectFill" @tap="avatar_event"></image>
                    <view class="padding-left-main flex-1">
                        <view class="flex-row align-c">
                            <text class="user-name single-text" @tap="login_event">{{ nickname }}</text>
                            <view v-if="user != null" class="edit-icon" @tap="edit_profile_event"></view>
                        </view>
                        <view class="member-badge flex-row align-c">
                            <view class="member-badge-icon"></view>
                            <text>{{ $t('index-points.index-points.premium_member') }}</text>
                        </view>
                    </view>
                </view>
                <view class="sign-btn flex-row align-c jc-c" @tap="signin_event">
                    <view class="sign-icon"></view>
                    <text>{{ $t('index-points.index-points.sign_in') }}</text>
                </view>
            </view>
        </view>

        <view class="body">
            <!-- 积分卡 -->
            <view class="points-card">
                <view class="flex-row points-card-top">
                    <view class="flex-1">
                        <view class="pc-label">{{ $t('index-points.index-points.points_balance') }}</view>
                        <view class="pc-value">{{ user == null ? '✻✻✻✻✻✻' : integral_view }}</view>
                    </view>
                    <view class="flex-1">
                        <view class="pc-label">{{ $t('index-points.index-points.exchanged_coupons') }}</view>
                        <view class="pc-value"><text>{{ user == null ? '✻' : exchange_coupon_count }}</text><text v-if="user != null" class="pc-unit"> {{ $t('index-points.index-points.zhang_used') }}</text></view>
                    </view>
                </view>
                <view class="points-card-tabs flex-row align-c">
                    <view class="pc-tab flex-1 flex-row align-c jc-c" @tap="nav_event('/pages/user-integral/user-integral')">
                        <view class="pc-tab-icon pc-icon-detail"></view>
                        <text>{{ $t('index-points.index-points.points_detail') }}</text>
                    </view>
                    <view class="pc-tab-divider"></view>
                    <!-- TODO：兑换记录目标页待后端/产品确认（积分订单列表） -->
                    <view class="pc-tab flex-1 flex-row align-c jc-c" @tap="exchange_record_event">
                        <view class="pc-tab-icon pc-icon-record"></view>
                        <text>{{ $t('index-points.index-points.exchange_record') }}</text>
                    </view>
                </view>
            </view>

            <!-- 我的订单 -->
            <view class="order-card">
                <view class="order-header flex-row align-c jc-sb" @tap="nav_event('/pages/user-order/user-order')">
                    <text class="card-title">{{ $t('index-points.index-points.my_orders') }}</text>
                    <view class="more flex-row align-c">
                        <text>{{ $t('index-points.index-points.view_more') }}</text>
                        <text class="more-arrow">&gt;</text>
                    </view>
                </view>
                <view class="order-status flex-row">
                    <view v-for="(item, index) in order_status" :key="index" class="order-item flex-1 tc" @tap="nav_event(item.url)">
                        <view class="order-icon-box pr">
                            <view class="order-icon" :class="'order-icon-' + item.key"></view>
                            <view v-if="item.count > 0" class="order-badge pa">{{ item.count > 99 ? '99+' : item.count }}</view>
                        </view>
                        <view class="order-name">{{ item.name }}</view>
                    </view>
                </view>
            </view>

            <!-- 功能菜单 -->
            <view class="menu-card">
                <view class="menu-item flex-row align-c jc-sb" @tap="nav_event('/pages/user-address/user-address')">
                    <view class="flex-row align-c">
                        <view class="menu-icon menu-icon-address"></view>
                        <text class="menu-text">{{ $t('index-points.index-points.address_manage') }}</text>
                    </view>
                    <text class="menu-arrow">&gt;</text>
                </view>
                <view class="menu-item flex-row align-c jc-sb" @tap="service_event">
                    <view class="flex-row align-c">
                        <view class="menu-icon menu-icon-service"></view>
                        <text class="menu-text">{{ $t('index-points.index-points.service_entry') }}</text>
                    </view>
                    <text class="menu-arrow">&gt;</text>
                </view>
                <!-- 常见问题 → 帮助中心页 /pages/faq/faq -->
                <view class="menu-item flex-row align-c jc-sb" @tap="faq_event">
                    <view class="flex-row align-c">
                        <view class="menu-icon menu-icon-faq"></view>
                        <text class="menu-text">{{ $t('index-points.index-points.faq') }}</text>
                    </view>
                    <text class="menu-arrow">&gt;</text>
                </view>
            </view>
        </view>

        <!-- 自定义底部 Tab：商品浏览 / 礼包劵兑换 / 个人中心 -->
        <view class="custom-tabbar flex-row align-c">
            <view
                v-for="(item, index) in tabbar_list"
                :key="index"
                class="tabbar-item tc flex-1"
                :class="item.active ? 'tabbar-item-active' : ''"
                @tap="tabbar_event(item)"
            >
                <view class="tabbar-icon" :class="'tabbar-icon-' + item.icon + (item.active ? '-active' : '')"></view>
                <view class="tabbar-text">{{ item.name }}</view>
            </view>
        </view>
        <view class="custom-tabbar-seat"></view>

        <!-- 公共 -->
        <component-common ref="common" :propIsFooter="false"></component-common>
    </view>
</template>
<script>
    const app = getApp();
    import componentCommon from '@/components/common/common';
    export default {
        data() {
            return {
                theme_view: app.globalData.get_theme_value_view(),
                status_bar_height: parseInt(app.globalData.get_system_info('statusBarHeight', 20, true)),
                avatar_default: app.globalData.data.default_user_head_src,

                user: null,
                avatar: app.globalData.data.default_user_head_src,
                nickname: this.$t('index-points.index-points.click_login'),
                integral: 0,
                // TODO：已兑换礼劵数量字段待后端在 center/user 接口补充
                exchange_coupon_count: 0,

                // 订单状态（status：1待付款 2待发货 3待收货 101售后）
                order_status: [
                    { key: 'unpay', name: this.$t('index-points.index-points.order_unpay'), status: 1, count: 0, url: '/pages/user-order/user-order?status=1' },
                    { key: 'unsend', name: this.$t('index-points.index-points.order_unsend'), status: 2, count: 0, url: '/pages/user-order/user-order?status=2' },
                    { key: 'unreceive', name: this.$t('index-points.index-points.order_unreceive'), status: 3, count: 0, url: '/pages/user-order/user-order?status=3' },
                    { key: 'aftersale', name: this.$t('index-points.index-points.order_aftersale'), status: 101, count: 0, url: '/pages/user-orderaftersale/user-orderaftersale' },
                ],

                // 底部自定义 Tab
                tabbar_list: [
                    { name: this.$t('index-points.index-points.nav_goods'), icon: 'goods', active: false, url: '/pages/index/index' },
                    { name: this.$t('index-points.index-points.nav_gift'), icon: 'gift', active: false, url: '/pages/plugins/points/scan/scan' },
                    { name: this.$t('index-points.index-points.nav_user'), icon: 'user', active: true, url: '/pages/user/user' },
                ],
            };
        },
        components: {
            componentCommon,
        },
        computed: {
            integral_view() {
                return String(this.integral || 0).replace(/\B(?=(\d{3})+(?!\d))/g, ',');
            },
        },
        onShow() {
            app.globalData.page_event_onshow_handle();
            this.setData({ user: app.globalData.get_user_cache_info() });
            this.set_user_base();
            this.get_data();
            if ((this.$refs.common || null) != null) {
                this.$refs.common.on_show();
            }
        },
        onPullDownRefresh() {
            this.get_data();
        },
        onPageScroll(res) {
            uni.$emit('onPageScroll', res);
        },
        methods: {
            // 用户基础信息
            set_user_base() {
                if (this.user != null) {
                    this.setData({
                        avatar: this.user.avatar || this.avatar_default,
                        nickname: this.user.user_name_view || this.$t('index-points.index-points.click_login'),
                    });
                } else {
                    this.setData({ avatar: this.avatar_default, nickname: this.$t('index-points.index-points.click_login') });
                }
            },

            // 获取用户中心数据
            get_data() {
                uni.request({
                    url: app.globalData.get_request_url('center', 'user'),
                    method: 'POST',
                    data: {},
                    dataType: 'json',
                    success: (res) => {
                        uni.stopPullDownRefresh();
                        if (res.data.code == 0) {
                            var data = res.data.data;
                            // 订单各状态数量
                            var order_status = this.order_status;
                            if ((data.user_order_status || null) != null && data.user_order_status.length > 0) {
                                for (var i in order_status) {
                                    for (var t in data.user_order_status) {
                                        if (order_status[i].status == data.user_order_status[t].status) {
                                            order_status[i].count = data.user_order_status[t].count || 0;
                                            break;
                                        }
                                    }
                                }
                            }
                            var upd = {
                                integral: data.integral || 0,
                                order_status: order_status,
                                // 已兑换礼劵数量（后端 user/center 已返回 exchange_coupon_count）
                                exchange_coupon_count: data.exchange_coupon_count || 0,
                            };
                            if ((data.avatar || null) != null) {
                                upd.avatar = data.avatar;
                            }
                            if ((data.user_name_view || null) != null) {
                                upd.nickname = data.user_name_view;
                            }
                            this.setData(upd);
                        } else {
                            // 登录失效(-400)则引导重新登录，登录后回调本方法
                            app.globalData.is_login_check(res.data, this, 'get_data');
                        }
                    },
                    fail: () => {
                        uni.stopPullDownRefresh();
                    },
                });
            },

            // 通用跳转（需登录的页面会由目标页/全局拦截处理）
            nav_event(url) {
                if ((url || '') == '') {
                    return;
                }
                uni.navigateTo({ url: url });
            },

            // 头像
            avatar_event() {
                if (this.user == null) {
                    this.login_event();
                    return;
                }
                if (this.avatar_default != this.user.avatar && (this.user.avatar || null) != null) {
                    uni.previewImage({ current: this.user.avatar, urls: [this.user.avatar] });
                }
            },

            // 编辑资料
            // TODO：确认个人资料编辑页地址（如 /pages/personal/personal）
            edit_profile_event() {
                uni.navigateTo({ url: '/pages/personal/personal' });
            },

            // 登录
            login_event() {
                if (this.user != null) {
                    return;
                }
                this.setData({ user: app.globalData.get_user_info(this, 'login_event') || null });
                if (this.user != null) {
                    this.set_user_base();
                    this.get_data();
                }
            },

            // 签到（依赖后端 signin 插件，未安装时提示；安装后自动开放）
            signin_event() {
                if (app.globalData.get_config('plugins_base.signin', null) == null) {
                    uni.showToast({ title: this.$t('index-points.index-points.signin_not_open'), icon: 'none' });
                    return;
                }
                uni.navigateTo({ url: '/pages/plugins/signin/user/user' });
            },

            // 兑换记录
            // TODO：确认积分兑换记录页面（订单列表筛选）
            exchange_record_event() {
                uni.navigateTo({ url: '/pages/user-order/user-order' });
            },

            // 客服（优先级与 components/online-service 一致：客服系统 → 自定义客服 → 企业微信客服 → 电话客服）
            service_event() {
                var client = app.globalData.application_client_type();

                // 1) 在线客服系统
                var is_chat = app.globalData.get_config('plugins_base.chat.data.is_mobile_chat', 0);
                var chat_url = app.globalData.get_config('plugins_base.chat.data.chat_url', null);
                if (is_chat == 1 && (chat_url || null) != null) {
                    app.globalData.chat_entry_handle(chat_url);
                    return;
                }

                // 2) 自定义客服（按端取地址）
                var custom = app.globalData.get_config('config.common_app_customer_service_custom', null);
                var custom_url = custom == null ? null : custom[client] || null;
                if ((custom_url || null) != null) {
                    app.globalData.url_open(custom_url);
                    return;
                }

                // 3) 企业微信客服
                var weixin_corpid = app.globalData.get_config('config.common_app_customer_service_company_weixin_corpid', null);
                var weixin_url = app.globalData.get_config('config.common_app_customer_service_company_weixin_url', null);
                if ((weixin_corpid || null) != null && (weixin_url || null) != null) {
                    // #ifdef MP-WEIXIN
                    uni.openCustomerServiceChat({ extInfo: { url: weixin_url }, corpId: weixin_corpid });
                    return;
                    // #endif
                    // #ifdef APP
                    plus.share.getServices((res) => {
                        var wechat = res.find((i) => i.id === 'weixin');
                        if (wechat) {
                            wechat.openCustomerServiceChat({ corpid: weixin_corpid, url: weixin_url });
                        }
                    });
                    return;
                    // #endif
                    // #ifdef H5
                    app.globalData.url_open(weixin_url);
                    return;
                    // #endif
                }

                // 4) 电话客服
                var tel = app.globalData.get_config('config.common_app_customer_service_tel', null);
                if ((tel || null) != null) {
                    app.globalData.call_tel(tel);
                    return;
                }

                // 未配置任何客服
                uni.showToast({ title: this.$t('index-points.index-points.service_not_configured'), icon: 'none' });
            },

            // 常见问题
            faq_event() {
                uni.navigateTo({ url: '/pages/faq/faq' });
            },

            // 底部 Tab
            tabbar_event(item) {
                if (item.active) {
                    return;
                }
                // 首页/个人中心是 tabBar 页，用 switchTab；其它页（如礼包劵兑换）用 navigateTo
                if (item.url.indexOf('/pages/index/index') != -1 || item.url.indexOf('/pages/user/user') != -1) {
                    uni.switchTab({ url: item.url });
                    return;
                }
                uni.navigateTo({ url: item.url });
            },

            url_event(e) {
                app.globalData.url_event(e);
            },
        },
    };
</script>
<style>
    @import './user.css';
</style>
