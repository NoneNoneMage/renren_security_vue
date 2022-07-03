<template>
    <div class="restaurant-manage" :key="'restaurant-' + restaurant.id">
        <restaurant-manage-directory class="left" :restaurant="restaurant" @selectedFolderTrigger="selectedFolderTrigger"/>
        <restaurant-manage-goods class="right" ref="goodsList" :restaurant="restaurant" :selectedfolder="selectedFolder"/>
    </div>
</template>

<script>
  import RestaurantManageDirectory from './restaurant-manage-directory.vue'
  import RestaurantManageGoods from './restaurant-manage-goods.vue'
  export default {
    data () {
      return {
        restaurant: {
          id: this.$route.query.id || 0,
          name: this.$route.query.name || ''
        },
        selectedFolder: {
          id: 0
        }
      }
    },
    activated () {
      if (this.$route.query.id !== undefined) {
        this.restaurant.id = this.$route.query.id
      }
      console.log('active ', this.restaurant.id)
    },
    components: {
      RestaurantManageDirectory,
      RestaurantManageGoods
    },
    created () {
    },
    mounted () {
    },
    methods: {
      selectedFolderTrigger (id) {
        console.log('manage ', id)
        // let tmpFolder = {id}
        // this.selectedFolder = tmpFolder
        this.$refs.goodsList.getDataList(id)
      }
    }
  }
</script>
<style lang="scss" scoped>
    .restaurant-manage {
        display: flex;
        .left {
            // flex: 0 0 100px;
            width: 15%;
            padding-right: 10px;
        }
        .right {
            flex: 1;
        }
    }
</style>
