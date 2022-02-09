<template>
    <div class="scan-detail">
        <div class="items grayLightBg hd">
            <div class="item">
                <span>设备信息</span>
                <i @click="reflash" class="iconfont icon-shuaxin" style="padding-left: .3rem;"></i>
            </div>
        </div>
        <div class="items">
            <div class="item">
                <span>电量</span>
                <span :class="info.energy < 50 ? 'red' : ''">{{info.energy}}%</span>
            </div>
        </div>
        <div class="items">
            <div class="item">
                <span>状态</span>
                <span :class="info.status == 1 ? '' : 'red'">{{info.status == 1 ? '在线' : '离线'}}</span>
            </div>
        </div>
        <div v-if="columnObj.temperature" class="items">
            <div class="item">
                <span>温度</span>
                <span :class="info.temperatureStatus ? '' : 'red'">{{info.temperature}}℃</span>
            </div>
        </div>
        <div v-if="columnObj.humidity" class="items">
            <div class="item">
                <span>湿度</span>
                <span :class="info.humidityStatus ? '' : 'red'">{{info.humidity}}%RH</span>
            </div>
        </div>
        <div v-if="columnObj.antenna" class="items">
            <div class="item">
                <span>外置GPS</span>
                <span :class="info.antenna == 1 ? '' : 'red'">{{info.antenna == 1 ? '连接' : '不连接'}}</span>
            </div>
        </div>
        <div v-if="columnObj.external_switch" class="items">
            <div class="item">
                <span>外开关</span>
                <span :class="info.external_switch == -1 ? 'red' : ''">{{info.external_switch == 1 ? '开' : info.external_switch == 0 ? '关' : '--'}}</span>
            </div>
        </div>
        <div v-if="columnObj.rock" class="items">
            <div class="item">
                <span>震动</span>
                <span :class="info.rock >= 0 && info.rock <= 30 ? '' : 'red'">{{info.rock}}g</span>
            </div>
        </div>
        <div v-if="columnObj.angle" class="items">
            <div class="item">
                <span>倾斜</span>
                <span :class="info.angle >= 0 && info.angle <= 180 ? '' : 'red'">{{info.angle}}°</span>
            </div>
        </div>
        <div v-if="columnObj.exposure" class="items">
            <div class="item">
                <span>光感</span>
                <span :class="info.exposure >= 0 && info.exposure <= 65535 ? '' : 'red'">{{info.exposure !== null ? info.exposure : '--' }}LUX</span>
            </div>
        </div>
        <div v-if="columnObj.door_magnetism" class="items">
            <div class="item">
                <span>门磁</span>
                <span :class="info.door_magnetism == 1  ? '' : 'red'">{{info.door_magnetism == 1 ? '开' : '关'}}</span>
            </div>
        </div>
        <div v-if="columnObj.location_mode" class="items">
            <div class="item">
                <span>定位模式</span>
                <span>{{info.location_mode == 1 ? 'GPS' : '基站'}}</span>
            </div>
        </div>
        <div v-if="columnObj.location" class="items">
            <div class="item" style="position:relative">
                <span>最新定位</span>
                <span style="max-width: 75%;position: absolute;right: 0;">{{info.location}}</span>
            </div>
        </div>
        <div v-if="columnObj.time" class="items">
            <div class="item">
                <span>位置上报时间</span>
                <span>{{info.locationtime}}</span>
            </div>
        </div>
        <!-- <div class="items picker">
            <van-cell :title="`${pickerValue}`" is-link @click="pickShow = true"/>
            <van-popup v-model="pickShow" position="bottom" :style="{ height: '30%' }">
                <van-picker
                title=""
                show-toolbar
                :columns="columns"
                :default-index="0"
                :visible-item-count="3"
                @confirm="onConfirm"
                @cancel="onCancel"
                @change="onChange"
                />
            </van-popup>
        </div> -->
        <!-- <div class="grayLightBg result">
            <div class="tipCon" v-if="info.exception === ''">
                <van-icon name="passed" class="green" />
                <span class="green text">设备正常，可正常签收</span>
            </div>
            <div class="tipCon" v-if="info.exception !==undefined && info.exception !== ''">
                <van-icon name="warning-o" class="red" />
                <span class="red text">{{info.exception}}</span>
            </div>
        </div>
        <div @click="save" class="submitBtn active">确认</div> -->
    </div>
</template>
<script>
import _ from 'lodash';
// import dayjs from 'dayjs';
import { OrderService } from '@/services';
import { execNative } from '@/services/utils';

let Deviceno = '';
let currentInd = 1;
export default {
    data() {
        return {
            pickShow: false,
            columns: ['正常签收', '异常签收', '拒收'],
            pickerValue: '正常签收',
            info: {},
            currentTip: null,
            columnObj: { // 相应设备号对应的能力
                temperature: false, // 温度能力
                humidity: false, // 湿度能力
                rock: false, // 震动
                angle: false, // 倾斜
                exposure: false, // 光感
                external_switch: false, // 外开关
                door_magnetism: false, // 门磁
                location_mode: false, // 定位模式
                time: false, // 时间
                location: false, // 地点
                antenna: false, // 外置GPS
            },
        };
    },
    created() {
        const query = this.$route.query;
        console.log('activated', query)
        if (query.deviceno) {
            Deviceno = query.deviceno;
            this.getColumn(Deviceno);
            this.getDatail();
        }
    },
    activated() {
        const query = this.$route.query;
        console.log('activated', query)
        if (query.deviceno) {
            Deviceno = query.deviceno;
            this.getColumn(Deviceno);
            this.getDatail();
        }
    },
    methods: {
        /** 获取相应能力的列 */
        getColumn(deviceno) {
            OrderService.getColumn({ deviceno }).then((res) => {
                if (res) {
                    this.columnObj = res;
                }
            }).catch((err) => {
                console.log('获取所有设备号失败', err);
            });
        },
        getDatail() {
            OrderService.getDeviceInfo({ deviceno: Deviceno }).then((res) => {
                if (res) {
                    this.info = res;
                    // let more = 0;
                    // this.currentTip = null;
                    // const diffD = dayjs(new Date()).diff(dayjs(res.locationtime), 'day');
                    // for (let p in res) {
                    //     if (p === 'energy' && res[p] < 50) {
                    //         this.currentTip = '设备电量过低，请使用前先充电。';
                    //         more++;
                    //     }
                    //     if (p === 'status' && res[p] != 1 && diffD > 10) {
                    //         this.currentTip = '设备长时间离线，请确认设备是否异常。';
                    //         more++;
                    //     }
                    //     if (!res.temperatureStatus) {
                    //         this.currentTip = '设备温度异常，请确认设备是否异常。';
                    //         more++;
                    //     }
                    //     if (!res.humidityStatus) {
                    //         this.currentTip = '设备湿度异常，请确认设备是否异常。';
                    //         more++;
                    //     }
                    //     if (res.antenna == 0) {
                    //         this.currentTip = '请将外置附件与主机连接。';
                    //         more++;
                    //     }
                    //     if (res.antenna && res.external_switch == -1) {
                    //         this.currentTip = '外置附件的开关状态异常，请确认附件是否正常。';
                    //         more++;
                    //     }
                    //     if (more > 1) {
                    //         this.currentTip = '设备多处数据异常，请确认设备是否异常。';
                    //     } else if (more === 0) {
                    //         this.currentTip = '';
                    //     }
                    // }
                } else {
                    this.info = {};
                }
            }).catch((err) => {
                console.log('获取所有设备号失败', err);
                this.info = {};
            });
        },
        onConfirm(value, index) {
            console.log('确定', value, index)
            this.pickerValue = value;
            this.pickShow = false;
            currentInd = index + 1;
        },
        save: _.throttle(function () {
            OrderService.save({ ...this.info, signstatus: currentInd }).then((res) => {
                setTimeout(() => {
                    execNative('showToast', { content: '确认成功' });
                }, 100);
            }).catch((err) => {
                execNative('showToast', { content: '确认失败' });
                console.log('获取所有设备号失败', err);
            });
        }, 500),
        onChange(picker, value, index) {
            console.log('改变', picker, value, index)
        },
        onCancel() {
            console.log('取消')
            this.pickShow = false;
        },
        reflash() {
            this.getDatail();
            currentInd = 1;
            this.pickerValue = '正常签收';
        },
    },
};
</script>
<style lang="less">
    .scan-detail{
        font-size:.28rem;
        color: #0A0A0A;
        padding-bottom:.5rem;
        .grayLightBg{
            background: #F2F2F4;
        }
        .red{
           color: #D7142E; 
        }
        .green{
            color: #20AF70;
        }
        .blueBg{
            background: #5A32D2;
        }
        .grayBg{
           background: #BABACE; 
        }
        .hd{
            span{
                font-weight: 600;
            }
        }
        .tipCon{
            display: flex;
            align-items: center;
        }
        .items{
            padding:0 .32rem;
            border-bottom:1px solid #eeeefb;
            height:1rem;
            i{
                font-size: .42rem;
            }
        }
        .picker{
            align-items: center;
            display: flex;
            .van-cell{
                padding:0;
                font-size:.28rem;
            }
        }
        .item{
            display: flex;
            align-items: center;
            justify-content: space-between;
            height: 100%;
        }
        .result{
           height:1.1rem; 
           padding:0 .32rem;
           font-size: .26rem;
           display: flex;
            align-items: center;
            i{font-size:.34rem;line-height: 1;}
           .text{
               padding-left:.15rem;
           }
        }
        .submitBtn{
            background: #BABACE;
            border-radius: 5px;
            height:1.1rem;
            line-height: 1.1rem;
            color:#fff;
            position: fixed;
            width: 90%;
            margin: 0;
            left: 50%;
            transform: translate(-50%,0);
            bottom: .2rem;
            text-align: center;
            &.active{
                background: #5A32D2; 
                box-shadow: 0 3px 6px 0 rgba(90,50,210,0.40);
            }
        }
    }
</style>