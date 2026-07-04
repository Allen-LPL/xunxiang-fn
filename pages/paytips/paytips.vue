<template>
    <view :class="theme_view" class="paytips">
        <!-- 金色导航 -->
        <view class="pt-nav-bar">
            <view :style="'height:' + status_bar_height + 'px;'"></view>
            <view class="pt-nav-content flex-row align-c">
                <view class="pt-nav-back flex-row align-c" @tap="back_event">
                    <view class="pt-nav-arrow"></view>
                    <text class="pt-nav-back-text">{{ $t('common.return') }}</text>
                </view>
                <text class="pt-nav-title">支付结果</text>
            </view>
        </view>

        <view class="pt-body tc">
            <!-- 结果图标 -->
            <view v-if="params.code == '9000'" class="pt-icon pt-icon-success">
                <view class="pt-icon-check"></view>
            </view>
            <view v-else class="pt-icon pt-icon-fail">
                <view class="pt-icon-cross"></view>
            </view>

            <!-- 结果文案 -->
            <view class="pt-title">{{ params.msg || $t('paytips.paytips.679rxu') }}</view>

            <!-- 按钮：主操作（跳转）+ 返回 -->
            <button class="pt-btn pt-btn-primary" type="default" hover-class="none" data-redirect="1" :data-value="default_to_url" @tap="url_event">{{ params.title || $t('paytips.paytips.jifuu8') }}</button>
            <button class="pt-btn pt-btn-default" type="default" hover-class="none" @tap="back_event">{{ $t('common.return') }}</button>
        </view>

        <!-- 公共 -->
        <component-common ref="common" :propIsFooter="false"></component-common>
    </view>
</template>
<script>
    const app = getApp();
    import base64 from '@/common/js/lib/base64.js';
    import componentCommon from '@/components/common/common';
    export default {
        data() {
            return {
                theme_view: app.globalData.get_theme_value_view(),
                status_bar_height: parseInt(app.globalData.get_system_info('statusBarHeight', 20, true)),
                params: {},
                default_round_success_icon: app.globalData.data.default_round_success_icon,
                default_round_error_icon: app.globalData.data.default_round_error_icon,
                default_to_url: '',
            };
        },
        components: {
            componentCommon
        },

        /**
         * 页面加载初始化
         */
        onLoad(params) {
            // 参数处理
            if((params || null) != null) {
                params = JSON.parse(base64.decode(decodeURIComponent(params.params)));
            }

            // 调用公共事件方法
            app.globalData.page_event_onload_handle(params);

            // 根据状态处理
            var msg = null;
            switch (params.code) {
                // 支付成功
                case '9000':
                    msg = this.$t('paytips.paytips.679rxu');
                    break;
                // 正在处理中
                case '8000':
                    msg = this.$t('paytips.paytips.d8m853');
                    break;
                // 支付失败
                case '4000':
                    msg = this.$t('paytips.paytips.6y488i');
                    break;
                // 用户中途取消
                case '6001':
                    msg = this.$t('paytips.paytips.e732we');
                    break;
                // 网络连接出错
                case '6002':
                    msg = this.$t('paytips.paytips.13v11t');
                    break;
                // 支付结果未知（有可能已经支付成功），请查询商户订单列表中订单的支付状态
                case '6004':
                    msg = this.$t('paytips.paytips.u1153p');
                    break;
                // 用户点击忘记密码导致快捷界面退出(only iOS)
                case '99':
                    msg = this.$t('paytips.paytips.6mpsl7');
                    break;
                // 默认错误
                default:
                    msg = this.$t('paytips.paytips.59u769');
            }
            params['msg'] = msg;
            this.setData({
                params: params,
                default_to_url: params.page || app.globalData.app_tabbar_pages()[0],
            });
        },

        onShow() {
            // 调用公共事件方法
            app.globalData.page_event_onshow_handle();

            // 公共onshow事件
            if ((this.$refs.common || null) != null) {
                this.$refs.common.on_show();
            }

            // 分享菜单处理
            app.globalData.page_share_handle();
        },

        methods: {
            // 返回
            back_event(e) {
                app.globalData.page_back_prev_event();
            },
            // url事件
            url_event(e) {
                app.globalData.url_event(e);
            },
        },
    };
</script>
<style>
    @import './paytips.css';
</style>
