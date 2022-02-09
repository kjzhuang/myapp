<template>
    <div class="scan">
        <div class="hd"><img src="../assets/imgs/sign.png" alt=""></div>
         <div class="bd">
            <div class="item">
                <i class="iconfont icon-chaxun"></i>
                <div class="field">
                    <van-field v-model="scanModel" label="" placeholder="请输入设备号或扫描设备条形码" />
                </div>
                <div @click="scanGpsno" class="scanBtn">
                    <img src="../assets/imgs/scan.png" alt="">
                </div>
            </div>
            <div @click="search" class="submitBtn">搜索</div>
        </div>
    </div>
</template>
<script>
import _ from 'lodash';
import { execNative } from '@/services/utils';
import { OrderService } from '@/services';

export default {
    data() {
        return {
            scanModel: '',
        };
    },
    mounted() {
        this.initWx();
    },
    methods: {
        async initWx() {
            const data = await OrderService.getSignPackage({
                url: encodeURIComponent(window.location.href.split('#')[0])
                // url: window.location.href.split('#')[0],
            });
            window.wx.config({
                debug: false,
                appId: data.appId,
                timestamp: data.timestamp,
                nonceStr: data.nonceStr,
                signature: data.signature,
                jsApiList: ['scanQRCode', 'startRecord', 'stopRecord'],
            })
        },
        scanGpsno() {
            const that = this;
            window.wx.ready(() => {
                console.log('ready')
                window.wx.scanQRCode({
                    needResult: 1,
                    scanType: ['qrCode', 'barCode'],
                    success: function(res) {
                        const result = res.resultStr.split(',');
                        const ordercode = !!result[1] ? result[1] : result[0];
                        that.scanModel = ordercode;
                    }
                });
            });
        },
        search: _.throttle(async function() {
            this.scanModel = this.scanModel.replace(/(^\s*)|(\s*$)/g, '');
            if (!this.scanModel) {
                execNative('showToast', { content: '设备号不能为空' });
                return;
            }
            this.$router.push({ 
                name: 'scanDetail',
                query: { deviceno: this.scanModel },
            });

        }, 500),
    },
};
</script>
<style lang="less">
    .scan{
        font-size: .3rem;
        .van-cell{
            background: none;
        }
        img{
            width:100%;
        }
        .hd{
            width:1.68rem;
            margin:1rem auto;
        }
        .bd{
            padding: 0 .32rem;
            i{
                font-size: .4rem;
                padding-left:.3rem;
                color:#666666;
            }
        }
        .item{
            height:1.1rem;
            background: #FAFAFA;
            box-shadow: inset 0 1px 3px 0 rgba(0,0,0,0.15);
            border-radius: 5px;
            display: flex;
            align-items: center;
            &:first-child{
                margin-bottom:.4rem;
            }
        }
        .field{flex: 1;}
        .scanBtn{
            padding-right:.3rem;
            img{
                width:.4rem;
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
    }
</style>