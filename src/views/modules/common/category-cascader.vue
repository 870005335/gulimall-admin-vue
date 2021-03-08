<!--  -->
<template>
  <div>
    <el-cascader
      filterable
      clearable
      placeholder="试试搜索：手机"
      v-model="paths"
      :options="categorys"
      :props="setting"
    ></el-cascader>
  </div>
</template>

<script>
// 这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
// 例如：import 《组件名称》 from '《组件路径》';

export default {
  props: {
    catelogPath: {
      type: Array,
      default() {
        return [];
      },
    },
  },
  // import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    //这里存放数据
    return {
      categorys: [],
      paths: this.catelogPath,
      setting: {
        value: "catId",
        label: "name",
        children: "children",
      },
    };
  },
  // 监听属性 类似于data概念
  computed: {},
  // 监控data中的数据变化
  watch: {
    catelogPath(val) {
      this.paths = val;
    },
    paths(val) {
      this.$emit("update:catelogPath", val);
    },
  },
  // 方法集合
  methods: {
    getCategorys() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then((data) => {
        if (data.data.code === 0) {
          this.categorys = data.data.trees;
        }
      });
    },
  },
  // 生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getCategorys();
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
