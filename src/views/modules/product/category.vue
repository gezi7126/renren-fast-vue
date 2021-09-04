<template>
  <el-tree :data="data" :props="defaultProps"  
    :expand-on-click-node="false" show-checkbox node-key="catId"
    :default-expanded-keys="expandedKey">
    <span class="custom-tree-node" slot-scope="{ node, data }">
      <span>{{ node.label }}</span>
      <span>
        <el-button v-if="node.level <3" type="text" size="mini" @click="() => append(data)">
          Append
        </el-button>
        <el-button v-if="node.childNodes.length==0" type="text" size="mini" @click="() => remove(node, data)">
          Delete
        </el-button>
      </span>
    </span>
  </el-tree>
</template>

<script>
export default {
  data() {
    return {
      data: [],
      expandedKey:[],
      defaultProps: {
        children: "childCategorys",
        label: "name",
      },
    };
  },

  created() {
    this.getMenus();
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/treelist"),
        method: "get",
      }).then(({data}) => {
        this.data = data.list;
      });
    },

    append(data){
       console.log(data);
    },
    remove(node,data1){
        let ids=[data1.catId];
        this.$confirm(`是否刪除${data1.name}?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
            this.$http({
            url: this.$http.adornUrl('/product/category/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '刪除成功',
                type: 'success'
              })
              //刷新菜單
              this.getMenus();
              //展開父節點
              this.expandedKey=[node.parent.data.catId];
            } else {
              this.$message.error(data.msg);

            }
          }).catch(() => { });

        });
       
    }

  },
};
</script>

<style>
</style>