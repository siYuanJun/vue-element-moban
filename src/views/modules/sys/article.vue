<template>
  <div class="mod-dict">
    <el-form :inline="true" :model="searchForm" @keyup.enter.native="getArticleDataList()">
      <el-form-item>
        <el-input v-model="searchForm.name" placeholder="文章名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getArticleDataList()">查询</el-button>
        <el-button v-if="isAuth('sys:article:save')" type="primary" @click="addOrUpdateArticleHandle()">新增</el-button>
        <el-button v-if="isAuth('sys:article:delete')" type="danger" @click="deleteArticleHandle()"
                   :disabled="articleDataListSelections.length <= 0">批量删除
        </el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="articleDataList"
      border
      @selection-change="selectionChangeArticleHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="title"
        header-align="center"
        align="center"
        label="标题">
      </el-table-column>
      <el-table-column
        prop="author"
        header-align="center"
        align="center"
        label="作者">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间">
      </el-table-column>
      <el-table-column
        prop="image"
        header-align="center"
        align="center"
        label="图片信息">
      </el-table-column>
      <el-table-column
        prop="content"
        header-align="center"
        align="center"
        label="内容">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('sys:article:update')" type="text" size="small"
                     @click="addOrUpdateArticleHandle(scope.row.id)">修改
          </el-button>
          <el-button v-if="isAuth('sys:article:delete')" type="text" size="small"
                     @click="deleteArticleHandle(scope.row.id)">删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeArticleHandle"
      @current-change="currentChangeArticleHandle"
      :current-page="articlePageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="articlePageSize"
      :total="articleTotalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <article-add-or-update v-if="articleVisible" ref="articleAddOrUpdate"
                        @refreshDataList="getArticleDataList"></article-add-or-update>
  </div>
</template>

<script>
  import ArticleAddOrUpdate from './dict-add-or-update'

  export default {
    data () {
      return {
        searchForm: {
          name: ''
        },
        articleDataList: [],
        articlePageIndex: 1,
        articlePageSize: 10,
        articleTotalPage: 0,
        articleDataListSelections: [],
        articleVisible: false,
        groupId: '',
        code: ''
      }
    },
    components: {
      ArticleAddOrUpdate
    },
    mounted () {
      this.getArticleDataList()
    },
    methods: {
      // 获取数据列表
      getArticleDataList (title) {
        if (title) {
          this.title = title
        }
        this.$http({
          url: '/sys/article/list',
          method: 'get',
          params: {
            'page': this.articlePageIndex,
            'limit': this.articlePageSize,
            'title': this.searchForm.name
          }
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.articleDataList = data.page.records
            this.articleTotalPage = data.page.total
          } else {
            this.articleDataList = []
            this.articleTotalPage = 0
          }
        })
      },
      // 每页数
      sizeChangeArticleHandle (val) {
        this.articlePageSize = val
        this.articlePageIndex = 1
        this.getArticleDataList()
      },
      // 当前页
      currentChangeArticleHandle (val) {
        this.articlePageIndex = val
        this.getArticleDataList()
      },
      // 多选
      selectionChangeArticleHandle (val) {
        this.articleDataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateArticleHandle (id) {
        this.articleVisible = true
        this.$nextTick(() => {
          this.$refs.articleAddOrUpdate.init(id, this.groupId)
        })
      },
      // 删除
      deleteArticleHandle (id) {
        let ids = id ? [id] : this.articleDataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[删除]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: '/sys/article/delete',
            method: 'post',
            data: ids
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500
              })
              this.getArticleDataList()
            }
          })
        }).catch(() => {
        })
      }
    }
  }
</script>
