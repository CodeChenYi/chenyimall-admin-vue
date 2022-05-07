<template>
  <div>
    <el-input
      v-model="treeName"
      placeholder="输入关键字查询"
      clearable
    ></el-input>
    <el-tree
      ref="categoryList"
      :data="categoryList"
      :props="defaultProps"
      node-key="catId"
      @node-click="treeNodeClick"
      :filter-node-method="filterNode"
    >
    </el-tree>
  </div>
</template>

<script>
export default {
  data() {
    return {
      treeName: "",
      categoryList: [],
      defaultProps: {
        children: "categoryEntityChildrenList",
        label: "name",
      },
    };
  },
  created() {
    this.getMenu();
  },
  watch: {
    // 监听当前输入框
    treeName(treeName) {
      if (treeName == "") {
        console.log(treeName);
        this.getMenu();
      } else {
        this.$refs.categoryList.filter(treeName);
      }
    },
  },
  methods: {
    getMenu() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then((data) => {
        this.categoryList = data.data.data;
        console.log(this.categoryList);
      });
    },

    treeNodeClick(data, node, components) {
      console.log("当前节点被点击", data, node, components);
      this.$emit("tree-node-click", data, node, components);
    },
    // tree模糊查询功能
    filterNode(value, data, node) {
      if (!value) {
        return true;
      }
      let level = node.level;
      let _array = []; //这里使用数组存储 只是为了存储值。
      this.getReturnNode(node, _array, value);
      let result = false;
      _array.forEach((item) => {
        result = result || item;
      });
      return result;
    },
    getReturnNode(node, _array, value) {
      let isPass =
        node.data && node.data.name && node.data.name.indexOf(value) !== -1;
      isPass ? _array.push(isPass) : "";
      this.index++;
      if (!isPass && node.level != 1 && node.parent) {
        this.getReturnNode(node.parent, _array, value);
      }
    },
  },
};
</script>

<style>
</style>