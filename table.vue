<template>
     <div class="tableScroll" ref="tableScroll">
      	<div class="content" ref="table">
            <!--首列固定-->           
          	<div class="left-content">                 
                 <div class="table-head"> 
                      <table class="full-table">
                      		<thead>
                                <!-- <tr class="tr-head-left" v-for="(header, ind) in tableHeader" :key="`a${ind}`">
                                    <th class="firstCol" v-for="(b,index) in header" :key="`b${index}`" v-if="index==0">
                                        <p>{{b}}</p>
                                    </th>
                                </tr> -->
                                 <tr class="tr-head-left">
                                    <th class="firstCol" v-for="(b,index) in tableHeader" :key="`b${index}`" v-if="index==0">
                                        <p :style="{width: b.w + 'px'}">{{b.name}}</p>
                                    </th>
                                </tr>
                            </thead> 
                      </table>
                  </div>
                  <div class="table-left">
                      <div class="table"
                           ref="firstColLayer"
                           data-_ref="firstColLayer"
                           >
                         <table class="full-table"> 
                            <tbody>
                                <tr class="tr-body-left" v-for="(row, ind) in dataSheet" :key="`a${ind}`">
                                    <td v-for="(c,index) in row" v-if="index==0" :key="`b${index}`">
                                        <p>{{c}}</p>
                                    </td>
                                </tr>
                            </tbody>
                         </table>
                      </div>
                  </div> 
            </div>
          	<div class="right-content" ref="right">
            	<!--首行固定-->
                <div class="table-head"
                     ref="firstRowLayer"
                     data-_ref="firstRowLayer">
                     <table class="full-table">
                        <thead>
                            <!-- <tr class="tr-head-right" v-for="(header, ind) in tableHeader" :key="`a${ind}`">
                                <th v-for="(b, index) in header" v-if="index!=0" :key="`b${index}`"
                                    :style="{ width: leftContent + 'px' }">
                                    <p class="hd">{{b}}</p>
                              </th>
                            </tr> -->
                             <tr class="tr-head-right">
                                <th v-for="(b, index) in tableHeader" v-if="index!=0" :key="`b${index}`"
                                    :style="{ width: b.w + 'px' }">
                                    <p class="hd" :style="{ width: b.w + 'px' }">{{b.name}}</p>
                              </th>
                            </tr>
                        </thead>
                     </table>
                </div>
                <!--正常表格内容(只有表格内容，没有表头和首列)-->
                <div class="table table-right"
                     style="overflow:scroll"
                     ref="tableContainer"
                     @scroll="tableDivScroll($event)"
                     >
                    <table class="content-table">
                        <tbody ref="tbody">
                            <tr class="tr-body-right" v-for="(row, ind) in dataSheet" :key="`a${ind}`">
                                <td v-for="(c, index) in row" v-if="index!=0" :key="`b${index}`">
                                    <p :style="{ width: tableHeader[index].w + 'px' }">{{c}}</p>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
          	</div>
        </div> 
    </div>

</template>
<script>
    export default {
        name: 'home',
        data() {
            return {
                leftContent: 140,
                dataSheet: [
                    ['2020-10-10 12:00:00', '23.9℃', '26.7%RH', '0.01LUX', '5.5g', '85°', '上海上海上海上不上上 不'],
                    ['2020-10-10 12:00:00', '23.9℃', '26.7%RH', '0.01LUX', '5.5g', '85°', 16],
                    [21, 22, 23, 24, 25, '85°', 26],
                    [31, 32, 33, 34, 35, '85°', 36],
                     ['第二次', 2, 3, 4, 5, '85°', 6],
                    [11, 12, 13, 14, 15, '85°', 16],
                    [21, 22, 23, 24, 25, '85°', 26],
                    [31, 32, 33, 34, 35, '85°', 36],
                     ['第三次', 2, 3, 4, 5, '85°', 6],
                    [11, 12, 13, 14, 15, '85°', 16],
                    [21, 22, 23, 24, 25, '85°', 26],
                    [31, 32, 33, 34, 35, '85°', 36],
                     ['第四次', 2, 3, 4, 5, '85°', 6],
                    [11, 12, 13, 14, 15, '85°', 16],
                    [21, 22, 23, 24, 25, '85°', 26],
                    [31, 32, 33, 34, 35, '85°', 36],
                     [1, 2, 3, 4, 5, 6],
                    [11, 12, 13, 14, 15, 16],
                    [21, 22, 23, 24, 25, 26],
                    [31, 32, 33, 34, 35, 36],
                     ['第五次', 2, 3, 4, 5, 6],
                    [11, 12, 13, 14, 15, 16],
                    [21, 22, 23, 24, 25, 26],
                    [31, 32, 33, 34, 35, 36],
                     ['第六次', 2, 3, 4, 5, 6],
                    [11, 12, 13, 14, 15, 16],
                    [21, 22, 23, 24, 25, 26],
                    [31, 32, 33, 34, 35, 36],
                     ['第七次', 2, 3, 4, 5, 6],
                    [11, 12, 13, 14, 15, 16],
                    [21, 22, 23, 24, 25, 26],
                    [31, 32, 33, 34, 35, 36],
                ],
                tableHeader: [
                    {
                        name: '时间', w: 140,
                    }, {
                        name: '温度', w: 60,
                    }, {
                        name: '湿度', w: 70,
                    }, {
                        name: '光感', w: 70,
                    }, {
                        name: '震动', w: 50,
                    }, {
                        name: '倾斜', w: 60,
                    }, {
                        name: '地址', w: 140,
                    }],
            };
        },
        // 定一个生命周期钩子监听变动
        mounted() {
            // let maxHeight = window.screen.height
            document.body.style.overflow = 'hidden';
            this.$refs.right.style.width = '' + this.$refs.table.offsetWidth - this.leftContent + 'px'; // 这里的减100是减去左侧div宽度
            console.log('offsetWidth', this.$refs.table.offsetWidth);
            // console.log(this.placeholderTop);
        },
        methods: {
            tableDivScroll(event) {
                console.log('event', event);
                const $target = this.$refs.tableContainer;
                // 首行固定
                this.$refs.firstRowLayer.scrollLeft = $target.scrollLeft;
                // 首列固定
                this.$refs.firstColLayer.scrollTop = $target.scrollTop;
            },
        },
    };
</script>
 
<style lang="less">
@leftContent: 140px;
//    body{
//         overflow:hidden;
//     }
    .tableScroll{
        font-size: .26rem;
		height:100%;
        overflow:hidden;
        color: #0A0A0A;
        p{margin:0;}
        table,thead,tbody{
            border-spacing: 0;
        }
        .content{
            width:100%;
        }
        .tr-head-right{
            display: flex;
        }
        .tr-head-left{
            display: inline-block;
            width:@leftContent;
        }
        .tr-body-right{
            border-bottom:1px solid #DEDDE9;
            display: inline-block;
        }
        .tr-body-left{
            border-bottom:1px solid #DEDDE9;
            display: inline-block;
        }
        .left-content{ // 左边固定一列宽度
            width: @leftContent;
            float:left;
            p{width: @leftContent;}
        }
        .right-content{
            float:left
        }
        .table-body{
            width:100%;
            overflow:auto;
        }
        .table-head{
            width:100%;
            overflow:hidden;
        }
        .left-content .table-body{
            overflow:hidden;
        }
        .table-left .table{
            height:530px;
            background-color:#FFFFFF;
            overflow:hidden;
            margin-right:0;
            padding-right:0;
        }
        .table-right{
            height:530px;
        }
        table::-webkit-scrollbar{display:none}
        .content-table th, .full-table th{ // 表头样式
            font-size:.24rem;
            font-weight:400;
            color:#176BED;
            height:.7rem;
            line-height:.7rem;
            text-align:center;
            color: #666666;
            padding:0;
        }
        .content-table td, .full-table td {
            line-height: .7rem;
            height: .7rem;
            text-align: center;
            word-wrap: break-word;
            word-wrap: normal\0;
            overflow: hidden; 
            -o-text-overflow: ellipsis;
            text-overflow: ellipsis;
            padding:0;
        }
        th,td p{
            display: inline-block;
            vertical-align: middle;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
        }
        .content-table {
            display:block;
            background-color:#FFFFFF;
        }
        thead,tbody{
            background-color:#FFFFFF;
        }
  	}
</style>