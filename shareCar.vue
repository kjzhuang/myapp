<template>
	<div class="share-car">
		<div class="Hd">
			<div>来自 <span class="company">{{shareInfo.nickname}}</span> 的共享</div>
			<div class="shareTime"><span>共享时间段：</span> {{shareInfo.startTime}} ~ {{shareInfo.endtime}}</div>
			<div class="process">
				<div class="barBig">
					<div v-if="shareInfo.availTime !== undefined" :style="{width: shareInfo.availTime/allHour * 100 + '%'}" class="barSmall"></div>
				</div>
				<div class="availTime">剩余<span class="hour">{{shareInfo.availTime}}d</span></div>
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
		<div class="Ft" :class="ftShow ? 'active' : ''">
            <div class="switchCon" @click="toggleFoldBt">
				<div class="switchBtn" :class="ftShow ? '' : 'active'"></div>
			</div>
			<div class="info"> <span class="addText">订单号：</span><span class="gray addData ord">{{shareInfo.orderno ? shareInfo.orderno : '--'}}</span>
                <span @click="ordersMore = true" class="more" v-if="shareInfo.orderno && shareInfo.orderno.length > 35">更多 ></span>
            </div>
			<div class="info"> <span class="addText">设备号：</span><span class="gray addData">{{shareInfo.deviceno ? shareInfo.deviceno : '--'}}</span></div>
			<div class="info"> <span class="addText">定位时间：</span><span class="green1 addData">{{shareInfo.locationtime}}</span></div>
			<div class="info address"> <span class="addText">最新定位：</span><span class="gray addData">{{shareInfo.location}}</span></div>
		</div>
        <van-popup v-model="ordersMore"><span class="ordersMoreC">{{shareInfo.orderno}}</span></van-popup>
	</div>
</template>

<script>
import dayjs from 'dayjs';
import aMapLoader from '../assets/js/amap-loader';
import { OrderService } from '@/services';

let omap = null;
let mcMrkers = []; // 数组
let mapZoom = 3; // [3, 18]
let contentMarker = null, circleMarker = null, lineMap = null;
export default {
    data() {
        return {
            mapHeight: 0,
            shareInfo: {},
            params: '',
            allHour: 1,
            ftShow: false,
            ordersMore: false,
        };
    },
    // components: { },
    mounted() {
        const option = this.$route.query;
        if (option.params) {
            this.params = option.params;
        }
        const winH = document.body.clientHeight || document.documentElement.clientHeight;
        this.mapHeight = winH - 88 - 110;
        this.mapInit();
        this.init();
    },
    methods: {
        /** 点击折叠 */
        toggleFoldBt() {
            this.ftShow = !this.ftShow;
        },
        async init() {
            const that = this;
            if (!omap) await this.mapInit();
            const option = this.$route.query;
            if (!option.id) {
                return;
            }
            OrderService.getShareLocation({ id: option.id }).then((res) => {
                // res = {
                //     "starttime": "2021-01-10 00:00:00",
                //     "endtime": "2021-01-30 23:59:59",
                //     "effectstarttime": "2021-01-10 00:00:00",//无视 实际定位查询时间
                //     "effectendtime": "2021-01-30 23:59:59",//无视
                //     "orderno": "ceshidingdan111",
                //     "deviceno": "71018958",
                //     "nickname": "不用设置名字",
                //     "locationtime": "2021-01-10 01:23:59",
                //     "location": "上海市闵行区万象城购物中心(吴中路南)",
                //     "locusInfos": [
                //     {
                //         "locationtime": "2021-01-10 01:23:59",
                //         "location": "上海市闵行区万象城购物中心(吴中路南)",
                //         "lng": 121.3714339676218,
                //         "lat": 31.171938987368748
                //     },
                //     {
                //         "locationtime": "2021-01-10 03:23:59",
                //         "location": "上海市闵行区上海帝璟丽致大酒店(虹莘路东)",
                //         "lng": 121.372315714153,
                //         "lat": 31.16995231279563
                //     },
                //     {
                //         "locationtime": "2021-01-10 07:23:59",
                //         "location": "上海市闵行区万象城购物中心(吴中路南)",
                //         "lng": 121.3714339676218,
                //         "lat": 31.171938987368748
                //     },
                //     {
                //         "locationtime": "2021-01-10 09:23:59",
                //         "location": "上海市闵行区上海帝璟丽致大酒店(虹莘路东)",
                //         "lng": 121.372315714153,
                //         "lat": 31.16995231279563
                //     },
                //     {
                //         "locationtime": "2021-01-10 12:23:59",
                //         "location": "上海市闵行区万象城购物中心(吴中路南)",
                //         "lng": 121.3714339676218,
                //         "lat": 31.171938987368748
                //     },
                //     {
                //         "locationtime": "2021-01-10 14:23:59",
                //         "location": "上海市闵行区上海帝璟丽致大酒店(虹莘路东)",
                //         "lng": 121.372315714153,
                //         "lat": 31.16995231279563
                //     },
                //     {
                //         "locationtime": "2021-01-10 16:23:59",
                //         "location": "上海市闵行区万象城购物中心(吴中路南)",
                //         "lng": 121.3714339676218,
                //         "lat": 31.171938987368748
                //     },
                //     {
                //         "locationtime": "2021-01-10 20:23:59",
                //         "location": "上海市闵行区上海帝璟丽致大酒店(虹莘路东)",
                //         "lng": 121.37231351378483,
                //         "lat": 31.16995421248525
                //     },
                //     {
                //         "locationtime": "2021-01-10 21:23:59",
                //         "location": "上海市闵行区万象城购物中心(吴中路南)",
                //         "lng": 121.3714339676218,
                //         "lat": 31.171938987368748
                //     },
                //     {
                //         "locationtime": "2021-01-11 01:23:59",
                //         "location": "上海市闵行区万象城购物中心(吴中路南)",
                //         "lng": 121.37143916847567,
                //         "lat": 31.171937888082965
                //     },
                //     {
                //         "locationtime": "2021-01-11 09:23:59",
                //         "location": "上海市闵行区上海帝璟丽致大酒店(虹莘路东)",
                //         "lng": 121.37231351378483,
                //         "lat": 31.16995421248525
                //     }
                // ]
                // }
                if (res) {
                    mcMrkers = [];
                    const result = res;
                    this.allHour = result.totalDays || 1;
                    const availTime = result.remainDays || 0;
                    this.shareInfo = {
                        startTime: result.starttime,
                        endtime: result.endtime,
                        location: result.location,
                        availTime,
                        nickname: result.nickname,
                        orderno: result.orderno,
                        deviceno: result.deviceno,
                        locationtime: result.locationtime,
                    };
                    const points = [];
                    const locusInfos = result.locusInfos;
                    if (locusInfos && locusInfos.length > 0) {
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
                        this.addMarker(pointEnd, result.deviceno, locusInfos[locusInfos.length - 1].course);
                        setTimeout(() => { // 30s刷新一次
                            that.init();
                        }, 30000);
                    }
                }
            }).catch((err) => {
                console.log(err);
            });
        },
        addCircle(point) {
            if (circleMarker) {
                omap.remove(circleMarker);
            }
            const content = '<div class="marker-circle"></div>';
            circleMarker = new window.AMap.Marker({
                content,
                position: point,
                offset: new window.AMap.Pixel(-7, -7),
                zIndex: 1,
                // angle: ele.course
            });
            circleMarker.setMap(omap);
            mcMrkers.push(circleMarker);
        },
        addMarker(point, deviceno, course) {
            if (contentMarker) {
                omap.remove(contentMarker);
            }
            const url = 'static/img/truck_position.png';
            const content = `<div class="marker-content">
                <div class="info">
                    <div>
                        <p>${deviceno}</p>
                    </div>
                    <div class="sharps"></div>
                </div>
                <div class="imgInfo" style="transform:rotate(${course}deg)"> <img src="${url}" /></div>
            </div>`;
            contentMarker = new window.AMap.Marker({
                content,
                position: point,
                offset: new window.AMap.Pixel(-13, -55),
                zIndex: 0,
                // angle: ele.course
            });
            contentMarker.setMap(omap);
            mcMrkers.push(contentMarker);
            mapZoom = omap.getZoom();
            omap.setFitView(mcMrkers);
        },
        updateMarkerInfo() {

        },
        // 初始化地图
        async mapInit() {
            await aMapLoader();
            // alert(JSON.stringify(window.AMap));
            omap = new window.AMap.Map('mapInner', {
                center: [108.946781, 34.270311],
                zoom: 4,
            });
        },
        setFitView() {
            omap.setFitView(mcMrkers);
        },
        /**
         * 缩放地图
         * @param type down：缩小；up：方法
         * desc 最大方法倍数20
         */
        zoomMap(type) {
            if (type === 'up' && mapZoom < 18) {
                mapZoom += 1;
            }
            if (type === 'down' && mapZoom > 3) {
                mapZoom -= 1;
            }
            omap.setZoom(mapZoom);
        },
    },
};
</script>

<style lang="less">
.share-car{
	background: #fff;
	flex-direction: column;
	height: 100vh;
    display:flex;
    font-size:.24rem;
    box-sizing: border-box;
    .imgInfo{
        width: 26px;
        height: 26px;
        img{
            width:100%;
            height:100%;
        }
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
  .Hd {
	padding:.2rem .3rem;
	height:1.5rem;
    box-shadow: 0 1px 0 0 #E0E0EB;
    position: absolute;
    top:0;
    left:0;
    width:100%;
    background: #fff;
    z-index:2;
    box-sizing: border-box;
  }
  .company{
	  font-size: .28rem;
	  font-weight: 600;
  }
  .process{
	  	display: flex;
        align-items: center;
        padding-top: .1rem;
	  .barBig{
		  height:.1rem;
		  background: #E0E0EB;
		  border-radius: .2rem;
		    flex:1;
		  position: relative;
		  .barSmall{
			   height:.1rem;
			   background: #7E60FF;
			   border-radius: .2rem;
			   position: absolute;
			   right:0;
			   top:0;
		  }
	  }
	  .availTime{
		  	width: 1.44rem;
            padding-left:.2rem;
            font-size: .26rem;
	  }
	  .hour{
		  color:#7E60FF;
		  font-size:.28rem;
	  }
  }
  .mapCon{
	  flex:1;
	  position: relative;
  }
  .Ft{
      position: absolute;
      bottom:0;
      left:0;
      width:100%;
      background: #fff;
      z-index:2000;
      box-sizing: border-box;
    //   height: 2.8rem;
    &.active{
        height: .8rem;
    }
    &.orig{
        height:2.26rem;
    }
	font-size:.26rem;
	box-shadow: 0 -3px 6px 0 rgba(0,0,0,0.06);
    padding: .1rem .3rem .2rem;
    .switchBtn{
        height:.08rem;
        width:.8rem;
        background: #E0E0EB;
        border-radius: .04rem;
        position: absolute;
        left:50%;
        top:50%;
        transform: translate(-50%,-50%);
        &.active{
            background: #7E60FF;;
        }
    }
    .switchCon{
        height:.6rem;
        width:100%;
        position: relative;
    }
	.gray{
		color:#84849A;
	}  
	.green1{
		color:#20AF70;
	}
	.info{
        padding: .1rem 0;
        display: flex;
        .addText{
            width: 1.5rem;
        }
        .addData{flex:1;}
        .ord{
            white-space: nowrap;
            text-overflow: ellipsis;
            overflow: hidden;
            word-break: break-all;
        }
	}
	.address{
        display: flex;
        padding-bottom:.4rem;
	}
  }
     .mapIcon{
        position: absolute;
        z-index: 2;
        width:.72rem;
        // box-shadow: 0 2px 5px 0 rgba(0, 0, 0, 0.17);
        img{
            width:100%;
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
        z-index: 2;
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
}
</style>
