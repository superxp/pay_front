<template>
    <div>

        <p class="agency-top-text">通道管理</p>
        <div class="agency-box">
            <Modal
                    v-model="modal1"
                    title="添加支付方式"
                    @on-ok="ok"
                    @on-cancel="cancel">
                <Form :label-width="80" v-model='fromData'>
                    <FormItem label="通道名称">
                        <Input v-model="fromData.channelName" placeholder="请输入通道名称"/>
                    </FormItem>
                    <FormItem label="限制固定金额">
                        <Input v-model="fromData.channelQuota" placeholder="请输入限额"/>
                    </FormItem>
                    <FormItem label="费率">
                        <Input v-model="fromData.rate" placeholder="请输入费率"/>
                    </FormItem>
                    <FormItem label="编码">
                        <Input v-model="fromData.channelCode" placeholder="请输入编码"/>
                    </FormItem>
                    <FormItem label="扩展字段">
                        <Input v-model="fromData.extend" placeholder="请输入编码"/>
                    </FormItem>
                    <FormItem label="请求地址">
                        <Input v-model="fromData.requestUrl" placeholder="请输入请求地址"/>
                    </FormItem>
                    <FormItem label="回调地址">
                        <Input v-model="fromData.notifyUrl" placeholder="请输入回调地址"/>
                    </FormItem>
                    <FormItem label="请求方式">
                        <Input v-model="fromData.contentType" placeholder="请输入回调地址"/>
                    </FormItem>
                    <FormItem label="限额">
                        <Input v-model="fromData.limitAmount" placeholder="请输入限额"/>
                    </FormItem>
                    <FormItem label="每分钟并发">
                        <Input v-model="fromData.limitTimes" placeholder="请输入每分钟并发"/>
                    </FormItem>
                    <FormItem label="状态">
                        <Select v-model="fromData.status">
                            <Option v-for="item in statusList" :key='item.id' :value="item.title">{{item.name}}</Option>
                        </Select>
                    </FormItem>
                    <FormItem label="支付方式">
                        <Select v-model="fromData.payTypeId">
                            <Option v-for="item in payTypeList" :key='item.id' :value="item.id">{{item.name}}</Option>
                        </Select>
                    </FormItem>
                </Form>
            </Modal>
            <Modal
                    v-model="modal3"
                    title="测试通道"
                    @on-ok="testPay()"
                    @on-cancel="testPayBack()">
                <Form :label-width="80" v-model='testData'>
                    <FormItem label="金额">
                        <Input v-model="testData.amount" placeholder="请输入金额"/>
                    </FormItem>
                    <FormItem label="支付方式">
                        <Select v-model="testData.payTypeId">
                            <Option v-for="item in payTypeList" :key='item.id' :value="item.id">{{item.name}}</Option>
                        </Select>
                    </FormItem>
                </Form>
            </Modal>
            <Modal
                    v-model="modal4"
                    title="支付链接"
                    @on-ok="cancel()"
                    @on-cancel="cancel()" style="width: 300px;word-wrap: break-word;word-break: break-all;overflow: hidden;">
                {{testUrl}}
            </Modal>
            <Modal
                    v-model="modal2"
                    title="添加支付方式"
                    @on-ok="ok2"
                    @on-cancel="cancel2">
                <Form :label-width="80" v-model='fromlk'>
                    <FormItem label="通道名称">
                        <Input v-model="fromlk.channelName" placeholder="请输入通道名称"/>
                    </FormItem>
                    <FormItem label="限制固定金额">
                        <Input v-model="fromlk.channelQuota" placeholder="请输入限额"/>
                    </FormItem>
                    <FormItem label="费率">
                        <Input v-model="fromlk.rate" placeholder="请输入费率"/>
                    </FormItem>
                    <FormItem label="编码">
                        <Input v-model="fromlk.channelCode" placeholder="请输入编码"/>
                    </FormItem>
                    <FormItem label="扩展字段">
                        <Input v-model="fromlk.extend" placeholder="请输入编码"/>
                    </FormItem>
                    <FormItem label="请求地址">
                        <Input v-model="fromlk.requestUrl" placeholder="请输入请求地址"/>
                    </FormItem>
                    <FormItem label="回调地址">
                        <Input v-model="fromlk.notifyUrl" placeholder="请输入回调地址"/>
                    </FormItem>
                    <FormItem label="请求方式">
                        <Input v-model="fromlk.contentType" placeholder="请输入回调地址"/>
                    </FormItem>
                    <FormItem label="限额">
                        <Input v-model="fromlk.limitAmount" placeholder="请输入限额"/>
                    </FormItem>
                    <FormItem label="每分钟并发">
                        <Input v-model="fromlk.limitTimes" placeholder="请输入并发"/>
                    </FormItem>
                    <FormItem label="状态">
                        <Select v-model="fromlk.status">
                            <Option v-for="item in statusList" :key='item.id' :value="item.title">{{item.name}}</Option>
                        </Select>
                    </FormItem>
                    <FormItem label="支付方式">
                        <Select v-model="fromData.payTypeId">
                            <Option v-for="item in payTypeList" :key='item.id' :value="item.id">{{item.name}}</Option>
                        </Select>
                    </FormItem>
                </Form>
            </Modal>
            <Button type="primary" @click="modal1 = true">添加通道</Button>
            <Button type="primary" @click="modal3 = true">测试通道</Button>
            <Table highlight-row ref="currentRowTable" :columns="columns3" :data="dataList"
                   style="margin-top:20px;"></Table>
            <Row type="flex" justify="end" style="margin-top:10px;">
                <Page :total="totals" :page-size="table_limit" show-total :current='table_current'/>
            </Row>
        </div>
    </div>
</template>
<script>
    import {
        getChannel,
        updateChannel,
        deleteChannel,
        getAllPayType,
        testOrder,
        insertChannel
    } from "@/api/index";

    export default {
        data() {
            return {
                totals: 0,
                table_limit: 10,
                table_current: 1,
                id: '',
                modal1: false,
                modal2: false,
                modal3: false,
                modal4: false,
                testUrl: "",
                testData:{
                    payTypeId: "",
                    amount:""
                },
                fromData: {
                    channelName: '',
                    channelCode: '',
                    channelQuota: '',
                    payTypeId: '',
                    extend: '',
                    //pdduid: "",
                    //addressId:"",
                    status: '',
                    requestUrl: "",
                    notifyUrl: "",
                    limitAmount: "",
                    contentType: "",
                    limitTimes:"",
                    rate: ''
                },
                fromlk: {
                    id: '',
                    channelName: '',
                    channelCode: '',
                    channelQuota: '',
                    payTypeId: '',
                    extend: '',
                    //pdduid: "",
                    //addressId:"",
                    status: '',
                    requestUrl: "",
                    notifyUrl: "",
                    limitAmount: "",
                    contentType: "",
                    limitTimes:"",
                    rate: ''
                },
                statusList: [
                    {
                        title: 1,
                        name: '启用'
                    },
                    {
                        title: 0,
                        name: '禁用'
                    },
                ],payTypeList: [

                ],
                columns3: [
                    {
                        type: 'index',
                        width: 50,
                        align: 'center'
                    },
                    {
                        title: 'id',
                        width: 50,
                        key: 'id'
                    },
                    {
                        title: '通道名称',
                        key: 'channelName'
                    },
                    {
                        title: '编码',
                        key: 'channelCode'
                    },
                    {
                        title: '限额',
                        key: 'channelQuota'
                    },
                    {
                        title: '每分钟并发',
                        key: 'limitTimes'
                    },
                    {
                        title: '请求方式',
                        key: 'contentType'
                    },
                    {
                        title: '支付方式',
                        key: 'payTypeId',
                        render :(h,params) => {
                            let payTypeId = params.row.payTypeId;
                            console.log(params.row.payTypeId)
                            for (let i = 0; i < this.payTypeList.length ; i++) {

                                if(payTypeId == this.payTypeList[i].id){
                                    return h("div",this.payTypeList[i].name);
                                }
                            }
                            return payTypeId;
                        }
                    },
                    {
                        title: '扩展字段',
                        key: 'extend',
                        width: 350,
                        ellipsis:true
                    },
                    {
                        title: '请求地址',
                        key: 'requestUrl'
                    },
                    {
                        title: '回调地址',
                        key: 'notifyUrl'
                    },
                    {
                        title: '限额',
                        key: 'limitAmount'
                    },
                    {
                        title: '今日已收金额',
                        key: 'todayAmount'
                    },
                    {
                        title: '总收金额',
                        key: 'totalAmount'
                    },
                    {
                        title: '费率',
                        key: 'rate'
                    },
                    {
                        title: '状态',
                        key: 'status',
                        render: (h, params) => {
                            return h('div', [
                                h('strong', {
                                    style: {
                                        marginRight: '5px'
                                    },
                                }),
                                h('i-switch', {
                                        props: {
                                            type: 'primary',
                                            size: "large",
                                            value: params.row.status == '1'
                                        },
                                        style: {
                                            marginRight: '5px'
                                        },
                                        on: {
                                            'on-change': (value) => {
                                                console.log(value)
                                                var channel = {
                                                    id:params.row.id,
                                                    status:value?1:0
                                                }
                                                updateChannel(channel).then(res => {
                                                    this.$Message.info(res.data);
                                                }).catch(err => {
                                                    this.treeLoading = false;
                                                });
                                            }
                                        }
                                    },
                                    [
                                        h('span', {
                                            slot: "open",
                                            domProps: {
                                                innerHTML: '开启'
                                            }
                                        }),
                                        h('span', {
                                            slot: "close",
                                            domProps: {
                                                innerHTML: '禁用'
                                            }
                                        }),
                                    ])
                            ])
                        }
                    },
                    {
                        title: '操作',
                        render: (h, params) => {
                            return h('div', [
                                h('Button', {
                                    props: {
                                        size: 'small',
                                        type: 'error'
                                    },
                                    style: {
                                        marginRight: '10px'
                                    },
                                    on: {
                                        click: () => {
                                            this.id = params.row.id
                                            this.delete()
                                        }
                                    }
                                }, '删除'),
                                h('Button', {
                                    props: {
                                        size: 'small',
                                        type: 'info'
                                    },
                                    style: {
                                        marginRight: '10px'
                                    },
                                    on: {
                                        click: () => {
                                            this.modal2 = true
                                            this.fromlk = params.row
                                        }
                                    }
                                }, '编辑'),

                            ])
                        }
                    },
                ],
                dataList: []
            }
        },
        mounted() {
            this.contains()
        },
        methods: {
            contains(page) {
                let params = {
                    // id:localStorage.getItem('uid'),
                    // parentId:localStorage.getItem('parentId')
                }
                getAllPayType().then(res => {
                    console.log(res)
                    this.payTypeList = res.data

                }).catch(err => {
                    this.treeLoading = false;
                });
                getChannel(params).then(res => {
                    console.log(res)
                    this.dataList = res.data.list

                }).catch(err => {
                    this.treeLoading = false;
                });
            },
            testPay(){
                testOrder(this.testData).then(res => {
                    this.testUrl = res.data.qrCode;
                    this.modal4= true;
                });
            },
            testPayBack(){
                this.modal3 = false;

            },
            ok() {
                let params = this.fromData
                insertChannel(params).then(res => {
                    console.log(res)
                    this.contains()

                }).catch(err => {
                    this.treeLoading = false;
                });
            },
            cancel() {
            },
            ok2() {
                let params = this.fromlk
                updateChannel(params).then(res => {
                    console.log(res)
                    this.contains()
                }).catch(err => {
                    this.treeLoading = false;
                });
            },
            cancel2() {
            },
            delete() {
                let params = {
                    id: this.id
                }
                deleteChannel(params).then(res => {
                    this.$Message.info(res.data);
                    this.contains()

                }).catch(err => {
                    this.treeLoading = false;
                });
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
</style>
