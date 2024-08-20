<template>
  <div>
    <div class="card" style="padding: 15px">
      您好，{{ user?.name }}！欢迎使用本系统
    </div>
    <el-card style="margin: 15px; min-height: calc(100vh - 80px)">
      <!--    头部数据-->
      <div>
        <el-row :gutter="20" class="topInfo">
          <el-col :span="6">
            <div id="stuNumDiv" class="el-colDiv">
              <div id="ssv1-main-text" class="nowDiv">实时</div>
              <span class="title">订单总量</span><br/>
              <span class="digital">{{ this.total_count }}</span><br/>
              <span class="last-span">当前商家总记录数</span>
            </div>
          </el-col>
          <el-col :span="6">
            <div id="haveRoomDiv" class="el-colDiv">
              <div id="ssv2-main-text" class="nowDiv">实时</div>
              <span class="title">总价</span><br/>
              <span class="digital">{{ this.total_price }}</span><br/>
              <span class="last-span">当前商家总记录数</span>
            </div>
          </el-col>
          <el-col :span="6">
            <div id="repairNum" class="el-colDiv">
              <div id="ssv3-main-text" class="nowDiv">实时</div>
              <span class="title">实际总价</span><br/>
              <span class="digital">{{ this.total_price_real }}</span><br/>
              <span class="last-span">当前商家总记录数</span>
            </div>
          </el-col>
          <el-col :span="6">
            <div id="emptyRoom" class="el-colDiv">
              <div id="ssv4-main-text" class="nowDiv">实时</div>
              <span class="title">折扣</span><br/>
              <span class="digital">{{ this.total_price_real / this.total_price }}</span><br/>
              <span class="last-span">当前商家记录数</span>
            </div>
          </el-col>
        </el-row>
      </div>
    <div style="display: flex; margin: 10px 0">
      <div style="width: 50%;" class="card">
        <div style="margin-bottom: 30px; font-size: 20px; font-weight: bold">公告列表</div>
        <div >
          <el-timeline  reverse slot="reference">
            <el-timeline-item v-for="item in notices" :key="item.id" :timestamp="item.time">
              <el-popover
                  placement="right"
                  width="200"
                  trigger="hover"
                  :content="item.content">
                <span slot="reference">{{ item.title }}</span>
              </el-popover>
            </el-timeline-item>
          </el-timeline>
        </div>
      </div>
      <div style="display: flex; margin: 0px 20px">
        <div style="width: 100%;" class="card"  v-if="user.role === 'ADMIN'">
          <div style="margin-bottom: 30px; font-size: 20px; font-weight: bold">商家销售情况统计</div>
          <div id="echarts-dom" style="width: 650px;height: 500px"></div>
        </div>
      </div>
      <div style="display: flex; margin: 0px 20px">
        <div style="width: 100%;" class="card"  v-if="user.role === 'ADMIN'">
          <div style="margin-bottom: 30px; font-size: 20px; font-weight: bold">商家销售额统计</div>
          <div id="pieChart" style="width: 500px;height:500px;"></div>
        </div>
      </div>
      <div style="display: flex; margin: 0px 20px">
        <div style="width: 100%;" class="card"  v-if="user.role !== 'ADMIN'">
          <div style="margin-bottom: 30px; font-size: 20px; font-weight: bold">商家销售情况统计</div>
          <div id="echarts-dom" style="width: 650px;height: 500px"></div>
        </div>
      </div>
    </div>
    </el-card>
  </div>
</template>

<script>
import * as echarts from 'echarts';
require("echarts/theme/macarons");
export default {

  name: 'Home',
  data() {
    return {
      user: JSON.parse(localStorage.getItem('xm-user') || '{}'),
      notices: [],
      option: {
        barWidth: 35,
        tooltip: {},
        xAxis: {
          data: []
        },
        yAxis: {
          type: "value"
        },
        series: [
          {
            name: '人数',
            type: 'bar',
            data: []
          },
        ],
        grid: {
          x: 40,
          y: 40,
          x2: 40,
          y2: 40,
          borderWidth: 10,
          top: '10%',
          bottom: '0%',
          containLabel: true
        }
      },
      myEcharts: '',
      chartWidth: '',
      chartHeight: '',
      total_count: 0,
      total_price: 0,
      total_price_real: 0,
      discount: '',
      admin_analysis_key: {},
      admin_analysis_value: {},
      analysis: {},
      analysis_pie: {},
    }
  },
  created() {
    this.$request.get('/orders/selectAll').then(res => {
      this.notices = res.data || []
      this.total_count = res.data.length
      let arr = res.data
      let tp=0, tpr=0, discount
      let analysis = {}
      let analysis_pie = {}
      let that = this
      arr.forEach(function(element) {
        tp += element.amount
        tpr += element.actual
        if(JSON.parse(localStorage.getItem('xm-user')).role === 'ADMIN'){ // 管理员
          if(analysis[element.businessName] == undefined || analysis[element.businessName] == null){
            analysis[element.businessName] = 1
            analysis_pie[element.businessName] = 0.0
          }else{
            analysis[element.businessName] += 1
            analysis_pie[element.businessName] += element.actual
          }
          console.log(analysis_pie)
          setTimeout(function() {
            that.option.xAxis.data = Object.keys(analysis)
            that.option.series[0].data = Object.values(analysis)
            that.analysis_pie = analysis_pie
          }, 1000); // 1秒钟后执行
        }
        else{ // 商家
          that.$request.get('/ordersItem/selectByOrderIdExtra/'+element.id).then(ret => {
            console.log("看看ret",ret)
            let aarr = ret.data
            aarr.forEach(function(ee) {
              if(analysis[ee.goodsName] == undefined || analysis[ee.goodsName] == null){
                analysis[ee.goodsName] = 1
              }else{
                analysis[ee.goodsName] += 1
              }
              that.analysis = analysis
              setTimeout(function() {
                that.option.xAxis.data = Object.keys(analysis)
                that.option.series[0].data = Object.values(analysis)
              }, 500); // 1秒钟后执行
            })
          })
        }
      });
      this.total_price = tp
      this.total_price_real = tpr
      discount = (tpr / tp)
      this.discount = discount.toFixed(2)
    })
  },
  mounted() {
    this.createEcharts()
    let that = this
    if(JSON.parse(localStorage.getItem('xm-user')).role === 'ADMIN')
      setTimeout(function (e){
        that.initPieChart();
      }, 2000)
  },
  watch: {
    //观察option的变化
    option: {
      handler(newVal, oldVal) {
        if (this.myEcharts) {
          if (newVal) {
            this.myEcharts.setOption(newVal);
          } else {
            this.myEcharts.setOption(oldVal);
          }
        } else {
          this.createEcharts();
        }
      },
      deep: true //对象内部属性的监听，关键。
    }
  },
  methods: {
    createEcharts() {
      const chartDmo = document.getElementById("echarts-dom");
      this.myEcharts = echarts.init(chartDmo, null);
      this.myEcharts.setOption(this.option, true);
    },
    initPieChart() {
      // 获取容器
      let pieChartContainer = document.getElementById('pieChart');

      // 基于容器创建 ECharts 实例
      let myPieChart = echarts.init(pieChartContainer);
      let json = this.analysis_pie
      let keys = Object.keys(json)
      let values = Object.values(json)
      let index = 0
      let data = [];
      keys.forEach(function (e){
        let data0 = {value: 0, name: ''}
        data0['value'] = values[index]
        data0['name'] = keys[index]
        data[index] = data0
        index++
      })
      // 饼状图数据
      console.log(data)
      // setTimeout(function() {
      // 饼状图配置
      let options = {
        title: {
          text: '', // 标题
          left: 'center' // 居中显示
        },
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b}: {c} ({d}%)' // 提示框格式
        },
        legend: {
          orient: 'vertical', // 图例垂直排列
          left: 10, // 左边距
          data: [] // 图例数据
        },
        series: [
          {
            name: 'Fruit', // 数据名称
            type: 'pie', // 图表类型为饼状图
            radius: '55%', // 饼图的半径，可设成 '20%' 到 '80%'
            center: ['50%', '60%'], // 饼图的中心（圆心）坐标
            data: data, // 饼图数据
            emphasis: {
              itemStyle: {
                shadowBlur: 10, // 阴影模糊度
                shadowOffsetX: 0, // 阴影水平偏移
                shadowColor: 'rgba(0, 0, 0, 0.5)' // 阴影颜色
              }
            }
          }
        ]
      };
      options.legend.data = keys
      // 使用配置项显示饼状图

      myPieChart.setOption(options);
      // }, 1000);
    }
  }
}
</script>
<style scoped>@import '../../assets/css/Home.css';</style>
