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
        xData: [],
        startPos: {},
        endPos: {},
        dragFlag: false

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
            data: [2001, 2002, 2003, 2004, 2005, 2006, 2009]
          },
          yAxis: {
            type: 'value'
          },
          series: [
            {
              name: 'test',
              type: 'line',
              data: [1, 2, 3, 4, 5, 6, 7],
              markLine: {
                lineStyle: {
                  normal: {
                    type: 'dashed'
                  }
                },
                data: [
                  [
                    {
                      yAxis: 3,
                      xAxis: '2005'
                    },
                    {
                      yAxis: 6,
                      xAxis: '2009'
                    }
                  ],
                  [
                    {
                      yAxis: 3,
                      xAxis: '2007'
                    },
                    {
                      yAxis: 6,
                      xAxis: '2012'
                    }
                  ]
                ]
              }

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

        this.chartLine.getZr().on('mouseup', params => {
          const pointInPixel = [params.offsetX, params.offsetY]
          if (this.chartLine.containPixel('grid', pointInPixel)) {
            // let xIndex = this.chartLine.convertFromPixel({ seriesIndex: 0 }, [params.offsetX, params.offsetY])
            // this.dragFlag = false
            // console.log('zhangwei:')
            // console.log(this.option.series[0].markLine.data)
            // var endPos = {}
            // endPos['yAxis'] = xIndex[1]
            // let x = xIndex[0]
            // endPos['xAxis'] = this.xData[x]
            // var starPos = {}
            // starPos['yAxis'] = this.startPos['yAxis']
            // starPos['xAxis'] = this.startPos['xAxis']
            //
            // var temp = []
            // temp.push(starPos)
            // temp.push(endPos)
            // console.log(this.option.series[0].markLine.data)
            // console.log(this.option.series[0].markLine.data)
            // this.option.series[0].markLine.data.push(temp)
            // this.chartLine.setOption(this.option)
          }
        })

        this.chartLine.getZr().on('mousemove', params => {
          const pointInPixel = [params.offsetX, params.offsetY]
          if (this.chartLine.containPixel('grid', pointInPixel)) {
            // let xIndex = this.chartLine.convertFromPixel({ seriesIndex: 0 }, [params.offsetX, params.offsetY])
            this.dragFlag = true

            let xIndex = this.chartLine.convertFromPixel({ seriesIndex: 0 }, [params.offsetX, params.offsetY])
            this.dragFlag = false
            console.log('zhangwei:')
            console.log(this.option.series[0].markLine.data)
            var endPos = {}
            this.endPos['yAxis'] = xIndex[1]
            let x = xIndex[0]
            this.endPos['xAxis'] = this.xData[x]
            var starPos = {}
            starPos['yAxis'] = this.startPos['yAxis']
            starPos['xAxis'] = this.startPos['xAxis']

            var temp = []
            temp.push(starPos)
            temp.push(this.endPos)
            console.log(this.option.series[0].markLine.data)
            console.log(this.option.series[0].markLine.data)
            this.option.series[0].markLine.data.push(temp)
            this.chartLine.setOption(this.option)
          }
        })

        this.chartLine.getZr().on('mousedown', params => {
          const pointInPixel = [params.offsetX, params.offsetY]
          if (this.chartLine.containPixel('grid', pointInPixel)) {
            let xIndex = this.chartLine.convertFromPixel({ seriesIndex: 0 }, [params.offsetX, params.offsetY])
            // alert(xIndex)
            this.startPos['yAxis'] = xIndex[1]
            this.endPos['yAxis'] = xIndex[1]
            let x = xIndex[0]
            this.startPos['xAxis'] = this.xData[x]
            this.endPos['xAxis'] = this.xData[x]
            console.log(this.startPos)
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
            this.xData = data.xAxis
            this.option.series[0].data = data.series.data
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
