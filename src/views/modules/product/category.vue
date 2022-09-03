<!--  -->
<template>
  <el-tree
    :data="menus"
    :props="defaultProps"
    :default-expanded-keys="expandedKey"
    show-checkbox
    node-key="catId"
    :expand-on-click-node="false"
    @node-click="handleNodeClick"
  >
    <span
      class="custom-tree-node"
      slot-scope="{ node, data }"
    >
      <span>{{ node.label }}</span>
      <span>
        <el-button
          v-if="node.level<=2"
          type="text"
          size="mini"
          @click="() => append(data)"
        >
          Append
          </el-button>
          <el-button
            v-if="node.childNodes.length==0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
            </el-button>
      </span>
      </span>
  </el-tree>
</template>

<script>
// 这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    return {
      menus: [],
      expandedKey:[],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  methods: {
    handleNodeClick(data) {
      console.log(data);
    },
    // 获取数据列表
    getMenus() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get"
      }).then(({ data }) => {
        this.menus = data.page;
        console.log("成功！！！", data.page);
      });
    },
    append(data) {
      const newChild = { id: id++, label: "testtest", children: [] };
      if (!data.children) {
        this.$set(data, "children", []);
      }
      data.children.push(newChild);
    },

    remove(node, data) {
      var ids = [data.catId];

      this.$confirm(`是否删除【${data.name}】`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl("/product/category/delete"),
          method: "post",
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          console.log("删除成功！！！");
          this.$message({
            message: `删除成功  `,
            type: "success"
          });
          this.getMenus();
          //测试11
          this.expandedKey=[node.parent.data.catId]
        });
      });
      // console.log("remove", node, data);
      // const parent = node.parent;
      // const children = parent.data.children || parent.data;
      // const index = children.findIndex(d => d.id === data.id);
      // children.splice(index, 1);
    }
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},

  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMenus();
    console.log("获取数据成功！");
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {} //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
