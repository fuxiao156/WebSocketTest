<script setup lang="ts">
  import {io} from "socket.io-client";
  import {ref,onMounted} from "vue";
  import {Zstd} from "@hpcc-js/wasm/zstd"
  
  //导入echarts
import * as echarts from "echarts/core";
import { LineChart,BarChart} from "echarts/charts";
import { LabelLayout, UniversalTransition } from 'echarts/features';
import {CanvasRenderer} from "echarts/renderers";
import {
  TitleComponent,
  TooltipComponent,
  GridComponent,
  // 数据集组件
  DatasetComponent,
  // 内置数据转换器组件 (filter, sort)
  TransformComponent,
  LegendComponent,
} from 'echarts/components';
import type {
  // 系列类型的定义后缀都为 SeriesOption
  BarSeriesOption, 
  LineSeriesOption
} from 'echarts/charts';
import type {
  // 组件类型的定义后缀都为 ComponentOption
  TitleComponentOption,
  TooltipComponentOption,
  GridComponentOption,
  DatasetComponentOption
} from 'echarts/components';
import type { 
  ComposeOption, 
} from 'echarts/core';
import test from "node:test";

type ECOption = ComposeOption<
  | BarSeriesOption
  | LineSeriesOption
  | TitleComponentOption
  | TooltipComponentOption
  | GridComponentOption
  | DatasetComponentOption
>;

  echarts.use([
    TitleComponent,
    TooltipComponent,
    GridComponent,
    DatasetComponent,
    TransformComponent,
    LineChart,
    BarChart,
    LegendComponent,
    LabelLayout,
    UniversalTransition,
    CanvasRenderer
  ]);

//用来设置
var isCompress = [false,false,false,false];
var DelayChartUse = [true,true,true,true,true,true,true,true]


//创建ref与表格
const chart1 = ref();
const chart2 = ref();
const chart5 = ref();
const chart10 = ref();
const eCharts = ref([]);

//横坐标值
var xray =  [0,0,0,0]
//每个表格的长度
var chartNum = [1,1,1,1,1,1,1,1]

const DelayChartObjs = ref([
{
    title: {
      text: '10000数据'
    },
    tooltip: {},
    legend: {
      data: ['原始','压缩']
    },
    xAxis: {
      name: '次数',
      data: []
    },
    yAxis: {
      name: '时间(ms)',
    },
    series: [
      {
        name: '原始',
        type: 'line',
        data: []
      },
      {
        name: '压缩',
        type: 'line',
        data: []
      }
    ]
},
{
    title: {
      text: '20000数据'
    },
    tooltip: {},
    legend: {
      data: ['原始','压缩']
    },
    xAxis: {
      name: '次数',
      data: []
    },
    yAxis: {
      name: '时间(ms)',
    },
    series: [
      {
        name: '原始',
        type: 'line',
        data: []
      },
      {
        name: '压缩',
        type: 'line',
        data: []
      }
    ]
},
{
    title: {
      text: '50000数据'
    },
    tooltip: {},
    legend: {
      data: ['原始','压缩']
    },
    xAxis: {
      name: '次数',
      data: []
    },
    yAxis: {
      name: '时间(ms)',
    },
    series: [
      {
        name: '原始',
        type: 'line',
        data: []
      },
      {
        name: '压缩',
        type: 'line',
        data: []
      }
    ]
},
{
    title: {
      text: '100000数据'
    },
    tooltip: {},
    legend: {
      data: ['原始','压缩']
    },
    xAxis: {
      name: '次数',
      data: []
    },
    yAxis: {
      name: '时间(ms)',
    },
    series: [
      {
        name: '原始',
        type: 'line',
        data: []
      },
      {
        name: '压缩',
        type: 'line',
        data: []
      }
    ]
}])
onMounted(() => {
  // 基于准备好的dom，初始化echarts实例
  eCharts.value.push(echarts.init(chart1.value));
  eCharts.value.push(echarts.init(chart2.value));
  eCharts.value.push(echarts.init(chart5.value));
  eCharts.value.push(echarts.init(chart10.value));
  eCharts.value[0].setOption(DelayChartObjs.value[0]);
  eCharts.value[1].setOption(DelayChartObjs.value[1]);
  eCharts.value[2].setOption(DelayChartObjs.value[2]);
  eCharts.value[3].setOption(DelayChartObjs.value[3]);
});

  const socket = io("http://175.6.203.76:24630");
  const decoder = new TextDecoder()
  socket.on('connect', () => {
    console.log('Connected to the WebSocket server');
  });

  // 监听服务器发送的消息
  socket.on('greeting', (data) => {
    console.log('Received message:', data);
  });

  // 服务器传来的数据
  //以下是没有压缩的数据
  socket.on('data_10000', (data) => {
    reDrawChart(0,0,0,data.time)
  })
  socket.on('data_20000', (data) => {
    reDrawChart(1,1,0,data.time)
  })
  socket.on('data_50000', (data) => {
    reDrawChart(2,2,0,data.time)
  })
  socket.on('data_100000', (data) => {
    reDrawChart(3,3,0,data.time)
  })
  //以下是压缩了的数据
  socket.on('data_10000_comp', (data) => {
    reDrawChart(0,4,1,data.time)
    if(isCompress[0]){
      const unit8Array = new Uint8Array(data.data);
      hpcc_handleCompress(unit8Array)
    }
  })
  socket.on('data_20000_comp', (data) => {
    reDrawChart(1,5,1,data.time)
    if(isCompress[1]){
      const unit8Array = new Uint8Array(data.data);
      hpcc_handleCompress(unit8Array)
    }
  })
  socket.on('data_50000_comp', (data) => {
    reDrawChart(2,6,1,data.time)
    if(isCompress[2]){
      const unit8Array = new Uint8Array(data.data);
      hpcc_handleCompress(unit8Array)
    }
  })
  socket.on('data_100000_comp', (data) => {
    reDrawChart(3,7,1,data.time)
    if(isCompress[3]){
      const unit8Array = new Uint8Array(data.data);
      hpcc_handleCompress(unit8Array)
    }
  })
      //hpcc-js/wasm/zstd
  async function hpcc_handleCompress(unit8Array:Uint8Array) {
      const zstd = await Zstd.load();
      const originstr = btoa(String.fromCharCode.apply(null, unit8Array))
      const decompressed_data = zstd.decompress(unit8Array)
      const decompressed_string = decoder.decode(decompressed_data)
      console.log("解压完成")
      console.log(decompressed_string)
  }

  async function getdelay(pushTime:string){
    const timestamp = new Date(pushTime).getTime();
    try {
        const TheFirstreceivedTime = Date.now();
        const response = await fetch('http://175.6.203.76:24680/getTime');
        const TheSecondreceivedTime = Date.now();
        const fontTime = (TheSecondreceivedTime - TheFirstreceivedTime)/2;
        const data = await response.json();
        const latency = data.time - fontTime - timestamp;
        return latency;
    } catch (error) {
        console.error('Error fetching time:', error);
        // Return a default value or handle the error as per your requirement
        return 0;
    }
  }

  async function reDrawChart(objNum:number,chartNumnum:number,seriesNum:number,pushTime:string){
    // if(false){
    if(DelayChartUse[chartNumnum]){
      var latency = await getdelay(pushTime);
      if(chartNum[chartNumnum]>xray[objNum]){
        DelayChartObjs.value[objNum].xAxis.data.push(chartNum[chartNumnum]);
        xray[objNum]++; 
      }
        DelayChartObjs.value[objNum].series[seriesNum].data.push(latency);
        chartNum[chartNumnum]++;
      eCharts.value[objNum].setOption(DelayChartObjs.value[objNum]);
    }
  }
  function changeUse(num:number){
    DelayChartUse[num] = !DelayChartUse[num];
  }
  function changeCompress(num:number){
    isCompress[num] = !isCompress[num];
  }

  //显示数据
  function showData(){
    console.log("10000未压缩")
    console.log(DelayChartObjs.value[0].series[0].data)
    console.log("10000压缩")
    console.log(DelayChartObjs.value[0].series[1].data)
    console.log("20000未压缩")
    console.log(DelayChartObjs.value[1].series[0].data)
    console.log("20000压缩")
    console.log(DelayChartObjs.value[1].series[1].data)
    console.log("50000未压缩")
    console.log(DelayChartObjs.value[2].series[0].data)
    console.log("50000压缩")
    console.log(DelayChartObjs.value[2].series[1].data)
    console.log("100000未压缩")
    console.log(DelayChartObjs.value[3].series[0].data)
    console.log("100000压缩")
    console.log(DelayChartObjs.value[3].series[1].data)
  }
  //检测资源使用率
    // 使用 ref 创建响应式变量
    const monitoring = ref(false);
    const cpuUsage = ref<number | null>(null);
    const jsHeapSize = ref<number | null>(null);
    let monitoringInterval: ReturnType<typeof setInterval> | null = null;
    let lastime = performance.now();
    let nextTime = performance.now();

    const startMonitoring = () => {
      // 开始记录性能
      monitoring.value = true;
      cpuUsage.value = null;
      jsHeapSize.value = null;
      performance.clearMarks();
      performance.clearMeasures();
      performance.mark('startMonitoring');

      // 设置定时器，每秒获取一次 CPU 使用率和 JavaScript 堆大小
      monitoringInterval = setInterval(() => {
        lastime = nextTime;
        nextTime = performance.now();
        cpuUsage.value = nextTime - lastime;
        jsHeapSize.value = window.console?.memory?.usedJSHeapSize || 0;
        var locator=new ActiveXObject ("WbemScripting.SWbemLocator");
        var service=locator.ConnectServer(".");

        var cpu=new Enumerator (service.ExecQuery("SELECT * FROM Win32_Processor")).item();
        document.title = cpu.LoadPercentage;
      }, 500);
    };

    const stopMonitoring = () => {
      // 停止定时器
      if (monitoringInterval) {
        clearInterval(monitoringInterval);
      }

      // 停止记录性能
      monitoring.value = false;
      performance.mark('stopMonitoring');
      performance.measure('monitoringDuration', 'startMonitoring', 'stopMonitoring');
      const duration = performance.getEntriesByName('monitoringDuration')[0].duration;

      // 在控制台显示监测时长
      console.log('Monitoring Duration:', duration.toFixed(2), 'ms');
    };

    // 将字节数转换为人类可读格式
    const formatBytes = (bytes: number, decimals = 2) => {
      if (bytes === 0) return '0 Bytes';

      const k = 1024;
      const dm = decimals < 0 ? 0 : decimals;
      const sizes = ['Bytes', 'KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB'];

      const i = Math.floor(Math.log(bytes) / Math.log(k));

      return parseFloat((bytes / Math.pow(k, i)).toFixed(dm)) + ' ' + sizes[i];
    };
</script>
<template>

  <div class="charts">
    <div class="chartsrow">
      <div class="chart" ref="chart1" ></div>
      <div class="chart" ref="chart2" ></div>
    </div>
    <div class="chartsrow">
      <div class="chart" ref="chart5" ></div>
      <div class="chart" ref="chart10"></div>
    </div>
  </div>
  <div>
    <div>控制台</div>
    <div>
      <span>禁用绘制：</span>
      <input type="checkbox" @change="changeUse(0)">
      <label>10000未压缩</label>
      <input type="checkbox" @change="changeUse(4)">
      <label>10000压缩</label>
      <input type="checkbox" @change="changeUse(1)">
      <label>20000未压缩</label>
      <input type="checkbox" @change="changeUse(5)">
      <label>20000压缩</label>
      <input type="checkbox" @change="changeUse(2)">
      <label>50000未压缩</label>
      <input type="checkbox" @change="changeUse(6)">
      <label>50000压缩</label>
      <input type="checkbox" @change="changeUse(3)">
      <label>100000未压缩</label>
      <input type="checkbox" @change="changeUse(7)">
      <label>100000压缩</label>
    </div>
    <div>
      <span>启用压缩：</span>
      <input type="checkbox" @change="changeCompress(0)">
      <label>10000</label>
      <input type="checkbox" @change="changeCompress(1)">
      <label>2000</label>
      <input type="checkbox" @change="changeCompress(2)">
      <label>50000</label>
      <input type="checkbox" @change="changeCompress(3)">
      <label>100000</label>
    </div>
  </div>
  <button @click="showData">打印数据</button>

  <div>
    <h1>Performance Monitoring</h1>
    <button @click="startMonitoring" v-if="!monitoring">Start Monitoring</button>
    <button @click="stopMonitoring" v-if="monitoring">Stop Monitoring</button>
    <div v-if="cpuUsage !== null">CPU Usage: {{ cpuUsage.toFixed(2) }} ms</div>
    <div v-if="jsHeapSize !== null">JS Heap Size: {{ formatBytes(jsHeapSize) }}</div>
  </div>
  <div>
  </div>
  <ul id="messages"></ul>
  <form id="form" action="">
    <input id="input" autocomplete="off" /><button>Send</button>
  </form>
</template>

<style scoped>
     body { margin: 0; padding-bottom: 3rem; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; }

      #form { background: rgba(0, 0, 0, 0.15); padding: 0.25rem; position: fixed; bottom: 0; left: 0; right: 0; display: flex; height: 3rem; box-sizing: border-box; backdrop-filter: blur(10px); }
      #input { border: none; padding: 0 1rem; flex-grow: 1; border-radius: 2rem; margin: 0.25rem; }
      #input:focus { outline: none; }
      #form > button { background: #333; border: none; padding: 0 1rem; margin: 0.25rem; border-radius: 3px; outline: none; color: #fff; }

      #messages { list-style-type: none; margin: 0; padding: 0; }
      #messages > li { padding: 0.5rem 1rem; }
      #messages > li:nth-child(odd) { background: #efefef; }
      .charts{
      }
      .chartsrow{
        display: flex;
        align-items: center;
        flex-direction: row;
        justify-content:space-evenly;
      }
      .chart{
        width: 400px;
        height: 300px;
      }
</style>
