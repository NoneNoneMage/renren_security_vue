<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="菜名" prop="name">
        <el-input v-model="dataForm.name" placeholder="菜名"></el-input>
      </el-form-item>
      <el-form-item label="缩略图" prop="avatarUrl">
        <el-input v-model="dataForm.avatarUrl" placeholder="缩略图"></el-input>
      </el-form-item>
      <el-form-item label="高清图" prop="imageUrl">
        <el-input v-model="dataForm.imageUrl" placeholder="高清图"></el-input>
      </el-form-item>
      <el-form-item label="价格" prop="price">
        <el-input v-model="dataForm.price" placeholder="价格"></el-input>
      </el-form-item>
      <el-form-item label="原价" prop="oldPrice">
        <el-input v-model="dataForm.oldPrice" placeholder="原价"></el-input>
      </el-form-item>
      <el-form-item label="概述" prop="description">
        <el-input v-model="dataForm.description" placeholder="概述"></el-input>
      </el-form-item>
      <el-form-item label="详述" prop="info">
        <el-input v-model="dataForm.info" placeholder="详述"></el-input>
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
    name: 'goods-add-or-update',
    data () {
      return {
        visible: false,
        roleList: [],
        dataForm: {
          id: 0,
          folderId: 0,
          name: '',
          avatarUrl: '',
          imageUrl: '',
          price: '',
          oldPrice: '',
          description: '',
          info: '',
          status: 1
        },
        dataRule: {
          name: [
            { required: true, message: '菜名不能为空', trigger: 'blur' }
          ],
          price: [
            { required: true, message: '价格不能为空', trigger: 'blur' }
          ],
          avatarUrl: [
            { required: true, message: '缩略图不能为空', trigger: 'blur' }
          ],
          imageUrl: [
            { required: true, message: '高清图不能为空', trigger: 'blur' }
          ],
          // description: [
          //   { required: true, message: '概述不能为空', trigger: 'blur' }
          // ],
          info: [
            { required: true, message: '详述不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id, folderId) {
        this.dataForm.id = id || 0
        this.folderId = folderId - 0 || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
        })
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/sm/goods/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm = data.goods
            }
          })
        }
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sm/goods/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'folderId': this.folderId,
                'name': this.dataForm.name,
                'avatarUrl': this.dataForm.avatarUrl,
                'imageUrl': this.dataForm.imageUrl,
                'price': this.dataForm.price,
                'oldPrice': this.dataForm.oldPrice,
                'description': this.dataForm.description,
                'info': this.dataForm.info,
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
                    this.$emit('refreshGoodsList', this.folderId)
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
