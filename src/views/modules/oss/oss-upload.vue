<template>
  <el-dialog
    title="upload file"
    :close-on-click-modal="false"
    @close="closeHandle"
    :visible.sync="visible">
    <el-upload
      drag
      :action="url"
      :before-upload="beforeUploadHandle"
      :on-success="successHandle"
      multiple
      :file-list="fileList"
      style="text-align: center;">
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">drag file to here，or<em>click</em></div>
      <div class="el-upload__tip" slot="tip">！</div>
    </el-upload>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        url: '',
        num: 0,
        successNum: 0,
        fileList: []
      }
    },
    methods: {
      init (id) {
        this.url = this.$http.adornUrl(`/generator/tfile/uploadFile?token=${this.$cookie.get('token')}`)
        this.visible = true
      },
      // 上传之前
      beforeUploadHandle (file) {
        // if (file.type !== 'image/jpg' && file.type !== 'image/jpeg' && file.type !== 'image/png' && file.type !== 'image/gif' && file.type !== 'file/') {
        //   this.$message.error('只支持jpg、png、gif格式的图片！')
        //   return false
        // }
        this.num++
      },
      // 上传成功
      successHandle (response, file, fileList) {
        this.fileList = fileList
        this.successNum++
        if (response && response.code === 0) {
          if (this.num === this.successNum) {
            this.$confirm('operator success, is need operator?', 'alert', {
              confirmButtonText: 'make sure',
              cancelButtonText: 'cancel',
              type: 'warning'
            }).catch(() => {
              this.visible = false
            })
          }
        } else {
          this.$message.error(response.msg)
        }
      },
      // 弹窗关闭时
      closeHandle () {
        this.fileList = []
        this.$emit('refreshDataList')
      }
    }
  }
</script>
