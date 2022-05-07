<template>
  <el-dialog
    :title="!dataForm.attrId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="auto"
      size="small"
    >
      <el-form-item label="属性名" prop="attrName">
        <el-input v-model="dataForm.attrName" placeholder="属性名"></el-input>
      </el-form-item>
      <el-form-item label="属性图标" prop="icon">
        <el-input v-model="dataForm.icon" placeholder="属性图标"></el-input>
      </el-form-item>
      <el-form-item label="可选值列表" prop="valueSelect">
        <el-input
          v-model="dataForm.valueSelect"
          placeholder="可选值列表"
        ></el-input>
        <!-- <el-select v-model="dataForm.valueSelect" multiple filterable placeholder="请选择">
          <el-option
            :key="dataForm.valueSelect"
            :label="dataForm.valueSelect"
            :value="dataForm.valueSelect"
          >
          </el-option>
        </el-select> -->
      </el-form-item>
      <el-form-item label="属性类型" prop="attrType">
        <el-select v-model="dataForm.attrType" placeholder="请选择">
          <el-option label="销售属性" :value="0"> </el-option>
          <el-option label="基本属性" :value="1"> </el-option>
          <el-option label="两项都是" :value="2"> </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="所属分类" prop="catelogId">
        <category-cascader
          :categoryList.sync="categoryPath"
        ></category-cascader>
      </el-form-item>
      <el-form-item label="所属分组" prop="attrGroupId">
        <el-select
          v-model="dataForm.attrGroupId"
          placeholder="请先选择所属分类再进行选择"
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
      </el-form-item>
      <el-form-item label="启用状态" prop="enable">
        <el-switch
          v-model="dataForm.enable"
          :inactive-value="0"
          :active-value="1"
          active-text="启用"
          inactive-text="禁用"
        >
        </el-switch>
      </el-form-item>
      <el-form-item label="是否启用检索" prop="searchType">
        <el-switch
          v-model="dataForm.searchType"
          :inactive-value="0"
          :active-value="1"
          active-text="启用"
          inactive-text="禁用"
        >
        </el-switch>
      </el-form-item>
      <el-form-item label="快速展示" prop="showDesc">
        <el-switch
          v-model="dataForm.showDesc"
          :inactive-value="0"
          :active-value="1"
          active-text="启用"
          inactive-text="禁用"
        >
        </el-switch>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import CategoryCascader from "../common/category-cascader.vue";
export default {
  components: {
    CategoryCascader,
  },
  data() {
    return {
      visible: false,
      dataForm: {
        attrId: "",
        attrGroupId: "",
        attrGroupName: "",
        categoryPath: [],
        attrName: "",
        searchType: 0,
        icon: "",
        valueSelect: "",
        attrType: "",
        enable: 0,
        catelogId: "",
        showDesc: 0,
      },
      clearCount: 0,
      categoryPath: [],
      attrGroupList: [],
      dataRule: {
        attrName: [
          { required: true, message: "属性名不能为空", trigger: "blur" },
        ],
        icon: [
          { required: true, message: "属性图标不能为空", trigger: "blur" },
        ],
        valueSelect: [
          { required: true, message: "可选值列表不能为空", trigger: "blur" },
        ],
        attrType: [
          { required: true, message: "属性类型不能为空", trigger: "blur" },
        ],
        catelogId: [
          { required: true, message: "所属分类不能为空", trigger: "blur" },
        ],
        attrGroupId: [
          { required: true, message: "所属分组不能为空", trigger: "blur" },
        ],
      },
    };
  },
  watch: {
    categoryPath() {
      if (this.clearCount != 0) {
        this.dataForm.attrGroupId = "";
        this.dataForm.attrGroupName = "";
        this.attrGroupList = [];
      } else {
        this.clearCount += 1;
      }
      if (this.categoryPath.length > 0) {
        this.dataForm.catelogId =
          this.categoryPath[this.categoryPath.length - 1];
        this.getAttrGroup();
      }
    },
    visible() {
      this.clearCount = 0;
      if (this.visible === false) {
        console.log("----------------------------");
        this.dataForm.categoryPath = [];
        this.categoryPath = [];
      }
    },
  },
  methods: {
    getAttrGroup() {
      this.$http({
        url: this.$http.adornUrl(
          `/product/attrgroup/list/${this.dataForm.catelogId}`
        ),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        this.attrGroupList = data.page.list;
      });
    },
    init(id) {
      this.dataForm.attrId = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.attrId) {
          this.$http({
            url: this.$http.adornUrl(
              `/product/attr/info/${this.dataForm.attrId}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm = data.attr;
              this.categoryPath = data.attr.categoryPath;
              this.attrGroupList[0] = {
                attrGroupId: data.attr.attrGroupId,
                attrGroupName: data.attr.attrGroupName,
              };
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/product/attr/${!this.dataForm.attrId ? "save" : "update"}`
            ),
            method: `${!this.dataForm.attrId ? "post" : "put"}`,
            data: this.$http.adornData({
              attrId: this.dataForm.attrId || undefined,
              attrName: this.dataForm.attrName,
              searchType: this.dataForm.searchType,
              icon: this.dataForm.icon,
              valueSelect: this.dataForm.valueSelect,
              attrType: this.dataForm.attrType,
              enable: this.dataForm.enable,
              catelogId: this.categoryPath[this.categoryPath.length - 1],
              showDesc: this.dataForm.showDesc,
              attrGroupId: this.dataForm.attrGroupId,
            }),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                },
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
  },
};
</script>
