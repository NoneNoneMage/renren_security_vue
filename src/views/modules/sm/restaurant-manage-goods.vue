<template>
  <div class="mod-restaurant-manage-goods">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.name" placeholder="菜名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button disabled>{{restaurant.name}}</el-button>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('sm:restaurant:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('sm:restaurant:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        width="60"
        label="ID">
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        width="80"
        label="菜名">
      </el-table-column>
      <el-table-column
        prop="avatarUrl"
        header-align="center"
        align="center"
        width="80"
        label="菜品缩略图">
        <template slot-scope="scope">
          <viewer>
            <img v-if="scope.row.avatarUrl != ''" style="width:50px;height:50px;border:none;" :src="scope.row.avatarUrl"/>
          </viewer>
        </template>
      </el-table-column>
      <el-table-column
        prop="price"
        header-align="center"
        align="center"
         width="80"
        label="价格">
      </el-table-column>
      <el-table-column
        prop="oldPrice"
        header-align="center"
        align="center"
         width="80"
        label="原价">
      </el-table-column>
      <el-table-column
        prop="description"
        header-align="center"
        align="center"
        width="80"
        label="概述">
      </el-table-column>
      <el-table-column
        prop="info"
        header-align="center"
        align="center"
        width="180"
        label="详述">
      </el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        width="80"
        label="状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 0" size="small" type="danger">禁用</el-tag>
          <el-tag v-else size="small">正常</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        width="100"
        label="创建时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="100"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('sm:restaurant:manageGoods')" type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button v-if="isAuth('sm:restaurant:manageGoods')" type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
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
    <restaurant-goods-add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshGoodsList="getDataList"></restaurant-goods-add-or-update>
  </div>
</template>

<script>
  import RestaurantGoodsAddOrUpdate from './restaurant-goods-add-or-update.vue'
  export default {
    data () {
      return {
        dataForm: {
          name: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        selectedFolderId: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    props: {
      restaurant: {
        type: Object,
        id: {
          type: Number
        }
      },
      selectedFolder: {
        type: Object,
        id: {
          type: Number
        }
      }
    },
    components: {
      RestaurantGoodsAddOrUpdate
    },
    watch: {
      selectedFolder: {
        deep: true,  // 深度监听
        handler (newVal, oldVal) {
          debugger
          console.log(newVal, oldVal)
        }
      }
    },
    activated () {
      this.getDataList(this.selectedFolder && this.selectedFolder.id)
    },
    methods: {
      // 获取数据列表
      getDataList (folderId) {
        if (folderId === undefined && this.selectedFolderId === undefined) {
          return
        } else if (folderId !== undefined) {
          this.selectedFolderId = folderId
        }
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/sm/goods/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'name': this.dataForm.name,
            'folderId': this.selectedFolderId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
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
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id, this.selectedFolderId)
        })
      },
      // 删除
      deleteHandle (id) {
        var goodsIds = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${goodsIds.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/sm/goods/delete'),
            method: 'post',
            data: this.$http.adornData(goodsIds, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }).catch(() => {})
      }
    }
  }
</script>
