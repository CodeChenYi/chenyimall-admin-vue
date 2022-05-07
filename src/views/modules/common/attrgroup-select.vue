<template>
  <div>
    <el-select
      v-model="attrGroup"
      placeholder="请先选择所属分类"
      clearable
      filterable
    >
      <el-option
        v-for="item in attrGroupList"
        :key="item.attrGroupId"
        :label="item.attrGroupName"
        :value="item.attrGroupId"
      >
      </el-option>
    </el-select>
  </div>
</template>

<script>
export default {
    props: {
    categoryList: {
      type: Array,
      default() {
        return [];
      },
    },
    attrGroupList: {
      type: Array,
      default() {
        return [];
      },
    }
  },
  data() {
    return {
      attrGroup: '',
      attrGroupList: this.attrGroupList,
      catelogId: this.categoryList,
    };
  },
  created() {},
  methods: {
    getAttrGroup() {
      this.$http({
        url: this.$http.adornUrl(`/product/attrgroup/list/${this.catelogId}`),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        this.attrGroupList = data.page.list;
      });
    },
  },
};
</script>