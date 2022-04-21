<!--
  功能：功能描述
  作者：uin
  邮箱：2686221966@qq.com
  时间：2022年04月20日 17:06:46
  版本：v1.0
-->
<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandKey"
      draggable
      :allow-drop="allowDrop"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            增加
          </el-button>

          <el-button type="text" size="mini" @click="edit(data)">
            修改
          </el-button>

          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            删除
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="modal"
    >
      <el-form :model="category">
        <el-form-item label="分类名">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="分类的图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="单位">
          <el-input
            v-model="category.productUnint"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》'

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data() {
    return {
      maxLevel: 1,
      title: "",
      dialogType: "", //edit,add
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnint: "",
      },
      menus: [],
      expandKey: [],
      dialogVisible: false,
      modal: false,
      defaultProps: {
        //子菜单的数据
        children: "children",
        label: "name",
      },
    };
  },
  methods: {
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
    //拖拽时判定目标节点能否被放置。
    //判断的依据是被拖动的当前节点层数以及他的父节点的层数不能大于3
    //draggingNode 被拖拽的的当前节点
    //dropNode 要放的目标节点
    //type：prev、inner、next
    allowDrop(draggingNode, dropNode, type) {
      console.log("allowDrop", draggingNode, dropNode, type);
      //首先要判断的是，当前节点的层数
      this.countCurrNodeLevel(draggingNode.data);
      let deep = this.maxLevel - draggingNode.data.catLevel + 1;
      console.log("被拖动的当前节点的深度", deep);
      //其次，判断目标节点的层数，二者之和 不大于3
      if (type == "inner") {
        return deep + dropNode.catLevel <= 3;
      } else {
        return deep + dropNode.parent.catLevel <= 3;
      }

      if (type == "next") {
        return deep + dropNode.catLevel <= 3;
      }

      if (type == "prev") {
        return deep + dropNode.catLevel > 3 ? false : true;
      }
    },
    //判断的是，当前节点的层数
    countCurrNodeLevel(currNode) {
      //先找到当前节点的子节点的最大深度
      if (currNode.children != null && currNode.children.length > 0) {
        for (let i = 0; i < currNode.children.length; i++) {
          if (currNode.children[i].catLevel > this.maxLevel) {
            this.maxLevel = currNode.children[i].catLevel;
          }
          //看一下当前被遍历的currNode.children[i]节点，有没有子节点
          this.countCurrNodeLevel(currNode.children[i]);
        }
      }
    },
    //判断弹出框是修改提交数据还是add提交数据
    submitData() {
      if (this.dialogType == "add") {
        this.addCateory();
      }
      if (this.dialogType == "edit") {
        this.editCateory();
      }
    },
    //修改按钮
    edit(data) {
      console.log("要修改的数据", data);
      this.dialogType = "edit";
      this.title = "修改分类";
      this.dialogVisible = true;
      //要回显的内容
      //发送请求获取当前分类的最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        //请求成功之后
        console.log("要回显的数据", data.data);
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnint = data.data.productUnint;
        this.category.parentCid = data.data.parentCid;
      });
    },
    //修改要提交的数据
    editCateory() {
      //给需要的数据解构
      var { catId, name, icon, productUnint } = this.category;
      var data = {
        catId: catId,
        name: name,
        icon: icon,
        productUnint: productUnint,
      };
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(data, false),
      }).then((data) => {
        this.$message({
          message: `分类修改成功`,
          type: "success",
        });
      });
      //成功之后管你对话框
      this.dialogVisible = false;
      //刷新分类
      this.getMenus();
      //展开原来的分类
      this.expandKey = [this.category.parentCid];
    },
    //点击增加按钮 弹出对话框
    append(data) {
      console.log("append", data);
      this.dialogType = "add";
      this.title = "增加分类";
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      //清除修改之后的数据回显
      this.category.name = "";
      this.category.catId = null;
      this.category.icon = "";
      this.category.productUnint = "";
    },
    //点击对话框的确认，提交数据
    addCateory() {
      console.log("提交的数据", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: `增加分类成功`,
          type: "success",
        });
        //成功之后管你对话框
        this.dialogVisible = false;
        //刷新分类
        this.getMenus();
        //展开原来的分类
        this.expandKey = [this.category.parentCid];
      });
    },
    remove(node, data) {
      var ids = [data.catId];
      //弹框提示
      this.$confirm(`是否删除该[${data.name}]分类, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            this.$message({
              message: `删除分类成功`,
              type: "success",
            });
            //删除成功以后，需要刷新menu
            this.getMenus();
            //设置需要展开的菜单
            this.expandKey = [node.parent.data.catId];
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });

      console.log("remove", node, data);
    },
  },
  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
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
