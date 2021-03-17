<template>
  <div>
    <el-row>
      <el-col :span="24">
        <el-steps :active="step" finish-status="success">
          <el-step title="基本信息"></el-step>
          <el-step title="规格参数"></el-step>
          <el-step title="销售属性"></el-step>
          <el-step title="SKU信息"></el-step>
          <el-step title="保存完成"></el-step>
        </el-steps>
      </el-col>
      <el-col :span="24" v-show="step == 0"
        ><el-card class="box-card" style="width: 80%; margin: 20px auto">
          <el-form
            ref="spuBaseForm"
            :model="spu"
            label-width="120px"
            :rules="spuBaseInfoRules"
          >
            <el-form-item prop="spuName" label="商品名称">
              <el-input v-model="spu.spuName"></el-input>
            </el-form-item>
            <el-form-item prop="spuDescription" label="商品描述">
              <el-input v-model="spu.spuDescription"></el-input>
            </el-form-item>
            <el-form-item prop="catelogId" label="选择分类">
              <category-cascader></category-cascader>
            </el-form-item>
            <el-form-item prop="brandId" label="选择品牌">
              <brand-select></brand-select>
            </el-form-item>
            <el-form-item prop="weight" label="商品重量(Kg)">
              <el-input-number
                v-model.number="spu.weight"
                :min="0"
                :precision="3"
                :step="0.1"
              ></el-input-number>
            </el-form-item>
            <el-form-item label="设置积分" prop="bounds">
              <label>金币</label>
              <el-input-number
                style="width: 130px"
                placeholder="金币"
                v-model="spu.bounds.buyBounds"
                :min="0"
                controls-position="right"
              ></el-input-number>
              <label style="margin-left: 15px">成长值</label>
              <el-input-number
                style="width: 130px"
                placeholder="成长值"
                v-model="spu.bounds.growBounds"
                :min="0"
                controls-position="right"
              >
                <template slot="prepend">成长值</template>
              </el-input-number>
            </el-form-item>
            <el-form-item prop="decript" label="商品介绍">
                <multi-upload v-model="spu.decript"></multi-upload>
            </el-form-item>
            <el-form-item prop="images" label="商品图集">
                <multi-upload v-model="spu.images"></multi-upload>
            </el-form-item>
            <el-form-item>
                <el-button type="success" @click="collectSpuBaseInfo">下一步：设置基本参数</el-button>
            </el-form-item>
          </el-form>
        </el-card></el-col
      >
    </el-row>
  </div>
</template>

<script>
import MultiUpload from '@/components/upload/multiUpload.vue';
import BrandSelect from "../common/brand-select.vue";
import CategoryCascader from "../common/category-cascader.vue";
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》'

export default {
  //import引入的组件需要注入到对象中才能使用
  components: { CategoryCascader, BrandSelect, MultiUpload },
  props: {},
  data() {
    //这里存放数据
    return {
      catPathSub: {},
      brandIdSub: {},
      step: 0,
      spu: {
        catelogId: 0,
        brandId: 0,
        weight: 0,
        bounds: {

        }
      },
      spuBaseInfoRules: {},
    };
  },
  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
      collectSpuBaseInfo() {

      }
  },
  //声明周期 - 创建完成（可以访问当前this实例）
  created() {},
  //声明周期 - 挂载完成（可以访问DOM元素）
  mounted() {
    this.catPathSub = PubSub.subscribe("catPath", (msg, val) => {
      if (val.length === 3) {
        this.spu.catelogId = val[val.length - 1];
      } else {
        this.spu.catelogId = 0;
      }
    });
    this.brandIdSub = PubSub.subscribe("brandId", (msg, val) => {
      this.spu.brandId = val;
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