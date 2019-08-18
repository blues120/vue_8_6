<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <!--<el-form-item>-->
        <!--<el-input v-model="dataForm.key" placeholder="key" clearable></el-input>-->
      <!--</el-form-item>-->
      <el-form-item>
        <!--<el-button @click="getDataList()">query</el-button>-->
        <el-button v-if="isAuth('generator:ttask:save')" type="primary" @click="addOrUpdateHandle()">new</el-button>
        <!--<el-button v-if="isAuth('generator:ttask:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">batch delete</el-button>-->
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="primary id">
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="task name">
      </el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="status">
      </el-table-column>
      <el-table-column
        prop="fileId"
        header-align="center"
        align="center"
        label="file Id">
      </el-table-column>
      <el-table-column
        prop="userName"
        header-align="center"
        align="center"
        label="create name">
      </el-table-column>
      <el-table-column
        prop="groupName"
        header-align="center"
        align="center"
        label="group name">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="create time">
      </el-table-column>
      <el-table-column
        prop="modifyTime"
        header-align="center"
        align="center"
        label="modify time">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="operator">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">edit</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">delete</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, new / edit -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './ttask-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/ttask/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // new / edit
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // delete
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`make sure [id=${ids.join(',')}] go [${id ? 'delete' : 'batch delete'}]operator?`, 'alert', {
          confirmButtonText: 'make sure',
          cancelButtonText: 'cancel',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/generator/ttask/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: 'operator ok',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }
  }
</script>
