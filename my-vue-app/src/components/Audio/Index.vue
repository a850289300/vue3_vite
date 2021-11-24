<script setup>
import { onMounted, onUpdated, onUnmounted } from 'vue'
import { fabric } from "fabric";
onMounted(()=>{
  let canvas = new fabric.Canvas("canvas"); //可以通过鼠标方法缩小,旋转
})
</script>
<template>
  <canvas id="canvas" width="100%" height="100%"></canvas>
</template>