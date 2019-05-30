<template>
  <div id="app">
    <div style="height: 400px; width: 800px; border: 1px solid red; position: relative;">
      <vue-draggable-resizable
        ref = 'resizable'
        :x="x"
        :y="y"
        :w="w"
        :h="h"
        :parent="true"
        :debug="true"
        @dragging = "onDragging"
        @resizing = "onResizing"
        @resizestop = "onResizstop"
      >
      </vue-draggable-resizable>
      <div v-for="(item, index) in lsComponentList">
        <div
          class="componentItem"
          :style="{top: item.css.y+'px', left: item.css.x+'px', width:item.css.width+'px', height:item.css.height+'px'}"
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
      maxWidth: 0, // 拖拽框的宽度
      maxHeight: 0, // 拖拽框的高度
      maxTop: 0, //计算最大的top值
      maxLeft: 0, //计算最左边的值
      lsComponentList: [
        {isSelect: 0,
          css: {
            width: 150,  height: 150, x: 10, y: 100
          }
         },
        {isSelect: 0,
          css: {
            width: 150,  height: 150, x: 10, y: 100
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

      console.log(  _arrX,  Math.floor(Math.min(..._arrX)), '???')

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
      console.log(left, top)
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
    componentItemClickHandle(index) { // 点击某个元素发生的事情
      var lsComponentList = this.lsComponentList
      for (let v of lsComponentList) {
        v.x = v.css.x
        v.y = v.css.y
      }
      lsComponentList[index].isSelect = 1
      this.lsComponentList = lsComponentList
      this._calculateXYWH()
      for (let v of lsComponentList) {
        v.pidW = v.css.width/this.w
        v.pidH = v.css.height/this.h
        v.pidX = (v.css.x - this.x)/this.w
        v.pidY= (v.css.y - this.y)/this.h
      }
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
              v.x = this.x+this.w-v.css.width
              v.y = v.css.y
            }

          }
          break;
        case 'bottom': // 下对齐
          for (let v of lsComponentList){
            if(v.isSelect){
              v.css.y = this.y+this.h-v.css.height
              v.x = v.css.x
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
  .componentItem_border:after{content:'';position: absolute; left: 0; top: 0; right: 0; bottom: 0; border: 1px solid #333333}
  .componentItem{position: absolute;}
</style>
