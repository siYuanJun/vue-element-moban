<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : !disabled ? '修改' : '查看'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="120px">
      <el-form-item label="优惠券名称" prop="name">
        <el-input v-model="dataForm.name" :disabled="disabled" placeholder="优惠券名称"></el-input>
      </el-form-item>
      <el-form-item label="最低消费金额" prop="minPrice">
        <el-input v-model="dataForm.minPrice" :disabled="disabled" placeholder="最低消费金额"></el-input>
      </el-form-item>
      <el-form-item label="优惠券说明" prop="couponDesc">
        <el-input type="textarea" v-model="dataForm.couponDesc" :disabled="disabled" placeholder="优惠券说明"></el-input>
      </el-form-item>
      <el-form-item label="优惠券图片" prop="picUrl">
        <el-img v-model="dataForm.picUrl" :disabled="disabled">
        </el-img>
      </el-form-item>
      <el-form-item label="优惠券类型" prop="type">
        <el-radio-group v-model="dataForm.type" :disabled="disabled">
          <el-radio :label="1">折扣</el-radio>
          <el-radio :label="2">满减</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="折扣率" v-if="dataForm.type === 1" prop="discount">
        <el-input v-model="dataForm.discount" :disabled="disabled" placeholder="折扣率"></el-input>
      </el-form-item>
      <el-form-item label="优惠金额" v-if="dataForm.type === 2" prop="subPrice">
        <el-input v-model="dataForm.subPrice" :disabled="disabled" placeholder="优惠金额"></el-input>
      </el-form-item>
      <el-form-item label="到期类型" prop="expireType">
        <el-radio-group v-model="dataForm.expireType" :disabled="disabled">
          <el-radio :label="1">领取后N天过期</el-radio>
          <el-radio :label="2">指定有效期</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="有效天数" v-if="dataForm.expireType === 1" prop="expireDay">
        <el-input v-model="dataForm.expireDay" :disabled="disabled" placeholder="有效天数，EXPIRE_TYPE:1时"></el-input>
      </el-form-item>
      <el-form-item label="有效期开始时间" v-if="dataForm.expireType === 2" prop="beginTime">
        <el-input v-model="dataForm.beginTime" :disabled="disabled" placeholder="有效期开始时间"></el-input>
      </el-form-item>
      <el-form-item label="有效期结束时间" v-if="dataForm.expireType === 2" prop="endTime">
        <el-input v-model="dataForm.endTime" :disabled="disabled" placeholder="有效期结束时间"></el-input>
      </el-form-item>
      <el-form-item label="投放总数量" prop="totalCount">
        <el-input v-model="dataForm.totalCount" :disabled="disabled" placeholder="投放总数量"></el-input>
      </el-form-item>
      <el-form-item label="已发数量" prop="sendCount">
        <el-input v-model="dataForm.sendCount" :disabled="disabled" placeholder="已发数量"></el-input>
      </el-form-item>
      <el-form-item label="是否指定商品" prop="limitGoods">
        <el-radio-group v-model="dataForm.limitGoods" :disabled="disabled">
          <el-radio :label="0">否</el-radio>
          <el-radio :label="1">是</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="是否指定商品" v-if="dataForm.limitGoods === 1" prop="limitGoods">
        <el-select v-model="dataForm.goodsIds" :disabled="disabled" clearable multiple filterable placeholder="请选择"
                   style="width: 100%">
          <el-option
            v-for="goods in goodsList"
            :key="goods.id"
            :label="goods.name"
            :value="goods.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="每人限领数量" prop="userNum">
        <el-input v-model="dataForm.userNum" :disabled="disabled" placeholder="每人限领数量"></el-input>
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-radio-group v-model="dataForm.status" :disabled="disabled">
          <el-radio :label="0">禁用</el-radio>
          <el-radio :label="1">正常</el-radio>
        </el-radio-group>
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
          name: '',
          couponDesc: '',
          picUrl: '',
          type: '',
          minPrice: '',
          subPrice: '',
          discount: '',
          expireType: '',
          expireDay: '',
          beginTime: '',
          endTime: '',
          totalCount: '',
          sendCount: '',
          limitGoods: '',
          userNum: '',
          status: '',
          sort: '',
          goodsIds: ''
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
              url: `/mall/coupon/info/${this.dataForm.id}`,
              method: 'get'
            }).then(({ data }) => {
              if (data && data.code === 0) {
                this.dataForm = data.coupon
              }
            })
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
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm']
          .validate((valid) => {
            if (valid) {
              this.$http({
                url: `/mall/coupon/${!this.dataForm.id ? 'save' : 'update'}`,
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
