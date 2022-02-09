<template>
    <div class="device-detail">
        <div class="tp">
            <div class="item"><span class="gray">设备号：</span>{{info.deviceno}}</div>
            <div class="item"><span class="gray">电量：</span>{{info.energy}}%</div>
            <div class="item">
                <span class="gray">设备上报频率：</span>{{info.repfrequency ? info.repfrequency : '--'}}分/次
                <!-- <span class="modifyBtn blue" @click="dialogShow = true"><i class="iconfont icon-bianji"></i>修改</span> -->
            </div>
            <div class="item"><span class="gray">设备类型：</span>{{info.devtypename}}</div>
            <div class="item"><span class="gray">设备购买时间：</span>{{info.gsp_createtime}}</div>
            <div class="item"><span class="gray">服务到期日期：</span>{{info.serviceexpiration}}</div>
            <div class="item"><span class="gray">所属机构：</span>{{info.orgname}}</div>
        </div>
        <div class="bd">
            <div class="item flexStart">
                <span class="gray flexL">绑定订单：</span><span class="flexR">{{info.bindOrders}}</span>
                <!-- <span @click="goOrderDetail(info.dtype, info.deviceno)" class="modifyBtn blue">查看详情<i class="iconfont icon-ic_arrow_gray_right"></i></span> -->
            </div>
            <div class="item"><span class="gray">定位时间：</span>{{info.locationtime}}</div>
            <div class="item" style="line-height: .48rem;"><span class="gray" style="white-space: nowrap;">定位位置：</span>{{info.location}}</div>
        </div>
        <div class="ft">
            <div v-if="columnObj.temperature" class="item"><span class="gray">温度：</span><span :class="info.tempStatus ? 'red' : ''">{{info.temperature === 0 || info.temperature ? info.temperature + '℃' : '--'}}</span></div>
            <div v-if="columnObj.rock" class="item"><span class="gray">震动：</span><span :class="info.vibrationStatus ? 'red' : ''">{{info.rock === 0 || info.rock ? info.rock + 'g' : '--'}}</span></div>
            <div v-if="columnObj.exposure" class="item"><span class="gray">光感：</span><span :class="info.lightStatus ? 'red' : ''">{{info.exposure === 0 || info.exposure ? info.exposure + 'LUX' : '--'}}</span></div>
            <div v-if="columnObj.door_magnetism" class="item"><span class="gray">门磁：</span>{{info.door_magnetism ? info.door_magnetism : '--'}}</div>
            <div v-if="columnObj.humidity" class="item"><span class="gray">湿度：</span><span :class="info.humidityStatus ? 'red' : ''">{{info.humidity === 0 || info.humidity ? info.humidity + '%RH' : '--'}}</span></div>
            <div v-if="columnObj.angle" class="item"><span class="gray">倾斜：</span><span :class="info.tiltStatus ? 'red' : ''">{{info.angle === 0 || info.angle ? info.angle + '°' : '--'}}</span></div>
            <div v-if="columnObj.external_switch" class="item"><span class="gray">外开关：</span>{{info.external_switch == 1 ? '开' : info.external_switch == -1 ? '--' : '关'}}</div>
            <div v-if="columnObj.location_mode" class="item"><span class="gray">定位模式：</span>{{info.location_mode ? info.location_mode : '--'}}</div>
        </div>
        <div class="btns">
            <div @click="goHistory" class="historyBtn btn">查看历史数据</div>
            <!-- <div class="alarmBtn btn">查看报警事件</div> -->
        </div>
        <van-dialog v-model="dialogShow" title="设备上报频率修改" show-cancel-button>
            <div class="inputBox">
                <van-field v-model="inputModel" label="" placeholder="请输入" />
                <span class="unit">分/次</span>
            </div>   
        </van-dialog>
    </div>
</template>
<script>
import { Dialog } from 'vant';
import { OrderService } from '@/services';

export default {
    data() {
        return {
            dialogShow: false,
            inputModel: '',
            info: {},
            columnObj: { // 相应设备号对应的能力
                temperature: false, // 温度能力
                humidity: false, // 湿度能力
                rock: false, // 震动
                angle: false, // 倾斜
                exposure: false, // 光感
                external_switch: false, // 外开关
                door_magnetism: false, // 门磁
                location_mode: false, // 定位模式
            },
        };
    },
    components: { [Dialog.Component.name]: Dialog.Component },
    mounted() {
    },
    beforeRouteEnter: (to, from, next) => {
        next((vm) => {
            const DEVICENO = vm.$route.query.deviceno;
            if (DEVICENO) {
                vm.getColumn(DEVICENO); // 获取详情能力值
            }
        });
    },
    methods: {
        /** 获取相应能力的列 */
        getColumn(deviceno) {
            OrderService.getColumn({ deviceno }).then((res) => {
                if (res) {
                    this.getDetail(); // 得到表格数据接口
                    this.columnObj = res;
                }
            }).catch((err) => {
                console.log('获取所有设备号失败', err);
            });
        },
        getDetail() {
            const id = this.$route.query.id;
            const  deviceno = this.$route.query.deviceno;
            if (!id) {
                OrderService.getDeviceDetailByDeviceNo({ deviceno }).then((res) => {
                    if (res) {
                        this.info = res;
                    } else {
                        this.info = {};
                    }
                }).catch((err) => {
                    console.log('获取所有设备号失败', err);
                    this.info = {};
                });
            } else {
                OrderService.getDeviceDetail({ id }).then((res) => {
                if (res) {
                    this.info = res;
                } else {
                    this.info = {};
                }
            }).catch((err) => {
                console.log('获取所有设备号失败', err);
                this.info = {};
            });
            }
            
        },
        goHistory() {
            this.$router.push({
                name: 'historyData',
                query: { deviceno: this.info.deviceno },
            });
        },
        goOrderDetail(type, deviceno) {
            // this.$router.push({
            //     name: 'deviceDetail',
            //     query: { id },
            // });
            if (window.location.hostname.indexOf('order-mobile.huoyunren') > -1) {
                window.location.href = `http://g7s.huoyunren.com/product/order/wechat/order/page/infoDetail.html?type=${type}&deviceno=${deviceno}`; // 线上
            } else {
                window.location.href = `http://demo.g7s.chinawayltd.com/product/order/wechat/order/page/infoDetail.html?type=${type}&deviceno=${deviceno}`;
            }
        },
    },
};
</script>
<style lang="less">
    .device-detail{
        color:#0A0A0A;
        font-size:.28rem;
        padding:.32rem;
        .gray{
            color: #666666;
        }
        .red{
            color: #D7142E;
        }
        .blue{
            color: #5A32D2;
        }
        .bd{
            border-top:1px dashed #DEDDE9;
            border-bottom:1px dashed #DEDDE9;
            padding:.3rem 0 .12rem 0;
        }
        .item{
            padding-bottom:.18rem;
            position: relative;
        }
        .tp{
            padding-bottom:.12rem;
        }
        .ft{
            padding-top:.3rem;
            display:flex;
            align-items: center;
            flex-wrap: wrap;
            .item{
                width:50%;
            }
        }
        .flexStart{
            display: flex;
            align-items: center;
        }
        .flexL{
            width: 1.5rem;
        }
        .flexR{
            word-break: break-all;
            flex: 1;
        }
        .modifyBtn{
            position: absolute;
            right:0;
            top:50%;
            transform: translate(0, -50%);
            padding-bottom: .18rem;
            i{
                padding-right:.1rem;
                padding-left:.1rem;
                &.icon-ic_arrow_gray_right{padding-right:0; font-size: 12px;}
            }
        }
        .btns{
            position: fixed;
            bottom:.4rem;
            left:0;
            display: flex;
            // padding:0 .32rem;
            width: 100%;
            .btn{
                background: #5A32D2;
                box-shadow: 0 3px 6px 0 rgba(90,50,210,0.40);
                border-radius: 5px;
                color:#fff;
                height: 1rem;
                line-height: 1rem;
                text-align: center;
                font-size: .28rem;
                margin-left: .32rem;
                // width:50%;
                width:100%;
                &:last-child{margin-right:.32rem;}
            }
        }
        .inputBox{
            display: flex;
            align-items: center;
            border: 1px solid #DEDDE9;
            border-radius: 2px;
            margin: .6rem .4rem;
        }
        .van-dialog__header{
            padding: .3rem .32rem;
            background: #5A32D2;
            color: #fff;
            line-height: 1;
            // text-align: left;
        }
        .van-dialog__confirm, .van-dialog__confirm:active{
            color:#5A32D2;
        }
        .unit{
            width:1.2rem;
        }
    }
</style>