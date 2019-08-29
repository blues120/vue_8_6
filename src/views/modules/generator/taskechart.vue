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
            <!--<canvas id="myCanvas" resize style="width: 100%;height: 100%;"></canvas>-->
          </div>

        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  var echarts = require('echarts')
  // var paper = require('paper')
  export default {
    data () {
      return {
        chartLine: null,
        taskId: 0,
        option: {},
        xData: []

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
          // dataZoom: {
          //   show: true,
          //   realtime: true,
          //   orient: 'vertical',   // 'horizontal'
          //   x: 0,
          //   start: 40,
          //   end: 60
          // },
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
            type: 'category',
            data: []
          },
          yAxis: {
            type: 'value'
          },
          series: [
            {
              name: 'test',
              type: 'line',
              data: []

            }
          ]
        }

        this.chartLine = echarts.init(document.getElementById('J_chartLineBox'))
        this.chartLine.setOption(this.option)
        window.addEventListener('resize', () => {
          this.chartLine.resize()
        })
        this.chartLine.on('click', function (params) {
          console.log(this.option)
          console.log(params)
        })
        // debugger
        // var canvas = document.getElementById('myCanvas')
        // paper.setup(canvas)
        // var shapesLayer = new paper.Layer()
        // // var path
        // var point, length
        // var tool = new paper.Tool()
        // tool.minDistance = 10
        // tool.onMouseDown = function (event) {
        //   point = event.point
        //   length = shapesLayer.children.length
        // }
        // tool.onMouseDrag = function (event) {
        //   var topLeft = new paper.Point(point)
        //   var rectSize = new paper.Size((event.point.x - point.x), (event.point.y - point.y))
        //   var rect = new paper.Rectangle(topLeft, rectSize)
        //   var path = new paper.Path.Rectangle(rect, 0)
        //   path.strokeColor = 'black'
        // /* path.dashArray = [5, 1]; */
        //   if (shapesLayer.children.length >= (length + 2) && shapesLayer.children.length >= 2) {
        //     shapesLayer.removeChildren(shapesLayer.children.length - 2, shapesLayer.children.length - 1)
        //   }
        // }

        // this.chartLine.getZr().on('click', params => {
        //   const pointInPixel = [params.offsetX, params.offsetY]
        //   if (this.chartLine.containPixel('grid', pointInPixel)) {
        //     let xIndex = this.chartLine.convertFromPixel({ seriesIndex: 0 }, [params.offsetX, params.offsetY])
        //     alert(xIndex)
        //   }
        // })
        this.chartLine.getZr().on('mousedown', params => {
          const pointInPixel = [params.offsetX, params.offsetY]
          if (this.chartLine.containPixel('grid', pointInPixel)) {
            let xIndex = this.chartLine.convertFromPixel({ seriesIndex: 0 }, [params.offsetX, params.offsetY])
            // alert(xIndex)
            debugger
            console.log(this.option.series[0].data)
            this.option.series[0].data.push(xIndex[1])
            console.log(this.option.series[0].data)
            this.chartLine.setOption(this.option)
          }
        })
        this.$http({
          url: this.$http.adornUrl(`/generator/ttask/getTaskEchartData`),
          method: 'get',
          params: this.$http.adornParams({
            taskId: this.taskId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            data.legend.push('test')
            this.option.legend.data = data.legend
            this.option.xAxis.data = data.xAxis
            // this.option.series.push(data.series)
            this.option.title.text = data.group.name
            this.xData = data.series.data
            this.option.series[0].data = this.xData
            // for (let i = 0; i < 30; i += 0.1) {
            //   var tempObj = {}
            //   tempObj['name'] = 'test' + i
            //   tempObj['type'] = 'line'
            //
            //   var color = {}
            //   color['color'] = '#f3f3f3'
            //
            //   var lineStyle = {}
            //   lineStyle['lineStyle'] = color
            //
            //   var normal = {}
            //   normal['color'] = color
            //   normal['lineStyle'] = lineStyle
            //
            //   var itemStyle = {}
            //   itemStyle['normal'] = normal
            //   debugger
            //   tempObj['itemStyle'] = itemStyle
            //   var dataArray = []
            //   for (let j = 0; j < data.xAxis.length; j++) {
            //     dataArray.push(i)
            //   }
            //   tempObj['data'] = dataArray
            //   this.option.series.push(tempObj)
            // }

            // var dataArray = []
            //
            // dataArray.push(7.3)
            // dataArray.push(6.3)
            // dataArray.push(5.3)
            // dataArray.push(4.3)
            // dataArray.push(8.3)
            // tempObj['data'] = dataArray
            // this.option.series.push(tempObj)
            console.log(this.option)
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
