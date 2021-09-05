<template>
  <div>
    <el-tree
      :data="data"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level < 3"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="商品分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      data: [],
      category: { name: "", parentCid: 0, catLevel: 0, showStatus: 1, sort: 0 },
      expandedKey: [],
      dialogVisible: false,
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
      }).then(({ data }) => {
        this.data = data.list;
      });
    },

    append(data) {
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.showStatus = 1;
      this.category.sort = 0;
      this.dialogVisible = true;
    },
    addCategory() {
      //添加商品分类
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      })
        .then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: "添加成功",
              type: "success",
            });
            this.dialogVisible = false;
            //刷新菜單
            this.getMenus();
            //展開父節點
            this.expandedKey = [this.category.parentCid];
          } else {
            this.$message.error(data.msg);
          }
        })
        .catch(() => {});
    },
    remove(node, data1) {
      let ids = [data1.catId];
      this.$confirm(`是否刪除${data1.name}?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl("/product/category/delete"),
          method: "post",
          data: this.$http.adornData(ids, false),
        })
          .then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "刪除成功",
                type: "success",
              });
              //刷新菜單
              this.getMenus();
              //展開父節點
              this.expandedKey = [node.parent.data.catId];
            } else {
              this.$message.error(data.msg);
            }
          })
          .catch(() => {});
      });
    },
  },
};
</script>

<style>
</style>