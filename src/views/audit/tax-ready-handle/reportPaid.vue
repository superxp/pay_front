<template>
  <div class="box">
    <div class="dataDetils">
      <table width="100%" height="auto" class="taxesTables">
         <tbody>
           <tr>
           <td colspan="12" width="1124" style="text-align:center;font-size:22px;font-weight:bold;">税金申请表单</td>
         </tr>
         <tr>
           <td width="82">公司名称</td>
           <td colspan="5" width="504">{{dataDetils.companyName}}</td>
           <td width="100">后台识别号码</td>
           <td colspan="5" width="502">{{dataDetils.tin}}</td>
         </tr>
         <tr>
           <td width="82">国家</td>
           <td colspan="5" width="504">{{dataDetils.countryName}}</td>
           <td width="72">币种</td>
           <td colspan="5" width="502">{{dictCurrencysMap && dictCurrencysMap.get(dataDetils.currency)}}</td>
         </tr>
         <tr>
           <td width="82">申请人</td>
           <td colspan="5" width="504">{{dataDetils.applicantName}}</td>
           <td width="72">备注</td>
           <td colspan="5" width="502">{{dataDetils.remarks}}</td>
         </tr>
         <tr>
           <td width="82" >财务报表</td>
           <td colspan="5" style="border-right-color:#fff">{{dataDetils.oriName}}</td>
           <td style="border-right-color:#fff" class="myspan" @click.stop="priviewFile(dataDetils.financialReportPath)">预览</td>
           <td class="myspan" @click.stop="uploadFile(dataDetils.financialReportPath)">下载</td>
         </tr>
       </tbody>
       </table>
    </div>
    <div class="details">
      <Table  border :columns="detailsColumns" :data="details" ref="table" sortable="custom" id="myTable"></Table>
    </div>
    <div class="auditLogVoList">
      <Table  border :columns="auditLogVoListColumns" :data="auditLogVoList"  sortable="custom" ></Table>
    </div>
    <Modal
        :closable="false"
        class-name="preview-modal-inline"
        width=600       
        v-model="showUploadModal">
      <div>
          <Form label-position="left" :label-width="100" :modal="fileUploadForm" :rules="fileUploadFormRules" ref="showUploadRefs">
          <FormItem label="预申报表" prop="preTaxReturnsPath">
            <Input type="text" disabled v-model="fileUploadForm.preTaxReturnsPathFileName" style="width:150px;float:left"/>
            <Button   @click.stop="priviewFile(fileUploadForm.preTaxReturnsPath)">预览</Button>
            <Button   @click.stop="uploadFile(fileUploadForm.preTaxReturnsPath)">下载</Button>
          </FormItem>
          <FormItem label="申报表" prop="taxReturnsPath">
            <Input type="text" disabled v-model="fileUploadForm.taxReturnsPathFileName" style="width:150px;float:left"/>
            <Upload action="/api/file/upload"
            :headers="{accessToken: accessToken}" name="file"
            :accept="fileTypeString"
            :data="{materialTypeDict: 'TAX_REPORT',taxDict:colSelectCurrencyCode,currency:selectCurrencyCode}" :show-upload-list="false"
            :on-success="uploadSuc" style="float:left">
              <Button icon="ios-cloud-upload-outline">{{`${fileUploadForm.taxReturnsPathFileName?"已上传":"上传文件"}`}}</Button>
            </Upload>
            <Button  @click.stop="priviewFile(fileUploadForm.taxReturnsPath)">预览</Button>
            <Button  @click.stop="delFiles(fileUploadForm.taxReturnsPath,'1')">删除</Button>
            <Button  @click.stop="uploadFile(fileUploadForm.taxReturnsPath)">下载</Button>
          </FormItem>
          <FormItem label="完税申报表" prop="paymentCertificatePath">
            <Input type="text" disabled v-model="fileUploadForm.paymentCertificatePathFileName" style="width:150px;float:left"/>
            <Upload action="/api/file/upload"
            :headers="{accessToken: accessToken}" name="file"
            :accept="fileTypeString"
            :data="{materialTypeDict: 'DONE_TAX_REPORT',taxDict:colSelectCurrencyCode,currency:selectCurrencyCode}" :show-upload-list="false"
            :on-success="uploadSuc" style="float:left">
              <Button icon="ios-cloud-upload-outline">{{`${fileUploadForm.paymentCertificatePathFileName?"已上传":"上传文件"}`}}</Button>
            </Upload>
            <Button  @click.stop="priviewFile(fileUploadForm.paymentCertificatePath)">预览</Button>
            <Button  @click.stop="delFiles(fileUploadForm.paymentCertificatePath,'2')">删除</Button>
            <Button  @click.stop="uploadFile(fileUploadForm.paymentCertificatePath)">下载</Button>
          </FormItem>
          <FormItem label="其它" prop="otherUploadId">
            <Input type="text" disabled  v-model="fileUploadForm.otherUploadFileName"  style="width:150px;float:left"/>
            <Upload action="/api/file/upload"
            :headers="{accessToken: accessToken}" name="file"
            :accept="fileTypeString"
            :data="{materialTypeDict: 'OTHER',taxDict:colSelectCurrencyCode,currency:selectCurrencyCode}" :show-upload-list="false"
            :on-success="uploadSuc" style="float:left">
              <Button icon="ios-cloud-upload-outline">{{`${fileUploadForm.otherUploadFileName?"已上传":"上传文件"}`}}</Button>
            </Upload>
            <Button @click.stop="priviewFile(fileUploadForm.otherUploadId)">预览</Button>
            <Button  @click.stop="delFiles(fileUploadForm.otherUploadId,'3')">删除</Button>
            <Button  @click.stop="uploadFile(fileUploadForm.otherUploadId)">下载</Button>
          </FormItem>
      </Form>
      </div>
      <footer class="vertical-center" slot="footer">
          <Button style="width: 100px;"  @click="fileuploadFormCancel">取消</Button>
          <Button type="primary" style="width: 100px;margin-left:20px" @click="tempSubmitOk" >确定</Button>
      </footer>
      <span style="color:red">只能上传 {{fileTypeString}} 文件</span>
    </Modal>
    <div style="margin-top:30px">
        <Button type="primary" @click="handleSubmit">提交</Button>
    </div>
    <Modal
      v-model="taxModal"
      title="注意!"
      @on-ok="taxModalOk"
      @on-cancel="taxModalCancel">
      <p>应缴税额和实缴税额,应缴滞纳金和实缴滞纳金。金额不相等,是否继续提交?</p>
    </Modal>
  </div>
</template>

<script>
import fileLoadPath from '@/api/fileload';
import { taxReadyHandle,
  getFileType,getReviewer,dbrwAudit,lookLiuchengtu,getAllCompany,getUserListData,getDictListDataByType,delFile } from '@/api/index.js'
import { getStore } from '@/libs/storage';
import Cookies from "js-cookie";
import { dictType } from '@/libs/constance.js'
export default {
  data() {
    return {
      taxModal:false,
      dataDetils:{},
      details:[],
      auditLogVoList:[],
      currentLinkType:"ABCDAS",
      showUploadModal:false,
      fileTypeString:getStore("fileTypeString"),
      accessToken: getStore('accessToken'),
      colSelectCurrencyCode:"", // 行选择的税种code
      selectCurrencyCode:"", // 选择的币种
      fileUploadForm:{
        preTaxReturns:"",
        preTaxReturnsPath:"",
        taxReturns:"",
        taxReturnsPath:'',
        paymentCertificate:"",
        paymentCertificatePath:"",
        otherUpload:"",
        otherUploadId:"",
        index:""
      },
      auditLogVoListColumns:[
        {
          title:"任务",
          key:"taskName"
        },
        {
          title:"角色名称",
          key:"roleName"
        },
        {
          title:"姓名",
          key:"name"
        },
        {
          title:"审批结果",
          key:"auditResult"
        },
        {
          title:"意见",
          key:"advice"
        },
        {
          title:"审批时间",
          key:"auditDate",
          render:(h,parms)=>{
            return h('div',new Date(parms.row.auditDate).format())
          }
        }
      ],
      detailsColumns:[
          {
            title: '所属期间',
            key: "taxPeriod",
            width: 160,
            align: 'center',
            render:(h,params) => {
              if(params.row.taxPeriod) {
                let tempDate = params.row.taxPeriod.split("-")
                let submitDate = tempDate[0]+"-"+tempDate[1]
                return h('div', submitDate)
              }
            }
            // render: this.renderSelect
            // render: this.renderDatePicker
          },
          {
            title: '税种',
            key: "taxDict",
            align: 'center',
            width: 160,
            render: this.renderTaxDict
          },
          {
            title: '应缴税额',
            //key: "payableTax",
            align: 'center',
            width: 120,
            // render: this.renderPayableTax
            render:(h,params)=>{ 
              return h("div",params.row.payableTax?`${parseFloat(params.row.payableTax)}`.replace(/([0-9]+\.[0-9]{2})[0-9]*/,"$1"):0)
            }
          },
          {
            title: '应缴滞纳金',
            //key: "lateFeePayable",
            align: 'center',
            width: 120,
            render:(h,params)=>{
              return h("div",params.row.lateFeePayable?`${parseFloat(params.row.lateFeePayable)}`.replace(/([0-9]+\.[0-9]{2})[0-9]*/,"$1"):0)
            }
            //render: this.renderInput
          },
          {
            title: '申请缴纳税款',
            key: "applTaxPayment",
            align: 'center',
            width: 120,
            render: (h, params) => {
              return h('div', {
                domProps: {
                  innerText: `${parseFloat(params.row.payableTax) + parseFloat(params.row.lateFeePayable)}`.replace(/([0-9]+\.[0-9]{2})[0-9]*/,"$1")
                }
              })
              return h('div', `${parseFloat(this.data[params.index].payableTax) + parseFloat(this.data[params.index].lateFeePayable)}`.replace(/([0-9]+\.[0-9]{2})[0-9]*/,"$1"))
            }
          },
          {
            title: '缴款截止日期',
            key: "deadline",
            width: 120,
            align: 'center',
            render: (h,params)=>{
              if(params.row.deadline) {
                return h('div',new Date(params.row.deadline).format())
              }
              return h('div')
            }
          },
          {
            title: '实缴税额',
            key: "taxPaid",
            align: 'center',
            width: 100,
            render: this.renderInput
            // render: (h, params) => {
            //   return h('div', params.row.taxPaid)
            // }
          },
          {
            title: '实缴滞纳金',
            key: "overduePayment",
            align: 'center',
            width: 100,
            render: this.renderInput
            // render: (h, params) => {
            //   return h('div', params.row.overduePayment)
            // }
          },
          {
            title: '实际缴纳税款',
            align: 'center',
            width: 120,
            render: (h, params) => {
              return h('div', {
                domProps: {
                  innerText: `${parseFloat(params.row.taxPaid?params.row.taxPaid:0) + parseFloat(params.row.overduePayment?params.row.overduePayment:0)}`.replace(/([0-9]+\.[0-9]{2})[0-9]*/,"$1")
                }
              })
              return h('div', `${parseFloat(this.data[params.index].taxPaid?this.data[params.index].taxPaid:0) + parseFloat(this.data[params.index].overduePayment?this.data[params.index].overduePayment:0)}`.replace(/([0-9]+\.[0-9]{2})[0-9]*/,"$1"))
            }
          },
          {
            title: '实际缴纳日期',
            key: "paymentTime",
            align: 'center',
            width: 140,
            // render: (h,params)=>{
            //   if(params.row.paymentTime) {
            //     return h('div',new Date(params.row.paymentTime).format())
            //   }
            //   return h('div')
            // }
            render: this.renderDatePicker
          },
          {
            title:'操作',
            width:100,
            render: (h, params) => {
              return h('Button', {
                props: {
                  size: "small"
                },
                on: {
                  click:()=>{
                    console.log("adasdadasd",  this.dataDetils)
                    console.log('paramsparams',params)
                    this.colSelectCurrencyCode=this.dataDetils.details[params.index].taxDict
                    this.fileUploadForm = {
                      uploadFileIndex:params.index,
                      preTaxReturns:this.dataDetils.details[params.index].preTaxReturns,
                      preTaxReturnsPath:this.dataDetils.details[params.index].preTaxReturnsPath,
                      preTaxReturnsPathFileName:this.dataDetils.details[params.index].preTaxReturnsPathFileName,
                      taxReturns:this.dataDetils.details[params.index].taxReturns,
                      taxReturnsPath:this.dataDetils.details[params.index].taxReturnsPath,
                      taxReturnsPathFileName:this.dataDetils.details[params.index].taxReturnsPathFileName,
                      paymentCertificate:this.dataDetils.details[params.index].paymentCertificate,
                      paymentCertificatePath:this.dataDetils.details[params.index].paymentCertificatePath,
                      paymentCertificatePathFileName:this.dataDetils.details[params.index].paymentCertificatePathFileName,
                      otherUpload:this.dataDetils.details[params.index].otherUpload,
                      otherUploadId:this.dataDetils.details[params.index].otherUploadId,
                      otherUploadFileName:this.dataDetils.details[params.index].otherUploadFileName,
                    }
                    this.showUploadModal = true
                  }
                }
              }, '操作补全')
            }
          },
          {
            title: '备注',
            key: "remarks",
            align: 'center',
            width: 160,
            // render: this.renderInput
          },
      ],
      dictCurrencys:[],
      dictCurrencysMap:"",
      dictTaxCategorys:[],
      dictTaxCategorysMap:'',
      fileUploadFormRules:{
        preTaxReturnsPath:[{required:true,message:"请上传",trigger: 'blur'}],
        taxReturnsPath:[{required:true,message:"请上传",trigger: 'blur'}],
        paymentCertificatePath:[{required:true,message:"请上传",trigger: 'blur'}],
      },
    }
  },
  methods:{

    delFiles(filepath,type) {
      const that= this
      var indexs = this.fileUploadForm.uploadFileIndex;
      // 申报表
      if(type=="1") {
        if(filepath){
          delFile(filepath).then((res)=>{
            if(res.status==0) {
              that.$Message.success(res.data)
              that.fileUploadForm.taxReturns = ""
              that.fileUploadForm.taxReturnsPath =""
              that.fileUploadForm.taxReturnsPathFileName =""
              that.dataDetils.details[indexs].taxReturns = ""
              that.dataDetils.details[indexs].taxReturnsPath = ""
              that.dataDetils.details[indexs].taxReturnsPathFileName = ""
              that.fileUploadForm.uploadColomunIndex=""
              that.$forceUpdate()
            }else{
              that.$Message.error(res.errMsg)
            }
          })
        }else if(this.fileUploadForm.taxReturnsPath){
          if(res.status==0) {
            that.$Message.success(res.data)
            that.fileUploadForm.taxReturns = ""
            that.fileUploadForm.taxReturnsPath =""
            that.fileUploadForm.taxReturnsPathFileName =""
            that.dataDetils.details[indexs].taxReturns = ""
            that.dataDetils.details[indexs].taxReturnsPath = ""
            that.dataDetils.details[indexs].taxReturnsPathFileName = ""
            that.$forceUpdate()
          }else{
            that.$Message.error(res.errMsg)
          }
        }else{
          this.$Message.error("没有文件")
        }
      }
      // 完税申报表
      if(type=="2") {
        if(filepath){
          delFile(filepath).then((res)=>{
            if(res.status==0) {
              that.$Message.success(res.data)
              that.fileUploadForm.paymentCertificate = ""
              that.fileUploadForm.paymentCertificatePath =""
              that.fileUploadForm.paymentCertificatePathFileName =""
              that.dataDetils.details[indexs].paymentCertificate = ""
              that.dataDetils.details[indexs].paymentCertificatePath = ""
              that.dataDetils.details[indexs].paymentCertificatePathFileName = ""
              that.fileUploadForm.uploadColomunIndex=""
              that.$forceUpdate()
            }else{
              that.$Message.error(res.errMsg)
            }
          })
        }else if(this.fileUploadForm.paymentCertificatePath){
          if(res.status==0) {
            that.$Message.success(res.data)
            that.fileUploadForm.paymentCertificate = ""
            that.fileUploadForm.paymentCertificatePath =""
            that.fileUploadForm.paymentCertificatePathFileName =""
            that.dataDetils.details[indexs].paymentCertificate = ""
            that.dataDetils.details[indexs].paymentCertificatePath = ""
            that.dataDetils.details[indexs].paymentCertificatePathFileName = ""
            that.$forceUpdate()
          }else{
            that.$Message.error(res.errMsg)
          }
        }else{
          this.$Message.error("没有文件")
        }
      }


      // 其他
      if(type=="3") {
        if(filepath){
          delFile(filepath).then((res)=>{
            if(res.status==0) {
              that.$Message.success(res.data)
              this.fileUploadForm.otherUpload = ""
              this.fileUploadForm.otherUploadId =""
              this.fileUploadForm.otherUploadFileName=""
              this.dataDetils.details[indexs].otherUpload = ""
              this.dataDetils.details[indexs].otherUploadId = ""
              this.dataDetils.details[indexs].otherUploadFileName = ""
              that.fileUploadForm.uploadColomunIndex=""
              that.$forceUpdate()
            }else{
              that.$Message.error(res.errMsg)
            }
          })
        }else if(this.fileUploadForm.otherUploadId){
          if(res.status==0) {
            that.$Message.success(res.data)
            this.fileUploadForm.otherUpload = ""
            this.fileUploadForm.otherUploadId =""
            this.fileUploadForm.otherUploadFileName=""
            this.fileUploadForm.otherUpload = ""
            this.fileUploadForm.otherUploadId =""
            this.fileUploadForm.otherUploadFileName=""
            this.dataDetils.details[indexs].otherUpload = ""
            this.dataDetils.details[indexs].otherUploadId = ""
            this.dataDetils.details[indexs].otherUploadFileName = ""
            that.$forceUpdate()
          }else{
            that.$Message.error(res.errMsg)
          }
        }else{
          this.$Message.error("没有文件")
        }
      }


    },

    // 取消
    fileuploadFormCancel() {
      this.showUploadModal = false
    },
    tempSubmitOk(){
      let indexs = this.fileUploadForm.uploadFileIndex;
      this.dataDetils.details[indexs].preTaxReturns = this.fileUploadForm.preTaxReturns
      this.dataDetils.details[indexs].preTaxReturnsPath = this.fileUploadForm.preTaxReturnsPath
      this.dataDetils.details[indexs].preTaxReturnsPathFileName = this.fileUploadForm.preTaxReturnsPathFileName
      this.dataDetils.details[indexs].taxReturns = this.fileUploadForm.taxReturns
      this.dataDetils.details[indexs].taxReturnsPath = this.fileUploadForm.taxReturnsPath
      this.dataDetils.details[indexs].taxReturnsPathFileName = this.fileUploadForm.taxReturnsPathFileName
      this.dataDetils.details[indexs].paymentCertificate = this.fileUploadForm.paymentCertificate
      this.dataDetils.details[indexs].paymentCertificatePath = this.fileUploadForm.paymentCertificatePath
      this.dataDetils.details[indexs].paymentCertificatePathFileName = this.fileUploadForm.paymentCertificatePathFileName
      this.dataDetils.details[indexs].otherUpload = this.fileUploadForm.otherUpload
      this.dataDetils.details[indexs].otherUploadId = this.fileUploadForm.otherUploadId
      this.dataDetils.details[indexs].otherUploadFileName = this.fileUploadForm.otherUploadFileName
      this.showUploadModal=false
    },
    uploadSuc(res) {
      console.log('1231231',res)
      let key = {
        'PRE_TAX_REPORT': 'preTaxReturns',
        'TAX_REPORT': 'taxReturns',
        'DONE_TAX_REPORT': 'paymentCertificate',
        'OTHER': 'otherUpload'
      }[res.data.materialTypeDict];
      if(res.data.materialTypeDict=="OTHER") {
        this.fileUploadForm[key] = res.data.id;
        this.fileUploadForm[key + 'Id'] = res.data.fileName;
        this.fileUploadForm[key + 'FileName'] = res.data.oriName;
      }else{
        this.fileUploadForm[key] = res.data.id;
        this.fileUploadForm[key + 'Path'] = res.data.fileName;
        this.fileUploadForm[key + 'Path' + 'FileName'] = res.data.oriName
      }
    },
    // 确定
    taxModalOk() {

      let tempObj = this.dataDetils.details
      for(let i=0;i<tempObj.length;i++) {
        if(!tempObj[i].taxReturns || !tempObj[i].paymentCertificate || !tempObj[i].preTaxReturns) {
          this.$Message.error(`税种 ${this.dictTaxCategorysMap.get(tempObj[i].taxDict)} 资料未补全`);
          return
        }
      }
      let params = {
        operateApprove:'0',
        comment:"上报实缴,并补全资料",
        taskId:this.dataDetils.serialNumber,
        userId:this.userInfo.id,
        currentHandler:this.dataDetils.applicantId,
        bean:{
            businessFlowNumber:this.dataDetils.businessFlowNumber,
            saveTime:this.dataDetils.createTime,
            details:this.dataDetils.details,
            applicantId: this.dataDetils.applicantId,
            applicantName: this.dataDetils.applicantName,
            companyId: this.dataDetils.companyId,
            companyName: this.dataDetils.companyName,
            countryCode: this.dataDetils.countryCode,
            countryName: this.dataDetils.countryName,
            currency: this.dataDetils.currency,
            currentHandler: this.dataDetils.currentHandler,
            currentLink: this.dataDetils.currentLink,
            deploymentId: this.dataDetils.deploymentId,
            executeType: this.dataDetils.executeType,
            filePath:this.dataDetils.filePath,
            financialReport: this.dataDetils.financialReport,
            financialReportPath:this.dataDetils.financialReportPath,
            oriName:this.dataDetils.oriName,
            id: this.dataDetils.id,
            remarks: this.dataDetils.remarks,
            serialNumber: this.dataDetils.serialNumber,
            status: this.dataDetils.status,
            tin: this.dataDetils.tin
        }
      }
      console.log('params',params)
      const that = this
      dbrwAudit(params).then(res=>{
        that.$Message.success("操作成功")
        // this.dataDetils={}
        // this.details = []
        // this.auditLogVoList = []
        that.$store.commit("removeTag","reportPaid")
        this.$router.push({name: 'home_index'});
      }).finally(() => {
        // this.loading = false;
      })
    },

    // 取消
    taxModalCancel() {
      this.taxModal=false
    },

    renderTaxDict(h,params) {
      if(this.dictTaxCategorysMap) {
        return h("div",this.dictTaxCategorysMap.get(params.row.taxDict))
      }

    },
    /* 表格日期选择框渲染函数 */
    renderDatePicker(h, params) {
      return h('DatePicker', {
        props: {
          type: params.column.key === 'taxPeriod' ? 'month' : 'date',
          value: params.row[params.column.key]
        },
        on: {
          'on-change': val => {
            this.details[params.index][params.column.key] = val;
          }
        }
      })
    },
    // 操作
    handleSubmit(){
      console.log("adasd",this.dataDetils)
      const that =this
      var tempAll = 0;
      this.dataDetils.details.forEach((item,index)=>{
        tempAll = parseFloat(item.taxPaid) + parseFloat(item.overduePayment);
      })
      if(tempAll==0){
          this.$Message.error('实际缴纳税款不能为空,请输入实缴税额或实缴滞纳金');
          return;
      }
      // //实际缴纳日期
      let sjjnrqi = this.dataDetils.details.some(item => {
        return !item.paymentTime
      })
      if (sjjnrqi) {
        this.$Message.error('请选择实际缴纳日期');
        return;
      }

      this.dataDetils.details.map((item,index)=>{
        if(item.payableTax!=item.taxPaid || item.lateFeePayable!=item.overduePayment){
          that.taxModal=true
          that.dataDetils.status=5
          return
        }else{
          that.dataDetils.status=0
          that.taxModalOk()
        }
      })

    },
    /* 表格栏输入框渲染函数 */
    renderInput(h, params) {
      console.log("21212",params);
      let temp = null;
      if (params.row.taxDict) {
        return h('InputNumber', {
          props: {
            maxlength: 10,
            value: temp || Number(params.row[params.column.key]),
            formatter:(values)=>`${values}`.replace(/([0-9]+\.[0-9]{2})[0-9]*/, '$1'),
            parser:(values) => `${values}`.replace(/$s?|(,*)/g, '$1')
          },
          on: {
            input: e => {
              params.row[params.column.key] = e;
              this.details[params.index] = params.row;
              this.$forceUpdate()
            }
          }
        })
      }
    },
    // table下面添加table
    addTable() {
      let el=this.$refs.table.$el
      let tbodyEl=el.querySelector("tbody")
      let parentEL=tbodyEl.parentNode;
      var tbodyDIV = document.createElement("tbody");
      let tdEL = document.createElement("td");
      tdEL.style['text-align']="center"
      tdEL.innerHTML="合计"
      tbodyDIV.append(tdEL)
      for(let i=0;i<this.detailsColumns.length-1;i++) {
        switch (i) {
          case 1:
            let tdEsL=document.createElement("td")
            tdEsL.setAttribute("id","payableTaxALL")
            tdEsL.style['text-align']="center"
            tbodyDIV.append(tdEsL)
            break;
          case 2:
            let lateFeePayabletdEsL=document.createElement("td")
            lateFeePayabletdEsL.setAttribute("id","lateFeePayable")
            lateFeePayabletdEsL.style['text-align']="center"
            tbodyDIV.append(lateFeePayabletdEsL)
            break;
          case 3:
            let applTaxPaymenttdEsL=document.createElement("td")
            applTaxPaymenttdEsL.setAttribute("id","applTaxPayment")
            applTaxPaymenttdEsL.style['text-align']="center"
            tbodyDIV.append(applTaxPaymenttdEsL)
            break;
          case 5:
            let taxPaidtdEsL=document.createElement("td")
            taxPaidtdEsL.setAttribute("id","taxPaid")
            taxPaidtdEsL.style['text-align']="center"
            tbodyDIV.append(taxPaidtdEsL)
            break;
          case 6:
            let overduePaymenttdEsL=document.createElement("td")
            overduePaymenttdEsL.setAttribute("id","overduePayment")
            overduePaymenttdEsL.style['text-align']="center"
            tbodyDIV.append(overduePaymenttdEsL)
            break;
          case 7:
            let taxsjsktdEsL=document.createElement("td")
            taxsjsktdEsL.setAttribute("id","taxsjsk")
            taxsjsktdEsL.style['text-align']="center"
            tbodyDIV.append(taxsjsktdEsL)
            break;
          default:
            tbodyDIV.append(document.createElement("td"))
            break;
        }

      }
      parentEL.appendChild(tbodyDIV)
      // console.log(parentEL)
    },
    inintPage() {
      let type = this.$route.params.type;
      const dataDetilInfo = this.$route.params.params && this.$route.params.params
      console.log("0000",this.$route.params.params)
      this.selectCurrencyCode = dataDetilInfo.currency
      this.dataDetils = dataDetilInfo
      this.details = dataDetilInfo.details
      this.auditLogVoList = dataDetilInfo.auditLogVoList
      getDictListDataByType(dictType.currency)
        .then(res => {
          let map = new Map()
          this.dictCurrencys = res.data;
          res.data.map((item,index)=>{
            map.set(item.code,item.name)
          })
          this.dictCurrencysMap = map
        });
      getDictListDataByType(dictType.taxCategory)
        .then(res => {
          this.dictTaxCategorys = res.data;
          let maps = new Map()
          res.data.map((item,index)=>{
            maps.set(item.code,item.name)
          })
          this.dictTaxCategorysMap = maps
        })
    },
    // 预览
      priviewFile(v) {
        console.log("v",v)
        const that = this;
        if(v){
          let lastString = v.lastIndexOf(".")
          let filelastName = v.substr(lastString+1)
          if(filelastName=="png" || filelastName=="jpg" || filelastName=="jpeg") {
            let baseurl = fileLoadPath.loadFilePath
            window.open(`${baseurl}${v}?view`)
          }else{
            let base="/api"
            var xhr = new XMLHttpRequest();
            xhr.responseType = "blob";
              xhr.onreadystatechange = function(){
                  if( xhr.readyState == 4){
                      if( xhr.status >= 200 && xhr.status < 300 || xhr.status == 304){
                        let blob = xhr.response
                        let imgTag = URL.createObjectURL(blob)
                        that.priviewFilePath=imgTag
                        window.open(imgTag)
                      }
                  }
              };
              xhr.open("post",`${base}/previewFile`,true);
              xhr.setRequestHeader("Content-Type", "application/json;charset=UTF-8");
              xhr.setRequestHeader("accessToken", this.accessToken);
              xhr.send(JSON.stringify({fileName:v}));
          }
        }else{
          this.$Message.error('没有上传文件!');
        }
      },
    // 下载
    uploadFile(path) {
      if(path){
        let baseurl = fileLoadPath.loadFilePath
        window.open(baseurl+path)
      }else{
        this.$Message.error('没有上传文件!');
      }

    },
    init() {
      this.inintPage()
      this.addTable()
      getFileType("fileType").then((res)=>{
        this.setStore("fileTypeString", res.data&&res.data.propertyValue);
      })
    }
  },
  mounted(){
    this.init()
  },
  updated:function() {
    var  payableTaxALL=0 // 应缴税额合计
    var  lateFeePayable=0// 应缴滞纳金合计
    var  applTaxPayment=0 // 申请纳税款合计
    var  taxPaid=0 // 实缴税款合计
    var  overduePayment=0 //实缴滞纳金合计
    var  taxsjsk=0 // 实际缴纳税款合计
    for(let i=0;i<this.details.length;i++) {
        // payableTaxALL+=this.details[i].payableTax
        payableTaxALL+= this.details[i].payableTax? parseFloat(`${this.details[i].payableTax}`.replace(/([0-9]+\.[0-9]{2})[0-9]*/,"$1")) :0
        lateFeePayable+=this.details[i].lateFeePayable ? parseFloat(`${this.details[i].lateFeePayable}`.replace(/([0-9]+\.[0-9]{2})[0-9]*/,"$1")):0
        applTaxPayment=parseFloat(payableTaxALL)+parseFloat(lateFeePayable)

        taxPaid=parseFloat(taxPaid)
        overduePayment=parseFloat(overduePayment)
        taxPaid+=this.details[i].taxPaid ? parseFloat(`${this.details[i].taxPaid}`.replace(/([0-9]+\.[0-9]{2})[0-9]*/,"$1")) : 0
        overduePayment+=this.details[i].overduePayment ? parseFloat(`${this.details[i].overduePayment}`.replace(/([0-9]+\.[0-9]{2})[0-9]*/,"$1")) : 0
        taxsjsk=parseFloat(taxPaid)+parseFloat(overduePayment)
    }
    // console.log("taxsjsk",taxsjsk)
    document.getElementById("payableTaxALL").innerHTML=payableTaxALL.toFixed(2)
    document.getElementById("lateFeePayable").innerHTML=lateFeePayable.toFixed(2)
    document.getElementById("applTaxPayment").innerHTML=applTaxPayment.toFixed(2)
    document.getElementById("taxPaid").innerHTML=taxPaid.toFixed(2)
    document.getElementById("overduePayment").innerHTML=overduePayment.toFixed(2)
    document.getElementById("taxsjsk").innerHTML=taxsjsk.toFixed(2)
  },
  created:function() {
    this.userInfo = JSON.parse(Cookies.get("userInfo"));
  }
}
</script>

<style scoped lang="less">
.box {
  background: #fff;
  padding: 10px;
}
.taxesTables {
  border-collapse:collapse;
  border:1px solid #e8eaec !important;
  color: #515a6e;
  // font-weight: bold;
   tr,td {
    text-align: center;
    height: 40px;
    border: 1px solid #e8eaec;
  }
  .centerHeight {
    height: 40px;
    td {
      border:none;
    }
  }
}
.myspan {
  cursor: pointer;
  color:rgb(45, 140, 240);
  padding: 2px;
}
.auditLogVoList {
  margin-top: 10px;
}
</style>
