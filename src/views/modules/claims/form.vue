<template>
  <div class="mod-config">
    <h1>理赔记录</h1>
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="!isAuth('claim:form:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="!isAuth('claim:form:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
        <!-- <el-button class="filter-item" type="primary" icon="document" @click="handleDownload">导出</el-button> -->
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      ref="multipleTable"
      style="width: 100%;"
      @expand-change="changeForm">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <!-- <el-table-column
        prop="claimFormId"
        header-align="center"
        align="center"
        label="理赔id">
      </el-table-column> -->
      <el-table-column
        header-align="center"
        align="center"
        type="index"
        :index="indexMethod"
        label="理赔序号"
        width="100">
      </el-table-column>
      <el-table-column
        prop="claimFormRecorder"
        header-align="center"
        align="center"
        label="记录人">
      </el-table-column>
      <el-table-column
        prop="claimFormCause"
        header-align="center"
        align="center"
        label="事故原因">
      </el-table-column>
      <el-table-column
        prop="claimFormSite"
        header-align="center"
        align="center"
        label="事故地点">
      </el-table-column>
      <el-table-column
        prop="insuranceInserIncludeId"
        header-align="center"
        align="center"
        label="险种id">
      </el-table-column>
      <!-- <el-table-column
        prop="claimFormPicture"
        header-align="center"
        align="center"
        label="事故图片">
      </el-table-column> -->
      <el-table-column
        prop="claimFormExploration"
        header-align="center"
        align="center"
        label="勘探人">
      </el-table-column>
      <el-table-column
        prop="claimFormExplorationPhone"
        header-align="center"
        align="center"
        label="投保人手机号"
         width="130">
      </el-table-column>
      <el-table-column
        prop="lossAssessmentId"
        header-align="center"
        align="center"
        label="定损id">
      </el-table-column>
      <el-table-column
        prop="materialId"
        header-align="center"
        align="center"
        label="理赔资料id">
      </el-table-column>
      <el-table-column
        prop="indemnityId"
        header-align="center"
        align="center"
        label="赔款id">
      </el-table-column>
      <el-table-column type="expand">
        <template scope="props">
          <el-form label-position="left" inline class="demo-table-expand" size="medium">
            <el-form-item label="投保人姓名:">
              <span>{{userInfo.insuranceUser.userName}}</span>
            </el-form-item>
            <el-form-item label="投保人手机号:">
              <span>{{ props.row.claimFormExplorationPhone }}</span>
            </el-form-item>
            <el-form-item label="投保人身份证号:">
              <span>{{userInfo.insuranceUser.userCard}}</span>
            </el-form-item>5
            <el-form-item label="所买保险类型:" label-width="150px;">
              <!-- <span>{{ props.row.date }}</span> -->
            <el-row v-for="(carInsur,index) in carInsurs" v-if="index in includeOption" style="width:100%;">
                    {{carInsur.ciName}}           {{carInsur.maxmoney}} 元   
                <!-- {{carInsur.ciMoney}} -->
            </el-row>
                <!-- <el-row v-for="(carInsur,index) in carInsurs" v-if="index in includeOption" style="width:100%;">
                     
                </el-row> -->
            </el-form-item>
            <el-form-item>
                <img v-for="item in props.row.claimImgs" :src="item.claimImgUri" width="220" height="220" :key="item" alt="">
            </el-form-item>
            <!-- <el-form-item label="所买保险额度:" label-width="150px;">
                  <span>{{ props.row.date }}</span>
              
            </el-form-item> -->
          </el-form>
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.claimFormId)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.claimFormId)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <div style="margin-top: 20px">
         <el-button @click="toggleSelection()">取消选择</el-button>
      </div>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './form-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        carInsurs:[],//所有保险类型
        len:0,//长度
        insuranceInserCheChuan:null,//车船税
        insuranceInserJiaoQiang:null,//交强险税
        contracts:null,//保险信息
        insuranceInserIncludeOption:null,//套餐信息
        includeOption:[],//套餐中带有的保险
        userInfo:null,//用户信息
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList();
      this.getAllData();
    },
    methods: {
      toggleSelection (rows) {
          this.$refs.multipleTable.clearSelection();
    },
     selectionChangeHandle (val) {
      this.multipleSelection = val;
    },
    changeForm(row,expandedRows,expanded){
      console.log("点击了展开事件",row.insuranceInserIncludeId,expandedRows.length,expanded);
          this.taoCanInfo(row.insuranceInserIncludeId);
          this.getUserInfo(row.claimFormExplorationPhone);
    },
      // 获取数据列表
      getDataList (datas) {
        console.log("父组件传递过来的修改数据",datas);
        this.contracts=datas;
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/claim/form/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key
          })
        }).then(({data}) => {
          console.log("返回的数据",data.page.list);
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.claimFormId
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/claim/form/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      },
      indexMethod(index){
        return index+1;
      },
      getUserInfo(phone){
             return new Promise((resolve, reject)=>{
        this.$http({
        url: this.$http.adornUrl('/claim/form/getUserInfo'),
        method: 'get',
        params: this.$http.adornParams({phone:`${phone}` })
        }).then(({ data }) => {
          // this.carInsurs = data.data.data;
          // this.len = this.carInsurs.length;
          console.log("返回的用回信息", data.data);
          this.userInfo=data.data;
          resolve(data.data);
          
        })
      });
      },
async getAllData(){
		 await this.getInsurs();
		 await this.getinsuranceInserJiaoQiang();
		 await this.getinsuranceInserCheChuan();
   },
   //获取所有商业险
	 getInsurs() {
      return new Promise((resolve, reject)=>{
        this.$http({
        url: this.$http.adornUrl('/claim/form/getAllTypeCarInsur'),
        method: 'get',
        params: this.$http.adornParams({ciType: "商业险", ciState: 1 })
        }).then(({ data }) => {
          this.carInsurs = data.data.data;
          this.len = this.carInsurs.length;
          console.log("返回的商业险数据", data.data.data);
          resolve(data.data.data);
          
        })
      });
	},
	//获取交强险
	getinsuranceInserJiaoQiang() {
    return new Promise((resolve, reject)=>{
    this.$http({
    url: this.$http.adornUrl('/claim/form/getCarInsur'),
    method: 'get',
    params: this.$http.adornParams({id: 1})
    }).then(({ data }) => {
       this.insuranceInserJiaoQiang = data.data;
          console.log("交强税",data.data);
          resolve(data.data);
    })
    });
    },
    //获取车船
    getinsuranceInserCheChuan() {
       return new Promise((resolve, reject)=>{
      this.$http({
      url: this.$http.adornUrl('/claim/form/getCarInsur'),
      method: 'get',
      params: this.$http.adornParams({id: 2})
      }).then(({ data }) => {
         // this.carInsurs = data.data.data;
          this.insuranceInserCheChuan = data;
          console.log("车船税",data.data);
          resolve(data.data);
      })
      });
    },
    //根据套餐id获取套餐信息
    taoCanInfo(id){
         return new Promise((resolve, reject)=>{
           this.$http({
           url: this.$http.adornUrl('/claim/form/getInclude'),
           method: 'get',
           params: this.$http.adornParams({id:`${id}`})
           }).then(({ data }) => {
          this.insuranceInserIncludeOption=data.include;
          let index = 0;
				  let index2 = -3;
		 for (const option in this.insuranceInserIncludeOption){		
		  	 if (option == "ci" + index &&this.insuranceInserIncludeOption[option] == 1) {
				if (index>2) {
					this.includeOption.push(index2);
				}	
			}
				index++;
				index2++;	
     }
          resolve(data);
          // console.log("返回的数据", data.data.data);
        });
      });
    }
    }
  }
</script>
<style type="text/css">
.mod-config .demo-table-expand {
    font-size: 0;
  }
 .mod-config .demo-table-expand label {
    width: 130px;
    color: #99a9bf;
  }
 .mod-config .demo-table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 33%;
  }
</style>