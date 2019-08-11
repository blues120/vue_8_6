<template>
  <el-dialog
    title="注册用户"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="200px">
      <el-form-item label="邮箱" prop="email">
        <el-row>
          <el-row :span="20">
            <el-input v-model="dataForm.email" placeholder="需输入邮箱"></el-input>
          </el-row>
          <el-row :span="4">
            <el-button  type="primary" @click="sendVerify()">发送验证码</el-button>
          </el-row>
        </el-row>

      </el-form-item>
      <el-form-item label="邮箱验证码" prop="code">
        <el-input v-model="dataForm.code" placeholder="邮箱验证码"></el-input>
      </el-form-item>

      <el-form-item label="昵称" prop="username">
        <el-input v-model="dataForm.username" placeholder="英文+数字"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input v-model="dataForm.password" placeholder="密码"></el-input>
      </el-form-item>
      <el-form-item label="重复密码" prop="secondPassword">
        <el-input v-model="dataForm.secondPassword" placeholder="重复密码"></el-input>
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
            { required: true, message: '参数名不能为空', trigger: 'blur' }
          ],
          code: [
            { required: true, message: '参数值不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      sendVerify () {
        this.$http({
          url: this.$http.adornUrl(`/sys/sendEmail`),
          method: 'get',
          params: {
            'email': this.dataForm.email,
            'type': 1
          }
        }).then(({data}) => {
          debugger
          if (data && data.code === 0) {

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


<style scoped>

</style>
