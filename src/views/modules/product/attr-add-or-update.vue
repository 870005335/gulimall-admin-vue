<template>
  <div>
    <el-dialog
      :title="dataForm.id ? '编辑' : '新增'"
      :visible.sync="visible"
      @closed="handleClose"
      :close-on-click-modal="false"
    >
      <el-form :rules="dataRule" :model="dataForm" ref="dataForm" label-width="120px">
        <el-form-item prop="attrName" label="属性名">
          <el-input v-model="dataForm.attrName" placeholder="属性名"></el-input>
        </el-form-item>
        <el-form-item prop="attrType" label="属性类型">
          <el-select v-model="dataForm.attrType" placeholder="请选择" disabled>
            <el-option label="规格参数" :value="1"></el-option>
            <el-option label="销售属性" :value="0"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item prop="valueType" label="值类型">
          <el-switch
            v-model="dataForm.valueType"
            active-text="允许多个值"
            inactive-text="只能单个值"
            active-color="#13ce66"
            inactive-color="#ff4949"
            :active-value="1"
            :inactive-value="0"
          >
          </el-switch>
        </el-form-item>
        <el-form-item prop="valueSelect" label="可选值">
          <el-select
            v-model="dataForm.valueSelect"
            multiple
            filterable
            allow-create
            placeholder="请输入内容"
          ></el-select>
        </el-form-item>
        <el-form-item prop="icon" label="属性图标">
          <el-input v-model="dataForm.icon" placeholder="属性图标"></el-input>
        </el-form-item>
        <el-form-item prop="catelogId" label="所属分类">
          <category-cascader :catelogPath.sync="catelogPath"></category-cascader>
        </el-form-item>
        <el-form-item prop="attrGroupId" label="所属分组" v-if="type == 1">
          <el-select
            ref="groupSelect"
            v-model="dataForm.attrGroupId"
            placeholder="请选择"
          >
            <el-option
              v-for="item in attrGroups"
              :key="item.attrGroupId"
              :label="item.attrGroupName"
              :value="item.attrGroupId"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item prop="searchType" label="可检索" v-if="type == 1">
          <el-switch
            v-model="dataForm.searchType"
            active-color="#13ce66"
            inactive-color="#ff4949"
            :active-value="1"
            :inactive-value="0"
          >
          </el-switch>
        </el-form-item>
        <el-form-item prop="showDesc" label="快速展示" v-if="type == 1">
          <el-switch
            v-model="dataForm.showDesc"
            active-color="#13ce66"
            inactive-color="#ff4949"
            :active-value="1"
            :inactive-value="0"
          >
          </el-switch>
        </el-form-item>
        <el-form-item label="启用状态" prop="enable">
          <el-switch
            v-model="dataForm.enable"
            active-color="#13ce66"
            inactive-color="#ff4949"
            :active-value="1"
            :inactive-value="0"
          ></el-switch>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="visible = false">取 消</el-button>
        <el-button type="primary" @click="dataFormSubmit()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import CategoryCascader from "../common/category-cascader.vue";
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》'

export default {
  //import引入的组件需要注入到对象中才能使用
  components: { CategoryCascader },
  props: {
    type: Number,
    default() {
      return 1;
    },
  },
  data() {
    //这里存放数据
    return {
      visible: false,
      dataForm: {
        attrName: "",
        icon: "",
        attrGroupId: "",
        valueSelect: "",
        attrType: "",
        valueType: 1,
        searchType: 1,
        showDesc: 1,
        enable: 1,
      },
      attrGroups: [],
      catelogPath: [],
      dataRule: {
        attrName: [{ required: true, message: "属性名不能为空", trigger: "blur" }],
        searchType: [
          {
            required: true,
            message: "是否需要检索不能为空",
            trigger: "blur",
          },
        ],
        valueType: [
          {
            required: true,
            message: "值类型不能为空",
            trigger: "blur",
          },
        ],
        icon: [{ required: true, message: "属性图标不能为空", trigger: "blur" }],
        attrType: [
          {
            required: true,
            message: "属性类型不能为空",
            trigger: "blur",
          },
        ],
        enable: [
          {
            required: true,
            message: "启用状态不能为空",
            trigger: "blur",
          },
        ],
        catelogPath: [
          {
            required: true,
            message: "需要选择正确的三级分类数据",
            trigger: "blur",
          },
        ],
        showDesc: [
          {
            required: true,
            message: "快速展示不能为空",
            trigger: "blur",
          },
        ],
      },
    };
  },
  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {
    catelogPath(path) {
      this.dataForm.catelogId = path[path.length - 1];
      if (path && path.length == 3) {
        this.$http({
          url: this.$http.adornUrl(`/product/attrgroup/list/${this.dataForm.catelogId}`),
          method: "get",
          params: this.$http.adornParams({ page: 1, limit: 1000 }),
        }).then(({ data }) => {
          if (data.code === 0) {
            this.attrGroups = data.page.list;
          } else {
            this.$message.error(data.msg);
          }
        });
      } else if (path.length == 0) {
        this.dataForm.catelogId = "";
      } else {
        this.$message.error("请选择正确的分类");
        this.dataForm.catelogId = "";
      }
    },
  },
  //方法集合
  methods: {
    init(id) {
      this.visible = true;
      this.dataForm.attrType = this.type;
      this.dataForm.attrId = id || 0;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.attrId) {
          this.$http({
            url: this.$http.adornUrl(`/product/attr/info/${this.dataForm.attrId}`),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data.code === 0) {
              this.dataForm = data.attr;
              this.dataForm.valueSelect = data.attr.valueSelect.split(";");
              this.catelogPath = data.attr.catelogPath;
              this.$nextTick(() => {
                this.dataForm.attrGroupId = data.attr.attrGroupId;
              });
            }
          });
        }
      });
    },
    handleClose() {
      this.catelogPath = [];
    },
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.dataForm.valueSelect = this.dataForm.valueSelect.join(";");
          this.$http({
            url: this.$http.adornUrl(
              `/product/attr/${this.dataForm.attrId == 0 ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData(this.dataForm),
          }).then(({ data }) => {
            if (data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1000,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                },
              });
            } else {
              this.message.error(data.msg);
            }
          });
        }
      });
    },
  },
  //声明周期 - 创建完成（可以访问当前this实例）
  created() {},
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
<style scoped></style>
