<template>
    <div>
        <Row>
            <Col span="6">
                <Card>
                    <p slot="title">充值</p>
                    <p>预存手续费余额：103.73</p>
                    <div class="Card-cpde">
                         充值金额:<Input v-model="actualAmount" style="width: 100px"/>
                    </div>
                     <div class="Card-cpde">
                        <Button type="primary" @click='recharges'>立即支付</Button>
                     </div>
                </Card>
            </Col>
            <Col span="16" style="margin-left:60px;">
                <Card>
                    <p slot="title">充值记录</p> 
                    <Input v-model="orderNo" placeholder="订单号" style="width: 200px"/>
                    <Button type="primary" style="margin-left:10px;" @click="query">查询</Button>
                     <Table  highlight-row ref="currentRowTable" :columns="columns3" :data="dataList"  style="margin-top:10px;"></Table>
                </Card>     
            </Col>
        </Row>    
    </div>
</template>
<script>
import {
rechargesList,
chargeList
} from "@/api/index";
export default {
    data(){
        return{
         orderNo:'',
         userid:'',
         username:'',
         actualAmount:'',
         dataList:[],
         columns3:[
             {
                title: '订单号',
                key: 'orderno'
            }, 
             {
                title: '实付金额',
                key: 'actualAmount'
            },
             {
                title: '支付时间',
                key: 'createTime'
            },
             {
                    title: '结算状态',
                    key: 'payStatus',
                       render:(h,params)=>{
                           let payStatus = params.row.payStatus
                           if(payStatus=='0'){
                                payStatus='未支付'
                           }else if(payStatus=='1'){
                                payStatus='已支付'
                           }else if(payStatus=='3'){
                                payStatus='异常'
                           }
                           return h('span',payStatus)
                       }
                }
         ]   
        }
    },
    mounted(){
        this.userid=localStorage.getItem('userid')
         this.username=localStorage.getItem('username')
         this.searchrerecharges()
    }, 
    methods:{
        recharges(){
            let params={
                username:this.username,
                userId:this.userid,
                actualAmount:this.actualAmount
            }
            rechargesList(params).then(res=>{
                   this.searchrerecharges()
            }).catch(err=>{
                  this.treeLoading = false;
            })
        },
        searchrerecharges(){
            let params={
                "pageVo": {
                    "pageNumber": "1",
                    "pageSize": "3"
                },
                "searchVo": {
                    "startDate": "",
                    "endDate": ""
                },
                "rechargeVo": {
                  "orderNo": this.orderNo
                }
            }
            chargeList(params).then(res=>{
               console.log(res)
               this.dataList=res.data.list
            }).catch(err=>{
                  this.treeLoading = false;
            })
        },
        query(){
             this.searchrerecharges()
        }
    }
}
</script>

<style lang="less" scoped>
.Card-cpde{
    text-align: center;
    margin-top: 20px;
}
</style>
