<template>
  <div>
    <el-tree
      :data="menu"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
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
          <el-button type="text" size="mini" @click="() => edit(data)">
            编辑
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
          <el-input
            v-model="category.productUnit"
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
export default {
  components: {},
  data() {
    return {
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
<style lang='scss' scoped>
//@import url(); 引入公共css类
</style>