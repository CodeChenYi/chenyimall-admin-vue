<template>
  <div class="mod-config">
    <el-form
      :inline="true"
      :model="dataForm"
      @keyup.enter.native="getDataList()"
    >
      <el-form-item>
        <el-input
          v-model="dataForm.key"
          placeholder="参数名"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
          >批量删除</el-button
        >
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%"
    >
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50"
      >
      </el-table-column>
      <el-table-column
        prop="brandId"
        header-align="center"
        align="center"
        label="品牌id"
      >
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="品牌名"
      >
      </el-table-column>
      <el-table-column
        prop="logo"
        header-align="center"
        align="center"
        label="品牌logo地址"
        width="250"
      >
        <template slot-scope="scope">
          <el-image :src="scope.row.logo" :fit="fit"></el-image>
        </template>
      </el-table-column>
      <el-table-column
        prop="descript"
        header-align="center"
        align="center"
        label="介绍"
      >
      </el-table-column>
      <el-table-column
        prop="showStatus"
        header-align="center"
        align="center"
        label="显示状态"
        width="100"
      >
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.showStatus"
            :active-value="'1'"
            :inactive-value="'0'"
            @change="updateShowStatus(scope.row.showStatus, scope.row.brandId)"
          >
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column
        prop="firstLetter"
        header-align="center"
        align="center"
        label="检索首字母"
        width="100"
      >
      </el-table-column>
      <el-table-column
        prop="sort"
        header-align="center"
        align="center"
        label="排序"
        width="100"
      >
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="240"
        label="操作"
      >
        <template slot-scope="scope">
          <el-button
            type="text"
            size="small"
            @click="addCategoryBrand(scope.row)"
          >
            关联分类</el-button
          >
          <el-button
            type="text"
            size="small"
            @click="addOrUpdateHandle(scope.row.brandId)"
            >修改</el-button
          >
          <el-button
            type="text"
            size="small"
            @click="deleteHandle(scope.row.brandId)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    >
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdate"
      @refreshDataList="getDataList"
    ></add-or-update>
    <el-dialog title="添加关联分类" :visible.sync="addCategoryBrandVisible">
      <el-popover placement="right-end" v-model="popCatelogSelectVisible">
        <category-cascader
          :categoryList.sync="categoryList"
        ></category-cascader>
        <div style="text-align: right; margin: 0">
          <el-button
            size="mini"
            type="text"
            @click="popCatelogSelectVisible = false"
            >取消</el-button
          >
          <el-button type="primary" size="mini" @click="addCategoryBrandData"
            >确定</el-button
          >
        </div>
        <el-button slot="reference">新增关联</el-button>
      </el-popover>
      <el-table :data="categoryBrandDataList">
        <el-table-column
          property="brandName"
          label="品牌名称"
        ></el-table-column>
        <el-table-column
          property="catelogName"
          label="分类名称"
        ></el-table-column>
        <el-table-column property="address" label="操作" width="150"
          ><template slot-scope="scope">
            <el-button
              type="text"
              size="small"
              @click="removeCategoryBrandData(scope.row.id)"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>

<script>
import CategoryCascader from "../common/category-cascader.vue";
import AddOrUpdate from "./brand-add-or-update";
export default {
  data() {
    return {
      dataForm: {
        key: "",
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      brandId: "",
      brandName: "",
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      addCategoryBrandVisible: false,
      categoryList: [],
      popCatelogSelectVisible: false,
      categoryBrandDataList: [],
    };
  },
  components: {
    AddOrUpdate,
    CategoryCascader,
  },
  activated() {
    this.getDataList();
  },
  methods: {
    // 修改显示状态
    updateShowStatus(showStatus, brandId) {
      console.log(showStatus, brandId);
      this.$http({
        url: this.$http.adornUrl("/product/brand/update/ShowStatus"),
        method: "put",
        data: this.$http.adornData(
          { brandId: brandId, showStatus: showStatus },
          false
        ),
      }).then(({ data }) => {
        this.$message({
          message: "修改显示状态成功",
          type: "success",
        });
        this.getDataList();
      });
    },

    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/brand/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
        }),
      }).then(({ data }) => {
        if (data && data.code === 20000) {
          this.dataList = data.page.list;
          this.totalPage = data.page.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    // 打开关联分类菜单
    addCategoryBrand(brand) {
      this.addCategoryBrandVisible = true;
      this.brandId = brand.brandId;
      this.name = brand.name;
      console.log(this.name);
      this.getCateRelation();
      console.log(this.categoryList);
    },
    // 添加关联分类数据
    addCategoryBrandData() {
      this.popCatelogSelectVisible = false;
      this.$http({
        url: this.$http.adornUrl("/product/categorybrandrelation/save"),
        method: "post",
        data: this.$http.adornData(
          {
            brandId: this.brandId,
            catelogId: this.categoryList[this.categoryList.length - 1],
            brandName: this.name,
          },
          false
        ),
      }).then(({ data }) => {
        this.getCateRelation();
        this.categoryList = [];
      });
    },
    // 删除关联分类数据
    removeCategoryBrandData(id) {
      this.$confirm(`确定删除当前数据?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl("/product/categorybrandrelation/delete"),
          method: "delete",
          data: this.$http.adornData([id], false),
        }).then(({ data }) => {
          if (data && data.code === 20000) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
              onClose: () => {
                this.getCateRelation();
              },
            });
          } else {
            this.$message.error(data.msg);
          }
        });
      });
    },
    // 获取关联分组数据
    getCateRelation() {
      this.$http({
        url: this.$http.adornUrl(
          `/product/categorybrandrelation/infoByBrandId/${this.brandId}`
        ),
        method: "get",
      }).then(({ data }) => {
        this.categoryBrandDataList = data.list;
      });
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    },
    // 删除
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map((item) => {
            return item.brandId;
          });
      this.$confirm(
        `确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/product/brand/delete"),
          method: "delete",
          data: this.$http.adornData(ids, false),
        }).then(({ data }) => {
          if (data && data.code === 20000) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
              onClose: () => {
                this.getDataList();
              },
            });
          } else {
            this.$message.error(data.msg);
          }
        });
      });
    },
  },
};
</script>
