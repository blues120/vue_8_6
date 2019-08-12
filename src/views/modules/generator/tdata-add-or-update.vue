<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="" prop="userTaskId">
      <el-input v-model="dataForm.userTaskId" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="startX">
      <el-input v-model="dataForm.startX" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="startY">
      <el-input v-model="dataForm.startY" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="endX">
      <el-input v-model="dataForm.endX" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="endY">
      <el-input v-model="dataForm.endY" placeholder=""></el-input>
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
          userTaskId: '',
          startX: '',
          startY: '',
          endX: '',
          endY: '',
          createTime: '',
          modifyTime: ''
        },
        dataRule: {
          userTaskId: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          startX: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          startY: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          endX: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          endY: [
            { required: true, message: '不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/generator/tdata/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.userTaskId = data.tdata.userTaskId
                this.dataForm.startX = data.tdata.startX
                this.dataForm.startY = data.tdata.startY
                this.dataForm.endX = data.tdata.endX
                this.dataForm.endY = data.tdata.endY
                this.dataForm.createTime = data.tdata.createTime
                this.dataForm.modifyTime = data.tdata.modifyTime
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
              url: this.$http.adornUrl(`/generator/tdata/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'userTaskId': this.dataForm.userTaskId,
                'startX': this.dataForm.startX,
                'startY': this.dataForm.startY,
                'endX': this.dataForm.endX,
                'endY': this.dataForm.endY,
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
