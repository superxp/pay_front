<template>
    <div>

        <p class="agency-top-text">通道商户管理</p>
        <div class="agency-box">
            <Modal
                    v-model="modal1"
                    title="添加通道商户"
                    @on-ok="ok"
                    @on-cancel="cancel">
                <Form :label-width="80" v-model='fromData'>
                    <FormItem label="账号">
                        <Input v-model="fromData.account" placeholder="请输入商户号"/>
                    </FormItem>
                    <FormItem label="密钥">
                        <Input v-model="fromData.signKey" placeholder="请输入密钥"/>
                    </FormItem>
                    <FormItem label="限额">
                        <Input v-model="fromData.limitAmount" placeholder="请输入限额"/>
                    </FormItem>
                    <FormItem label="状态">
                        <Select v-model="fromData.status">
                            <Option v-for="item in statusList" :key='item.id' :value="item.title">{{item.name}}</Option>
                        </Select>
                    </FormItem>
                    <FormItem label="通道">
                        <Select v-model="fromData.channelId">
                            <Option v-for="item in payTypeList" :key='item.id' :value="item.id">{{item.channelName}}</Option>
                        </Select>
                    </FormItem>
                </Form>
            </Modal>
            <Modal
                    v-model="modal2"
                    title="修改通道商户"
                    @on-ok="ok2"
                    @on-cancel="cancel2">
                <Form :label-width="80" v-model='fromlk'>
                    <FormItem label="账号">
                        <Input v-model="fromlk.account" placeholder="请输入商户号"/>
                    </FormItem>
                    <FormItem label="密钥">
                        <Input v-model="fromlk.signKey" placeholder="请输入密钥"/>
                    </FormItem>
                    <FormItem label="限额">
                        <Input v-model="fromlk.limitAmount" placeholder="请输入限额"/>
                    </FormItem>
                    <FormItem label="状态">
                        <Select v-model="fromlk.status">
                            <Option v-for="item in statusList" :key='item.id' :value="item.title">{{item.name}}</Option>
                        </Select>
                    </FormItem>
                    <FormItem label="通道">
                        <Select v-model="fromlk.channelId">
                            <Option v-for="item in payTypeList" :key='item.id' :value="item.id">{{item.channelName}}</Option>
                        </Select>
                    </FormItem>
                </Form>
            </Modal>
            <Button type="primary" @click="modal1 = true">添加通道</Button>
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
        getChannelMchAccount,
        updateChannelMchAccount,
        deleteChannelMchAccount,
        getAllChannel,
        insertChannelMchAccount
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
                fromData: {
                    account: '',
                    signKey: '',
                    publicKey: '',
                    channelId: '',
                    orderNum: '',
                    status: '',
                    limitAmount: "",
                    password: ''
                },
                fromlk: {
                    id: '',
                    account: '',
                    signKey: '',
                    publicKey: '',
                    channelId: '',
                    orderNum: '',
                    status: '',
                    limitAmount: "",
                    password: ''
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
                        width: 60,
                        align: 'center'
                    },
                    {
                        title: 'id',
                        key: 'id'
                    },
                    {
                        title: '通道账号',
                        key: 'account'
                    },
                    {
                        title: '私钥',
                        key: 'signKey'
                    },
                    {
                        title: '公钥',
                        key: 'publicKey'
                    },
                    {
                        title: '通道',
                        key: 'channelId',
                        render :(h,params) => {
                            let payTypeId = params.row.channelId;
                            console.log(params.row.channelId)
                            for (let i = 0; i < this.payTypeList.length ; i++) {

                                if(payTypeId == this.payTypeList[i].id){
                                    return h("div",this.payTypeList[i].channelName);
                                }
                            }
                            return payTypeId;
                        }
                    },
                    {
                        title: '订单量',
                        key: 'orderNum'
                    },
                    {
                        title: '密码',
                        key: 'password'
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
                                                updateChannelMchAccount(channel).then(res => {
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
                getAllChannel().then(res => {
                    console.log(res)
                    this.payTypeList = res.data

                }).catch(err => {
                    this.treeLoading = false;
                });
                getChannelMchAccount(params).then(res => {
                    console.log(res)
                    this.dataList = res.data.list

                }).catch(err => {
                    this.treeLoading = false;
                });
            },
            ok() {
                let params = this.fromData
                insertChannelMchAccount(params).then(res => {
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
                updateChannelMchAccount(params).then(res => {
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
                deleteChannelMchAccount(params).then(res => {
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
