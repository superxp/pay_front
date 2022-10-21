<template>
    <div>
          <p class="agency-top-text">支付方式管理</p>
           <div class="agency-box">
                 <Button type="primary" @click="addpay = true">添加支付方式</Button>
                <Table  highlight-row ref="currentRowTable" :columns="columns3" :data="dataList"  @on-current-change="handleRowChange" style="margin-top:20px;"></Table>  
            <Modal
            v-model="addpay"
              title="添加支付方式"
                @on-ok="addok"
                @on-cancel="addcancel"
            >
            <Form :label-width="100" :model='addData'>
                     <FormItem label="支付名称" >
                            <Input v-model="addData.paymentName" placeholder="金额"/>
                    </FormItem>
                     <FormItem label="支付编码" >
                            <Input v-model="addData.paymentCode" placeholder="金额"/>
                    </FormItem>
                     <FormItem label="所属通道" >
                        <Select v-model="addData.status">
                                <Option v-for="item in statusList" :key='item.id' :value="item.id">{{item.name}}</Option>
                        </Select>
                    </FormItem>
                     <FormItem label="运营费率" >
                            <Input v-model="addData.runRate" placeholder="金额"/>
                    </FormItem>
                     <FormItem label="成本费率" >
                            <Input v-model="addData.costRate" placeholder="金额"/>
                    </FormItem>
                     <FormItem label="单笔最小金额" >
                            <Input v-model="addData.minAmount" placeholder="金额"/>
                    </FormItem>
                     <FormItem label="单笔最最多金额" >
                            <Input v-model="addData.maxAmount" placeholder="金额"/>
                    </FormItem>
                     <FormItem label="状态" >
                        <Select v-model="addData.status">
                                <Option v-for="item in statusList" :key='item.id' :value="item.id">{{item.name}}</Option>
                        </Select>
                    </FormItem>
                </Form>
            </Modal>    
            <Modal
             v-model="editor"
                    title="编辑支付方式"
                    @on-ok="editorsok"
                    @on-cancel="editorscancel"
            >
                <Form :label-width="100" :model='editordata'>
                     <FormItem label="支付名称" >
                            <Input v-model="editordata.paymentName" placeholder="金额"/>
                    </FormItem>
                     <FormItem label="支付编码" >
                            <Input v-model="editordata.amount" placeholder="金额"/>
                    </FormItem>
                     <FormItem label="所属通道" >
                            <Input v-model="editordata.amount" placeholder="金额"/>
                    </FormItem>
                     <FormItem label="运营费率" >
                            <Input v-model="editordata.amount" placeholder="金额"/>
                    </FormItem>
                     <FormItem label="成本费率" >
                            <Input v-model="editordata.amount" placeholder="金额"/>
                    </FormItem>
                     <FormItem label="单笔最小金额" >
                            <Input v-model="editordata.amount" placeholder="金额"/>
                    </FormItem>
                     <FormItem label="单笔最最多金额" >
                            <Input v-model="editordata.amount" placeholder="金额"/>
                    </FormItem>
                     <FormItem label="状态" >
                            <Input v-model="editordata.amount" placeholder="金额"/>
                    </FormItem>
                </Form>
            </Modal>    
           </div> 
    </div>
</template>
<script>
import {
payList,
editorspayList,
addpayList,
eremovepayList
} from "@/api/index";
export default {
    data(){
        return{
            id:'',
            addpay:false,
            editor:false,
            editordata:{
                amount:''
            },
            addData:{
                channelId:'',
                 paymentName:'',
                  paymentCode:'',
                   runRate:'',
                    costRate:'',
                     minAmount:'',
                      maxAmount:'',
                    status:''                
            },
              statusList:[
                {
                    id:'0',
                    name:'启用'
                },
                 {
                    id:'1',
                    name:'禁用'
                }
            ],
            dataList:[],
            columns3:[
                  {
                    type: 'index',
                    width: 60,
                    align: 'center'
                },
                {
                    title: '支付名称',
                    key: 'paymentName'
                },
                {
                    title: '支付编码',
                    key: 'paymentCode'
                },
                {
                    title: '所属通道',
                    key: 'runRate'
                },
                {
                    title: '运营费率',
                    key: 'runRate'
                },
                {
                    title: '单笔限额',
                    key: 'minAmount'-'maxAmount'
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
                                                 this.editor=true
                                             }
                                        }
                                 },'编辑'),
                                   h('Button',{
                                      props: {
                                         size: 'small',
                                         type: 'error'
                                        },
                                         style: {
                                            marginRight: '10px'
                                            },
                                        on:{
                                             click: () =>{
                                                this.id= params.row.id
                                                this.delete()
                                             }
                                        }
                                 },'删除'),   
                           ])
                     }
                },
            ]
        }
    },
    mounted(){
        this.paymethode()
    },
    methods:{
        paymethode(){
                let params={
                pageNumber:"1",
                pageSize:"3"
             }
            payList(params).then(res => {
                 console.log(res)
                 this.dataList=res.data.list
                }).catch(err => {
                    this.treeLoading = false;
                });
        },
        addok(){},
        addcancel(){},
        editorsok(){},
        editorscancel(){},
         handleRowChange(currentRow){
             this.editordata=currentRow
         },
        // 编辑
        editors(){
            editorspayList().then(res=>{

            }).catch(err=>{
                this.treeLoading = false;
            });
        },
        //删除
        delete(){
            let params={
                id:this.id
            }
           eremovepayList(params).then(res=>{
               if(res.status==0){
                    this.$Message.success('删除成功');
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
</style>
