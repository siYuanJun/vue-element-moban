<template>
  <el-dialog
    title="SKU配置"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <table class="table table-hover">
      <tbody>
      <tr v-for="(productEntity,productIndex) in productEntityList">
        <th v-for="(item,index) in allSpc">
          <div v-for="(kv,indexKv) in productEntity.keyValue">
            <template v-if="kv.key==item">
              {{item}}
              <el-input v-model="kv.value" :placeholder="item"/>
            </template>
            <template v-if="kv.key=='' && indexKv ==0">
              {{item}}
              <el-input v-model="kv.value" :placeholder="item"/>
            </template>
          </div>
        </th>
        <th>
          序列号：
          <el-input v-model="productEntity.goodsSn" placeholder="商品序列号"/>
        </th>
        <th>
          库存：
          <el-input v-model="productEntity.goodsNumber" placeholder="商品库存"/>
        </th>
        <th>
          零售价格：
          <el-input v-model="productEntity.retailPrice" placeholder="零售价格"/>
        </th>
        <th>
          市场价格：
          <el-input v-model="productEntity.marketPrice" placeholder="市场价格"/>
        </th>
        <th>
          <el-button type="warning" icon="el-icon-plus" v-if="productIndex == 0"
                     @click="addAttrRow"
                     circle></el-button>
        </th>
        <th>
          <el-button type="danger" icon="el-icon-delete" @click="delAttrRow(productIndex)"
                     circle></el-button>
        </th>
      </tr>
      </tbody>
    </table>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        goodsId: '',
        visible: false,
        allSpc: [],
        productEntityList: [] // 商品的所有产品
      }
    },
    methods: {
      init (id) {
        this.goodsId = id
        this.productEntityList = [{
          keyValue: [{
            key: '',
            value: ''
          }],
          goodsSn: '',
          goodsId: id,
          goodsNumber: '',
          retailPrice: '',
          marketPrice: ''
        }]
        this.allSpc = []
        this.visible = true
        this.$nextTick(() => {
          this.$http({
            url: '/mall/specification/queryAll',
            method: 'get',
            params: {
              goodsId: this.goodsId
            }
          }).then(({ data }) => {
            if (data && data.code === 0) {
              for (let i = 0; i < data.list.length; i++) {
                let name = data.list[i].name
                this.allSpc.push(name)
              }
            }
          })
          this.$http({
            url: '/mall/goodssku/queryByGoodsId/' + this.goodsId,
            method: 'get'
          }).then(({ data }) => {
            if (data && data.code === 0) {
              let keyValue = []
              for (let i = 0; i < this.allSpc.length; i++) {
                keyValue.push({
                  key: this.allSpc[i],
                  value: ''
                })
              }

              this.productEntityList = data.list
              if (this.productEntityList.length === 0) {
                this.productEntityList = [{
                  keyValue: keyValue,
                  goodsSn: '',
                  goodsId: this.goodsId,
                  goodsNumber: '',
                  retailPrice: '',
                  marketPrice: ''
                }]
              }
              for (let i = 0; i < this.productEntityList.length; i++) {
                if (this.productEntityList[i].keyValue.length === 0) {
                  this.productEntityList[i].keyValue = keyValue
                } else if (this.productEntityList[i].keyValue.length !== keyValue.length) {
                  // 原来一个规格，然后又新增一个规格，原来的值都置空
                  this.productEntityList[i].keyValue = keyValue
                }
              }
            }
          })
        })
      },
      delAttrRow: function (index) {
        // 最后一行时禁止删除
        if (this.productEntityList.length === 1) {
          return
        }
        this.productEntityList.splice(index, 1)
      },
      addAttrRow: function () {
        let keyValue = []
        for (let i = 0; i < this.allSpc.length; i++) {
          keyValue.push({
            key: this.allSpc[i],
            value: ''
          })
        }
        this.productEntityList.push({
          keyValue: keyValue,
          goodsSn: '',
          goodsId: this.goodsId,
          goodsNumber: '',
          retailPrice: '',
          marketPrice: ''
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$http({
          url: `/mall/goodssku/saveGoodsProduct`,
          method: 'post',
          data: this.productEntityList
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
    }
  }
</script>
<style>
  .el-dialog {
    width: 70%;
    min-width: 920px;
  }
</style>
