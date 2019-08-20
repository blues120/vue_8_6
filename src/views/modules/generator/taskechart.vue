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
  var echarts = require('echarts')
  export default {
    data () {
      return {
        chartLine: null,
        taskId: 0
      }
    },
    mounted () {
      this.initChart()
    },
    activated () {
      this.taskId = this.$route.query.taskId
      if (this.taskId === undefined) {
        this.taskId = 0
      }
      if (this.chartLine) {
        this.chartLine.resize()
      }
    },
    methods: {
      initChart () {
        var option = {
          tooltip: {         // Option config. Can be overwrited by series or data
            trigger: 'axis',
            // show: true,   //default true
            showDelay: 0,
            hideDelay: 50,
            transitionDuration: 0,
            backgroundColor: 'rgba(255,0,255,0.7)',
            borderColor: '#f50',
            borderRadius: 8,
            borderWidth: 2,
            padding: 10,    // [5, 10, 15, 20]
            position: function (p) {
              // 位置回调
              // console.log && console.log(p);
              return [p[0] + 10, p[1] - 10]
            },
            textStyle: {
              color: 'yellow',
              decoration: 'none',
              fontFamily: 'Verdana, sans-serif',
              fontSize: 15,
              fontStyle: 'italic',
              fontWeight: 'bold'
            },
            formatter: function (params, ticket, callback) {
              console.log(params)
              var res = ' ' + params[0].name
              for (var i = 0, l = params.length; i < l; i++) {
                res += '<br/>' + params[i].seriesName + ' : ' + params[i].value
              }
              setTimeout(function () {
                // 仅为了模拟异步回调
                callback(ticket, res)
              }, 500)
              return 'loading'
            }
            // formatter: "Template formatter: <br/>{b}<br/>{a}:{c}<br/>{a1}:{c1}"
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
          'legend': {
            'data': ['坐标轴触发1', '坐标轴触发2', '数据项触发1', '数据项触发2']
          },
          calculable: true,
          xAxis: {
            data: ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
          },
          yAxis: {
            type: 'value'
          },
          series: [
            {
              name: '坐标轴触发1',
              type: 'bar',
              data: [
                {value: 320, extra: 'Hello~'},
                332, 301, 334, 390, 330, 320
              ]
            },
            {
              name: '坐标轴触发2',
              type: 'bar',
              data: [862, 1018, 964, 1026, 1679, 1600, 157]
            },
            {
              name: '数据项触发1',
              type: 'bar',
              tooltip: {             // Series config.
                trigger: 'item',
                backgroundColor: 'black',
                position: [0, 0],
                formatter: 'Series formatter: <br/>{a}<br/>{b}:{c}'
              },
              stack: '数据项',
              data: [
                120, 132,
                {
                  value: 301,
                  itemStyle: {normal: {color: 'red'}},
                  tooltip: {     // Data config.
                    backgroundColor: 'blue',
                    formatter: 'Data formatter: <br/>{a}<br/>{b}:{c}'
                  }
                },
                134, 90,
                {
                  value: 230,
                  tooltip: {show: false}
                },
                210
              ]
            },
            {
              name: '数据项触发2',
              type: 'bar',
              tooltip: {
                show: false,
                trigger: 'item'
              },
              stack: '数据项',
              data: [150, 232, 201, 154, 190, 330, 410]
            }
          ]
        }

        this.chartLine = echarts.init(document.getElementById('J_chartLineBox'))
        this.chartLine.setOption(option)
        window.addEventListener('resize', () => {
          this.chartLine.resize()
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
