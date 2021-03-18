<template>
  <div class="mod-config">
    <el-form
      :inline="true"
      :model="dataForm"
      @keyup.enter.native="getDataList()"
    >
      <el-form-item>
        <el-input
          v-model="dataForm.key"
          placeholder="参数名"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button
          v-if="isAuth('member:member:save')"
          type="primary"
          @click="addOrUpdateHandle()"
          >新增</el-button
        >
        <el-button
          v-if="isAuth('member:member:delete')"
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
          >批量删除</el-button
        >
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selction-change="selectionChangeHandle"
      style="width: 100%"
      fixed
    >
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        style="width: 50"
      ></el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="id"
      ></el-table-column>
      <el-table-column
        prop="levelId"
        header-align="center"
        align="center"
        label="会员等级id"
      ></el-table-column>
      <el-table-column
        prop="username"
        header-align="center"
        align="center"
        label="用户名"
      ></el-table-column>
      <el-table-column
        prop="nickname"
        header-align="center"
        align="center"
        label="昵称"
      ></el-table-column>
      <el-table-column
        prop="mobile"
        header-align="center"
        align="center"
        label="手机号"
      ></el-table-column>
      <el-table-column
        prop="email"
        header-align="center"
        align="center"
        label="邮箱"
      ></el-table-column>
      <el-table-column
        prop="header"
        header-align="center"
        align="center"
        label="头像"
        ><template slot-scope="scope">
          <el-image
            style="width: 100px; height: 100px"
            :src="scope.row.header"
            fit="fit"
          ></el-image> </template
      ></el-table-column>
      <el-table-column
        prop="gender"
        header-align="center"
        align="center"
        label="性别"
      >
        <template slot-scope="scope">
          <span v-if="scope.row.gender == 0">男</span>
          <span v-if="scope.row.gender == 1">女</span>
        </template></el-table-column
      >
      <el-table-column
        prop="birth"
        header-align="center"
        align="center"
        label="生日"
      ></el-table-column>
      <el-table-column
        prop="city"
        header-align="center"
        align="center"
        label="所在城市"
        ><template slot-scope="scope"
          ><el-cascader
            :options="provinceAndCityData"
            v-model="scope.row.city"
            disabled
          ></el-cascader
        ></template>
      </el-table-column>
      <el-table-column
        prop="job"
        header-align="center"
        align="center"
        label="职业"
      ></el-table-column>
      <el-table-column
        prop="sign"
        header-align="center"
        align="center"
        label="个性签名"
      ></el-table-column>
      <el-table-column
        prop="sourceType"
        header-align="center"
        align="center"
        label="用户来源"
      ></el-table-column>
      <el-table-column
        prop="integration"
        header-align="center"
        align="center"
        label="积分"
      ></el-table-column>
      <el-table-column
        prop="growth"
        header-align="center"
        align="center"
        label="成长值"
      ></el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="启用状态"
      >
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.status"
            active-color="#13ce66"
            inactive-color="#ff4949"
            :active-value="1"
            :inactive-value="0"
            @change="updateMemberStatus(scope.row)"
          ></el-switch></template></el-table-column
      ><el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="注册时间"
      ></el-table-column
      ><el-table-column
        fixed="right"
        header-align="center"
        align="center"
        label="操作"
        width="50"
      >
        <template slot-scope="scope">
          <el-button
            type="text"
            size="small"
            @click="addOrUpdateHandle(scope.row.id)"
            >编辑</el-button
          >
          <el-button
            type="text"
            size="small"
            @click="deleteHandle(scope.row.id)"
            >移除</el-button
          >
          <el-button type="text" size="small">送券</el-button>
          <el-button
            type="text"
            size="small"
            @click="getMemberOrderList(scope.row.id)"
            >查订单</el-button
          >
        </template></el-table-column
      ></el-table
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
    <member-add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdate"
      @refreshDataList="getDataList()"
    ></member-add-or-update>
  </div>
</template>

<script>
import MemberAddOrUpdate from "./member-add-or-update.vue";
import { provinceAndCityData } from "element-china-area-data";
// 这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
// 例如：import 《组件名称》 from '《组件路径》';

export default {
  // import引入的组件需要注入到对象中才能使用
  components: { MemberAddOrUpdate },
  data() {
    //这里存放数据
    return {
      provinceAndCityData: provinceAndCityData,
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
  // 监听属性 类似于data概念
  computed: {},
  // 监控data中的数据变化
  watch: {},
  // 方法集合
  methods: {
    updateMemberStatus(row) {
      console.log(row);
      let { id, status } = row;
      this.$http({
        url: this.$http.adornUrl("/member/member/update/status"),
        method: "post",
        data: this.$http.adornData({ id, status }, false),
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
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map((item) => {
            return item.id;
          });
      this.$confirm(
        `确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/member/member/delete"),
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
      });
    },
    sizeChangeHandle(pageSize) {
      this.pageSize = pageSize;
      this.pageIndex = 1;
      this.getDataList();
    },
    currentChangeHandle(page) {
      this.pageIndex = page;
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
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/member/member/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
        }),
      }).then(({ data }) => {
        if (data.code === 0) {
          this.dataList = data.page.list;
          this.dataList.forEach((item) => {
            item.city = item.city.split(",");
          });
          this.totalPage = data.page.totalPage;
        } else {
          this.$message.error(data.msg);
        }
        this.dataListLoading = false;
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
