<template>
    <div class="log-in">
        <div class="logo"><img src="../assets/imgs/logo.png" alt=""></div>
        <div class="bd">
            <div class="item">
                <!-- <i class="iconfont icon-shouji icon"></i>
                <div class="field">
                    <van-field v-model="param.username" label="" placeholder="请输入用户名" />
                </div> -->
                <div class="searchInn">
                    <div @click="$refs.searchVan.focus()">
                        <i class="iconfont icon-shouji icon"></i>
                        <input v-model="param.username" class="searchBox" type="text" ref="searchVan" placeholder="请输入用户名" style="100%">
                    </div>
                </div>
            </div>
            <div class="item">
                <!-- <i class="iconfont icon-jiesuo1 icon"></i>
                <div class="field">
                    <van-field v-model="param.password" :type="inputType" label="" placeholder="请输入密码" />
                    <i class="iconfont eye " :class="inputType == 'password' ? 'icon-chakanmima' : 'icon-yulan'" @click="togglePasswordType"></i>
                </div> -->
                <div class="searchInn">
                    <div @click="$refs.searchVan2.focus()">
                        <i class="iconfont icon-shouji icon"></i>
                        <input v-model="param.password" :type="inputType" class="searchBox" ref="searchVan2" placeholder="请输入密码" style="100%">
                    </div>
                     <i class="iconfont eye " :class="inputType == 'password' ? 'icon-chakanmima' : 'icon-yulan'" @click="togglePasswordType"></i>
                </div>
            </div>
            <div class="submitBtn" @click="login">登录</div>
        </div>
    </div>
</template>
<script>
import { OrderService } from '@/services';
import { execNative } from '@/services/utils';

export default {
    data() {
        return {
            param: {
                username: '',
                password: '',
            },
            inputType: 'password', // text
        };
    },
    methods: {
        login() {
            if (!this.param.username) {
                execNative('showToast', { content: '用户名不能为空' });
                return;
            } else if (!this.param.password) {
                execNative('showToast', { content: '密码不能为空' });
                return;
            } else if (this.param.password.length < 7 || this.param.password.length > 32) {
                execNative('showToast', { content: '密码长度在7到32之间' });
                return;
            }
            OrderService.login(this.param).then((res) => {
                console.log('登陆res', res);
                if (res) { // 登录成功去登出页面 返不返回openid
                    const id = localStorage.getItem('wxParamsTianyan');
                    const openid = id ? JSON.parse(id).openid : '';
                    localStorage.setItem('wxParamsTianyan', JSON.stringify({ openid, ...res }));
                    // this.$router.back(-1); // 返回上一页
                    setTimeout(() => {
                        this.$router.replace({ name: 'index', query: { code: '' ,openid}}); // 去登出页面
                    }, 100);
                }
            }).catch((err) => {
                console.log('登录失败', err);
            });
        },
        togglePasswordType() {
            if (this.inputType === 'password') {
                this.inputType = 'text';
            } else {
                this.inputType = 'password';
            }
        },
    },
};
</script>
<style lang="less">
    .log-in{
        font-size: .3rem;
        .van-cell{
            background: none;
        }
        img{
            width:100%;
        }
        .logo{
            width:1.6rem;
            margin:1.2rem auto;
        }
        .bd{
            padding: 0 .32rem;
            .icon{
                font-size: .44rem;
                padding-left:.38rem;
                color:#666666;
            }
        }
        .searchInn{
            position:relative;
            width: 100%;
            .icon{
                position: absolute;
                top: 50%;
                transform: translate(0,-50%);
            }
            .searchBox{
                display: block;
                box-sizing: border-box;
                width: 100%;
                min-width: 0;
                margin: 0;
                padding: 0;
                color: #323233;
                line-height: .64rem;
                border: 0;
                background: none;
                border-radius: .4rem;
                padding-left: 1rem;
            }
        }

        .item{
            height:1.1rem;
            background: #FAFAFA;
            box-shadow: inset 0 1px 3px 0 rgba(0,0,0,0.15);
            border-radius: 5px;
            display: flex;
            align-items: center;
            position: relative;
            &:first-child{
                margin-bottom:.4rem;
            }
        }
        .submitBtn{
            background: #5A32D2;
            box-shadow: 0 3px 6px 0 rgba(90,50,210,0.40);
            border-radius: 5px;
            height:1.1rem;
            line-height: 1.1rem;
            text-align: center;
            color:#fff;
            margin-top:.8rem;
        }
        .eye{
            position: absolute;
            right:.38rem;
            top:50%;
            font-size: .45rem;
            transform: translate(0, -50%);
            color:#5A32D2;
            &.icon-yulan{
                font-size: .3rem;
            }
        }
    }
</style>