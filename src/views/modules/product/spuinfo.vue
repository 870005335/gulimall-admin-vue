<template>
  <div class="mod-config">
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%"
    >
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50"
      ></el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="id"
      ></el-table-column>
      <el-table-column
        prop="spuName"
        header-align="center"
        align="center"
        label="名称"
      ></el-table-column>
      <el-table-column
        prop="spuDescription"
        header-align="center"
        align="center"
        label="描述"
      ></el-table-column>
      <el-table-column
        prop="catalogId"
        header-align="center"
        align="center"
        label="分类Id"
      ></el-table-column>
      <el-table-column
        prop="brandId"
        header-align="center"
        align="center"
        label="品牌Id"
      ></el-table-column>
      <el-table-column
        prop="weight"
        header-align="center"
        align="center"
        label="重量"
      ></el-table-column>
      <el-table-column
        prop="publishStatus"
        header-align="center"
        align="center"
        label="上架状态"
        ><template slot-scope="scope">
          <el-tag v-if="scope.row.publishStatus == 0">已下架</el-tag>
          <el-tag v-if="scope.row.publishStatus == 1">已上架</el-tag>
        </template></el-table-column
      >
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间"
      ></el-table-column>
      <el-table-column
        prop="updateTime"
        header-align="center"
        align="center"
        label="修改时间"
      ></el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作"
      >
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.publishStatus == 0"
            type="text"
            size="small"
            @click="productUp(scope.row.id)"
            >上架</el-button
          >
          <el-button
            v-if="scope.row.publishStatus == 1"
            type="text"
            size="small"
            @click="productDown(scope.row.id)"
            >下架</el-button
          >
          <el-button type="text" size="small" @click="attrUpdateShow(scope.row)"
            >规格</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    ></el-pagination>
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
    //这里存放数据
    return {
      dataListLoading: false,
      dataList: [],
      dataListSelections: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
    };
  },
  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    productUp(id) {
      console.log(id);
      this.updateSpuPublishStatus(id, 1);
    },
    productDown(id) {
      this.updateSpuPublishStatus(id, 0);
    },
    updateSpuPublishStatus(id, publishStatus) {
      this.$http({
        url: this.$http.adornUrl("/product/spuinfo/updatePublishStatus"),
        method: "post",
        data: this.$http.adornData({ id, publishStatus }, false),
      }).then(({ data }) => {
        if (data.code === 0) {
          this.$message({
            type: "success",
            message: "操作成功",
            duration: 1000,
            onClose: () => {
              this.getDataList();
            },
          });
        } else {
          this.$message.error(data.msg);
        }
      });
    },
    currentChangeHandle(page) {
      this.pageIndex = page;
      this.getDataList();
    },
    sizeChangeHandle(pageSize) {
      this.pageSize = pageSize;
      this.pageIndex = 1;
      this.getDataList();
    },
    attrUpdateShow(row) {
      console.log(row);
    },
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    getDataList() {
      let params = {};
      Object.assign(params, this.dataForm, {
        page: this.pageIndex,
        limit: this.pageSize,
      });
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/spuinfo/list"),
        method: "get",
        params: this.$http.adornParams(params),
      }).then(({ data }) => {
        if (data.code === 0) {
          this.dataList = data.page.list;
          this.totalPage = data.page.totalPage;
        } else {
          this.$message.error(data.msg);
        }
        this.dataListLoading = false;
      });
    },
  },
  //声明周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getDataList();
  },
  //声明周期 - 挂载完成（可以访问DOM元素）
  mounted() {
    this.dataSub = PubSub.subscribe("dataForm", (msg, val) => {
      this.dataForm = val;
      this.getDataList();
    });
  },
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {
    PubSub.unsubscribe(this.dataSub);
  }, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>