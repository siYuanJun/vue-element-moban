<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : !disabled ? '修改' : '查看'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="120px">
      <el-form-item label="状态" prop="status">
        <el-radio-group v-model="dataForm.status" :disabled="disabled">
          <el-radio :label="0">未使用</el-radio>
          <el-radio :label="1">已使用</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="领取类型" prop="type">
        <el-radio-group v-model="dataForm.type" :disabled="disabled">
          <el-radio :label="0">平台发放</el-radio>
          <el-radio :label="1">自动发放</el-radio>
          <el-radio :label="2">领券中心领取</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="是否已过期" prop="isExpire">
        <el-radio-group v-model="dataForm.isExpire" :disabled="disabled">
          <el-radio :label="0">未过期</el-radio>
          <el-radio :label="1">已过期</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="使用时间" prop="usedTime">
        <el-date-picker type="datetime" v-model="dataForm.usedTime" :disabled="disabled"
                        placeholder="使用时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="有效期开始时间" prop="beginTime">
        <el-date-picker type="datetime" v-model="dataForm.beginTime" :disabled="disabled"
                        placeholder="有效期开始时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="有效期结束时间" prop="endTime">
        <el-date-picker type="datetime" v-model="dataForm.endTime" :disabled="disabled"
                        placeholder="有效期结束时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="领用时间" prop="addTime">
        <el-date-picker type="datetime" v-model="dataForm.addTime" :disabled="disabled"
                        placeholder="领用时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="优惠金额" prop="subPrice">
        <el-input v-model="dataForm.subPrice" :disabled="disabled" placeholder="优惠金额"></el-input>
      </el-form-item>
      <el-form-item label="使用的订单ID" prop="orderId">
        <el-input v-model="dataForm.orderId" :disabled="disabled" placeholder="使用的订单ID"></el-input>
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
          couponId: '',
          subPrice: '',
          addTime: '',
          beginTime: '',
          endTime: '',
          type: '',
          isExpire: '',
          status: '',
          usedTime: '',
          orderId: ''
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
              url: `/mall/usercoupon/info/${this.dataForm.id}`,
              method: 'get'
            }).then(({ data }) => {
              if (data && data.code === 0) {
                this.dataForm = data.usercoupon
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
                url: `/mall/usercoupon/${!this.dataForm.id ? 'save' : 'update'}`,
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
