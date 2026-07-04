<template>
    <view :class="theme_view" class="points-home">
        <!-- 顶部金色背景区（状态栏 + 自定义导航 + 周年庆 Banner） -->
        <view class="top-banner pr">
            <!-- 状态栏占位 + 自定义导航高度 -->
            <view :style="'height:' + status_bar_height + 'px;'"></view>
            <!-- 周年庆 Banner 图（后端配置 home_banner_images） -->
            <image
                v-if="(default_images_data.home_banner_images || null) != null"
                class="banner-img dis-block wh-auto"
                :src="default_images_data.home_banner_images"
                mode="widthFix"
                :data-value="(data_base && data_base.home_banner_images_url) || ''"
                @tap="url_event"
            ></image>
        </view>

        <!-- 主体内容上浮覆盖在 Banner 之上 -->
        <view class="main-content pr">
            <!-- 会员积分卡 -->
            <view class="member-card flex-row align-c jc-sb">
                <view class="flex-row align-c flex-1">
                    <image class="avatar circle dis-block" :src="(user && user.avatar) || avatar_default" mode="aspectFill" @tap="avatar_event"></image>
                    <view class="member-info padding-left-main flex-1">
                        <block v-if="(user || null) != null">
                            <view class="member-name single-text">{{ user.user_name_view || $t('index-points.index-points.greeting') }}</view>
                            <view class="member-identity single-text">MEMBERIDENTITY</view>
                        </block>
                        <block v-else>
                            <view class="member-name single-text" @tap="login_event">{{ $t('index-points.index-points.greeting') }}</view>
                            <view class="member-identity single-text">MEMBERIDENTITY</view>
                        </block>
                    </view>
                </view>
                <!-- 可用积分 -->
                <view class="member-points tc">
                    <view class="points-label">{{ $t('index-points.index-points.points_usable') }}</view>
                    <view class="points-value">{{ integral_view }}</view>
                </view>
            </view>

            <!-- 搜索框 -->
            <view class="search-bar flex-row align-c" @tap="search_event">
                <view class="search-icon"></view>
                <text class="search-placeholder">{{ $t('index-points.index-points.search_placeholder') }}</text>
            </view>

            <!-- 筛选 Tab：全部 / 我能兑换 / 积分 / 分类筛选 -->
            <view class="filter-tabs flex-row align-c">
                <view
                    v-for="(item, index) in filter_tabs"
                    :key="index"
                    class="filter-tab pr"
                    :class="active_tab == index ? 'filter-tab-active' : ''"
                    @tap="tab_change_event(index)"
                >
                    <text>{{ item.name }}</text>
                    <!-- 积分/分类带上下三角排序图标 -->
                    <view v-if="item.sortable" class="sort-arrows">
                        <view class="arrow-up"></view>
                        <view class="arrow-down"></view>
                    </view>
                    <view v-if="active_tab == index" class="tab-underline pa"></view>
                </view>
            </view>

            <!-- 商品兑换网格 -->
            <view v-if="goods_list_view.length > 0" class="goods-grid flex-row flex-wrap jc-sb">
                <view
                    v-for="(item, index) in goods_list_view"
                    :key="index"
                    class="goods-card pr"
                    :data-value="item.goods_url"
                    @tap="goods_event"
                >
                    <view class="goods-img-box pr">
                        <image class="goods-img dis-block wh-auto" :src="item.images" mode="aspectFill"></image>
                        <!-- 已兑完水印：TODO 待后端售罄字段确认（见对接文档） -->
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

            <!-- 热门推荐 -->
            <view v-if="hot_list.length > 0" class="hot-section">
                <view class="hot-header flex-row align-c jc-sb" @tap="hot_more_event">
                    <text class="hot-title">{{ $t('index-points.index-points.hot_recommend') }}</text>
                    <view class="flex-row align-c">
                        <text class="hot-more">{{ $t('index-points.index-points.view_all') }}</text>
                        <text class="hot-more-arrow">&gt;</text>
                    </view>
                </view>
                <view class="goods-grid flex-row flex-wrap jc-sb">
                    <view
                        v-for="(item, index) in hot_list"
                        :key="index"
                        class="goods-card pr"
                        :data-value="item.goods_url"
                        @tap="goods_event"
                    >
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
            </view>

            <!-- 无数据提示 -->
            <component-no-data
                v-if="goods_list_view.length == 0"
                :propStatus="data_list_loding_status"
                :propMsg="data_list_loding_msg"
            ></component-no-data>

            <!-- 结尾 -->
            <component-bottom-line v-else :propStatus="data_bottom_line_status"></component-bottom-line>
        </view>

        <!-- 自定义底部 Tab：商品兑换 / 礼包劵兑换 / 个人中心 -->
        <view class="custom-tabbar flex-row align-c">
            <view
                v-for="(item, index) in tabbar_list"
                :key="index"
                class="tabbar-item tc flex-1"
                :class="item.active ? 'tabbar-item-active' : ''"
                @tap="tabbar_event(item)"
            >
                <view class="tabbar-icon" :class="'tabbar-icon-' + item.icon"></view>
                <view class="tabbar-text">{{ item.name }}</view>
            </view>
        </view>
        <!-- 底部 Tab 占位 -->
        <view class="custom-tabbar-seat"></view>

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
                avatar_default: app.globalData.data.default_user_head_src,
                currency_symbol: app.globalData.currency_symbol(),

                // 页面数据
                params: null,
                user: null,
                data_base: null,
                user_integral: null,
                default_images_data: {},

                // 列表加载状态
                data_bottom_line_status: false,
                data_list_loding_status: 1,
                data_list_loding_msg: '',

                // 商品兑换列表
                goods_list: [],
                // 热门推荐列表（TODO：当前复用兑换列表，待后端独立推荐位接口）
                hot_list: [],

                // 筛选 Tab
                active_tab: 0,
                // 积分排序方向（积分 Tab 再次点击时切换）
                points_sort: 'asc',
                // 分类筛选选中的分类名（空为全部）
                category_filter: '',
                filter_tabs: [
                    { name: this.$t('index-points.index-points.tab_all'), sortable: false },
                    { name: this.$t('index-points.index-points.tab_can_exchange'), sortable: false },
                    { name: this.$t('index-points.index-points.tab_points'), sortable: true },
                    { name: this.$t('index-points.index-points.tab_category'), sortable: true },
                ],

                // 底部自定义 Tab
                tabbar_list: [
                    { name: this.$t('index-points.index-points.nav_goods'), icon: 'goods', active: true, url: '/pages/index/index' },
                    { name: this.$t('index-points.index-points.nav_gift'), icon: 'gift', active: false, url: '/pages/plugins/points/scan/scan' },
                    { name: this.$t('index-points.index-points.nav_user'), icon: 'user', active: false, url: '/pages/user/user' },
                ],

                // 自定义分享
                share_info: {},
            };
        },
        components: {
            componentCommon,
            componentNoData,
            componentBottomLine,
        },
        computed: {
            // 可用积分展示（千分位）
            integral_view() {
                var v = (this.user_integral && this.user_integral.integral) || 0;
                return String(v).replace(/\B(?=(\d{3})+(?!\d))/g, ',');
            },
            // 筛选 Tab 作用后的商品列表（前端过滤/排序）
            goods_list_view() {
                var list = (this.goods_list || []).slice();
                switch (this.active_tab) {
                    // 我能兑换：所需积分 ≤ 我的可用积分、且未售罄
                    case 1:
                        var mine = parseInt((this.user_integral && this.user_integral.integral) || 0);
                        list = list.filter((v) => parseInt(v.plugins_points_exchange_integral || 0) > 0 && parseInt(v.plugins_points_exchange_integral || 0) <= mine && !this.is_sold_out(v));
                        break;
                    // 积分：按所需积分排序（再次点击切换升/降序）
                    case 2:
                        list.sort((a, b) => (parseInt(a.plugins_points_exchange_integral || 0) - parseInt(b.plugins_points_exchange_integral || 0)) * (this.points_sort == 'asc' ? 1 : -1));
                        break;
                    // 分类筛选：按选中分类名过滤
                    case 3:
                        if (this.category_filter != '') {
                            list = list.filter((v) => (v.category_names || []).indexOf(this.category_filter) != -1);
                        }
                        break;
                }
                return list;
            },
        },
        onLoad(params) {
            app.globalData.page_event_onload_handle(params);
            this.setData({ params: params });
        },
        onShow() {
            app.globalData.page_event_onshow_handle();
            this.setData({ user: app.globalData.get_user_cache_info() });
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
            // 获取首页数据（复用积分商城 points 插件接口）
            get_data() {
                uni.request({
                    url: app.globalData.get_request_url('index', 'index', 'points'),
                    method: 'POST',
                    data: {},
                    dataType: 'json',
                    success: (res) => {
                        uni.stopPullDownRefresh();
                        if (res.data.code == 0) {
                            var data = res.data.data;
                            var goods_list = (data.base && data.base.goods_exchange_data) || [];
                            this.setData({
                                data_base: data.base || null,
                                user_integral: data.user_integral || null,
                                default_images_data: data.default_images_data || {},
                                goods_list: goods_list,
                                // TODO：热门推荐暂复用兑换列表，待后端提供独立推荐位接口
                                hot_list: goods_list.slice(0, 2),
                                data_list_loding_status: 3,
                                data_list_loding_msg: '',
                                data_bottom_line_status: true,
                            });

                            if ((this.data_base || null) != null) {
                                this.setData({
                                    share_info: {
                                        title: this.data_base.seo_title || this.data_base.application_name,
                                        desc: this.data_base.seo_desc,
                                        path: '/pages/index/index',
                                        img: this.data_base.right_images,
                                    },
                                });
                                // #ifndef MP-ALIPAY
                                if ((this.data_base.application_name || null) != null) {
                                    uni.setNavigationBarTitle({ title: this.data_base.application_name });
                                }
                                // #endif
                            }
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

            // 单品积分价展示
            // TODO：确认积分商城单品的积分价字段名（见对接文档），当前优先取常见字段
            goods_points_view(item) {
                // 兑换所需积分（后端字段 plugins_points_exchange_integral；give_integral 是购买赠送积分，语义不同，勿混用）
                return item.plugins_points_exchange_integral || 0;
            },

            // 是否已兑完
            // TODO：确认售罄字段名（inventory / stock / is_sold_out），当前优先取常见字段
            is_sold_out(item) {
                if ((item.is_sold_out || 0) == 1) {
                    return true;
                }
                if (item.inventory != undefined && parseInt(item.inventory) <= 0) {
                    return true;
                }
                return false;
            },

            // 搜索
            search_event() {
                uni.navigateTo({ url: '/pages/goods-search/goods-search' });
            },

            // 筛选 Tab 切换（前端过滤/排序，见 computed goods_list_view）
            tab_change_event(index) {
                // 积分：已选中时再次点击切换升/降序
                if (index == 2 && this.active_tab == 2) {
                    this.setData({ points_sort: this.points_sort == 'asc' ? 'desc' : 'asc' });
                    return;
                }
                // 分类筛选：弹出分类选单（分类名来自商品的 category_names）
                if (index == 3) {
                    var names = [];
                    for (var i in this.goods_list) {
                        var cs = this.goods_list[i].category_names || [];
                        for (var k in cs) {
                            if (names.indexOf(cs[k]) == -1) {
                                names.push(cs[k]);
                            }
                        }
                    }
                    if (names.length == 0) {
                        app.globalData.showToast(this.$t('index-points.index-points.category_none'));
                        return;
                    }
                    var items = [this.$t('index-points.index-points.category_all')].concat(names);
                    uni.showActionSheet({
                        itemList: items,
                        success: (res) => {
                            this.setData({
                                active_tab: 3,
                                category_filter: res.tapIndex == 0 ? '' : items[res.tapIndex],
                            });
                        },
                    });
                    return;
                }
                // 我能兑换需要账户积分，未登录先登录
                if (index == 1 && this.user == null) {
                    this.login_event();
                    return;
                }
                this.setData({ active_tab: index, category_filter: '' });
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

            // 热门推荐查看全部
            // TODO：待后端独立推荐位/列表页确认目标地址
            hot_more_event() {
                uni.navigateTo({ url: '/pages/goods-search/goods-search' });
            },

            // 头像
            avatar_event() {
                if (this.user == null) {
                    this.login_event();
                    return;
                }
                if (app.globalData.data.default_user_head_src != this.user.avatar) {
                    uni.previewImage({ current: this.user.avatar, urls: [this.user.avatar] });
                }
            },

            // 登录
            login_event() {
                this.setData({ user: app.globalData.get_user_info(this, 'login_event') || null });
            },

            // 底部 Tab
            tabbar_event(item) {
                // 当前页（商品兑换）即首页，不跳转
                if (item.active) {
                    return;
                }
                if (item.url.indexOf('/pages/user/user') != -1) {
                    uni.switchTab({ url: item.url });
                    return;
                }
                uni.navigateTo({ url: item.url });
            },

            // url 事件
            url_event(e) {
                app.globalData.url_event(e);
            },
        },
    };
</script>
<style>
    @import './index.css';
</style>
