<script setup lang="ts">
import { onMounted, onUpdated, onUnmounted } from 'vue'
import { fabric } from "fabric";
interface mouseData {
  cx: number,
  cy: number
}
let canvas;
let mouseStatus:boolean = false; // 绘制状态
let mouseData: mouseData = {
  cx: 0,
  cy: 0
}
let drawType: string = ''; // 绘制类型
let currentDraw: any = null; // 当前图形
onMounted(()=>{
  canvas = new fabric.Canvas("canvas"); //可以通过鼠标方法缩小,旋转
  initCanvas(canvas)
})

// 初始化canvas
function initCanvas(canvas){
  canvas.clear();
  bindEvent(canvas)
  canvas.add(new fabric.Rect({
    top: 15.770717097847495,
    left: 142.43883678963954,
    angle: 46.802765841263536,
    height: 109,
    width: 186,
    fill: '',
    stroke: 'red'
  }))
}

// 初始化绑定事件
function bindEvent(canvas) {
  canvas.on(
    {
      'mouse:down': (e) => mouseDown(e), // 鼠标mousedown
      'mouse:up': (e) => mouseUp(e), // 鼠标 mouseup
      'mouse:move': (e) => mouseMove(e), // 鼠标移动
      'object:added': (e) => addObject(e), // 添加元素
      'object:modified': (e) => modifyObject(e) // 修改元素
    }
  )
}


// 修改元素
function modifyObject(e) {
  console.log('修改', e)
}

// 添加元素
function addObject(e) {
}

// 点击事件
function mouseDown(e) {
  let pointer = e.pointer || {}
  let { x, y } = pointer;
  mouseStatus = true;
  mouseData = {
    cx : x,
    cy : y
  }
}

function mouseUp(e) {
  mouseStatus = false;
  if (currentDraw) {
    currentDraw = null;
    setDrawType('')
  }
}

// 移动事件
function mouseMove(e) {
  if (mouseStatus) {
    switch(drawType) {
      case 'rect':
        drawRect(e)
        break;
      default: 

    }
  }
}

// 设置类型
function setDrawType(type) {
  drawType = type
}

// 绘制矩形
function drawRect(e) {
  if (currentDraw) {
    canvas.remove(currentDraw)
  }
  let { cx, cy} = mouseData;
  let { pointer = {}} = e;
  let { x, y } = pointer;
  let w = Math.abs(x - cx);
  let h = Math.abs(y - cy);
  currentDraw = new fabric.Rect(
    {
      left: cx,
      top: cy,
      width: w,
      height: h,
      fill: '',
      stroke: '#FD9D91'
    }
  )
  canvas.add(currentDraw)
}
</script>
<template>
  <el-container class="canvas-box">
      <el-aside width="80px" style="padding-top: 20px">
        <el-popover
          width="80"
          placement="right"
          trigger="hover"
          content="绘制矩形"
        >
          <template #reference>
            <span class="control-btn" @click="setDrawType('rect')">矩形</span>
          </template>
        </el-popover>
        <el-popover
          width="80"
          placement="right"
          trigger="hover"
          content="取消绘制"
        >
          <template #reference>
            <span class="control-btn" @click="setDrawType('')">取消</span>
          </template>
        </el-popover>
      </el-aside>
      <el-main>
        <canvas id="canvas" width="800" height="400"></canvas>
      </el-main>
    </el-container>
</template>
<style lang="less" scoped>
canvas {
  border: 1px solid #ccc;
}
.canvas-box {
  display: flex;
  .control-btn {
    display: block;
    width: 78px;
    margin-top: 2px;
    text-align: center;
    line-height: 40px;
    border: 1px solid #ccc;
    cursor: pointer;
    border-radius: 5px;
    &:hover {
      background-color:#ECF5FF;
      border-color: #C6E2FF;
      color: #409EFF;
    }
  }
}
</style>