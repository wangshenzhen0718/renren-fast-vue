<!--  -->
<template>
  <div>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :default-expanded-keys="expandedKey"
      show-checkbox
      node-key="catId"
      :expand-on-click-node="false"
      @node-click="handleNodeClick"
      ref="menuTree"

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
          type="text"
          size="mini"
          @click="() => edit(data)"
        >
          编辑
          </el-button>
          <el-button
            v-if="node.childNodes.length==0"
            type="text"
            size="mini"
            @click="() => remove(node,data)"
          >
            Delete
            </el-button>
      </span>
      </span>
    </el-tree>
    <el-dialog
      :title="dialogTitle"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false">
      <el-form ref="form" :model="catagory">
        <el-form-item label="分类名称">
          <el-input v-model="catagory.name"></el-input>
        </el-form-item>
        <el-form-item label="分类图标">
          <el-input v-model="catagory.icon"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="catagory.productUnit"></el-input>
        </el-form-item>

      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="saveOrEdit">确 定</el-button>
  </span>
    </el-dialog>
  </div>

</template>

<script>
// 这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    return {
      dialogTitle: "",
      dialogType: "",
      menus: [],
      dialogVisible: false,
      expandedKey: [],
      form: {
        name: '',

      },
      catagory: {
        name: "",
        catId: null,
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: "",
        productUnit: ""
      },
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  methods: {
    batchDelete() {
      let catIds = [];
      let catNames = [];
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      for (let i = 0; i < checkedNodes.length; i++) {
        catIds.push(checkedNodes[i].catId)
      }
      for (let i = 0; i < checkedNodes.length; i++) {
        catNames.push(checkedNodes[i].name)
      }
      this.$confirm(`是否批量删除【${catNames}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false)
          }).then(({data}) => {
            this.$message({
              message: "菜单批量删除成功",
              type: "success"
            });
            this.getMenus();
          });
        })
        .catch(() => {
        });
      console.log("被选中的节点", checkedNodes)
    },
    handleNodeClick(data) {
      console.log(data);
    },
    // 获取数据列表
    getMenus() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get"
      }).then(({data}) => {
        this.menus = data.data;
        console.log("成功！！！", data.data);
      });
    },
    append(data) {
      this.catagory.name = ""
      this.catagory.catId = null
      this.catagory.sort = 0
      this.catagory.icon = null
      this.catagory.productUnit = ""
      this.catagory.showStatus = 1
      this.dialogTitle = "添加"
      this.dialogType = "add"
      console.log(data)
      this.dialogVisible = true
      this.catagory.parentCid = data.catId;
      this.catagory.catLevel = data.catLevel * 1 + 1
      // const newChild = {id: id++, label: "testtest", children: []};
      // if (!data.children) {
      //   this.$set(data, "children", []);
      // }
      // data.children.push(newChild);
    },
    edit(data) {
      this.dialogTitle = "修改"
      this.dialogType = "edit"
      this.dialogVisible = true
      this.catagory.name = data.name
      this.catagory.catId = data.catId
      this.catagory.parentCid = data.parentCid

      this.getCategory(data)

      console.log(this.dialogType)
      console.log(data.catId + "--------------------id")
    },
    getCategory(data) {
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get',
      }).then(({data}) => {
        console.log("要回显的数据" + data)
        this.catagory.icon = data.data.icon
        this.catagory.productUnit = data.data.productUnit

      })
    },
    saveOrEdit() {
      if (this.dialogType == "save") {
        this.addCategory()
      }
      if (this.dialogType == "edit") {
        this.editCategory()
      }
      this.dialogVisible = false
    },
    addCategory() {
      console.log("提交的三级分类" + this.catagory.catLevel);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.catagory, false)
      }).then(({data}) => {
        console.log("添加成功！！！");
        this.$message({
          message: `添加成功  `,
          type: "success"
        });

      });
      this.dialogVisible = false
      this.getMenus();
      this.expandedKey = [this.catagory.parentCid]
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
        }).then(({data}) => {
          console.log("删除成功！！！");
          this.$message({
            message: `删除成功  `,
            type: "success"
          });
          this.getMenus();
          //测试11
          this.expandedKey = [node.parent.data.catId]
        });
      }).catch(() => {
        //防止前台红色错误
      });
      // console.log("remove", node, data);
      // const parent = node.parent;
      // const children = parent.data.children || parent.data;
      // const index = children.findIndex(d => d.id === data.id);
      // children.splice(index, 1);
    },
    editCategory() {
      var {catId, name, icon, productUnit} = this.catagory
      var data = {catId: catId, name: name, icon: icon, productUnit: productUnit}
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(data, false)
      }).then(({data}) => {
        console.log("更新成功！！！");
        this.$message({
          message: `更新成功  `,
          type: "success"
        });
        this.getMenus();
        //测试11
        this.expandedKey = [this.catagory.parentCid]
      });

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
  mounted() {
  },
  beforeCreate() {
  }, //生命周期 - 创建之前
  beforeMount() {
  }, //生命周期 - 挂载之前
  beforeUpdate() {
  }, //生命周期 - 更新之前
  updated() {
  }, //生命周期 - 更新之后
  beforeDestroy() {
  }, //生命周期 - 销毁之前
  destroyed() {
  }, //生命周期 - 销毁完成
  activated() {
  } //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
