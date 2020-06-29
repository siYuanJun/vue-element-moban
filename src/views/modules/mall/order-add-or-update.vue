<template>
  <el-dialog
    title="订单详情"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form inline :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="140px">
      <el-card class="box-card" shadow="hover">
        <div slot="header" class="clearfix">
          <span>收货人信息</span>
        </div>
        <el-form-item label="收货人：" prop="consignee">
          <span>{{dataForm.consignee}}</span>
        </el-form-item>
        <el-form-item label="收货地址：" prop="province">
          <span>{{dataForm.province + dataForm.city + dataForm.district + dataForm.address}}</span>
        </el-form-item>
        <el-form-item label="手机号：" prop="mobile">
          <span>{{dataForm.mobile}}</span>
        </el-form-item>
        <el-form-item label="留言：" prop="postscript">
          <span>{{dataForm.postscript}}</span>
        </el-form-item>
        <el-form-item label="邮编：" prop="postalCode">
          <span>{{dataForm.postalCode}}</span>
        </el-form-item>
      </el-card>
      <el-card class="box-card" shadow="hover">
        <div slot="header" class="clearfix">
          <span>商品列表</span>
        </div>
        <el-table
          :data="dataForm.orderGoodsEntityList"
          border
          style="width: 100%">
          <el-table-column
            prop="goodsName"
            label="商品名称"
            header-align="center"
            align="center">
          </el-table-column>
          <el-table-column
            prop="goodsSn"
            label="商品编码"
            header-align="center"
            align="center">
          </el-table-column>
          <el-table-column
            prop="number"
            label="商品数量"
            header-align="center"
            align="center">
          </el-table-column>
          <el-table-column
            prop="retailPrice"
            label="零售价格"
            header-align="center"
            align="center">
          </el-table-column>
          <el-table-column
            prop="goodsSpecifitionNameValue"
            label="商品规格"
            header-align="center"
            align="center">
          </el-table-column>
          <el-table-column
            prop="listPicUrl"
            label="图片"
            header-align="center"
            align="center">
            <template slot-scope="scope">
              <img style="height: 50%;width: 50%" @click="openImg(scope.row.listPicUrl)" :src="scope.row.listPicUrl"/>
            </template>
          </el-table-column>
        </el-table>
      </el-card>

      <el-card class="box-card" shadow="hover">
        <div slot="header" class="clearfix">
          <span>订单信息</span>
        </div>
        <el-form-item label="订单总价：" prop="orderPrice">
          <el-tag type="danger">{{dataForm.orderPrice}}</el-tag>
        </el-form-item>
        <el-form-item label="实付金额：" prop="actualPrice">
          <el-tag type="danger">{{dataForm.actualPrice}}</el-tag>
        </el-form-item>
        <el-divider></el-divider>
        <el-form-item label="下单来源：" prop="fromType">
          <el-radio-group v-model="dataForm.fromType" :disabled="disabled">
            <el-radio :label="1">微信小程序</el-radio>
            <el-radio :label="2">微信公众号</el-radio>
            <el-radio :label="3">APP</el-radio>
            <el-radio :label="4">H5</el-radio>
            <el-radio :label="5">支付宝小程序</el-radio>
            <el-radio :label="6">QQ小程序</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="订单类型：" prop="orderType">
          <el-radio-group v-model="dataForm.orderType" :disabled="disabled">
            <el-radio :label="1">商城订单</el-radio>
            <el-radio :label="2">店铺自提订单</el-radio>
            <el-radio :label="3">秒杀订单</el-radio>
            <el-radio :label="4">积分订单</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item v-if="dataForm.orderType===3" label="秒杀商品：">
          <span>{{dataForm.goodsName}}</span>
        </el-form-item>
        <el-form-item label="订单状态：" prop="orderStatus">
          <el-radio-group v-model="dataForm.orderStatus" :disabled="disabled">
            <el-radio :label="0">待付款</el-radio>
            <el-radio :label="101">已取消</el-radio>
            <el-radio :label="102">已删除</el-radio>
            <el-radio :label="201">待发货</el-radio>
            <el-radio :label="300">已发货</el-radio>
            <el-radio :label="301">确认收货</el-radio>
            <el-radio :label="401">退款</el-radio>
            <el-radio :label="402">售后退款</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="付款状态：" prop="payStatus">
          <el-radio-group v-model="dataForm.payStatus" :disabled="disabled">
            <el-radio :label="1">未付款</el-radio>
            <el-radio :label="2">付款中</el-radio>
            <el-radio :label="3">已付款</el-radio>
            <el-radio :label="4">退款</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="支付方式：" prop="payType">
          <el-radio-group v-model="dataForm.payType" :disabled="disabled">
            <el-radio :label="1">微信支付</el-radio>
            <el-radio :label="2">余额支付</el-radio>
            <el-radio :label="3">积分支付</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="发货状态：" prop="shippingStatus">
          <el-radio-group v-model="dataForm.shippingStatus" :disabled="disabled">
            <el-radio :label="1">未发货</el-radio>
            <el-radio :label="2">已发货</el-radio>
            <el-radio :label="3">已收货</el-radio>
            <el-radio :label="4">退货</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="订单编号：" prop="orderSn">
          <span>{{dataForm.orderSn}}</span>
        </el-form-item>
        <el-form-item label="支付PREPAY_ID：" prop="prepayId">
          <span>{{dataForm.prepayId}}</span>
        </el-form-item>
        <el-form-item label="父级订单ID：" prop="parentId">
          <span>{{dataForm.parentId}}</span>
        </el-form-item>
        <el-form-item label="店铺ID：" prop="shopsId">
          <span>{{dataForm.shopsId}}</span>
        </el-form-item>
        <el-form-item label="下单时间：" prop="addTime">
          <span>{{dataForm.addTime}}</span>
        </el-form-item>
        <el-form-item label="过期时间：" prop="addTime">
          <span>{{dataForm.expireTime}}</span>
        </el-form-item>
        <el-form-item label="确认时间：" prop="confirmTime">
          <span>{{dataForm.confirmTime}}</span>
        </el-form-item>
        <el-form-item label="付款时间：" prop="payTime">
          <span>{{dataForm.payTime}}</span>
        </el-form-item>
      </el-card>
      <el-card class="box-card" shadow="hover">
        <div slot="header" class="clearfix">
          <span>优惠</span>
        </div>
        <el-form-item label="积分抵扣金额：" prop="integralMoney">
          <span>{{dataForm.integralMoney}}</span>
        </el-form-item>
        <el-form-item label="使用的优惠券ID：" prop="couponId">
          <span>{{dataForm.couponId}}</span>
        </el-form-item>
        <el-form-item label="优惠价格：" prop="couponPrice">
          <span>{{dataForm.couponPrice}}</span>
        </el-form-item>
      </el-card>
      <el-card class="box-card" shadow="hover">
        <div slot="header" class="clearfix">
          <span>物流信息</span>
        </div>
        <el-form-item label="快递公司名称：" prop="shippingName">
          <span>{{dataForm.shippingName}}</span>
        </el-form-item>
        <el-form-item label="快递公司CODE：" prop="shippingCode">
          <span>{{dataForm.shippingCode}}</span>
        </el-form-item>
        <el-form-item label="快递单号：" prop="shippingNo">
          <span>{{dataForm.shippingNo}}</span>
        </el-form-item>
        <el-form-item label="快递费用：" prop="shippingFee">
          <span>{{dataForm.shippingFee}}</span>
        </el-form-item>
      </el-card>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
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
          orderType: '',
          orderSn: '',
          userId: '',
          orderStatus: '',
          shippingStatus: '',
          payStatus: '',
          consignee: '',
          country: '',
          province: '',
          city: '',
          district: '',
          address: '',
          mobile: '',
          postscript: '',
          shippingId: '',
          shippingName: '',
          shippingCode: '',
          shippingNo: '',
          shippingFee: '',
          prepayId: '',
          actualPrice: '',
          integralMoney: '',
          orderPrice: '',
          addTime: '',
          confirmTime: '',
          payTime: '',
          couponId: '',
          couponPrice: '',
          parentId: '',
          shopsId: '',
          goodsName: '',
          orderGoodsEntityList: []
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
              url: `/mall/order/info/${this.dataForm.id}`,
              method: 'get'
            }).then(({ data }) => {
              if (data && data.code === 0) {
                this.dataForm = data.order
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
                url: `/mall/order/${!this.dataForm.id ? 'save' : 'update'}`,
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
<style>
  .el-dialog {
    width: 70%;
    min-width: 920px;
  }
</style>
