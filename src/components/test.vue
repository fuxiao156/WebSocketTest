<script setup lang="ts">
  import {Zstd} from "@hpcc-js/wasm/zstd"
  import {io} from "socket.io-client";

  const socket = io("http://175.6.203.76:24630");
  socket.on('data_all_comp', (data) => {
      const unit8Array = new Uint8Array(data.data);
      hpcc_handleCompress(unit8Array,'data_all_comp')
  })




  const decoder = new TextDecoder()
  //进行解压
  async function hpcc_handleCompress(unit8Array:Uint8Array,eventName:string) {
      const startTime = performance.now();
      const zstd = await Zstd.load();
      const decompressed_data = zstd.decompress(unit8Array)
      const decompressed_string = decoder.decode(decompressed_data)
      const endTime = performance.now();
      const time = (endTime - startTime).toFixed(1)
      console.log("解压完成")
      console.log(decompressed_string)
      console.log("解压时间")
      console.log(time + "ms")
      socket.emit(eventName, time);
  }
</script>
<template>
</template>

<style scoped>
</style>
