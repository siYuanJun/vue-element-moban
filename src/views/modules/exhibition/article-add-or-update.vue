<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : !disabled ? '修改' : '查看'"
    :close-on-click-modal="false"
    :visible.sync="visible" width="60%">
    <el-form inline :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="所属栏目" prop="navName">
        <el-popover
          ref="navListPopover"
          placement="top-start"
          trigger="click">
          <el-tree
            :data="navList"
            :props="navListTreeProps"
            node-key="id"
            ref="navListTree"
            @current-change="navListTreeCurrentChangeHandle"
            :default-expand-all="true"
            :highlight-current="true"
            :expand-on-click-node="false">
          </el-tree>
        </el-popover>


        <el-input v-model="dataForm.navName" :disabled="disabled" v-popover:navListPopover :readonly="true" placeholder="所属栏目" class="width430"></el-input>
      </el-form-item>
      <el-form-item label="标题" prop="title">
        <el-input v-model="dataForm.title" :disabled="disabled" placeholder="标题" class="width430"></el-input>
      </el-form-item>

      <el-form-item label="缩略图" prop="image">
        <el-img v-model="dataForm.image" :disabled="disabled" class="width430"></el-img>
      </el-form-item>

      <el-form-item label="logo" prop="logo">
        <el-img v-model="dataForm.logo" :disabled="disabled" class="width430"></el-img>
      </el-form-item>

      <el-form-item label="副标题" prop="subtitle">
        <el-input v-model="dataForm.subtitle" :disabled="disabled" placeholder="副标题" class="width430"></el-input>
      </el-form-item>


      <el-form-item label="链接地址" prop="link">
        <el-input v-model="dataForm.link" :disabled="disabled" placeholder="链接" class="width430"></el-input>
      </el-form-item>

      <el-form-item label="来源" prop="befrom">
        <el-input v-model="dataForm.befrom" :disabled="disabled" placeholder="来源" class="width430"></el-input>
      </el-form-item>

      <el-form-item label="作者" prop="author">
        <el-input v-model="dataForm.author" :disabled="disabled" placeholder="作者" class="width430"></el-input>
      </el-form-item>

      <el-form-item label="权重" prop="weight">
        <el-input-number v-model="dataForm.weight" :disabled="disabled" controls-position="right" :min="0" label="权重（越大越靠前）" class="width430"></el-input-number>
      </el-form-item>

      <el-form-item label="关键词" prop="seoKeywords">
        <el-input v-model="dataForm.seoKeywords" :disabled="disabled" placeholder="关键词" class="width430"></el-input>
      </el-form-item>

      <el-form-item label="描述摘要" prop="seoDescription">
        <el-input type="textarea"  v-model="dataForm.seoDescription" :disabled="disabled" placeholder="用于SEO优化" class="width952"></el-input>
      </el-form-item>

     <!-- <el-form-item label="发布时间" prop="createDate">
        <el-input v-model="dataForm.createDate" :disabled="disabled" placeholder="创建时间" class="width430"></el-input>
      </el-form-item>-->

      <el-form-item label="推荐位" prop="posidStr" >
        <el-checkbox-group v-model="dataForm.posidStr" :disabled="disabled" class="width430"  @change="setPosid">
          <el-checkbox label="1">首页</el-checkbox>
          <el-checkbox label="2">栏目页</el-checkbox>
        </el-checkbox-group>
      </el-form-item>
      <el-form-item label="内容类型" prop="type"  class="width430">
        <el-radio v-model="dataForm.type" :disabled="disabled" label="1">新闻</el-radio>
        <el-radio v-model="dataForm.type" :disabled="disabled" label="2">报告</el-radio>
        <el-radio v-model="dataForm.type" :disabled="disabled" label="3">其他</el-radio>
      </el-form-item>

      <el-form-item label="是否发布" prop="isdeploy"  class="width430">
        <el-radio v-model="dataForm.isdeploy" :disabled="disabled" label="0">否</el-radio>
        <el-radio v-model="dataForm.isdeploy" :disabled="disabled" label="1">是</el-radio>
      </el-form-item>

      <el-form-item label="关联新闻"  prop="searContent" class="width952">
        <el-select @change="setAticleIds" size="medium" @focus="clear"
          v-model="searContent"
          multiple
          filterable
          remote
          reserve-keyword
          placeholder="请输入关键词"
          :remote-method="remoteMethod"
          :disabled="disabled"
          :loading="loading">
          <el-option
            v-for="item in dataList"
            :key="item.id"
            :label="item.title"
            :value="item.id"
          >
          </el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="文章内容" prop="content">
        <ueditor v-model="dataForm.content" :disabled="disabled" placeholder="文章内容"></ueditor>
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
        dataList: [],
        loading: false,
        searContent: [],
        isfirst: true,
        dataForm: {
          id: 0,
          nav: '',
          navName: '',
          title: '',
          subtitle: '',
          logo: '',
          befrom: '',
          profile: '',
          link: '',
          image: '',
          seoKeywords: '',
          seoDescription: '',
          weight: '',
          hits: '',
          posid: '',
          posidStr: [],
          createBy: '',
          createDate: '',
          updateBy: '',
          updateDate: '',
          remarks: '',
          delFlag: '',
          author: '',
          type:'1',
          isdeploy: '1',
          content: '',
          aticleIds: ''
        },
        dataRule: {
          title: [
            {required: true, message: '名称不能为空', trigger: 'blur'}
          ],
          navName: [
            {required: true, message: '所属栏目不能为空', trigger: 'blur'}
          ],
          link: [
            {required: true, message: '链接地址不能为空', trigger: 'blur'}
          ],
          befrom: [
            {required: true, message: '来源不能为空', trigger: 'blur'}
          ],
          author: [
            {required: true, message: '作者不能为空', trigger: 'blur'}
          ]
        },
        navList: [],
        navListTreeProps: {
          label: 'name',
          children: 'children'
        }
      }
    },
    methods: {
      init (id, disabled) {
        this.disabled = disabled
        this.isfirst = true
        this.dataForm.id = id || 0
        this.$http({
          url: '/exhibition/nav/select',
          method: 'get'
        }).then(({ data }) => {
          this.navList = this.treeDataTranslate(data.navList, 'id')
        }).then(() => {
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
          })
        }).then(() => {
          if (!this.dataForm.id) {
            // 新增
            this.navListTreeSetCurrentNode()
          } else {
            // 修改
            this.$http({
              url: `/exhibition/article/info/${this.dataForm.id}`,
              method: 'get'
            }).then(({ data }) => {
              let article = data.article
              this.dataForm.id = article.id
              this.dataForm.nav = article.nav
              this.dataForm.title = article.title
              this.dataForm.image = article.image
              this.dataForm.logo = article.logo
              this.dataForm.subtitle = article.subtitle
              this.dataForm.link = article.link
              this.dataForm.befrom = article.befrom
              this.dataForm.author = article.author
              this.dataForm.weight = article.weight
              this.dataForm.seoKeywords = article.seoKeywords
              this.dataForm.seoDescription = article.seoDescription
              this.dataForm.posidStr = article.posid.split(',')
              this.dataForm.posid = article.posid
              this.dataForm.type = article.type
              this.dataForm.isdeploy = article.isdeploy
              if (article.searContent != null) {
                this.searContent = article.searContent.split(',')
              }
              this.dataForm.aticleIds = article.aticleIds
              this.dataForm.content = article.content
              this.navListTreeSetCurrentNode('mod')
            })
          }
        })
      },
      setAticleIds (id) {
        this.dataForm.aticleIds = this.searContent.join(',')
      },
      clear () {
        if (this.isfirst) {
          this.searContent = []
          this.isfirst = false
        }
      },
      remoteMethod (query) {
        this.loading = true
        this.$http({
          url: '/exhibition/article/list',
          method: 'get',
          params: {
            'page': 1,
            'limit': 10,
            'searContent': query
          }
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.records
          } else {
            this.dataList = []
          }
          this.loading = false
        })
      },
      setPosid () {
        var that = this
        this.dataForm.posid = that.dataForm.posidStr.join(',')
      },
      // 菜单树选中
      navListTreeCurrentChangeHandle (data, node) {
        this.dataForm.nav = data.id
        this.dataForm.navName = data.name

        this.$refs[`navListPopover`].doClose()
      },
      // 菜单树设置当前选中节点
      navListTreeSetCurrentNode (type) {
        if (type === 'mod') {
          this.$refs.navListTree.setCurrentKey(this.dataForm.nav)
          this.dataForm.navName = (this.$refs.navListTree.getCurrentNode() || {})['name']
        }
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm']
          .validate((valid) => {
            if (valid) {
              this.$http({
                url: `/exhibition/article/${!this.dataForm.id ? 'save' : 'update'}`,
                method: 'post',
                data: this.dataForm
              }).then(({data}) => {
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
