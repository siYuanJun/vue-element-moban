<template>
  <div class="mod-shopsgoods">
    <el-form :inline="true" :model="searchForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-select v-model="searchForm.shopsId" clearable filterable placeholder="店铺名称" class="width185">
          <el-option
            v-for="shops in shopsList"
            :key="shops.id"
            :label="shops.name"
            :value="shops.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-select v-model="searchForm.shopsCategoryId" clearable filterable placeholder="商品分类" class="width185">
          <el-option
            v-for="shopsCategory in shopsCategoryList"
            :key="shopsCategory.id"
            :label="shopsCategory.name"
            :value="shopsCategory.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('mall:shopsgoods:save')" type="primary" @click="addOrUpdateHandle()">添加</el-button>
        <el-button v-if="isAuth('mall:shopsgoods:delete')" type="danger" @click="deleteHandle()"
                   :disabled="dataListSelections.length <= 0">批量删除
        </el-button>
      </el-form-item>
    </el-form>
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
        prop="shopsName"
        header-align="center"
        align="center"
        label="店铺名称">
      </el-table-column>
      <el-table-column
        prop="shopsCategoryName"
        header-align="center"
        align="center"
        label="商品分类">
      </el-table-column>
      <el-table-column
        prop="goodsName"
        header-align="center"
        align="center"
        label="商品名称">
      </el-table-column>
      <el-table-column
        prop="listPicUrl"
        header-align="center"
        align="center"
        label="商品图片">
        <template slot-scope="scope">
          <img style="height: 30%;width: 30%" v-if="scope.row.listPicUrl" @click="openImg(scope.row.listPicUrl)"
               :src="scope.row.listPicUrl"/>
        </template>
      </el-table-column>
      <el-table-column
        prop="retailPrice"
        header-align="center"
        align="center"
        label="零售价格">
      </el-table-column>
      <el-table-column
        prop="goodsNumber"
        header-align="center"
        align="center"
        label="店铺商品库存">
      </el-table-column>
      <el-table-column
        prop="sort"
        header-align="center"
        align="center"
        label="排序">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('mall:shopsgoods:info')" type="text" size="small" @click="showDetails(scope.row.id)">
            查看
          </el-button>
          <el-button v-if="isAuth('mall:shopsgoods:update')" type="text" size="small"
                     @click="addOrUpdateHandle(scope.row.id)">修改
          </el-button>
          <el-button v-if="isAuth('mall:shopsgoods:delete')" type="text" size="small"
                     @click="deleteHandle(scope.row.id)">删除
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
  import AddOrUpdate from './shopsgoods-add-or-update'

  export default {
    data () {
      return {
        searchForm: {
          shopsId: '',
          shopsCategoryId: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListSelections: [],
        addOrUpdateVisible: false,
        shopsCategoryList: [],
        shopsList: []
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
      this.$http({
        url: '/mall/shopscategory/queryMyShopCategory',
        method: 'get'
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.shopsCategoryList = data.list
        }
      })
      this.$http({
        url: '/mall/shops/queryMyShop',
        method: 'get'
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.shopsList = data.list
        }
      })
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.$http({
          url: '/mall/shopsgoods/list',
          method: 'get',
          params: {
            'page': this.pageIndex,
            'limit': this.pageSize,
            'shopsCategoryId': this.searchForm.shopsCategoryId,
            'shopsId': this.searchForm.shopsId
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
      // 删除
      deleteHandle (id) {
        let ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[删除]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: '/mall/shopsgoods/delete',
            method: 'post',
            data: ids
          }).then(({ data }) => {
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
