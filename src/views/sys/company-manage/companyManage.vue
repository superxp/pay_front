<style lang="less">
@import "./companyManage.less";
.vertical-center {
  margin: 5px;
  text-align: center;
}
</style>
<template>
    <div class="search">
        <Row>
          <Form ref="searchForm" :model="searchForm" inline :label-width="120" class="search-form">
              <Form-item label="E编码" prop="eCode">
                <Input type="text" v-model="searchForm.eCode" clearable placeholder="请输入E编码" style="width: 200px" :maxlength="20"/>
              </Form-item>
              <Form-item style="margin-left:-35px;" class="br">
                <Button @click="handleSearch" type="primary" icon="ios-search">搜索</Button>
                <Button @click="handleResetSearch" >重置</Button>
              </Form-item>
          </Form>
            <Col>
                <Card>
                    <Row class="operation">
                        <Button @click="addCompany" type="primary" icon="md-add">新增公司</Button>
                        <!-- <Button @click="delAll" icon="md-trash">批量删除</Button> -->
                        <Button @click="init" icon="md-refresh">刷新</Button>
                        <circleLoading v-if="operationLoading"/>
                    </Row>
                    <Row>
                        <Table :loading="loading" border :columns="columns" :data="data" ref="table" sortable="custom" @on-sort-change="changeSort" @on-selection-change="changeSelect"></Table>
                    </Row>
                    <Row type="flex" justify="end" class="page">
                        <Page :current="pageNumber" :total="total" :page-size="pageSize" @on-change="changePage" @on-page-size-change="changePageSize" :page-size-opts="[10,20,50]" size="small" show-total show-elevator show-sizer></Page>
                    </Row>
                </Card>
            </Col>
        </Row>
        <Modal :title="modalTitle" v-model="modalVisible" :mask-closable='false' :width="500">
          <Form ref="form" :model="form" :label-width="120" :rules="formValidate">
            <FormItem label="公司名称" prop="name">
              <Input v-model="form.name" placeholder="请输入公司名称" clearable :maxlength="20"/>
            </FormItem>
            <FormItem label="后台识别号码" prop="tin">
              <Input v-model="form.tin" clearable :maxlength="20"/>
            </FormItem>
            <FormItem label="所属国家" prop="countryCode">
              <Select v-model="form.countryCode" style="width:200px">
                <Option v-for="item in dictCountrys" :value="item.code" :key="item.id">{{ item.name }}</Option>
              </Select>
            </FormItem>
            <FormItem label="成立日期" prop="establishmentTime">
              <DatePicker v-model="form.establishmentTime" type="date"  clearable placeholder="选择日期" style="width: 200px"></DatePicker>
            </FormItem>
            <FormItem label="注销日期" prop="writeOffTime">
              <DatePicker v-model="form.writeOffTime" type="date"  clearable placeholder="选择日期" style="width: 200px"></DatePicker>
            </FormItem>
            <FormItem label="币种" prop="currencyCode">
              <Select v-model="form.currencyCode" style="width:200px">
                <Option v-for="item in dictCurrencys" :value="item.code" :key="item.id">{{ item.name }}</Option>
              </Select>
            </FormItem>
            <FormItem label="备注" prop="remarks">
              <Input v-model="form.remarks" :maxlength="40"/>
            </FormItem>
          </Form>
          <div slot="footer">
            <Button type="text" @click="cancelSubmit">取消</Button>
            <Button type="primary" :loading="submitLoading" @click="handleSubmit">提交</Button>
          </div>
        </Modal>
        <Modal title="税种列表" v-model="modalTaxes" :mask-closable='false' :width="700">
          <main>
            <Table border ref="selection" :columns="columns4" @on-selection-change="selectionChange" :data="taxesList"></Table>
          </main>
          <footer class="vertical-center" slot="footer">
              <Button @click="handleReset" style="width: 100px;">取消</Button>
              <Button type="primary" style="width: 100px;margin-left:8px" :loading="savePassLoading" @click="handleSubmitTaxe">保存</Button>
          </footer>
        </Modal>
        <Modal title="分配人员" v-model="userListModel" :mask-closable='false' :width="700" >
          <main style="max-height: 500px;overflow: auto;">
            <Tabs :animated="false">
                <TabPane label="后台专员">
                  <Table border
                  ref="taxationUserListSelectionChange"
                  :columns="userListTable"
                  @on-selection-change="taxationUserListSelectionChange"
                  :data="taxationUserList"></Table>
                </TabPane>
                <TabPane label="审核人">
                  <Table
                  border
                  ref="reviewerUserListSelectionChange"
                  :columns="userListTable"
                   @on-selection-change="reviewerUserListSelectionChange"
                   :data="reviewerUserList"></Table>
                </TabPane>
                <TabPane label="查看人">
                  <Table
                  border
                  ref="viewerUserListSelectionChange"
                  :columns="userListTable"
                  @on-selection-change="viewerUserListSelectionChange"
                  :data="viewerUserList"></Table>
                </TabPane>
            </Tabs>
          </main>
          <footer class="vertical-center" slot="footer">
              <Button @click="handleReset" style="width: 100px;">取消</Button>
              <Button type="primary" style="width: 100px;margin-left:8px"  @click="handleSubmitUserlist">保存</Button>
          </footer>
        </Modal>
    </div>
</template>

<script>
import {
  getAssignTaxes,
  getCompanyListData,
  addCompany,
  editCompany,
  deleteCompany,
  getDictListDataByType,
  getUserListByRoleCode,
  submitUserListByRole
} from "@/api/index";
import { dictType } from "@/libs/constance.js";
import circleLoading from "../../my-components/circle-loading.vue";
export default {
  name: "company-manage",
  components: {
    circleLoading
  },
  data() {
    return {
      taxesTypes: [],
      savePassLoading: false,
      userListModel:false,
      searchForm:{
        eCode:""
      },
      columns4: [
        {
          type: "selection",
          width: 60,
          align: "center"
        },
        {
          title: "税种名称",
          key: "name"
        },
        {
          title: "税种代码",
          key: "code"
        }
      ],
      userListTable:[
        {
          type: "selection",
          width: 60,
          align: "center"
        },
        {title:"姓名",key:"username"},
      ],
      taxesList: [],
      modalTaxes: false,
      loading: true,
      operationLoading: false,
      sortColumn: "createTime",
      sortType: "desc",
      modalType: 0,
      modalVisible: false,
      modalTitle: "",
      form: {
        name: "",
        tin: "",
        countryCode: "",
        currencyCode: "",
        remarks: "",
        establishmentTime:"",
        writeOffTime:""
      },
      formValidate: {
        name: [
          { required: true, message: "公司名称不能为空", trigger: "blur" }
        ],
        tin: [
          { required: true, message: "后台识别编号不能为空", trigger: "blur" }
        ],
        countryCode: [
          { required: true, message: "所属国家不能为空", trigger: "blur" }
        ],
        currencyCode: [
          { required: true, message: "币种不能为空", trigger: "blur" }
        ],
        establishmentTime:[
            { required: true, message: "成立日期不能为空", trigger: "blur",pattern: /.+/ }
        ]
      },
      submitLoading: false,
      selectList: [],
      selectCount: 0,
      columns: [
        {
          type: "selection",
          width: 60,
          align: "center"
        },
        {
          type: "index",
          // width: 60,
          align: "center"
        },
        {
          title: "公司名称",
          key: "name",
          sortable: true,
          // width: 200
        },
        {
          title: "后台识别号码",
          key: "tin",
          sortable: true,
          // width: 140
        },
        {
          title: "成立日期",
          key: "establishmentTime",
          sortable: true,
          // width: 160,
          render: (h, params) => {
            return h("div", params.row.establishmentTime && new Date(params.row.establishmentTime).format());
          }
        },
        {
          title: "注销日期",
          key: "writeOffTime",
          sortable: true,
          // width: 160,
          render: (h, params) => {
            return h("div", params.row.writeOffTime && new Date(params.row.writeOffTime).format());
          }
        },
        {
          title: "所属国家",
          key: "countryCode",
          sortable: true,
          // width:100,
          render:this.renderName
        },
        {
          title: "币种",
          key: "currencyCode",
          sortable: true,
          // width:100,
          render:this.renderCurrencysName
        },
        {
          title:"是否分配",
          key:"isAssign",
          sortable: true,
          // width:100,
          render: (h, params) => {
            return h('div', params.row.isAssign && params.row.isAssign==1?"已分配":"未分配")
          }
        },
        {
          title: "备注",
          key: "remarks",
          // width:150
        },
        {
          title: "操作",
          key: "action",
          align: "center",
          fixed:"right",
          minWidth: 300,
          render: (h, params) => {
            return h("div", [
              h(
                "Button",
                {
                  props: {
                    type: "primary",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.checkPeople(params.row);
                    }
                  }
                },
                "分配人员"
              ),
              h(
                "Button",
                {
                  props: {
                    type: "primary",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.DisTaxes(params.row);
                    }
                  }
                },
                "增加税种"
              ),
              h(
                "Button",
                {
                  props: {
                    type: "primary",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.edit(params.row);
                    }
                  }
                },
                "编辑"
              ),
              h(
                "Button",
                {
                  props: {
                    type: "error",
                    size: "small"
                  },
                  on: {
                    click: () => {
                      this.remove(params.row);
                    }
                  }
                },
                "删除"
              )
            ]);
            // }
          }
        }
      ],
      data: [],
      pageNumber: 1,
      pageSize: 10,
      total: 0,
      dictCountrys: [],
      dictCurrencys: [],
      currentRows: [],
      companyId: "",
      dictCurrencysMap:"",
      dictCountrysMap:"",
      taxationUserList:[],
      reviewerUserList:[],
      viewerUserList:[],
      taxationIds:"",
      reviewerIds:"",
      viewerIds:"",
      tempItem:""
    };
  },
  methods: {
    handleSearch(){
      this.loading = true;
      let params = {
        pageVo: {
          pageNumber: this.pageNumber,
          pageSize: this.pageSize
        },
        companyVo: {
          name: "",
          tin: "",
          eCode:this.searchForm.eCode
        }
      };
      getCompanyListData(params)
        .then(res => {
          this.loading = false;
          this.data = res.data.list;
          this.total = res.data.totalElements;
        })
        .catch(() => {
          this.loading = false;
        });
    },
    handleResetSearch(){
      this.searchForm.eCode=''
    },
    fetchTaxes() {
      let vm = this;
      vm.taxesList = [];
      getDictListDataByType(dictType.taxCategory).then(result => {
        if (result.status == "0") {
          let res = result.data;
          for (let index = 0; index < vm.taxesTypes.length; index++) {
            const element = vm.taxesTypes[index];
            res.forEach(row => {
              if (element.name === row.name) {
                if (typeof row._checked == "undefined") {
                  vm.$set(row, "_checked", true);
                } else {
                  row._checked = true;
                }
              }
            });
          }
          vm.taxesList = res;
        }
      });
    },
    selectionChange(val) {
      val.forEach(row => {
        if (typeof row._checked == "undefined") {
          this.$set(row, "_checked", true);
        } else {
          row._checked = true;
        }
      });
      this.currentRows = val
    },
    handleReset() {
      this.modalTaxes = false;
      this.userListModel = false;
    },
    handleSubmitUserlist() {
      console.log("this.taxationIds",this.taxationIds)
      var temptaxationIds=[]
      var tempreviewerIds=[]
      var tempviewerIds=[]
      this.taxationIds && this.taxationIds.map((item,index)=>{
        temptaxationIds.push(item.id)
      })
      this.reviewerIds && this.reviewerIds.map((item,index)=>{
        tempreviewerIds.push(item.id)
      })
      this.viewerIds && this.viewerIds.map((item,index)=>{
        tempviewerIds.push(item.id)
      })

      if(temptaxationIds.length>1) {
        this.$Message.error("只能选择一个后台专员")
        return
      }

      if(tempreviewerIds.length>1) {
        this.$Message.error("只能选择一个审核人")
        return
      }


      let params ={
        id:this.tempItem && this.tempItem.id,
        taxationIds:temptaxationIds.join(","),
        reviewerIds:tempreviewerIds.join(","),
        viewerIds:tempviewerIds.join(",")
      }
      submitUserListByRole(params).then((res)=>{
        if (res.status == "0") {
          this.$Message.success(res.data)
          this.modalTaxes = false;
          this.userListModel = false;
          this.getCompanyList();
        }
      })
    },
    taxationUserListSelectionChange(val){
      console.log("val",val)
      this.taxationIds=val
    },
    reviewerUserListSelectionChange(val){
      this.reviewerIds=val
    },
    viewerUserListSelectionChange(val){
      this.viewerIds=val
    },
    handleSubmitTaxe() {
      let vm = this;
      let arrIds = []
      vm.currentRows.forEach(element => {
        if (element._checked) {
          arrIds.push(element.id)
        }
      });
      let params = {
        companyId: vm.companyId,
        taxesIds: arrIds.toString()
      };
      vm.savePassLoading = true;
      getAssignTaxes(params).then(result => {
        vm.savePassLoading = false;
        if (result.status == "0") {
          this.$Message.success(result.data);
          this.modalTaxes = false;
          this.userListModel = false;
          this.getCompanyList();
        }
      }).catch(res => {
        vm.savePassLoading = false;
      });
    },
    DisTaxes(row) {
      this.companyId = row.id;
      this.taxesTypes = row.dicts;
      this.fetchTaxes();
      this.modalTaxes = true;
    },
    init() {
      this.getCompanyList();
      this.getDictData();
      getUserListByRoleCode("ROLE_COMMISSIONER_OF_TAX").then((res)=>{
        this.taxationUserList=res.data
      })
      getUserListByRoleCode("ROLE_REVIEWER").then((res)=>{
        this.reviewerUserList=res.data
      })
      getUserListByRoleCode("VIEW").then((res)=>{
        this.viewerUserList=res.data
      })
    },
    changePage(v) {
      this.pageNumber = v;
      this.getCompanyList();
      this.clearSelectAll();
    },
    changePageSize(v) {
      this.pageSize = v;
      this.getCompanyList();
    },
    changeSort(e) {
      this.sortColumn = e.key;
      this.sortType = e.order;
      if (e.order === "normal") {
        this.sortType = "";
      }
      this.getCompanyList();
    },
    getCompanyList() {
      this.loading = true;
      let params = {
        pageVo: {
          pageNumber: this.pageNumber,
          pageSize: this.pageSize
        },
        companyVo: {
          name: "",
          tin: ""
        }
      };
      getCompanyListData(params)
        .then(res => {
          this.loading = false;
          this.data = res.data.list;
          this.total = res.data.totalElements;
        })
        .catch(() => {
          this.loading = false;
        });
    },
    cancelSubmit() {
      this.modalVisible = false;
    },
    checkPeople(v){
      this.tempItem = v
      console.log("v",v)
      var temptaxationIdsArry=v.taxationIds.split(",")
      var tempreviewerIdsArry=v.reviewerIds.split(",")
      var tempviewerIdsArry=v.viewerIds.split(",")
      var tempTaxationList = JSON.parse(JSON.stringify(this.taxationUserList))
      var tempReviewerUserList = JSON.parse(JSON.stringify(this.reviewerUserList))
      var tempViewerUserList = JSON.parse(JSON.stringify(this.viewerUserList))

      tempTaxationList.forEach((item,index)=>{
        item._checked=false
      })

      tempReviewerUserList.forEach((item,index)=>{
        item._checked=false
      })

      tempViewerUserList.forEach((item,index)=>{
        item._checked=false
      })

      for(let i=0;i<temptaxationIdsArry.length;i++) {
        let tempItem = temptaxationIdsArry[i]
        tempTaxationList.forEach((item,index)=>{
          if(item.id==tempItem){
            item._checked=true
          }
        })
      }
      for(let i=0;i<tempreviewerIdsArry.length;i++) {
        let tempItem = tempreviewerIdsArry[i]
        tempReviewerUserList.forEach((item,index)=>{
          if(item.id==tempItem){
            item._checked=true
          }
        })
      }
      for(let i=0;i<tempviewerIdsArry.length;i++) {
        let tempItem = tempviewerIdsArry[i]
        tempViewerUserList.forEach((item,index)=>{
          if(item.id==tempItem){
            item._checked=true
          }
        })
      }
      this.taxationUserList=tempTaxationList
      this.reviewerUserList=tempReviewerUserList
      this.viewerUserList=tempViewerUserList
      this.userListModel=true
    },
    handleSubmit() {
      this.$refs.form.validate(valid => {
        if (valid) {
          if (this.modalType === 0) {
            // 添加
            this.submitLoading = true;
            addCompany(this.form)
              .then(res => {
                this.submitLoading = false;
                this.$Message.success("操作成功");
                this.getCompanyList();
                this.modalVisible = false;
              })
              .catch(() => {
                this.submitLoading = false;
              });
          } else {
            this.submitLoading = true;
            editCompany(this.form)
              .then(res => {
                this.submitLoading = false;
                this.$Message.success("操作成功");
                this.getCompanyList();
                this.modalVisible = false;
              })
              .catch(() => {
                this.submitLoading = false;
              });
          }
        }
      });
    },
    addCompany() {
      this.modalType = 0;
      this.modalTitle = "添加公司";
      this.$refs.form.resetFields();
      (this.form = {
        name: "",
        tin: "",
        countryCode: "",
        currencyCode: "",
        remarks: ""
      }),
        (this.modalVisible = true);
    },
    edit(v) {
      this.modalType = 1;
      this.modalTitle = "编辑公司";
      // 转换null为""
      for (let attr in v) {
        if (v[attr] === null) {
          v[attr] = "";
        }
      }
      v.establishmentTime = new Date(v.establishmentTime).format("yyyy-MM-dd")
      v.writeOffTime = v.writeOffTime && new Date(v.writeOffTime).format("yyyy-MM-dd")
      let str = JSON.stringify(v);
      let data = JSON.parse(str);


      this.form = data;
      this.modalVisible = true;
    },
    remove(v) {
      this.$Modal.confirm({
        title: "确认删除",
        content: "您确认要删除公司 " + v.name + " ?",
        onOk: () => {
          this.operationLoading = true;
          deleteCompany(v.id)
            .then(res => {
              this.operationLoading = false;
              this.$Message.success("操作成功");
              this.getCompanyList();
            })
            .catch(() => {
              this.operationLoading = false;
            });
        }
      });
    },
    clearSelectAll() {
      this.$refs.table.selectAll(false);
    },
    changeSelect(e) {
      this.selectList = e;
      this.selectCount = e.length;
    },
    delAll() {
      if (this.selectCount <= 0) {
        this.$Message.warning("您还未选择要删除的数据");
        return;
      }
      this.$Modal.confirm({
        title: "确认删除",
        content: "您确认要删除所选的 " + this.selectCount + " 条数据?",
        onOk: () => {
          let ids = "";
          this.selectList.forEach(function(e) {
            ids += e.id + ",";
          });
          ids = ids.substring(0, ids.length - 1);
          this.operationLoading = true;
          deleteQuartz(ids).then(res => {
            this.operationLoading = false;
            if (res.success === true) {
              this.$Message.success("删除成功");
              this.clearSelectAll();
              this.getCompanyList();
            }
          });
        }
      });
    },
    /* 获取字典信息 */
    getDictData() {
      getDictListDataByType(dictType.country).then(res => {
        this.dictCountrys = res.data;
        let maps = new Map()
        res.data.map((item,index)=>{
          maps.set(item.code,item.name)
        })
        this.dictCountrysMap=maps
      });
      getDictListDataByType(dictType.currency).then(res => {
        this.dictCurrencys = res.data;
        var maps = new Map()
        res.data.map((item,index)=>{
          maps.set(item.code,item.name)
        })
        this.dictCurrencysMap=maps
      });
    },
    // 国家根据code获取中文
    renderName(h,params) {
      // var submitStr=""
      // this.dictCountrys.map((item,index)=>{
      //   if(item.code==params.row.countryCode) {
      //       submitStr=item.name
      //   }
      // })
      //   return h('div',submitStr)
      if(this.dictCountrysMap) {
          return h('div',this.dictCountrysMap.get(params.row.countryCode))
      }
    },
    // 根据币种code返回中文名
    renderCurrencysName(h,params) {
      if(this.dictCurrencysMap) {
        return h('div',this.dictCurrencysMap.get(params.row.currencyCode))
      }
    }
  },
  mounted() {
    this.init();
  }
};
</script>
