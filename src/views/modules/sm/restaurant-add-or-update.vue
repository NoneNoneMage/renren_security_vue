<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="门店名" prop="name">
        <el-input v-model="dataForm.name" placeholder="门店名"></el-input>
      </el-form-item>
      <el-form-item label="门店头像" prop="avatarUrl">
        <el-input v-model="dataForm.avatarUrl" placeholder="门店头像"></el-input>
      </el-form-item>
      <el-form-item label="概述" prop="description">
        <el-input v-model="dataForm.description" maxlength="10" placeholder="配送方式"></el-input>
      </el-form-item>
      <el-form-item label="起送价格" prop="minPriceTip">
        <el-input v-model="dataForm.minPriceTip" type="number" placeholder="起送价格"></el-input>
      </el-form-item>
      <el-form-item label="配送价格" prop="shippingFeeTip">
        <el-input v-model="dataForm.shippingFeeTip" type="number" placeholder="配送价格"></el-input>
      </el-form-item>
      <el-form-item label="公告" prop="bulletin">
        <el-input v-model="dataForm.bulletin" type="textarea" placeholder="公告" :rows="5"></el-input>
      </el-form-item>
  
      <el-form-item label="状态" size="mini" prop="status">
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="0">禁用</el-radio>
          <el-radio :label="1">正常</el-radio>
        </el-radio-group>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        roleList: [],
        dataForm: {
          id: 0,
          name: '',
          avatarUrl: '',
          status: 1,
          description: '',
          minPriceTip: 0,
          shippingFeeTip: 0,
          bulletin: ''
        },
        dataRule: {
          name: [
            { required: true, message: '门店名不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
        })
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/sm/restaurant/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.name = data.restaurant.name
              this.dataForm.avatarUrl = data.restaurant.avatarUrl
              this.dataForm.status = data.restaurant.status
            }
          })
        }
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sm/restaurant/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'avatarUrl': this.dataForm.avatarUrl,
                'status': this.dataForm.status
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
