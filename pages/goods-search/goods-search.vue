<template>
    <view :class="theme_view" class="points-search">
        <!-- 顶部金色搜索区 -->
        <view class="gs-top">
            <view class="gs-search-bar flex-row align-c">
                <view class="gs-search-icon"></view>
                <input class="gs-search-input flex-1" type="text" confirm-type="search" v-model="wd" :placeholder="$t('index-points.index-points.search_placeholder')" placeholder-class="gs-search-ph" @confirm="search_event" />
                <view class="gs-search-btn tc" @tap="search_event">{{ $t('index-points.index-points.search_btn') }}</view>
            </view>
        </view>

        <view class="gs-body">
            <!-- 商品兑换网格（与首页同款卡片） -->
            <view v-if="goods_list_view.length > 0" class="goods-grid flex-row flex-wrap jc-sb">
                <view v-for="(item, index) in goods_list_view" :key="index" class="goods-card pr" :data-value="item.goods_url" @tap="goods_event">
                    <view class="goods-img-box pr">
                        <image class="goods-img dis-block wh-auto" :src="item.images" mode="aspectFill"></image>
                        <view v-if="is_sold_out(item)" class="sold-out-mask flex-row align-c jc-c">
                            <view class="sold-out-stamp tc">{{ $t('index-points.index-points.sold_out') }}</view>
                        </view>
                    </view>
                    <view class="goods-info">
                        <view class="goods-title multi-text">{{ item.title }}</view>
                        <view class="flex-row align-c jc-sb goods-bottom">
                            <view class="goods-points">
                                <text class="goods-points-label">{{ $t('index-points.index-points.points_colon') }}</text>
                                <text class="goods-points-value">{{ goods_points_view(item) }}</text>
                            </view>
                            <view class="exchange-btn tc" @tap.stop="exchange_event(item)">{{ $t('index-points.index-points.exchange') }}</view>
                        </view>
                    </view>
                </view>
            </view>

            <!-- 无数据提示 -->
            <component-no-data v-else :propStatus="data_list_loding_status" :propMsg="data_list_loding_msg || $t('index-points.index-points.search_empty')"></component-no-data>
        </view>

        <!-- 公共 -->
        <component-common ref="common" :propIsFooter="false"></component-common>
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
                params: {},
                // 搜索关键词（输入中）与已生效关键词
                wd: '',
                keyword: '',
                // 积分兑换商品全量列表（来自 points 插件 goodslist 接口，含未勾选首页显示的商品）
                goods_list: [],
                page: 1,
                limit: 20,
                has_more: true,
                loading: false,
                data_list_loding_status: 1,
                data_list_loding_msg: '',
                share_info: {},
            };
        },
        components: {
            componentCommon,
            componentNoData,
        },
        computed: {
            // 商品列表（关键词已由后端过滤）
            goods_list_view() {
                return this.goods_list || [];
            },
        },
        onLoad(params) {
            app.globalData.page_event_onload_handle(params);
            params = app.globalData.launch_params_handle(params);
            this.setData({
                params: params,
                wd: params.keywords || '',
                keyword: params.keywords || '',
            });
        },
        onShow() {
            app.globalData.page_event_onshow_handle();
            // #ifndef MP-ALIPAY
            uni.setNavigationBarTitle({ title: this.$t('index-points.index-points.search_title') });
            // #endif
            this.get_data();
            if ((this.$refs.common || null) != null) {
                this.$refs.common.on_show();
            }
        },
        onPullDownRefresh() {
            this.get_data(1);
        },
        // 触底加载下一页
        onReachBottom() {
            if (this.has_more && !this.loading) {
                this.get_data(this.page + 1);
            }
        },
        methods: {
            // 获取全量可兑换商品（points/goodslist，分页+后端关键词过滤，不受首页勾选限制）
            get_data(page) {
                if (this.loading) {
                    return;
                }
                page = page || 1;
                this.setData({ loading: true });
                uni.request({
                    url: app.globalData.get_request_url('goodslist', 'index', 'points'),
                    method: 'POST',
                    data: { page: page, limit: this.limit, wd: this.keyword || '' },
                    dataType: 'json',
                    success: (res) => {
                        uni.stopPullDownRefresh();
                        this.setData({ loading: false });
                        if (res.data.code == 0) {
                            var list = res.data.data || [];
                            this.setData({
                                goods_list: page <= 1 ? list : this.goods_list.concat(list),
                                page: page,
                                has_more: list.length >= this.limit,
                                data_list_loding_status: 3,
                                data_list_loding_msg: '',
                            });
                        } else {
                            this.setData({
                                data_list_loding_status: 2,
                                data_list_loding_msg: res.data.msg,
                            });
                        }
                        app.globalData.page_share_handle(this.share_info);
                    },
                    fail: () => {
                        uni.stopPullDownRefresh();
                        this.setData({
                            loading: false,
                            data_list_loding_status: 2,
                            data_list_loding_msg: this.$t('common.internet_error_tips'),
                        });
                    },
                });
            },

            // 搜索（关键词生效，回到第一页）
            search_event() {
                this.setData({ keyword: this.wd || '' });
                this.get_data(1);
            },

            // 兑换所需积分（后端字段 plugins_points_exchange_integral）
            goods_points_view(item) {
                return item.plugins_points_exchange_integral || 0;
            },

            // 是否已兑完
            is_sold_out(item) {
                if ((item.is_sold_out || 0) == 1) {
                    return true;
                }
                if (item.inventory != undefined && parseInt(item.inventory) <= 0) {
                    return true;
                }
                return false;
            },

            // 商品详情
            goods_event(e) {
                app.globalData.url_event(e);
            },

            // 立即兑换（进入商品详情）
            exchange_event(item) {
                if ((item.goods_url || null) != null) {
                    uni.navigateTo({ url: item.goods_url });
                }
            },
        },
    };
</script>
<style>
    @import './goods-search.css';
</style>
