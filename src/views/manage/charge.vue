<template>
    <div>
         <p class="agency-top-text">商户充值</p>
         <div class="agency-box">
                 <Button type="primary" style="margin-bottom:10px;" @click="accharge">账号充值</Button>
              <Row>
                            <Input v-model="chargedata.orderNo" placeholder="充值订单号" style="width: 150px;margin-right:10px;"/>
                            <Input v-model="chargedata.merchant" placeholder="商户号" style="width: 150px; margin-right:10px;"/>
                            <DatePicker type="daterange" v-model="chargedata.data" split-panels placeholder="创建时间" style="width: 200px; margin-right:10px;"></DatePicker>
                             <Select v-model="chargedata.payStatus" placeholder="充值状态" style="width: 150px; margin-right:10px;">
                                <Option value="0">未支付</Option>
                                <Option value="1">已支付</Option>
                                 <Option value="1">异常</Option>
                            </Select>
                         <Button type="primary" @click="searches">查询</Button>
               </Row>
               <Table  highlight-row ref="currentRowTable" :columns="columns3" :data="dataList"  style="margin-top:10px;"></Table>  
         </div>
    </div>
</template>
<script>
import {
chargeList,
souchargeList
} from "@/api/index";
export default {
    data(){
        return{
          account:'',
          dataList:[],
          chargedata:{
              orderNo:'',
              merchant:'',
              payStatus:'',
              data:''
          },
          columns3:[
                {
                      title: '用户名',
                        key: 'username'
                },
               {
                      title: '商户号',
                        key: 'merchant'
                },
                 {
                      title: '充值订单号',
                        key: 'orderno'
                },
                 {
                      title: '充值金额',
                        key: 'rechargeAmount'
                },
                 {
                        title: '充值状态',
                        key: 'payStatus',
                       render:(h,params)=>{
                           let payStatus = params.row.payStatus
                           if(payStatus=='0'){
                                payStatus='未支付'
                           }else if(payStatus=='1'){
                                payStatus='已支付'
                           }else{
                               payStatus='异常'
                           }
                           return h('span',payStatus)
                       }
                },
                 {
                      title: '处理时间',
                        key: 'createTime'
                },
                 {
                      title: '充值备注',
                        key: 'ordermk'
                },
                 {
                      title: '操作',
                        key: 'merchant'
                },

          ]
        }
    },
    mounted(){
        this.charge()
    },
    methods:{
        accharge(){
              this.$router.push('/accharge')
        },
        charge(){
            let params={
                pageNumber:"1",
                pageSize:"2"
            }
           chargeList(params).then(res=>{
               console.log(res)
               this.dataList=res.data.list
            }).catch(err=>{
                  this.treeLoading = false;
            })
        },
        searches(){
            let params={
                "pageVo": {
                    "pageNumber": "1",
                    "pageSize": "3"
                },
                "searchVo": {
                    "startDate": "2019-03-17 00:00:00",
                    "endDate": "2019-04-17 23:59:59"
                },
                "rechargeVo": {
                    "merchant": this.chargedata.merchant,
                    "orderNo": this.chargedata.orderNo,
                    "payStatus":this.chargedata.payStatus
                }
            }
            souchargeList(params).then(res=>{
                 console.log(res)
                 this.dataList=res.data.list
            }).catch(err=>{
                this.treeLoading = false;
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
</style>
