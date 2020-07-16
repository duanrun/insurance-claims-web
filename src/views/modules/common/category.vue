<!--  -->
<template>
<el-tree
      :data="menus"
      node-key="catId"
      :props="defaultProps"
      ref="menuTree"
      @node-click="nodeClick"
    >
    </el-tree>
</template>

<script>
export default {
//import引入的组件需要注入到对象中才能使用
components: {},
props:{},
data() {
//这里存放数据
return {
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
},
//监听属性 类似于data概念
computed: {},
//监控data中的数据变化
watch: {},
//方法集合
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
    nodeClick(data,node,component){
            console.log("子组件category被点击"+data,node,component);
            //向父组件发送事件
            this.$emit("tree-node-click",data,node,component);
            
    }
},
//生命周期 - 创建完成（可以访问当前this实例）
created() {
    this.getMenus();
},
//生命周期 - 挂载完成（可以访问DOM元素）
mounted() {

},
beforeCreate() {}, //生命周期 - 创建之前
beforeMount() {}, //生命周期 - 挂载之前
beforeUpdate() {}, //生命周期 - 更新之前
updated() {}, //生命周期 - 更新之后
beforeDestroy() {}, //生命周期 - 销毁之前
destroyed() {}, //生命周期 - 销毁完成
activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
}
</script>
<style scoped>

</style>