<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="auto"
    >
      <el-form-item label="仓库名称" prop="wareId">
        <el-select
          v-model="dataForm.wareId"
          clearable
          filterable
          placeholder="请选择仓库名称"
        >
          <el-option
            v-for="item in wareDataList"
            :key="item.id"
            :label="item.name"
            :value="item.id"
          >
          </el-option>
        </el-select>
        <!-- <el-input v-model="dataForm.wareId" placeholder="仓库"></el-input> -->
      </el-form-item>
      <el-form-item label="商品名称" prop="skuId">
        <el-select
          v-model="dataForm.skuId"
          clearable
          filterable
          ref="selection"
          placeholder="请选择商品名称"
   
        >
          <el-option
            v-for="item in skuDataList"
            :key="item.skuId"
            :label="item.skuName"
            :value="item.skuId"
            @click.native="getSkuName(item.skuName)"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="库存数" prop="stock">
        <el-input v-model="dataForm.stock" placeholder="库存数"></el-input>
      </el-form-item>
      <el-form-item label="锁定库存" prop="stockLocked">
        <el-input
          v-model="dataForm.stockLocked"
          placeholder="锁定库存"
        ></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      visible: false,
      dataForm: {
        id: "",
        skuId: "",
        wareId: "",
        stock: "",
        skuName: "",
        stockLocked: "",
      },
      wareDataList: [],
      skuDataList: [],
      dataRule: {
        wareId: [
          { required: true, message: "仓库id不能为空", trigger: "blur" },
        ],
        stock: [{ required: true, message: "库存数不能为空", trigger: "blur" }],
        skuId: [
          { required: true, message: "sku名称不能为空", trigger: "blur" },
        ],
        stockLocked: [
          { required: true, message: "锁定库存不能为空", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getWareInfo();
    this.getProductSku()
  },
  methods: {
    getWareInfo() {
      this.$http({
        url: this.$http.adornUrl("/ware/wareinfo/list"),
        method: "get",
      }).then(({ data }) => {
        if (data && data.code === 20000) {
          this.wareDataList = data.list;
        } else {
          this.wareDataList = [];
        }
      });
    },
    getProductSku() {
      this.$http({
        url: this.$http.adornUrl("/product/skuinfo/listSku"),
        method: "get",
      }).then(({ data }) => {
        if (data && data.code == 20000) {
          this.skuDataList = data.list;
        } else {
          this.wareDataList = []
        }
      });
    },
    getSkuName(skuName) {
      this.dataForm.skuName = skuName;
      console.log(this.dataForm.skuName)
    },
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/ware/waresku/info/${this.dataForm.id}`),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 20000) {
              this.dataForm.skuId = data.wareSku.skuId;
              this.dataForm.wareId = data.wareSku.wareId;
              this.dataForm.stock = data.wareSku.stock;
              this.dataForm.skuName = data.wareSku.skuName;
              this.dataForm.stockLocked = data.wareSku.stockLocked;
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
              `/ware/waresku/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: `${!this.dataForm.id ? "post" : "put"}`,
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              skuId: this.dataForm.skuId,
              wareId: this.dataForm.wareId,
              stock: this.dataForm.stock,
              skuName: this.dataForm.skuName,
              stockLocked: this.dataForm.stockLocked,
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
