<template>
    <div>
         <p class="agency-top-text">通道商户池</p>
          <div class="agency-box">
               <Button type="primary" @click="modal1 = true">添加账号</Button>
               <Modal
                v-model="modal1"
                    title="添加账号"
                    @on-ok="ok"
                    @on-cancel="cancel"
               >
               <Form  :label-width="80" v-model='fromData'>
                        <FormItem label="账号名" >
                            <Input v-model="fromData.account" placeholder="请输入账号"/>
                        </FormItem>
                        <FormItem label="账号秘钥" >
                            <Input v-model="fromData.signkey" placeholder="请输入账号秘钥"/>
                        </FormItem>
                         <FormItem label="账号私钥" >
                           <Input v-model="fromData.privatekey" type="textarea" :autosize="{minRows: 3,maxRows: 6}" placeholder="账号私钥"/>
                        </FormItem>
                        <FormItem label="账号公钥" >
                           <Input v-model="fromData.publikey" type="textarea" :autosize="{minRows: 3,maxRows: 6}" placeholder="账号公钥"/>
                        </FormItem>
                         <Row>
                            <Col span="11">
                                <FormItem label="支付方式"  style="width: 200px;">
                                    <Select v-model="fromData.accountType">
                                        <Option value="1">当面付</Option>
                                        <Option value="2">及时支付</Option>
                                    </Select>
                                </FormItem>
                            </Col>
                             <Col span="11">    
                                <FormItem label="支付方式"  style="width: 200px;">
                                    <Select v-model="fromData.accountType">
                                        <Option value="1">当面付</Option>
                                        <Option value="2">及时支付</Option>
                                    </Select>
                                </FormItem>
                            </Col>
                        </Row>      
                        <FormItem label="限额" >
                            <Input v-model="fromData.dayQuota" placeholder="请输入限额"/>
                        </FormItem>
               </Form>    
               </Modal>    
                <Table  highlight-row ref="currentRowTable" :columns="columns3" :data="dataList"  style="margin-top:10px;"></Table>
          </div>    
    </div>
</template>
<script>
import {
undergroundList,
payList
} from "@/api/index";
export default {
    data(){
        return{
            modal1:false,
            dataList:[],
            fromData:{
                account:'',
                privatekey:'',
                publikey:'',
                dayQuota:'',
                accountType:'',
                signkey:''
            },
            columns3:[
                 {
                        type: 'index',
                        width: 60,
                        align: 'center'
                    },
               {
                    title: '通道商户名',
                    key: 'account'
                },
                 {
                    title: '通道名称',
                    key: 'channelName'
                },
                 {
                    title: '支付方式',
                    key: 'accountType'
                },
                 {
                    title: '今日订单量',
                    key: 'orderNum'
                },
                     {
                    title: '状态',
                    key: 'status',
                    render:(h,params)=>{
                           return h('div',[
                               h('strong',{
                                   style: {
                                       marginRight: '5px'
                                       },
                               }),
                               h('i-switch',{
                                   props: {
                                    type: 'primary',
                                     size:"large",
                                    value: params.row.status===0
                                    },
                                style: {
                                    marginRight: '5px'
                                    },
                                    on:{
                                         'on-change': (value) => {
                                         }
                                    }
                               },
                               [
                                h('span',{
                                      slot: "open",
                                        domProps: {
                                            innerHTML: '开启'
                                        }
                                }),
                                 h('span',{
                                      slot: "close",
                                        domProps: {
                                            innerHTML: '禁用'
                                        }
                                }),
                               ]
                               )
                           ])
                       }
               },
                 {
                    title: '操作',
                    key: 'merchant'
                },
            ]
        }
    },
    mounted(){
        this.underground()
        this.paymethode()
    },
    methods:{
        ok(){},
        cancel(){},
        underground(){
            let params ={
                "pageNumber":"2",
                "pageSize":"3"
                }
            undergroundList(params).then(res => {
              
                this.dataList=res.data.list
            }).catch(err => {
                    this.treeLoading = false;
            });
        },
        // 支付方式
           paymethode(){
                let params={
                pageNumber:"1",
                pageSize:"3"
             }
            payList(params).then(res => {
                 console.log(res)
                }).catch(err => {
                    this.treeLoading = false;
                });
        },
    }
}
</script>
<style lang="less" scoped>
    .agency-top-text{
        font-size: 24px;
        font-weight: 700;
        color: #333;
        margin-top: 30px 0 20px 30px;
    }
    .agency-box{
        width: 100%;
        height: 100%;
        padding: 15px;
        margin-top: 40px 0 20px 30px;
        background: #fff;
    }
    .fl{
        margin-right: 10px;
    }
</style>