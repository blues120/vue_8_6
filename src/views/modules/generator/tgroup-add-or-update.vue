<template>
  <el-dialog
    :title="!dataForm.id ? 'new' : 'edit'"
    :close-on-click-modal="false"
    :visible.sync="visible" width="60%">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="100px">
    <el-form-item label="group name" prop="name">
      <el-input v-model="dataForm.name" :disabled="this.dataForm.id!=0" placeholder="group name" style="width: 300px"></el-input>
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
    <!--<el-form-item label="" prop="createTime">-->
      <!--<el-input v-model="dataForm.createTime" placeholder=""></el-input>-->
    <!--</el-form-item>-->
    <!--<el-form-item label="" prop="modifyTime">-->
      <!--<el-input v-model="dataForm.modifyTime" placeholder=""></el-input>-->
    <!--</el-form-item>-->
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
        // debugger
        // for (let i = 1; i <= this.leftInitList.size(); i++) {
        //   let temp = this.leftInitList[i]
        //   data.push({
        //     key: temp.userId,
        //     label: temp.username
        //     // disabled: i % 4 === 0
        //   })
        // }
        return data
      }
      return {
        data: generateData(),
        tempData: {},
        value: [],
        renderFunc (h, option) {
          return `<span>{ option.key } - { option.label }</span>`
        },
        visible: false,
        teacherList: [],
        studentList: [],
        leftInitList: [],
        rightInitList: [],
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
          ]
        }
      }
    },
    methods: {
      handleChange (value, direction, movedKeys) {
        console.log(value, direction, movedKeys)
        if (direction === 'right') {
          for (let i = 0; i < value.length; i++) {
            let temp = value[i]
            this.tempData['' + temp] = temp
          }
        } else {
          for (let i = 0; i < value.length; i++) {
            let temp = value[i]
            this.tempData['' + temp] = ''
          }
        }
        debugger
        console.log('zhangwei' + this.tempData)
      },
      selectTeacher () {

      },
      init (id) {
        this.$forceUpdate()

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

            this.leftInitList = data.studentList
            const tempData = []
            for (let i = 0; i <= data.studentList.length - 1; i++) {
              console.log(data.studentList[i])
              tempData.push({
                key: data.studentList[i].userId,
                label: data.studentList[i].username
                // disabled: i % 4 === 0
              })
            }
            console.log('one')
            this.data = tempData
          }
        })

        var that = this
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/generator/tgroup/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                debugger
                this.dataForm.name = data.tgroup.name
                this.dataForm.teacherId = data.tgroup.teacherId
                this.dataForm.leaderId = data.tgroup.leaderId
                this.dataForm.createTime = data.tgroup.createTime
                this.dataForm.modifyTime = data.tgroup.modifyTime

                this.leftInitList = data.canSelectList
                this.rightInitList = data.groupList

                that.value = []
                for (let i = 0; i < data.groupList.length; i++) {
                  that.value.push(data.groupList[i].userId)
                }

                debugger
                console.log('two' + that.leftCheck + 'right:' + that.rightCheck)
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
                'selectList': this.value
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
