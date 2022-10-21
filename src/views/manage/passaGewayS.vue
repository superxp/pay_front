<template>
    <div>
        <p class="agency-top-text">通道管理</p>
            <div class="agency-box">
                 <Button type="primary" @click="modal1 = true">添加通道</Button>
                  <Button type="primary" style="margin-left:10px;" @click="underground">添加通道池</Button>
                 <Table  highlight-row ref="currentRowTable" :columns="columns3" :data="dataList" @on-current-change="handleRowChange" style="margin-top:10px;"></Table>
                 <Modal
                  v-model="modal1"
                    title="添加通道"
                    @on-ok="ok"
                    @on-cancel="cancel"
                 >
                  <Form  :label-width="80" v-model='fromData'>
                       <FormItem label="通道名称" >
                            <Input v-model="fromData.channelName" placeholder="请输入用户名"/>
                        </FormItem>
                         <FormItem label="通道编码" >
                            <Input v-model="fromData.channelCode" placeholder="只能是字母且唯一"/>
                        </FormItem>
                        <FormItem label="上游商户号" >
                            <Input v-model="fromData.merchant" placeholder="上游商户号"/>
                        </FormItem>
                        <FormItem label="上游秘钥" >
                            <Input v-model="fromData.signkey" placeholder="上游秘钥"/>
                        </FormItem>
                        <FormItem label="通道限额" >
                            <Input v-model="fromData.channelQuota" placeholder="0不限额"/>
                        </FormItem>
                        <FormItem label="扩展" >
                           <Input v-model="fromData.signkey" type="textarea" :autosize="{minRows: 2,maxRows: 5}" placeholder="扩展"/>
                        </FormItem>
                        <FormItem label="状态">
                             <Select v-model="fromData.status">
                                <Option v-for="item in statusList" :key='item.id' :value="item.id">{{item.name}}</Option>
                            </Select>
                        </FormItem>
                  </Form>    
                 </Modal>    
            </div>
    </div>
</template>
<script>
import {
 passagewayfenList,
 addpassagewayList,
 delectpassagewayList
} from "@/api/index";
export default {
    data(){
        return{
            id:'',
            modal1:false,
            fromData:{
                channelName:'',
                signkey:'',
                channelCode:'',
                channelQuota:'',
                status:'',
                merchant:''
            },
                statusList:[
                {
                    id:'0',
                    name:'启用'
                },
                 {
                    id:'1',
                    name:'禁用'
                },
                 {
                    id:'2',
                    name:'删除'
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
                        title: '通道名称',
                        key: 'channelname'
                    },
                 {
                        title: '通道编码',
                        key: 'channelcode'
                    },
                     {
                        title: '费率',
                        key: 'channelquota'
                    },
                     {
                    title: '接口状态',
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
                                               this.modal1=true
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
                               },'删除')
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
        btained(){
                 let params={"pageNumber":"1","pageSize":"3"}
           passagewayfenList(params).then(res => {
                 console.log(res)
                  this.dataList=res.data.list
                 
                }).catch(err => {
                    this.treeLoading = false;
                });
        },
          handleRowChange(currentRow, oldCurrentRow){
               this.chargedata=currentRow
            },
        editors(){
            let params= this.fromData
          addpassagewayList(params).then(res => {
                if(res.status==0){
                     this.$Message.success('添加成功');
                }
                }).catch(err => {
                    this.treeLoading = false;
                });
        },
        handleRowChange(currentRow){
            this.fromData=currentRow
            this.id=currentRow.id
        },
        delete(){
             let params=this.id
             delectpassagewayList(params).then(res => {
                if(res.status==0){
                    this.$Message.success('删除成功');
                    }
                }).catch(err => {
                    this.treeLoading = false;
            });
        },
        underground(){
            this.$router.push('/underground')
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