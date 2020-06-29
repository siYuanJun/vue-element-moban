<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : !disabled ? '修改' : '查看'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             inline label-width="80px">
      <el-form-item label="店铺" prop="shopsId">
        <el-select v-model="dataForm.shopsId" :disabled="disabled" clearable filterable placeholder="店铺名称"
                   class="width185">
          <el-option
            v-for="shops in shopsList"
            :key="shops.id"
            :label="shops.name"
            :value="shops.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="商品分类" prop="shopsCategoryId">
        <el-select v-model="dataForm.shopsCategoryId" :disabled="disabled" clearable filterable placeholder="商品分类"
                   class="width185">
          <el-option
            v-for="shopsCategory in shopsCategoryList"
            :key="shopsCategory.id"
            :label="shopsCategory.name"
            :value="shopsCategory.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="商品" prop="goodsId">
        <el-select v-model="dataForm.goodsId" :disabled="disabled" clearable filterable placeholder="商品"
                   class="width185">
          <el-option
            v-for="shopsCategory in goodsList"
            :key="shopsCategory.id"
            :label="shopsCategory.name"
            :value="shopsCategory.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="零售价格" prop="retailPrice">
        <el-input v-model="dataForm.retailPrice" :disabled="disabled" placeholder="零售价格"></el-input>
      </el-form-item>
      <el-form-item label="商品库存" prop="goodsNumber">
        <el-input v-model="dataForm.goodsNumber" :disabled="disabled" placeholder="店铺商品库存"></el-input>
      </el-form-item>
      <el-form-item label="排序" prop="sort">
        <el-input v-model="dataForm.sort" :disabled="disabled" placeholder="排序"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button v-if="!disabled" type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        disabled: false,
        visible: false,
        dataForm: {
          id: '',
          shopsId: '',
          shopsCategoryId: '',
          goodsId: '',
          retailPrice: '',
          goodsNumber: '',
          sort: ''
        },
        dataRule: {
          name: [
            {
              required: true,
              message: '名称不能为空',
              trigger: 'blur'
            }
          ]
        },
        shopsList: [],
        shopsCategoryList: [],
        goodsList: []
      }
    },
    methods: {
      init (id, disabled) {
        this.disabled = disabled
        this.dataForm.id = id || ''
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: `/mall/shopsgoods/info/${this.dataForm.id}`,
              method: 'get'
            }).then(({ data }) => {
              if (data && data.code === 0) {
                this.dataForm = data.shopsgoods
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm']
          .validate((valid) => {
            if (valid) {
              this.$http({
                url: `/mall/shopsgoods/${!this.dataForm.id ? 'save' : 'update'}`,
                method: 'post',
                data: this.dataForm
              }).then(({ data }) => {
                if (data && data.code === 0) {
                  this.$message({
                    message: '操作成功',
                    type: 'success',
                    duration: 1500
                  })
                  this.visible = false
                  this.$emit('refreshDataList')
                }
              })
            }
          })
      }
    },
    created () {
      this.$http({
        url: '/mall/shopscategory/queryMyShopCategory',
        method: 'get'
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.shopsCategoryList = data.list
        }
      })
      this.$http({
        url: '/mall/shops/queryMyShop',
        method: 'get'
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.shopsList = data.list
        }
      })
      this.$http({
        url: '/mall/goods/queryAll',
        method: 'get'
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.goodsList = data.list
        }
      })
    }
  }
</script>
