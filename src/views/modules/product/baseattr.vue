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
          <el-table border v-loading="dataListLoading" style="width: 100%">
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
              prop="searchType"
              header-align="center"
              align="center"
              label="可检索"
            ></el-table-column>
          </el-table></div
      ></el-col>
    </el-row>
  </div>
</template>

<script>
import Category from "../common/category";
// 这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
// 例如：import 《组件名称》 from '《组件路径》';

export default {
  // import引入的组件需要注入到对象中才能使用
  components: { Category },
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
      }
    },
    getAllDataList() {},
    getDataList() {},
    addOrUpdateHandle() {},
    deleteHandle() {},
  },
  // 生命周期 - 创建完成（可以访问当前this实例）
  created() {},
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
