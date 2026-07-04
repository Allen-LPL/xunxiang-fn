<template>
    <view :class="theme_view">
        <view v-if="(data || null) != null" class="exchange-page bs-bb">
            <!-- 初始态：兑换码录入 -->
            <block v-if="data.status == -10000">
                <!-- 标题 -->
                <view class="ex-hero">
                    <text class="ex-hero-title">兑换中心</text>
                    <view class="ex-hero-underline"></view>
                </view>

                <!-- 礼劵代码 -->
                <view class="ex-code-card">
                    <view class="ex-code-label">礼劵代码</view>
                    <view class="ex-code-row">
                        <input class="ex-code-input" type="text" maxlength="19" confirm-type="done" v-model="redeem_code" placeholder="请输入兑换码（支持扫码输入）" placeholder-class="ex-code-ph" @confirm="confirm_event" />
                        <view class="ex-code-qr" @tap="scan_event">
                            <iconfont name="icon-qrcode" size="52rpx" color="#2b2b2b"></iconfont>
                        </view>
                    </view>
                </view>

                <!-- 扫码兑换 -->
                <view class="ex-scan-wrap">
                    <view class="ex-scan-circle" @tap="scan_event">
                        <iconfont name="icon-camera" size="84rpx" color="#ffffff"></iconfont>
                        <text class="ex-scan-circle-text">扫码兑换</text>
                    </view>
                    <view class="ex-scan-tip">点击开启相机扫描礼券二维码</view>
                </view>

                <!-- 确认兑换 -->
                <view class="ex-confirm" hover-class="ex-confirm-hover" @tap="confirm_event">
                    <text class="ex-confirm-text">确认兑换</text>
                    <view class="ex-confirm-arrow">
                        <iconfont name="icon-arrow-right" size="30rpx" color="#fffaf0"></iconfont>
                    </view>
                </view>

                <!-- 近期记录 -->
                <view class="ex-record">
                    <view class="ex-record-head">
                        <text class="ex-record-title">近期记录</text>
                        <view class="ex-record-more" @tap="record_more_event">
                            <text class="ex-record-more-text">查看更多</text>
                            <iconfont name="icon-angle-right" size="24rpx" color="#999999"></iconfont>
                        </view>
                    </view>

                    <block v-if="(record_list || null) != null && record_list.length > 0">
                        <view v-for="(item, index) in record_list" :key="index" :class="'ex-rec-card ex-rec-card-'+item.state" @tap="record_use_event(item)">
                            <view :class="'ex-rec-icon ex-rec-icon-'+item.state">
                                <iconfont name="icon-youhuiquan-shangchengwu" size="56rpx" :color="item.state == 'usable' ? '#735b00' : '#9d9d9d'"></iconfont>
                            </view>
                            <view class="ex-rec-info">
                                <view :class="'ex-rec-info-title '+(item.state == 'usable' ? '' : 'ex-rec-info-disabled')">{{item.title || '适用于:指定商品'}}</view>
                                <view class="ex-rec-info-date">{{item.date_text}}</view>
                            </view>
                            <view :class="'ex-rec-status ex-rec-status-'+item.state">{{item.status_text}}</view>
                        </view>
                    </block>
                    <view v-else class="ex-record-empty">暂无兑换记录</view>
                </view>
            </block>

            <!-- 结果态：兑换成功/失败 -->
            <block v-else>
                <view class="ex-result">
                    <image v-if="data.status == 0 && (default_images_data.scan_success_images || '') != ''" :src="default_images_data.scan_success_images" mode="widthFix" class="ex-result-img"></image>
                    <image v-else-if="data.status == -100 && (default_images_data.scan_fail_images || '') != ''" :src="default_images_data.scan_fail_images" mode="widthFix" class="ex-result-img"></image>
                    <view :class="'ex-result-icon '+(data.status == 0 ? 'ex-result-icon-ok' : 'ex-result-icon-fail')" v-else>
                        <iconfont :name="data.status == 0 ? 'icon-checked' : 'icon-close-round'" size="120rpx" :color="data.status == 0 ? '#caa635' : '#c55b5b'"></iconfont>
                    </view>
                    <view class="ex-result-msg">{{data.msg}}</view>
                    <view class="ex-confirm ex-result-back" hover-class="ex-confirm-hover" @tap="reset_event">
                        <text class="ex-confirm-text">继续兑换</text>
                    </view>
                </view>
            </block>

            <!-- 未登录 -->
            <view v-if="user == null" class="ex-login-fixed">
                <view class="ex-confirm" hover-class="ex-confirm-hover" @tap="login_event">
                    <text class="ex-confirm-text">{{$t('pages.login')}}</text>
                </view>
            </view>

            <!-- 公共 -->
            <component-common ref="common"></component-common>
        </view>
        <block v-else>
            <!-- 提示信息 -->
            <component-no-data :propStatus="data_list_loding_status" :propMsg="data_list_loding_msg"></component-no-data>
        </block>
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
                params: null,
                user: null,
                data_base: null,
                data: null,
                default_images_data: {},
                // 兑换码输入
                redeem_code: '',
                // 近期兑换记录（由 get_record 从 giftcard 兑换记录接口填充，支持分页加载更多）
                record_list: [],
                record_page: 1,
                record_limit: 10,
                record_no_more: false,
                record_loading: false,
                // 自定义分享信息
                share_info: {},
            };
        },

        components: {
            componentCommon,
            componentNoData,
        },

        onLoad(params) {
            // 调用公共事件方法
            app.globalData.page_event_onload_handle(params);

            // 设置参数
            this.setData({
                params: app.globalData.launch_params_handle(params),
            });
        },

        onShow() {
            // 调用公共事件方法
            app.globalData.page_event_onshow_handle();

            // 用户信息
            this.setData({
                user: app.globalData.get_user_cache_info(),
            });

            // 获取数据
            this.get_data();

            // 获取近期兑换记录
            this.get_record();

            // 公共onshow事件
            if ((this.$refs.common || null) != null) {
                this.$refs.common.on_show();
            }
        },

        // #ifdef H5
        onHide() {
            this.h5_scan_stop();
        },

        onUnload() {
            this.h5_scan_stop();
        },
        // #endif

        methods: {
            // 获取数据
            get_data() {
                uni.request({
                    url: app.globalData.get_request_url('index', 'scan', 'points'),
                    method: 'POST',
                    data: this.params,
                    dataType: 'json',
                    success: (res) => {
                        if (res.data.code == 0) {
                            var data = res.data.data;
                            // 兑换码兜底场景：底层“没有相关扫码”类提示对用户不可理解，换成明确文案
                            if (this.code_fallback && (data.data || null) != null && data.data.status != 0 && data.data.status != -10000) {
                                data.data.msg = '兑换码无效或已被使用，请核对后重试';
                            }
                            this.code_fallback = false;
                            this.setData({
                                data_base: data.base || null,
                                data: data.data || null,
                                default_images_data: data.default_images_data || {},
                                data_list_loding_msg: '',
                                data_list_loding_status: 0,
                            });

                            if ((this.data_base || null) != null) {
                                // 基础自定义分享
                                this.setData({
                                    share_info: {
                                        title: this.$t('pages.plugins-points-scan'),
                                        path: '/pages/plugins/points/scan/scan'
                                    }
                                });
                            }
                        } else if (app.globalData.is_login_check(res.data, this, 'get_data')) {
                            // 非登录失效的业务错误才展示错误态（登录失效已由 is_login_check 引导登录）
                            this.setData({
                                data_list_loding_status: 2,
                                data_list_loding_msg: res.data.msg,
                            });
                        }
                        // 分享菜单处理
                        app.globalData.page_share_handle(this.share_info);
                    },
                    fail: () => {
                        this.setData({
                            data_list_loding_status: 2,
                            data_list_loding_msg: this.$t('common.internet_error_tips'),
                        });
                        app.globalData.showToast(this.$t('common.internet_error_tips'));
                    },
                });
            },

            // 获取近期兑换记录（giftcard 兑换记录，需登录；page>1 时追加）
            get_record(page) {
                if ((this.user || null) == null) {
                    this.setData({ record_list: [], record_page: 1, record_no_more: false });
                    return;
                }
                if (this.record_loading) {
                    return;
                }
                page = page || 1;
                this.setData({ record_loading: true });
                uni.request({
                    url: app.globalData.get_request_url('exchangerecord', 'index', 'giftcard'),
                    method: 'POST',
                    data: { page: page, limit: this.record_limit },
                    dataType: 'json',
                    success: (res) => {
                        this.setData({ record_loading: false });
                        if (res.data.code == 0) {
                            var list = res.data.data || [];
                            this.setData({
                                record_list: page <= 1 ? list : this.record_list.concat(list),
                                record_page: page,
                                record_no_more: list.length < this.record_limit,
                            });
                            if (page > 1 && list.length == 0) {
                                app.globalData.showToast('没有更多记录了');
                            }
                        } else {
                            // 登录失效(-400)则引导重新登录，登录后回调本方法
                            app.globalData.is_login_check(res.data, this, 'get_record', page);
                        }
                    },
                    fail: () => {
                        this.setData({ record_loading: false });
                    },
                });
            },

            // 登录事件
            login_event() {
                var user = app.globalData.get_user_info(this, 'login_event');
                this.setData({
                    user: user || null,
                });
            },

            // 确认兑换：先试礼品卡(giftcard，支持余额/优惠券/积分/实物)，非礼品卡再回退积分码(points/scan)
            confirm_event() {
                var code = (this.redeem_code || '').replace(/\s/g, '');
                if (code == '') {
                    app.globalData.showToast('请输入兑换码');
                    return false;
                }
                this.giftcard_exchange(code);
            },

            // 礼品卡兑换
            giftcard_exchange(code) {
                uni.showLoading({ title: this.$t('common.processing_in_text') });
                uni.request({
                    url: app.globalData.get_request_url('exchange', 'index', 'giftcard'),
                    method: 'POST',
                    data: { secret_key: code },
                    dataType: 'json',
                    success: (res) => {
                        uni.hideLoading();
                        if (res.data.code == 0) {
                            // 兑换成功，按卡密类型路由（3=实物商品）
                            var data_type = res.data.data && res.data.data.data_type != undefined ? parseInt(res.data.data.data_type) : -1;
                            if (data_type == 3) {
                                app.globalData.showToast('兑换成功，请领取实物商品');
                                this.setData({ redeem_code: '' });
                                setTimeout(() => {
                                    uni.navigateTo({ url: '/pages/plugins/points/exchange-goods/exchange-goods' });
                                }, 800);
                            } else {
                                // 余额/优惠券/积分：提示并刷新页面数据与记录
                                app.globalData.showToast(res.data.msg || '兑换成功');
                                var temp = this.params || {};
                                delete temp['id'];
                                this.setData({ redeem_code: '', params: temp });
                                this.get_data();
                                this.get_record();
                            }
                        } else if ((res.data.msg || '').indexOf('没有相关礼品卡密') != -1) {
                            // 非礼品卡兑换码 → 回退积分码(points/scan)
                            this.points_scan_exchange(code);
                        } else if (app.globalData.is_login_check(res.data, this, 'confirm_event')) {
                            // 其它业务错误（如已被使用），非登录问题时提示
                            app.globalData.showToast(res.data.msg);
                        }
                    },
                    fail: () => {
                        uni.hideLoading();
                        app.globalData.showToast(this.$t('common.internet_error_tips'));
                    },
                });
            },

            // 积分码兑换（原 points/scan 逻辑，将兑换码作为 id 提交，结果渲染到本页结果态）
            points_scan_exchange(code) {
                var temp = this.params || {};
                temp['id'] = code;
                // 标记兑换码兜底场景，失败提示走友好文案
                this.code_fallback = true;
                this.setData({
                    params: temp,
                });
                this.get_data();
            },

            // 继续兑换（重置回录入态）
            reset_event() {
                var temp = this.params || {};
                delete temp['id'];
                this.setData({
                    params: temp,
                    redeem_code: '',
                });
                this.get_data();
            },

            // 查看更多记录（分页追加加载）
            record_more_event() {
                if (this.record_no_more) {
                    app.globalData.showToast('没有更多记录了');
                    return;
                }
                this.get_record(this.record_page + 1);
            },

            // 记录点击：实物→我的兑换商品/已领取提示；其余积分类兑换→积分明细（业务无钱包/优惠券概念）
            record_use_event(item) {
                var dt = item.data_type != undefined ? parseInt(item.data_type) : -1;
                if (dt == 3) {
                    if (item.state == 'usable') {
                        uni.navigateTo({ url: '/pages/plugins/points/exchange-goods/exchange-goods' });
                    } else {
                        app.globalData.showToast('该实物商品已领取');
                    }
                    return;
                }
                uni.navigateTo({ url: '/pages/user-integral/user-integral' });
            },

            // 扫码事件
            scan_event() {
                // #ifdef H5
                this.h5_scan_start();
                return;
                // #endif
                // #ifndef H5
                var self = this;
                uni.scanCode({
                    success: function (res) {
                        if (res.result !== '') {
                            self.handle_scan_result(res.result);
                        }
                    },
                });
                // #endif
            },

            // 处理扫码结果（兼容“带 id 的链接”与“纯兑换码”两种二维码）
            handle_scan_result(text) {
                if ((text || '') === '') {
                    return;
                }
                var arr = ['/points-scan-index-id-', 'plugins/index/pluginsname/points/pluginscontrol/scan/pluginsaction/index/id/'];
                var ret = app.globalData.web_url_value_mate(text, arr);
                var value = ret.status == 1 && ret.value != null ? ret.value : text;
                var temp = this.params || {};
                temp['id'] = value;
                this.setData({
                    params: temp,
                    redeem_code: value,
                });
                this.get_data();
            },

            // #ifdef H5
            // H5 调用摄像头扫码（getUserMedia + 原生 BarcodeDetector，零依赖；需 https 或 localhost 安全上下文）
            h5_scan_start() {
                var self = this;
                if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
                    app.globalData.showToast('当前浏览器不支持调用摄像头，请用 App/小程序扫码或手动输入兑换码');
                    return;
                }
                if (typeof window.BarcodeDetector === 'undefined') {
                    app.globalData.showToast('当前浏览器不支持二维码识别，请更换浏览器或手动输入兑换码');
                    return;
                }
                navigator.mediaDevices
                    .getUserMedia({ video: { facingMode: { ideal: 'environment' } } })
                    .then(function (stream) {
                        self._h5_stream = stream;
                        self.h5_scan_build_overlay(stream);
                        self.h5_scan_detect_loop();
                    })
                    .catch(function (err) {
                        var name = err && err.name ? err.name : '';
                        var tips = name == 'NotAllowedError' ? '摄像头权限被拒绝' : '无法访问摄像头（需 https 安全环境）';
                        app.globalData.showToast(tips);
                    });
            },

            // 构建全屏扫码遮罩（直接操作 DOM，规避 uni video 组件不支持 srcObject 的限制）
            h5_scan_build_overlay(stream) {
                var self = this;
                var mask = document.createElement('div');
                mask.setAttribute('style', 'position:fixed;left:0;top:0;right:0;bottom:0;z-index:9999;background:#000;overflow:hidden;');

                var video = document.createElement('video');
                video.setAttribute('playsinline', 'true');
                video.setAttribute('webkit-playsinline', 'true');
                video.muted = true;
                video.autoplay = true;
                video.setAttribute('style', 'width:100%;height:100%;object-fit:cover;');
                video.srcObject = stream;

                var frame = document.createElement('div');
                frame.setAttribute('style', 'position:absolute;left:50%;top:44%;transform:translate(-50%,-50%);width:62vw;height:62vw;border:3px solid #b99359;border-radius:20px;box-shadow:0 0 0 9999px rgba(0,0,0,0.5);');

                var tip = document.createElement('div');
                tip.innerText = '将二维码放入框内，自动识别';
                tip.setAttribute('style', 'position:absolute;left:0;right:0;bottom:16%;text-align:center;color:#fff;font-size:15px;');

                var close = document.createElement('div');
                close.innerText = '✕';
                close.setAttribute('style', 'position:absolute;right:16px;top:16px;width:44px;height:44px;line-height:44px;text-align:center;color:#fff;font-size:26px;');
                close.onclick = function () {
                    self.h5_scan_stop();
                };

                mask.appendChild(video);
                mask.appendChild(frame);
                mask.appendChild(tip);
                mask.appendChild(close);
                document.body.appendChild(mask);

                self._h5_mask = mask;
                self._h5_video = video;
                if (video.play) {
                    video.play();
                }
            },

            // 逐帧识别二维码
            h5_scan_detect_loop() {
                var self = this;
                if (!self._h5_detector) {
                    try {
                        self._h5_detector = new window.BarcodeDetector({ formats: ['qr_code'] });
                    } catch (e) {
                        self._h5_detector = new window.BarcodeDetector();
                    }
                }
                var tick = function () {
                    if (!self._h5_video || !self._h5_mask) {
                        return;
                    }
                    self._h5_detector
                        .detect(self._h5_video)
                        .then(function (codes) {
                            if (codes && codes.length > 0 && codes[0].rawValue) {
                                self.h5_scan_stop();
                                self.handle_scan_result(codes[0].rawValue);
                            } else {
                                self._h5_timer = setTimeout(tick, 300);
                            }
                        })
                        .catch(function () {
                            self._h5_timer = setTimeout(tick, 500);
                        });
                };
                self._h5_timer = setTimeout(tick, 500);
            },

            // 关闭扫码（释放摄像头、移除遮罩）
            h5_scan_stop() {
                if (this._h5_timer) {
                    clearTimeout(this._h5_timer);
                    this._h5_timer = null;
                }
                if (this._h5_stream) {
                    this._h5_stream.getTracks().forEach(function (t) {
                        t.stop();
                    });
                    this._h5_stream = null;
                }
                if (this._h5_mask && this._h5_mask.parentNode) {
                    this._h5_mask.parentNode.removeChild(this._h5_mask);
                }
                this._h5_mask = null;
                this._h5_video = null;
            },
            // #endif
        }
    };
</script>
<style>
    @import './scan.css';
</style>
