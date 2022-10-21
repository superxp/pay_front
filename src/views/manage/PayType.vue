<template>
    <div>

        <p class="agency-top-text">支付方式管理</p>
        <div class="agency-box">
            <Modal
                    v-model="modal1"
                    title="添加支付方式"
                    @on-ok="ok"
                    @on-cancel="cancel">
                <Form :label-width="80" v-model='fromData'>
                    <FormItem label="支付方式名称">
                        <Input v-model="fromData.name" placeholder="请输入支付方式名称"/>
                    </FormItem>
                    <FormItem label="描述">
                        <Input v-model="fromData.description" placeholder="请输入描述"/>
                    </FormItem>
                    <FormItem label="费率">
                        <Input v-model="fromData.rate" placeholder="请输入费率"/>
                    </FormItem>
                    <FormItem label="编码">
                        <Input v-model="fromData.code" placeholder="请输入编码"/>
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
                    title="添加支付方式"
                    @on-ok="ok2"
                    @on-cancel="cancel2">
                <Form :label-width="80" v-model='fromlk'>
                    <FormItem label="支付方式名称">
                        <Input v-model="fromlk.name" placeholder="请输入支付方式名称"/>
                    </FormItem>
                    <FormItem label="描述">
                        <Input v-model="fromlk.description" placeholder="请输入描述"/>
                    </FormItem>
                    <FormItem label="费率">
                        <Input v-model="fromlk.rate" placeholder="请输入费率"/>
                    </FormItem>
                    <FormItem label="编码">
                        <Input v-model="fromlk.code" placeholder="请输入编码"/>
                    </FormItem>
                    <FormItem label="状态">
                        <Select v-model="fromlk.status">
                            <Option v-for="item in statusList" :key='item.id' :value="item.title">{{item.name}}</Option>
                        </Select>
                    </FormItem>
                </Form>
            </Modal>
            <Button type="primary" @click="modal1 = true">添加支付方式</Button>
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
        getPayType,
        updatePayType,
        deletePayType,
        insertPayType
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
                    name: '',
                    description: '',
                    //pdduid: "",
                    //addressId:"",
                    status: '',
                    code: "",
                    rate: ''
                },
                fromlk: {
                    id: '',
                    name: '',
                    description: '',
                    //pdduid: "",
                    //addressId:"",
                    status: '',
                    code: "",
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
                        title: '支付方式名称',
                        key: 'name'
                    },
                    {
                        title: '描述',
                        key: 'description'
                    },
                    {
                        title: '编码',
                        key: 'code'
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
                                            value: params.row.status === 1
                                        },
                                        style: {
                                            marginRight: '5px'
                                        },
                                        on: {
                                            'on-change': (value) => {
                                                console.log(value)
                                                var payType = {
                                                    id:params.row.id,
                                                    status:value?1:0
                                                }
                                                updatePayType(payType).then(res => {
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
                getPayType(params).then(res => {
                    console.log(res)
                    this.dataList = res.data.list

                }).catch(err => {
                    this.treeLoading = false;
                });
            },
            ok() {
                let params = this.fromData
                insertPayType(params).then(res => {
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
                updatePayType(params).then(res => {
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
                deletePayType(params).then(res => {
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
