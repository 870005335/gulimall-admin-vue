<template>
  <div>
    <el-row>
      <el-col :span="24"
        ><el-form :inline="true" :model="dataForm">
          <el-form-item label="分类">
            <category-cascader
              :catelogPath.sync="catelogPath"
            ></category-cascader>
          </el-form-item>
          <el-form-item label="品牌">
            <brand-select style="width: 160px"></brand-select>
          </el-form-item>
          <el-form-item label="状态">
            <el-select style="width: 160px" v-model="dataForm.status" clearable>
              <el-option label="上架" :value="1"></el-option>
              <el-option label="下架" :value="0"></el-option
            ></el-select>
          </el-form-item>
          <el-form-item label="检索">
            <el-input
              style="width: 160px"
              v-model="dataForm.key"
              clearable
            ></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="searchSpuInfo">查询</el-button>
          </el-form-item>
        </el-form>
        <spuinfo :dataForm="dataForm"></spuinfo>
      </el-col>
    </el-row>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》'
import CategoryCascader from "../common/category-cascader.vue";
import BrandSelect from "../common/brand-select.vue";
import Spuinfo from "./spuinfo.vue";
import PubSub from "pubsub-js";

export default {
  //import引入的组件需要注入到对象中才能使用
  components: { CategoryCascader, BrandSelect, Spuinfo },
  props: {},
  data() {
    //这里存放数据
    return {
      dataForm: {
        brandId: 0,
        catelogId: 0,
      },
      catelogPath: [],
      catPathSub: {},
      brandIdSub: {},
    };
  },
  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    searchSpuInfo() {
      PubSub.publish("dataForm", this.dataForm);
    },
  },
  //声明周期 - 创建完成（可以访问当前this实例）
  created() {},
  //声明周期 - 挂载完成（可以访问DOM元素）
  mounted() {
    this.catPathSub = PubSub.subscribe("catPath", (msg, val) => {
      this.dataForm.catelogId = val[val.length - 1];
    });
    this.brandIdSub = PubSub.subscribe("brandId", (msg, val) => {
      this.dataForm.brandId = val;
    });
  },
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {
    PubSub.unsubscribe(this.catPathSub);
    PubSub.unsubscribe(this.brandIdSub);
  }, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>