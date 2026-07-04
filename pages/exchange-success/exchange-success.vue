<template>
    <view :class="theme_view" class="exch-success">
        <!-- 金色导航 -->
        <view class="es-nav-bar">
            <view :style="'height:' + status_bar_height + 'px;'"></view>
            <view class="es-nav-content flex-row align-c">
                <view class="es-nav-back flex-row align-c" @tap="back_event">
                    <view class="es-nav-arrow"></view>
                    <text class="es-nav-back-text">{{ $t('index-points.index-points.back') }}</text>
                </view>
                <text class="es-nav-title">{{ $t('index-points.index-points.exch_success_title') }}</text>
            </view>
        </view>

        <view class="es-body tc">
            <view class="es-check">
                <view class="es-check-mark"></view>
            </view>
            <view class="es-title">{{ $t('index-points.index-points.exch_success_title') }}</view>
            <view class="es-tips">{{ $t('index-points.index-points.notify_tips') }}</view>
            <button class="es-btn es-btn-yes" type="default" hover-class="none" @tap="auth_yes_event">{{ $t('index-points.index-points.btn_yes') }}</button>
            <button class="es-btn es-btn-no" type="default" hover-class="none" @tap="auth_no_event">{{ $t('index-points.index-points.btn_no') }}</button>
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
                params: {},
            };
        },
        components: {
            componentCommon,
        },
        onLoad(params) {
            app.globalData.page_event_onload_handle(params);
            this.setData({ params: params || {} });
        },
        onShow() {
            app.globalData.page_event_onshow_handle();
            if ((this.$refs.common || null) != null) {
                this.$refs.common.on_show();
            }
        },
        methods: {
            // 返回
            back_event() {
                app.globalData.page_back_prev_event();
            },

            // 是 —— 授权接收物流状态通知（微信订阅消息）
            // TODO：填入后端配置的「物流状态通知」订阅消息模板ID（tmplIds），可由接口下发
            auth_yes_event() {
                // #ifdef MP-WEIXIN
                uni.requestSubscribeMessage({
                    tmplIds: [],
                    complete: () => {
                        this.go_order();
                    },
                });
                // #endif
                // #ifndef MP-WEIXIN
                this.go_order();
                // #endif
            },

            // 否 —— 不授权，直接进入订单
            auth_no_event() {
                this.go_order();
            },

            // 跳转到我的订单
            go_order() {
                uni.redirectTo({ url: '/pages/user-order/user-order' });
            },
        },
    };
</script>
<style>
    @import './exchange-success.css';
</style>
