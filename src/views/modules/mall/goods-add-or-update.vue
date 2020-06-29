<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : !disabled ? '修改' : '查看商品详情'"
    :close-on-click-modal="false"
    :visible.sync="visible">

    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             inline label-width="120px">
      <el-tabs tab-position="left">
        <el-tab-pane label="通用信息">
          <el-form-item label="名称" prop="name">
            <el-input v-model="dataForm.name" :disabled="disabled" placeholder="名称"></el-input>
          </el-form-item>
          <el-form-item label="商品编码" prop="goodsSn">
            <el-input v-model="dataForm.goodsSn" :disabled="disabled" placeholder="商品编码"></el-input>
          </el-form-item>
          <el-form-item label="商品类型" prop="categoryId">
            <el-select v-model="dataForm.categoryId" :disabled="disabled" clearable filterable placeholder="请选择"
                       class="width185">
              <el-option
                v-for="category in categorys"
                :key="category.id"
                :label="category.name"
                :value="category.id">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="品牌" prop="brandId">
            <el-select v-model="dataForm.brandId" :disabled="disabled" clearable filterable placeholder="请选择"
                       class="width185">
              <el-option
                v-for="brand in brands"
                :key="brand.id"
                :label="brand.name"
                :value="brand.id">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="商品库存" prop="goodsNumber">
            <el-input v-model="dataForm.goodsNumber" :disabled="disabled" placeholder="商品库存"></el-input>
          </el-form-item>
          <el-form-item label="销量" prop="sales">
            <el-input v-model="dataForm.sales" :disabled="disabled" placeholder="销量"></el-input>
          </el-form-item>
          <el-form-item label="是否热销" prop="isHot">
            <el-dict code="IS_NOT" :disabled="disabled" v-model="dataForm.isHot"></el-dict>
          </el-form-item>
          <el-form-item label="是否新商品" prop="isNew">
            <el-dict code="IS_NOT" :disabled="disabled" v-model="dataForm.isNew"></el-dict>
          </el-form-item>
          <el-form-item label="是否限购" prop="isLimited">
            <el-dict code="IS_NOT" :disabled="disabled" v-model="dataForm.isLimited"></el-dict>
          </el-form-item>
          <el-form-item label="是否APP专属" prop="isAppExclusive">
            <el-dict code="IS_NOT" :disabled="disabled" v-model="dataForm.isAppExclusive"></el-dict>
          </el-form-item>
          <el-form-item label="APP专享价" prop="appExclusivePrice">
            <el-input v-model="dataForm.appExclusivePrice" :disabled="disabled" placeholder="APP专享价"></el-input>
          </el-form-item>
          <el-form-item label="列表图" prop="listPicUrl">
            <el-img v-model="dataForm.listPicUrl" :disabled="disabled">
            </el-img>
          </el-form-item>
          <el-form-item label="关键词" prop="keywords">
            <el-input v-model="dataForm.keywords" :disabled="disabled" placeholder="关键词"></el-input>
          </el-form-item>
          <el-form-item label="简明介绍" prop="goodsBrief">
            <el-input v-model="dataForm.goodsBrief" :disabled="disabled" placeholder="简明介绍"></el-input>
          </el-form-item>
          <el-form-item label="进价" prop="unitPrice">
            <el-input v-model="dataForm.unitPrice" :disabled="disabled" placeholder="进价"></el-input>
          </el-form-item>
          <el-form-item label="市场价" prop="marketPrice">
            <el-input v-model="dataForm.marketPrice" :disabled="disabled" placeholder="市场价"></el-input>
          </el-form-item>
          <el-form-item label="零售价格" prop="retailPrice">
            <el-input v-model="dataForm.retailPrice" :disabled="disabled" placeholder="零售价格"></el-input>
          </el-form-item>
          <el-form-item label="专柜价格" prop="counterPrice">
            <el-input v-model="dataForm.counterPrice" :disabled="disabled" placeholder="专柜价格"></el-input>
          </el-form-item>
          <el-form-item label="销售量" prop="sellVolume">
            <el-input v-model="dataForm.sellVolume" :disabled="disabled" placeholder="销售量"></el-input>
          </el-form-item>
          <el-form-item label="推广标签" prop="promotionTag">
            <el-input v-model="dataForm.promotionTag" :disabled="disabled" placeholder="推广标签"></el-input>
          </el-form-item>
          <el-form-item label="推广描述" prop="promotionDesc">
            <el-input v-model="dataForm.promotionDesc" :disabled="disabled" placeholder="推广描述"></el-input>
          </el-form-item>
          <el-form-item label="排序" prop="sort">
            <el-input v-model="dataForm.sort" :disabled="disabled" placeholder="排序"></el-input>
          </el-form-item>
        </el-tab-pane>
        <el-tab-pane label="商品参数">
          <div v-for="(item,index) in dataForm.goodsAttributeEntityList">
            <el-row>
              <el-col :span="10">
                <el-form-item label="属性">
                  <el-select v-model="item.attributeId" :disabled="disabled" clearable filterable placeholder="请选择"
                             class="width185">
                    <el-option v-for="attribute in attributes"
                               :value="attribute.id"
                               :label="attribute.name"
                               :key="attribute.id">
                    </el-option>
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="11">
                <el-form-item label="值">
                  <el-input type="text" :disabled="disabled" v-model="item.value"/>
                </el-form-item>
              </el-col>
              <el-col :span="3" style="min-width: 95px;">
                <el-button type="warning" :disabled="disabled" icon="el-icon-plus" v-if="index == 0" @click="addAttrRow"
                           circle></el-button>
                <el-button type="danger" :disabled="disabled" icon="el-icon-delete" @click="delAttrRow(index)"
                           circle></el-button>
              </el-col>
            </el-row>
          </div>
        </el-tab-pane>
        <el-tab-pane label="商品规格">
          <div v-for="(item,index) in dataForm.specificationEntityList">
            <el-row>
              <el-col :span="10">
                <el-form-item label="名称">
                  <el-input type="text" :disabled="disabled" v-model="item.name"/>
                </el-form-item>
              </el-col>
              <el-col :span="11">
                <el-form-item label="排序">
                  <el-input type="text" :disabled="disabled" v-model="item.sort"/>
                </el-form-item>
              </el-col>
              <el-col :span="3" style="min-width: 95px;">
                <el-button type="warning" :disabled="disabled" icon="el-icon-plus" v-if="index == 0" @click="addSpeRow"
                           circle></el-button>
                <el-button type="danger" :disabled="disabled" icon="el-icon-delete" @click="delSpeRow(index)"
                           circle></el-button>
              </el-col>
            </el-row>
          </div>
        </el-tab-pane>
        <el-tab-pane label="详细描述">
          <el-upload
            :disabled="disabled"
            class="upload-demo"
            :action="url"
            :on-remove="handleRemove"
            :before-upload="beforeUploadHandle"
            :on-success="successHandle"
            :file-list="dataForm.attachmentEntityList"
            list-type="picture-card">
            <el-button v-if="!disabled" size="small" type="primary">点击上传</el-button>
          </el-upload>
          <el-button type="success" @click="openDetail">预览详情</el-button>
          <ueditor v-model="dataForm.goodsDesc" :disabled="disabled" placeholder="商品描述"></ueditor>
        </el-tab-pane>
      </el-tabs>
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
        url: '',
        disabled: false,
        visible: false,
        dataForm: {
          id: '',
          name: '',
          categoryId: '',
          goodsSn: '',
          brandId: '',
          goodsNumber: '',
          isHot: '',
          isNew: '',
          isLimited: '',
          isDelete: '',
          primaryPicUrl: '',
          listPicUrl: '',
          keywords: '',
          goodsBrief: '',
          unitPrice: '',
          marketPrice: '',
          retailPrice: '',
          counterPrice: '',
          extraPrice: '',
          sellVolume: '',
          isAppExclusive: '',
          appExclusivePrice: '',
          goodsDesc: '',
          sort: '',
          promotionTag: '',
          promotionDesc: '',
          createUserId: '',
          createTime: '',
          createUserDeptId: '',
          updateUserId: '',
          updateTime: '',
          attachmentEntityList: [],
          specificationEntityList: [{
            'id': '',
            'goodsId': '',
            'name': '',
            'sort': ''
          }],
          goodsAttributeEntityList: [{
            'id': '',
            'goodsId': '',
            'attributeId': '',
            'value': ''
          }]
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
        categorys: [],
        brands: [],
        attributes: []
      }
    },
    methods: {
      openDetail () {
        this.$alert(`<div style="width: 360px;height: 720px;overflow-y: auto;overflow-x: hidden"">${this.dataForm.goodsDesc}</div>`, this.dataForm.name, {
          dangerouslyUseHTMLString: true,
          closeOnClickModal: true,
          callback: action => {
          }
        })
      },
      init (id, disabled) {
        this.url = this.$http.BASE_URL + `/sys/oss/upload?token=${this.$cookie.get('token')}`
        this.disabled = disabled
        this.dataForm.id = id || ''
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: `/mall/goods/info/${this.dataForm.id}`,
              method: 'get'
            }).then(({ data }) => {
              if (data && data.code === 0) {
                this.dataForm = data.goods
                if (data.goods.goodsAttributeEntityList.length > 0) {
                  this.dataForm.goodsAttributeEntityList = data.goods.goodsAttributeEntityList
                } else {
                  this.dataForm.goodsAttributeEntityList = [{
                    'id': '',
                    'goodsId': this.dataForm.id,
                    'attributeId': '',
                    'value': ''
                  }]
                }
                if (data.goods.specificationEntityList.length > 0) {
                  this.dataForm.specificationEntityList = data.goods.specificationEntityList
                } else {
                  this.dataForm.specificationEntityList = [{
                    'id': '',
                    'goodsId': this.dataForm.id,
                    'attributeId': '',
                    'value': ''
                  }]
                }
              }
            })
          } else {
            this.dataForm = {
              id: '',
              name: '',
              categoryId: '',
              goodsSn: '',
              brandId: '',
              goodsNumber: '',
              isHot: '',
              isNew: '',
              isLimited: '',
              isDelete: '',
              primaryPicUrl: '',
              listPicUrl: '',
              keywords: '',
              goodsBrief: '',
              unitPrice: '',
              marketPrice: '',
              retailPrice: '',
              counterPrice: '',
              extraPrice: '',
              sellVolume: '',
              isAppExclusive: '',
              appExclusivePrice: '',
              goodsDesc: '',
              sort: '',
              promotionTag: '',
              promotionDesc: '',
              createUserId: '',
              createTime: '',
              createUserDeptId: '',
              updateUserId: '',
              updateTime: '',
              attachmentEntityList: [],
              specificationEntityList: [{
                'id': '',
                'goodsId': '',
                'name': '',
                'sort': ''
              }],
              goodsAttributeEntityList: [{
                'id': '',
                'goodsId': '',
                'attributeId': '',
                'value': ''
              }]
            }
          }
        })
      },
      delAttrRow: function (index) {
        // 最后一行时禁止删除
        if (this.dataForm.goodsAttributeEntityList.length === 1) {
          return
        }
        this.dataForm.goodsAttributeEntityList.splice(index, 1)
      },
      addAttrRow: function () {
        let goodsId = ''
        if (this.dataForm) {
          goodsId = this.dataForm.id
        }
        this.dataForm.goodsAttributeEntityList.push({
          'id': '',
          'goodsId': goodsId,
          'attributeId': '',
          'value': ''
        })
      },
      delSpeRow: function (index) {
        // 最后一行时禁止删除
        if (this.dataForm.specificationEntityList.length === 1) {
          return
        }
        this.dataForm.specificationEntityList.splice(index, 1)
      },
      addSpeRow: function () {
        let goodsId = ''
        if (this.dataForm.goods) {
          goodsId = this.dataForm.goods.id
        }
        this.dataForm.specificationEntityList.push({
          'id': '',
          'goodsId': goodsId,
          'name': '',
          'sort': ''
        })
      },
      handleRemove (file, fileList) {
        this.dataForm.attachmentEntityList = fileList
      },
      // 上传之前
      beforeUploadHandle (file) {
        if (file.type !== 'image/jpg' && file.type !== 'image/jpeg' && file.type !== 'image/png' && file.type !== 'image/gif') {
          this.$message.error('只支持jpg、png、gif格式的图片！')
          return false
        }
      },
      // 上传成功
      successHandle (response, file, fileList) {
        if (response && response.code === 0) {
          this.dataForm.attachmentEntityList.push({
            name: file.name,
            url: response.url
          })
        } else {
          this.$message.error(response.msg)
        }
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm']
          .validate((valid) => {
            if (valid) {
              this.$http({
                url: `/mall/goods/${!this.dataForm.id ? 'save' : 'update'}`,
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
        url: `/mall/category/queryAll`,
        method: 'get',
        params: {
          'level': 2
        }
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.categorys = data.list
        }
      })
      this.$http({
        url: `/mall/brand/queryAll`,
        method: 'get'
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.brands = data.list
        }
      })
      this.$http({
        url: `/mall/attribute/queryAll`,
        method: 'get'
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.attributes = data.list
        }
      })
    }
  }
</script>
<style>
  .el-dialog {
    width: 70%;
    min-width: 920px;
  }
</style>
