<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : !disabled ? '修改' : '查看'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form inline :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="80px">
      <el-form-item label="收货人" prop="userName">
        <el-input v-model="dataForm.userName" :disabled="disabled" placeholder="收货人姓名"></el-input>
      </el-form-item>
      <el-form-item label="手机" prop="mobile">
        <el-input v-model="dataForm.mobile" :disabled="disabled" placeholder="手机"></el-input>
      </el-form-item>
      <el-form-item label="邮编" prop="postalCode">
        <el-input v-model="dataForm.postalCode" :disabled="disabled" placeholder="邮编"></el-input>
      </el-form-item>
      <el-form-item label="省" prop="provinceName">
        <el-input v-model="dataForm.provinceName" :disabled="disabled" placeholder="省"></el-input>
      </el-form-item>
      <el-form-item label="市" prop="cityName">
        <el-input v-model="dataForm.cityName" :disabled="disabled" placeholder="市"></el-input>
      </el-form-item>
      <el-form-item label="区" prop="countyName">
        <el-input v-model="dataForm.countyName" :disabled="disabled" placeholder="区"></el-input>
      </el-form-item>
      <el-form-item label="详细地址" prop="detailInfo">
        <el-input v-model="dataForm.detailInfo" :disabled="disabled" placeholder="详细收货地址信息"></el-input>
      </el-form-item>
      <el-form-item label="默认地址" prop="isDefault">
        <el-radio-group v-model="dataForm.isDefault" :disabled="disabled">
          <el-radio :label="0">否</el-radio>
          <el-radio :label="1">是</el-radio>
        </el-radio-group>
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
          userId: '',
          userName: '',
          mobile: '',
          postalCode: '',
          nationalCode: '',
          provinceName: '',
          cityName: '',
          countyName: '',
          detailInfo: '',
          isDefault: ''
        },
        dataRule: {
          name: [
            {
              required: true,
              message: '名称不能为空',
              trigger: 'blur'
            }
          ]
        }
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
              url: `/mall/address/info/${this.dataForm.id}`,
              method: 'get'
            }).then(({ data }) => {
              if (data && data.code === 0) {
                this.dataForm = data.address
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
                url: `/mall/address/${!this.dataForm.id ? 'save' : 'update'}`,
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
    }
  }
</script>
