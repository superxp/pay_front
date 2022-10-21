<template>
    <div>
        <p class="agency-top-text">通道管理</p>
           <div class="agency-box">
                <Button type="primary" @click="fan">返回上一页</Button>
                 <Table  highlight-row ref="currentRowTable" :columns="columns3" :data="dataList" @on-current-change="handleRowChange" style="margin-top:20px;"></Table>
                 
                 <Modal
                  v-model="modal1"
                    title="用户通道编辑"
                    @on-ok="ok"
                    @on-cancel="cancel"
                 >
               <Form  :label-width="80" :model='chargedata'>
                    <FormItem label="支付方式" >
                            <Input v-model="chargedata.paymentName" disabled />
                    </FormItem>
                    <FormItem label="费率" >
                            <Input v-model="chargedata.rate"/>
                    </FormItem>
                    <FormItem label="修改状态">
                             <Select v-model="chargedata.status">
                                <Option value="0">启用</Option>
                                <Option value="1">禁用</Option>
                            </Select>
                    </FormItem>
                 </Form>    
                 </Modal>    
           </div>
    </div>
</template>
<script>
import {
 passagewayList,
 editorswayList
} from "@/api/index";
export default {
    data(){
        return{
            modal1:false,
            chargedata:{
                amount:'',
                status:'',
                rate:''
            },
            dataList:[],
            columns3:[
                 {
                    type: 'index',
                    width: 60,
                    align: 'center'
                },
                 {
                        title: '支付方式',
                        key: 'paymentName'
                    },
                     {
                        title: '费率',
                        key: 'rate'
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
                                    value: params.row.status===0
                                    },
                                style: {
                                    marginRight: '5px'
                                    },
                                    on:{
                                         'on-change': (value) => {
                                            
                                         }
                                    }
                               })
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
                                                this.modal1=true
                                             }
                                    }
                               },'编辑')
                           ])
                       }
                    },
            ]
        }
    },
    mounted(){
        this.btained()
    },
    methods:{
        ok(){
            this.editors()
        },
        cancel(){},
        fan(){
            this.$router.go(-1);
        },
        btained(){
             let params={
                id:localStorage.getItem('uid'),
                parentId:localStorage.getItem('parentId')
             }
           passagewayList(params).then(res => {
                 console.log(res)
                 this.dataList=res.data
                 
                }).catch(err => {
                    this.treeLoading = false;
                });
        },
          handleRowChange(currentRow, oldCurrentRow){
               this.chargedata=currentRow
            },
        editors(){
            let params= this.chargedata
          editorswayList(params).then(res => {
                if(res.status==0){
                     this.$Message.success('编辑成功');
                }
                }).catch(err => {
                    this.treeLoading = false;
                });
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