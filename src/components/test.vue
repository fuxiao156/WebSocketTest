<script setup lang="ts">
  import {Zstd} from "@hpcc-js/wasm/zstd"
  import {originStr100} from "../assets/originStr"
  
  const encoder = new TextEncoder()
  const bytes = encoder.encode(originStr100)
  console.log(bytes)
  hpcc_handle(bytes)
  async function hpcc_handle(unit8Array:Uint8Array) {
    var tempBytes = await hpcc_Compress(unit8Array)
    hpcc_handleCompress(tempBytes)
  }
  //进行压缩
  const decoder = new TextDecoder()
  async function hpcc_Compress(unit8Array:Uint8Array) {
      const zstd = await Zstd.load();
      const compressed_data = zstd.compress(unit8Array)
      const decompressed_string = decoder.decode(compressed_data)
      console.log("压缩完成")
      console.log(decompressed_string)
      return compressed_data
  }
  //进行解压
  async function hpcc_handleCompress(unit8Array:Uint8Array) {
      const zstd = await Zstd.load();
      const decompressed_data = zstd.decompress(unit8Array)
      const decompressed_string = decoder.decode(decompressed_data)
      console.log("解压完成")
      console.log(decompressed_string)
  }
</script>
<template>
</template>

<style scoped>
</style>
