<template>
  <div class="mod-article">
    <el-form :inline="true" :model="searchForm" ref="searchForm"  @keyup.enter.native="getDataList()">
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
<!--        <el-select v-model="searchForm.navName" placeholder="请选择">-->
<!--          <el-option-->
<!--            v-for="item in dataList"-->
<!--            :key="item.navName"-->
<!--            :label="item.id"-->
<!--            :value="item.navName">-->
<!--          </el-option>-->
<!--        </el-select>-->
<!--        <el-input v-model="searchForm.navName" placeholder="所属栏目"></el-input>-->
        <el-input v-model="searchForm.navName" v-popover:navListPopover  placeholder="所属栏目"></el-input>
      </el-form-item>

      <el-form-item prop="title">
        <el-input v-model="searchForm.title" placeholder="标题"></el-input>
      </el-form-item>

      <el-form-item label="发布日期：" prop="rangeDate">
        <el-date-picker
          v-model="rangeDate"
          type="daterange"
          align="center"
          unlink-panels
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          :default-time="['00:00:00', '23:59:59']">
        </el-date-picker>
      </el-form-item>


      <el-form-item label="是否发布：" prop="isdeploy">
        <el-radio v-model="searchForm.isdeploy" label="0">否</el-radio>
        <el-radio v-model="searchForm.isdeploy" label="1">是</el-radio>
      </el-form-item>

      <el-form-item label="是否删除：" prop="delFlag">
        <el-radio v-model="searchForm.delFlag" label="0">否</el-radio>
        <el-radio v-model="searchForm.delFlag" label="1">是</el-radio>
      </el-form-item>
      <el-button @click="getDataList()">查询</el-button>
      <el-button @click="resetForm('searchForm')">重置</el-button>

      <el-row>
        <el-col :span="24">
          <el-form-item>
            <el-button v-if="isAuth('exhibition:article:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
           <el-button  v-if="isAuth('exhibition:article:save')"  type="info"  icon="el-icon-upload2" size="small"  @click="handleImport">导入</el-button>
            <el-button v-if="isAuth('exhibition:article:delete')" type="danger" @click="deleteHandle(null,1)" :disabled="dataListSelections.length <= 0">批量删除</el-button>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>

    <!-- 用户导入对话框 -->
    <el-dialog :title="upload.title" :visible.sync="upload.open" width="400px" append-to-body>
      <el-upload
        ref="upload"
        :limit="1"
        accept=".xlsx, .xls"
        :action="upload.url"
        :disabled="upload.isUploading"
        :on-progress="handleFileUploadProgress"
        :on-success="onSuccess"
        :auto-upload="false"
        drag
      >
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">
          将文件拖到此处，或
          <em>点击上传</em>
        </div>
        <div class="el-upload__tip" slot="tip">
          <el-link type="info" style="font-size:12px;color:blue;" @click="outputExcel">下载模板</el-link>
        </div>
        <div class="el-upload__tip" style="color:red" slot="tip">提示：仅允许导入“xls”或“xlsx”格式文件！</div>
      </el-upload>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitFileForm">确 定</el-button>
        <el-button @click="upload.open = false">取 消</el-button>
      </div>
    </el-dialog>


    <el-table
      :data="dataList"
      border
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="navName"
        header-align="center"
        align="center"
        label="所属栏目">
      </el-table-column>
      <el-table-column
        prop="title"
        header-align="center"
        align="center"
        label="标题">
      </el-table-column>
      <el-table-column
        prop="posid"
        header-align="center"
        align="center"
        label="推荐位">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.posid == '1,2' || scope.row.posid == '2,1'" size="small" type="danger">首页,栏目</el-tag>
          <el-tag v-else-if="scope.row.posid == '1'" size="small" type="success">首页</el-tag>
          <el-tag v-else-if="scope.row.posid == '2'" size="small" type="success">栏目</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="isdeploy"
        header-align="center"
        align="center"
        label="是否发布">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.isdeploy == '0'" size="small" type="danger">否</el-tag>
          <el-tag v-else-if="scope.row.isdeploy == '1'" size="small" type="success">是</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="deployDate"
        header-align="center"
        align="center"
        label="发布日期">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('exhibition:article:info')" type="text" size="small" @click="showDetails(scope.row.id)">查看</el-button>
          <el-button v-if="isAuth('exhibition:article:update')" type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button v-if="isAuth('exhibition:article:delete')" type="text" size="small" @click="deleteHandle(scope.row.id,1)">删除</el-button>
          <el-button v-if="isAuth('exhibition:article:delete')" type="text" size="small" @click="deleteHandle(scope.row.id,2)">彻底删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './article-add-or-update'

  export default {
    data () {
      return {
        // 用户导入参数
        upload: {
          // 是否显示弹出层（用户导入）
          open: false,
          // 弹出层标题（用户导入）
          title: '',
          // 是否禁用上传
          isUploading: false,
          // 是否更新已经存在的用户数据
          updateSupport: 0,
          // 上传的地址
          url: this.$http.BASE_URL + `/exhibition/article/upload?token=${this.$cookie.get('token')}`
        },
        rangeDate: [],
        searchForm: {
          title: '',
          navName: '',
          nav: '',
          isdeploy: '',
          delFlag: '',
          beginDate: '',
          endDate: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListSelections: [],
        addOrUpdateVisible: false,

        navList: [],
        navListTreeProps: {
          label: 'name',
          children: 'children'
        }
      }
    },
    watch: {
      rangeDate: function (newVal, oldVal) {
        if (newVal !== null) {
          this.searchForm.beginDate = newVal[0]
          this.searchForm.endDate = newVal[1]
        } else {
          this.searchForm.beginDate = null
          this.searchForm.endDate = null
        }
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
      this.getTreeData()
    },
    methods: {
      // 文件上传中处理
      handleFileUploadProgress (event, file, fileList) {
        this.upload.isUploading = true
      },
      // 提交上传文件
      submitFileForm () {
        this.$refs.upload.submit()
      },
      /** 导入按钮操作 */
      handleImport () {
        this.upload.title = '用户导入'
        this.upload.open = true
      },
      onSuccess (response, file, fileList) {
        this.upload.open = false
        this.upload.isUploading = false
        this.$refs.upload.clearFiles()
        this.$alert(response.msg, '导入结果', { dangerouslyUseHTMLString: true })
        this.getDataList()
      },
      resetForm (formName) {
        this.searchForm.beginDate = ''
        this.searchForm.endDate = ''
        this.searchForm.deployDate = ''
        this.rangeDate = ''
        this.searchForm.nav = ''
        this.$refs[formName].resetFields()
        this.getDataList()
      },
      getTreeData () {
        this.$http({
          url: '/exhibition/nav/select',
          method: 'get'
        }).then(({ data }) => {
          this.navList = this.treeDataTranslate(data.navList, 'id')
          console.log(this.navList)
        })
      },
      // 菜单树选中
      navListTreeCurrentChangeHandle (data, node) {
        this.searchForm.nav = data.id
        this.searchForm.navName = data.name

        this.$refs[`navListPopover`].doClose()
      },
      // 获取数据列表
      getDataList () {
        this.$http({
          url: '/exhibition/article/list',
          method: 'get',
          params: {
            'page': this.pageIndex,
            'limit': this.pageSize,
            'title': this.searchForm.title,
            'nav': this.searchForm.nav,
            'isdeploy': this.searchForm.isdeploy,
            'delFlag': this.searchForm.delFlag,
            'beginDate': this.searchForm.beginDate,
            'endDate': this.searchForm.endDate
          }
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.records
            console.log(this.dataList)
            this.totalPage = data.page.total
          } else {
            this.dataList = []
            this.totalPage = 0
          }
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 查看详情
      showDetails (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id, true)
        })
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      outputExcel () {
        this.$http({
          url: '/exhibition/article/outputExcel',
          responseType: 'arraybuffer',
          method: 'get'
        }).then(({ data }) => {
          let url = window.URL.createObjectURL(new Blob([data], {type: 'application/vnd.ms-excel;charset=UTF-8'}))
          let link = document.createElement('a')
          link.style.display = 'none'
          link.href = url
          link.setAttribute('download', 'import_model.xlsx')
          document.body.appendChild(link)
          link.click()
          document.body.removeChild(link)
          window.URL.revokeObjectURL(url)
        })
      },
    // 删除
      deleteHandle (id, deltype) {
        let ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[删除]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: '/exhibition/article/delete?deltype=' + deltype,
            method: 'post',
            data: ids
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500
              })
              this.getDataList()
            }
          })
        }).catch(() => {
        })
      }
    }
  }
</script>
