<template>
    <div>

        <p class="agency-top-text">乐盈账号管理</p>
        <div class="agency-box">
            <Modal
                    v-model="modal1"
                    title="添加账号"
                    @on-ok="ok"
                    @on-cancel="cancel">
                <Form :label-width="80" v-model='fromData'>
                    <FormItem label="账号">
                        <Input v-model="fromData.account" placeholder="请输入账号"/>
                    </FormItem>
                    <!--<FormItem label="私钥">
                        <Input v-model="fromData.signKey" placeholder="请输入私钥"/>
                    </FormItem>-->
                    <FormItem label="限额">
                        <Input v-model="fromData.limitAmount" placeholder="请输入限额"/>
                    </FormItem>
                    <FormItem label="密码">
                        <Input v-model="fromData.password" placeholder="请输入商品名称"/>
                    </FormItem>
                    <FormItem label="状态">
                        <Select v-model="fromData.status">
                            <Option v-for="item in statusList" :key='item.id' :value="item.title">{{item.name}}</Option>
                        </Select>
                    </FormItem>
                </Form>
            </Modal>
            <Modal
                    v-model="modal2"
                    title="修改账号"
                    @on-ok="ok2"
                    @on-cancel="cancel2">
                <Form :label-width="80" v-model='fromlk'>
                    <FormItem label="账号">
                        <Input v-model="fromlk.account" placeholder="请输入账号"/>
                    </FormItem>
                    <!--<FormItem label="私钥">
                        <Input v-model="fromlk.signKey" placeholder="请输入限额"/>
                    </FormItem>-->
                    <FormItem label="限额">
                        <Input v-model="fromlk.limitAmount" placeholder="请输入限额"/>
                    </FormItem>
                    <!--<FormItem label="商品名称">
                        <Input v-model="fromlk.createBy" placeholder="请输入商品名称"/>
                    </FormItem>-->
                    <FormItem label="密码">
                        <Input v-model="fromlk.password" placeholder="请输入商品名称"/>
                    </FormItem>
                    <FormItem label="状态">
                        <Select v-model="fromlk.status">
                            <Option v-for="item in statusList" :key='item.id' :value="item.title">{{item.name}}</Option>
                        </Select>
                    </FormItem>
                </Form>
            </Modal>
            <Button type="primary" @click="modal1 = true">添加账号</Button>
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
        getYinlianAccount,
        updateYinlianAccount,
        deleteYinlianAccount,
        loginYinlianAccount,
        getAllChannel,
        insertYinlianAccount
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
                    status: '',
                    limitAmount: "",
                    createBy: "",
                    password: ''
                },
                fromlk: {
                    id: '',
                    account: '',
                    signKey: '',
                    orderNum: '',
                    status: '',
                    limitAmount: "",
                    createBy: "",
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
                        title: '账号',
                        key: 'account'
                    },
                    {
                        title: 'uid',
                        key: 'signKey'
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
                                                updateYinlianAccount(channel).then(res => {
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
                                            this.login()
                                        }
                                    }
                                }, '登录'),
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
                getYinlianAccount(params).then(res => {
                    console.log(res)
                    this.dataList = res.data.list

                }).catch(err => {
                    this.treeLoading = false;
                });
            },
            ok() {
                let params = this.fromData
                insertYinlianAccount(params).then(res => {
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
                updateYinlianAccount(params).then(res => {
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
                deleteYinlianAccount(params).then(res => {
                    this.$Message.info(res.data);
                    this.contains()

                }).catch(err => {
                    this.treeLoading = false;
                });
            }
            ,
            login() {
                loginYinlianAccount(this.id,{}).then(res => {
                    this.$Message.info("登陆成功");
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
