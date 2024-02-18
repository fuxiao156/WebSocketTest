<script setup lang="ts">
//<!-- 计算压缩时间 -->
  import {io} from "socket.io-client";
  import {ref,onMounted} from "vue";
  import {Zstd} from "@hpcc-js/wasm/zstd"
  
  const socket = io("http://175.6.203.76:24630");
  const decoder = new TextDecoder()
//   var isCompress = [false,false,false,false]
  var isCompress = [true,true,true,true]
  //用来计数
  const countNum = ref(0)
  socket.on('connect', () => {
    console.log('Connected to the WebSocket server');
  });

  // 监听服务器发送的消息
  socket.on('greeting', (data) => {
    console.log('Received message:', data);
  });

  // 服务器传来的数据
  //以下是没有压缩的数据
  const encoder = new TextEncoder()
  socket.on('sendData',(data)=>{
    if(data.isComp){
      const unit8Array = new Uint8Array(data.data);
      hpcc_handleCompress(unit8Array,'data_10000000_comp')
    }
    else{
      var Size = encoder.encode(data.data).length/1024
      getJsHeap('0', Size)
    }
  })
      //进行解压
  async function hpcc_handleCompress(unit8Array:Uint8Array,eventName:string) {
    console.log("开始解压")
      const startTime = performance.now();
      const zstd = await Zstd.load();
      const decompressed_data = zstd.decompress(unit8Array)
      const Size = decompressed_data.length/1024
      const endTime = performance.now();
      const time = (endTime - startTime).toFixed(1)
      console.log("解压完成")
      getJsHeap(time,Size)
      const decompressed_string = decoder.decode(decompressed_data)
      console.log(Size)
      console.log(decompressed_string)
  }

  function getJsHeap(decompressTime:string='0',Size:number=0){
    const totalHeapSize = (performance.memory.jsHeapSizeLimit/1024).toFixed(0);
    const usedHeapSize = (performance.memory.usedJSHeapSize/1024).toFixed(0);
    const freeHeapSize = (totalHeapSize - usedHeapSize).toFixed(0);
    socket.emit('responce', {
        totalHeapSize,
        usedHeapSize,
        freeHeapSize,
        decompressTime,
        Size
    });
  }


  //检测资源使用率
    // 使用 ref 创建响应式变量
    const monitoring = ref(false);
    const jsHeapSize = ref<number | null>(null);
    let monitoringInterval: ReturnType<typeof setInterval> | null = null;
    let lastime = performance.now();
    let nextTime = performance.now();

    const startMonitoring = () => {
      // 开始记录性能
      monitoring.value = true;
      jsHeapSize.value = null;
      performance.clearMarks();
      performance.clearMeasures();
      performance.mark('startMonitoring');

      // 设置定时器，每秒获取一次 CPU 使用率和 JavaScript 堆大小
      monitoringInterval = setInterval(() => {
        lastime = nextTime;
        nextTime = performance.now();
        jsHeapSize.value = window.console?.memory?.usedJSHeapSize || 0;
        countNum.value = countNum.value + 1
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

  <div>
    <h1>Performance Monitoring</h1>
    <div>{{ countNum }}</div>
    <button @click="startMonitoring" v-if="!monitoring">Start Monitoring</button>
    <button @click="stopMonitoring" v-if="monitoring">Stop Monitoring</button>
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
