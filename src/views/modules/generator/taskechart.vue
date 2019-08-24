<template>
  <div class="mod-demo-echarts">
    <el-alert
      title="alert："
      type="warning"
      :closable="false">
    </el-alert>

    <el-row :gutter="20">
      <el-col :span="24">
        <el-card>
          <div id="J_chartLineBox" class="chart-box"></div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  // debugger
  // 引入
  let echarts = require('echarts')
  export default {
    data () {
      return {
        chartLine: null,
        taskId: 0,
        option: {}

      }
    },
    mounted () {
      this.taskId = this.$route.query.taskId
      if (this.taskId === undefined) {
        this.taskId = 0
      }
      this.initChart()
    },
    activated () {
      if (this.chartLine) {
        this.chartLine.resize()
      }
    },
    methods: {
      initChart () {
        this.option = {
          title: {
            'text': '',
            'subtext': ''
          },
          tooltip: {
            trigger: 'item',
            formatter: function (val) {
              // debugger
            }
          },
          dataZoom: {
            show: true,
            realtime: true,
            orient: 'vertical',   // 'horizontal'
            x: 0,
            start: 40,
            end: 60
          },
          toolbox: {
            show: true,
            feature: {
              mark: {
                show: true,
                title: {
                  mark: '辅助线开关',
                  markUndo: '删除辅助线',
                  markClear: '清空辅助线'
                },
                lineStyle: {
                  width: 2,
                  color: '#1e90ff',
                  type: 'dashed'
                }},
              dataView: {show: true, readOnly: false},
              magicType: {show: true, type: ['line', 'bar', 'stack', 'tiled']},
              restore: {show: true},
              saveAsImage: {show: true}
            }

          },
          legend: {
            data: []
          },
          calculable: false,
          xAxis: {
            data: []
          },
          yAxis: {
            type: 'value'
          },
          series: [

          ]
        }
        debugger
        this.chartLine = echarts.init(document.getElementById('J_chartLineBox'))
        this.chartLine.setOption(this.option)
        window.addEventListener('resize', () => {
          this.chartLine.resize()
        })

// 处理点击事件并且跳转到相应的百度搜索页面
//         'click'、'dblclick'、'mousedown'、'mousemove'、'mouseup'、'mouseover'、'mouseout'、'globalout'、'contextmenu'
        this.chartLine.on('globalout', function (params) {
          // debugger
          console.log(params)
        })
        this.chartLine.on('contextmenu', function (params) {
          // debugger
          console.log(params)
        })
        this.chartLine.on('mousemove', function (params) {
          // debugger
          console.log(params)
        })

        // this.chartLine.
        // 点击
        this.chartLine.getZr().on('click', params => {
          debugger
          console.log(params)
          // const pointInPixel = [params.offsetX, params.offsetY]
          // if (myChart.containPixel('grid', pointInPixel)) {
          //   let xIndex = myChart.convertFromPixel({ seriesIndex: 0 }, [params.offsetX, params.offsetY])[0]
          //   alert(xIndex)
          // }
        })

        this.$http({
          url: this.$http.adornUrl(`/generator/ttask/getTaskEchartData`),
          method: 'get',
          params: this.$http.adornParams({
            taskId: this.taskId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.option.legend.data = data.legend
            this.option.xAxis.data = data.xAxis
            this.option.series.push(data.series)
            this.option.title.text = data.group.name
            this.chartLine.setOption(this.option)
          }
        })
      }
    }

  }
</script>

<style scoped>

  #father {
    width: 500px;
    height: 300px;
    /*background-color: skyblue;*/
    display: flex;
    justify-content: center;
    align-items: center;
  }

  #son {
    /*background-color: green;*/

  }
</style>
<style lang="scss">
  .mod-demo-echarts {
    > .el-alert {
      margin-bottom: 10px;
    }
    > .el-row {
      margin-top: -10px;
      margin-bottom: -10px;
      .el-col {
        padding-top: 10px;
        padding-bottom: 10px;
      }
    }
    .chart-box {
      min-height: 400px;
    }
  }
</style>
