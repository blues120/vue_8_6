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
            data: []
          },
          yAxis: {
            type: 'value'
          },
          series: [
            {
              name: 'test',
              type: 'line',
              data: [],
              markLine: {
                symbol: 'none',
                lineStyle: {
                  normal: {
                    type: 'dashed'
                  }
                },
                data: [

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
        // this.chartLine.on('click', function (params) {
        //   console.log(this.option)
        //   console.log(params)
        // })

        this.chartLine.getZr().on('mousemove', params => {
          const pointInPixel = [params.offsetX, params.offsetY]
          if (this.chartLine.containPixel('grid', pointInPixel)) {
            // let xIndex = this.chartLine.convertFromPixel({ seriesIndex: 0 }, [params.offsetX, params.offsetY])
            if (this.dragFlag === true) {
              let xIndex = this.chartLine.convertFromPixel({ seriesIndex: 0 }, [params.offsetX, params.offsetY])
              // 获取最后一个markline坐标，如果起点坐标相同，就更新终点坐标，否则插入新的markline
              var tempArray = this.option.series[0].markLine.data

              var endPos = {}
              // this.endPos['yAxis'] = xIndex[1]
              let x = xIndex[0]
              // this.endPos['xAxis'] = this.xData[x]
              endPos['yAxis'] = xIndex[1]
              endPos['xAxis'] = this.xData[x]

              var starPos = {}
              starPos['yAxis'] = this.startPos['yAxis']
              starPos['xAxis'] = this.startPos['xAxis']

              if (tempArray.length === 0) {
                console.log('3')
                var tempPos2 = []
                tempPos2.push(starPos)
                tempPos2.push(endPos)
                this.option.series[0].markLine.data.push(tempPos2)
              } else {
                var lastPoint = tempArray[tempArray.length - 1]
                var lastStartObj = lastPoint[0]
                var lastXPos = lastStartObj['xAxis']
                var lastYPos = lastStartObj['yAxis']

                console.log('zhangwei:lastXPos:' + lastXPos + 'lastYPos:' + lastYPos)
                if ((lastXPos === this.startPos['xAxis'])) {
                  console.log('1')
                  var temp = []
                  temp.push(starPos)
                  temp.push(endPos)

                  this.option.series[0].markLine.data[tempArray.length - 1] = temp
                } else {
                  console.log('2')
                  var tempPos = []
                  tempPos.push(starPos)
                  tempPos.push(endPos)
                  this.option.series[0].markLine.data.push(tempPos)
                }
              }

              console.log('zhangwei:xAxis:' + starPos['xAxis'] + 'yAxis:' + starPos['yAxis'])

              this.chartLine.setOption(this.option)
            }
          }
        })

        this.chartLine.getZr().on('mousedown', params => {
          const pointInPixel = [params.offsetX, params.offsetY]
          if (this.chartLine.containPixel('grid', pointInPixel)) {
            let xIndex = this.chartLine.convertFromPixel({ seriesIndex: 0 }, [params.offsetX, params.offsetY])
            // alert(xIndex)点击的时候记录起点，终点（没有啥必要）
            let x = xIndex[0]
            if (this.checkPoint(x)) {
              this.dragFlag = true
            } else {
              this.dragFlag = false
            }
            if (this.dragFlag) {
              this.startPos['yAxis'] = xIndex[1]
              this.startPos['xAxis'] = this.xData[x]
            }
            console.log(this.startPos)
          }
        })

        this.chartLine.getZr().on('mouseup', params => {
          const pointInPixel = [params.offsetX, params.offsetY]
          if (this.chartLine.containPixel('grid', pointInPixel)) {
            let xIndex = this.chartLine.convertFromPixel({ seriesIndex: 0 }, [params.offsetX, params.offsetY])
            // alert(xIndex)点击的时候记录起点，终点（没有啥必要）
            let x = xIndex[0]
            if (this.dragFlag === true) {
              if (this.checkPoint(x)) {

              } else {
                this.$message.error('Draw line x axis can not cross')
                this.option.series[0].markLine.data.pop()
                this.chartLine.setOption(this.option)
              }
            }
            this.dragFlag = false
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
            console.log(this.option.series[0])
            this.chartLine.setOption(this.option)
          }
        })
      },
      checkPoint (value) {
        var tempArray = this.option.series[0].markLine.data
        for (let i = 0; i < tempArray.length; i++) {
          var pointArray = tempArray[i]
          var startObj = pointArray[0]
          var endObj = pointArray[1]
          var startX = startObj['xAxis']
          var endX = endObj['xAxis']
          var startIndex = 0
          var endIndex = 0
          for (let j = 0; j < this.xData.length; j++) {
            if (this.xData[j] === startX) {
              startIndex = j
            }
          }
          for (let j = 0; j < this.xData.length; j++) {
            if (this.xData[j] === endX) {
              endIndex = j
            }
          }
          if (endIndex < startIndex) {
            var tempIndex = startIndex
            startIndex = endIndex
            endIndex = tempIndex
          }
          if (value > startIndex && value < endIndex) {
            return false
          }
        }
        return true
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
