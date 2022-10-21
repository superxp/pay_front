<template>
    <div>
        <p class="agency-top-text">代理商户</p>
        <div class="agency-box">
            <Button type="primary" @click="modal1 = true">添加代理</Button>
                <Modal
                    v-model="modal1"
                    title="添加代理"
                    @on-ok="ok"
                    @on-cancel="cancel">
                    <Form  :label-width="80" v-model='fromData'>
                        <FormItem label="用户名" >
                            <Input v-model="fromData.username" placeholder="请输入用户名"/>
                        </FormItem>
                         <FormItem label="密码">
                            <Input v-model="fromData.password" type="password" placeholder="请输入密码"/>
                        </FormItem>
                         <FormItem label="邮箱" >
                            <Input v-model="fromData.email" placeholder="请输入邮箱"/>
                        </FormItem>
                         <FormItem label="电话">
                            <Input v-model="fromData.mobile" placeholder="请输入电话"/>
                        </FormItem>
                         <FormItem label="状态">
                             <Select v-model="fromData.status">
                                <Option v-for="item in statusList" :key='item.id' :value="item.id">{{item.name}}</Option>
                            </Select>
                        </FormItem>
                    </Form>    
                </Modal>
            <Row style="margin-top:10px;">    
                <Input v-model="agencyName" placeholder="商户号" style="width: 150px" class="fl"/>
                 <Input v-model="agencyName" placeholder="用户名" style="width: 150px" class="fl"/>
                 <Select  placeholder="状态" class="fl" v-model="status" style="width:80px">
                    <Option value="0">启用</Option>
                    <Option value="1">禁用</Option>
                    <Option value="2">删除</Option>
                </Select>
                 <Button type="primary" @click="query">查询</Button>
            </Row>
            <Table  highlight-row ref="currentRowTable" :columns="columns3" :data="dataList"  @on-current-change="handleRowChange" style="margin-top:10px;"></Table>
            <Modal
             v-model="modal2"
                    title="充值"
                    @on-ok="twook"
                    @on-cancel="twocancel"
            >
            <Row style="margin-top:10px;">    
                 <Form  :label-width="80" :model='chargedata'>
                    <FormItem label="金额" >
                            <Input v-model="chargedata.amount" placeholder="金额"/>
                    </FormItem>
                    <FormItem label="修改状态">
                             <Select v-model="chargedata.status">
                                <Option value="0">增加</Option>
                                <Option value="1">减少</Option>
                            </Select>
                    </FormItem>
                 </Form>    
            </Row>
            </Modal>
            <Modal
                    v-model="modal3"
                    title="会员编辑"
                    @on-ok="threeok"
                    @on-cancel="threecancel">
                    <Form  :label-width="80" v-model='fromData'>
                        <FormItem label="用户名" >
                            <Input v-model="from.username" placeholder="请输入用户名"/>
                        </FormItem>
                         <FormItem label="密码">
                            <Input v-model="from.password" type="password" placeholder="请输入密码"/>
                        </FormItem>
                         <FormItem label="邮箱" >
                            <Input v-model="from.email" placeholder="请输入邮箱"/>
                        </FormItem>
                         <FormItem label="电话">
                            <Input v-model="from.mobile" placeholder="请输入电话"/>
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
 agencyadd,
 merchantList,
 subtract,
 editList,
 deleteuserList
} from "@/api/index";
export default {
    data(){
        return{
            id:'',
            text:'启用',
            qi:'',
            modal1:false,
            modal2:false,
            modal3:false,
            amountss:'',
            agencyName:'',
            status:'',
            fromData:{
                roleIds:'2',
                username:'', //用户名
                password:'', //密码
                status:'',   //状态
                email:'', //邮箱
                mobile:'' //手机号
            },
                from:{
                id:'',
                username:'', //用户名
                password:'', //密码
                status:'',   //状态
                email:'', //邮箱
                mobile:'' //手机号
            },
            chargedata:{
                amount:'',
                status:'',
                userid:'',
                type:''
            },
            dataList:[
                
            ],
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
             columns3: [
                    {
                        type: 'index',
                        width: 60,
                        align: 'center'
                    },
                    {
                        title: '商户号',
                        key: 'merchant'
                    },
                    {
                        title: '用户名',
                        key: 'username'
                    },
                    {
                        title: '余额',
                        key: 'amount'
                    },
                     {
                        title: '充值余额',
                        key: 'address'
                    },
                      {
                        title: '邮箱',
                        key: 'email'
                    },
                      {
                        title: '电话',
                        key: 'mobile'
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
                    {     width: 380,
                         title: '操作', 
                         render:(h, params)=>{
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
                                                //  this.handleRowChange(currentRow, oldCurrentRow)
                                                this.$router.push('/passageway')
                                             }
                                        }
                                 },'通道设置'),
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
                                               this.modal3=true
                                            this.from.username= params.row.merchant
                                             this.from.password= params.row.password
                                              this.from.status= params.row.status
                                               this.from.email= params.row.email
                                                this.from.mobile= params.row.mobile
                                                 this.from.id= params.row.id
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
                                                 this.id=params.row.id
                                                 this.delete()
                                             }
                                        }
                                 },'删除'),
                                  h('Button',{
                                      props: {
                                         size: 'small',
                                         type: 'warning'
                                        },
                                         style: {
                                            marginRight: '10px'
                                            },
                                        on:{
                                             click: () =>{
                                               this.modal2=true
                                             }
                                        }
                                 },'充值余额'),
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
                                             }
                                        }
                                 },'余额加减')
                             ])
                         }
                    }
                ],
        }
    },
    mounted(){
        this.cc()
    },
     methods: {
            //确认
            ok () {
                let params=this.fromData
                agencyadd(params).then(res => {
                  if(res.status==0){
                    this.$Message.info('新增成功');
                      this.cc()
                  }
                }).catch(err => {
                    this.treeLoading = false;
                });
                this.formdata=''
            },
            //取消
            cancel () {
            },
            threeok(){
                this.edit()
            },
            threecancel () {
            },
            query(){
                this.cc()
            },
            twook(){this.addsubtract()},
            twocancel(){},
            cc(){
                 let params={"pageVo":{"pageNumber":1,"pageSize":10},"userVo":{"roleIds":"2"},"searchVo":{"startDate":"","endDate":""}}
               merchantList(params).then(res => {
                   console.log(res)
                   this.dataList=res.data.list
                   this.qi=res.data.list.status
                }).catch(err => {
                    this.treeLoading = false;
                });
            },
             handleRowChange(currentRow, oldCurrentRow){
                this.chargedata.userid=currentRow.id
                this.chargedata.type=currentRow.type
                localStorage.setItem('uid', currentRow.id)
                localStorage.setItem('parentId', currentRow.parentId)
            },
            addsubtract(){
                let params={"walletVo":{"userId":this.chargedata.userid,"type":this.chargedata.type},"option":this.chargedata.status,"amount":this.chargedata.amount}
            subtract(params).then(res=>{
                if(res.status==0){
                    this.$Message.success('增加成功');
                }
            }).catch(err=>{
                  this.treeLoading = false;
            })
            },
            edit(){
                let params=this.from
                editList(params).then(res=>{
                    if(res.status==0){
                     this.$Message.success('修改成功');
                }
                }).then(res=>{

                })
            },
            delete(){
                 let params=this.id
                deleteuserList(params).then(res=>{
                if(res.status==0){
                    this.$Message.success('修改成功');
                    } 
                }).then(res=>{

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
