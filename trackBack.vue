<template>
  <div class="track-back">
    <div id="map-wrap"></div>
    <div class="now-state">
      <ul>
        <!-- <li @click="goBack()">
          <p>返回</p>
        </li> -->
        <li class="car-plice clearfix">
          <p class="nos">
            <span v-if="deviceno">{{tasktype == 2 ? '设备号：' : '订单号：'}}{{deviceno}}</span>
            <!-- <span v-if="playState.deviceno && playState.matchCarnum"> | </span>
            <span v-if="playState.matchCarnum">{{playState.matchCarnum}}</span> -->
          </p>
        </li>
        <!-- <li class="distance clearfix">
          <p>
            {{nowDistance}}km |
            <span>{{trackData.totalMileage}}km</span>
          </p>
        </li>
        <li class="time clearfix">
          <p>
            {{nowCarTime.hour}} |
            <span>{{nowCarTime.day}}</span>
          </p>
        </li> -->
      </ul>
    </div>
    <div class="btn-card">
      <ol>
        <li class="one clearfix">
          <div class="left">
            <i class="iconfont icontime"></i>
            回放时间段
          </div>
          <div class="right">
            <i
              @click="playHandler"
              class="iconfont" :class="[playState.play && !playState.pause?'iconpause':'iconxiasanjiaoxing']"
            ></i>
          </div>
        </li>
        <li class="two l-bet">
          <div class="time v-cen">
            <span>{{selecttime}}</span>
          </div>
          <!-- <div class="day v-cen">
            <span :class="{act:playState.day == 'Today'}" @click="playState.day = 'Today'">今天</span>&nbsp;&nbsp;|&nbsp;&nbsp;
            <span
              :class="{act:playState.day == 'Yesterday'}"
              @click="playState.day = 'Yesterday'"
            >昨天</span>
          </div> -->
        </li>
        <li class="address">
          <div class="time address flexLay">
            <span>时间：</span>
            <span class="lightGray">{{addressTime}}</span>
          </div>
        </li>
        <li class="address">
          <div class="time address flexLay">
            <span>位置：</span>
            <span class="lightGray">{{address}}</span>
          </div>
        </li>
        <li class="three clearfix">
            <div class="slider-wrap">
                <!-- <el-slider v-model="playState.sliderVal" :show-tooltip="false" @input="changeSlider"></el-slider> -->
                <!-- <Range :value="sliderVal"></Range> -->
                <van-slider v-model="playState.sliderVal" :max="max" active-color="#7E60FF" @input="changeSlider">
                    <template #button>
                        <div class="custom-button"></div>
                    </template>
                </van-slider>
            </div>
            <div class="right dropRight" :class="speedDropShow ? 'active' : ''">
                <!-- <PopupRadio v-model="playState.nowPlaySpeed" :options="playSpeed"></PopupRadio> -->
                <van-dropdown-menu direction="up" z-index="1000">
                    <van-dropdown-item v-model="playState.nowPlaySpeed" @opened="openDrop" @closed="closeDrop" 
                    :options="[{text: '0.5倍', value: '0.5'}, {text: '1倍', value: '1'}, {text: '2倍', value: '2'}, {text: '4倍', value: '4'}]" />
                </van-dropdown-menu>
                <div class="speed-btn">倍数 {{playState.nowPlaySpeed}}x</div>
          </div>
        </li>
      </ol>
    </div>
  </div>
</template>
<script>
// import dayjs from 'dayjs';
import aMapLoader from '../assets/js/amap-loader';
import { OrderService } from '@/services';
import { execNative } from '@/services/utils';
// import { WxService } from '@/services';
// import { PopupRadio } from 'vux';

let pauseFlag = '', timeSlide = null, param = null;
export default {
    name: 'weixinCarAuthTrackBack',
    // components: { PopupRadio },
    data() {
        return {
            map: null, // 地图实例
            marker: null,
            marker2: null, // 提示当前速度的marker
            moveInter: null, // 做动画的定时器
            // 后台返回轨迹数据
            trackData: { paths: [] },
            // 回放的各种参数
            playState: {
                play: false, /* 是否正在播放 */
                pause: false, /* 暂停 */
                day: 'Today', /* 播放的日期 Today,Yesterday */
                sliderVal: 0, // 滑块的位置
                nowCoordIndex: 0, // marker当前的坐标点
                baseMoveSpeed: 50, // 移动到下一个点的速度基础速度，毫秒
                nowPlaySpeed: '1', // 当前播放的倍数
                startTime: '0', /* 开始时间 */
                endTime: '0', /* 结束时间 */
                // deviceno: '', /* 车牌号 */
                // matchCarnum: '', /* 车牌号 */
            },
            selecttime: '',
            deviceno: '',
            addressTime: '',
            address: '',
            speedDropShow: false,
            max: 1,
            tasktype: 0,
        };
    },
    computed: {
    // 过滤出经纬度
        lineArr() {
            const arr = [];
            this.trackData.paths.forEach((item) => {
                const arr1 = [];
                arr1.push(item.lng);
                arr1.push(item.lat);
                arr.push(arr1);
            });
            return arr;
        },
        // 实时计算行驶的公里数
        // nowDistance() {
        //     let allDis = 0;
        //     const nowPath = this.trackData.paths.slice(0, this.playState.nowCoordIndex + 1);
        //     if (nowPath && nowPath.length > 0) {
        //         nowPath.forEach((item) => {
        //             allDis += Number(item.distance);
        //         });
        //     }
        //     return allDis.toFixed(2);
        // },
        // 当前点位的时间
        // nowCarTime() {
        //     let hour = 0;
        //     let day = 0;
        //     if (this.trackData.paths && this.trackData.paths.length > 0 && this.playState.nowCoordIndex >= 0) {
        //         let time = this.trackData.paths[this.playState.nowCoordIndex].timeStamp * 1000;
        //         time = new Date(time);
        //         hour = this.formatDate(time, 'hh:mm:ss');
        //         day = this.formatDate(time, 'MM-dd');
        //     }
        //     return { hour, day };
        // },
        // ,做动画的速度ms
        animateSpeed() {
            return (
                Number(this.playState.baseMoveSpeed) / Number(this.playState.nowPlaySpeed)
            );
        },
        // 当前速度
        nowSpeed() {
            if (this.trackData.paths[this.playState.nowCoordIndex]) {
                return this.trackData.paths[this.playState.nowCoordIndex].speed || 0;
            }
            return 0;
        },
    },
    watch: {
    // 切换今天昨天
    // eslint-disable-next-line func-names
        'playState.day': function (val) {
            // 停止动画
            this.getTrackData(val);
        },
        // 倍数的变化
        // eslint-disable-next-line func-names
        'playState.nowPlaySpeed': function (val) {
            if (this.playState.play && !this.playState.pause) {
            // 正在移动式切换倍数，重置动画
                this.setMoveInter();
            }
        },
        // eslint-disable-next-line func-names
        'playState.nowCoordIndex': function (val) {
            // 判断当前坐标点的变化
            const that = this;
            if (timeSlide) {
                clearTimeout(timeSlide);
            }
            timeSlide = setTimeout(() => {
                if (pauseFlag !== '' && pauseFlag === val && (Number(val) !== this.lineArr.length - 1) && Number(val) !== 0) { // 1秒内没有拖滑块 则认为是拖动暂停了
                    console.log('1秒内没有拖滑块 则认为是拖动暂停了')
                    this.getAddress(val);
                }
            }, 2000);
            if (Number(val) === 0) { // 索引是0时，默认赋值时间与地址
                const path = this.trackData.paths[0];
                this.addressTime = path.time || '--';
                this.address = path.address || '--';
            }
            if (this.lineArr.length > 0 && val >= 0) {
                if (val < this.lineArr.length) {
                    this.marker.setPosition(this.lineArr[val]);
                    this.marker.setAngle(this.trackData.paths[val].course);
                    this.marker2.setPosition(this.lineArr[val]);
                    this.marker2.setContent(`<div class="drop"><h5>${that.nowSpeed}</h5><p>公里/时</p></div>`);
                }
                if (Number(val) === this.lineArr.length - 1) {
                    // 最后一个点停止动画
                    console.log('最后一个点停止动画索引', val)
                    clearInterval(that.moveInter);
                    this.playState.play = false;
                    this.playState.pause = false;
                    const path = this.trackData.paths[val];
                    this.addressTime = path.time || '--';
                    this.address = path.address || '--';
                    // this.playState.sliderVal = val; // 处理滑块位置
                }
                this.playState.sliderVal = Number(val);
                // this.playState.sliderVal = (Number(val) / Number(this.lineArr.length - 1)) * 100;// 改变滑块的位置
                //  判断当前点是否在屏幕范围，不再的话移动到屏幕内
                if (this.lineArr[val]) {
                    const spot = this.lineArr[val];
                    const isVisible = this.checkIsVisible(spot);
                    if (!isVisible) {
                        this.map.setCenter(this.lineArr[val]);
                    }
                }
            }
            pauseFlag = val;
        },
    },
    created() {
        console.log('activated')
        const query = this.$route.query;
        this.deviceno = query.deviceno ? query.deviceno : query.orderno;
        this.selecttime = query.selecttime;
        this.tasktype = query.tasktype;
        const obj = query.deviceno ? { deviceno: query.deviceno } : { orderno: query.orderno };
        param = { selecttime: this.selecttime, tasktype: this.tasktype, ...obj };
        this.getTrackData();
    },
    methods: {
        /** 打开选择设备号或订单号下拉框 */
        openDrop() {
            this.speedDropShow = true;
        },
        /** 关闭设备号或订单号下拉框 */
        closeDrop() {
            this.speedDropShow = false;
        },
        // 获取轨迹数据
        async getTrackData() {
            // 初始化参数
            this.playState.nowCoordIndex = 0;
            this.playState.sliderVal = 0;
            this.playState.play = this.playState.pause = false;
            clearInterval(this.moveInter);
            // console.log('?????', this.selecttime, this.deviceno)
            OrderService.queryTrack(param).then((res) => {
                // console.log('回放res', res)
                // res = {
                //     curaddress: "上海市闵行区万象城购物中心(虹莘路西)",
                //     gpstimeStr: "2021-03-11 15:25:01",
                //     pointList: [{
                //         address: "上海市1",
                //         course: 0,
                //         lat: 31.170369464862176,
                //         lng: 121.37170524383158,
                //         speed: 1,
                //         time: "2021-03-11",
                //     },{
                //         address: "上海市2",
                //         course: 0,
                //         lat: 31.570369464862176,
                //         lng: 121.57170524383158,
                //         speed: 2,
                //         time: "2021-03-12",
                //     },{
                //         address: "上海市3",
                //         course: 0,
                //         lat: 31.870369464862176,
                //         lng: 121.87170524383158,
                //         speed: 3,
                //         time: "2021-03-13",
                //     },{
                //         address: "上海市4",
                //         course: 0,
                //         lat: 31.970369464862176,
                //         lng: 121.97170524383158,
                //         speed: 4,
                //         time: "2021-03-14",
                //     },{
                //         address: "上海市5",
                //         course: 0,
                //         lat: 30.170369464862176,
                //         lng: 120.17170524383158,
                //         speed: 5,
                //         time: "2021-03-15",
                //     },{
                //         address: "上海市6",
                //         course: 0,
                //         lat: 30.470369464862176,
                //         lng: 120.57170524383158,
                //         speed: 6,
                //         time: "2021-03-16",
                //     }]
                // }
                if (res) {
                    this.address = res.curaddress;
                    this.addressTime = res.gpstimeStr;
                    // this.playState.deviceno = res.deviceno || '';
                    // this.playState.matchCarnum = res.macthInfo && res.macthInfo.headCarNo || '';
                }

                if (res && res.pointList && res.pointList.length > 0) {
                    // 格式化开始结束时间
                    this.trackData = res;
                    this.trackData.paths = res.pointList;
                    this.max = res.pointList.length - 1;
                } else {
                    this.max = 1;
                    this.trackData = { paths: [] };
                    setTimeout(() => {
                        execNative('showToast', { content: '当前时间段无轨迹数据' });
                    }, 100);
                }
                this.mapInit();
            }).catch((err) => {
                this.max = 1;
                console.log('设备定位失败', err);
            });
        },
        // 初始化地图
        async mapInit() {
            const that = this;
            await aMapLoader();
            that.initTrackBack();
        },
        // 初始化轨迹回放相关
        initTrackBack() {
            const that = this;
            if (that.lineArr.length && that.lineArr.length > 0) {
                this.map = new window.AMap.Map('map-wrap', {
                    resizeEnable: true,
                    center: that.lineArr[Math.floor(that.lineArr.lenght / 2)],
                    zoom: 17,
                });
                this.map.plugin('AMap.ToolBar', () => {
                // 异步加载插件
                    const toolbar = new window.AMap.ToolBar();
                    this.map.addControl(toolbar);
                });
                // const url = 'static/img/truck_position.png';
                this.marker = new window.AMap.Marker({
                    map: this.map,
                    position: that.lineArr[0],
                    icon: require('@/assets/imgs/truck_position.png'),
                    // angle: '90',
                    offset: new window.AMap.Pixel(-8, -16),
                    autoRotation: true,
                    animation: 'AMAP_ANIMATION_NONE', // 动画效果
                });
                this.marker2 = new window.AMap.Marker({
                    map: this.map,
                    position: that.lineArr[0],
                    offset: new window.AMap.Pixel(-26, -80),
                    content: '<div class="drop"><h5>0</h5><p>公里/时</p></div>', // 设置文本标注内容
                    animation: 'AMAP_ANIMATION_NONE', // 动画效果
                });
                // 绘制轨迹
                const polyline = new window.AMap.Polyline({
                    map: this.map,
                    path: that.lineArr,
                    showDir: false,
                    strokeColor: '#7E60FF', // 线颜色
                    // strokeOpacity: 1,     //线透明度
                    strokeWeight: 4, // 线宽
                    strokeStyle: 'solid', // 线样式
                });
                this.map.setFitView();
            } else {
                this.map = new window.AMap.Map('map-wrap');
            }
        },
        // 点击播放事件
        playHandler() {
            this.playState.play = true;
            if (this.playState.nowCoordIndex === 0) {
                // 第一个点，点击开始
                this.playState.pause = false;
                this.setMoveInter();
            }
            console.log('点击播放事件时索引', this.playState.nowCoordIndex)
            // console.log('this.lineArr', this.lineArr)
            console.log('播放状态', this.playState.pause)
            if (this.playState.nowCoordIndex > 0 && this.playState.nowCoordIndex < this.lineArr.length - 1) {
                // 在中间点位，只有暂停和继续
                if (!this.playState.pause) {
                    // 暂停 
                    console.log('每次暂停都进来吗？', this.playState.nowCoordIndex)
                    this.playState.sliderVal = this.playState.nowCoordIndex; // 处理滑块位置
                    const path = this.trackData.paths[this.playState.nowCoordIndex];
                    console.log(path, 'path')
                    this.addressTime = path.time || '--'; // 处理当前时间
                    clearInterval(this.moveInter);
                    this.playState.pause = true;
                    this.getAddress(this.playState.nowCoordIndex);
                } else if (this.playState.pause) {
                    // 继续
                    this.playState.pause = false;
                    this.setMoveInter();
                }
            } else { // 第一个点或最后一个点
                console.log('点击播放事件-第一个点或最后一个点')
                // const path = this.trackData.paths[this.playState.nowCoordIndex];
                // this.addressTime = path.time || '--';
                // this.address = path.address || '--';
            }
            if (this.playState.nowCoordIndex === this.lineArr.length - 1) {
                // 最后一个点，点击只能重新开始
                this.playState.nowCoordIndex = 0;// 改到第一个点
                this.playState.pause = false;
                this.setMoveInter();
            }
        },
        // 设置index累加动画
        setMoveInter() {
            const that = this;
            clearInterval(that.moveInter);
            this.moveInter = setInterval(() => {
                that.playState.nowCoordIndex += 1;
            }, that.animateSpeed);
        },
        // 拖动滑块
        changeSlider(val) {  
            // const index = Math.floor((val * (this.lineArr.length - 1)) / 100);
            console.log('拖动滑块', val)
            if (!this.playState.play || this.playState.pause) {
                this.playState.nowCoordIndex = val;
                this.playState.pause = true;// 一拖动就暂定状态
            }
        },
        // 判断点是否在屏幕内
        checkIsVisible(spot) {
            const bounds = this.map.getBounds();
            const NorthEast = bounds.getNorthEast();
            const SouthWest = bounds.getSouthWest();
            const SouthEast = [NorthEast.lng, SouthWest.lat];
            const NorthWest = [SouthWest.lng, NorthEast.lat];
            const path = [[NorthEast.lng, NorthEast.lat], SouthEast, [SouthWest.lng, SouthWest.lat], NorthWest]; // 将地图可视区域四个角位置按照顺序放入path，用于判断point是否在可视区域
            const isPointInRing = window.AMap.GeometryUtil.isPointInRing(spot, path); // 判断pospot是否在可视区域
            return isPointInRing;
        },
        // 暂停的时候获取当前位置
        async getAddress(index) {
            console.log('index', index)
            const nowPath = this.trackData.paths[index];
            OrderService.getAddress({ lat: nowPath.lat, lng: nowPath.lng }).then((res) => {
                if (res) {
                    this.address = res;
                }
            }).catch((err) => {
                console.log('设备定位失败', err);
            });
            this.addressTime = nowPath.time ? nowPath.time : '';
        },
        goBack() {
            this.$router.back();
        },
    },
    beforeRouteLeave(to, from, next) {
        clearInterval(this.setMoveInter);
        next();
    },
};
</script>
<style lang="less">
*{margin:0;}
html,body {
  height: 100%;
}
.track-back {
    .custom-button, .van-slider__button{
		width: .18rem;
		height: .18rem;
		border: .05rem solid #532fe6;
		border-radius: 50%;
		background: #fff;
	}
	.van-slider__button-wrapper{
		padding:.2rem;
	}
    .van-dropdown-menu__title--active{
        color:#fff;
    }
    .van-dropdown-menu__title{
        color:#fff;
    }
  //   放大缩小
  .amap-touch-toolbar .amap-zoomcontrol {
    bottom: 120px;
  }
  .amap-logo {
    opacity: 0;
    margin-left: -9999px;
  }
  .amap-copyright {
    opacity: 0;
    margin-left: -9999px;
  }
  .weui-cell {
    opacity: 0;
  }
  .amap-marker-label {
    background-color: transparent;
    border: 0;
  }
  .nos{
      word-break: break-all;
  }
  .el-slider__button,.el-slider__runway.disabled .el-slider__button {
    border-color: #7e60ff;
  }
  .el-slider__bar,.el-slider__runway.disabled .el-slider__bar {
    background-color: #7e60ff;
  }
  .el-slider__button-wrapper {
    z-index: 10;
  }
  .drop {
    width: 1.16rem;
    height: 1.16rem;
    background-color: rgb(0, 0, 0);
    border-radius: 50%;
    font-size: .28rem;
    position: relative;
    text-align: center;
    &:after {
      position: absolute;
      content: "";
      width: .24rem;
      height: .24rem;
      background-color: rgb(0, 0, 0);
      // background-color: red;
      left: 50%;
      transform: translateX(-50%) rotate(45deg);
      bottom: -0.08rem;
    }
    h5 {
      padding-top: .2rem;
      color: #f8a52b;
      font-size: .28rem;
      line-height: 1.2;
    }
    p {
      font-size: .24rem;
      color: #afafc2;
    }
  }
}
.dropRight{
    &.active{
        .van-dropdown-menu__title--active{
            color:#ee0a24;
        }
        .van-dropdown-menu__title{
            color:#ee0a24;
        }
    }
}
</style>
<style lang="less" scoped>

.clearfix:after {
  content: "";
  clear: both;
  height: 0;
  display: block;
}
.clearfix {
  zoom: 1;
}
.v-cen {
  display: flex;
  display: -webkit-flex;
  align-content: center;
  align-items: center;
}
.l-bet {
  display: flex;
  justify-content: space-between;
}
.track-back {
  height: 100%;
  background: #fff;
  position: relative;
  overflow: hidden;
  #map-wrap {
    height: 85%;
  }

  .now-state {
    position: absolute;
    top: 0;
    left: 0;
    padding: .2rem;
    ul {
      list-style: none;
      li {
        p {
          float: left;
          // height: .52rem;
          line-height: .52rem;
          border-radius: .26rem;
          background-color: #532fe6;
          opacity: 0.8;
          color: #e0e0eb;
          font-size: .24rem;
          margin-bottom: 0.2rem;
          padding: 0 0.2rem;
          span {
            color: #c3b5ff;
          }
        }
      }
    }
  }
  //   控制开始结束
  .btn-card {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    // height: 1.2rem;
    background-color: #fff;
    box-sizing: border-box;
    padding: 0.2rem 0 0.12rem;
    font-size: 0.28rem;
    box-shadow: 0 -3px 6px 0 rgba(0, 0, 0, 0.06);
    ol {
      list-style: none;
      li {
        padding: 0 0.3rem;
        .left {
          float: left;
        }
        .right {
          float: right;
        }
      }
      li.one {
        .left {
          color: #39394d;
          i {
            color: #7e60ff;
          }
        }
        .right {
          width: 1rem;
          height: 1rem;
          border-radius: 50%;
          margin-top: -0.6rem;
          background-color: #fff;
          background-color: rgba(83, 47, 230, 1);
          box-shadow: 0 2px 4px 0 rgba(83, 47, 230, 0.4);
          color: #fff;
          display: flex;
          justify-content: center;
          align-items: center;
          .iconfont {
			  font-size: 0.56rem;
		  }
		  .iconxiasanjiaoxing {
			  transform: rotate(-90deg);
		  }
          .iconpause {
			font-size: .68rem;
          }
        }
      }
      li.two {
        height: 1rem;
        border-bottom: 1px solid #e0e0eb;
        .time {
          color: #babace;
          span {
            color: #39394d;
          }
        }
        .day {
          color: #babace;
          span.act {
            color: #532fe6;
          }
        }
      }
      li.address{
        padding: .2rem;
        border-bottom: 1px solid #e0e0eb;
        color: #39394d;
      }
      li.three {
        //   margin-top: .4rem;
          display: flex;
        justify-content: space-between;
        align-items: center;
        .slider-wrap {
          margin-right: .2rem;
          flex: 1;
        }
        .right {
           width: 1.5rem;
          height: 0.8rem;
          line-height: 0.8rem;
          position: relative;
          .speed-btn {
            position: absolute;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            text-align: right;
          }
        }
      }
    }
  }
}
</style>