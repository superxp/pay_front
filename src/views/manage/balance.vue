<template>
    <div>
        <p class="agency-top-text">结算管理</p>
        <Row style="margin-top:14px;" type="flex" justify="space-between">
            <Col :span="6" class="balance-box">
                <div class="balance-box-left">
                    <Icon type="logo-yen" class="balance-box-icon" size="36"/>
                </div>
                <div class="balance-box-right">
                    <p>结算总额</p>
                    <div>{{generalAccount.allToAmount}}元</div>
                </div>
            </Col>
             <Col :span="6" class="balance-box">
                <div class="balance-box-left">
                    <Icon type="logo-yen" class="balance-box-icon" size="36"/>
                </div>
                <div class="balance-box-right">
                    <p>手续费</p>
                    <div>{{generalAccount.allRate}}元</div>
                </div>
            </Col>
             <Col :span="6" class="balance-box">
                <div class="balance-box-left">
                    <Icon type="logo-yen" class="balance-box-icon" size="36"/>
                </div>
                <div class="balance-box-right">
                    <p>实际结算</p>
                    <div>{{generalAccount.allAmount}}元</div>
                </div>
            </Col>
             <Col :span="6" class="balance-box">
                <div class="balance-box-left">
                    <Icon type="logo-yen" class="balance-box-icon" size="36"/>
                </div>
                <div class="balance-box-right">
                    <p>结算笔数</p>
                    <div>{{generalAccount.allcount}}笔</div>
                </div>
            </Col>
        </Row>
           <div class="agency-box" style="margin-top:10px;">
               <Row>
                    <Input   placeholder="系统订单号" style="width: 150px; margin-left:10px;" v-model="account" class="fl"/>
                     <Input   placeholder="商户订单号" style="width: 150px; margin-left:10px;" v-model="account" class="fl"/>
                      <Input   placeholder="商户号" style="width: 150px; margin-left:10px;" v-model="account" class="fl"/>
                       <Input   placeholder="创建时间" style="width: 150px; margin-left:10px;" v-model="account" class="fl"/>
                        <Input   placeholder="系统订单号" style="width: 150px; margin-left:10px;" v-model="account" class="fl"/>
                         <Button type="primary">查询</Button>
               </Row>  
                <Table  highlight-row ref="currentRowTable" :columns="columns3" :data="dataList"  style="margin-top:10px;"></Table>  
                 <Page :total="totals"  :page-size="table_limit" show-total :current='table_current' @on-change="ccount"/>
           </div>    
        <div>

        </div>
    </div>
</template>
<script>
import {
 ccountList,
 withdraws
} from "@/api/index";
export default {
    data(){
        return{
             totals:0,
            table_limit: 10,
           table_current: 1,
            account:'',
            dataList:[],
            generalAccount:{

            },
            workid:'',
            columns3:[
                 {
                    title: '商户名',
                    key: 'accountname'
                },
                 {
                    title: '系统订单号',
                    key: 'orderid'
                },
                 {
                    title: '结算金额',
                    key: 'withdrawamount'
                },
                 {
                    title: '手续费',
                    key: 'withdrawrate'
                },
                 {
                    title: '实际结算',
                    key: 'toamount'
                },
                 {
                    title: '结算银行',
                    key: 'branchname'
                },
                 {
                    title: '开户名',
                    key: 'accountname'
                },
                 {
                    title: '银行账户',
                    key: 'bankcardno'
                },
                 {
                    title: '申请时间',
                    key: 'createTime'
                },
                 {
                    title: '处理时间',
                    key: 'updateTime'
                },
                 {
                    title: '结算备注',
                    key: 'comment'
                },
                 {
                    title: '结算状态',
                    key: 'status',
                       render:(h,params)=>{
                           let status = params.row.status
                           if(status=='0'){
                                status='未处理'
                           }else if(status=='1'){
                                status='处理中'
                           }else if(status=='2'){
                                status='已结算'
                           }else if(status=='3'){
                                status='结算异常'
                           }
                           else{
                               status='取消结算'
                           }
                           return h('span',status)
                       }
                },
                 {
                        title: '操作',
                           render:(h,params)=>{
                           return h('div',[
                               h('Button',{
                                   props: {
                                    size: 'small',
                                    type: 'success'
                                    },
                                style: {
                                    marginRight: '10px'
                                    },
                                    on:{
                                        click: () =>{
                                            console.log(444)
                                                this.workid=params.row.id
                                                this.withdrawsty()
                                             }
                                    }
                               },'确认')
                           ])
                       }
                    },
            ]
        }
    },
    mounted(){
        this.ccount()
    },
    methods:{
        ccount(page){
            let params={"pageVo":{"pageSize":this.table_limit, "pageNumber":page},
                        "withdrawsVo":{
                           
                    
                        },
                        "searchVo":{
                            "startDate":''
                            }
                        }
            ccountList(params).then(res => {
                 console.log(res)
                 this.totals=res.data.pageInfo.total
                 this.dataList=res.data.pageInfo.list
                 this.generalAccount=res.data.generalAccount
                }).catch(err => {
                    this.treeLoading = false;
                });
        },
        withdrawsty(){
            let params={id:this.workid}
            withdraws(params).then(res=>{
               if(res.status==0){
                this.$Message.info('申请通过');
                this.ccount()
               }
            }).catch(err=>{

            })
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
    .balance-box{
        width: 340px;
        height: 90px;
    }
    .balance-box-left{
        height: 90px;
        width: 90px;
         float: left;
        background: #009ABF;
    }
    .balance-box-icon{
        color: #fff;
        margin-top: 25px;
        margin-left: 25px;
    }
    .balance-box-right{
        height: 90px;
        width: 250px;
        background: #00C0EF;
        float: left;
    }
     .balance-box-right p{
         color: #fff;
         font-size: 14px;
         margin-left: 10px;
         line-height: 40px;
     }
     .balance-box-right div{
         color: #fff;
         font-size: 20px;
         height: 50px;
         border-top: 2px solid #fff;
         line-height: 50px;
         padding-left: 10px;
     }
</style>
