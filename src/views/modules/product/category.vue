<template>
  <div>
    <el-button type="danger" size="small" @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="menu"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      :draggable="true"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
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
            新增
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)"> 编辑 </el-button>
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
      :title="dialogTitle"
      :visible.sync="dialogVisible"
      width="40%"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
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
export default {
  components: {},
  data() {
    return {
      updateNodes: [],
      maxLevel: 0,
      dialogTitle: "新增分类",
      dialogType: "add",
      category: { name: "" },
      dialogVisible: false,
      menu: [],
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  computed: {},
  watch: {},
  methods: {
    // 向后台发送请求获取树形结构数据
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then((data) => {
        if (data.data.code === 0) {
          this.menu = data.data.trees;
        }
      });
    },
    batchDelete() {
      let catIds = [];
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      catIds = checkedNodes.map((check) => check.catId);
      console.log(checkedNodes);
      if (catIds.length === 0) {
        this.$message({
          message: "未选中节点，请重试",
          type: "warning",
          duration: 1000,
        });
      } else {
        this.$http({
          url: this.$http.adornUrl("/product/category/delete"),
          method: "post",
          data: this.$http.adornData(catIds, false),
        }).then(({ data }) => {
          if (data.code === 0) {
            this.$message({
              message: "批量删除成功",
              type: "success",
              duration: 1000,
            });
          } else {
            this.$message.error(data.msg);
          }
          this.getMenus();
        });
      }
    },
    // 批量修改分类节点
    updateCategoryBatch() {
      this.$http({
        url: this.$http.adornUrl("/product/category/update/batch"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      }).then(({ data }) => {
        if (data.code === 0) {
          this.$message({
            message: "编辑成功",
            type: "success",
            duration: 1000,
          });
        } else {
          this.$message.error(data.message);
        }
        //刷新出新的菜单
        this.getMenus();
      });
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      // console.log(draggingNode, dropNode, dropType);
      // 拖拽节点完成之后的父节点id
      let parentCid = 0;
      // 拖拽节点完成之后的兄弟节点数组
      let siblings = [];
      if (dropType == "before" || dropType == "after") {
        parentCid =
          dropNode.parent.data.catId == undefined ? 0 : dropNode.parent.data.catId;
        siblings = dropNode.parent.childNodes;
      } else {
        parentCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }
      // 重新遍历排序兄弟节点数组
      for (let i = 0; i < siblings.length; i++) {
        // 遍历到拖拽节点
        if (siblings[i].data.catId == draggingNode.data.catId) {
          // 节点层级发生变化
          let draggingNodeLevel = draggingNode.level;
          if (draggingNode.level != siblings[i].level) {
            draggingNodeLevel = siblings[i].level;
            // 修改子节点层级
            this.updateChildNodesLevel(siblings[i]);
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: parentCid,
            catLevel: draggingNodeLevel,
          });
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }
      // console.log(this.updateNodes);
      // 向后台发送请求
      this.updateCategoryBatch();
      //设置需要默认展开的菜单
      this.expandedKey = [parentCid];
      this.updateNodes = [];
      this.maxLevel = 0;
    },
    updateChildNodesLevel(node) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level,
          });
          this.updateChildNodesLevel(node.childNodes[i]);
        }
      }
    },
    allowDrop(draggingNode, dropNode, type) {
      // console.log(draggingNode, dropNode, type)
      // 计算以当前拖拽节点为根节点的树的深度
      this.caculateMaxLevel(draggingNode);
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1;
      if (type === "inner") {
        return deep + dropNode.level <= 3;
      }
      return deep + dropNode.parent.level <= 3;
    },
    caculateMaxLevel(node) {
      // 找出节点的最深度子节点
      if (node.childNodes && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level;
          }
          this.caculateMaxLevel(node.childNodes[i]);
        }
      } else {
        this.maxLevel = node.level;
      }
    },
    edit(data) {
      this.dialogType = "edit";
      this.dialogTitle = "编辑分类";
      this.dialogVisible = true;
      // 请求后台接口获取分类详情数据
      this.$http({
        url: this.$http.adornUrl("/product/category/info"),
        method: "get",
        params: this.$http.adornParams({ catId: data.catId }, false),
      }).then(({ data }) => {
        this.category = data.data;
      });
    },
    append(data) {
      this.dialogVisible = true;
      this.category = {};
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.showStatus = 1;
      this.category.sort = 0;
    },
    submitData() {
      if (this.dialogType === "edit") {
        this.editCategory();
      } else {
        this.addCategory();
      }
    },
    editCategory() {
      var { catId, name, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false),
      }).then(({ data }) => {
        if (data.code === 0) {
          this.$message({
            message: "编辑分类成功",
            type: "success",
            duration: 1000,
          });
          this.expandedKey = [this.category.parentCid];
          this.dialogVisible = false;
          this.getMenus();
        } else {
          this.$message.error(data.msg);
        }
      });
    },
    // 向后台发送请求保存分类数据
    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        if (data.code === 0) {
          this.$message({
            message: "新增分类成功",
            type: "success",
            duration: 1000,
          });
          this.expandedKey = [this.category.parentCid];
          this.dialogVisible = false;
          this.getMenus();
        } else {
          this.$message.error(data.msg);
        }
      });
    },
    remove(node, data) {
      // 获取当前删除的节点id
      console.log(data);
      var ids = [data.catId];
      this.$confirm(`是否删除[${data.name}]菜单?`, "提示", {
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
            if (data.code === 0) {
              this.$message({
                message: "删除分类成功",
                type: "success",
                duration: 1000,
              });
              this.expandedKey = [node.parent.data.catId];
              this.getMenus();
            } else {
              this.$message.error(data.msg);
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
  },
  created() {
    this.getMenus();
  },
  mounted() {},
  beforeCreate() {},
  beforeMount() {},
  beforeUpdate() {},
  updated() {},
  beforeDestroy() {},
  destroyed() {},
  activated() {},
};
</script>
<style lang="scss" scoped>
//@import url(); 引入公共css类
</style>
