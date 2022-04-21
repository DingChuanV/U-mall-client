<!--
  功能：功能描述
  作者：uin
  邮箱：2686221966@qq.com
  时间：2022年04月20日 17:06:46
  版本：v1.0
-->
<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    >
    </el-switch>
    <el-button @click="batchSave" v-if="draggable">批量保存</el-button>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>

    <el-tree
      @node-drop="handleDrop"
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      ref="menuTree"
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
  name: "category",
  //import引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data() {
    return {
      pCid: [],
      updateNodes: [],
      maxLevel: 1,
      title: "",
      dialogType: "", //edit,add
      draggable: false,
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
    // 批量删除
    batchDelete() {
      let catIds = [];
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      console.log("被选中的元素", checkedNodes);
      for (let i = 0; i < checkedNodes.length; i++) {
        catIds.push(checkedNodes[i].catId);
      }

      this.$confirm(`是否批量删除【${catIds}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false),
          })
            .then(({ data }) => {
              this.$message({
                type: "success",
                message: "菜单批量删除成功!",
              });
              // 刷新出新的菜单
              this.getMenus();
            })
            .catch(() => {});
        })
        .catch(() => {});
    },
    // 批量修改
    batchSave() {
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      })
        .then(({ data }) => {
          this.$message({
            type: "success",
            message: "菜单顺序修改成功!",
          });
          // 刷新出新的菜单
          this.getMenus();
          // 设置需要默认展开的菜单
          this.expandedKey = this.pCid;
          this.updateNodes = [];
          //清空方法
          this.maxLevel = 0;
          // this.pCid = 0;
        })
        .catch(() => {});
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log("handleDrop: ", draggingNode, dropNode, dropType);

      //1 当前节点最新的父节点
      let pCid = 0;
      let siblings = null;
      if (dropType == "before" || dropType == "after") {
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId;
        siblings = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }
      this.pCid.push(pCid);
      //2 当前拖拽节点的最新顺序
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId == draggingNode.data.catId) {
          // 如果遍历的是当前正在拖拽的节点
          let catLevel = draggingNode.level;
          if (siblings[i].level != draggingNode.level) {
            // 当前节点的层级发生变化
            catLevel = siblings[i].level;
            // 修改他子节点的层级
            this.updateChildNodeLevlel(siblings[i]);
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel,
          });
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }
      //3 当前拖拽节点的最新层级
      console.log("updateNodes", this.updateNodes);
    },
    updateChildNodeLevlel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level,
          });
          this.updateChildNodeLevlel(node.childNodes[i]);
        }
      }
    },
    allowDrop(draggingNode, dropNode, type) {
      //1 被拖动的当前节点以及所在的父节点总层数不能大于3

      //1 被拖动的当前节点总层数
      console.log("allowDrop:", draggingNode, dropNode, type);

      var level = this.countNodeLevel(draggingNode);

      // 当前正在拖动的节点+父节点所在的深度不大于3即可
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1;
      console.log("深度:", deep);

      // this.maxLevel
      if (type == "innner") {
        // console.log(
        //   `this.maxLevel: ${this.maxLevel}; draggingNode.data.catLevel:${draggingNode.data.catLevel};dropNode.level: ${dropNode.level}`
        // );
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
    },
    countNodeLevel(node) {
      // 找到所有子节点，求出最大深度
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level;
          }
          this.countNodeLevel(node.childNodes);
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
