<template>
    <div>

        <p class="agency-top-text">有赞店铺管理</p>
        <div class="agency-box">
            <Modal
                    v-model="modal1"
                    title="添加账号"
                    @on-ok="ok"
                    @on-cancel="cancel">
                <Form :label-width="80" v-model='fromData'>
                    <FormItem label="店铺名称">
                        <Input v-model="fromData.accountName" placeholder="请输入用户名"/>
                    </FormItem>
                    <FormItem label="密码">
                        <Input v-model="fromData.password" placeholder="请输入密码"/>
                    </FormItem>
                    <FormItem label="手机号">
                        <Input v-model="fromData.phone" placeholder="请输入邮箱"/>
                    </FormItem>
                    <FormItem label="限额">
                        <Input v-model="fromData.limitAmount" placeholder="请输入手机号"/>
                    </FormItem>
                    <FormItem label="cookie">
                        <Input v-model="fromData.cookie" placeholder="请输入cookie"/>
                    </FormItem>
                    <FormItem label="扩展字段">
                        <Input v-model="fromData.extent" placeholder="请输入备注"/>
                    </FormItem>
                    <FormItem label="备注">
                        <Input v-model="fromData.mark" placeholder="请输入备注"/>
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
                    title="编辑账号"
                    @on-ok="ok2"
                    @on-cancel="cancel2">
                <Form :label-width="80" v-model='fromlk'>
                    <FormItem label="店铺名称">
                        <Input v-model="fromlk.accountName" placeholder="请输入用户名"/>
                    </FormItem>
                    <FormItem label="密码">
                        <Input v-model="fromlk.password" placeholder="请输入密码"/>
                    </FormItem>
                    <FormItem label="手机号">
                        <Input v-model="fromlk.phone" placeholder="请输入邮箱"/>
                    </FormItem>
                    <FormItem label="限额">
                        <Input v-model="fromlk.limitAmount" placeholder="请输入手机号"/>
                    </FormItem>
                    <FormItem label="cookie">
                        <Input v-model="fromlk.cookie" placeholder="请输入cookie"/>
                    </FormItem>
                    <FormItem label="扩展字段">
                        <Input v-model="fromlk.extent" placeholder="请输入备注"/>
                    </FormItem>
                    <FormItem label="备注">
                        <Input v-model="fromlk.mark" placeholder="请输入备注"/>
                    </FormItem>
                    <FormItem label="状态">
                        <Select v-model="fromlk.status">
                            <Option v-for="item in statusList" :key='item.id' :value="item.title">{{item.name}}</Option>
                        </Select>
                    </FormItem>
                </Form>
            </Modal>
            <Modal
                    v-model="modal3"
                    title="商铺管理"
                    @on-ok="ok3"
                    @on-cancel="cancel3"
                    width="760px;"
            >
                <Button type="primary" @click="modal5 = true">添加商品</Button>
                <Modal
                        v-model="modal5"
                        title="添加商品"
                        @on-ok="ok5"
                        @on-cancel="cancel5"
                >
                    <Form :label-width="80" v-model='addcompile'>
                        <FormItem label="skuId">
                            <Input v-model="addcompile.skuId" placeholder="请输入skuId"/>
                        </FormItem>
                        <FormItem label="goodsId">
                            <Input v-model="addcompile.goodsId" placeholder="请输入goodsId"/>
                        </FormItem>
                        <FormItem label="groupId">
                            <Input v-model="addcompile.groupId" placeholder="groupId"/>
                        </FormItem>
                        <FormItem label="amount">
                            <Input v-model="addcompile.amount" placeholder="请输入amount"/>
                        </FormItem>
                        <FormItem label="stock">
                            <Input v-model="addcompile.stock" placeholder="请输入stock"/>
                        </FormItem>
                    </Form>
                </Modal>
                <Table highlight-row ref="currentRowTable" :columns="columns4" :data="dataList4"
                       style="margin-top:20px;"></Table>


            </Modal>
            <Modal
                    v-model="modal4"
                    title="编辑商铺"
                    @on-ok="ok4"
                    @on-cancel="cancel4"
            >
                <Form :label-width="80" v-model='compile'>
                    <FormItem label="id">
                        <Input v-model="compile.id" placeholder="请输入用id"/>
                    </FormItem>
                    <FormItem label="skuId">
                        <Input v-model="compile.skuId" placeholder="请输入skuId"/>
                    </FormItem>
                    <FormItem label="yzAccountId">
                        <Input v-model="compile.yzAccountId" placeholder="请输入skuId"/>
                    </FormItem>
                    <FormItem label="goodsId">
                        <Input v-model="compile.goodsId" placeholder="请输入goodsId"/>
                    </FormItem>
                    <FormItem label="groupId">
                        <Input v-model="compile.groupId" placeholder="groupId"/>
                    </FormItem>
                    <FormItem label="amount">
                        <Input v-model="compile.amount" placeholder="请输入amount"/>
                    </FormItem>
                    <FormItem label="stock">
                        <Input v-model="compile.stock" placeholder="请输入stock"/>
                    </FormItem>
                </Form>
            </Modal>
            <Button type="primary" @click="modal1 = true">添加店铺</Button>
            <Table highlight-row ref="currentRowTable" :columns="columns3" :data="dataList"
                   style="margin-top:20px;"></Table>
            <Row type="flex" justify="end" style="margin-top:10px;">
                <Page :total="totals" :page-size="table_limit" show-total :current='table_current'
                      @on-change="contains"/>
            </Row>
        </div>
    </div>
</template>
<script>
    import {
        yzAccount,
        insertYzAccount,
        updateYzAccount,
        getYzGoods,
        updateYzGoods,
        deleteYzAccount,
        insertYzGoods
    } from "@/api/index";

    export default {
        data() {
            return {
                totals: 0,
                table_limit: 10,
                table_current: 1,
                removeid: '',
                id: '',
                modal5: false,
                modal1: false,
                modal2: false,
                modal3: false,
                modal4: false,
                fromData: {
                    accountName: '',
                    password: '',
                    extar: '',
                    mark: '',
                    limitAmount: "",
                    cookie: "",
                    // returnAddressId: '',
                    //createTime: '',
                    status: '',
                    // todayAmount: "",
                    phone: '',
                    // totalAmount:''
                },
                fromlk: {
                    id: '',
                    accountName: '',
                    password: '',
                    extar: '',
                    mark: '',
                    limitAmount: "",
                    cookie: "",
                    //returnAddressId: '',
                    //createTime: '',
                    status: '',
                    // todayAmount: "",
                    phone: '',
                    // totalAmount:''
                },
                compile: {
                    id: '',
                    skuId: '',
                    yzAccountId: '',
                    goodsId: '',
                    groupId: '',
                    amount: '',
                    stock: ''
                },
                addcompile: {
                    yzAccountId: '',
                    skuId: '',
                    goodsId: '',
                    groupId: '',
                    amount: '',
                    stock: ''
                },
                statusList: [
                    {
                        title: 'true',
                        name: '启用'
                    },
                    {
                        title: 'false',
                        name: '禁用'
                    },
                ],
                columns3: [
                    //  {
                    //     type: 'index',
                    //     width: 80,
                    //     align: 'center'
                    // },
                    {
                        title: '用户ID',
                        key: 'id'
                    },
                    {
                        title: '店铺名称',
                        key: 'accountName'
                    },
                    {
                        title: '密码',
                        key: 'password'
                    },
                    {
                        title: '手机号',
                        key: 'phone'
                    },
                    {
                        title: '限额',
                        key: 'limitAmount'
                    },
                    {
                        title: 'cookie',
                        key: 'cookie',
                        width: 350,
                        //  className: 'demo-table-info-column'
                        ellipsis: true
                    },
                    {
                        title: 'sessionId',
                        key: 'kdtSessionId'
                    },
                    {
                        title: 'kdtId',
                        key: 'kdtId'
                    },
                    {
                        title: '今日已收金额',
                        key: 'todayAmount'
                    },
                    {
                        title: '累计已收金额',
                        key: 'totalAmount'
                    },
                    {
                        title: '备注',
                        key: 'mark'
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
                                            value: params.row.status == true
                                        },
                                        style: {
                                            marginRight: '5px'
                                        },
                                        on: {
                                            'on-change': (value) => {
                                                this.fromlk.id = params.row.id
                                                this.fromlk.status = value
                                                this.bianji(params)


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
                        width: 200,
                        title: '操作',
                        render: (h, params) => {
                            return h('div', [
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
                                            this.modal3 = true
                                            this.id = params.row.id
                                            this.delete()
                                        }
                                    }
                                }, '管理'),
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
                                            this.fromlk.extar = params.row.extar
                                            this.fromlk.mark = params.row.mark
                                            this.fromlk.accountName = params.row.accountName
                                            this.fromlk.password = params.row.password
                                            this.fromlk.status = params.row.status
                                            this.fromlk.limitAmount = params.row.limitAmount
                                            this.fromlk.id = params.row.id
                                            this.fromlk.phone = params.row.phone
                                            this.fromlk.cookie = params.row.cookie
                                            this.fromlk.returnAddressId = params.row.returnAddressId
                                            this.fromlk.createTime = params.row.createTime
                                        }
                                    }
                                }, '编辑'),
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
                                            this.removeid = params.row.id
                                            this.remove()
                                        }
                                    }
                                }, '删除'),
                            ])
                        }
                    },
                ],
                dataList: [],
                columns4: [
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
                        title: 'skuId',
                        key: 'skuId'
                    },
                    {
                        title: 'goodsId',
                        key: 'goodsId'
                    },
                    {
                        title: 'amount',
                        key: 'amount'
                    },
                    {
                        title: 'stock',
                        key: 'stock'
                    },
                    {
                        title: '操作',
                        render: (h, params) => {
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
                                            //   this.modal3=false
                                            this.modal4 = true
                                            this.compile.id = params.row.id
                                            this.compile.skuId = params.row.skuId
                                            this.compile.yzAccountId = params.row.yzAccountId
                                            this.compile.goodsId = params.row.goodsId
                                            this.compile.groupId = params.row.groupId
                                            this.compile.amount = params.row.amount
                                            this.compile.stock = params.row.stock
                                            //     this.workid=params.row.id
                                            //     this.withdrawsty()
                                        }
                                    }
                                }, '编辑')
                            ])
                        }
                    },
                ],
                dataList4: []
            }
        },
        mounted() {
            this.contains()
        },
        methods: {
            contains(page) {
                let params = {
                    "pageNumber": page,
                    "pageSize": 10
                }
                yzAccount(params).then(res => {
                    console.log(res)
                    this.dataList = res.data.list
                    this.totals = res.data.total

                }).catch(err => {
                    this.treeLoading = false;
                });
            },
            ok() {
                let params = this.fromData
                insertYzAccount(params).then(res => {
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
                updateYzAccount(params).then(res => {
                    if (res.status == 0) {
                        this.$Message.info('编辑成功');
                        this.contains()
                    }
                }).catch(err => {
                    this.treeLoading = false;
                });
            },
            bianji() {
                let params = {
                    'id': this.fromlk.id,
                    'status': this.fromlk.status
                }
                updateYzAccount(params).then(res => {
                    if (res.status == 0) {
                        this.$Message.info('编辑成功');
                        this.contains()
                    }
                }).catch(err => {
                    this.treeLoading = false;
                });
            },
            cancel2() {
            },
            ok3() {

            },
            cancel3() {
            },
            ok4() {
                let params = this.compile
                updateYzGoods(params).then(res => {
                    if (res.status == 0) {
                        this.$Message.info('编辑成功');
                        this.delete()
                    }
                }).catch(err => {

                })
            },
            cancel4() {
            },
            ok5() {
                this.addcompile.yzAccountId = this.id
                let params = this.addcompile
                insertYzGoods(params).then(res => {
                    if (res.status == 0) {
                        this.$Message.info('新增成功');
                        this.delete()
                    }
                }).catch(err => {

                })
            },
            delete() {
                let params = {
                    yzAccountId: this.id
                }
                getYzGoods(params).then(res => {
                    console.log(res)
                    this.dataList4 = res.data
                    this.contains()

                }).catch(err => {
                    this.treeLoading = false;
                });
            },
            remove() {
                let params = {id: this.removeid}
                deleteYzAccount(params).then(res => {
                    if (res.status == 0) {
                        this.$Message.info('删除成功');
                        this.contains()
                    }
                }).catch(err => {

                })
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

    .demo-table-info-column {
        height: 60px;
        overflow: hidden;
    }
</style>
