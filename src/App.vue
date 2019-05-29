<template>
  <div id="app">
    <div style="height: 500px; width: 500px; border: 1px solid red; position: relative;">
      <vue-draggable-resizable
        ref = 'resizable'
        :x="x"
        :y="y"
        :w="w"
        :h="h"
        :parent="true"
        :debug="true"
        @dragging = "onDragging"
        @resizing = 'onResizing'
      >
      </vue-draggable-resizable>


      <div v-for="(item, index) in lsComponentList">
        <div
          class="componentItem"
          :style="{top: item.top+'px', left: item.left+'px', width:item.width+'px', height:item.height+'px'}"
          :class="{componentItem_border: item.isSelect}"
          @click="componentItemClickHandle(index)"
        >我是元素{{index}}
        </div>
      </div>


    </div>
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
        {isSelect: 0, width: 150,  height: 150, left: 10, top: 100},
        {isSelect: 0, width: 120, height: 110, left: 30, top: 32},
        {isSelect: 0, width: 120, height: 110, left: 30, top: 34},
      ]
    }
  },
  mounted(){
    this._calculateXYWH() // 计算x, y, w, h

  },
  computed: {
    style () {
      return {
        position: 'absolute',
        top: this.top + 'px',
        left: this.left + 'px',
      }
    },
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
          _arrX.push(v.left)
          _arrY.push(v.top)
          _arrR.push(v.left+v.width)
          _arrB.push(v.top+v.height)
        }
      }


      this.maxLeft = this.x = Math.min(..._arrX)
      this.maxTop = this.y = Math.min(..._arrY)
      this.maxWidth = this.w = Math.max(..._arrR)-this.maxLeft+2
      this.maxHeight = this.h = Math.max(..._arrB)-this.maxTop+2
    },
    onDragging(left, top){ // 拖拽移动的时候
      var lsComponentList = this.lsComponentList
      for (let v of lsComponentList) {
        if(v.isSelect){ // 被选中才移动
          v.left = left-this.maxLeft+ v.x
          v.top = top-this.maxTop+ v.y
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
          v.left = left-this.maxLeft+ v.x
          v.top = top-this.maxTop+ v.y
          v.width =  width * v.pidW
          v.height =  height * v.pidH
        }
      }
      this.lsComponentList = lsComponentList
      this.x = left
      this.t = top
      this.w = width
      this.h = height
    },
    componentItemClickHandle(index) { // 点击某个元素发生的事情
      var lsComponentList = this.lsComponentList
      for (let v of lsComponentList) {
        v.x = v.left
        v.y = v.top
      }
      lsComponentList[index].isSelect = 1
      this.lsComponentList = lsComponentList
      this._calculateXYWH()
      for (let v of lsComponentList) {
        v.pidW = v.width/this.w
        v.pidH = v.height/this.h
      }

    }
  }
}
</script>

<style>
  .componentItem_border{ border: 1px solid #333333}
  .componentItem{position: absolute;}
</style>
