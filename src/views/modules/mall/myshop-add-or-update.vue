<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : !disabled ? '修改' : '查看'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="100px">
      <el-form-item label="店铺名字" prop="name">
        <el-input v-model="dataForm.name" :disabled="disabled" placeholder="店铺名字"></el-input>
      </el-form-item>
      <el-form-item label="营业时间" prop="workTime">
        <el-time-picker
          is-range
          value-format="H:mm"
          :disabled="disabled"
          v-model="dataForm.workTime"
          range-separator="至"
          start-placeholder="开始时间"
          end-placeholder="结束时间"
          placeholder="营业时间">
        </el-time-picker>
      </el-form-item>
      <el-form-item label="店铺编码" prop="shopsSn">
        <el-input v-model="dataForm.shopsSn" :disabled="disabled" placeholder="店铺编码"></el-input>
      </el-form-item>
      <el-form-item label="店铺图片" prop="imgUrl">
        <el-img v-model="dataForm.imgUrl" :disabled="disabled">
        </el-img>
      </el-form-item>
      <el-form-item label="查询经纬度" prop="longitude">
        <el-link href="https://lbs.qq.com/tool/getpoint/index.html" type="danger" :disabled="disabled" target="_blank">
          https://lbs.qq.com/tool/getpoint/index.html
        </el-link>
      </el-form-item>
      <el-form-item label="经度" prop="longitude">
        <el-input v-model="dataForm.longitude" :disabled="disabled" placeholder="经度"></el-input>
      </el-form-item>
      <el-form-item label="纬度" prop="latitude">
        <el-input v-model="dataForm.latitude" :disabled="disabled" placeholder="纬度"></el-input>
      </el-form-item>
      <el-form-item label="详细位置" prop="details">
        <el-input v-model="dataForm.details" :disabled="disabled" placeholder="详细位置"></el-input>
      </el-form-item>
      <el-form-item label="联系电话" prop="telephone">
        <el-input v-model="dataForm.telephone" :disabled="disabled" placeholder="联系电话"></el-input>
      </el-form-item>
      <el-form-item label="描述" prop="shopDesc">
        <ueditor v-model="dataForm.shopDesc" :disabled="disabled" placeholder="描述"></ueditor>
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
          shopsSn: '',
          imgUrl: '',
          userId: '',
          workTime: [],
          longitude: '',
          latitude: '',
          details: '',
          telephone: '',
          deleteStatus: '',
          shopDesc: '',
          sort: ''
        },
        dataRule: {
          name: [
            {
              required: true,
              message: '名称不能为空',
              trigger: 'blur'
            }
          ],
          workTime: [
            {
              required: true,
              message: '营业时间不能为空',
              trigger: 'blur'
            }
          ],
          shopsSn: [
            {
              required: true,
              message: '店铺编码不能为空',
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
              url: `/mall/shops/info/${this.dataForm.id}`,
              method: 'get'
            }).then(({ data }) => {
              if (data && data.code === 0) {
                this.dataForm = data.shops
                if (data.shops.workTime) {
                  this.dataForm.workTime = data.shops.workTime.split('-')
                }
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
              this.dataForm.workTime = this.dataForm.workTime[0] + '-' + this.dataForm.workTime[1]
              this.$http({
                url: `/mall/shops/myUpdate`,
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
