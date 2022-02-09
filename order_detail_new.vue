<template>
    <div class="device-detail">
        <van-collapse v-model="active">
            <van-collapse-item title="订单基本信息" name="1">
                <div class="item"><span class="black nowrap">订单编号：</span>{{info.orderno || '--'}}</div>
                <div class="item"><span class="black nowrap">客户名称：</span>{{info.customer || '--'}}</div>
                <div class="item"><span class="black nowrap">创建时间：</span>{{info.createtime || '--'}}</div>
                <div class="item"><span class="black nowrap">所属机构：</span>{{info.orgcode || '--'}}</div>

                <div class="item"><span class="black nowrap">始发站点：</span>{{info.startstation || '--'}}</div>
                <div class="item"><span class="black nowrap">预计发货时间：</span>{{info.estimatestarttime || '--'}}</div>
                <div class="item"><span class="black nowrap">实际发货时间：</span>{{info.starttime || '--'}}</div>
                <div class="item"><span class="black nowrap">目的站点：</span>{{info.endstation || '--'}}</div>
                <div class="item"><span class="black nowrap">预计到达时间：</span>{{info.estimateendtime || '--'}}</div>
                <div class="item"><span class="black nowrap">实际到达时间：</span>{{info.endtime || '--'}}</div>

                <div class="item"><span class="black nowrap">状态：</span>{{statusMap.get(info.status) || '--'}}</div>

                <div class="item"><span class="black nowrap">集装箱数量：</span>{{info.containercount || '--'}}</div>
                <div class="item"><span class="black nowrap">是否配置邮件推送：</span>{{issettingemailMap.get(info.issettingemail) || '--'}}</div>
            </van-collapse-item>
            <van-collapse-item :name="index" v-for="(option,index) in info.taskList">
                <template #title>
                    <div class="flex-center-between">
                        <div>集装箱{{index+1}}：{{option.containerno}}</div>
                        <div v-html="getContainerStatus(option.status)"></div>
                    </div>
                </template>
                <div class="item flex-center-between"><span class="black nowrap">设备号：{{option.gpsno || '--'}} </span><span v-if="option.gpsno" style="color:#5A32D2" @click="toDeviceDetail(option)">查看设备详情></span></div>
                <div class="item"><span class="black nowrap">绑定时间：</span>{{option.binddevicetime || '--'}}</div>

                <div v-if="option.deviceInfo && option.deviceInfo.time" class="item"><span class="black nowrap">更新时间：</span>{{option.deviceInfo.time || '--'}}</div>
                <div v-if="option.deviceInfo && option.deviceInfo.temperature" class="item"><span class="black nowrap">温度：</span><span :class="option.deviceInfo.temperature.alarmstatus ? 'red' : ''">{{option.deviceInfo.temperature.value}} {{option.deviceInfo.temperature.alarmnum>0?`(${option.deviceInfo.temperature.alarmnum}次异常)`:''}}</span></div>
                <div v-if="option.deviceInfo && option.deviceInfo.humidity" class="item"><span class="black nowrap">湿度：</span><span :class="option.deviceInfo.humidity.alarmstatus ? 'red' : ''">{{option.deviceInfo.humidity.value}} {{option.deviceInfo.humidity.alarmnum>0?`(${option.deviceInfo.humidity.alarmnum}次异常)`:''}}</span></div>
                <div v-if="option.deviceInfo && option.deviceInfo.rock" class="item"><span class="black nowrap">震动：</span><span :class="option.deviceInfo.rock.alarmstatus ? 'red' : ''">{{option.deviceInfo.rock.value}} {{option.deviceInfo.rock.alarmnum>0?`(${option.deviceInfo.rock.alarmnum}次异常)`:''}}</span></div>
                <div v-if="option.deviceInfo && option.deviceInfo.angle" class="item"><span class="black nowrap">倾斜：</span><span :class="option.deviceInfo.angle.alarmstatus ? 'red' : ''">{{option.deviceInfo.angle.value}} {{option.deviceInfo.angle.alarmnum>0?`(${option.deviceInfo.angle.alarmnum}次异常)`:''}}</span></div>
                <div v-if="option.deviceInfo && option.deviceInfo.light" class="item"><span class="black nowrap">光感：</span><span :class="option.deviceInfo.light.alarmstatus ? 'red' : ''">{{option.deviceInfo.light.value}} {{option.deviceInfo.light.alarmnum>0?`(${option.deviceInfo.light.alarmnum}次异常)`:''}}</span></div>
                <div v-if="option.deviceInfo && option.deviceInfo.door" class="item"><span class="black nowrap">门磁：</span><span :class="option.deviceInfo.door.alarmstatus ? 'red' : ''">{{option.deviceInfo.door.value}} {{option.deviceInfo.door.alarmnum>0?`(${option.deviceInfo.door.alarmnum}次异常)`:''}}</span></div>
                <div v-if="option.deviceInfo && option.deviceInfo.external_switch" class="item"><span class="black nowrap">开关门：</span><span :class="option.deviceInfo.external_switch.alarmstatus ? 'red' : ''">{{option.deviceInfo.external_switch.value}} {{option.deviceInfo.external_switch.alarmnum>0?`(${option.deviceInfo.external_switch.alarmnum}次异常)`:''}}</span></div>
                <div v-if="option.deviceInfo && option.deviceInfo.energy" class="item"><span class="black nowrap">剩余电量：</span><span :class="option.deviceInfo.energy&&option.deviceInfo.energy.split('%')[0]&&option.deviceInfo.energy.split('%')[0]<30 ? 'red' : ''">{{option.deviceInfo.energy || '--'}}</span></div>
                <div v-if="option.deviceInfo && option.deviceInfo.address" class="item"><span class="black nowrap">当前位置：</span>{{option.deviceInfo.address || '--'}}</div>

                <div class="item"><span class="black nowrap">下个站点：</span>{{option.nextstation || '--'}}</div>
                <div class="item"><span class="black nowrap">距离下个站点：</span>{{option.distancetonext || '--'}} km</div>
                <div class="item"><span class="black nowrap">距离目的地站点：</span>{{option.distancetodest || '--'}} km</div>
                <!-- <div class="item"><span class="black">状态</span>：<span v-html="getContainerStatus(option.status)"></span></div> -->
                <!-- <div class="item "><span class="black">历史轨迹：</span><span @click="toTrackBack(option)" style="color:#5A32D2">查看</span></div> -->
            </van-collapse-item>
        </van-collapse>
        <div class="btns">
            <div @click="toTrackBack(info)" class="historyBtn btn">查看历史轨迹</div>
            <div @click="goAlarm" class="alarmBtn btn">查看报警事件 {{alarmnum>0?` ( ${alarmnum}条) `:''}}</div>
        </div>
    </div>
</template>
<script>
import dayjs from 'dayjs';
import {OrderApiService} from '@/services'
import { Collapse, CollapseItem } from 'vant';
Vue.use(Collapse);
Vue.use(CollapseItem);
export default {
    data() {
        return {
            alarmnum: 0,
            active: ['1','2','3','4'],
            info: {},
            statusMap: new Map([[0,'未开始'],[1,'已开始'],[2,'已完成'],[3,'部分开始']]),
            issettingemailMap: new Map([[0,'否'],[1,'是']])
        };
    },
    beforeRouteEnter: (to, from, next) => {
        next((vm) => {
            const id = vm.$route.query.id;
            if (id) {
                vm.active = ['1']
                vm.getColumn(id); // 获取详情能力值
            }
        });
    },
    methods: {
        formatTime (time) {
            return time?dayjs(time).format('YYYY-MM-DD HH:mm:ss'):'--'
        },
        getContainerStatus (status) {
            if (status == 0) {
                return `<span class="gray">未开始</span>`
            } else if (status == 1) {
                // 判断是否延误
                if (this.info.estimateendtime) {
                    let rate = new Date(this.info.estimateendtime.replace(/\-/g, '/')).getTime()
                    let current = new Date().getTime()
                    let diff = current - rate
                    if (diff<0) {
                        // 没有延误
                        return `<span class="green">正常运输中</span>`
                    } else {
                        // 延误
                        return `<span class="red">已延误</span>`
                    }
                } else {
                    // 未设置预计达到,则运输中
                    return `<span class="green">正常运输中</span>`
                }
            } else if (status == 2) {
                return `<span class="blue">已到达</span>`
            }
        },
        toDeviceDetail(info){
            this.$router.push({
                name: 'deviceDetail',
                query: { deviceno: info.gpsno ,id: info.deviceid},
            });
        },
        // 跳转到订单查询页面
        toTrackBack (info) {
            if (window.location.hostname.indexOf('order-mobile.huoyunren') > -1) {
                window.location.href = `https://g7s.huoyunren.com/product/order/railwayOrder/#/positionQueryDetail?container=${info.orderno}&lang=zh-cn`; // 线上
            } else {
                window.location.href = `https://test.g7s.chinawayltd.com/product/order/railwayOrder/#/positionQueryDetail?container=${info.orderno}&lang=zh-cn`;
            }
        },
        /** 获取相应能力的列 */
        getColumn(id) {
            this.info = {}
            this.alarmnum = 0
            OrderApiService.getOrderById({orderId: id}).then((res) => {
                if (res) {
                    if (res.taskList) {
                        res.taskList.map(e=>{
                            // 加载当前任务的设备号
                            this.getDeviceInfoByTaskId(e.id,e)
                            // 去加载属于当前task的设备信息和地址
                            if (e && e.lastloclat && e.lastloclng) {
                                OrderApiService.getAddress({lng:e.lastloclng,lat:e.lastloclat}).then((add) => {
                                    if (add) {
                                        this.$set(e,'address',add)
                                    }
                                })
                            }
                        })
                    }
                    this.info = res;
                }
            }).catch((err) => {
                console.log('获取所有设备号失败', err);
            });
        },
        goHistory() {
            this.$router.push({
                name: 'historyData',
                query: { deviceno: this.info.secode },
            });
        },
        // 获取设备详情
        getDeviceInfoByTaskId(id,option) {
            OrderApiService.getDeviceInfoByTaskId({taskid: id }).then((res) => {
                if (res) {
                    this.$set(option,'deviceInfo',res)
                    this.alarmnum += res.alarmnum?res.alarmnum:0
                } else {
                    this.$set(option,'deviceInfo',{})
                }
            }).catch((err) => {
                console.log('获取所有设备详情失败', err);
                this.$set(option,'deviceInfo',res)
            });
        },
        goAlarm () {
            this.$router.push({
                name: 'alarmList',
                query: { type: 1 ,keyword: this.info.orderno},
            });
        },
    },
};
</script>
<style lang="less">
    .device-detail{
        height: 100vh;
        color:#0A0A0A;
        font-size:.28rem;
        position: relative;
        padding:.32rem;
        .black{
            color: #323233;
        }
        .gray{
            color: #666666;
        }
        .red{
            color: #D7142E;
        }
        .blue{
            color: #5A32D2;
        }
        .green{
            color: green;
        }
        .nowrap{
            white-space: nowrap;
        }
        .bd{
            border-top:1px dashed #DEDDE9;
            border-bottom:1px dashed #DEDDE9;
            padding:.3rem 0 .12rem 0;
        }
        .item{
            padding-bottom:.18rem;
            position: relative;
            box-sizing: border-box;
            display: flex;
            align-items: center;
             word-break: break-all;
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
        .van-cell::after{
        content: none;
    }
    .van-collapse{
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 1.4rem;
        overflow: auto;
        margin-bottom: 0.12rem;
    }
    .van-collapse-item--border::after{
        content: none;
    }
    .van-hairline--top-bottom::after, .van-hairline-unset--top-bottom::after{
        content: none;
    }
    .van-collapse-item__content{
        padding-top: 0;
        font-size: 0.26rem !important;
        background: #f2f2f4;
        padding-top: 20px;
    }
    .van-cell__title{
        font-size: 0.3rem;
        font-weight: 600;
    }
    }
    .table_th{
        padding: 0.1rem 0;
    }
    .table_td{
        padding: 0.1rem 0;
        border-bottom: #dcdcdc solid 1px;
        color:#323233;font-size:0.28rem;
    }
</style>