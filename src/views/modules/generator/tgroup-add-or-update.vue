<template>
  <el-dialog
    :title="!dataForm.id ? 'new' : 'edit'"
    :close-on-click-modal="false"
    :visible.sync="visible" width="60%">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="100px">
    <el-form-item label="group name" prop="name">
      <el-input v-model="dataForm.name" placeholder="group name" style="width: 300px"></el-input>
    </el-form-item>
    <el-form-item label="teacher" prop="teacherId">
      <el-row>
        <el-col :span="12">
          <el-select v-model="dataForm.teacherId" clearable placeholder="select teacher" @change="selectTeacher()">
            <el-option
              v-for="item in teacherList"
              :key="item.userId"
              :label="item.username"
              :value="item.userId">
            </el-option>
          </el-select>
        </el-col>
        <el-col :span="12">
          <el-form-item label="student " prop="studentId">
            <el-select v-model="dataForm.leaderId" clearable placeholder="select student" @change="selectTeacher()">
              <el-option
                v-for="item in studentList"
                :key="item.userId"
                :label="item.username"
                :value="item.userId">
              </el-option>
            </el-select>
          </el-form-item>

        </el-col>
      </el-row>



    </el-form-item>
    <el-form-item label="add student in group">

      <div style="text-align: center">
        <el-transfer
          style="text-align: left; display: inline-block"
          v-model="value"
          filterable
          :left-default-checked="[]"
          :right-default-checked="[]"
          :titles="['Source', 'Target']"
          :button-texts="['go left', 'go right']"
          :format="{
        noChecked: '${total}',
        hasChecked: '${checked}/${total}'
      }"
          @change="handleChange"
          :data="data">
          <span slot-scope="{ option }">{{ option.key }} - {{ option.label }}</span>
          <!--<el-button class="transfer-footer" slot="left-footer" size="small">操作</el-button>-->
          <!--<el-button class="transfer-footer" slot="right-footer" size="small">操作</el-button>-->
        </el-transfer>
      </div>

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
      const generateData = _ => {
        const data = []
        for (let i = 1; i <= 15; i++) {
          data.push({
            key: i,
            label: `备选项 ${i}`,
            disabled: i % 4 === 0
          })
        }
        return data
      }
      return {
        data: generateData(),
        value: [],
        renderFunc (h, option) {
          return `<span>{ option.key } - { option.label }</span>`
        },
        visible: false,
        teacherList: [],
        studentList: [],
        dataForm: {
          id: 0,
          name: '',
          teacherId: '',
          leaderId: '',
          createTime: '',
          modifyTime: ''
        },
        dataRule: {
          name: [
            { required: true, message: 'group name不能为空', trigger: 'blur' }
          ],
          teacherId: [
            { required: true, message: 'teacher不能为空', trigger: 'blur' }
          ],
          leaderId: [
            { required: true, message: 'leader不能为空', trigger: 'blur' }
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
      handleChange (value, direction, movedKeys) {
        console.log(value, direction, movedKeys)
      },
      selectTeacher () {

      },
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true

        this.$http({
          url: this.$http.adornUrl(`/generator/tgroup/getTeacherAndStudent`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.teacherList = data.teacherList
            this.studentList = data.studentList
          }
        })

        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/generator/tgroup/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.tgroup.name
                this.dataForm.teacherId = data.tgroup.teacherId
                this.dataForm.leaderId = data.tgroup.leaderId
                this.dataForm.createTime = data.tgroup.createTime
                this.dataForm.modifyTime = data.tgroup.modifyTime
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
              url: this.$http.adornUrl(`/generator/tgroup/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'teacherId': this.dataForm.teacherId,
                'leaderId': this.dataForm.leaderId,
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
