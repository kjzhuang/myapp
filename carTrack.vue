<template>
    <div class="car-track">
        <div class="searchCon">
            <div class="typeCon">
                <van-dropdown-menu >
                    <van-dropdown-item v-model="searchtype" @change="changeDrop" @opened="openDrop" @closed="closeDrop" :options="[{text: '设备号', value: 2}, {text: '订单号', value: 3}]" />
                </van-dropdown-menu>
            </div>
            <div class="keywords">
                <!-- <van-search v-model="keyword" placeholder="请输入" @input="matchingByKey" style="100%"/> -->
                <div style="position:relative">
                    <div @click="$refs.searchVan.focus()">
                        <i class="van-icon van-icon-search searchIcon"></i>
                        <input @input="matchingByKey" v-model="keyword" class="searchBox" type="text" id="searchVan"  ref="searchVan" placeholder="请输入" style="100%">
                    </div>
                    <i v-if="keyword" @click="clearSearch" class="van-icon van-icon-clear van-field__clear clearIcon"></i>
                </div>

                <div class="checkByKey">
                    <SelectList :dataList="tempArrByKey" :type="searchtype" @selectedItem="selectedItem"></SelectList>
                </div>
            </div>
        </div>
        <div class="mapCon">
			<div class="page-section">
                <div class="mapInner" id="mapInner" style="width:100%;height:100vh;"></div>
                <ul class="map-btn">
                    <li class="track-map-add" @click="zoomMap('up')">
                        <img src="../assets/imgs/ic_map_in@2x.png">
                    </li>
                    <li class="track-map-reduce" @click="zoomMap('down')">
                        <img src="../assets/imgs/ic_map_out@2x.png">
                    </li>
                </ul>
			    <div class="cover-best mapIcon"  @click="setFitView"> <img src="../assets/imgs/best.png"></div>
            </div>
		</div>
        <div class="mapBt" v-if="lineShow" :class="bdShow ? 'active' : ''">
            <div class="switchCon" @click="toggleFoldBt">
				<div class="switchBtn" :class="bdShow ? 'active' : ''"></div>
			</div>
            <div class="part">
                <div class="tit ellipsis" v-if="searchtype == 2">
                    <span class="">{{info.deviceno ? info.deviceno : '--'}}</span> 
                    <span class="green ord" v-if="info.orderno">（{{info.orderno}}）</span>
                    <br>
                    <span @click="ordersMore = true" class="more" v-if="info.orderno && info.orderno.length > 25">更多 ></span>
                </div>
                <div class="tit" v-else>{{info.orderno ? info.orderno : '--'}} 
                    <span :class="info.status ? 'green' : 'gray'">{{sebindstatus ? `（${info.deviceno ? info.deviceno : '--'}）` : '（未绑定设备）'}}</span>
                </div>
                <div>
                    <div class="item"><span class="gray">定位时间：</span>{{info.locationtime}}</div>
                    <div class="item"><span class="gray">定位位置：</span>{{info.location}}</div>
                </div>
                <div class="shareBtn" @click="goShare">
                    <div class="shareInn" :class="!sebindstatus && searchtype == 3 ? 'disable' : ''">
                        <i class="iconfont icon-gongxiang"></i>
                        <div>共享</div>
                    </div>
                </div>
            </div>
            <div class="radioBox">
                <van-checkbox v-model="checkedTrack" checked-color="#5A32D2" icon-size="17px">查询轨迹</van-checkbox>
                <div v-if="checkedTrack">
                    <div class="timeCell">
                        <div class="time">
                            <van-icon name="clock-o" size=".3rem"/> 
                            <van-cell title="查询时间段" is-link :value="`${startTime} ~ ${endTime}`" @click="timeStartShow = true"/>
                        </div>
                        <van-calendar v-model="timeStartShow" :default-date="defaultDate" @confirm="onConfirmS" :min-date="minDate" :max-date="maxDate" confirm-text="搜索" color="rgb(83, 47, 230)"
                        type="range"/>
                    </div>
                    <!-- <div class="timeCell">
                        <div class="time"><van-icon name="clock-o" size=".3rem"/> <van-cell title="结束时间" is-link value="2020-12-10 00:00:00" /></div>
                        <van-calendar v-model="timeEndShow" @confirm="onConfirmE" color="rgb(83, 47, 230)"
                        type="range"/>
                    </div> -->
                </div>
                <div v-if="keyword && checkedTrack" class="goTrackBack" @click="goTrackBack">
                    <i class="iconfont iconhuifang"></i><span>轨迹回放</span>
                </div> 
            </div>
        </div>
         <van-popup v-model="ordersMore" overlay-class="orderPop"><span class="ordersMoreC">{{info.orderno}}</span></van-popup>
    </div>
</template>
<script>
import dayjs from 'dayjs';
import { OrderService } from '@/services';
import aMapLoader from '../assets/js/amap-loader';
import SelectList from '@/components/SelectList.vue';
import { execNative } from '@/services/utils';
import statisticTime from '@/mixins/statisticTime';

let omap = null, massMarkers = null;
let mcMrkers = []; // 数组
let mapZoom = 3; // [3, 18]
let contentMarker = null, circleMarker = null, lineMap = null;
let bigCityArr = [], smallCityArr = [], markerObject = {}, overlays = { 1: [], 2: []};
let deviceid = '', orderids = '', sebindtime = '';
let lineShowTemp = false; // toggle下拉框时 控制最下方弱窗的显示隐藏
let timer = null;
export default {
    data() {
        return {
            keyword: '',
            searchtype: 2,
            tempArrByKey: [],
            params: '',
            bdShow: false,
            checkedTrack: false, // 是否勾选查询轨迹时间
            timeStartShow: false, // 日历组件显示
            // timeEndShow: false,
            startTime: '',
            endTime: '',
            defaultDate: [],
            minDate: new Date('2018-01-01'),
            maxDate: new Date(),
            lineShow: false, // 轨迹时显示最下面信息弹层 轨迹线与聚合显示
            curLayer: 1, // 默认当前层级显示最大聚合
            deviceno: '', // 当前搜索设备号
            orderno: '', // 当前搜索订单号
            info: {},
            ordersMore: false,
            sebindstatus: 0, // 共享按钮是否允许 
            statisticParams: {
                resourceNum: 'dingweizhuizong',
                resourceName: '定位追踪',
			}
        };
    },
    mixins: [statisticTime],
    components: { SelectList },
    mounted() {
        let currentDate = new Date();
        currentDate.setDate(currentDate.getDate() - 1);
        this.defaultDate = [currentDate, new Date()];
        this.startTime = dayjs(new Date()).subtract(1, 'day').format('YYYY-MM-DD');
        this.endTime = dayjs(new Date()).format('YYYY-MM-DD');
        const query = this.$route.query;
        if (!query.deviceno) { // 硬件返回不刷新地图  首次从首页进来点来
            console.log('mounted')
            this.mapInit();
        }
    },
    beforeRouteEnter: (to, from, next) => {
        next((vm) => {
            const query = vm.$route.query;
            if (query && query.deviceno) { // 1.从设备列表跳转带订单号或设备号重新刷新页面  2从分享设置页返回时不走这段 3从首页进来不走这段
                vm.init(query.deviceno);
                console.log('beforeRouteEnter')
            } else if (query.home) { // 从首页进来重置
                vm.init('');
            }
        });
    },
    methods: {
        init(no) {
            this.startTime = dayjs(new Date()).subtract(1, 'day').format('YYYY-MM-DD');
            this.endTime = dayjs(new Date()).format('YYYY-MM-DD');
            this.timeStartShow = false; // 日历组件显示
            this.bdShow = false;
            this.deviceno = no;
            this.keyword = no;
            this.searchtype = 2;
            if (no) { // 从设备列表进来 直接查询单条轨迹
                this.lineShow = true; // 轨迹线情景是否显示
                this.checkedTrack = true; // 是否勾选查询轨迹时间
                this.mapInit(no);
                this.getDeviceOrderBaseInfo();
            } else { // 从首页进来
                this.lineShow = false; // 轨迹线情景是否显示
                this.orderno = '';
                this.checkedTrack = false; // 是否勾选查询轨迹时间
                this.curLayer = 1;
                this.sebindstatus = 0;
                this.mapInit();
            }
        },
        matchingByKey() {
            if (timer) {
                clearTimeout(timer);
            }
            if (this.keyword.length === 1) {
                return;
            }
            timer = setTimeout(() => {
                if (!this.keyword) { // 清除或输入空 回到聚合第一级
                    this.tempArrByKey = [];
                    this.lineShow = false;
                    this.addOverLay(bigCityArr, 1); // 显示最少聚合
                    return;
                }
                if (this.searchtype == 2) {
                    this.getGpsnos();
                } else {
                    this.getAllOrdernos();
                }
            }, 500);
        },
        /** 点击触发选择下拉框内选项 */
        selectedItem(val) {
            const searchtype = this.searchtype;
            if (searchtype == 2) {
                this.deviceno = val + '';
                this.keyword = val + '';
            } else if (searchtype == 3) {
                this.orderno = val;
                this.keyword = val;
            }
            this.tempArrByKey = [];
            this.getDeviceOrderBaseInfo();
        },
        setFitView() {
            if (this.lineShow) {
                omap.setFitView(mcMrkers);
            } else {
                omap.setFitView(overlays[this.curLayer]);
            }
        },
        /** 请求单条轨迹接口 渲染单条轨迹显示 */
        getDeviceOrderBaseInfo() {
            this.lineShow = true;
            this.clearMarkers(); // 清除聚合markers
            this.clearMassMarkers(); // 清除海量聚合markers
            const Params = {};
            if (this.searchtype == 2) { // 设备号
                Params.deviceno = this.deviceno;
            }
            if (this.searchtype == 3) { // 订单号
                Params.orderno = this.orderno;
            }
            Params.tasktype = this.searchtype == 2 ? 2 : 1;
            if (this.checkedTrack) {
                Params.starttime = `${this.startTime} 00:00:00`;
                Params.endtime = `${this.endTime} 23:59:59`;
            } else {
                Params.starttime = '';
                Params.endtime = '';
            }
            OrderService.getDeviceOrderBaseInfo(Params).then((res) => {
                if (res) {
                    deviceid = res.deviceid;
                    orderids = res.orderids;
                    sebindtime = res.sebindtime;
                    this.info = {
                        location: res.location,
                        locationtime: res.locationtime,
                        orderno: res.orderno,
                        deviceno: res.deviceno,
                        status: res.status,
                    };
                    this.sebindstatus = res.sebindstatus == 1 && this.searchtype == 3 ? 1 : 0; // 1已绑定  0未绑定 2：已解绑
                    const no = this.searchtype == 2 ? this.deviceno : this.orderno; // 当前是搜索设备号则轨迹显示设备号
                    if (this.checkedTrack) { // 勾选时间
                        if (res.locusInfos && res.locusInfos.length > 0) {
                            const points = [];
                            const locusInfos = res.locusInfos;
                            locusInfos.forEach((ele) => {
                                points.push(new window.AMap.LngLat(parseFloat(ele.lng), parseFloat(ele.lat)));
                            });
                            const normalLineOptions = {
                                strokeColor: '#7E60FF', // 轨迹未选中蓝色
                            };
                            if (lineMap) {
                                omap.remove(lineMap);
                            }
                            lineMap = new window.AMap.Polyline({
                                path: points,
                                strokeWeight: 3,
                                showDir: true,
                                ...normalLineOptions,
                            });
                            lineMap.setMap(omap);
                            mcMrkers.push(lineMap);
                            const point = new window.AMap.LngLat(parseFloat(locusInfos[0].lng), parseFloat(locusInfos[0].lat));
                            if (locusInfos.length > 5) {
                                this.addCircle(point);
                            }
                            const pointEnd = new window.AMap.LngLat(parseFloat(locusInfos[locusInfos.length - 1].lng), parseFloat(locusInfos[locusInfos.length - 1].lat));
                            this.addMarker(pointEnd, no, locusInfos[locusInfos.length - 1].course);
                        } else {
                            setTimeout(() => {
                                execNative('showToast', { content: '无轨迹数据' });
                            }, 100);
                            this.clearLine();
                        }
                    } else if (res.lng) { // 不勾选时间，默认显示一个点
                        const no2 = this.searchtype == 2 ? this.deviceno : this.orderno; // 当前是搜索设备号则轨迹显示设备号
                        const point1 = new window.AMap.LngLat(parseFloat(res.lng), parseFloat(res.lat));
                        this.addMarker(point1, no2, 0);
                    } else {
                        if (contentMarker) {
                            omap.remove(contentMarker);
                        }
                        if (circleMarker) {
                            omap.remove(circleMarker);
                        }
                    }
                } else {
                    if (this.checkedTrack) { 
                        setTimeout(() => {
                            execNative('showToast', { content: '无轨迹数据' });
                        }, 100);
                    }
                    this.sebindstatus = 0;
                    this.clearLine();
                }
            }).catch((err) => {
                this.clearLine();
                console.log('设备定位失败', err);
            });
        },
        /** 轨迹上的圆点 */
        addCircle(point) {
            if (circleMarker) {
                omap.remove(circleMarker);
            }
            const content = '<div class="marker-circle"></div>';
            circleMarker = new window.AMap.Marker({
                content,
                position: point,
                offset: new window.AMap.Pixel(-10, -10),
                zIndex: 1,
                // angle: ele.course
            });
            circleMarker.setMap(omap);
            mcMrkers.push(circleMarker);
        },
        /** 轨迹上的marker 设备号 */
        addMarker(point, carName, course) {
            if (contentMarker) {
                omap.remove(contentMarker);
            }
            const url = 'static/img/truck_position.png';
            const content = `<div class="marker-content">
                <div class="info">
                    <div>
                        <p>${carName}</p>
                    </div>
                    <div class="sharps"></div>
                </div>
                <div class="imgInfo" style="transform:rotate(${course}deg)"> <img src="${url}" /></div>
            </div>`;
            contentMarker = new window.AMap.Marker({
                content,
                position: point,
                offset: new window.AMap.Pixel(-16, -53),
                zIndex: 0,
                // angle: ele.course
            });
            contentMarker.setMap(omap);
            mcMrkers.push(contentMarker);
            mapZoom = omap.getZoom();
            omap.setFitView(mcMrkers);
        },
        /* 清除轨迹线，轨迹线上的圆点和 设备号marker */
        clearLine() {
            if (lineMap) {
                omap.remove(lineMap);
            }
            if (circleMarker) {
                omap.remove(circleMarker);
            }
            if (contentMarker) {
                omap.remove(contentMarker);
            }
        },
        /** 清除海量点 */
        clearMassMarkers() {
            if (massMarkers) {
                massMarkers.clear();
            }
        },
        /** 清除聚合marker */
        clearMarkers() {
            if (omap && overlays[1] && overlays[1].length > 0) {
                omap.remove(overlays[1]); // 清除最少聚合marker
            }
            if (omap && overlays[2] && overlays[2].length > 0) {
                omap.remove(overlays[2]); // 清除最多聚合marker
            }
            overlays[1] = [];
            overlays[2] = [];
        },
        /** 清除搜索框 */
        clearSearch() {
            this.keyword = '';
            this.tempArrByKey = [];
            this.lineShow = false;
            if (contentMarker) {
                omap.remove(contentMarker);
            }
            this.addOverLay(bigCityArr, 1); // 显示最少聚合
        },
        // 初始化地图 flag 1设备号从设备列表进来，2空从首页进来
        async mapInit(flag) {
            await aMapLoader();
            omap = new window.AMap.Map('mapInner', {
                center: [108.946781, 34.270311],
                zoom: 4,
            });
            mapZoom = omap.getZoom();
            this.getAllDeviceLocation(flag);
        },
        /**  */
        /** 设备定位聚合所有点 */
        getAllDeviceLocation(flag) {
            return;
            bigCityArr = [];
            smallCityArr = [];
            OrderService.getAllDeviceLocation().then((res) => {
                if (res && res.length > 0) {
                    res.forEach((ele) => {
                        const no = ele.addressInfos.length === 1 ? ele.addressInfos[0].deviceno : '';
                        bigCityArr.push({
                            lat: ele.center_lat,
                            lng: ele.center_lng,
                            location: ele.county,
                            total: ele.addressInfos.length,
                            deviceno: no,
                        });
                        if (ele.addressInfos && ele.addressInfos.length > 0) {
                            ele.addressInfos.forEach((val) => {
                                smallCityArr.push(val);
                            });
                        }
                    });
                    if (flag) { // 从设备列表跳过来 点击查看轨迹
                        this.getDeviceOrderBaseInfo();
                    } else {
                        this.addOverLay(bigCityArr, 1);
                    }
                }
            }).catch((err) => {
                console.log('设备定位失败', err);
            });
        },
        /*  生成聚合时的marker  type 1: 最少聚合，2：最多聚合 */
        addOverLay(arr, type) {
            return;
            const style = [{
                url: 'https://webapi.amap.com/images/mass/mass0.png',
                anchor: new window.AMap.Pixel(15, 15),
                // size: new window.AMap.Size(11, 11),
                size: new window.AMap.Size(30, 30),
                zIndex: 3,
            }, {
                url: 'https://webapi.amap.com/images/mass/mass1.png',
                anchor: new window.AMap.Pixel(15, 15),
                // size: new window.AMap.Size(7, 7),
                size: new window.AMap.Size(30, 30),
                zIndex: 2,
            }, {
                url: 'https://webapi.amap.com/images/mass/mass2.png',
                anchor: new window.AMap.Pixel(3, 3),
                size: new window.AMap.Size(5, 5),
                zIndex: 1,
            }];
            this.curLayer = type;
            this.clearMarkers();
            this.clearMassMarkers(); // 清除海量聚合markers
            const locationData = [];
            arr.forEach((ele, index) => {
                if (type === 1) { // 最外层聚合
                    locationData.push({
                        lnglat: [ele.lng, ele.lat],
                        style: 0,
                        location: ele.location,
                        total: ele.total || 1,
                        deviceno: ele.deviceno,
                    });
                } else {
                    locationData.push({
                        lnglat: [ele.lng, ele.lat],
                        style: 1,
                        location: ele.deviceno,
                        total: 1,
                        deviceno: ele.deviceno,
                    });
                }

                const markId = `id_${ele.id || index}`;
                const point = new window.AMap.LngLat(parseFloat(ele.lng), parseFloat(ele.lat));
                // const w = ele.location.length * 14;
                // const content = `<div style="width:${w}px;" class="markers"><p class="city">${ele.location}</p><p class="citynum">${ele.total === undefined ? 1 : ele.total}</p></div>`;
                markerObject[markId] = new window.AMap.Marker({
                    content: ' ',
                    position: point,
                    // offset: new window.AMap.Pixel(-34, -40),
                    zIndex: 0,
                    // angle: ele.course
                });
                overlays[type].push(markerObject[markId]); // 得到所有点，为了缩放地图到合适的视野级别
            });
            /* 大量点所对应的方法 */
            console.log('locationData', locationData)
            const that = this;
            massMarkers = new window.AMap.MassMarks(locationData, {
                opacity: 0.8,
                zIndex: 111,
                cursor: 'pointer',
                style,
                // content: 112
            });
            const marker = new window.AMap.Marker({ content: ' ', map: omap });
            massMarkers.on('touchstart', function (e) { // touchstart
                marker.setPosition(e.data.lnglat); // 用户相对应的经纬度
                marker.setLabel({content: e.data.location}) // 用户相对应的名字
            });
            massMarkers.on('click', function (e) { // touchstart
                if (e.data.total === undefined || e.data.total == 1) {
                    // console.log('ele', ele)
                    that.searchtype = 2; // 当点击地图marker时 都以设备号搜轨迹
                    that.keyword = e.data.deviceno;
                    that.deviceno = e.data.deviceno;
                    that.orderno = '';
                    that.getDeviceOrderBaseInfo();
                } else {
                    that.addOverLay(smallCityArr, 2);
                }
            });
            omap.setFitView(overlays[type]);
            massMarkers.setMap(omap);
        },
        /* 生成聚合时的marker  type 1: 最少聚合，2：最多聚合 */
        addOverLay1(arr, type) {
            this.curLayer = type;
            this.clearMarkers();
            this.clearMassMarkers(); // 清除海量聚合markers
            arr.forEach((ele, index) => {
                const markId = `id_${ele.id || index}`;
                const point = new window.AMap.LngLat(parseFloat(ele.lng), parseFloat(ele.lat));
                const w = ele.location.length * 14;
                const content = `<div style="width:${w}px;" class="markers"><p class="city">${ele.location}</p><p class="citynum">${ele.total === undefined ? 1 : ele.total}</p></div>`;
                markerObject[markId] = new window.AMap.Marker({
                    content,
                    position: point,
                    offset: new window.AMap.Pixel(-34, -40),
                    zIndex: 0,
                    // angle: ele.course
                });
                markerObject[markId].on('click', () => {
                    if (ele.total === undefined || ele.total == 1) {
                        // console.log('ele', ele)
                        this.searchtype = 2; // 当点击地图marker时 都以设备号搜轨迹
                        this.keyword = ele.deviceno;
                        this.deviceno = ele.deviceno;
                        this.orderno = '';
                        this.getDeviceOrderBaseInfo();
                    } else {
                        this.addOverLay(smallCityArr, 2);
                    }
                });
                markerObject[markId].setMap(omap);
                overlays[type].push(markerObject[markId]);
            });
            omap.setFitView(overlays[type]);
            // omap.setZoom(6);
        },
        /**
         * 点击分享去分享页
         */
        goShare() {
            if (!this.sebindstatus && this.searchtype == 3) { // 设备号查询全允许查询，订单号下只有sebindstatus = 1才允许查
                execNative('showToast', { content: '当前无位置信息可分享' });
                return;
            }
            const obj = sebindtime ? { no: this.orderno, tasktype: 1, orderids, sebindtime } : { no: this.orderno, tasktype: 1, orderids };
            if (this.searchtype == 2) { // 设备号
                this.$router.push({ 
                    name: 'setShare',
                    query: { no: this.deviceno, tasktype: 2, deviceid },
                });
            } else { // 订单号
                this.$router.push({
                    name: 'setShare',
                    query: obj,
                });
            }
        },
        toggleFoldBt() {
            this.bdShow = !this.bdShow;
        },
        /** 时间选择后搜索 */
        onConfirmS(tS) {
            this.timeStartShow = false;
            this.startTime = dayjs(tS[0]).format('YYYY-MM-DD');
            this.endTime = dayjs(tS[1]).format('YYYY-MM-DD');
            this.getDeviceOrderBaseInfo();
        },
        getGpsnos() {
            OrderService.getAllDevicenos({ keyword: this.keyword }).then((res) => {
                if (res) {
                    this.tempArrByKey = res;
                } else {
                    this.tempArrByKey = [];
                }
            }).catch((err) => {
                this.tempArrByKey = [];
                console.log('获取所有设备号失败', err);
            });
        },
        getAllOrdernos() {
            OrderService.getAllOrdernos({ keyword: this.keyword }).then((res) => {
                if (res) {
                    this.tempArrByKey = res;
                } else {
                    this.tempArrByKey = [];
                }
            }).catch((err) => {
                this.tempArrByKey = [];
                console.log('获取所有设备号失败', err);
            });
        },
        /** 放大缩小地图只针对聚合有效 */
        zoomMap(type) {
            mapZoom = omap.getZoom();
            if (type === 'up' && mapZoom < 18) {
                mapZoom += 1;
            }
            if (type === 'down' && mapZoom > 3) {
                mapZoom -= 1;
            }
            // console.log('层级', mapZoom)
            if (this.keyword && !this.lineShow) {
                if (mapZoom > 5 && this.curLayer == 1) { // 展示最多聚合
                    // console.log('最多聚合')
                    this.addOverLay(smallCityArr, 2);
                } 
                if (mapZoom <= 5 && this.curLayer == 2) {
                    // console.log('最少聚合')
                    this.addOverLay(bigCityArr, 1);
                }
            }
            omap.setZoom(mapZoom);
        },
        /** 打开选择设备号或订单号下拉框 */
        openDrop() {
            console.log('打开')
            if (this.lineShow) {
                lineShowTemp = true;
                this.lineShow = false;
            }
        },
        changeDrop() {
            if (contentMarker) {
                omap.remove(contentMarker);
            }
            lineShowTemp = false;
            this.lineShow = false;
        },
        /** 关闭设备号或订单号下拉框 */
        closeDrop() {
            if (this.searchtype == 3 && this.deviceno) { // 切到订单号 清空设备号
                this.keyword = '';
            } else if (this.searchtype == 2 && this.orderno) { // 切到设备号 清空订单号
                this.keyword = '';
            }
            if (lineShowTemp) {
                this.lineShow = true;
            }
            lineShowTemp = false;
        },
        goTrackBack() {
            const tasktype = this.searchtype == 2 ? 2 : 1; // 2:设备号 1:订单号
            const obj = this.searchtype == 2 ? { deviceno: this.deviceno } : { orderno: this.orderno };
            this.$router.push({
                name: 'trackBack',
                query: {
                    selecttime: `${this.startTime} - ${this.endTime}`,
                    tasktype,
                    ...obj,
                },
            });
        },
    },
};
</script>
<style lang="less">
.car-track{
    font-size: .24rem;
    .green{
        color:#20AF70;
    }
    .gray{
        color: #666666;
    }
    .van-search{
        padding:0;
    }
    .van-search__content{
        background: #F4F4F4;
        border-radius: 20px;
    }
    .van-dropdown-menu__bar{
        height:auto;
        background: none;
        box-shadow: none;
    }
    .van-dropdown-menu__title{
        font-size: 12px;
    }
    .van-cell__title{
        width: 1.5rem;
        flex: none;
    }
    .goTrackBack{
        display: flex;
        align-items: center;
        justify-content: center;
        position: absolute;
        right:.1rem;
        top:.19rem;
        padding:0.06rem 0.08rem;
        border:1px solid #7E60FF;
        border-radius: 4px;
        color:#7E60FF;
        font-size:.12rem;
        i{
            padding-right:4px;vertical-align: middle;
        }
        span{line-height: 1;}
    }
    #searchVan{
        display: block;
        box-sizing: border-box;
        width: 100%;
        min-width: 0;
        margin: 0;
        padding: 0;
        color: #323233;
        line-height: .64rem;
        border: 0;
        background: #F4F4F4;
        border-radius: .4rem;
        padding-left: .6rem;
    }
    .searchIcon{
        position: absolute;
        top: 50%;
        left: 12px;
        transform: translate(0,-50%);
        font-size: 16px;
    }
    .clearIcon{
        position: absolute;
        top: 50%;
        right: 12px;
        transform: translate(0,-50%);
    }
    .imgInfo{
        width: 26px;
        height: 26px;
        img{
            width:100%;
            height:100%;
        }
    }
    .ord{
         white-space: nowrap;
        text-overflow: ellipsis;
        overflow: hidden;
        word-break: break-all;
        flex:1;
    }
    .more{
        font-size: .24rem;
        color: #7E60FF;
    }
    .ordersMoreC{
        padding: .15rem;
        background: #fff;
        display: block;
        border-radius: .1rem;
    }
    .marker-circle{
        width:15px;
        height: 15px;
        border: 3px solid #7E60FF;
        background: #fff;
        border-radius: 50%;
    }
    .markers{
        border: 2px solid #FFFFFF;
        box-shadow: 0 4px 4px 0 rgba(0, 0, 0, 0.12);
        color: #fff;
        text-align: center;
        border-radius: 50%;
        border-radius: 4px;
        padding: 1px;
        background: rgba(255,255,255,1);
        color: #532FE6;
        transform: scale(.9);
        p{margin:0;}
        .city{
            padding-bottom:4px;
        }
    }
    .searchCon{
        display:flex;
        align-items: center;
        padding:.15rem .26rem .1rem;
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        box-sizing: border-box;
        z-index: 1;
        background: #fff;
    }
    .typeCon{
        width:1.32rem;
        margin-right:.16rem;
        height:.64rem;
        background: #F4F4F4;
        border-radius: 20px;
        display: flex;
        align-items: center;
        justify-content: center;
    }
    .keywords{
        flex:1;
    }
    .mapCon{
        flex:1;
        position: relative;
    }
    .mapIcon{
        position: absolute;
        // z-index: 2;
        width:.72rem;
        // box-shadow: 0 2px 5px 0 rgba(0, 0, 0, 0.17);
        img{
            width:100%;
        }
    }
      .marker-content{
		.info{
			padding: .1rem 0;
			background: rgba(0,0,0,.5);
			border-radius: .08rem;
			text-align: center;
			width:1.4rem;
			font-size:.2rem;
			color: #E0E0EB;
			position: relative;
            // margin-bottom: .3rem;
            margin-left: -.46rem;
			z-index: 3;
			&.highLight{
				background: rgba(126,96,255,.7);
				.sharps{
					border-top-color: hsla(260,50%,55%,.7);
					&:after{
						border-top-color: rgba(126,96,255,.7);
					}
				}
			}
		}
		.sharps{
			margin: 0 auto;
			// width: 0; 
			// height: 0;
			// border-width: 6px;
			// border-style: solid;
			// border-color:#fff transparent transparent transparent;
			border-color: transparent;
			border-bottom-width: 0;
			border-top-color: hsla(200,50%,85%,.7);
			bottom: 3px;
			border-width: 5px;
			display: block;
			width: 0;
			height: 0;
			border-style: solid;
			// position: relative;
		    position: absolute;
    		bottom: -7px;
    		left: 50%;
            transform: translate(-50%,0);
            border-bottom: 0!important;
			// top: 1px;
			&:after{
				content: "";
				bottom: 1px;
				margin-left: -4px;
				border-color: transparent;
				border-style: solid;
				position: absolute;
				border-bottom-width: 0;
				border-top-color: rgba(0,0,0,.5);
				border-width: 4px;
				display: block;
				width: 0;
				height: 0;
				left:50%;
				top:-7px;
			}

		}
	}
    .cover-best{
        left:.2rem;
        bottom:4rem;
    }
    .mapInner {
        width: 100%;
        height: 100%;
    }
    .map-btn{
        position: absolute;
        bottom: 4rem;
        right: .2rem;
        // z-index: 2;
        padding: 0;
        margin: 0;
        > li {
        list-style-type: none;
            cursor: pointer;
            line-height: .72rem;
            width: .72rem;
            height: .72rem;
            background: rgba(255, 255, 255, 0.9);
            border-radius: 2.5px;
            position: relative;
            img {
                width: 100%;
                background: #fff;
            }
        }
    }
    .mapBt{
		width:100%;
		background: #fff;
		border-top-left-radius: .4rem;
		border-top-right-radius: .4rem;
		box-shadow: 0 -2px 3px 0 rgba(0,0,0,0.12);
		position: absolute;
		bottom:0;
		left:0;
		z-index: 1000000;
        width:100%;
        padding:0 .28rem .28rem;
        box-sizing: border-box;
        height:auto;
        &.active{
            height:0.8rem;
        }
		.switchBtn{
			height:.08rem;
			width:.8rem;
			background: #7E60FF;;
			border-radius: .04rem;
			position: absolute;
			left:50%;
			top:50%;
			transform: translate(-50%,-50%);
			&.active{
				background: #E0E0EB;
			}
		}
		.switchCon{
			height:.8rem;
			width:100%;
			position: relative;
        }
        .part{
            position:relative;
            padding-bottom:.2rem;
            border-bottom: 1px dashed #DEDDE9;
        }
        .radioBox{
            padding:.24rem 0 0;
            position: relative;
        }
        .tit{
            font-size: .3rem;
            color: #0A0A0A;
            font-weight: 600;
            padding-bottom:.16rem;
            word-break: break-all;
            // display: flex;
            // align-items: center;
            padding-right: 1rem;
            span{
                font-weight: normal;
            }
        }
        .item{
            font-size: .24rem;
            padding-bottom:.16rem;
        }
        .shareBtn{
			position: absolute;
			top: 30%;
            right: 0;
            z-index: 10;
			transform: translate(0,-50%);
			width: .88rem;
			height: .88rem;
			font-size: .2rem;
			border-radius: 50%;
			text-align: center;
			border: 1px solid #e0e0e0;
			// padding-top:.08rem;
			i{font-size: .32rem;
    		padding: .06rem 0 .08rem;}
            display: flex;
                justify-content: center;
				flex-direction: column;
			.shareInn{
				display: flex;
                justify-content: center;
				flex-direction: column;
                &.disable{
                    color:#a9a9a9;
                }
			}
			div{
				line-height: 1;
			}
        }
        .van-cell{padding:0;}
        .time{
            display: flex;
            align-items: center;
            width: 100%;
            .van-cell{padding-left:.06rem;}
        }
    }
    .timeCell{
        // border-bottom:1px solid #EFEFF4;
        height:1.1rem;
        display: flex;
        align-items: center;
    }
}
.van-popup--center{
    max-width: 80%;
    word-wrap: break-word!important;
    word-break: normal!important;
    z-index: 2000005!important;
}
.van-overlay.orderPop{ // 所有黑cover
    z-index: 2000004!important;
}
.amap-logo{display: none!important;}
.ellipsis{
        width: 90%;
        display: inline-block;
        text-overflow: ellipsis;
        overflow: hidden;
        white-space: nowrap;
    }
</style>