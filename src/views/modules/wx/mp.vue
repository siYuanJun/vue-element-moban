<template>
  <div class="mod-demo-echarts">
    <el-row :gutter="20">
      <el-col :md="20">
        <wechat
          :data="data"
          @pubData="pubData"
          @delData="delData"
          @saveData="saveData"/>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  import wechat from '@/components/wechat'

  export default {
    data () {
      return {
        data: {
          button: []
        }
      }
    },
    components: {
      wechat
    },
    activated () {
      this.getWxMenu()
    },
    methods: {
      getWxMenu () {
        this.$http({
          url: `/sys/wx/menu/getMenuByDb`,
          method: 'get'
        }).then(({ data }) => {
          if (data && data.code === 0) {
            if (data.menu) {
              this.data = JSON.parse(data.menu)
            } else {
              this.data = { button: [] }
            }
          }
        })
      },
      // 保存菜单到数据库
      saveData (form) {
        this.$http({
          url: `/sys/wx/menu/menuCreateByDb`,
          method: 'post',
          data: form
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: '保存成功',
              type: 'success',
              duration: 1500
            })
          }
        })
      },
      // 发布菜单到微信
      pubData () {
        this.$http({
          url: `/sys/wx/menu/push`,
          method: 'post',
          data: { 'buttons': this.data.button }
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: '发布成功',
              type: 'success',
              duration: 1500
            })
          }
        })
      },
      // 删除数据库中保存的菜单
      delData () {
        this.$http({
          url: `/sys/wx/menu/menuDeleteByDb`,
          method: 'post'
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: '删除成功',
              type: 'success',
              duration: 1500
            })
          }
        })
      }
    }
  }
</script>
