<template>
  <div id="app">
    <div style="height: 500px; width: 500px; border: 1px solid red; position: relative;">
      <vue-draggable-resizable
        ref = 'resizable'
        :x="maxLeft"
        :y="maxTop"
        :w="maxWidth"
        :h="maxHeight"
        :parent="true"
        :debug="true"
        @dragging = "onDragging"
        @resizing = 'onResizing'
      >

      </vue-draggable-resizable>


        <div v-for="(item, index) in lsComponentList">
          <div :style="{top: item.top+'px', left: item.left+'px', width:item.width+'px', height:item.height+'px'}" class="componentItem">我是元素{{index}}</div>
        </div>
        <!--<div v-for="(item, index) in list">-->
          <!--<div :style="item.isClick?style:''" @click="componentClickHandle($event,index)">我是元素{{index}}</div>-->
        <!--</div>-->


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
      maxWidth: 100, // 拖拽框的宽度
      maxHeight: 200, // 拖拽框的高度
      maxTop: 30, //计算最大的top值
      maxLeft: 10, //计算最左边的值
      lsComponentList: [
        {isClick: 0, width: 150, height: 150, left: 10,x:10, top: 100, y:100 },
        {isClick: 0, width: 120, height: 110, left: 30,x:30, top: 30, y:30}
      ]
    }
  },
  mounted(){
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
    onDragging(left, top){ // 拖拽移动的时候
      console.log(left, top)
      var lsComponentList = this.lsComponentList
      for (let v of lsComponentList) {
        v.left = left-this.maxLeft+ v.x
        v.top = top-this.maxTop+ v.y
      }
      this.lsComponentList = lsComponentList
    },
    onResizing(left, top, width, height){ // 放大缩小的时候
      this.left = left
      this.top = top
      this.width = width
      this.height = height
    },
    // componentClickHandle (e,index) { // 点击某个元素发生的事情
    //   var list = this.list
    //   for (let v of list){
    //     console.log(v)
    //     v.isClick = 0
    //   }
    //
    //   this.left = list[index].left
    //   this.top = list[index].top
    //   this.width = list[index].width
    //   this.height = list[index].height
    //
    //   list[index].isClick = 1
    //   this.list = list
    //   this.$refs.resizable.elementDown(e)
    // }
  }
}
</script>

<style>
  .componentItem{position: absolute; border: 1px solid #333333}
</style>
