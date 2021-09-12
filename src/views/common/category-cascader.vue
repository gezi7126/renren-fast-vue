<template>
<!-- 
使用说明：
1）、引入category-cascader.vue
2）、语法：<category-cascader :catelogPath.sync="catelogPath"></category-cascader>
    解释：
      catelogPath：指定的值是cascader初始化需要显示的值，应该和父组件的catelogPath绑定;
          由于有sync修饰符，所以cascader路径变化以后自动会修改父的catelogPath，这是结合子组件this.$emit("update:catelogPath",v);做的
      -->
  <div>
   
    <el-cascader
      filterable
      clearable 
      placeholder="试试搜索：手机"
      v-model="paths"
      :options="categorys"
      :props="setting"
      @change="changeNodes"
      ref="elCascader"
    ></el-cascader>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //接受父组件传来的值
 // props:['catelogPath'],
   props: {
    catelogPath: {
      type: Array,
      default(){
        return [];
      }
    }
  },

  data() {
    //这里存放数据
    return {
      setting: {
        value: "catId",
        label: "name",
        children: "childCategorys"
      },
      categorys: [],
      paths: this.catelogPath
    };
  },
  

   watch:{
    catelogPath(){
      this.paths = this.catelogPath;
    },
  /* paths(value){
      let v=this.$refs.elCascader.checkedValue;
      alert("///////",v);
      this.$emit("listenChildNodesPath",v);
      //还可以使用pubsub-js进行传值
      //this.PubSub.publish("catPath",v);
    }*/
  },
  //方法集合
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/treelist"),
        method: "get",
      }).then(({ data }) => {
        this.categorys = data.list;
      });
    },
    changeNodes(){
       let nodes=this.$refs.elCascader.checkedValue;
       alert("nodes....",nodes);
       this.paths=nodes;
       this.$emit("listenChildNodesPath",nodes);
       
    }
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMenus();
  }
};
</script>
<style >
</style>