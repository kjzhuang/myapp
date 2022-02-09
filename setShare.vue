<template>
	<div class="local-address">
		<!-- <div class="back"  @click="goBack()">返回</div> -->
		<div class="tit"><span class="carNum">{{no ? no : '--'}}</span> 位置共享</div>
		<div class="sliderCon">
			<div class="name">
				<span class="iconfont icon-shichang time"></span>
				<span>选择共享时长</span>
			</div>
			<div class="bar">
				<div class="curTime" :style="{left: left + '%'}" :class="{less: (modelDay > 3 && modelDay < 45)}">{{modelDay}}天</div>
				<!-- <Range  v-model="modelHour"  :min="1" :max="48" :rangeBarHeight='2'></range> -->
				<van-slider v-model="modelDay" active-color="#7E60FF" :step="1" :max="60">
					<template #button>
						<div class="custom-button"></div>
					</template>
				</van-slider>
				<div class="range">
					<span>1</span>
					<span>最长60天</span>
				</div>
			</div>
		</div>
		<div class="timeshow">
			<div class="tp">
				<span class="lf">共享开始时间</span>
				<span class="rt">共享结束时间</span>
			</div>
			<div class="bt">
				<span class="lf" @click="calendarShowToggle">{{shareTimeS}}</span>
				<span class="lf">~</span>
				<span class="rt">{{shareTimeE}}</span>
			</div>
			<!-- <div v-if="tasktype == 2 && !sebindtime">
				iiooo
				<van-calendar v-model="calendarShow2" @confirm="onConfirmS" 
				 :show-confirm="false" color="rgb(83, 47, 230)"/>
			</div>
			<div v-else>
			嘎嘎k -->
				<van-calendar v-model="calendarShow" :default-date="defaultDate" @confirm="onConfirmS" 
				:min-date="minDate" :max-date="maxDate" :show-confirm="false" color="rgb(83, 47, 230)"/>
			<!-- </div> -->
		</div>
		<div class="btn" @click="sharePositon">共享货物位置</div>
		<!-- <div class="shareBtn">
			<span>温馨提示：分享请点击右上按钮</span>
      		<i class="iconfont icon-sandian tip"></i>
		</div> -->
		<div v-if="coverShare" @click="coverShare = false" class="cover">
			<img src="../assets/imgs/shareArrow.png" alt="">
		</div>
	</div>
</template>

<script>
import dayjs from 'dayjs';
import { OrderService } from '@/services';
/* eslint-disable */
export default {
    name: 'shareLocal',
    data() {
        return {
            // shareTimeS: dayjs().format('YYYY-MM-DD HH:mm'),
            shareTimeS: dayjs().format('YYYY-MM-DD'),
			modelDay: 30,
			no: '',
			coverShare: false,
			calendarShow: false,
			defaultDate: new Date(),
			minDate: new Date('2019-01-01'),
            maxDate: new Date(),
			tasktype: 1, // tasktype 1: 订单号, 2: 设备号
        };
    },
    computed: {
        left: function() {
            if (this.modelDay < 4) {
                return 0;
            } else if (this.modelDay < 57) {
                return this.modelDay / 60 * 100;
            } else {
                return 88;
			}
        },
        shareTimeE: function() {
            // return dayjs(this.shareTimeS).add(this.modelDay, 'day').format('YYYY-MM-DD HH:mm');
            return dayjs(this.shareTimeS).add(this.modelDay, 'day').format('YYYY-MM-DD');
        },
    },
	mounted() {
		this.initWx();
	},
	beforeRouteEnter: (to, from, next) => {
        next((vm) => {
			const query = vm.$route.query;
			vm.no = query.no;
			vm.tasktype = query.tasktype;
			if (query.sebindtime) { // 订单号带了绑定开始时间
				vm.defaultDate = new Date(dayjs(query.sebindtime).format('YYYY-MM-DD'));
				vm.minDate = new Date(dayjs(query.sebindtime).format('YYYY-MM-DD'));
				vm.maxDate = new Date(dayjs(query.sebindtime).add(60, 'day').format('YYYY-MM-DD'));
				vm.shareTimeS = dayjs(query.sebindtime).format('YYYY-MM-DD');
			} else {
				vm.defaultDate = new Date();
				vm.minDate = new Date('2019-01-01');
				vm.maxDate = new Date(dayjs().add(1, 'year').format('YYYY-MM-DD'));
				vm.shareTimeS = dayjs().format('YYYY-MM-DD');
			}
		})
    },
	methods: {
		/** 时间选择后搜索 */
        onConfirmS(tS) {
			// console.log('tS', tS)
			this.calendarShow = false;
            this.shareTimeS = dayjs(tS).format('YYYY-MM-DD');
        },
		calendarShowToggle() {
			this.calendarShow = true;
		},
        async initWx(){
            const data = await OrderService.getSignPackage({
                url: encodeURIComponent(window.location.href.split('#')[0])
            })
            window.wx.config({
                debug: false,
                appId: data.appId,
                timestamp: data.timestamp,
                nonceStr: data.nonceStr,
                signature: data.signature,
                // jsApiList: ['scanQRCode', 'openLocation', 'chooseImage', 'uploadImage', 'getLocalImgData', 'getLocation']
                jsApiList: [
                'updateAppMessageShareData',
                'updateTimelineShareData',
                'onMenuShareAppMessage',
                'onMenuShareTimeline'
                ]
            })
            window.wx.ready(() => {
                this.toShare();
            });
        },
		async toShare() {
			const query = this.$route.query;
			if (!query.tasktype) {
				return;
			}
			const starttime = `${this.shareTimeS} 00:00:00`;
			const endtime = `${this.shareTimeE} 23:59:59`;
            const ids =  await OrderService.shareLocation({ 
				starttime, endtime, tasktype: query.tasktype,
				deviceid: query.deviceid ? query.deviceid : '',
				orderids: query.orderids ? query.orderids : '',
			});
			const name = ids && ids.nickname ? ids.nickname : '--';
			const taskid = ids && ids.taskid ? ids.taskid : '';
            const domain = window.location.href.split('#')[0];
            window.wx.ready(() => {
                // id=encodeURI(id);
                let shareData = { 
                    title: name + '位置分享', // 分享标题
                    desc: '车辆位置分享', // 分享描述
                    link: `${domain}#/shareCar?id=${taskid}`, // 分享链接，该链接域名或路径必须与当前页面对应的公众号JS安全域名一致
                    imgUrl: 'https://mp.g7ia.com/static/img/pic.png', // 分享图标
                    success: function () {
                        // 设置成功
                        console.log('分享设置成功')
                    },
                    error:function(e){
                        console.log('error',e)
                        this.$router.push({ name:'index', query: { id: taskid }})
                    }
                }
                wx.updateTimelineShareData(shareData);
                wx.updateAppMessageShareData(shareData);
            });
        },
		sharePositon() {
			this.coverShare = true;
			this.toShare();
		},
        goBack() {
            this.$router.back();
        },
    },
};
</script>

<style lang="less">
.local-address{
	padding:.6rem;
	background: #fff;
    height:100vh;
	font-size:.26rem;
	.custom-button, .van-slider__button{
		width: .18rem;
		height: .18rem;
		border: .05rem solid #532fe6;
		border-radius: 50%;
		background: #fff;
	}
	.van-slider__button-wrapper{
		padding:.1rem;
	}
	.tip{
		font-size: .4rem;
		line-height: 1;
		color: black;
		padding-left: .1rem;
	}
	.cover{
		width:100vw;
		height:100vh;
		position:fixed;
		top:0;
		left:0;
		background: rgba(0,0,0,.5);
		img{width:80%;margin-left: 20%;margin-top: 5%;}
	}
	.btn{
		background: #5A32D2;
		box-shadow: 0 3px 6px 0 rgba(90,50,210,0.40);
		border-radius: 5px;
		height:1.1rem;
		line-height: 1.1rem;
		text-align: center;
		color:#fff;
		margin-top: .6rem;
	}
	.back {
		display: inline-block;
		height: 0.52rem;
		line-height: 0.52rem;
		border-radius: 0.26rem;
		background-color: #532fe6;
		opacity: 0.8;
		color: #e0e0eb;
		font-size: 0.24rem;
		margin-bottom: 0.2rem;
		padding: 0 0.2rem;
		span {
			color: #c3b5ff;
		}
	}
	.tit{
		font-size:.28rem;
	}
	.carNum{
		font-size:.34rem;
		line-height: 1;
		font-weight:600;
		padding-right:.2rem;
	}
	.sliderCon{
		.name{
			font-size:.26rem;
			line-height: .72rem;
			padding: .2rem 0 .44rem;
		}
		.time{
			color:#7E60FF;
			font-size:.3rem;
			// padding-right:.14rem;
		}
		.curTime{
			font-size: .28rem;
			color: #7E60FF;
			position: absolute;
			top: -.5rem;
			width:.92rem;
			text-align: center;
			&.less{
				margin-left: -.46rem;
			}
		}
		.bar{
			position: relative;
		}
		._slider{
			margin:0;
		}
		.range{
			display:flex;
			justify-content: space-between;
			padding:.16rem 0 .4rem;
			color:#84849A;
    }
    .vux-range-input-box{
      margin: 0 !important;
      .range-min,.range-max{
        display: none;
      }
      .range-handle{
        width: .28rem;
        height: .28rem;
        top:-.14rem !important;
      }
      .range-quantity,.range-handle{
        background-color: #7E60FF;
      }
    }
	}
	.timeshow{
		padding:.4rem 0;
		border-bottom:1px dashed #E0E0EB;
		border-top:1px dashed #E0E0EB;
		.tp,.bt{
			display:flex;
			justify-content: space-between;
		}
		.tp{
			font-size:.26rem;
			padding-bottom:.24rem;
		}
		.bt{
            padding-top:.24rem;
			font-size:.3rem;
		}
	}
	.shareBtn{
		display: flex;
		align-items: center;
		margin-top:.6rem;
		color: #b7b7b7;
		font-size: .28rem;
		.btn{
			background-color: #eeeeee;
			padding:0 .1rem;
		}
	}
}
</style>
