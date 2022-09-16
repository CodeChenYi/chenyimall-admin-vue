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
      <el-form-item label="是否允许多个值" prop="searchType">
        <el-switch
          v-model="dataForm.valueMany"
          :inactive-value="0"
          :active-value="1"
          active-text="启用"
          inactive-text="禁用"
        >
        </el-switch>
      </el-form-item>
      <el-form-item label="可选值列表" prop="valueSelect">
        <el-input
          v-model="dataForm.valueSelect"
          placeholder="可选值列表"
        ></el-input>
      </el-form-item>
      <el-form-item label="属性类型" prop="attrType">
        <el-select v-model="dataForm.attrType" placeholder="请选择" clearable>
          <el-option label="销售属性" :value="0"> </el-option>
          <el-option label="基本属性" :value="1" disabled> </el-option>
          <!-- <el-option label="两项都是" :value="2"> </el-option> -->
        </el-select>
      </el-form-item>
      <el-form-item label="所属分类" prop="catelogId">
        <category-cascader
          :categoryList.sync="categoryPath"
        ></category-cascader>
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
import CategoryCascader from "../common/category-cascader";
export default {
  components: {
    CategoryCascader,
  },
  data() {
    return {
      visible: false,
      dataForm: {
        attrId: 0,
        attrName: "",
        searchType: "",
        icon: "",
        valueSelect: "",
        categoryPath: [],
        attrType: "",
        enable: "",
        catelogId: "",
        showDesc: "",
      },
      categoryPath: [],
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
      },
    };
  },
  watch: {
    categoryPath() {
      if (this.categoryPath.length > 0) {
        this.dataForm.catelogId =
          this.categoryPath[this.categoryPath.length - 1];
      }
    },
    visible() {
      if (this.visible === false) {
        console.log("----------------------------");
        this.dataForm.categoryPath = [];
        this.categoryPath = [];
      }
    },
  },
  methods: {
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
            if (data && data.code === 20000) {
              this.dataForm.attrName = data.attr.attrName;
              this.dataForm.searchType = data.attr.searchType;
              this.dataForm.icon = data.attr.icon;
              this.dataForm.valueSelect = data.attr.valueSelect;
              this.dataForm.attrType = data.attr.attrType;
              this.dataForm.enable = data.attr.enable;
              this.dataForm.catelogId = data.attr.catelogId;
              this.dataForm.showDesc = data.attr.showDesc;
              this.dataForm.valueMany = data.attr.valueMany;
              this.categoryPath = data.attr.categoryPath;
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
              valueMany: this.dataForm.valueMany,
              enable: this.dataForm.enable,
              catelogId: this.categoryPath[this.categoryPath.length - 1],
              showDesc: this.dataForm.showDesc,
            }),
          }).then(({ data }) => {
            if (data && data.code === 20000) {
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
