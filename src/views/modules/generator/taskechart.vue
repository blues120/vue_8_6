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
          <div id="J_chartLineBox" class="chart-box">
            <canvas id="myCanvas" resize style="width: 100%;height: 100%;"></canvas>
          </div>

        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  var echarts = require('echarts')
  var paper = require('paper')
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
            trigger: 'item'
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
          calculable: true,
          xAxis: {
            data: []
          },
          yAxis: {
            type: 'value'
          },
          series: [

          ]
        }

        this.chartLine = echarts.init(document.getElementById('J_chartLineBox'))
        this.chartLine.setOption(this.option)
        window.addEventListener('resize', () => {
          this.chartLine.resize()
        })
        debugger
        var canvas = document.getElementById('myCanvas')
        paper.setup(canvas)
        var shapesLayer = new paper.Layer()
        // var path
        var point, length
        var tool = new paper.Tool()
        tool.minDistance = 10
        tool.onMouseDown = function (event) {
          point = event.point
          length = shapesLayer.children.length
        }
        tool.onMouseDrag = function (event) {
          var topLeft = new paper.Point(point)
          var rectSize = new paper.Size((event.point.x - point.x), (event.point.y - point.y))
          var rect = new paper.Rectangle(topLeft, rectSize)
          var path = new paper.Path.Rectangle(rect, 0)
          path.strokeColor = 'black'
        /* path.dashArray = [5, 1]; */
          if (shapesLayer.children.length >= (length + 2) && shapesLayer.children.length >= 2) {
            shapesLayer.removeChildren(shapesLayer.children.length - 2, shapesLayer.children.length - 1)
          }
        }

      // this.chartLine.getZr().on('click', params => {
      //   const pointInPixel = [params.offsetX, params.offsetY]
      //   if (this.chartLine.containPixel('grid', pointInPixel)) {
      //     let xIndex = this.chartLine.convertFromPixel({ seriesIndex: 0 }, [params.offsetX, params.offsetY])[0]
      //     alert(xIndex)
      //   }
      // })
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
