<template>
    <div>
        <p class="agency-top-text">所有账号</p>
        <div class="agency-box">
            <Row>
                <Select v-model="channelPaymentId" style="width: 150px" placeholder="请选择">
                    <Option v-for="item in statusList" :key='item.id' :value="item.id">{{item.paymentName}}</Option>
                </Select>
                <Input   placeholder="搜索账号" style="width: 150px; margin-left:10px;" v-model="account" class="fl"/>
                <DatePicker type="date" placeholder="请选择日期" v-model="startTime" style="width: 200px"></DatePicker>
                <Button type="primary" style="margin-left:10px;" @click="searches">搜索</Button>
            </Row>    
           <Table  highlight-row ref="currentRowTable" :columns="columns3" :data="dataList" style="margin-top:20px;"></Table>
        </div>
    </div>
</template>
<script>
import {
 accountList,payList
} from "@/api/index";
export default {
    data(){
        return{
            statetext:'',
            channelPaymentId:'',
            account:'',
            startTime:'',
            statusList:[],
            dataList:[],
            columns3:[
                {
                    title: '账号拥有者',
                    key: 'userName'
                },
                {
                    title: '手机标识',
                    key: 'phoneId'
                },
                 {
                    title: '账号实名',
                    key: 'alipayusername'
                },
                 {
                    title: '账号',
                    key: 'account'
                },
                 {
                    title: '账号类型',
                    key: 'accounttype'
                },
                 {
                    title: '单日交易额',
                    key: 'accountAmount'
                },
                 {
                    title: '今日订单量',
                    key: 'accountOrderCount'
                },
                 {
                    title: '今日成功订单量',
                    key: 'accountOrderSucCount'
                },
                 {
                    title: '今日成功率',
                    key: 'sucRates'
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
            ]
        }
    },
    mounted(){
        this.paymethode()
        this.suonumber()
    },
    methods:{
           paymethode(){
                let params={
                pageNumber:"1",
                pageSize:"3"
             }
            payList(params).then(res => {
                 this.statusList=res.data.list
                }).catch(err => {
                    this.treeLoading = false;
                });
        },
        suonumber(){
            let params={"pageVo":{"pageSize":10,"pageNumber":"1"},
                        "accountPhonesVo":{
                           
                            account:this.account,
                            channelPaymentId:this.channelPaymentId
                        },
                        "searchVo":{
                            "startDate":this.startTime
                            }
                        }
           accountList(params).then(res => {
               console.log(res)
                this.dataList=res.data.list
                }).catch(err => {
                    this.treeLoading = false;
                }); 
        },
        searches(){
            this.suonumber() 
        }
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
