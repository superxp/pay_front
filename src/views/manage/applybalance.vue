<template>
    <div>
        <Row>
            <Col span="6">
                <Card>
                    <p slot="title">结算申请</p>
                    <div class="Card-cpde">
                         <p >id：{{amountid}}</p>
                    </div>
                      <div class="Card-cpde">
                         当前余额:<Input v-model="amount" disabled style="width: 300px"/>
                    </div>
                      <div class="Card-cpde">
                        <p style="color:red;">手续费：0元</p>
                    </div>
                    <!-- <div class="Card-cpde">
                         手续费:<Input v-model="withdrawamount" disabled style="width: 300px"/>
                    </div> -->
                    <div class="Card-cpde">
                         提现金额:<Input v-model="withdrawamount" style="width: 300px"/>
                    </div>
                    <div class="Card-cpde">
                            银行卡 <Button type="primary" @click="bank" style="margin-left:10px;">选择银行卡</Button>
                                  <Button type="primary" @click="modal2 = true" style="margin-left:10px;">添加银行卡</Button>  
                    </div>
                      <div class="Card-cpde">
                         持卡姓名:<Input v-model="usernn" disabled  style="width: 300px"/>
                    </div>
                      <div class="Card-cpde">
                         支行名称:<Input v-model="branchName" disabled  style="width: 300px"/>
                    </div>
                      <div class="Card-cpde">
                         银行卡号:<Input v-model="bankCardNo" disabled  style="width: 300px"/>
                    </div>
                     <div class="Card-cpde" style="text-align:center;">
                        <Button type="primary" @click='recharges'>申请结算</Button>
                     </div>
                </Card>
            </Col>
            <Modal
                    v-model="modal1"
                    title="选择银行卡"
                    width=780px;
                    @on-ok="ok1"
                    @on-cancel="cancel1"
            >
                <Table  highlight-row ref="currentRowTable" :columns="columns" :data="dataList2"   @on-current-change="handleRowChange" style="margin-top:10px;"></Table>
            </Modal>    
             <Modal
                    v-model="modal2"
                    title="添加银行卡"
                    @on-ok="ok2"
                    @on-cancel="cancel2">
                          <Form  :label-width="80" v-model='fromData'>
                        <FormItem label="银行名称">
                             <Select v-model="fromData.bankId">
                                <Option v-for="item in bankList" :key='item.id' :value="item.id">{{item.bankname}}</Option>
                        </Select>
                        </FormItem>
                        <FormItem label="持卡人姓名" >
                            <Input v-model="fromData.accountName" placeholder="请输入用户名"/>
                        </FormItem>
                         <FormItem label="支行名称">
                            <Input v-model="fromData.branchName"  placeholder="请输入支行名称"/>
                        </FormItem>
                         <FormItem label="银行卡号" >
                            <Input v-model="fromData.bankCardNo" placeholder="请输入银行卡号"/>
                        </FormItem>
                    </Form>
             </Modal>        
            <Col span="16" style="margin-left:60px;">
                <Card>
                    <p slot="title">申请记录</p> 
                    <!-- <Input v-model="orderNo" placeholder="订单号" style="width: 200px"/> -->
                    <!-- <Button type="primary" style="margin-left:10px;">查询</Button> -->
                     <Table  highlight-row ref="currentRowTable" :columns="columns3" :data="dataList3"  style="margin-top:10px;"></Table>
                      <Page :total="totals"  :page-size="table_limit" show-total :current='table_current' @on-change="accountList"/>
                </Card>     
            </Col>
        </Row>    
    </div>
</template>
<script>
import {
bankcard,
usersLiat,
bankcardadd,
byUser,
account,
accountall,
userget
} from "@/api/index";
export default {
    inject:['reload'],
    data(){
        return{
              totals:0,
            table_limit: 10,
           table_current: 1,
            amountid:'',
            amount:'',
            agentId:'',
            banknamess:'',
            withdrawamount:'',
            usernn:'',
            branchName:'',
            bankCardNo:'',
            modal1:false,
            modal2:false,
         orderNo:'',
         userid:'',
         username:'',
         bankList:[],
         actualAmount:'',
         fromData:{
             userId:'',
             bankId:'',
             bankCardNo:'',
             accountName:'',
             branchName:''
         },
         columns:[
            {
                type: 'index',
                width: 60,
                align: 'center'
            },  
             {
                title: '银行名称',
                key: 'bankName'
            }, 
             {
                title: '支行名称',
                key: 'branchName'
            }, 
             {
                title: '银行卡号',
                key: 'bankCardNo'
            }, 
             {
                title: '持卡人',
                key: 'accountName'
            }, 
             {
                title: '创建时间',
                key: 'createTime'
            }, 

         ],
         dataList2:[],
         dataList:[],
         dataList3:[],
         columns3:[
             {
                title: '银行名称',
                key: 'bankname'
            }, 
             {
                title: '银行卡号',
                key: 'bankcardno'
            },
             {
                title: '提现额度',
                key: 'withdrawamount'
            },
              {
                title: '提现手续费',
                key: 'withdrawrate'
            },
              {
                title: '到账金额',
                key: 'toamount'
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
                    title: '状态',
                    key: 'status',
                       render:(h,params)=>{
                           let status = params.row.status
                           if(status=='0'){
                                status='申请中'
                           }else if(status=='1'){
                                status='已支付'
                           }else if(status=='2'){
                                status='已结算'
                           } else if(status=='3'){
                                status='异常'
                           }
                           return h('span',status)
                       }
                }
         ]   
        }
    },
    mounted(){
        this.acquire()
       this.bankcardlist()
       this.user()
       this.fromData.userId=this.userid
       this.accountList()
    }, 
    methods:{
      recharges(){
          if(this.withdrawamount>this.amount-0){
              this.$Message.error('提现金额不能大于余额');
          }else{
            let params={
            bankname: this.banknamess,
            withdrawamount: this.withdrawamount,
            branchname: this.branchName,
            bankcardno: this.bankCardNo,
                accountname: this.usernn
                }
                account(params).then(res=>{
                    if(res.status==0){
                            this.$Message.success('申请成功');
                            this.accountList()
                            this.reload()
                        }
                }).catch(res=>{
                    
                })
          }
      },
      bank(){
           this.modal1=true

           byUser().then(res=>{
               console.log(res)
               this.dataList2=res.data
           }).catch(res=>{

           })
      },
      acquire(){
          userget().then(res=>{
              this.amount=res.data.amount
              this.amountid=res.data.id
          }).catch(res=>{

          })
      },
      handleRowChange(currentRow){
          this.modal1=false
          this.branchName=currentRow.branchName
          this.bankCardNo=currentRow.bankCardNo
          this.usernn=currentRow.accountName
          this.banknamess=currentRow.bankName
          this.agentId=currentRow.id
      },
      ok1(){

      },
      cancel1(){},
      ok2(){
          console.log(this.fromData)
          let params=this.fromData
          bankcardadd(params).then(res=>{
              console.log(res)
          }).catch(res=>{

          })
      },
      cancel2(){},
      bankcardlist(){
             bankcard().then(res => {
                 this.bankList=res.data
                 console.log(this.bankList)
                }).catch(err => {
                    this.treeLoading = false;
                });
      },
      user(){
          usersLiat().then(res=>{
              this.fromData.userId=res.data.id
              this.userid=res.data.id
 }).catch(err=>{
               this.treeLoading = false;
          })
      },
    //   结算省请接口
    accountList(page){
        let params={"pageVo":{"pageSize":this.table_limit, "pageNumber":page},"withdrawsVo":{"roleIds":"2"},"searchVo":{"startDate":"","endDate":""}}
        accountall(params).then(res=>{
            console.log(res)
            this.totals=res.data.pageInfo.total
            this.dataList3=res.data.pageInfo.list
        }).catch(err=>{

        })
    }
    }
}
</script>

<style lang="less" scoped>
.Card-cpde{
    margin-top: 20px;
}
</style>
