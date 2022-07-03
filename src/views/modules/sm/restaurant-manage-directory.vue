<template :style="{'background':'black'}">
  <aside>
    <div>
      <el-menu
        :default-active="folderSelected || 'home'"
        :active="folderSelected"
        v-loading="dataListLoading"
        :collapseTransition="false">
        <el-menu-item index="home" @click="addOrUpdateDirectoryHandle()" class="addButton">
          <icon-svg name="add"></icon-svg>
          <span slot="title">添加目录 </span>
        </el-menu-item>
        <restaurant-manage-sub-menu
          v-for="folder in dataList"
          :key="folder.id"
          :folder="folder"
          :restaurant="restaurant"
          @selectFolder="selectedFolder"
          @editFolder="addOrUpdateDirectoryHandle"
          @deleteFolder="deleteHandle">
        </restaurant-manage-sub-menu>
      </el-menu>
      <restaurant-directory-add-or-update  v-if="addOrUpdateVisible" ref="addOrUpdateDirectory"  @refreshFolderList="getDataList"></restaurant-directory-add-or-update>
    </div>
  </aside>
</template>

<script>
  import RestaurantDirectoryAddOrUpdate from './restaurant-directory-add-or-update.vue'
  import RestaurantManageSubMenu from './restaurant-manage-sub-menu.vue'
  export default {
    data () {
      return {
        addOrUpdateVisible: false,
        dataListLoading: false,
        dataList: [],
        folderSelected: undefined
      }
    },
    props: {
      restaurant: {
        type: Object,
        id: {
          type: Number
        }
      }
    },
    computed: {
    //   folderActiveName: {
    //     get () { return this.$store.state.common.folderActiveName },
    //     set (val) { this.$store.commit('common/updateFolderActiveName', val) }
    //   }
    },
    components: {
      RestaurantDirectoryAddOrUpdate,
      RestaurantManageSubMenu
    },
    activated () {
      this.getDataList()
    },
    created () {
    },
    methods: {
      // 删除
      deleteHandle (id, name) {
        this.$confirm(`确定对[id=${id},name=${name}]进行['删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl(`/sm/folder/delete/${id}`),
            method: 'post',
            data: this.$http.adornData()
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
      },
      addOrUpdateDirectoryHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdateDirectory.init(id, this.restaurant.id)
        })
      },
      // 更新目录列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl(`/sm/folder/list/${this.restaurant.id}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
        //   debugger
          if (data && data.code === 0) {
            this.dataList = data.folders
            if (this.dataList.length > 0) {
              this.selectedFolder(this.dataList[0].id)
            }
          } else {
            this.dataList = []
          }
          this.dataListLoading = false
          console.log(this.dataList)
        })
      },
      selectedFolder (id) {
        this.folderSelected = id + ''
        console.log('manage-directory', 'edit folder ', id)
        this.$emit('selectedFolderTrigger', id)
      }
    }
  }
</script>
<style scoped>
    .addButton {
        /* text-align: right; */
        /* margin-bottom: -26px; */
        background-color: #17B3A3;
        border-radius: 5px;
        color: #fff;
    }
</style>
