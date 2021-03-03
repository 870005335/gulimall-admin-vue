<template>
  <el-tree
    :data="menu"
    :props="defaultProps"
    @node-click="handleNodeClick"
  ></el-tree>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      menu: [],
      defaultProps: {
          children: 'children',
          label: 'name'
        }
    };
  },
  computed: {},
  watch: {},
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: 'get'
      }).then((data) => {
        if (data.data.code === 0) {
          this.menu = data.data.trees;
        }
      }).catch((err) => {
        
      });
    }
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