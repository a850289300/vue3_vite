<script setup lang="ts">
// fabric 图形的 left top 指的是外层框子的 最小top 最小left （多边形中不是第一个点的位置）
import { onMounted, onUpdated, onUnmounted } from 'vue'
import { fabric } from "fabric";
interface mouseData {
  cx: number,
  cy: number
}

// 不知道具体值的 object
interface uncertainObject {
  [propName: string]: any
}

// 图形数据的 object
interface graphicalObject {
  top: number,
  left: number,
  angle: number,
  type: string,
  [propName: string]: any
}
let canvas:any;
let mouseStatus:boolean = false; // 绘制状态
let mouseData: mouseData = {
  cx: 0,
  cy: 0
}
let drawType: string = ''; // 绘制类型
let currentDraw: any = null; // 当前图形
let selected: Array<any> = []; // 选择的元素 
let canvasList: Array<any> = []; // 所有canvas列表
onMounted(()=>{
  canvas = new fabric.Canvas("canvas"); //可以通过鼠标方法缩小,旋转
  initCanvas(canvas)
})

// 初始化canvas
function initCanvas(canvas:any){
  canvas.clear();
  bindEvent(canvas)
  getData();
}

// 初始化绑定事件
function bindEvent(canvas:any) {
  canvas.on(
    {
      'mouse:down': (e:any) => mouseDown(e), // 鼠标mousedown
      'mouse:up': (e:any) => mouseUp(e), // 鼠标 mouseup
      'mouse:move': (e:any) => mouseMove(e), // 鼠标移动
      'object:added': (e:any) => addObject(e), // 添加元素
      'object:modified': (e:any) => modifyObject(e), // 修改元素
      'mouse:dblclick': (e:any) => changeStatus(e), // 修改元素状态
      'selection:created': (e:any) => selectObject(e) // 选中元素
    }
  )
}


// 修改元素
function modifyObject(e:any) {
  console.log('修改', e)
}

// 添加元素
function addObject(e:any) {
}

// 选中元素
function selectObject(e:any) {
  selected = e.selected || []
}

// 点击事件
function mouseDown(e:any) {
  let pointer = e.pointer || {}
  let { x, y } = pointer;
  mouseStatus = true;
  mouseData = {
    cx : x,
    cy : y
  }
  if (mouseStatus) {
    switch(drawType) {
      case 'polygon':
        drawPolygon(e)  
      default: 
    }
  }
}

function changeStatus(e:any) {
  let {target} = e;
  Edit(target);
}

function mouseUp(e:any) {
  mouseStatus = false;
  if (drawType == 'rect') {
    endDrawing()
  }
}

function endDrawing() {
  if (currentDraw) {
    currentDraw = null;
    setDrawType('')
  }
}

// 移动事件
function mouseMove(e:any) {
  if (mouseStatus) {
    switch(drawType) {
      case 'rect':
        drawRect(e)
        break;
      case 'polygon':
        drawLineByPolygon(e)
        break;
      default: 
    }
  }
}

// 设置类型
function setDrawType(type:string) {
  drawType = type
}

// 绘制线条 多边形的线
function drawLineByPolygon(e: any) {

}

function polygonPositionHandler(dim:any, finalMatrix:any, fabricObject:any) {
	  try {
      const self: any = this; // 
      var x = (fabricObject.points[self.pointIndex].x - fabricObject.pathOffset.x),
		    y = (fabricObject.points[self.pointIndex].y - fabricObject.pathOffset.y);
		return fabric.util.transformPoint(
			{ x: x, y: y },
      fabric.util.multiplyTransformMatrices(
        fabricObject.canvas.viewportTransform,
        fabricObject.calcTransformMatrix()
      )
		);
    }catch(e){}
}

function actionHandler(eventData:any, transform:any, x:any, y:any) {
	var polygon = transform.target,
	    currentControl = polygon.controls[polygon.__corner],
	    mouseLocalPosition = polygon.toLocalPoint(new fabric.Point(x, y), 'center', 'center'),
      polygonBaseSize = polygon._getNonTransformedDimensions(),
			size = polygon._getTransformedDimensions(0, 0),
			finalPointPosition = {
				x: mouseLocalPosition.x * polygonBaseSize.x / size.x + polygon.pathOffset.x,
				y: mouseLocalPosition.y * polygonBaseSize.y / size.y + polygon.pathOffset.y
			};
	polygon.points[currentControl.pointIndex] = finalPointPosition;
	return true;
}

function Edit(poly:any) {
    canvas.setActiveObject(poly);
		poly.edit = !poly.edit;
		if (poly.edit) {
      var lastControl = poly.points.length - 1;
      poly.cornerStyle = 'circle';
      poly.cornerColor = 'rgba(0,0,255,0.5)';
	    poly.controls = poly.points.reduce(function(acc:any, point:any, index:any) {
				acc['p' + index] = new fabric.Control({ // 生成控制点
					positionHandler: polygonPositionHandler, // 生成点的位置
					actionHandler: anchorWrapper(index > 0 ? index - 1 : lastControl, actionHandler), // 移动的事件
					actionName: 'modifyPolygon',
					pointIndex: index
				});
				return acc;
			}, { });
		} else {
      poly.cornerColor = 'blue';
      poly.cornerStyle = 'rect';
			poly.controls = fabric.Object.prototype.controls; // 默认的控制
		}
		poly.hasBorders = !poly.edit;
		canvas.requestRenderAll(); // 重新渲染（）
}

function anchorWrapper(anchorIndex:any, fn:any) {
  return function(eventData:any, transform:any, x:any, y:any) {
    var fabricObject = transform.target,
        absolutePoint = fabric.util.transformPoint({
            x: (fabricObject.points[anchorIndex].x - fabricObject.pathOffset.x),
            y: (fabricObject.points[anchorIndex].y - fabricObject.pathOffset.y),
        }, fabricObject.calcTransformMatrix()),
        actionPerformed = fn(eventData, transform, x, y),
        newDim = fabricObject._setPositionDimensions({}),
        polygonBaseSize = fabricObject._getNonTransformedDimensions(),
        newX = (fabricObject.points[anchorIndex].x - fabricObject.pathOffset.x) / polygonBaseSize.x,
		    newY = (fabricObject.points[anchorIndex].y - fabricObject.pathOffset.y) / polygonBaseSize.y;
    fabricObject.setPositionByOrigin(absolutePoint, newX + 0.5, newY + 0.5);
    return actionPerformed;
  }
}

// 绘制矩形
function drawRect(e:any) {
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

// 绘制多边形
function drawPolygon(e:any) {
  let points = [];
  let { cx, cy} = mouseData;
  let { pointer = {}} = e;
  let { x, y } = pointer;
  if (currentDraw) {
    points = currentDraw.points || [];
    cx = currentDraw.left;
    cy = currentDraw.top;
    canvas.remove(currentDraw);
  }
  points.push({
    x,
    y
  })
  currentDraw = new fabric.Polygon(points,
    {
      // left: 0, // 设置会导致偏移量错误
      // top: 0,
      fill: '',
      stroke: 'green',
      cornerColor: 'blue',
      objectCaching: false,
		  transparentCorners: false,
    }
  )
  canvas.add(currentDraw)
  // Edit(currentDraw)
}

function saveData() {
  let objs = canvas._objects || [];
  objs.forEach((item:uncertainObject)=> {
    let {top = 0, left = 0, width = 0, height = 0, angle = 0, scaleX = 1, scaleY = 1} = item as any;
    let type = item.get('type');
    let data:graphicalObject = {
      top,
      left,
      angle,
      type,
      scaleX,
      scaleY
    }
    if (type == 'polygon') { // 多边形
      data['points'] = item.get("points");
    } else if (type == 'rect') { // 矩形
      data['width'] = width
      data['height'] = height
    }
    canvasList.push(data)
  })
  localStorage.setItem('canvasData', JSON.stringify(canvasList))
}

// 多边形 点转化
function pointConversion (poly:any) {
  let matrix = poly.calcTransformMatrix();
  let transformedPoints = poly.get("points")
    .map(function(p:any){
      return new fabric.Point(
         p.x - poly.pathOffset.x,
         p.y - poly.pathOffset.y);
    })
  .map(function(p:any){
    return fabric.util.transformPoint(p, matrix);
  });
}

// 删除元素
function remove () {
  if (Array.isArray(selected)) {
    selected.forEach(item => {
      canvas.remove(item)
    })
  }
}

// 获取元素并渲染
function getData() {
  let canvasList = localStorage.getItem('canvasData');
  if (canvasList) {
    canvasList = JSON.parse(canvasList) || [];
    (canvasList as any).forEach((item: any)=> {
      let current;
      if (item.type == 'polygon') {
        current = new fabric.Polygon(item.points,
          {
            // left: item.top, // 设置会导致偏移量错误
            // top: item.left,
            scaleY: item.scaleY,
            scaleX: item.scaleX,
            fill: '',
            stroke: 'green',
            cornerColor: 'blue',
            objectCaching: false,
		        transparentCorners: false,
          }
        )
      } else if (item.type == 'rect') {
        current = new fabric.Rect({
          top: item.top,
          left: item.left,
          angle: item.angle,
          height: item.height,
          width: item.width,
          scaleY: item.scaleY,
          scaleX: item.scaleX,
          fill: '',
          stroke: 'red'
        });
      }
      canvas.add(current)
    })
  }
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
          content="绘制多边形"
        >
          <template #reference>
            <span class="control-btn" @click="setDrawType('polygon')">多边形</span>
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
        <el-popover
          width="80"
          placement="right"
          trigger="hover"
          content="保存数据"
        >
          <template #reference>
            <span class="control-btn" @click="saveData()">保存</span>
          </template>
        </el-popover>
         <el-popover
          width="80"
          placement="right"
          trigger="hover"
          content="删除框"
        >
          <template #reference>
            <span class="control-btn" @click="remove()">删除</span>
          </template>
        </el-popover>
      </el-aside>
      <el-main>
        <canvas id="canvas" width="500" height="400"></canvas>
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