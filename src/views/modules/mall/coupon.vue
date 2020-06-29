<template>
  <div class="mod-coupon">
    <el-form :inline="true" :model="searchForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="searchForm.name" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('mall:coupon:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('mall:coupon:delete')" type="danger" @click="deleteHandle()"
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
        prop="name"
        header-align="center"
        align="center"
        label="优惠券名称">
      </el-table-column>
      <el-table-column
        prop="picUrl"
        header-align="center"
        align="center"
        label="优惠券图片">
        <template slot-scope="scope">
          <img style="height: 30%;width: 30%" v-if="scope.row.picUrl" @click="openImg(scope.row.picUrl)"
               :src="scope.row.picUrl"/>
        </template>
      </el-table-column>
      <el-table-column
        prop="type"
        header-align="center"
        align="center"
        label="优惠券类型">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 1" size="small" type="danger">折扣</el-tag>
          <el-tag v-else-if="scope.row.status === 2" size="small" type="success">满减</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="minPrice"
        header-align="center"
        align="center"
        label="最低消费金额">
      </el-table-column>
      <el-table-column
        prop="beginTime"
        header-align="center"
        align="center"
        label="有效期开始时间">
      </el-table-column>
      <el-table-column
        prop="endTime"
        header-align="center"
        align="center"
        label="有效期结束时间">
      </el-table-column>
      <el-table-column
        prop="totalCount"
        header-align="center"
        align="center"
        label="投放总数量">
      </el-table-column>
      <el-table-column
        prop="sendCount"
        header-align="center"
        align="center"
        label="已发数量">
      </el-table-column>
      <el-table-column
        prop="limitGoods"
        header-align="center"
        align="center"
        label="是否指定商品">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.limitGoods === 0" size="small" type="danger">否</el-tag>
          <el-tag v-else-if="scope.row.limitGoods === 1" size="small" type="success">是</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="userNum"
        header-align="center"
        align="center"
        label="每人限领数量">
      </el-table-column>
      <el-table-column
        width="130px"
        prop="status"
        header-align="center"
        align="center"
        label="状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 0" size="small" type="danger">禁用</el-tag>
          <el-tag v-else-if="scope.row.status === 1" size="small" type="success">正常</el-tag>
          <el-button v-if="isAuth('mall:coupon:sendUser') && scope.row.status === 1" type="primary"
                     size="mini" @click="sendUser(scope.row.id)">发放
          </el-button>
        </template>
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
          <el-button v-if="isAuth('mall:coupon:info')" type="text" size="small" @click="showDetails(scope.row.id)">查看
          </el-button>
          <el-button v-if="isAuth('mall:coupon:update')" type="text" size="small"
                     @click="addOrUpdateHandle(scope.row.id)">修改
          </el-button>
          <el-button v-if="isAuth('mall:coupon:delete')" type="text" size="small" @click="deleteHandle(scope.row.id)">
            删除
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
    <el-dialog
      title="发放优惠券"
      :visible.sync="dialogVisible"
      :close-on-click-modal="false">
      <el-form :rules="dataRule" :model="userCoupon" ref="userCoupon" label-width="80px">
        <el-form-item label="发放用户" prop="userIds">
          <el-select v-model="userCoupon.userIds" multiple clearable filterable placeholder="请选择" style="width: 100%;">
            <el-option
              v-for="mallUser in mallUserList"
              :key="mallUser.id"
              :label="mallUser.userName"
              :value="mallUser.id">
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">取消</el-button>
          <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
    </el-dialog>
  </div>
</template>

<script>
  import AddOrUpdate from './coupon-add-or-update'

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
        dataListSelections: [],
        addOrUpdateVisible: false,
        dataRule: {
          userIds: [
            {
              required: true,
              message: '发放用户不能为空',
              trigger: 'blur'
            }
          ]
        },
        userCoupon: {
          id: '',
          userIds: []
        },
        dialogVisible: false,
        mallUserList: []
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
      this.$http({
        url: '/mall/user/queryAll',
        method: 'get'
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.mallUserList = data.list
        }
      })
    },
    methods: {
      // 给会员发放优惠券
      sendUser (id) {
        this.dialogVisible = true
        this.userCoupon = {
          id: id,
          userIds: []
        }
      },
      // 获取数据列表
      getDataList () {
        this.$http({
          url: '/mall/coupon/list',
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
      // 发货提交
      dataFormSubmit () {
        this.$refs['userCoupon']
          .validate((valid) => {
            if (valid) {
              this.$http({
                url: '/mall/coupon/sendUser',
                method: 'post',
                data: this.userCoupon
              }).then(({ data }) => {
                if (data && data.code === 0) {
                  this.$message({
                    message: '操作成功',
                    type: 'success',
                    duration: 1500
                  })
                  this.dialogVisible = false
                  this.getDataList()
                }
              })
            }
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
            url: '/mall/coupon/delete',
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
