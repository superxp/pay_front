<template>
    <div>
        <p class="agency-top-text">订单管理</p>
        <Row style="margin-top:14px;" type="flex" justify="space-between">
            <Col :span="4" class="balance-box">
                <div class="balance-box-left">
                    <Icon type="logo-yen" class="balance-box-icon" size="30"/>
                </div>
                <div class="balance-box-right">
                    <p>订单金额</p>
                    <div>{{generalAccount.allAmount}}元</div>
                </div>
            </Col>
            <Col :span="4" class="balance-box">
                <div class="balance-box-left">
                    <Icon type="logo-yen" class="balance-box-icon" size="30"/>
                </div>
                <div class="balance-box-right">
                    <p>手续费</p>
                    <div>{{generalAccount.sysAmount}}元</div>
                </div>
            </Col>
            <Col :span="4" class="balance-box">
                <div class="balance-box-left">
                    <Icon type="logo-yen" class="balance-box-icon" size="30"/>
                </div>
                <div class="balance-box-right">
                    <p>交易笔数</p>
                    <div>{{generalAccount.total}}笔</div>
                </div>
            </Col>
            <Col :span="4" class="balance-box">
                <div class="balance-box-left">
                    <Icon type="logo-yen" class="balance-box-icon" size="30"/>
                </div>
                <div class="balance-box-right">
                    <p v-if="this.orderVo.status == 1">成功率</p>
                    <p v-else="this.orderVo.status == 0">失败率</p>
                    <div v-if="this.orderVo.status == -1">--</div>
                    <div v-else>{{generalAccount.successRatio}}</div>
                </div>
            </Col>
            <Col :span="4" class="balance-box">
                <div class="balance-box-left">
                    <Icon type="logo-yen" class="balance-box-icon" size="30"/>
                </div>
                <div class="balance-box-right">
                    <p>用户收入</p>
                    <div>{{generalAccount.allUserMount}}元</div>
                </div>
            </Col>
        </Row>
        <div class="agency-box" style="margin-top:10px;">
            <Row>
                <Input placeholder="系统订单号" style="width: 150px; margin-left:10px;" v-model="orderVo.orderNo"
                       class="fl"/>
                <Input placeholder="商户订单号" style="width: 150px; margin-left:10px;" v-model="orderVo.underOrderNo"
                       class="fl"/>
                <Input placeholder="商户号" style="width: 150px; margin-left:10px;" v-model="orderVo.merchant" class="fl"/>
                <!-- <Input   placeholder="创建时间" style="width: 150px; margin-left:10px;" v-model="account" class="fl"/> -->
                <!--<FormItem label="订单状态">-->


                <!--</FormItem>-->


            </Row>
            <Row>
                <Col span="3">
                    <Select v-model="orderVo.status" style="margin-left:10px;">
                        <Option v-for="item in statusList" :key='item.id' :value="item.title">{{item.name}}</Option>
                    </Select>
                </Col>
                <Col span="3">
                    <Select v-model="orderVo.isHistory" style="margin-left:10px;">
                        <Option v-for="item in isHistoryList" :key='item.id' :value="item.title">{{item.name}}</Option>
                    </Select>
                </Col>
                <Col span="3">
                    <Select v-model="orderVo.channelId" placeholder="选择通道" style="margin-left:10px;">
                        <Option v-for="item in channelList" :key='item.id' :value="item.id">{{item.channelName}}</Option>
                    </Select>
                </Col>

            </Row>
            <Row>
                <Col span="3">
                    <Date-picker type="datetime" placeholder="选择起始时间" format="yyyy-MM-dd HH:mm:ss"
                                 v-model="searchVo.startDate" style="width: 200px; margin-left:10px;" ></Date-picker>
                </Col>
                <Col span="3">
                    <Date-picker type="datetime" placeholder="选择结束时间" format="yyyy-MM-dd HH:mm:ss"
                                 v-model="searchVo.endDate" style="width: 200px; margin-left:10px;"></Date-picker>
                </Col>

            </Row>
            <Row>
                <Button type="primary" @click="query()" style="margin-left:10px;">查询</Button>
            </Row>
            <Table highlight-row ref="currentRowTable" :columns="columns3"  :data="dataList"
                   style="margin-top:10px;" stripe></Table>
            <Row type="flex" justify="end" style="margin-top:10px;">
                <Page :total="totals" :page-size="table_limit" show-total :current='table_current' @on-change="ccount"/>
            </Row>
        </div>
        <div>

        </div>
    </div>
</template>
<script>
    import {
        getChannel,
        orderList,
        sendNotify
    } from "@/api/index";

    export default {
        data() {
            return {
                totals: 0,
                table_limit: 30,
                table_current: 1,
                orderVo: {
                    underOrderNo: '',
                    orderVo: '',
                    orderNo: '',
                    merchant: '',
                    status: '-1',
                    channelId: '',
                    isHistory: '0'
                }, searchVo: {
                    startDate: '',
                    endDate: ''
                },
                channelList: [

                ],
                isHistoryList: [
                    {
                        title: '0',
                        name: '今日订单'
                    },
                    {
                        title: '1',
                        name: '历史订单'
                    }, {
                        title: '-1',
                        name: '所有订单'
                    },
                ],
                statusList: [
                    {
                        title: '-1',
                        name: '所有'
                    },{
                        title: '1',
                        name: '已支付'
                    },
                    {
                        title: '0',
                        name: '未支付'
                    },
                ],
                dataList: [],
                generalAccount: {
                    sysAmount: "",
                    allAmount: "",
                    allUserMount: "",
                    allOrderRate: "",
                    allAgentAmount: "",
                    successRatio: "",
                    total: ''
                },
                columns3: [
                    {
                        title: '商户号',
                        key: 'merchant'
                    },
                    {
                        title: '系统订单',
                        key: 'orderNo',
                        width: 200,
                        //fixed: 'left',
                    },
                    {
                        title: '商户订单',
                        key: 'underOrderNo'
                    },
                    {
                        title: '创建时间',
                        key: 'createTime'
                    },
                    {
                        title: '订单金额',
                        key: 'amount'
                    },
                    {
                        title: '费率',
                        key: 'orderRate'
                    },
                    {
                        title: '手续费',
                        key: 'sysAmount'
                    },
                   /* {
                        title: '平台收入',
                        key: 'sysAmount'
                    },*/
                    {
                        title: '状态',
                        key: 'status',
                        render: (h, params) => {
                            let status = params.row.status
                            if (status == '0') {
                                status = '未支付'
                            } else if (status == '1') {
                                status = '已支付'
                            }
                            return h('span', status)
                        }
                    },
                    {
                        title: '操作',
                        key: 'createBy',
                        fixed: 'right',
                        render: (h, params) => {
                            if (params.row.updateBy == '1') {
                                return h('div', [
                                    h('Button', {
                                        props: {
                                            size: 'small',
                                            type: 'success'
                                        },
                                        style: {
                                            marginRight: '10px'
                                        },
                                        on: {
                                            click: () => {
                                                this.sendNotify(params.row)
                                            }
                                        }
                                    }, '补单')
                                ])
                            } else {
                                return ''
                            }
                        }
                    },
                ]
            }
        },
        mounted() {
            this.ccount()
        },
        methods: {
            ccount(page) {
                let params = {
                    "pageVo": {"pageSize": this.table_limit, "pageNumber": page},
                    "orderVo": this.orderVo,
                    "searchVo": this.searchVo
                }
                orderList(params).then(res => {
                    console.log(res)
                    let lst = res.data.pageInfo.list.map((item)=>{
                        item.extend = JSON.parse(item.extend)
                        return item;
                    })
                    this.dataList = lst
                    console.log(lst)
                    this.totals = res.data.pageInfo.total
                    this.generalAccount = res.data.generalAccount,
                        this.generalAccount.total = res.data.pageInfo.total
                }).catch(err => {
                    this.treeLoading = false;
                });
                getChannel(params).then(res => {

                    res.data.list.push({
                        id:"",
                        channelName:"无"
                    })
                    console.log(res.data.list)
                    this.channelList = res.data.list;

                }).catch(err => {
                    this.treeLoading = false;
                });
            }, sendNotify(v) {
                this.$Modal.confirm({
                    title: "补单",
                    content: "您确认补单 " + v.underOrderNo + " ?",
                    onOk: () => {
                        this.operationLoading = true;
                        sendNotify(v.orderNo).then(res => {
                            this.operationLoading = false;
                            this.$Message.success("补单成功");
                            this.query();

                        });
                    }
                });
            }, query(page) {
                let params = {
                    "pageVo": {"pageSize": this.table_limit, "pageNumber": page},
                    "orderVo": this.orderVo,
                    "searchVo": this.searchVo
                }
                orderList(params).then(res => {
                    console.log(res)
                    //this.dataList = res.data.pageInfo.list
                    let lst = res.data.pageInfo.list.map((item)=>{
                        item.extend = JSON.parse(item.extend)
                        return item;
                    })
                    this.dataList = lst
                    console.log(lst)
                    this.totals = res.data.pageInfo.total
                    this.generalAccount = res.data.generalAccount,
                        this.generalAccount.total = res.data.pageInfo.total
                }).catch(err => {
                    this.treeLoading = false;
                });
            }, reset() {
                this.orderVo.underOrderNo = '';
                this.orderVo.orderNo = '';
                this.orderVo.merchant = '';
                this.orderVo.status = '1';
                this.orderVo.isHistory = '0';
                this.orderVo.extend = '0';
                this.searchVo.startDate = '';
                this.searchVo.endDate = '';
                console.log(this.orderVo)
                console.log(this.searchVo)
            }
        }
    }
</script>
<style lang="less" scoped>
    .agency-top-text {
        font-size: 24px;
        font-weight: 700;
        color: #333;
        margin-top: 30px 0 20px 30px;
    }

    .agency-box {
        width: 100%;
        height: 100%;
        padding: 15px;
        margin-top: 40px 0 20px 30px;
        background: #fff;
    }

    .fl {
        margin-right: 10px;
    }

    .balance-box {
        width: 300px;
        height: 90px;
    }

    .balance-box-left {
        height: 90px;
        width: 90px;
        float: left;
        background: #009ABF;
    }

    .balance-box-icon {
        color: #fff;
        margin-top: 25px;
        margin-left: 25px;
    }

    .balance-box-right {
        height: 90px;
        width: 200px;
        background: #00C0EF;
        float: left;
    }

    .balance-box-right p {
        color: #fff;
        font-size: 14px;
        margin-left: 10px;
        line-height: 40px;
    }

    .balance-box-right div {
        color: #fff;
        font-size: 20px;
        height: 50px;
        border-top: 2px solid #fff;
        line-height: 50px;
        padding-left: 10px;
    }
</style>
