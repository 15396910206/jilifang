<template>
    <div v-loading="loading">
        <div class="pad_5 backfff">
            <div class="pad_5 nav_header">
                <el-row>
                    <el-col :span="24">
                        <el-form :inline="true" class="demo-form-inline">
                            <el-form-item label="开单日期">
                                <el-date-picker
                                        size="mini"
                                        v-model="start_end_data"
                                        type="daterange"
                                        range-separator="至"
                                        start-placeholder="开始日期"
                                        end-placeholder="结束日期">
                                </el-date-picker>
                            </el-form-item>
                            <el-form-item label="发货仓库">
                                <el-select v-model="warehouseID" placeholder="请选择" clearable size="mini">
                                    <el-option
                                            v-for="item in warehouseList"
                                            :key="item.warehouseID"
                                            :label="item.name"
                                            :value="item.warehouseID">
                                    </el-option>
                                </el-select>
                            </el-form-item>
                            <el-form-item label="客户商">
                                <el-select v-model="clientID" placeholder="请选择" clearable size="mini">
                                    <el-option
                                            v-for="item in clientList"
                                            :key="item.custID"
                                            :label="item.name"
                                            :value="item.custID">
                                    </el-option>
                                </el-select>
                            </el-form-item>
                            <el-form-item label="单据状态">
                                <el-select v-model="isConfirm" placeholder="请选择" clearable size="mini">
                                    <el-option
                                            v-for="item in status"
                                            :key="item.id"
                                            :label="item.name"
                                            :value="item.id">
                                    </el-option>
                                </el-select>
                            </el-form-item>
                            <el-form-item label="单据类型">
                                <el-select v-model="stockTotype" placeholder="请选择" clearable size="mini">
                                    <el-option
                                            v-for="item in orderType"
                                            :key="item.id"
                                            :label="item.name"
                                            :value="item.id">
                                    </el-option>
                                </el-select>
                            </el-form-item>
                            <el-form-item>
                                <el-button type="primary" size="mini" @click="handleSearch">搜索</el-button>
                            </el-form-item>
                            <el-form-item style="margin-left: 30px;">
                                <el-button type="primary" size="mini" :loading="downloadLoading" @click="handleDownload">导出</el-button>
                            </el-form-item>
                        </el-form>
                    </el-col>
                </el-row>
            </div>
            <div>
                <el-table :data="tableData" style="width: 100%" highlight-current-row @current-change="detailClick">
                    <el-table-column prop="billID" label="订单编号" >
                    </el-table-column>
                    <el-table-column prop="clientName" label="客户商"></el-table-column>
                    <el-table-column prop="custName" label="供应商">
                        <template slot-scope="scope">
                            <div v-if="scope.row.custName!=''">{{scope.row.custName}}</div>
                            <div v-else>{{siteIDName}}</div>
                        </template>
                    </el-table-column>
                    <el-table-column prop="stockDest" label="发货仓库"></el-table-column>
                    <el-table-column prop="billDate" label="开单日期"></el-table-column>
                    <el-table-column prop="verifyDate" label="审核日期"></el-table-column>
                    <el-table-column prop="stockTotype" label="单据类型" :formatter="formatType"></el-table-column>
                    <el-table-column prop="isConfirm" label="单据状态" :formatter="formatStatus"></el-table-column>
                    <el-table-column prop="empName" label="操作人"></el-table-column>
                    <el-table-column prop="remark" label="备注">
                        <template slot-scope="scope">
                            <el-popover
                                    placement="top-start"
                                    title=""
                                    width="200"
                                    trigger="hover"
                                    :content="scope.row.remark">
                                <div  class="text_nowarp" slot="reference">{{scope.row.remark}}</div>
                            </el-popover>
                        </template>
                    </el-table-column>
                    <el-table-column label="操作" width="180">
                        <template slot-scope="scope">
                            <el-button type="primary" size="mini" @click="showDetail(scope.row)">详情 </el-button>
                            <el-button type="primary" size="mini" @click="affirmClick(scope.row)" v-if="scope.row.isConfirm==1">确认审核</el-button>
                        </template>
                    </el-table-column>
                </el-table>
                <div class="pad_15 text_right">
                    <el-pagination background
                                   v-if="paginations.total > 0"
                                   @current-change="handleCurrentChange"
                                   @size-change="handleSizeChange"
                                   :current-page.sync="paginations.page"
                                   :page-size="paginations.page_size"
                                   :page-sizes="[10,15,20,30,40,50]"
                                   :layout="paginations.layout"
                                   :total="paginations.total"
                    ></el-pagination>
                </div>
            </div>
        </div>

        <div class="pad_10 backfff martp_15">
            <div class="pad_10 nav_header">
                <span class="font_14 marrt_20">销售明细</span>
            </div>
            <div>
                <el-table :data="orderDetail" show-summary :summary-method="getSummaries" max-height="400"
                          style="width: 100%">
                    <el-table-column type="index" label="序号"></el-table-column>
                    <el-table-column prop="productName" label="货品名称"></el-table-column>
                    <el-table-column prop="productID" label="货品编号"></el-table-column>
                    <el-table-column prop="disctRate" label="折扣率(%)"></el-table-column>
                    <el-table-column prop="inAmount" label="输入数量"></el-table-column>
                    <el-table-column prop="inUnit" label="选择单位"></el-table-column>
                    <el-table-column prop="maxPrice" label="选择单位单价"></el-table-column>
                    <el-table-column prop="unit" label="最小单位"></el-table-column>
                    <el-table-column prop="price" label="最小单价"></el-table-column>
                    <el-table-column prop="amount" label="最小单位数量合计"></el-table-column>
                    <el-table-column prop="totalPrice" label="合计金额"></el-table-column>
                </el-table>
            </div>
        </div>
        <div  class="martp_20">
            <img :src="imgUrl">
            <div class="pad_20 backfff">
                <el-row >
                    <!--<div class="div-box1">-->
                        <!--<div class="box">1</div>-->
                    <!--</div>-->
                    <el-col :span="3" class="el-cols">
                        <div class="grid-content">
                            <div class="box">1</div>
                            <div class="div1-1">销售订货</div>
                            <div class="div2-1">创建订单</div>
                        </div>
                    </el-col>
                    <!--<div class="div-box2">-->

                    <!--</div>-->
                    <el-col :span="3"  class="el-cols" :offset="3">
                        <div class="grid-content  bg-purple">
                            <div class="box">2</div>
                            <div class="div1-2">销售复核</div>
                            <div class="div2-2">订单复核</div>
                        </div>
                    </el-col>

                    <!--</div>-->
                    <el-col :span="3"  class="el-cols" :offset="3">
                        <div class="grid-content">
                            <div class="box">3</div>
                            <div class="div1-3">销售收发货</div>
                            <div class="div2-3">确认收/发货</div>
                        </div>
                    </el-col>
                </el-row>
            </div>
        </div>
    </div>
</template>

<script>
    import API from "@/api/salesBusiness";
    import util from "@/components/js/util"

    export default {
        data() {
            return {
                loading: false,
                dialogVisible: false,
                downloadLoading:false,
                start_end_data: "", //时间
                warehouseID: "", // 入库仓库
                isConfirm: "1",  // 单据状态
                billID: "",
                stockDestID:'',
                clientID:"",//客户商
                siteIDName:"",
                stockTotype:"", //单据类型
                clientList:[], //客户商列表
                warehouseList: [], //仓库列表
                imgUrl:require('../../../assets/img/biaoti.png'),
                status: [
                    {name: "未审核", id: "1"},
                    {name: "已审核", id: "3"},
                    {name: "全部", id: ""},
                ],
                orderType: [
                    {name: "销售订单", id: "OS"},
                    {name: "退货订单", id: "SI"},
                    {name: "全部", id: ""},

                ],
                orderTypeID:"",
                tableData: [],
                orderDetail: [],
                //需要给分页组件传的信息
                paginations: {
                    page: 1, // 当前位于哪页
                    total: 0, // 总数
                    page_size: 10, // 1页显示多少条
                    layout: "total,sizes, prev, pager, next,jumper" // 翻页属性
                },
            };
        },
        methods: {
            // 单据类型
            formatType: function (row) {
                if (row.stockTotype == "SI") {
                    return "退货订单";
                } else{
                    return "销售订单";
                }
            },
            // 单据状态
            formatStatus: function (row) {

                if (row.isConfirm == 0) {
                    return "未发送";
                } else if (row.isConfirm == 1) {
                    return "未审核";
                } else if (row.isConfirm == 2) {
                    return "已作废";
                } else if (row.isConfirm == 3) {
                    return "已审核";
                } else if (row.isConfirm == 4) {
                    return "已发货";
                }else {
                    return "已完成";
                }
            },
            handleDownload(){
                var that=this;
                var start = "";
                var end = "";
                if (that.start_end_data != "" && that.start_end_data != null) {
                    start = util.formatDate.format(that.start_end_data[0], "Y-MM-DD");
                    end = util.formatDate.format(that.start_end_data[1], "Y-MM-DD");
                }
                let params={
                    start: start,
                    end: end,
                    isConfirm: that.isConfirm,
                    stockDest: that.warehouseID,
                    clientID:that.clientID,
                    stockTotype:that.stockTotype,
                }
                that.downloadLoading=true
                API.exportFH(params)
                    .then(res => {
                        this.$message.success({
                            message: '正在导出',
                            duration: '4000'
                        })
                        const link = document.createElement("a");
                        let blob = new Blob([res], {type: 'application/octet-stream'});
                        link.style.display = "none";
                        link.href = URL.createObjectURL(blob);
                        // link.download = res.headers['content-disposition'] //下载后文件名
                        link.download = '销售复核.xlsx'; //下载的文件名
                        document.body.appendChild(link);
                        link.click();
                        document.body.removeChild(link);
                    })
                    .finally(function () {
                        that.downloadLoading = false;
                    });
            },
            getSummaries(param) {
                const {columns, data} = param;
                const sums = [];
                columns.forEach((column, index) => {
                    if (index === 0) {
                        sums[index] = '合计';
                        return;
                    }
                    if(index==3|| index==4 ||index==6 ||index==8 ||index==9){
                        sums[index] = '';
                        return
                    }
                    const values = data.map(item => Number(item[column.property]));
                    if (!values.every(value => isNaN(value))) {
                        sums[index] = values.reduce((prev, curr) => {
                            const value = Number(curr);
                            if (!isNaN(value)) {
                                return parseFloat((prev + curr).toFixed(2));
                            } else {
                                return prev;
                            }
                        }, 0);
                        if (index===10){
                            sums[index]=sums[index].toFixed(2)
                        }
                        // sums[index] += ' 元';
                    } else {
                        // sums[index] = 'N/A';
                    }
                });

                return sums;
            },
            // 客户商列表
            clientData: function () {
                var that = this;
                API.getClientList()
                    .then(res => {
                        if (res.stateCode == 100) {
                            that.clientList = res.data;
                        } else {
                            that.$message.error({
                                message: res.message,
                                duration: 2000
                            });
                        }
                    })
                    .catch(() => {
                        that.$message.error({
                            message: "网络连接失败，请稍后再试！",
                            duration: 2000
                        });
                    })
            },
            // 仓库列表
            warehouseData: function () {
                var that = this;
                var params = {
                    Type:3,
                }
                API.warehouseList(params)
                    .then(res => {
                        if (res.stateCode == 100) {
                            that.warehouseList = res.data;
                        } else {
                            that.$message.error({
                                message: res.message,
                                duration: 2000
                            });
                        }
                    })
                   
            },
            // 数据显示
            handleSearch: function () {
                let that = this;
                that.paginations.page = 1;
                that.billID = "";
                that.orderDetail = [];
                that.search();
            },
            // 数据显示
            search: function () {
                let that = this;
                that.loading = true;
                var start = "";
                var end = "";
                if (that.start_end_data != "" && that.start_end_data != null) {
                    start = util.formatDate.format(that.start_end_data[0], "Y-MM-DD");
                    end = util.formatDate.format(that.start_end_data[1], "Y-MM-DD");
                }
                var params = {
                    start: start,
                    end: end,
                    isConfirm: that.isConfirm,
                    stockDest: that.warehouseID,
                    clientID:that.clientID,
                    stockTotype:that.stockTotype,
                    pageNum: that.paginations.page,
                    pageSize: that.paginations.page_size
                }
                API.salesReviewList(params)
                    .then(res => {
                        if (res.stateCode == 100) {
                            that.tableData = res.data.list;
                            that.paginations.total = res.data.total;
                            if (res.data.list.length > 0) {
                                that.billID = res.data.list[0].billID;
                                that.stockDestID = res.data.list[0].stockDestID;
                                that.orderDetailData()
                            }
                        } else {
                            that.$message.error({
                                message: res.message,
                                duration: 2000
                            });
                        }
                    })
                    .finally(function () {
                        that.loading = false;
                    });
            },
            // 上下分页
            handleCurrentChange(page) {
                var that = this;
                that.paginations.page = page;
                that.search();
            },
            //每页条数
            handleSizeChange(val) {
                var that = this;
                that.paginations.page_size = val;
                that.search();
            },
            //详情
            showDetail: function (row) {

                var that = this;
                if(row.isConfirm ==1){
                    that.$router.push({path: "/salesBusiness/salesReview/editSalesReview", query: {detail: row}})
                }else {
                    that.$router.push({path: "/salesBusiness/salesReview/salesReviewDetail", query: {detail: row}})
                }
            },
            // 明细
            orderDetailData: function () {
                var that = this;
                that.loading = true;
                var params = {
                    billID: that.billID,
                    stockDestID:that.stockDestID,
                }
                API.salesQuotationOrders(params)
                    .then(res => {
                        if (res.stateCode == 100) {
                            res.data.forEach(function (item) {
                                item.totalPrice = (item.inAmount * item.maxPrice * item.disctRate).toFixed(2);
                                item.disctRate = item.disctRate * 100;
                            })
                            that.orderDetail = res.data;
                        } else {
                            that.$message.error({
                                message: res.message,
                                duration: 2000
                            });
                        }
                    })
                    .finally(function () {
                        that.loading = false;
                    });
            },
            // 确认审核
            affirmClick:function(row){
                var that = this;
                    this.$confirm('确认审核订单, 是否继续?', '提示', {
                        confirmButtonText: '确定',
                        cancelButtonText: '取消',
                        type: 'warning'
                    }).then(() => {
                        let params = {
                            billID: row.billID,
                            stockDestID:row.stockDestID,
                        }
                        API.salesQuotationOrders(params)
                            .then(res => {
                                if (res.stateCode == 100) {
                                    let params = {
                                        billID:row.billID,
                                        isConfirm:5,
                                        verifyDate:"",
                                        billDate:row.billDate,
                                        stockTotype:row.stockTotype,
                                        saleOrderDEntities:res.data
                                    }
                                    let flag = res.data.findIndex(p => p.amount > p.stock)
                                    if (flag != -1 &&row.stockTotype!='SI') {
                                        that.$message.error({
                                            message: "出库数量不能大于库存数量！",
                                        });
                                        return
                                    }
                                    that.loading = true;
                                    API.affirmData(params)
                                        .then(res => {
                                            if (res.stateCode == 100) {
                                                that.$message({
                                                    message: "审核成功",
                                                });
                                                that.search();
                                            } else {
                                                that.$message.error({
                                                    message: res.message,
                                                    duration: 2000
                                                });
                                            }
                                        })
                                        .finally(function () {
                                            that.loading = false;
                                        });
                                } else {
                                    that.$message.error({
                                        message: res.message,
                                        duration: 2000
                                    });
                                }
                            })
                            .finally(function () {
                                that.loading = false;
                            });
                        })
                        .catch(() => {
                            this.$message({
                                type: 'info',
                                message: '已取消审核'
                            });
                        });
                },
            // 仓库明细
            detailClick: function (row) {
                var that = this;
                that.billID = row.billID;
                that.stockDestID = row .stockDestID;
                that.orderDetailData();
            }
        },
        mounted() {
            var that = this;
            that.siteIDName = localStorage.getItem("siteIDName");
            that.warehouseData();
            that.clientData();
            that.handleSearch();
        }
    };
</script>


<style scoped lang="scss">
    .el-row {
        margin-bottom: 20px;
        &:last-child {
            margin-bottom: 0;
        }
    }
    .el-cols {
        border-radius: 8px;
        border: 1px solid rgb(238, 238, 238);
    }

    .bg-purple {
        background: rgb(244, 251, 255);
    }
    .bg-purple-light {
        background: white;
    }
    .grid-content {
        border-radius: 4px;
        min-height: 90px;
    }
    .div1-1{
        margin-left:30%;
        font-size: 15px
    }
    .div2-1{
        margin-left:34%;
        padding-top: 5px;
    }
    .div1-2{
        margin-left:30%;
        font-size: 15px
    }
    .div2-2{
        margin-left:34%;
        padding-top: 5px;
    }
    .div1-3{
        margin-left:30%;
        font-size: 15px
    }
    .div2-3{
        margin-left:33%;
        padding-top: 5px;

    }
    .div1-4{
        margin-left:30%;
        font-size: 15px
    }
    .div2-4{
        margin-left:33%;
        padding-top: 5px;
    }
    .box{
        width:30px;
        height:30px;
        line-height:30px;
        border-radius:15px;
        font-size:16px;
        background:hsl(210, 95%, 59%);
        color:white;
        text-align:center;
        margin-left:-15px;
        margin-top:-12px;
    }


</style>
