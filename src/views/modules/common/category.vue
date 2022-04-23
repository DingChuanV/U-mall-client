<!--
  功能：功能描述 分类菜单树公共部分提取出来 来复用
  作者：uin
  邮箱：2686221966@qq.com
  时间：2022年04月22日 21:18:06
  版本：v1.0
-->
<template>
  <el-tree
    :data="menus"
    :props="defaultProps"
    node-key="catId"
    ref="menuTree"
    @node-click="nodeClick"
  >
  </el-tree>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》'

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data() {
    //这里存放数据
    return {
      menus: [],
      expandKey: [],
      defaultProps: {
        //子菜单的数据
        children: "children",
        label: "name",
      },
    };
  },
  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    //获取菜单数据
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        //({data})将数据解构
        console.log("收到的分类数据", data.data);
        this.menus = data.data;
      });
    },
    //子组件给父组件中的table传递数据 利用点击事件的回调函数
    //data 需要传递的数据
    //node 当前节点
    // component 父组件
    nodeClick(data, node, component) {
      console.log("子组件被点击要传递给父组件的数据", data, node, component);
      //想父组件发送事件，让父组件也知道
      this.$emit("tree-node-click", data, node, component);
    },
  },
  //声明周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMenus();
  },
  //声明周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>
