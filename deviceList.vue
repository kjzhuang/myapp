<template>
    <div class="device-list">
        <div class="searchCon">
            <div class="typeCon">
                <van-dropdown-menu >
                    <van-dropdown-item @change="changeStatus" v-model="paramCommon.status" :options="[{text: '全部', value: ''},{text: '在线', value: 1},{text: '离线', value: 2},]" />
                </van-dropdown-menu>
            </div>
            <div class="keywords">
                <!-- <van-search v-model="keyword" placeholder="请输入设备号搜索" @input="matchingByKey" style="100%"/> -->
                <div style="position:relative">
                    <div @click="$refs.searchVanList.focus()">
                        <i class="van-icon van-icon-search searchIcon"></i>
                        <input @input="matchingByKey" v-model="keyword" class="searchVanList" type="text" id="searchVanList"  ref="searchVanList" placeholder="请输入设备号搜索" style="100%">
                    </div>
                    <i v-if="keyword" @click="clearSearch" class="van-icon van-icon-clear van-field__clear clearIcon"></i>
                </div>
                <div class="checkByKey">
                    <SelectList :dataList="tempArrByKey" :type="searchtype" @selectedItem="selectedItem"></SelectList>
                </div>
            </div>
        </div>
        <div class="bd">
            <van-tabs @click="onTab" color="#5A32D2" title-active-color="#0A0A0A" title-inactive-color="#A3A3A3">
                <van-tab :title="`已使用(${params[1].totalCount})`">
                    <van-list v-model="loading[1]" :finished="finished[1]" finished-text="没有更多了" :offset="100" @load="onMore">
                        <div class="item" v-for="(item, index) in listData[paramCommon.isuse]" :key="index">
                            <div class="gpsno frag">{{item.deviceno}}</div>
                            <div class="frag"><span class="gray">状态：</span><span>{{item.status ? '在线' : '离线'}}</span></div>
                            <div class="frag"><span class="gray">电量：</span><span :class="item.energy < 30 ? 'red' : ''">{{item.energy}}%</span></div>
                            <div class="frag"><span class="gray">绑定订单：</span><span class="orderText">{{item.bindOrders}}</span></div>
                            <div class="frag"><span class="gray addressL">当前位置：</span><span class="addressR">{{item.location}}</span></div>
                            <div class="frag"><span class="gray">上报时间：</span><span>{{item.locationtime}}</span></div>
                            <div  @click="gocarTrack(item.deviceno)" class="goTrack">查看轨迹</div>
                            <span class="goDetailBtn" @click="goDetail(item.id, item.deviceno)">查看详情<i class="iconfont icon-ic_arrow_gray_right"></i></span>
                        </div>
                    </van-list>
                </van-tab>
                <van-tab :title="`未使用(${params[2].totalCount})`">
                    <van-list v-model="loading[2]" :finished="finished[2]" finished-text="没有更多了" :offset="100" @load="onMore">
                        <div class="item" v-for="(item, index) in listData[paramCommon.isuse]" :key="index">
                            <div class="gpsno frag">{{item.deviceno}}</div>
                            <div class="frag"><span class="gray">状态：</span><span>{{item.status ? '在线' : '离线'}}</span></div>
                            <div class="frag"><span class="gray">电量：</span><span :class="item.energy < 30 ? 'red' : ''">{{item.energy}}%</span></div>
                            <div class="frag"><span class="gray">绑定订单：</span><span class="orderText">{{item.bindOrders}}</span></div>
                            <div class="frag"><span class="gray addressL">当前位置：</span><span class="addressR">{{item.location}}</span></div>
                            <div class="frag"><span class="gray">上报时间：</span><span>{{item.locationtime}}</span></div>
                            <div @click="gocarTrack(item.deviceno)" class="goTrack">查看轨迹</div>
                            <span class="goDetailBtn" @click="goDetail(item.id, item.deviceno)">查看详情<i class="iconfont icon-ic_arrow_gray_right"></i></span>
                        </div>
                    </van-list>
                </van-tab>
            </van-tabs>
        </div>
    </div>
</template>
<script>
import statisticTime from '@/mixins/statisticTime';
import { OrderService } from '@/services';
// import { execNative } from '@/services/utils';
import SelectList from '@/components/SelectList.vue';
let timer = null, timerInput = null;
export default {
    data() {
        return {
            keyword: '',
            tempArrByKey: [],
            searchtype: 2, // 设备号
            params: {
                1: {
                    pageNo: 1,
                    pageSize: 10,
                    totalCount: 0,
                },
                2: {
                    pageNo: 1,
                    pageSize: 10,
                    totalCount: 0,
                },
            },
            paramCommon: {
                isuse: 1, // 1.已使用tab 2.未使用tab
                deviceno: '', // 设备号
                status: '', // 状态 1.在线 2. 离线
            },
            listData: {
                1: [],
                2: []
            },
            loading: {
                1: false,
                2: false,
            },
            finished: {
                1: false,
                2: false,
            },
            statisticParams: {
                resourceNum: 'shebeiliebiao',
                resourceName: '设备列表',
			}
        };
    },
    mixins: [statisticTime],
    components: { SelectList },
    mounted() {
        this.getList(1);
        this.getList(2);
    },
    methods: {
        /** 获取所有设备号（模糊） */
        matchingByKey() {
            console.log('模糊', this.keyword)
            if (timerInput) {
                clearTimeout(timerInput);
            }
            if (this.keyword.length === 1) {
                return;
            }
            timerInput = setTimeout(() => {
                if (!this.keyword) { // 清除或输入空
                    this.tempArrByKey = [];
                    this.paramCommon.deviceno = '';
                    this.params[1].pageNo = 1; // 选择下拉设备号后重置 pageNo重置1
                    this.params[2].pageNo = 1; // 选择下拉设备号后重置 pageNo重置1
                    this.getList(1);
                    this.getList(2);
                    return;
                }
                OrderService.getAllDevicenos({ keyword: this.keyword }).then((res) => {
                    if (res) {
                        this.tempArrByKey = res;
                    }
                }).catch((err) => {
                    console.log('获取所有设备号失败', err);
                });
            }, 500);
        },
        /** 清除搜索框 */
        clearSearch() {
            this.keyword = '';
            this.tempArrByKey = [];
            this.paramCommon.deviceno = '';
            this.params[1].pageNo = 1; // 选择下拉设备号后重置 pageNo重置1
            this.params[2].pageNo = 1; // 选择下拉设备号后重置 pageNo重置1
            this.getList(1);
            this.getList(2);
        },
        /** 点击搜索框触发选择下拉框内选项 ??? */
        selectedItem(val) {
            console.log('val', val)
            this.paramCommon.deviceno = val;
            this.keyword = val;
            this.tempArrByKey = [];
            this.params[1].pageNo = 1; // 选择下拉设备号后重置 pageNo重置1
            this.params[2].pageNo = 1; // 选择下拉设备号后重置 pageNo重置1
            this.getList(1);
            this.getList(2);
        },
        /** 选择全部 离线 或 在线  ??? */
        changeStatus(value) {
            this.finished[1] = false;
            this.finished[2] = false;
            this.loading[1] = false;
            this.loading[2] = false;
            this.params[1].pageNo = 1; // 选择下拉设备号后重置 pageNo重置1
            this.params[2].pageNo = 1; // 选择下拉设备号后重置 pageNo重置1
            this.getList(1);
            this.getList(2);
        },
        onTab(name, title) {
            // console.log(name, title)
            this.paramCommon.isuse = name === 0 ? 1 : 2;
            // if (this.listData[this.paramCommon.isuse].length === 0) {
            // this.getList();
            // }
        },
        getList(isuse) {
            const curIsuse = isuse ? isuse : this.paramCommon.isuse;
            const Param = Object.assign({}, this.paramCommon, this.params[curIsuse]);
            Param.isuse = curIsuse;
            OrderService.getDeviceList(Param).then((res) => {
                if (res && res.result) {
                    this.params[curIsuse].totalCount = res.totalCount;
                    if (this.params[curIsuse].pageNo === 1) {
                        this.listData[curIsuse] = res.result;
                    } else {
                        this.listData[curIsuse] = this.listData[curIsuse].concat(res.result);
                    }
                    this.loading[curIsuse] = false;
                    if (res.totalCount === this.listData[curIsuse].length) {
                        this.finished[curIsuse] = true;
                    } else {
                        this.finished[curIsuse] = false;
                    }
                } else {
                    this.loading[curIsuse] = false;
                    this.finished[curIsuse] = true;
                    if (this.params[curIsuse].pageNo === 1) { // 第一页报错置空列表
                        this.listData[curIsuse] = [];
                    }
                }
                console.log('this.listData', this.listData)
            }).catch((err) => {
                console.log('设备列表失败', err);
                this.loading[curIsuse] = false;
                this.finished[curIsuse] = true;
            });
        },
        onMore() {
            if (!timer) {
                const type = this.paramCommon.isuse;
                if (this.params[type].totalCount > this.listData[type].length) {
                    this.loading[this.paramCommon.isuse] = true;
                    timer = setTimeout(() => {
                        this.params[type].pageNo++;
                        this.getList();
                        timer = null;
                    }, 500);
                }
            }
        },
        goDetail(id, deviceno) {
            this.$router.push({
                name: 'deviceDetail',
                query: { id, deviceno },
            });
        },
        gocarTrack(deviceno) { // 查看轨迹
            this.$router.push({
                name: 'carTrack',
                query: {
                    deviceno, 
                },
            });
        },
    },
};
</script>
<style lang="less">
.device-list{
    font-size: .28rem;
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
        font-size: .24rem;
    }
    .van-tabs{
        height: calc(93vh);
        width: 100%;
        display: flex;
        flex-direction: column;
        box-sizing: border-box;
        overflow: hidden;
    }
    .van-tabs__content{
        flex: 1;
        overflow: auto;
    }
    #searchVanList{
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
    .gray{
      color: #666666;  
    }
    .red{
        color:#D7142E;
    }
    .searchCon{
        display:flex;
        align-items: center;
        padding:.15rem .26rem .1rem;
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
        z-index: 1001;
    }
    .keywords{
            flex: 1;
            .van-search{
                background: #ffffff;
                padding: 0;
                width: 90%;
                .van-search__content{
                    height: 0.64rem;
                    background: #F4F4F4;
                    border-radius: 0.64rem !important;
                }
                .van-cell{
                    line-height: 0.5rem;
                }
            }
            .checkByKey{
                position: fixed;
                max-height:6rem;
                overflow: auto;
                top: 0.8rem;
                left: -16px;
                right: -16px;
                z-index: 1000;
                padding: 0 0.24rem 0.2rem 0.24rem;
                box-sizing: border-box;

            }
        }
    .bd{
        background: #F2F2F4;
    }
    .item{
        position: relative;
        border:.26rem solid #F2F2F4;
        border-top:none;
        background: #fff;
        padding:.26rem;
        padding-bottom: 1rem;
        &:first-child{
            border-top:.26rem solid #F2F2F4;
        }
        .gpsno{
            font-size: .34rem;
            font-weight: 600;
            color: #0A0A0A;
        }
    }
    .frag{
        padding-bottom:.25rem;
        display: flex;
        align-items: center;
        span{
            // line-height: 1;
        }
    }
    .addressL{
        min-width: .2rem;
    }
    .addressR{
        flex: 1;
    }
    .orderText{
        word-break: break-all;
        flex: 1;
        word-break: break-all;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
    }
    .goTrack{
        background: #5A32D2;
        box-shadow: 0 3px 6px 0 rgba(90,50,210,0.40);
        border-radius: 25px;
        text-align: center;
        height: .6rem;
        line-height: .6rem;
        color:#fff;
        width:1.72rem;
        margin-top:.1rem;
        font-size: .24rem;
        float: right;
    }
    .goDetailBtn{
        position: absolute;
        top:20px;
        right:.26rem;
        color: #5A32D2;
        i{font-size: 12px;padding-left: .05rem;}
    }
}
</style>