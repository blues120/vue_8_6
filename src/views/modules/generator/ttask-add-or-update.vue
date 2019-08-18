<template>
  <el-dialog
    :title="!dataForm.id ? 'new' : 'edit'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
    <el-form-item label="task name" prop="name">
      <el-input v-model="dataForm.name" placeholder="task name"></el-input>
    </el-form-item>
      <el-form-item label="select file" prop="fileId">
        <el-row>
          <el-col :span="12">
            <el-select v-model="dataForm.fileId" clearable placeholder="select file" >
              <el-option
                v-for="item in fileList"
                :key="item.id"
                :label="item.fileName"
                :value="item.id">
              </el-option>
            </el-select>
          </el-col>
          <el-col :span="12">
            <el-form-item label="group " prop="groupId">
              <el-select v-model="dataForm.groupId" clearable placeholder="select group" >
                <el-option
                  v-for="item in groupList"
                  :key="item.id"
                  :label="item.name"
                  :value="item.id">
                </el-option>
              </el-select>
            </el-form-item>

          </el-col>
        </el-row>
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
        fileList: [],
        groupList: [],
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
            { required: true, message: 'not empty', trigger: 'blur' }
          ],
          fileId: [
            { required: true, message: 'not empty', trigger: 'blur' }
          ],
          groupId: [
            { required: true, message: 'not empty', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0

        this.$http({
          url: this.$http.adornUrl(`/generator/ttask/getFileAndGroupList`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.fileList = data.fileList
            this.groupList = data.groupList
          }
        })
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
