<!--  -->
<template>
  <div>
    <el-switch v-model="draggable"
    active-text="开启拖拽"
    inactive-text="关闭拖拽"></el-switch>
    <el-button v-if="draggable"  @click="batchSave">批量保存</el-button>
    <el-button @click="batchDelete" type="danger">批量删除</el-button>
    <el-tree
      :data="menus"
      show-checkbox
      node-key="catId"
      :props="defaultProps"
      @node-drop="handleDrop"
      :expand-on-click-node="false"
      :default-expanded-keys="expandedKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button type="text" v-if="node.level<=2" size="mini" @click="() => append(data)">Append</el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">edit</el-button>
          <el-button
            type="text"
            v-if="node.childNodes.length==0"
            size="mini"
            @click="() => remove(node, data)"
          >Delete</el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      :close-on-click-modal="false"
      width="30%"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图表">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitDate">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data() {
    return {
      pCid:[],
      draggable:false,
      updateNodes:[],
      title: "",
      maxLevel:0,
      dialogType: "", //edit,add
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnit: ""
      },
      menus: [],
      expandedKey: [],
      dialogVisible: false,
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get"
      }).then(({ data }) => {
        //{}解构
        console.log("成功获取到菜单数据", data.data);
        this.menus = data.data;
      });
    },
    submitDate() {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "edit") {
        this.editCategory();
      }
    },
    editCategory() {
      //真正修改的方法
      let { catId, name, icon, productUnit } = this.category;
      let data = {
        catId: catId,
        name: name,
        icon: icon,
        productUnit: productUnit
      };
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(data, false)
      }).then(({ data }) => {
        this.$message({
          message: "菜单修改成功",
          type: "success"
        });
        this.dialogVisible = false;
        //刷新出新的菜单
        this.getMenus();
        //设置默认展开的菜单
        this.expandedKey = [this.category.parentCid];
      });
    },
    edit(data) {
      //修改的方法
      this.dialogType = "edit";
      console.log("要修改的数据", data, this.dialogType);
      this.title = "修改分类";
      this.dialogVisible = true;
      //发送请求获取最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get"
      }).then(({ data }) => {
        //请求成功
        console.log(data);
        this.category.name = data.category.name;
        this.category.catId = data.category.catId;
        this.category.icon = data.category.icon;
        this.category.productUnit = data.category.productUnit;
        this.category.parentCid = data.category.parentCid;
      });
    },
    append(data) {
      this.dialogType = "add";
      this.title = "添加分类";
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      //置空
      this.category.name = "";
      this.category.catId = null;
      this.category.icon = "";
      this.category.productUnit = "";
      console.log("添加方法", data, this.dialogType);
    },
    //添加三级分类的方法
    addCategory() {
      console.log(this.category);
      //发post请求
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.$message({
          message: "菜单保存成功",
          type: "success"
        });
        this.dialogVisible = false;
        //刷新出新的菜单
        this.getMenus();
        //设置默认展开的菜单
        this.expandedKey = [this.category.parentCid];
      });
    },
    remove(node, data) {
      //获取catId
      var ids = [data.catId];
      this.$confirm(`是否删除【${data.name}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            this.$message({
              message: "菜单删除成功",
              type: "success"
            });
            //刷新出新的菜单
            this.getMenus();
            //设置默认展开的菜单
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {});
    },
    allowDrop(draggingNode,dropNode,type){
        //1.被拖动的当前节点以及所在的父节点的总层数不能大于3
    var level=  this.countNodeLevel(draggingNode);
    //当前正在拖动的节点+父节点所在的深度<3即可
    let deep= Math.abs((this.maxLevel-draggingNode.level)+1);
    if(type=="inner"){
       return (deep+dropNode.level)<=3;
    }else {
       return (deep+dropNode.parent.level)<=3;
    }
    },countNodeLevel(node){
        //1.找到所有子节点，求出最大深度
        if(node.childNodes!=null&&node.childNodes.length>0){
          for (let index = 0; index < node.childNodes.length; index++) {
              if(node.childNodes[index].level>this.maxLevel){
                this.maxLevel=node.childNodes[index].level;
              }
              this.countNodeLevel(node.childNodes[index]);
          }
        }
    },
    handleDrop(draggingNode,dropNode,dropType,ev){
         //1.当前节点最新的父节点id
          let pCid=0;
          let siblings=null;
          if (dropType=="before"||dropType=="after") {
              pCid=dropNode.parent.data.catId==undefined?0:dropNode.parent.data.catId;
              siblings=dropNode.parent.childNodes;
          }else{
           pCid= dropNode.data.catId;
              siblings=dropNode.childNodes;
          }
         //2.当前拖拽节点的最新顺序
          for (let i = 0; i < siblings.length; i++) {
              if (siblings[i].data.catId==draggingNode.data.catId) {
                  let catLevel=draggingNode.level;
                  if (siblings[i].level!=draggingNode.level){
                      //如果层级发生了变化
                      catLevel=siblings[i].level;
                    //修改当前遍历的节点的子节点的层级
                      this.updateNodeChildNodeLevel(siblings[i]);
                  }
                //如果遍历到了自己，那么就要相对应的修改自己所在的父id，因为拖拽的父id可能会变
                  this.updateNodes.push({catId:siblings[i].data.catId,sort:i,parentCid:pCid,catLevel:catLevel});

              }else{
                //同级的其他不需要修改父id，只需要修改顺序
                  this.updateNodes.push({catId:siblings[i].data.catId,sort:i});
              }
          }
         //3.当前拖拽节点的最新层级
         console.log("updateNodes:",this.updateNodes);
         this.pCid.push(pCid);
    },
    updateNodeChildNodeLevel(node){
        if (node.childNodes.length>0) {
            for (let i = 0; i < node.childNodes.length; i++) {
                var cNode=node.childNodes[i].data;
                this.updateNodes.push({catId:cNode.catId,catLevel:node.childNodes[i].level});
                this.updateNodeChildNodeLevel(node.childNodes[i])
              }
        }
    },
    batchSave(){
         this.$http({
         url: this.$http.adornUrl('/product/category/update/sort'),
         method: 'post',
         data: this.$http.adornData(this.updateNodes, false)
         }).then(({ data }) => {
                  this.$message({
                  message:"菜单顺序修改成功",
                  type:"success"
                });
                 //刷新出新的菜单
            this.getMenus();
            //设置默认展开的菜单
            this.expandedKey = this.pCid;
            this.updateNodes=[];
            this.maxLevel=0;
            // this.pCid=0;
          });
    },
    batchDelete(){
      let checkNodes=  this.$refs.menuTree.getCheckedNodes();
      console.log("被选中的元素",checkNodes);
      //获取所有要删除元素的id
      let deleteId=[];
      for (let i = 0; i < checkNodes.length; i++) {
          deleteId.push(checkNodes[i].catId);
      }
      this.$confirm(`是否批量删除【${deleteId}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(()=>{
          this.$http({
      url: this.$http.adornUrl('/product/category/delete'),
      method: 'post',
      data: this.$http.adornData(deleteId, false)
      }).then(({ data }) => { 
          this.$message({
            message:"菜单批量删除成功",
            type:"success"
          }); 
          this.getMenus();
      });
      }).catch(()=>{

      })

      
    }
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMenus();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {} //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>