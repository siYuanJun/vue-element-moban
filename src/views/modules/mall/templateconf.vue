<template>
  <div class="mod-templateconf">
    <el-form :inline="true" :model="searchForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="searchForm.name" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      style="width: 100%;">
      <el-table-column
        prop="templateType"
        header-align="center"
        align="center"
        label="模板类型">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.templateType === 0" size="small" type="default">新订单提醒</el-tag>
          <el-tag v-if="scope.row.templateType === 1" size="small" type="success">下单成功通知</el-tag>
          <el-tag v-else-if="scope.row.templateType === 2" size="small" type="danger">订单评价提醒</el-tag>
          <el-tag v-else-if="scope.row.templateType === 3" size="small" type="info">退款通知</el-tag>
          <el-tag v-else-if="scope.row.templateType === 4" size="small">秒杀成功通知</el-tag>
          <el-tag v-else-if="scope.row.templateType === 5" size="small">订单配送通知</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="templateId"
        header-align="center"
        align="center"
        label="推送模板ID">
      </el-table-column>
      <el-table-column
        prop="page"
        header-align="center"
        align="center"
        label="跳转页面">
      </el-table-column>
      <el-table-column
        show-tooltip-when-overflow
        prop="headDesc"
        header-align="center"
        align="center"
        label="消息头部描述">
      </el-table-column>
      <el-table-column
        show-tooltip-when-overflow
        prop="bottomDesc"
        header-align="center"
        align="center"
        label="消息底部描述">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('mall:templateconf:info')" type="text" size="small"
                     @click="showDetails(scope.row.id)">查看
          </el-button>
          <el-button v-if="isAuth('mall:templateconf:update')" type="text" size="small"
                     @click="addOrUpdateHandle(scope.row.id)">修改
          </el-button>
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
  import AddOrUpdate from './templateconf-add-or-update'

  export default {
    data () {
      return {
        searchForm: {
          name: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        addOrUpdateVisible: false
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.$http({
          url: '/mall/templateconf/list',
          method: 'get',
          params: {
            'page': this.pageIndex,
            'limit': this.pageSize,
            'name': this.searchForm.name
          }
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.dataList = data.page.records
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
      }
    }
  }
</script>
