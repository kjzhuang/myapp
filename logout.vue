<template>
    <div class="log-out">
        <div class="logo"><img src="../assets/imgs/logo.png" alt=""></div>
        <div class="bd">
            <div class="item">您好，{{userInfo.realname}}</div>
            <div class="item">当前登录账号：{{userInfo.username}}</div>
            <div class="outBtn btn" @click="back">退出</div>
            <div class="backBtn btn" @click="home">返回首页</div>
        </div>
    </div>
</template>
<script>
import { Dialog } from 'vant';
import { OrderService } from '@/services';

export default {
    data() {
        return {
            userInfo: {},
        };
    },
    components: {
        [Dialog.Component.name]: Dialog.Component,
    },
    mounted() {
        const param = localStorage.getItem('wxParamsTianyan');
        if (param) {
            this.userInfo = JSON.parse(param);
        }
    },
    methods: {
        back() {
            Dialog.confirm({
                message: '确定退出？',
            }).then(() => {
                localStorage.setItem('wxParamsTianyan', JSON.stringify({ openid: this.userInfo.openid })); // 删除注册信息,只留openid
                this.userInfo = {};
                OrderService.logout({}).then(() => {
                    this.$router.push({ // 退出去登录页面
                        name: 'login'
                    }); 
                }).catch((err) => {
                    console.log('登出失败', err);
                });
            }).catch(() => {
                // on cancel
            });
        },
        home() {
            window.wx.closeWindow();
        },
    },
};
</script>
<style lang="less">
    .log-out{
        font-size: .3rem;
        img{
            width:100%;
        }
        .logo{
            width:1.6rem;
            margin:1.2rem auto;
        }
        .bd{
            padding: 0 .32rem;
        }
        .item{
            color: #399fd0;
            font-size: .36rem;
            text-align: center;
            &:first-child{padding-bottom: .2rem;}
        }
        .btn{
            background: #5A32D2;
            box-shadow: 0 3px 6px 0 rgba(90,50,210,0.40);
            border-radius: 5px;
            height:1.1rem;
            line-height: 1.1rem;
            text-align: center;
            color:#fff;
        }
        .outBtn{
            margin-top:.8rem;
        }
        .backBtn{
            margin-top:.4rem;
        }
    }
</style>