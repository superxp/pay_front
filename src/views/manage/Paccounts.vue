<template>
    <div>

          <p class="agency-top-text">拼多多账户管理</p>
           <div class="agency-box">
                <Modal
                    v-model="modal1"
                    title="添加代理"
                    @on-ok="ok"
                    @on-cancel="cancel">
                    <Form  :label-width="80" v-model='fromData'>
                        <FormItem label="用户ID" >
                            <Input v-model="fromData.pdduid" placeholder="请输入用户名"/>
                        </FormItem>
                         <FormItem label="地址ID">
                            <Input v-model="fromData.addressId"  placeholder="请输入密码"/>
                        </FormItem>
                         <FormItem label="token" >
                            <Input v-model="fromData.accesstoken" placeholder="请输入邮箱"/>
                        </FormItem>
                          <FormItem label="手机号" >
                            <Input v-model="fromData.phone" placeholder="请输入手机号"/>
                        </FormItem>
                         <FormItem label="扩展" >
                            <Input v-model="fromData.extar" placeholder="请输入扩展"/>
                        </FormItem>
                         <FormItem label="备注" >
                            <Input v-model="fromData.mark" placeholder="请输入备注"/>
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
                    title="添加代理"
                    @on-ok="ok2"
                    @on-cancel="cancel2">
                    <Form  :label-width="80" v-model='fromlk'>
                        <FormItem label="用户ID" >
                            <Input v-model="fromlk.pdduid" placeholder="请输入用户名"/>
                        </FormItem>
                         <FormItem label="地址ID">
                            <Input v-model="fromlk.addressId"  placeholder="请输入密码"/>
                        </FormItem>
                         <FormItem label="token" >
                            <Input v-model="fromlk.accesstoken" placeholder="请输入token"/>
                        </FormItem>
                          <FormItem label="手机号码" >
                            <Input v-model="fromlk.phone" placeholder="请输入手机号"/>
                        </FormItem>
                         <FormItem label="扩展" >
                            <Input v-model="fromlk.extar" placeholder="请输入扩展"/>
                        </FormItem>
                         <FormItem label="备注" >
                            <Input v-model="fromlk.mark" placeholder="请输入备注"/>
                        </FormItem>
                         <FormItem label="状态">
                             <Select v-model="fromlk.status">
                                <Option v-for="item in statusList" :key='item.id' :value="item.title">{{item.name}}</Option>
                            </Select>
                        </FormItem>
                    </Form>    
                </Modal>
                <Button type="primary" @click="modal1 = true">添加账户</Button>
                 <Table  highlight-row ref="currentRowTable" :columns="columns3" :data="dataList"  style="margin-top:20px;"></Table>
               <Row type="flex" justify="end" style="margin-top:10px;">
                   <Page :total="totals" :page-size="table_limit" show-total :current='table_current' @on-change="ccount"/>
               </Row>
           </div>    
    </div>
</template>
<script>
import {
 pingallList,
 pingupdateList,
 pingdeleteList,
 pinginsertList
} from "@/api/index";
export default {
    data(){
        return{
            totals: 0,
            table_limit: 10,
            table_current: 1,
            id:'',
            modal1:false,
            modal2:false,
            fromData:{
            extar:'',
            mark:'',
            pdduid: "",
            addressId:"",
            status: '',
            accesstoken: "",
            phone:''
            },
               fromlk:{
            id:'',       
            extar:'',
            mark:'',
            pdduid: "",
            addressId:"",
            status: false,
            accesstoken: "",
            phone:''
            },
                 statusList:[
                {
                    title:true,
                    name:'启用'
                },
                 {
                    title:false,
                    name:'禁用'
                },
            ],
            columns3:[
                 {
                    type: 'index',
                    width: 60,
                    align: 'center'
                },
                  {
                        title: '用户ID',
                        key: 'pdduid'
                    },
                      {
                        title: '地址ID',
                        key: 'addressId'
                    },
                     {
                        title: '手机号',
                        key: 'phone'
                    },
                    {
                         title: '扩展',
                        key: 'extar'
                    },
                    {
                         title: '备注',
                        key: 'mark'
                    },
                      {
                        title: 'token',
                        key: 'accesstoken'
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
                                    value: params.row.status===true
                                    },
                                style: {
                                    marginRight: '5px'
                                    },
                                    on:{
                                         'on-change': (value) => {
                                             let p = {
                                                 id : params.row.id,
                                                 status : value
                                             }
                                             pingupdateList(p).then(res => {

                                                 this.contains()
                                             }).catch(err => {
                                                 this.treeLoading = false;
                                             });
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
                               ])
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
                                    type: 'error'
                                    },
                                style: {
                                    marginRight: '10px'
                                    },
                                    on:{
                                        click: () =>{
                                            this.id=params.row.id
                                            this.delete()
                                             }
                                    }
                               },'删除'),
                              h('Button',{
                                      props: {
                                         size: 'small',
                                         type: 'info'
                                        },
                                         style: {
                                            marginRight: '10px'
                                            },
                                        on:{
                                             click: () =>{
                                                 this.modal2=true
                                                  this.fromlk.extar=params.row.extar
                                                   this.fromlk.mark =params.row.mark 
                                                     this.fromlk.pdduid=params.row.pdduid
                                                      this.fromlk.addressId=params.row.addressId
                                                       this.fromlk.status=params.row.status
                                                       this.fromlk.accesstoken=params.row.accesstoken
                                                        this.fromlk.id=params.row.id 
                                                        this.fromlk.phone=params.row.phone
                                             }
                                        }
                                 },'编辑'),
                               
                           ])
                       }
                    },
            ],
            dataList:[]
        }
    },
    mounted(){
        this.contains()
    },
    methods:{
            contains(page){
             let params={
                // id:localStorage.getItem('uid'),
                // parentId:localStorage.getItem('parentId')
             }
           pingallList(params).then(res => {
                 console.log(res)
                 this.dataList=res.data
                 
                }).catch(err => {
                    this.treeLoading = false;
                });
        },
        ok(){
            let params=this.fromData
            pinginsertList(params).then(res => {
                 console.log(res)
                 this.contains()
                 
                }).catch(err => {
                    this.treeLoading = false;
                });
        },
        cancel(){},
        ok2(){
            let params=this.fromlk
                pingupdateList(params).then(res => {
                  console.log(res)
                    this.contains()
                    }).catch(err => {
                        this.treeLoading = false;
                    }); 
        },
         cancel2(){},
         delete(){
                 let params={
                        id: this.id
                        }
                        pingdeleteList(params).then(res => {
                            console.log(res)
                            this.contains()
                                                        
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
