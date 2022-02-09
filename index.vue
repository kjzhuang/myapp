<template>
    <div class="home">
        <div v-if="show_notice" style="position: relative;width: 102vw;left: -0.32rem;top: -0.44rem;">
            <van-notice-bar wrapable :scrollable="false" color="#1989fa" background="#cf1322">
                <div slot="right-icon" @click="show_notice = false">
                    <img src="@/assets/imgs/close.svg" alt="">
                </div>
                <span style="color:#fff;font-size:0.24rem" v-if="currentVersions =='old'">
                    在途订单,我的订阅,异常推送等功能,存在新版本,可切换到新版本试用,旧版本将在2022-01-01号正式停用.
                </span>
                <span style="color:#fff;font-size:0.24rem" v-if="currentVersions =='new'">
                    已切换到新版本的体验版本,如果您有任何疑问和需求,都可以随时联系我们,旧版本将在2022-01-01号正式停用.
                </span>
            </van-notice-bar>
        </div>
        <div class="hd">
            <div class="log"><img src="../assets/imgs/img1.png" alt="">
                <span v-if="!username" class="status">未登录</span>
                <span v-else class="account status">{{username}}</span>
            </div>
            <div v-if="!username" class="logBtn" @click="login">登录</div>
            <div v-else class="logBtn" @click="backOut">退出</div>
        </div>
        <!-- 新版本页面 -->
        <van-row gutter="20" class="items" v-if="currentVersions =='new'">
            <van-col span="8">
                <div class="item">
                    <img @click="goNewPage('carTrack')" src="../assets/imgs/track.png" alt="">
                    <div class="text">定位追踪</div>
                </div>
            </van-col>
            <van-col span="8">
                <div @click="goNewPage('deviceList')" class="item">
                    <img src="../assets/imgs/deviceList.png" alt="">
                    <div class="text">设备列表</div>
                </div>
            </van-col>
            <van-col span="8">
                <div @click="goNewPage('scan')" class="item">
                    <img src="../assets/imgs/scanIcon.png" alt="">
                    <div class="text">扫码绑定</div>
                </div>
            </van-col>
            <!-- <van-col span="8">
                <div @click="goNewPage('order/page/infoList')" class="item">
                    <img src="../assets/imgs/orderList.png" alt="">
                    <div class="text">在途订单</div>
                </div>
            </van-col> -->
            <van-col span="8">
                <div @click="goNewPage('orderList')" class="item">
                    <img src="../assets/imgs/orderList.png" alt="">
                    <div class="text">订单列表</div>
                </div>
            </van-col>
            <van-col span="8">
                <div @click="goNewPage('alarmConfig')" class="item">
                    <img src="../assets/imgs/order.png" alt="">
                    <div class="text">报警订阅</div>
                </div>
            </van-col>
            <van-col span="8">
                <div @click="goNewPage('alarmList')" class="item">
                    <img src="../assets/imgs/event.png" alt="">
                    <div class="text">报警事件</div>
                </div>
            </van-col>
        </van-row>
        <!-- 老版本页面 -->
        <van-row gutter="20" class="items" v-if="currentVersions == 'old'">
            <van-col span="8">
                <div class="item">
                    <img @click="goOldPage('carTrack')" src="../assets/imgs/track.png" alt="">
                    <div class="text">定位追踪</div>
                </div>
            </van-col>
            <van-col span="8">
                <div @click="goOldPage('deviceList')" class="item">
                    <img src="../assets/imgs/deviceList.png" alt="">
                    <div class="text">设备列表</div>
                </div>
            </van-col>
            <van-col span="8">
                <div @click="goOldPage('scan')" class="item">
                    <img src="../assets/imgs/scanIcon.png" alt="">
                    <div class="text">扫码绑定</div>
                </div>
            </van-col>
            <!-- <van-col span="8">
                <div @click="goOldPage('order/page/infoList')" class="item">
                    <img src="../assets/imgs/orderList.png" alt="">
                    <div class="text">在途订单</div>
                </div>
            </van-col> -->
            <van-col span="8">
                <div @click="goOldPage('order/page/infoList')" class="item">
                    <img src="../assets/imgs/orderList.png" alt="">
                    <div class="text">在途订单</div>
                </div>
            </van-col>
            <van-col span="8">
                <div @click="goOldPage('subscription')" class="item">
                    <img src="../assets/imgs/order.png" alt="">
                    <div class="text">我的订阅</div>
                </div>
            </van-col>
            <van-col span="8">
                <div @click="goOldPage('exception')" class="item">
                    <img src="../assets/imgs/event.png" alt="">
                    <div class="text">异常推送</div>
                </div>
            </van-col>
        </van-row>
        <!-- 版本切换 -->
        <div v-if="currentVersions=='new'" class="change_version" @click="changeCurrentVersions('old')">
             返回旧版本<i class="iconfont iconchangyongicon" style="color:#FFFFFF;font-size: .28rem;"></i>
        </div>
        <div v-else class="change_version" @click="changeCurrentVersions('new')">
             前往新版本<i class="iconfont iconchangyongicon" style="color:#FFFFFF;font-size: .28rem;"></i>
        </div>
        <div v-if="!username" class="ps">PS：使用该页面功能查看数据，请先登录对应G7账号</div>

    </div>
</template>
<script>
import { Dialog } from 'vant';
import { OrderService } from '@/services';
import {OrderApiService} from '@/services'

import store from '@/store';

export default {
    data() {
        return {
            hasSavePageVisit: false,
            show_notice: true,
            currentVersions: 'new'
        };
    },
    components: {
        [Dialog.Component.name]: Dialog.Component,
    },
    created() {
        this.currentVersions = localStorage.getItem('appVerions')?localStorage.getItem('appVerions'):'new'
    },
    mounted() {
        if (!this.hasSavePageVisit && this.username) {
            console.log('mo上传');
            this.hasSavePageVisit = true
            this.savePageVisit()
        }
    },
    watch: {
        username () {
            if (!this.hasSavePageVisit && this.username) {
                console.log('us上传');
                this.hasSavePageVisit = true
                this.savePageVisit()
            }
        }
    },
    computed: {
        username: function() {
            return this.$store.getters.userName.userName
        } 
    },
    methods: {
        savePageVisit () {
            if (!this.currentVersions) return
            let parms = {
                viewid: this.currentVersions=='new'?1:2,
                view: `公众号首页-${this.currentVersions=='new'?'新页面':'旧页面'}`
            }
            OrderApiService.savePageVisit(parms).then(res=>{
                console.log('上传成功');
                console.log(this.currentVersions);
            })
        },
        login() {
            this.$router.push({
                name: 'login',
            });
        },
        changeCurrentVersions (v) {
            // 保存到缓存
            localStorage.setItem('appVerions',v)
            this.currentVersions = v
            location.reload();
        },
        goOldPage(page) {
            let new_page = ['carTrack','deviceList']
            if (new_page.indexOf(page)>-1) {
                this.$router.push({
                    name: page,
                    query: { home: 'home' },
                });
            } else {
                const wxParams = localStorage.getItem('wxParamsTianyan');
                const wxPara = wxParams ? JSON.parse(wxParams) : {};
                const openid = wxPara && wxPara.openid ? wxPara.openid : '';
                const openidQuery = openid ? `?openidNew=${openid}` : '';
                if (window.location.hostname.indexOf('order-mobile.huoyunren') > -1) {
                    window.location.href = `http://g7s.huoyunren.com/product/order/wechat/${page}.html${openidQuery}`; // 线上
                } else {
                    window.location.href = `http://demo.g7s.chinawayltd.com/product/order/wechat/${page}.html${openidQuery}`;
                }
            }
        },

        goNewPage(page) {
            let new_page = ['carTrack','deviceList','orderList','alarmConfig','alarmList']
            if (new_page.indexOf(page)>-1) {
                this.$router.push({
                    name: page,
                    query: { home: 'home' },
                });
            } else {
                const wxParams = localStorage.getItem('wxParamsTianyan');
                const wxPara = wxParams ? JSON.parse(wxParams) : {};
                const openid = wxPara && wxPara.openid ? wxPara.openid : '';
                const openidQuery = openid ? `?openidNew=${openid}` : '';
                if (window.location.hostname.indexOf('order-mobile.huoyunren') > -1) {
                    window.location.href = `http://g7s.huoyunren.com/product/order/wechat/${page}.html${openidQuery}`; // 线上
                } else {
                    window.location.href = `http://demo.g7s.chinawayltd.com/product/order/wechat/${page}.html${openidQuery}`;
                }
            }
        },
        backOut() {
            Dialog.confirm({
                message: '确定退出？',
            }).then(() => {
                const wxParamsTianyan = localStorage.getItem('wxParamsTianyan');
                if (wxParamsTianyan) {
                    const user = JSON.parse(wxParamsTianyan);
                    localStorage.setItem('wxParamsTianyan', JSON.stringify({ openid: user.openid })); // 删除注册信息,只留openid
                }
                store.commit('setUserName', { userName: '' })
                OrderService.logout({}).then(() => {
                    this.$router.push({ // 退出去登录页面
                        name: 'login',
                    });
                }).catch((err) => {
                    console.log('登出失败', err);
                });
            }).catch(() => {
                // on cancel
            });
        },
    },
};
</script>
<style lang="less">
    .home{
        padding:.44rem .32rem 0;
        font-size: .24rem;
        position: relative;
        img{
            width:100%;
        }
        .hd{
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom:.44rem;
            img{width:1.12rem;}
            .status{
                color: #666666;
                font-size:.4rem;
                padding-left:.32rem;
            }
            .account{
                color: #0A0A0A;
            }
        }
        .log{
            display: flex;
            align-items: center;
        }
        .logBtn{
            background: #5A32D2;
            box-shadow: 0 3px 6px 0 rgba(90,50,210,0.40);
            border-radius: .26rem;
            color:#fff;
            text-align: center;
            height: .52rem;
            line-height: .56rem;
            width:1.48rem;
        }
        .items{
            display: flex;
            flex-wrap: wrap;
            .item{
                text-align: center;
                background: #FFFFFF;
                box-shadow: 0 1px 8px 0 rgba(0,0,0,0.12);
                border-radius: 5px;
                font-size: .24rem;
                color: #0A0A0A;
                padding:.3rem 0;
                margin-bottom:.3rem;
            }
            img{width:1rem;}
        }
        .text{
            margin-top:.16rem;
        }
        .ps{
            // position: absolute;
            bottom:1rem;
            left:.34rem;
            color: #AAAAAA;
        }
        .backBtn{
            font-size: .3rem;
            background: #5A32D2;
            box-shadow: 0 3px 6px 0 rgba(90,50,210,0.40);
            border-radius: 5px;
            height:1.1rem;
            line-height: 1.1rem;
            text-align: center;
            color:#fff;
            position: absolute;
            left:50%;
            bottom:2rem;
            width: 90%;
            transform: translate(-50%, 0);
        }
        .change_version {
            position: fixed;
            right: 0;
            top: 9rem;
            height: .68rem;
            width: 2.2rem;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: .24rem;
            color: #FFFFFF;
            background-color: rgba(0, 0, 0, .7);
            box-shadow: -1px 2px 4px 0 rgba(0, 0, 0, 0.14);
            border-radius: 100px 0 0 100px;
        }
    }
</style>
<style lang="less">
.van-notice-bar--wrapable{
    height: 0.8rem;
    .van-notice-bar__content{
        padding-right: 4px;
    }
}
    
</style>