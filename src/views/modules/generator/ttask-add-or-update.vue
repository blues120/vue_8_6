<template>
  <el-dialog
    :title="!dataForm.id ? 'new' : 'edit'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="任务名称" prop="name">
      <el-input v-model="dataForm.name" placeholder="任务名称"></el-input>
    </el-form-item>
    <el-form-item label="状态开，关" prop="status">
      <el-input v-model="dataForm.status" placeholder="状态开，关"></el-input>
    </el-form-item>
    <el-form-item label="文件Id" prop="fileId">
      <el-input v-model="dataForm.fileId" placeholder="文件Id"></el-input>
    </el-form-item>
    <el-form-item label="创建人Id" prop="userId">
      <el-input v-model="dataForm.userId" placeholder="创建人Id"></el-input>
    </el-form-item>
    <el-form-item label="群组id" prop="groupId">
      <el-input v-model="dataForm.groupId" placeholder="群组id"></el-input>
    </el-form-item>
    <el-form-item label="" prop="createTime">
      <el-input v-model="dataForm.createTime" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="modifyTime">
      <el-input v-model="dataForm.modifyTime" placeholder=""></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">cancel</el-button>
      <el-button type="primary" @click="dataFormSubmit()">make sure</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          name: '',
          status: '',
          fileId: '',
          userId: '',
          groupId: '',
          createTime: '',
          modifyTime: ''
        },
        dataRule: {
          name: [
            { required: true, message: '任务名称不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '状态开，关不能为空', trigger: 'blur' }
          ],
          fileId: [
            { required: true, message: '文件Id不能为空', trigger: 'blur' }
          ],
          userId: [
            { required: true, message: '创建人Id不能为空', trigger: 'blur' }
          ],
          groupId: [
            { required: true, message: '群组id不能为空', trigger: 'blur' }
          ],
          createTime: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          modifyTime: [
            { required: true, message: '不能为空', trigger: 'blur' }
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
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/generator/ttask/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.ttask.name
                this.dataForm.status = data.ttask.status
                this.dataForm.fileId = data.ttask.fileId
                this.dataForm.userId = data.ttask.userId
                this.dataForm.groupId = data.ttask.groupId
                this.dataForm.createTime = data.ttask.createTime
                this.dataForm.modifyTime = data.ttask.modifyTime
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/ttask/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'status': this.dataForm.status,
                'fileId': this.dataForm.fileId,
                'userId': this.dataForm.userId,
                'groupId': this.dataForm.groupId,
                'createTime': this.dataForm.createTime,
                'modifyTime': this.dataForm.modifyTime
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: 'operator ok',
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
