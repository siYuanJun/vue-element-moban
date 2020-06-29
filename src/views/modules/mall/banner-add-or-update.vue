<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : !disabled ? '修改' : '查看'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="80px">
      <el-form-item label="类型" prop="mediaType">
        <el-radio-group v-model="dataForm.mediaType" :disabled="disabled">
          <el-radio :label="0">图片</el-radio>
          <el-radio :label="1">链接</el-radio>
          <el-radio :label="2">文本</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="名称" prop="title">
        <el-input v-model="dataForm.title" :disabled="disabled" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="图片链接" prop="imageUrl" v-if="dataForm.mediaType === 0">
        <el-img v-model="dataForm.imageUrl" :disabled="disabled">
        </el-img>
      </el-form-item>
      <el-form-item label="链接" prop="link" v-if="dataForm.mediaType === 1">
        <el-input v-model="dataForm.link" :disabled="disabled" placeholder="链接"></el-input>
      </el-form-item>
      <el-form-item label="文本内容" prop="content">
        <ueditor v-model="dataForm.content" :disabled="disabled" placeholder="文本内容"></ueditor>
      </el-form-item>
      <el-form-item label="结束时间" prop="endTime">
        <el-date-picker type="datetime" :picker-options="datePicker" v-model="dataForm.endTime" :disabled="disabled"
                        value-format="yyyy-MM-dd HH:mm:ss" placeholder="结束时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="状态" prop="enabled">
        <el-radio-group v-model="dataForm.enabled" :disabled="disabled">
          <el-radio :label="0">禁用</el-radio>
          <el-radio :label="1">正常</el-radio>
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
          mediaType: 0,
          title: '',
          imageUrl: '',
          link: '',
          content: '',
          endTime: '',
          enabled: 0
        },
        dataRule: {
          title: [
            {
              required: true,
              message: '标题不能为空',
              trigger: 'blur'
            }
          ]
        },
        datePicker: this.picker()
      }
    },
    methods: {
      picker () {
        return {
          // 可选时间大于等于当前时间
          disabledDate (time) {
            return time.getTime() < Date.now()
          }
        }
      },
      init (id, disabled) {
        this.disabled = disabled
        this.dataForm.id = id || ''
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: `/mall/banner/info/${this.dataForm.id}`,
              method: 'get'
            }).then(({ data }) => {
              if (data && data.code === 0) {
                this.dataForm = data.banner
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
                url: `/mall/banner/${!this.dataForm.id ? 'save' : 'update'}`,
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
