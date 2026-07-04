<template>
    <view :class="theme_view" class="exchange-goods-page">
        <view class="eg-header tc">
            <view class="eg-title">我的兑换商品</view>
            <view class="eg-subtitle">礼品券兑换的实物商品，点击去下单领取</view>
        </view>

        <block v-if="data_list_loding_status == 3">
            <view v-if="goods_list.length > 0" class="eg-list">
                <view v-for="(item, index) in goods_list" :key="index" class="eg-card flex-row align-c" :data-value="item.goods_url" @tap="goods_event">
                    <image class="eg-img" :src="item.images" mode="aspectFill"></image>
                    <view class="eg-info flex-1">
                        <view class="eg-goods-title multi-text">{{ item.title }}</view>
                        <view class="eg-tag">礼品券兑换</view>
                    </view>
                    <view class="eg-btn tc" :data-value="item.goods_url" @tap.stop="goods_event">去下单</view>
                </view>
            </view>
            <view v-else class="eg-empty tc">
                <view class="eg-empty-text">暂无可领取的兑换商品</view>
                <view class="eg-empty-btn tc" @tap="go_back">去兑换</view>
            </view>
        </block>
        <block v-else>
            <component-no-data :propStatus="data_list_loding_status" :propMsg="data_list_loding_msg"></component-no-data>
        </block>

        <!-- 公共 -->
        <component-common ref="common"></component-common>
    </view>
</template>
<script>
    const app = getApp();
    import componentCommon from '@/components/common/common';
    import componentNoData from '@/components/no-data/no-data';
    export default {
        data() {
            return {
                theme_view: app.globalData.get_theme_value_view(),
                data_list_loding_status: 1,
                data_list_loding_msg: '',
                goods_list: [],
            };
        },

        components: {
            componentCommon,
            componentNoData,
        },

        onLoad(params) {
            // 调用公共事件方法
            app.globalData.page_event_onload_handle(params);
        },

        onShow() {
            // 调用公共事件方法
            app.globalData.page_event_onshow_handle();

            // 获取数据
            this.get_data();

            // 公共onshow事件
            if ((this.$refs.common || null) != null) {
                this.$refs.common.on_show();
            }
        },

        methods: {
            // 获取已兑换未领取的实物商品
            get_data() {
                var user = app.globalData.get_user_info(this, 'get_data');
                if (user == false) {
                    this.setData({
                        data_list_loding_status: 0,
                        data_list_loding_msg: '',
                    });
                    return;
                }
                uni.request({
                    url: app.globalData.get_request_url('exchangegoods', 'index', 'giftcard'),
                    method: 'POST',
                    data: {},
                    dataType: 'json',
                    success: (res) => {
                        if (res.data.code == 0) {
                            this.setData({
                                goods_list: res.data.data || [],
                                data_list_loding_status: 3,
                                data_list_loding_msg: '',
                            });
                        } else {
                            if (app.globalData.is_login_check(res.data, this, 'get_data')) {
                                this.setData({
                                    data_list_loding_status: 2,
                                    data_list_loding_msg: res.data.msg,
                                });
                            }
                        }
                    },
                    fail: () => {
                        this.setData({
                            data_list_loding_status: 2,
                            data_list_loding_msg: this.$t('common.internet_error_tips'),
                        });
                    },
                });
            },

            // 进入商品详情（下单时礼品卡权益后端自动抵扣）
            goods_event(e) {
                app.globalData.url_event(e);
            },

            // 返回
            go_back() {
                uni.navigateBack();
            },
        },
    };
</script>
<style scoped>
    .exchange-goods-page {
        min-height: 100vh;
        background: linear-gradient(180deg, #fdf3d9 0%, #f7f7f7 40%);
    }
    .eg-header {
        padding: 40rpx 30rpx 20rpx;
    }
    .eg-title {
        font-size: 40rpx;
        font-weight: bold;
        color: #a5772a;
    }
    .eg-subtitle {
        font-size: 24rpx;
        color: #b9902f;
        margin-top: 10rpx;
    }
    .eg-list {
        padding: 20rpx 24rpx;
    }
    .eg-card {
        background: #fff;
        border-radius: 20rpx;
        padding: 20rpx;
        margin-bottom: 20rpx;
    }
    .eg-img {
        width: 160rpx;
        height: 160rpx;
        border-radius: 16rpx;
        flex-shrink: 0;
    }
    .eg-info {
        padding: 0 20rpx;
        min-width: 0;
    }
    .eg-goods-title {
        font-size: 28rpx;
        color: #333;
        line-height: 1.4;
    }
    .eg-tag {
        display: inline-block;
        margin-top: 16rpx;
        font-size: 22rpx;
        color: #a5772a;
        background: #fdf0d0;
        padding: 4rpx 16rpx;
        border-radius: 8rpx;
    }
    .eg-btn {
        flex-shrink: 0;
        padding: 14rpx 28rpx;
        background: #be8f5b;
        color: #fff;
        font-size: 26rpx;
        border-radius: 40rpx;
    }
    .eg-empty {
        padding: 160rpx 40rpx;
    }
    .eg-empty-text {
        font-size: 28rpx;
        color: #999;
    }
    .eg-empty-btn {
        display: inline-block;
        margin-top: 40rpx;
        padding: 16rpx 60rpx;
        background: #be8f5b;
        color: #fff;
        border-radius: 40rpx;
        font-size: 28rpx;
    }
</style>
