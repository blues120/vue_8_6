<template>
  <div class="mod-oss">
    <el-form :inline="true" :model="dataForm">
      <el-form-item>
        <!--<el-button type="primary" @click="configHandle()">云存储配置</el-button>-->
        <el-button type="primary" @click="uploadHandle()">upload file</el-button>
        <!--<el-button type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">batch delete</el-button>-->
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
        width="80"
        label="ID">
      </el-table-column>
      <el-table-column
        prop="fileName"
        header-align="center"
        align="center"
        label="file name">
      </el-table-column>
      <el-table-column
        prop="fileUrl"
        header-align="center"
        align="center"
        label="URL path">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        width="180"
        label="create time">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="operator">
        <template slot-scope="scope">
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
    <!-- 弹窗, 云存储配置 -->
    <config v-if="configVisible" ref="config"></config>
    <!-- 弹窗, 上传文件 -->
    <upload v-if="uploadVisible" ref="upload" @refreshDataList="getDataList"></upload>
  </div>
</template>

<script>
  import Config from './oss-config'
  import Upload from './oss-upload'
  export default {
    data () {
      return {
        dataForm: {},
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        configVisible: false,
        uploadVisible: false
      }
    },
    components: {
      Config,
      Upload
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/tfile/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize
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
      // 云存储配置
      configHandle () {
        this.configVisible = true
        this.$nextTick(() => {
          this.$refs.config.init()
        })
      },
      // 上传文件
      uploadHandle () {
        this.uploadVisible = true
        this.$nextTick(() => {
          this.$refs.upload.init()
        })
      },
      // delete
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`make sure [id=${ids.join(',')}]go [${id ? 'delete' : 'batch delete'}]operator?`, 'alert', {
          confirmButtonText: 'make sure',
          cancelButtonText: 'cancel',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/generator/tfile/delete'),
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
        }).catch(() => {})
      }
    }
  }
</script>
