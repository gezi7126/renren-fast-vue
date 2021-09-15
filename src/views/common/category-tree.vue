<template>
<!--商品管理-》平台属性-》属性分组页面等要引入的左侧商品三级分类菜单树-->
    <el-tree
      :data="data"
      :props="defaultProps"
      node-key="catId"
      :default-expanded-keys="expandedKey"
      @node-click="nodeClick"
    >
    </el-tree>
</template>

<script>
export default {
 data() {
    return {
      data: [],
      expandedKey: [],
      defaultProps: {
        children: "childCategorys",
        label: "name",
      },
    };
  },

  created() {
    this.getMenus();
  },

  methods:{
      getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/treelist"),
        method: "get",
      }).then(({ data }) => {
        this.data = data.list;
      });
    },
     nodeClick(data,node,component){
         console.log(data,node,component);
         this.$emit('getClickNode',data,node,component);
      }
  }
}
</script>

<style>

</style>