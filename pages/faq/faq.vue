<template>
    <view :class="theme_view" class="faq">
        <!-- 金色导航 -->
        <view class="faq-nav-bar">
            <view :style="'height:' + status_bar_height + 'px;'"></view>
            <view class="faq-nav-content flex-row align-c">
                <view class="faq-nav-back flex-row align-c" @tap="back_event">
                    <view class="faq-nav-arrow"></view>
                    <text class="faq-nav-back-text">{{ $t('common.return') }}</text>
                </view>
                <text class="faq-nav-title">常见问题</text>
            </view>
        </view>

        <!-- 头图 -->
        <view class="faq-hero">
            <view class="faq-hero-main">
                <text class="faq-hero-faq">FAQ</text>
                <text class="faq-hero-title">帮助中心</text>
            </view>
            <text class="faq-hero-sub">积分兑换商城常见问题</text>
            <view class="faq-hero-deco">
                <iconfont name="icon-gift" size="120rpx" color="#e9cd86"></iconfont>
            </view>
        </view>

        <!-- 分类卡 -->
        <view v-if="(category_cards || null) != null && category_cards.length > 0" class="faq-cards">
            <view
                v-for="(item, index) in category_cards"
                :key="item.id"
                :class="'faq-card flex-row align-c ' + (active_category == item.id ? 'faq-card-active' : '')"
                @tap="card_event(item)"
            >
                <view :class="'faq-card-icon faq-card-icon-' + (index % 4)">
                    <iconfont :name="card_icons[index % 4]" size="40rpx" color="#ffffff"></iconfont>
                </view>
                <view class="faq-card-text flex-1">
                    <view class="faq-card-title">{{ item.name }}</view>
                    <view class="faq-card-sub">查看相关问题</view>
                </view>
                <view class="faq-card-arrow"></view>
            </view>
        </view>

        <!-- Q&A 列表 -->
        <view class="faq-panel">
            <!-- 搜索 -->
            <view class="faq-search flex-row align-c">
                <iconfont name="icon-search" size="32rpx" color="#bbbbbb"></iconfont>
                <input class="faq-search-input flex-1" type="text" v-model="keyword" placeholder="搜索你遇到的问题" placeholder-class="faq-search-ph" confirm-type="search" />
            </view>

            <block v-if="(filtered_list || null) != null && filtered_list.length > 0">
                <view v-for="(item, index) in filtered_list" :key="item.id" class="faq-qa">
                    <view class="faq-qa-head flex-row align-c" @tap="toggle_event(item.id)">
                        <text class="faq-qa-q">Q</text>
                        <text class="faq-qa-title flex-1">{{ item.title }}</text>
                        <view :class="'faq-qa-chevron ' + (expanded_id == item.id ? 'faq-qa-chevron-up' : '')"></view>
                    </view>
                    <view v-if="expanded_id == item.id" class="faq-qa-body">
                        <mp-html v-if="(item.content || '') != ''" :content="item.content"></mp-html>
                        <view v-else class="faq-qa-text">{{ item.describe || '暂无内容' }}</view>
                    </view>
                </view>
            </block>
            <view v-else class="faq-empty">{{ list_loading ? '加载中…' : (keyword != '' ? '没有匹配的问题' : '暂无常见问题') }}</view>
        </view>

        <!-- 底部按钮占位 -->
        <view class="faq-service-seat"></view>

        <!-- 联系在线客服 -->
        <view class="faq-service">
            <!-- #ifdef MP-WEIXIN || MP-TOUTIAO || MP-BAIDU || MP-KUAISHOU -->
            <button class="faq-service-btn" open-type="contact" hover-class="none" type="default">
                <view class="faq-service-inner flex-row align-c jc-c">
                    <iconfont name="icon-customer-service" size="38rpx" color="#ffffff"></iconfont>
                    <text class="faq-service-text">联系在线客服</text>
                </view>
            </button>
            <!-- #endif -->
            <!-- #ifndef MP-WEIXIN || MP-TOUTIAO || MP-BAIDU || MP-KUAISHOU -->
            <view class="faq-service-btn flex-row align-c jc-c" hover-class="faq-service-hover" @tap="service_event">
                <iconfont name="icon-customer-service" size="38rpx" color="#ffffff"></iconfont>
                <text class="faq-service-text">联系在线客服</text>
            </view>
            <!-- #endif -->
        </view>

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
                params: null,
                // 文章分类（用于分类卡）
                category_list: [],
                category_cards: [],
                active_category: 0,
                // 分类卡图标（按索引循环）
                card_icons: ['icon-points', 'icon-shopping-bag', 'icon-delivery', 'icon-after-sales'],
                // Q&A 列表（article/datalist，列表项自带 content 富文本）
                data_list: [],
                list_loading: false,
                // 当前展开的问题 id（手风琴，单开）
                expanded_id: 0,
                // 搜索关键词
                keyword: '',
            };
        },

        components: {
            componentCommon,
        },

        computed: {
            // 按关键词过滤标题
            filtered_list() {
                var list = this.data_list || [];
                var kw = (this.keyword || '').trim();
                if (kw == '') {
                    return list;
                }
                return list.filter((item) => (item.title || '').indexOf(kw) != -1);
            },
        },

        onLoad(params) {
            app.globalData.page_event_onload_handle(params);
            this.setData({ params: params || {} });
            this.get_categories();
        },

        onShow() {
            app.globalData.page_event_onshow_handle();
            if ((this.$refs.common || null) != null) {
                this.$refs.common.on_show();
            }
            app.globalData.page_share_handle();
        },

        methods: {
            // 获取文章分类，取前 4 个作为分类卡，默认加载第一个分类的问答
            get_categories() {
                uni.request({
                    url: app.globalData.get_request_url('index', 'article'),
                    method: 'POST',
                    data: {},
                    dataType: 'json',
                    success: (res) => {
                        if (res.data.code == 0) {
                            var list = res.data.data.category_list || [];
                            // 参数指定分类则优先，否则取第一个分类
                            var active = this.params.id || (list.length > 0 ? list[0].id : 0);
                            this.setData({
                                category_list: list,
                                category_cards: list.slice(0, 4),
                                active_category: active,
                            });
                            this.get_list(active);
                        } else {
                            app.globalData.showToast(res.data.msg);
                        }
                    },
                    fail: () => {
                        app.globalData.showToast(this.$t('common.internet_error_tips'));
                    },
                });
            },

            // 获取某分类下的问答列表（列表项自带 content，可内联展开）
            get_list(category_id) {
                this.setData({
                    list_loading: true,
                    data_list: [],
                    expanded_id: 0,
                });
                uni.request({
                    url: app.globalData.get_request_url('datalist', 'article'),
                    method: 'POST',
                    data: {
                        page: 1,
                        id: category_id || 0,
                    },
                    dataType: 'json',
                    success: (res) => {
                        if (res.data.code == 0) {
                            this.setData({
                                data_list: res.data.data.data || [],
                                list_loading: false,
                            });
                        } else {
                            this.setData({ list_loading: false });
                            app.globalData.showToast(res.data.msg);
                        }
                    },
                    fail: () => {
                        this.setData({ list_loading: false });
                        app.globalData.showToast(this.$t('common.internet_error_tips'));
                    },
                });
            },

            // 点击分类卡：切换分类并加载其问答
            card_event(item) {
                if (this.active_category == item.id) {
                    return;
                }
                this.setData({
                    active_category: item.id,
                    keyword: '',
                });
                this.get_list(item.id);
            },

            // 展开/收起答案（手风琴）
            toggle_event(id) {
                this.setData({
                    expanded_id: this.expanded_id == id ? 0 : id,
                });
            },

            // 返回
            back_event() {
                app.globalData.page_back_prev_event();
            },

            // 联系在线客服（与个人中心一致：客服系统 → 自定义客服 → 企业微信客服 → 电话客服；MP 端用原生 open-type=contact）
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
        },
    };
</script>
<style>
    @import './faq.css';
</style>
