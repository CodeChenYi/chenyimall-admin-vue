<template>
  <div>
    <el-row :gutter="20">
      <el-col :span="6">
        <category @tree-node-click="treeNodeClick"></category>
      </el-col>
      <el-col :span="18">
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
              <el-button type="primary" @click="addOrUpdateHandle()"
                >新增</el-button
              >
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
              prop="attrId"
              header-align="center"
              align="center"
              label="属性id"
              width="200"
              fixed="left"
            >
            </el-table-column>
            <el-table-column
              prop="attrName"
              header-align="center"
              align="center"
              width="150"
              label="属性名"
            >
            </el-table-column>
            <el-table-column
              prop="icon"
              header-align="center"
              align="center"
              label="属性图标"
              width="150"
            >
              <template slot-scope="scope">
                <el-image :src="scope.row.icon" :fit="fit"></el-image>
              </template>
            </el-table-column>
            <el-table-column
              prop="icon"
              header-align="center"
              align="center"
              label="是否允许多个值"
              width="150"
            >
              <template slot-scope="scope">
                <i class="el-icon-success" v-if="scope.row.valueMany == 1"></i>
                <i class="el-icon-error" v-if="scope.row.valueMany == 0"></i>
              </template>
            </el-table-column>
            <el-table-column
              prop="valueSelect"
              header-align="center"
              align="center"
              label="可选值列表"
              width="200"
            >
              <template slot-scope="scope">
                <el-tooltip
                  class="item"
                  effect="dark"
                  :content="scope.row.valueSelect"
                  placement="top"
                >
                  <el-tag type="success">{{scope.row.valueSelect.split('，')[0] + '...'}}</el-tag>
                </el-tooltip>
              </template>
            </el-table-column>
            <el-table-column
              prop="attrType"
              header-align="center"
              align="center"
              label="属性类型"
              width="150"
            >
              <template slot-scope="scope">
                <p v-if="scope.row.attrType == '0'">销售属性</p>
                <p v-if="scope.row.attrType == '1'">基本属性</p>
                <p v-if="scope.row.attrType == '2'">两者都是</p>
              </template>
            </el-table-column>
            <el-table-column
              prop="categoryName"
              header-align="center"
              align="center"
              label="所属分类"
              width="150"
            >
            </el-table-column>
            <el-table-column
              prop="attrGroupName"
              header-align="center"
              align="center"
              label="所属分组"
              width="150"
            >
            </el-table-column>
            <el-table-column
              prop="searchType"
              header-align="center"
              align="center"
              label="是否需要检索"
              width="150"
            >
              <template slot-scope="scope">
                <el-switch
                  @change="
                    updateSearchType(scope.row.attrId, scope.row.searchType)
                  "
                  v-model="scope.row.searchType"
                  :inactive-value="0"
                  :active-value="1"
                >
                </el-switch>
              </template>
            </el-table-column>
            <el-table-column
              prop="enable"
              header-align="center"
              align="center"
              label="启用状态"
            >
              <template slot-scope="scope">
                <el-switch
                  @change="
                    updateEnableState(scope.row.attrId, scope.row.enable)
                  "
                  v-model="scope.row.enable"
                  :inactive-value="0"
                  :active-value="1"
                >
                </el-switch>
              </template>
            </el-table-column>
            <el-table-column
              prop="showDesc"
              header-align="center"
              align="center"
              label="快速展示"
            >
              <template slot-scope="scope">
                <el-switch
                  @change="
                    updateShowDescState(scope.row.attrId, scope.row.showDesc)
                  "
                  v-model="scope.row.showDesc"
                  :inactive-value="0"
                  :active-value="1"
                >
                </el-switch>
              </template>
            </el-table-column>
            <el-table-column
              fixed="right"
              header-align="center"
              align="center"
              width="150"
              label="操作"
            >
              <template slot-scope="scope">
                <el-button
                  type="text"
                  size="small"
                  @click="addOrUpdateHandle(scope.row.attrId)"
                  >修改</el-button
                >
                <el-button
                  type="text"
                  size="small"
                  @click="deleteHandle(scope.row.attrId)"
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
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import Category from "../common/category";
import AddOrUpdate from "./attr-add-or-update";
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
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
    };
  },
  components: {
    AddOrUpdate,
    Category,
  },
  activated() {
    this.getDataList();
  },
  methods: {
    treeNodeClick(data, node, components) {
      console.log("attrgorp接收到当前组件", data, node, components);
      if (node.level == 3) {
        this.$http({
          url: this.$http.adornUrl(`/product/attr/list/${data.catId}`),
          method: "get",
          params: this.$http.adornParams({
            page: this.pageIndex,
            limit: this.pageSize,
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
      }
    },
    updateShowDescState(attrId, showDesc) {
      this.$http({
        url: this.$http.adornUrl("/product/attr/updateShowDescState"),
        method: "put",
        data: this.$http.adornData(
          { attrId: attrId, showDesc: showDesc },
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
    updateSearchType(attrId, searchType) {
      this.$http({
        url: this.$http.adornUrl("/product/attr/updateSearchType"),
        method: "put",
        data: this.$http.adornData(
          { attrId: attrId, searchType: searchType },
          false
        ),
      }).then(({ data }) => {
        this.$message({
          message: "修改检索状态成功",
          type: "success",
        });
        this.getDataList();
      });
    },
    updateEnableState(attrId, enable) {
      this.$http({
        url: this.$http.adornUrl("/product/attr/updateEnableState"),
        method: "put",
        data: this.$http.adornData({ attrId: attrId, enable: enable }, false),
      }).then(({ data }) => {
        this.$message({
          message: "修改启用状态成功",
          type: "success",
        });
        this.getDataList();
      });
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/attr/list"),
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
            return item.attrId;
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
          url: this.$http.adornUrl("/product/attr/delete"),
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
