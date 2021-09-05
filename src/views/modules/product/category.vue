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
            type="text"
            size="mini"
            @click="() => edit(data)"
          >
            Edit
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

    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="商品分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" :v-model="method"  @click="handleCategory()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      data: [],
      title:"",
      method:2,
      category: { name: "", parentCid: 0, catLevel: 0, showStatus: 1, sort: 0,icon:"", productUnit:""},
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

    edit(data1){
      this.dialogVisible = true;
      this.title="修改商品分類";
      this.method=1;
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data1.catId}`),
        method: "get",
      }).then(({ data }) => {
        this.category.catId=data.category.catId;
        this.category.name=data.category.name;
        this.category.icon=data.category.icon;
        this.category.productUnit= data.category.productUnit;
        this.category.parentCid=data.category.parentCid;
      });
    },
    append(data) {
      this.dialogVisible = true;
      this.title="添加商品分類";
      this.method=0;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.showStatus = 1;
      this.category.sort = 0;

      //清空input 
       this.category.catId=null;
        this.category.name="";
        this.category.icon="";
        this.category.productUnit= "";
    },

    handleCategory(){
        if(this.method === 0)
           this.addCategory();
        else
           this.editCategory();
    },

    editCategory(){
      //结构赋值，只保存修改结构出来的字段
       let {catId,name,icon,productUnit} =this.category;
       this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId,name,icon,productUnit }, false),
      })
        .then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: "修改成功",
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