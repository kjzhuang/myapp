<template>
    <div class="device-detail">
        <van-collapse v-model="active">
            <van-collapse-item title="设备实时信息" name="1">
                <div class="item flex-center-between"><span class="black nowrap">设备号：{{info.secode || '--'}} </span><span v-if="info.secode" style="color:#5A32D2" @click="toDeviceDetail(info)">查看设备详情></span></div>
                <div class="item"><span class="black nowrap">数据更新时间：</span>{{deviceInfo.time || '--'}}</div>
                
                <div v-if="deviceInfo.temperature" class="item"><span class="black nowrap">温度：</span><span :class="deviceInfo.temperature.alarmstatus ? 'red' : ''">{{deviceInfo.temperature.value}} {{deviceInfo.temperature.alarmnum>0?`(${deviceInfo.temperature.alarmnum}次异常)`:''}}</span></div>
                <div v-if="deviceInfo.humidity" class="item"><span class="black nowrap">湿度：</span><span :class="deviceInfo.humidity.alarmstatus ? 'red' : ''">{{deviceInfo.humidity.value}} {{deviceInfo.humidity.alarmnum>0?`(${deviceInfo.humidity.alarmnum}次异常)`:''}}</span></div>
                <div v-if="deviceInfo.rock" class="item"><span class="black nowrap">震动：</span><span :class="deviceInfo.rock.alarmstatus ? 'red' : ''">{{deviceInfo.rock.value}} {{deviceInfo.rock.alarmnum>0?`(${deviceInfo.rock.alarmnum}次异常)`:''}}</span></div>
                <div v-if="deviceInfo.angle" class="item"><span class="black nowrap">倾斜：</span><span :class="deviceInfo.angle.alarmstatus ? 'red' : ''">{{deviceInfo.angle.value}} {{deviceInfo.angle.alarmnum>0?`(${deviceInfo.angle.alarmnum}次异常)`:''}}</span></div>
                <div v-if="deviceInfo.light" class="item"><span class="black nowrap">光感：</span><span :class="deviceInfo.light.alarmstatus ? 'red' : ''">{{deviceInfo.light.value}} {{deviceInfo.light.alarmnum>0?`(${deviceInfo.light.alarmnum}次异常)`:''}}</span></div>
                <div v-if="deviceInfo.door" class="item"><span class="black nowrap">门磁：</span><span :class="deviceInfo.door.alarmstatus ? 'red' : ''">{{deviceInfo.door.value}} {{deviceInfo.door.alarmnum>0?`(${deviceInfo.door.alarmnum}次异常)`:''}}</span></div>
                <div v-if="deviceInfo.external_switch" class="item"><span class="black nowrap">开关门：</span><span :class="deviceInfo.external_switch.alarmstatus ? 'red' : ''">{{deviceInfo.external_switch.value}} {{deviceInfo.external_switch.alarmnum>0?`(${deviceInfo.external_switch.alarmnum}次异常)`:''}}</span></div>
                <div class="item"><span class="black nowrap">剩余电量：</span><span :class="deviceInfo.energy&&deviceInfo.energy.split('%')[0]&&deviceInfo.energy.split('%')[0]<30 ? 'red' : ''">{{deviceInfo.energy || '--'}}</span></div>


                <div class="item"><span class="black nowrap">最新位置：</span>{{deviceInfo.address || '--'}}</div>

            </van-collapse-item>
            <van-collapse-item title="订单基本信息" name="2">
                <div class="item"><span class="black nowrap">订单编号：</span>{{info.orderno || '--'}}</div>
                <div class="item"><span class="black nowrap">客户名称：</span>{{info.customname || '--'}}</div>
                <div class="item"><span class="black nowrap">业务单号：</span>{{info.wmsno || '--'}}</div>
                <div class="item"><span class="black nowrap">项目编号：</span>{{info.projectnum || '--'}}</div>
                <div class="item"><span class="black nowrap">项目名称：</span>{{info.projectname || '--'}}</div>
                <div class="item"><span class="black nowrap">创建时间：</span>{{info.createtime || '--'}}</div>
                <div class="item"><span class="black nowrap">所属机构：</span>{{info.orgcode}}</div>
                <div class="item"><span class="black nowrap">是否绑定设备：</span>{{sebindstatusMap.get(info.sebindstatus*1)  || '--'}}</div>
                <div class="item"><span class="black nowrap">绑定时间：</span>{{info.sebindtime  || '--'}}</div>
                <div class="item"><span class="black nowrap">解绑时间：</span>{{info.seunbindtime  || '--'}}</div>
                <div class="item"><span class="black nowrap">状态：</span>{{statusMap.get(info.currentstatus*1)  || '--'}}</div>
                <div class="item"><span class="black nowrap">开始时间：</span>{{info.starttime || '--'}}</div>
                <div class="item"><span class="black nowrap">完成时间：</span>{{info.arrivetime || '--'}}</div>
            </van-collapse-item>
            <van-collapse-item title="收发货信息" name="3">
                <div style="color:#323233;font-size:0.3rem;margin-bottom:0.18rem;font-weight:500;">发货信息</div>
                <div class="item"><span class="black nowrap">发货人：</span>{{info.sname || '--'}}</div>
                <div class="item"><span class="black nowrap">发货单位：</span>{{info.scompany || '--'}}</div>
                <div class="item"><span class="black nowrap">发货人手机：</span>{{info.sphone || '--'}}</div>
                <div class="item"><span class="black nowrap">发货地址：</span>{{info.slocation || '--'}}</div>
                <div class="item"><span class="black nowrap">发货站点：</span>{{info.ssitename || '--'}}</div>
                <div class="item"><span class="black nowrap">计划提货时间：</span>{{info.sdatetime || '--'}}</div>
                <div class="item"><span class="black nowrap">实际发车时间：</span>{{info.starttime || '--'}}</div>
                <div style="color:#323233;font-size:0.3rem;margin-bottom:0.18rem;font-weight:500;">收货信息</div>
                <div class="item"><span class="black nowrap">收货人：</span>{{info.rname || '--'}}</div>
                <div class="item"><span class="black nowrap">收货单位：</span>{{info.rcompany || '--'}}</div>
                <div class="item"><span class="black nowrap">收货人手机：</span>{{info.rphone || '--'}}</div>
                <div class="item"><span class="black nowrap">收货地址：</span>{{info.rlocation || '--'}}</div>
                <div class="item"><span class="black nowrap">收货站点：</span>{{info.rsitename || '--'}}</div>
                <div class="item"><span class="black nowrap">计划送达时间：</span>{{info.rdatetime || '--'}}</div>
                <div class="item"><span class="black nowrap">实际到达时间：</span>{{info.arrivetime || '--'}} <span v-if="!info.arrivetime && info.rdatetime" v-html="getDelay(info.rdatetime)"></span></div>
            </van-collapse-item>
            <van-collapse-item title="节点信息" name="4">
                <div class="item"><span class="black nowrap">当前位置：</span>{{info.currentposition || '--'}}</div>
                <div class="item"><span class="black nowrap">当前位置更新时间：</span>{{info.currenttime || '--'}}</div>
                <!-- 展示节点 -->
                <div v-for="(item,index) in info.milestonesites" :key="item.mid">
                    <!-- 非目的站点 -->
                    <div v-if="item.mcode != 'ddmdd'">
                        <div style="color:#323233;font-size:0.3rem;margin-bottom:0.18rem;font-weight:500;">节点{{index+1}}</div>
                        <div class="item"><span class="black nowrap">{{item.mname}}：</span>{{item.sitename || '暂无信息'}}</div>
                        <div class="item"><span class="black nowrap">{{item.direct==1?'到达':'离开'}}时间：</span>{{item.mtime || '暂无信息'}}</div>
                    </div>
                    <!-- 目的站点 -->
                    <div v-if="item.mcode == 'ddmdd'">
                        <div style="color:#323233;font-size:0.3rem;margin-bottom:0.18rem;font-weight:500;">节点{{index+1}}</div>
                        <!-- 如果配置了目的地站点,就取目的地站点名称,及预计到达目的地站点的直线距离.如果没有配置目的地站点,配置了收货地址的,就取收货地址. 计算到收货地址的距离. -->
                        <!-- 如果mtime存在,代表目的站点已经到达,则不显示距离站点距离信息 -->
                        <div class="item"><span class="black nowrap">{{item.mname}}：</span>
                            {{`${item.sitename?item.sitename:item.maddress?item.maddress:'暂无信息'}`}}
                            <span v-if="!item.mtime && item.dest">(距离该站点{{item.dest}}km)</span>
                        </div>
                        <!-- 如果mtime存在,则取mtime,如果不存在,则去预计到达时间,并计算当前时间差,精确到天,超过了时间则展示 -->
                        <div class="item"><span class="black nowrap">
                            {{item.direct?item.direct==1?'到达':'离开':'到达'}}时间：</span><span v-html="getMDDtime(item,info)"></span>
                        </div>
                    </div>
                </div>
            </van-collapse-item>
            <van-collapse-item title="货品信息" name="5">
                <div class="table_th flex-center-between">
                    <div style="width:28%">货品名称</div>
                    <div style="width:18%">货品单位</div>
                    <div style="width:18%">货品数量</div>
                    <div style="width:18%">货品总量</div>
                    <div style="width:18%">货品总体</div>
                </div>
                <div v-for="option in info.goods" class="table_td flex-center-between">
                    <div style="width:28%">{{option.goodname}}</div>
                    <div style="width:18%">{{option.unit}}</div>
                    <div style="width:18%">{{option.number}}</div>
                    <div style="width:18%">{{option.weight}}</div>
                    <div style="width:18%">{{option.volume}}</div>
                </div>
            </van-collapse-item>
        </van-collapse>

        <div class="btns">
            <div @click="goHistory" v-if="info.secode" class="historyBtn btn">查看历史数据</div>
            <div @click="goAlarm" class="alarmBtn btn">查看报警事件 {{alarmCount>0?` ( ${alarmCount}条) `:''}}</div>
        </div>
    </div>
</template>
<script>
import axios from 'axios'
import dayjs from 'dayjs';
import {OrderApiService} from '@/services'
import { OrderService } from '@/services';
import { Collapse, CollapseItem } from 'vant';
Vue.use(Collapse);
Vue.use(CollapseItem);
export default {
    data() {
        return {
            active: ['1','2'],
            info: {},
            alarmCount: 0,
            deviceInfo: {},
            devicecolumnObj: {},
            statusMap: new Map([[0,'未开始'],[1,'已开始'],[2,'已完成'],[3,'部分开始']]),
            sebindstatusMap: new Map([[0,'未绑定'],[1,'已绑定'],[2,'已解绑']])
        };
    },
    // beforeRouteEnter: (to, from, next) => {
    //     next((vm) => {
    //         const id = vm.$route.query.id;
    //         if (id) {
    //             vm.active = ['1']
    //             vm.getColumn(id);
    //         }
    //     });
    // },
    created() {
        const id = this.$route.query.id;
        // 获取订单详情
        this.getColumn(id)
        // 获取设备详情
        this.getDeviceInfoByOrderId(id)
    },
    methods: {
        toDeviceDetail(info){
            this.$router.push({
                name: 'deviceDetail',
                query: { deviceno: info.secode ,id: info.deviceid},
            });
        },
        getDelay (time) {
            if (!time) return ''
            let rate = new Date(time.replace(/\-/g, '/')).getTime()
            let current = new Date().getTime()
            let diff = current - rate
            let delay = ''
            if (diff<0) {
                delay = ''
            } else {
                diff = Math.floor(diff/1000/3600/24)
                delay = diff>0?`<span class="red">(延误${diff}天)</span>`:`<span class="red">已延误</span>`
            }
            return delay
        },
        getMDDtime (item,info) {
            let time = ''
            let delay = ''
            if (item.mtime) {
                return item.mtime
            } else {
                if (!info.rdatetime) {
                    return '暂无信息'
                } else {
                    // 计算当前时间和预计到达时间差
                    let rate = new Date(info.rdatetime.replace(/\-/g, '/')).getTime()
                    let current = new Date().getTime()
                    let diff = current - rate
                    if (diff<0) {
                        // 没有延误
                        time = '预计到达时间 ' + info.rdatetime
                        delay = ''
                    } else {
                        time = '预计到达时间 ' + info.rdatetime
                        diff = Math.floor(diff/1000/3600/24)
                        delay = diff>0?`(延误${diff}天)`:'(已延误)'
                    }
                    return time + ' ' + `<span class="red">${delay}</span>`
                }
            }
        },
        // 查询订单详情
        getColumn(id) {
            // 清空页面
            this.info = {}
            OrderApiService.getOrderById({orderId: id}).then((res) => {
                if (res) {
                    this.info = res;
                }
            }).catch((err) => {
                console.log('订单详情加载失败', err);
                this.loading = false
            });
        },
        goHistory() {
            // 订单开始时间,不存在则为空
            let str = this.info.sebindtime?new Date(this.info.sebindtime.replace(/\-/g, '/')).getTime(): null
            // 订单结束时间,不存在则取当前时间
            let end = this.info.seunbindtime?new Date(this.info.seunbindtime.replace(/\-/g, '/')).getTime():new Date().getTime()
            this.$router.push({
                name: 'historyData',
                query: { deviceno: this.info.secode, from: 'order',str,end },
            });
        },
        goAlarm () {
            this.$router.push({
                name: 'alarmList',
                query: { type: 1 ,keyword: this.info.orderno},
            });
        },
        // 获取设备详情
        getDeviceInfoByOrderId(id) {
            this.alarmCount = 0
            OrderApiService.getDeviceInfoByOrderId({orderid: id }).then((res) => {
                if (res) {
                    this.deviceInfo = res;
                    this.alarmCount = res.alarmnum || 0
                } else {
                    this.deviceInfo = {};
                }
            }).catch((err) => {
                console.log('获取所有设备详情失败', err);
                this.deviceInfo = {};
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
        .gray{
            color: #666666;
        }
        .black{
            color: #323233;
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
        .nowrap{
            white-space: nowrap;
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
        font-size: 0.34rem;
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