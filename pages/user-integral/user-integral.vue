<template>
    <view :class="theme_view" class="integral-points">
        <!-- 自定义金色导航 -->
        <view class="nav-bar">
            <view :style="'height:' + status_bar_height + 'px;'"></view>
            <view class="nav-content flex-row align-c">
                <view class="nav-back flex-row align-c" @tap="back_event">
                    <view class="nav-back-arrow"></view>
                    <text class="nav-back-text">{{ $t('index-points.index-points.back') }}</text>
                </view>
                <text class="nav-title">{{ $t('index-points.index-points.points_detail') }}</text>
            </view>
        </view>

        <scroll-view :scroll-y="true" class="list-scroll" @scrolltolower="scroll_lower" lower-threshold="60">
            <view v-if="data_list.length > 0" class="list-card">
                <view v-for="(item, index) in data_list" :key="index" class="list-item flex-row jc-sb align-c">
                    <view class="item-left flex-1">
                        <view class="item-msg single-text">{{ item.msg }}</view>
                        <view class="item-time">{{ item.add_time_time }}</view>
                    </view>
                    <view class="item-value" :class="item.type == 1 ? 'is-plus' : 'is-minus'">{{ item.type == 1 ? '+' : '-' }}{{ item.operation_integral }}</view>
                </view>
            </view>
            <component-no-data v-else :propStatus="data_list_loding_status"></component-no-data>

            <!-- 结尾 -->
            <component-bottom-line :propStatus="data_bottom_line_status"></component-bottom-line>
        </scroll-view>

        <!-- 公共 -->
        <component-common ref="common" :propIsFooter="false"></component-common>
    </view>
</template>
<script>
    const app = getApp();
    import componentCommon from '@/components/common/common';
    import componentNoData from '@/components/no-data/no-data';
    import componentBottomLine from '@/components/bottom-line/bottom-line';
    export default {
        data() {
            return {
                theme_view: app.globalData.get_theme_value_view(),
                status_bar_height: parseInt(app.globalData.get_system_info('statusBarHeight', 20, true)),
                data_list: [],
                data_total: 0,
                data_page_total: 0,
                data_page: 1,
                data_is_loading: 0,
                data_list_loding_status: 1,
                data_bottom_line_status: false,
            };
        },
        components: {
            componentCommon,
            componentNoData,
            componentBottomLine,
        },
        onLoad(params) {
            app.globalData.page_event_onload_handle(params);
        },
        onShow() {
            app.globalData.page_event_onshow_handle();
            this.init();
            if ((this.$refs.common || null) != null) {
                this.$refs.common.on_show();
            }
            app.globalData.page_share_handle();
        },
        onPullDownRefresh() {
            this.setData({ data_page: 1 });
            this.get_data_list(1);
        },
        onPageScroll(res) {
            uni.$emit('onPageScroll', res);
        },
        methods: {
            // 返回
            back_event() {
                app.globalData.page_back_prev_event();
            },

            init() {
                var user = app.globalData.get_user_info(this, 'init');
                if (user != false) {
                    this.get_data_list();
                } else {
                    this.setData({ data_list_loding_status: 0, data_bottom_line_status: false });
                }
            },

            // 获取积分明细（复用 index/userintegral 接口，分页）
            get_data_list(is_mandatory) {
                if ((is_mandatory || 0) == 0) {
                    if (this.data_bottom_line_status == true) {
                        uni.stopPullDownRefresh();
                        return false;
                    }
                }
                if (this.data_is_loading == 1) {
                    return false;
                }
                this.setData({ data_is_loading: 1, data_list_loding_status: 1 });
                if (this.data_page > 1) {
                    uni.showLoading({ title: this.$t('common.loading_in_text') });
                }
                uni.request({
                    url: app.globalData.get_request_url('index', 'userintegral'),
                    method: 'POST',
                    data: { page: this.data_page },
                    dataType: 'json',
                    success: (res) => {
                        if (this.data_page > 1) {
                            uni.hideLoading();
                        }
                        uni.stopPullDownRefresh();
                        if (res.data.code == 0) {
                            var data = res.data.data;
                            if (data.data.length > 0) {
                                var temp_data_list = this.data_page <= 1 ? data.data : (this.data_list || []).concat(data.data);
                                this.setData({
                                    data_list: temp_data_list,
                                    data_total: data.total,
                                    data_page_total: data.page_total,
                                    data_list_loding_status: 3,
                                    data_page: this.data_page + 1,
                                    data_is_loading: 0,
                                });
                                this.setData({
                                    data_bottom_line_status: this.data_list.length > 0 && this.data_page > 1 && this.data_page > this.data_page_total,
                                });
                            } else {
                                this.setData({ data_list_loding_status: 0, data_is_loading: 0 });
                                if (this.data_page <= 1) {
                                    this.setData({ data_list: [], data_bottom_line_status: false });
                                }
                            }
                        } else {
                            this.setData({ data_list_loding_status: 0, data_is_loading: 0 });
                            if (app.globalData.is_login_check(res.data, this, 'get_data_list')) {
                                app.globalData.showToast(res.data.msg);
                            }
                        }
                    },
                    fail: () => {
                        if (this.data_page > 1) {
                            uni.hideLoading();
                        }
                        uni.stopPullDownRefresh();
                        this.setData({ data_list_loding_status: 2, data_is_loading: 0 });
                        app.globalData.showToast(this.$t('common.internet_error_tips'));
                    },
                });
            },

            scroll_lower(e) {
                this.get_data_list();
            },
        },
    };
</script>
<style>
    @import './user-integral.css';
</style>
