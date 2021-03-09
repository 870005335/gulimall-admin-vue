<template>
  <div>
    <el-row :gutter="20">
      <el-col :span="6"><category @tree-node-click="treeNodeClick"></category></el-col>
      <el-col :span="18"
        ><div class="mod-config">
          <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
            <el-form-item>
              <el-input v-model="dataForm.key" placeholder="参数名"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button @click="getDataList()">查询</el-button>
              <el-button type="success" @click="getAllDataList()">查询全部</el-button>
              <el-button
                v-if="isAuth('product:baseattr:save')"
                type="primary"
                @click="addOrUpdateHandle()"
                >新增</el-button
              >
              <el-button
                v-if="isAuth('product:baseattr:delete')"
                type="danger"
                @click="deleteHandle()"
                :disabled="dataListSelections.length <= 0"
                >批量删除</el-button
              >
            </el-form-item>
          </el-form>
          <el-table
            border
            v-loading="dataListLoading"
            style="width: 100%"
            :data="dataList"
            @selection-change="selectionChangeHandle"
          >
            <el-table-column
              type="selection"
              header-align="center"
              align="center"
              width="50"
            ></el-table-column>
            <el-table-column
              prop="attrId"
              header-align="center"
              align="center"
              label="id"
            ></el-table-column>
            <el-table-column
              prop="attrName"
              header-align="center"
              align="center"
              label="属性名"
            ></el-table-column>
            <el-table-column
              v-if="attrType === 1"
              prop="searchType"
              header-align="center"
              align="center"
              label="可检索"
            >
              <template slot-scope="scope">
                <i class="el-icon-success" v-if="scope.row.searchType == 1"></i>
                <i class="el-icon-error" v-else></i> </template
            ></el-table-column>
            <el-table-column
              prop="valueType"
              header-align="center"
              align="center"
              label="值类型"
            >
              <template slot-scope="scope">
                <el-tag type="success" v-if="scope.row.valueType == 0">单选</el-tag>
                <el-tag v-else>多选</el-tag>
              </template>
            </el-table-column>
            <el-table-column
              prop="icon"
              header-align="center"
              align="center"
              label="图标"
            ></el-table-column>
            <el-table-column
              prop="valueSelect"
              header-align="center"
              align="center"
              label="可选值"
            >
              <template slot-scope="scope">
                <el-tooltip placement="top"
                  ><div slot="content">
                    <span
                      v-for="(i, index) in scope.row.valueSelect.split(';')"
                      :key="index"
                      >{{ i }}<br
                    /></span>
                  </div>
                  <el-tag>{{
                    scope.row.valueSelect.split(";")[0] + " ..."
                  }}</el-tag></el-tooltip
                >
              </template>
            </el-table-column>
            <el-table-column
              prop="enable"
              header-align="center"
              align="center"
              label="启用"
            >
              <template slot-scope="scope">
                <i class="el-icon-success" v-if="scope.row.enable == 1"></i>
                <i class="el-icon-error" v-else></i> </template
            ></el-table-column>
            <el-table-column
              prop="catelogName"
              header-align="center"
              align="center"
              label="所属分类"
            ></el-table-column>
            <el-table-column
              v-if="attrType == 1"
              prop="groupName"
              header-align="center"
              align="center"
              label="所属分组"
            ></el-table-column>
            <el-table-column
              v-if="attrType == 1"
              prop="showDesc"
              header-align="center"
              align="center"
              label="快速展示"
            >
              <template slot-scope="scope">
                <i class="el-icon-success" v-if="scope.row.showDesc == 1"></i>
                <i class="el-icon-error" v-else></i>
              </template>
            </el-table-column>
            <el-table-column
              fixed="right"
              width="150"
              header-align="center"
              align="center"
              label="操作"
              ><template slot-scope="scope">
                <el-button
                  type="text"
                  size="small"
                  @click="addOrUpdateHandle(scope.row.attrId)"
                  >编辑</el-button
                >
                <el-button
                  type="text"
                  size="small"
                  @click="deleteHandle(scope.row.attrId)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
          <add-or-update
            :type="attrType"
            v-if="addOrUpdateVisible"
            @refreshDataList="getDataList"
            ref="addOrUpdate"
          ></add-or-update></div
      ></el-col>
    </el-row>
  </div>
</template>

<script>
import Category from "../common/category";
import AddOrUpdate from "./attr-add-or-update";
// 这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
// 例如：import 《组件名称》 from '《组件路径》';

export default {
  // import引入的组件需要注入到对象中才能使用
  components: { Category, AddOrUpdate },
  props: {
    attrType: {
      type: Number,
      default() {
        return 1;
      },
    },
  },
  data() {
    //这里存放数据
    return {
      catId: 0,
      pageIndex: 0,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataList: [],
      dataForm: {
        key: "",
      },
      dataListSelections: [],
      addOrUpdateVisible: false,
    };
  },
  // 监听属性 类似于data概念
  computed: {},
  // 监控data中的数据变化
  watch: {},
  // 方法集合
  methods: {
    treeNodeClick(data, node, component) {
      if (node.level === 3) {
        this.catId = data.catId;
        this.getDataList();
      }
    },
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    getAllDataList() {
      this.catId = 0;
      this.getDataList();
    },
    getDataList() {
      this.dataListLoading = true;
      let type = this.attrType == 0 ? "sale" : "base";
      this.$http({
        url: this.$http.adornUrl(`/product/attr/${type}/list/${this.catId}`),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
        }),
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.dataList = data.page.list;
          this.totalPage = data.page.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    },
    deleteHandle(id) {
      var ids = id ? [id] : this.dataListSelections.map((item) => item.attrId);
      this.$confirm(
        `确定对id[${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作`,
        "提示",
        { confirmButtonText: "确定", cancelButtonText: "取消", type: "warning" }
      )
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/attr/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            if (data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1000,
                onClose: () => {
                  this.getDataList();
                },
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        })
        .catch(() => {
          this.$message({
            message: "已取消操作",
            type: "info",
            duration: 1000,
          });
        });
    },
  },
  // 生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getDataList();
  },
  // 生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, // 生命周期 - 创建之前
  beforeMount() {}, // 生命周期 - 挂载之前
  beforeUpdate() {}, // 生命周期 - 更新之前
  updated() {}, // 生命周期 - 更新之后
  beforeDestroy() {}, // 生命周期 - 销毁之前
  destroyed() {}, // 生命周期 - 销毁完成
  activated() {}, // 如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style lang="scss" scoped></style>
