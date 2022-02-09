<template>
    <div class="history-data">
        <div class="timeCell" id="timeCell">
            <div class="time">
                <van-icon name="clock-o" size=".3rem"/> 
                <van-cell title="查询时间段" :is-link="!$route.query.from" :value="`${startTime} ~ ${endTime}`" @click="$route.query.from?false:timeStartShow = true"/>
            </div>
            <van-calendar v-model="timeStartShow" :default-date="defaultDate" @confirm="onConfirmS" confirm-text="搜索" color="rgb(83, 47, 230)"
            type="range" :min-date="minDate" :max-date="maxDate"/>
        </div>
        <!-- <div class="timeCell">
            <div class="time"><van-icon name="clock-o" size=".3rem"/> <van-cell title="结束时间" is-link value="2020-12-10 00:00:00" /></div>
            <van-calendar v-model="timeEndShow" @confirm="onConfirmE" color="rgb(83, 47, 230)"
            type="range"/>
        </div> -->
        <div v-show="tabs.length > 0" class="bd" id="chartBd">
            <div class="tit">{{tabs && tabs[currentTab] ? tabs[currentTab].tit : '--'}}</div>
            <div style="position:relative;" class="lineCon">
                <div class="chart-value" ref="chartLine" style=" width:100%;height:210px;" id="chartLine" ></div>
                <load-line :lineData="lineData"></load-line>
                <div v-if="!lineData || (lineData && lineData.dataX.length === 0)" class="emptyChart">暂无数据</div>
            </div>
            <div class="tabIcons">
                <span v-for="(item, index) in tabs" :key="index" 
                :class="currentTab == index ? 'active' : ''" @click="ontabChart(item.k, index)">{{item.name}}</span>
            </div>
        </div>
        <div class="tableCon">
            <load-table :tableHeader="tableHeader" :dataSheet="dataSheet" :haveData="haveData" ref="childTable"></load-table>
        </div>
    </div>
</template>
<script>
import dayjs from 'dayjs';
import loadLine from '@/components/line';
import loadTable from '@/components/table';
import { OrderService } from '@/services';

let Deviceno = '', resTemp = []; // 临时存储表格数据
const TABS = {
    temperature: { name: '温度曲线', tit: '温度曲线图(℃)' },
    humidity: { name: '湿度曲线', tit: '湿度曲线图(%RH)' },
    exposure: { name: '光感曲线', tit: '光感曲线图(LUX)' },
    rock: { name: '震动曲线', tit: '震动曲线图(g)' },
    angle: { name: '倾斜曲线', tit: '倾斜曲线图(°)' },
};
const TABSKey = ['temperature', 'humidity', 'exposure', 'rock', 'angle']; // 动态tab项
const TABLE = {
    time: { name: '时间', w: 140, k: 'time', unit: '' },
    location: { name: '地址', w: 280, k: 'location', unit: '' },
    temperature: { name: '温度', w: 70, k: 'temperature', unit: '℃' },
    humidity: { name: '湿度', w: 100, k: 'humidity', unit: '%RH' },
    rock: { name: '震动', w: 55, k: 'rock', unit: 'g' },
    angle: { name: '倾斜', w: 60, k: 'angle', unit: '°' },
    exposure: { name: '光感', w: 90, k: 'exposure', unit: 'LUX' },
    // doormagnetism_status: { name: '门磁', w: 50, k: 'doormagnetism_status', unit: '' },
    door_magnetism: { name: '门磁', w: 50, k: 'door_magnetism', unit: '' },
    location_mode: { name: '定位模式', w: 70, k: 'location_mode', unit: '' },
    external_switch: { name: '外开关', w: 50, k: 'external_switch', unit: '' },
    antenna: { name: '外置GPS', w: 60, k: 'antenna', unit: '' },
};
const myEvent = new Event('resize');
export default {
    data() {
        return {
            timeStartShow: false,
            // timeEndShow: false,
            lineData: null,
            startTime: '',
            endTime: '',
            defaultDate: [],
            minDate: new Date('2018-01-01'),
            maxDate: new Date(),
            dataSheet: [],
            tableHeader: [],
            currentTab: 0, // 当时tab索引
            ArrKey: [],
            tabs: [],
            haveData: '1',
        };
    },
    components: { loadLine, loadTable },
    mounted() {
    },
    beforeRouteEnter: (to, from, next) => {
        next((vm) => {
            const query = vm.$route.query;
            if (query.from && query.from == 'order') {
                // 初始化时间
                vm.startTime = query.str? dayjs(new Date(query.str*1)).format('YYYY-MM-DD'): dayjs(new Date(query.end*1)).subtract(2, 'day').format('YYYY-MM-DD');
                vm.endTime = dayjs(new Date(query.end*1)).format('YYYY-MM-DD');

                let end = new Date(query.end*1)
                let str = query.str?new Date(query.str*1) : end.setDate(end.getDate() - 2)
                vm.defaultDate = [str, new Date(query.end*1)];

            } else {
                vm.startTime = dayjs(new Date()).subtract(2, 'day').format('YYYY-MM-DD');
                vm.endTime = dayjs(new Date()).format('YYYY-MM-DD');
                let currentDate = new Date();
                currentDate.setDate(currentDate.getDate() - 2);
                vm.defaultDate = [currentDate, new Date()];
            }
            if (query.deviceno) {
                Deviceno = query.deviceno;
                vm.getTableTitle(Deviceno);
            }
        });
    },
    methods: {
        /** 重新排序表头和表格数据 */
        ontabChart(k, ind) {
            // this.$refs.firstRowLayer.scrollLeft = 0;
            // console.log(this.$refs.childTable)
            this.$refs.childTable.setLeft();
            this.currentTab = ind;
            if (this.dataSheet && this.dataSheet[0] && !this.dataSheet[0][0]) {
                return;
            }
            const arr = JSON.parse(JSON.stringify(this.ArrKey));
            for (let i = 0; i < arr.length; i++) {
                if (arr[i] === k) {
                    arr.splice(i, 1);
                    break;
                }
            }
            arr.splice(1, 0, k);
            const arrHead = [];
            arr.forEach((ele) => {
                arrHead.push({
                    name: TABLE[ele].name,
                    w: TABLE[ele].w,
                    k: ele,
                    unit: TABLE[ele].unit,
                });
            });
            this.ArrKey = arr; // 得到最新排序好的key
            this.tableHeader = arrHead; // 得到最新排序好的表头arr
            this.handleData(resTemp);
        },
        /** 获取表格的表头 */
        getTableTitle(deviceno) {
            OrderService.getColumn({ deviceno }).then((res) => {
                if (res) {
                    const arr = [];
                    this.ArrKey = [];
                    let flag = true, kTemp = '';
                    this.tabs = [];
                    for (let k in res) { // 注意排序
                        if (k === 'time') { // 时间在最第一列
                            arr.unshift({
                                name: TABLE[k].name,
                                w: TABLE[k].w,
                                k,
                                unit: TABLE[k].unit,
                            });
                            this.ArrKey.unshift(k);
                        } else if (res[k]) { // 有此能力
                            if (TABSKey.indexOf(k) !== -1) { // 处理tabs动态显示 在动态tabs数组中
                                this.tabs.push({
                                    name: TABS[k].name,
                                    k,
                                    tit: TABS[k].tit,
                                    data: [],
                                });
                            }
                            if (this.tabs.length === 1 && flag) { // 处理默认tab第一项
                                flag = false;
                                kTemp = k; // 得到处于第二列的表头
                            } else {
                                // console.log('k', k)
                                arr.push({
                                    name: TABLE[k].name,
                                    w: TABLE[k].w,
                                    k,
                                    unit: TABLE[k].unit,
                                });
                                this.ArrKey.push(k);
                            }
                        }
                    }
                    if (kTemp) {
                        arr.splice(1, 0, { // 处理tab第一项显示
                            name: TABLE[kTemp].name,
                            w: TABLE[kTemp].w,
                            k: kTemp,
                            unit: TABLE[kTemp].unit,
                        });
                        this.ArrKey.splice(1, 0, kTemp); // 处理tab第一项显示
                    }
                    // console.log('ArrKey', this.ArrKey)
                    this.tableHeader = arr;
                    this.getHistoryDetail(deviceno); // 得到表格数据接口
                }
            }).catch((err) => {
                console.log('获取所有设备号失败', err);
            });
        },
        /** 获取表格的值 */
        getHistoryDetail(deviceno) {
            const starttime = `${this.startTime} 00:00:00`;
            const endtime = `${this.endTime} 23:59:59`;
            OrderService.getHistoryDetail({ deviceno, starttime, endtime }).then((res) => {
                if (res && res.length === 0) {
                    this.haveData = '0';
                } else if (res && res.length > 0) {
                    this.haveData = '1';
                }
                resTemp = res; // 临时数据处理
                this.handleData(res);
            }).catch((err) => {
                console.log('获取所有设备号失败', err);
            });
        },
        /** 获取表格的值进行处理 this.ArrKey成关键排序数组 */
        handleData(res) {
            let arrData = [];
            let arr = [];
            const lineData = { id: 'chartLine', dataX: [], dataY: [], name: '', unit: '' };
            if (res && res.length > 0) {
                arrData = [];
                arr = [];
                res.forEach((ele, index) => { // 循环处理数据
                    arr = [];
                    this.ArrKey.forEach((val) => { // 所有key值找到对应的值
                        arr.push(ele[val]);
                    });
                    this.tabs.forEach((item) => {
                        if (index === 0) {
                            item.data = [];
                        }
                        item.data.push({ value: ele[item.k], unit: TABLE[item.k].unit });
                    });
                    arrData.push(arr);
                    const tt = ele.time ? ele.time.substring(5) : '';
                    lineData.dataX.push(tt);
                });
            } else {
                arrData = [];
                arr = [];
                this.ArrKey.forEach((val) => { // 所有key值找到对应的值
                    arr.push('');
                });
                arrData.push(arr);
            }
            if (this.tabs.length > 0) {
                lineData.dataY = this.tabs[this.currentTab].data;
                lineData.name = this.tabs[this.currentTab].name;
                this.lineData = lineData;
            }
            this.dataSheet = arrData;
            setTimeout(() => {
                window.dispatchEvent(myEvent);
            }, 50);
            // console.log('arrData', arrData)
            // console.log('lineData', lineData)
        },
        onConfirmS(tS) {
            // console.log('ts', tS)
            this.timeStartShow = false;
            this.startTime = dayjs(tS[0]).format('YYYY-MM-DD');
            this.endTime = dayjs(tS[1]).format('YYYY-MM-DD');
            this.getHistoryDetail(Deviceno);
        },
    },
};
</script>
<style lang="less">
    .history-data{
        color: #0A0A0A;
        font-size: .26rem;
        .emptyChart{
            position: absolute;
            top:0;
            left:0;
            background: #fff;
            color:#ababab;
            height:210px;
            line-height:210px;
            text-align: center;
            width:100%;
            font-size: .3rem;
        }
        .lineCon{position: relative;}
        .timeCell{
            // border-bottom:1px solid #EFEFF4;
            height:0.9rem;
            display: flex;
            align-items: center;
            padding:0 .32rem;
             .van-cell__title{
                width: 1.7rem;
                flex: none;
            }
        }
         .van-cell{
            padding-left:0;
            padding-right:0;
         }
        .time{
            display: flex;
            align-items: center;
            width: 100%;
            .van-cell{padding-left:.06rem;}
        }
        .bd{
            border-top: .2rem solid  #EFEFF4;
            border-bottom: .2rem solid  #EFEFF4;
        }
        .tit{
            padding:.3rem;
            font-size: .3rem;
            font-weight: 600;
        }
        .tabIcons{
            display:flex;
            align-items: center;
            justify-content: space-around;
            padding:.2rem 0;
            span{
                font-size: .2rem;
                background: #F2F2F4;
                border-radius: 25px;
                height:.52rem;
                line-height: .52rem;
                text-align: center;
                padding:.06rem .15rem;
                color: #666666;
                transform: scale(.9);
                &.active{
                    background: #5A32D2;
                    color:#fff;
                }
            }
        }
        .tableCon{
            padding-bottom: .3rem;
        }
    }
</style>