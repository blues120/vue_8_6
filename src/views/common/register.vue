<template>
  <el-dialog
    title="注册用户"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="200px">
      <el-form-item label=" email" prop="email">
        <el-row>
          <el-row :span="20">
            <el-input v-model="dataForm.email" placeholder="please input email"></el-input>
          </el-row>
          <el-row :span="4">
            <el-button  type="primary" @click="sendVerify()">send  code</el-button>
          </el-row>
        </el-row>

      </el-form-item>
      <el-form-item label="email verify code" prop="code">
        <el-input v-model="dataForm.code" placeholder="email verify code"></el-input>
      </el-form-item>

      <el-form-item label="昵称" prop="username">
        <el-input v-model="dataForm.username" placeholder="英文+数字"></el-input>
      </el-form-item>
      <el-form-item label="password" prop="password">
        <el-input v-model="dataForm.password" placeholder="password"></el-input>
      </el-form-item>
      <el-form-item label="reset password" prop="secondPassword">
        <el-input v-model="dataForm.secondPassword" placeholder="reset password"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">cancel</el-button>
      <el-button type="primary" @click="registerClick()">make sure</el-button>
    </span>
  </el-dialog>
</template>


<script>
  export default {
    name: 'register',
    data () {
      return {
        visible: false,
        dataForm: {
          email: '',
          username: '',
          password: '',
          secondPassword: '',
          code: ''
        },
        dataRule: {
          email: [
            { required: true, message: 'key不能为空', trigger: 'blur' }
          ],
          code: [
            { required: true, message: 'param not empty', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      registerClick () {
        if (this.dataForm.password !== this.dataForm.password) {
          this.$message.error('password not the same second')
          return
        }
        if (this.dataForm.username === '') {
          this.dataForm.username = this.dataForm.email
        }
        this.$http({
          url: this.$http.adornUrl(`/sys/register`),
          method: 'post',
          data: this.$http.adornData({
            'email': this.dataForm.email || undefined,
            'username': this.dataForm.username,
            'password': this.dataForm.password,
            'code': this.dataForm.code
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.visible = false
            this.$alert('register success ,please login')
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      sendVerify () {
        this.$http({
          url: this.$http.adornUrl(`/sys/sendEmail`),
          method: 'get',
          params: {
            'email': this.dataForm.email,
            'type': 1
          }
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$alert('send success')
          }
        })
      },
      init () {
        this.visible = true
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sys/config/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'paramKey': this.dataForm.paramKey,
                'paramValue': this.dataForm.paramValue,
                'remark': this.dataForm.remark
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


<style scoped>

</style>
