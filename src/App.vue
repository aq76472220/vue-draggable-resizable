<template>
  <div id="app">
    <div style="height: 500px; width: 500px; border: 1px solid red; position: relative;">
      <vue-draggable-resizable
        ref = 'resizable'
        :x="left"
        :y="top"
        :w="width"
        :h="height"
        :parent="true"
        :debug="true"
        @dragging = "onDragging"
        @resizing = 'onResizing'
      >
        <div>1322</div>
      </vue-draggable-resizable>

      <div>
        <div v-for="(item, index) in list">
          <div :style="item.isClick?style:''" @click="componentClickHandle($event,index)">我是元素{{index}}</div>
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
      top: 0,
      left: 0,
      width: 0,
      height: 0,
      list: [{isClick: 0, width: 150, height: 150, left: 10, top: 20},{isClick: 0, width: 120, height: 110, left: 30, top: 30}]
    }
  },
  computed: {
    style () {
      return {
        position: 'absolute',
        top: this.top + 'px',
        left: this.left + 'px',
        width: this.width + 'px',
        height: this.height + 'px'
      }
    },
  },
  components: {
    VueDraggableResizable
  },
  methods: {
    onDragging(left, top){ // 拖拽移动的时候
      this.left = left
      this.top = top
    },
    onResizing(left, top, width, height){ // 放大缩小的时候
      this.left = left
      this.top = top
      this.width = width
      this.height = height
    },
    componentClickHandle (e,index) { // 点击某个元素发生的事情
      var list = this.list
      for (let v of list){
        console.log(v)
        v.isClick = 0
      }

      this.left = list[index].left
      this.top = list[index].top
      this.width = list[index].width
      this.height = list[index].height

      list[index].isClick = 1
      this.list = list
      console.log(this.list)
    }
  }
}
</script>

