<!--  -->
<template>
  <div>
    <el-tree
      :data="menus"
      show-checkbox
      :props="defaultProps"
      @node-click="handleNodeClick"
      :expand-on-click-node="false"
      node-key="catId"
      :default-expanded-keys="expandedKey"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="text"
            v-if="node.level <= 2"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <el-button
            type="text"
            v-if="node.childNodes.length == 0"
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
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    return {
      category: { name: "", parentCid: 0, catLevel: 0, showStatus: 1, sort: 0 },
      dialogVisible: false,
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: "children", //子节点
        label: "name", //name属性作为标签的值，展示出来
      },
    };
  },
  methods: {
    handleNodeClick(data) {},
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("成功了获取到菜单数据....", data.data);
        this.menus = data.data;
      });
    },
    append(data) {
      console.log("append", data);
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },
    // 添加三级分类
    addCategory() {
      console.log("提交的三级分类数据", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        console.log("菜单保存成功");
      });
      this.dialogVisible = false;
      // 刷新出新的菜单
      this.getMenus();
      // 设置需要展开的菜单
      this.expandedKey = [this.category.parentCid];
    },
    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`是否删除当前【${data.name}】菜单, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            this.$message({
              message: "菜单删除成功",
              type: "success",
            });
            // 刷新出新的菜单
            this.getMenus();
            // 设置需要展开的菜单
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {
          console.log("取消删除");
        });

      console.log("remove", node, data);
    },
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMenus();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>
