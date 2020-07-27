<!--  -->
<template>
<div>
    <h1>测试oss</h1>
    <multiUpload v-model="test"></multiUpload>
    <img src="https://gulimall-wb.oss-cn-hangzhou.aliyuncs.com/2020-07-27/5f3aca24-449a-4ce1-8641-55cf0e9175fe_1.jpg" alt="">
</div>
</template>

<script>
import multiUpload from "@/components/upload/multiUpload"
export default {
//import引入的组件需要注入到对象中才能使用
components: {multiUpload},
props:{},
data() {
//这里存放数据
return {
    test:''
};
},
//监听属性 类似于data概念
computed: {},
//监控data中的数据变化
watch: {},
//方法集合
methods: {

},
//生命周期 - 创建完成（可以访问当前this实例）
created() {

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