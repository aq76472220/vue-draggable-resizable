<template>
  <div id="app">
    <div style="height: 400px; width: 800px; border: 1px solid red; position: relative;">
      <vue-draggable-resizable
        ref = 'resizable'
        :x="x"
        :y="y"
        :w="w"
        :h="h"
        :r="r"
        :parent="true"
        :debug="true"
        @dragging = "onDragging"
        @resizing = "onResizing"
        @resizestop = "onResizstop"
        @rotateing = "onRotateing"
      >
      </vue-draggable-resizable>
      <div v-for="(item, index) in lsComponentList">
        <div
          class="componentItem"
          :style="{top: item.css.y+'px', left: item.css.x+'px', width:item.css.width+'px', height:item.css.height+'px',transform:'rotate('+ item.css.r +'deg)'}"
          :class="{componentItem_border: item.isSelect}"
          @click="componentItemClickHandle(index)"
        >我是元素{{index}}
        </div>
      </div>
    </div>
    <ul class="ss-ul-box">
      <li @click="onPositonHandle('left')">左</li>
      <li @click="onPositonHandle('top')">上</li>
      <li @click="onPositonHandle('right')">右</li>
      <li @click="onPositonHandle('bottom')">下</li>
    </ul>
  </div>
</template>

<script>
import VueDraggableResizable from './components/vue-draggable-resizable'
import './components/vue-draggable-resizable.css'

export default {
  props: {
    classNameHandle: {
      type: String,
      default: 'ss-handle'
    },
  },
  name: 'app',
  data(){
    return {
      w:0,
      h:0,
      x:0,
      y:0,
      r:0,
      maxWidth: 0, // 拖拽框的宽度
      maxHeight: 0, // 拖拽框的高度
      maxTop: 0, //计算最大的top值
      maxLeft: 0, //计算最左边的值
      lsComponentList: [
        {isSelect: 0,
          css: {
            width: 150,  height: 150, x: 10, y: 100, r: 20
          }
         },
        {isSelect: 0,
          css: {
            width: 150,  height: 150, x: 10, y: 100, r: 0
          }
        },
        {isSelect: 0,
          css: {
            width: 150,  height: 150, x: 10, y: 100, r: 0
          }
        }
      ]
    }
  },
  mounted(){
    this._calculateXYWH() // 计算x, y, w, h

  },
  computed: {

  },
  components: {
    VueDraggableResizable
  },
  methods: {
    _calculateXYWH(){ // 计算x, y, w, h
      var _arrX = []
      var _arrY = []
      var _arrR = []
      var _arrB = []
      var lsComponentList = this.lsComponentList;
      for (let v of lsComponentList) {
        if(v.isSelect){
          _arrX.push(v.css.x)
          _arrY.push(v.css.y)
          _arrR.push(v.css.x+v.css.width)
          _arrB.push(v.css.y+v.css.height)
        }
      }
      this.maxLeft = this.x = Math.floor(Math.min(..._arrX))
      this.maxTop = this.y = Math.floor(Math.min(..._arrY))
      this.maxWidth = this.w = Math.floor(Math.max(..._arrR)-this.maxLeft)
      this.maxHeight = this.h = Math.floor(Math.max(..._arrB)-this.maxTop)
      for (let v of lsComponentList) {
        if(v.isSelect){
          v.pidW = v.css.width/this.w
          v.pidH = v.css.height/this.h
          v.pidX = (v.css.x - this.x)/this.w
          v.pidY = (v.css.y - this.y)/this.h
        }
      }



    },
    onDragging(left, top){ // 拖拽移动的时候
      var lsComponentList = this.lsComponentList
      for (let v of lsComponentList) {
        if(v.isSelect){ // 被选中才移动
          v.css.x = Math.floor(left + v.x- this.maxLeft)
          v.css.y = Math.floor(top + v.y - this.maxTop)
        }
      }
      this.lsComponentList = lsComponentList
      this.x = left
      this.y = top
    },
    onResizing(left, top, width, height){ // 放大缩小的时候
      var lsComponentList = this.lsComponentList
      for (let v of lsComponentList) {
        if(v.isSelect){ // 被选中才移动
          v.css.x = Math.floor(width * v.pidX + this.x)
          v.css.y = Math.floor(height * v.pidY + this.y)
          v.css.width = Math.floor(width * v.pidW)
          v.css.height = Math.floor(height * v.pidH)
        }
      }
      this.lsComponentList = lsComponentList
      this.x = left
      this.y = top
      this.w = width
      this.h = height
    },
    onResizstop (left, top, width, height) { // 拖拽结束后
      this._calculateXYWH()
      var lsComponentList = this.lsComponentList
      for (let v of lsComponentList) {
        v.x = v.css.x
        v.y = v.css.y
        v.pidW = v.css.width/this.w
        v.pidH = v.css.height/this.h
        v.pidX = (v.css.x - this.x)/this.w
        v.pidY = (v.css.y - this.y)/this.h
      }
    },
    onRotateing(rotate){ // 旋转的时候发生的事情
      var lsComponentList = this.lsComponentList
      for (let v of lsComponentList) {
        if(v.isCanRotate){
          v.css.r = rotate
        }
      }
      this.lsComponentList = lsComponentList
    },
    componentItemClickHandle(index) { // 点击某个元素发生的事情
      var lsComponentList = this.lsComponentList
      for (let v of lsComponentList) {
        v.x = v.css.x
        v.y = v.css.y
        v.isCanRotate = 0
      }
      lsComponentList[index].isSelect = 1
      lsComponentList[index].isCanRotate = 1
      this.lsComponentList = lsComponentList
      this._calculateXYWH()
      for (let v of lsComponentList) {
        v.pidW = v.css.width/this.w
        v.pidH = v.css.height/this.h
        v.pidX = (v.css.x - this.x)/this.w
        v.pidY= (v.css.y - this.y)/this.h
      }

      this.r = lsComponentList[index].css.r
    },

    onPositonHandle (type) { // 元素对齐
      var lsComponentList = this.lsComponentList
      switch (type) {
        case 'left': // 左对齐
          for (let v of lsComponentList){
            if(v.isSelect){
              v.css.x = this.x
              v.x =  this.x
              v.y = v.css.y
            }
          }
          break;
        case 'top': // 上对齐
          for (let v of lsComponentList){
            if(v.isSelect){
              v.css.y = this.y
              v.x =  v.css.x
              v.y = this.y
            }
          }
          break;
        case 'right': // 右对齐
          for (let v of lsComponentList){
            if(v.isSelect){
              v.css.x = this.x+this.w-v.css.width
              v.x =  this.$refs.resizable.left = this.x+this.w-v.css.width
              v.y = v.css.y
            }
          }
          break;
        case 'bottom': // 下对齐
          for (let v of lsComponentList){
            if(v.isSelect){
              v.css.y = this.y+this.h-v.css.height
              v.x = this.$refs.resizable.top = v.css.x
              v.y = this.y+this.h-v.css.height
            }
          }
          break;
        this.lsComponentList = lsComponentList
      }
      this._calculateXYWH() // 重新计算位置
    }
  }
}
</script>

<style>
  .ss-ul-box li{padding: 15px 0;}
  .componentItem_border:after{content:'';position: absolute; left: 0; top: 0; right: 0; bottom: 0; border: 2px solid #1486ff}
  .componentItem{position: absolute;}
  .componentItem:hover:after{content:'';position: absolute; left: 0; top: 0; right: 0; bottom: 0; border: 2px solid #1486ff}
</style>




<!--<!DOCTYPE html>-->
<!--<html lang="en">-->
<!--<head>-->
  <!--<meta charset="UTF-8">-->
  <!--<title>Title</title>-->
  <!--<script src="../static/js/jq.js"></script>-->
  <!--<style>-->
    <!--.a, .b, .c {-->
      <!--width: 100px;-->
      <!--height: 100px;-->
      <!--background-color: #fff;-->
      <!--position: absolute;-->
      <!--border: 1px solid #FF00FF;-->
    <!--}-->

    <!--.a {-->
      <!--left: 788px;-->
      <!--top: 144px;-->
    <!--}-->

    <!--.b {-->
      <!--left: 223px;-->
      <!--top: 23px;-->
    <!--}-->

    <!--.c {-->
      <!--left: 455px;-->
      <!--top: 366px;-->
    <!--}-->
  <!--</style>-->
<!--</head>-->
<!--<body>-->
<!--<div class="a"></div>-->
<!--<div class="b"></div>-->
<!--<div class="c"></div>-->
<!--<script>-->
  <!--$(function () {-->
    <!--var mousePosition = [[0, 0], [0, 0]];-->
    <!--$(document).mousedown(function (e) {-->
      <!--mousePosition[0] = [e.clientX, e.clientY];-->
    <!--}).mouseup(function (e) {-->
      <!--mousePosition[1] = [e.clientX, e.clientY];-->
      <!--for (let i = 0; i < $('div').length; i++) {-->
        <!--let el = $('div').eq(i)[0];-->
        <!--if (isBaohan4Points(el, mousePosition))-->
          <!--el.style.backgroundColor ="#f00";-->
        <!--else-->
          <!--el.style.backgroundColor ="#fff";-->
      <!--}-->
    <!--});-->

    <!--//元素四个点是否都在范围内-->
    <!--function isBaohan4Points(el, areas) {-->
      <!--let lt = [el.offsetLeft, el.offsetTop];//左上-->
      <!--let rb = [el.offsetLeft + el.offsetWidth, el.offsetTop + el.offsetHeight];//右下-->
      <!--return isBaohanPoint(lt, areas) && isBaohanPoint(rb, areas);-->
    <!--}-->


    <!--//一个点的xy坐标是否包含在范围之内;[121,321],[[123,456],[321,567]] (x,y) ((x,y),(x,y))-->
    <!--function isBaohanPoint(positions, areas) {-->
      <!--if (!Array.isArray(positions) || positions.length !== 2) {-->
        <!--console.log('positions 不对');-->
        <!--return false;-->
      <!--}-->

      <!--//取x区域和y区域-->
      <!--let xArr = [areas[0][0], areas[1][0]];//x的范围-->
      <!--let yArr = [areas[0][1], areas[1][1]];//y的范围-->

      <!--return isBaohanNum(positions[0], xArr) && isBaohanNum(positions[1], yArr);-->
    <!--}-->


    <!--//一个坐标数据是否在区域内; 234 ,[123,456]  x坐标;  x取值范围-->
    <!--function isBaohanNum(position, area) {-->
      <!--if (!Array.isArray(area) || area.length !== 2) {-->
        <!--console.log('area 不对');-->
        <!--return false;-->
      <!--}-->

      <!--let start = Math.min(area[0], area[1]);//开始位置-->
      <!--let end = Math.max(area[0], area[1]);//结束为止-->

      <!--return isInclude(position, start, end);-->
    <!--}-->

    <!--//判断是否包含-->
    <!--function isInclude(p, start, end) {-->
      <!--return p >= start && p <= end;-->
    <!--}-->
  <!--})-->
<!--</script>-->
<!--</body>-->
<!--</html>-->
