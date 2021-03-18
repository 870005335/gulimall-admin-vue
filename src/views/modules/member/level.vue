<template>
  <div class="mod-config">
    <el-form
      :inline="true"
      :model="dataForm"
      @keyup.enter.native="getDataList()"
    >
      <el-form-item
        ><el-input
          v-model="dataForm.key"
          placeholder="参数名"
          clearable
        ></el-input
      ></el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button
          v-if="isAuth('member:memberlevel:save')"
          type="primary"
          @click="addOrUpdateHandle()"
          >新增</el-button
        >
        <el-button
          v-if="isAuth('member:memberlevel:delete')"
          type="danger"
          @click="DeleteHandle()"
          :disabled="dataListSelections.length <= 0"
          >批量删除</el-button
        >
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%"
      fixed
      ><el-table-column
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
        prop="name"
        header-align="center"
        align="center"
        label="等级名称"
      ></el-table-column>
      <el-table-column
        prop="growthPoint"
        header-align="center"
        align="center"
        label="所需成长值"
      ></el-table-column
      ><el-table-column
        prop="defaultStatus"
        header-align="center"
        align="center"
        label="默认等级"
      >
        <template slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.defaultStatus == 1"></i>
          <i class="el-icon-error" v-else></i>
        </template>
      </el-table-column>
      <el-table-column
        prop="freeFreightPoint"
        header-align="center"
        align="center"
        label="免运费标准"
      ></el-table-column>
      <el-table-column
        prop="commentGrowthPoint"
        header-align="center"
        align="center"
        label="每次评价获取的成长值"
      ></el-table-column>
      <el-table-column label="特权" align="center">
        <el-table-column
          prop="priviledgeFreeFreight"
          header-align="center"
          align="center"
          label="免邮特权"
        >
          <template slot-scope="scope">
            <i
              class="el-icon-success"
              v-if="scope.row.priviledgeFreeFreight == 1"
            ></i>
            <i class="el-icon-error" v-else></i>
          </template>
        </el-table-column>
        <el-table-column
          prop="priviledgeMemberPrice"
          header-align="center"
          align="center"
          label="会员价格特权"
        >
          <template slot-scope="scope">
            <i
              class="el-icon-success"
              v-if="scope.row.priviledgeMemberPrice == 1"
            ></i>
            <i class="el-icon-error" v-else></i>
          </template>
        </el-table-column>
        <el-table-column
          prop="priviledgeBirthday"
          header-align="center"
          align="center"
          label="生日特权"
        >
          <template slot-scope="scope">
            <i
              class="el-icon-success"
              v-if="scope.row.priviledgeBirthday == 1"
            ></i>
            <i class="el-icon-error" v-else></i>
          </template>
        </el-table-column>
      </el-table-column>
      <el-table-column
        prop="note"
        header-align="center"
        align="center"
        label="备注"
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
            type="text"
            size="small"
            @click="addOrUpdateHandle(scope.row.id)"
            >修改</el-button
          >
          <el-button
            type="text"
            size="small"
            @click="deleteHandle(scope.row.id)"
            >删除</el-button
          >
        </template>
      </el-table-column></el-table
    >
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    ></el-pagination>
    <memberlevel-add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdate"
      @refreshDataList="getDataList()"
    ></memberlevel-add-or-update>
  </div>
</template>

<script>
import MemberlevelAddOrUpdate from "./memberlevel-add-or-update.vue";
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》'

export default {
  //import引入的组件需要注入到对象中才能使用
  components: { MemberlevelAddOrUpdate },
  props: {},
  data() {
    //这里存放数据
    return {
      dataForm: {
        key: "",
      },
      dataList: [],
      dataListSelections: [],
      dataListLoading: false,
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      addOrUpdateVisible: false,
    };
  },
  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    currentChangeHandle(page) {
      this.pageIndex = page;
      this.getDataList();
    },
    sizeChangeHandle(pageSize) {
      this.pageSize = pageSize;
      this.pageIndex = 1;
      this.getDataList();
    },
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    },
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map((item) => {
            return item.id;
          });
    },
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/member/memberlevel/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
        }),
      }).then(({ data }) => {
        if (data.code === 0) {
          this.dataList = data.page.list;
          this.totalPage = data.page.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
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