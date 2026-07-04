<template>
    <view :class="theme_view" class="goods-detail-points">
        <!-- 自定义导航 -->
        <view class="nav-bar">
            <view :style="'height:' + status_bar_height + 'px;'"></view>
            <view class="nav-content flex-row align-c">
                <view class="nav-back flex-row align-c" @tap="back_event">
                    <view class="nav-back-arrow"></view>
                    <text class="nav-back-text">{{ $t('index-points.index-points.back') }}</text>
                </view>
                <text class="nav-title single-text">{{ (goods && goods.title) || '' }}</text>
            </view>
        </view>

        <block v-if="(goods || null) != null">
            <!-- 主图轮播 -->
            <swiper
                class="main-swiper"
                :indicator-dots="goods_photo.length > 1"
                indicator-color="rgba(255,255,255,0.5)"
                indicator-active-color="#ffffff"
                :autoplay="goods_photo.length > 1"
                :circular="true"
            >
                <swiper-item v-for="(item, index) in goods_photo" :key="index">
                    <image class="main-img dis-block" :src="item.images" mode="aspectFill" :data-index="index" @tap="photo_view_event"></image>
                </swiper-item>
            </swiper>

            <!-- 商品信息卡 -->
            <view class="info-card flex-row jc-sb">
                <view class="info-left flex-1">
                    <view class="goods-title multi-text">{{ goods.title }}</view>
                    <view v-if="(goods.simple_desc || null) != null" class="goods-desc single-text">{{ goods.simple_desc }}</view>
                    <view class="goods-stock">{{ $t('index-points.index-points.remaining') }} {{ goods.inventory }} {{ $t('index-points.index-points.piece') }}</view>
                </view>
                <view class="info-right tr">
                    <text class="points-label">{{ $t('index-points.index-points.points_colon') }}</text>
                    <text class="points-value">{{ goods_points_view }}</text>
                </view>
            </view>

            <!-- 商品详情 -->
            <view class="detail-content">
                <view class="detail-title">{{ $t('index-points.index-points.detail_title') }}</view>
                <!-- 移动端图文详情 -->
                <block v-if="goods_content_app.length > 0">
                    <view v-for="(item, index) in goods_content_app" :key="index" class="detail-item">
                        <image v-if="(item.images || null) != null" class="wh-auto dis-block" :src="item.images" mode="widthFix" :data-value="item.images" @tap="detail_image_view_event"></image>
                        <view v-if="(item.text || null) != null" class="padding-main">{{ item.text }}</view>
                    </view>
                </block>
                <!-- PC 富文本详情 -->
                <view v-else-if="(goods.content_web || null) != null" class="padding-main web-html-content">
                    <mp-html :content="goods.content_web" />
                </view>
            </view>

            <!-- 结尾 -->
            <component-bottom-line :propStatus="data_bottom_line_status"></component-bottom-line>
            <!-- 底部操作栏占位 -->
            <view class="bottom-bar-seat"></view>

            <!-- 底部操作栏 -->
            <view class="bottom-bar flex-row align-c">
                <view class="bottom-icon tc" @tap="service_event">
                    <view class="bottom-icon-img bottom-icon-service"></view>
                    <view class="bottom-icon-text">{{ $t('index-points.index-points.service') }}</view>
                </view>
                <view class="bottom-icon tc" @tap="home_event">
                    <view class="bottom-icon-img bottom-icon-home"></view>
                    <view class="bottom-icon-text">{{ $t('index-points.index-points.home') }}</view>
                </view>
                <view class="exchange-btn tc flex-1" @tap="exchange_event">{{ $t('index-points.index-points.exchange_now') }}</view>
            </view>
        </block>
        <block v-else>
            <component-no-data :propStatus="data_list_loding_status" :propMsg="data_list_loding_msg"></component-no-data>
        </block>

        <!-- 立即兑换（复用通用购买组件，弹出规格/数量并下单） -->
        <component-goods-buy
            ref="goods_buy"
            :propParams="params"
            :propCurrencySymbol="currency_symbol"
            v-on:BackReleaseEvent="goods_buy_back_release_event"
            v-on:BackSuccessEvent="goods_buy_back_success_event"
            v-on:CartSuccessEvent="goods_cart_back_event"
            v-on:SpecChoiceEvent="goods_spec_back_event"
        ></component-goods-buy>

        <!-- 公共 -->
        <component-common ref="common" :propIsFooter="false"></component-common>
    </view>
</template>
<script>
    const app = getApp();
    import componentCommon from '@/components/common/common';
    import componentNoData from '@/components/no-data/no-data';
    import componentBottomLine from '@/components/bottom-line/bottom-line';
    import componentGoodsBuy from '@/components/goods-buy/goods-buy';
    export default {
        data() {
            return {
                theme_view: app.globalData.get_theme_value_view(),
                status_bar_height: parseInt(app.globalData.get_system_info('statusBarHeight', 20, true)),
                currency_symbol: app.globalData.currency_symbol(),

                params: {},
                goods: null,
                goods_photo: [],
                goods_content_app: [],
                buy_button: null,
                // 购买事件类型（buy 立即购买/兑换）
                buy_event_type: 'buy',

                data_bottom_line_status: false,
                data_list_loding_status: 1,
                data_list_loding_msg: '',

                share_info: {},
            };
        },
        components: {
            componentCommon,
            componentNoData,
            componentBottomLine,
            componentGoodsBuy,
        },
        computed: {
            // 单品兑换所需积分（后端字段 plugins_points_exchange_integral）
            goods_points_view() {
                if (this.goods == null) {
                    return 0;
                }
                return this.goods.plugins_points_exchange_integral || 0;
            },
        },
        onLoad(params) {
            app.globalData.page_event_onload_handle(params);
            this.setData({ params: params || {} });
        },
        onShow() {
            app.globalData.page_event_onshow_handle();
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
            // 获取商品详情数据（复用通用商品详情接口）
            get_data() {
                uni.request({
                    url: app.globalData.get_request_url('detail', 'goods'),
                    method: 'POST',
                    data: this.params,
                    dataType: 'json',
                    success: (res) => {
                        uni.stopPullDownRefresh();
                        if (res.data.code == 0) {
                            var data = res.data.data;
                            var goods = data.goods;
                            this.init_result_data_handle(goods);
                            this.setData({
                                buy_button: data.buy_button || null,
                                data_bottom_line_status: true,
                                data_list_loding_status: 3,
                            });

                            // 分享配置
                            this.setData({
                                share_info: {
                                    title: goods.seo_title || goods.title,
                                    desc: goods.seo_desc || goods.simple_desc,
                                    path: '/pages/goods-detail/goods-detail?id=' + goods.id,
                                    img: goods.share_images || goods.images,
                                },
                            });

                            // #ifndef MP-ALIPAY
                            uni.setNavigationBarTitle({ title: goods.title || '' });
                            // #endif
                        } else {
                            this.setData({
                                data_bottom_line_status: false,
                                data_list_loding_status: 2,
                                data_list_loding_msg: res.data.msg,
                            });
                        }
                        app.globalData.page_share_handle(this.share_info);
                    },
                    fail: () => {
                        uni.stopPullDownRefresh();
                        this.setData({
                            data_bottom_line_status: false,
                            data_list_loding_status: 2,
                            data_list_loding_msg: this.$t('common.internet_error_tips'),
                        });
                    },
                });
            },

            // 商品数据处理（相册、详情）
            init_result_data_handle(goods) {
                var photo = goods.photo || [];
                if (photo.length == 0 && (goods.images || null) != null) {
                    photo.push({ images: goods.images });
                }
                this.setData({
                    goods: goods,
                    goods_photo: photo,
                    goods_content_app: goods.content_app || [],
                });
            },

            // 立即兑换（库存>0 时弹出规格/数量确认）
            exchange_event() {
                if (this.goods == null) {
                    return;
                }
                if (parseInt(this.goods.inventory || 0) <= 0) {
                    uni.showToast({ title: this.$t('index-points.index-points.sold_out'), icon: 'none' });
                    return;
                }
                if ((this.$refs.goods_buy || null) != null) {
                    // 积分商城无购物车：弹层按钮去掉加购，购买按钮文案统一为“立即兑换”
                    var buy_button = this.buy_button;
                    if (buy_button != null && (buy_button.data || null) != null) {
                        buy_button = { ...buy_button, data: buy_button.data.filter((v) => v.type != 'cart').map((v) => (v.type == 'buy' ? { ...v, name: this.$t('index-points.index-points.exchange_now') } : v)) };
                    }
                    this.$refs.goods_buy.init(this.goods, { ...{ buy_event_type: this.buy_event_type, buy_button: buy_button }, ...this.params });
                }
            },

            // 购买组件回调（积分兑换走通用下单链路，详情页无需更新价格，留空即可）
            goods_buy_back_release_event(e) {},
            goods_buy_back_success_event(e) {},
            goods_cart_back_event(e) {},
            goods_spec_back_event(e) {},

            // 主图预览
            photo_view_event(e) {
                var index = parseInt(e.currentTarget.dataset.index || 0);
                var urls = this.goods_photo.map(function (v) {
                    return v.images;
                });
                if (urls.length > 0) {
                    uni.previewImage({ current: urls[index], urls: urls });
                }
            },

            // 详情图预览
            detail_image_view_event(e) {
                var url = e.currentTarget.dataset.value;
                if ((url || null) != null) {
                    uni.previewImage({ current: url, urls: [url] });
                }
            },

            // 返回
            back_event() {
                app.globalData.page_back_prev_event();
            },

            // 客服
            // TODO：接入项目在线客服（online-service 组件 / 后台客服配置）
            service_event() {
                uni.showToast({ title: this.$t('index-points.index-points.service'), icon: 'none' });
            },

            // 首页
            home_event() {
                uni.switchTab({ url: '/pages/index/index' });
            },

            url_event(e) {
                app.globalData.url_event(e);
            },
        },
    };
</script>
<style>
    @import './goods-detail.css';
</style>
