<script setup lang="ts">
import { ref,onMounted } from "vue";
//echarts相关
import * as echarts from "echarts/core";
import { LineChart} from "echarts/charts";
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

type ECOption = ComposeOption<
  | BarSeriesOption
  | LineSeriesOption
  | TitleComponentOption
  | TooltipComponentOption
  | GridComponentOption
  | DatasetComponentOption
>;

const testChart = ref();
const myChart = ref();
onMounted(() => {
  // echarts.use([LineChart]);
  // 基于准备好的dom，初始化echarts实例
  echarts.use([
    TitleComponent,
    TooltipComponent,
    GridComponent,
    DatasetComponent,
    TransformComponent,
    LineChart,
    LabelLayout,
    UniversalTransition,
    LegendComponent,
    CanvasRenderer
  ]);
  myChart.value = echarts.init(testChart.value);
myChart.value.setOption(obj.value);
});
const obj = ref(
{
    title: {
      text: 'ECharts 入门示例'
    },
    tooltip: {},
    xAxis: {
      data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子']
    },
    legend: {
          data: ['销量']
        },
    yAxis: {},
    series: [
      {
        name: '销量',
        type: 'line',
        data: [5, 20, 36, 10, 10, 20]
      }
    ]
});

function click() {
  // obj.value.xAxis.data.push('裙子');
  obj.value.series[0].data.push(30);
  myChart.value.setOption(obj.value);
}
</script>
<template>
  <div ref="testChart" id="main" style="width: 600px;height:400px;"></div>
  <button @click="click"></button>
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
</style>
