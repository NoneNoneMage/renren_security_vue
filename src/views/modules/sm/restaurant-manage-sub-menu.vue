<template>
    <el-menu-item :index="folder.id + ''" @click.native="selectFolder(folder.id)">
    <!-- <icon-svg :name="folder.icon || ''"></icon-svg> -->
    <span>{{ folder.name }}</span>
    <icon-svg name="bianji" class="first" @click.native="editFolder(folder.id)"></icon-svg>
    <icon-svg name="delete" @click.native="deleteFolder(folder.id, folder.name)"></icon-svg>
    </el-menu-item>
  
</template>

<script>
  export default {
    data () {
      return {
        addOrUpdateVisible: false
      }
    },
    props: {
      folder: {
        type: Object,
        required: true
      },
      restaurant: {
        type: Object
      }
    },
    components: {
    },
    methods: {
      selectFolder (id) {
        console.log('sub-folder ', id)
        this.$emit('selectFolder', id)
      },
      editFolder (id) {
        this.$emit('editFolder', id)
      },
      deleteFolder (id, name) {
        this.$emit('deleteFolder', id, name)
      },
      // 通过menuId与动态(菜单)路由进行匹配跳转至指定路由
      gotoRouteHandle (folder) {
      }
    }
  }
</script>
<style lang="scss" scoped>
    .first {
        margin-left: 10px;
    }
    
</style>
