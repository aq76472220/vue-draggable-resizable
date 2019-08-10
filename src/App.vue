<template>
  <div style="height:1000px; width: 1000px; margin-left: 100px;overflow: auto;" class="scrollFathter">
 
    <!--编辑文本框-->
    <div
        class="absolute textsFocus isEditor"
        contenteditable="true"
        @input= "onInputHandle"
    >
      1.1.8
    </div>
    
    
    <div  style="height:1200px; width: 1200px; margin-top:400px;" class="cancelSelected">
      <div class="fathter cancelSelected" style="height: 600px; width: 600px; border: 1px solid red; position: relative;">
        <vue-draggable-resizable
          ref = 'resizable'
          :x="x"
          :y="y"
          :w="w"
          :h="h"
          :r="r"
          :isCanDragSelect="true"
          :isRotate="isRotate"
          parent="fathter"
          scrollFathter = 'scrollFathter'
          :debug="true"
          :isCanDirMove="true"
          :grid="[1,1]"
          :preventDeactivation = "preventDeactivation"
          @dragging = "onDragging"
          @dragstop = "onDragstop"
          @resizing = "onResizing"
          @resizestop = "onResizstop"
          @rotateing = "onRotateing"
          @dragSelecting = "onDragSelecting"
          @deactivated = "onDeactivated"
          @dirmoveing = "onDirmoveing"
          @dirmoveup = "onDirmoveup"
        >
        </vue-draggable-resizable>
        <div v-for="(item, index) in lsComponentList"  class="componentItem_box">
          <div
            class="componentItem  n_active"
            :style="{top: item.css.y+'px', left: item.css.x+'px', width:item.css.width+'px', height:item.css.height+'px',transform:'rotate('+ item.css.r +'deg)',zIndex: item.css.z}"
            :class="{componentItem_border: item.isSelect}"
            @mousedown.stop.prevent="componentItemHandle($event, index)"
          >我是元素{{index}}
          </div>
        </div>
      </div>
      <!--<ul class="ss-ul-box n_active" @mousedown.stop>-->
        <!--<li @mousedown.stop="onPositonHandle($event, 'verticalcenter')" class="n_active">垂直居中</li>-->
        <!--<li @mousedown.stop="onPositonHandle($event, 'horizontalcenter')"  class="">水平居中</li>-->
        <!--<li @mousedown.stop.prevent="onPositonHandle($event, 'left')" class="">左</li>-->
        <!--<li @mousedown.stop.prevent="onPositonHandle($event, 'top')" class="">上</li>-->
        <!--<li @mousedown.stop.prevent="onPositonHandle($event, 'right')" class="">右</li>-->
        <!--<li @mousedown.stop.prevent="onPositonHandle($event, 'bottom')"class="">下</li>-->
      <!--</ul>-->
      <!--<input type="text"  placeholder="请输入文字.." />-->
    </div>
  </div>
</template>

<script>
import VueDraggableResizable from './components/vue-draggable-resizable'
import _ from 'lodash'
import './components/vue-draggable-resizable.css'

export default {
  name: 'app',
  data () {
    return {
      w: 0,
      h: 0,
      x: 0,
      y: 0,
      r: 0,
      preventDeactivation: false,
      isRotate: true,
      isSlectResiz: false, // 是否有选中的
      maxWidth: 0, // 拖拽框的宽度
      maxHeight: 0, // 拖拽框的高度
      maxTop: 0, // 计算最大的top值
      maxLeft: 0, // 计算最左边的值
      lsComponentList: [
        { isSelect: 0,
          css: {
            width: 150, height: 121, x: 100, y: 200, r: 0, z: 3
          }
        },
        { isSelect: 0,
          css: {
            width: 110, height: 111, x: 10, y: 20, r: 0, z: 4
          }
        },
        { isSelect: 0,
          css: {
            width: 80, height: 201, x: 100, y: 200, r: 0, z: 5
          }
        },
        { isSelect: 0,
          css: {
            width: 80, height: 201, x: 100, y: 200, r: 0, z: 5
          }
        },
        { isSelect: 0,
          css: {
            width: 80, height: 201, x: 100, y: 200, r: 0, z: 5
          }
        }
      ]
    }
  },
  mounted () {
      document.onkeydown = function(event){
      }
    this._calculateXYWH(true) // 计算x, y, w, h
  },
  computed: {

  },
  components: {
    VueDraggableResizable
  },
  watch: {

  },
  methods: {
    onInputHandle(e){
      let selection = window.getSelection();
      let range = selection.getRangeAt(0);
      console.log(range, '...拖蓝...')
    },
    onDeactivated (e) { // 点击document发生的事情
      var target = e.target || e.srcElement
      var regex = new RegExp('cancelSelected')
      if (regex.test(target.className)){
        var lsComponentList = this.lsComponentList
        if (!e.shiftKey) {
          for (let v of lsComponentList) {
            v.isSelect = 0
          }
          this.lsComponentList = lsComponentList
          this.x = 0
          this.y = 0
          this.w = 0
          this.h = 0
          this.isRotate = false
        }
      }
    },
    _calculateXYWH (isAlign = false) { // 计算x, y, w, h  isAlign 是否走的是对其的
      var _arrX = []
      var _arrY = []
      var _arrR = []
      var _arrB = []
      var lsComponentList = this.lsComponentList
      for (let v of lsComponentList) {
        if (v.isSelect) {
          _arrX.push(v.css.x)
          _arrY.push(v.css.y)
          _arrR.push(v.css.x + v.css.width)
          _arrB.push(v.css.y + v.css.height)
        }
      }
      this.maxLeft = this.x = _arrX.length > 0 ? Math.floor(Math.min(..._arrX)) : 0
      this.maxTop = this.y = _arrY.length > 0 ? Math.floor(Math.min(..._arrY)) : 0
      this.maxWidth = this.w = _arrR.length > 0 ? Math.floor(Math.max(..._arrR) - this.maxLeft) : 0
      this.maxHeight = this.h = _arrB.length > 0 ? Math.floor(Math.max(..._arrB) - this.maxTop) : 0
      if (isAlign) { // 对其另行处理
        var [parentWidth, parentHeight] = this.$refs.resizable.getParentSize()
        this.$refs.resizable.left = this.maxLeft
        this.$refs.resizable.top = this.maxTop
        this.$refs.resizable.right = parentWidth - this.maxLeft - this.maxWidth
        this.$refs.resizable.bottom = parentHeight - this.maxTop - this.maxHeight
      }
      for (let v of lsComponentList) {
        if (v.isSelect) {
          v.pidW = v.css.width / this.w
          v.pidH = v.css.height / this.h
          v.pidX = (v.css.x - this.x) / this.w
          v.pidY = (v.css.y - this.y) / this.h
          v.cx = v.css.x - this.maxLeft
          v.cy = v.css.y - this.maxTop
        }
      }
      this.lsComponentList = lsComponentList
    },
    componentItemHandle (e, index) { // 点击某个元素发生的事情
      this.$refs.resizable.checkParentSize() // mmp
      var lsComponentList = this.lsComponentList
      let num = 0
      for (let v of lsComponentList) {
        if (v.isSelect) {
          num++
        }
      }
      if (e.shiftKey) {
        console.log(e.shiftKey)
        if (lsComponentList[index].isSelect) {
          lsComponentList[index].isSelect = 0
        } else {
          lsComponentList[index].isSelect = 1
        }
      } else {
        if (!lsComponentList[index].isSelect) {
          for (let v of lsComponentList) {
            v.isSelect = 0
          }
          lsComponentList[index].isSelect = 1
        }
      }
      this.lsComponentList = lsComponentList
      this._calculateXYWH(true)
      for (let v of lsComponentList) {
        v.pidW = v.css.width / this.w
        v.pidH = v.css.height / this.h
        v.pidX = (v.css.x - this.x) / this.w
        v.pidY = (v.css.y - this.y) / this.h
      }
      this.r = lsComponentList[index].css.r
      setTimeout(()=>{ // 尼玛马到处是坑
        this._calculateIsHasRotate() // 计算是否只有一个
      },10)
      this.$refs.resizable.elementDown(e, this.$refs.resizable.$el)
    },
    onDragging (left, top) { // 拖拽移动的时候
      var lsComponentList = _.cloneDeep(this.lsComponentList)
      for (let v of lsComponentList) {
        if (v.isSelect) { // 被选中才移动
          v.css.x = Math.round(left + v.cx)
          v.css.y = Math.round(top + v.cy)
        }
      }
      this.lsComponentList = lsComponentList
      this.x = left
      this.y = top
    },
    onDirmoveing (left, top) { // 上下左右方向键
      var lsComponentList = _.cloneDeep(this.lsComponentList)
      for (let v of lsComponentList) {
        if (v.isSelect) { // 被选中才移动
          v.css.x = Math.round(left + v.cx)
          v.css.y = Math.round(top + v.cy)
        }
      }
      this.lsComponentList = lsComponentList
      this.x = left
      this.y = top
    },
    onDirmoveup(left, top){
      var lsComponentList = _.cloneDeep(this.lsComponentList)
      for (let v of lsComponentList) {
        if (v.isSelect) { // 被选中才移动
          v.css.x = Math.round(left + v.cx)
          v.css.y = Math.round(top + v.cy)
        }
      }
      this.lsComponentList = lsComponentList
      this.x = left
      this.y = top
    },
    onDragstop (left, top) { // 拖拽停止发生的事情
      this.maxLeft = left
      this.maxTop = top
    },
    onResizing (left, top, width, height) { // 放大缩小的时候
      var lsComponentList = _.cloneDeep(this.lsComponentList)
      for (let v of lsComponentList) {
        if (v.isSelect) { // 被选中才移动
          v.css.x = Math.round(width * v.pidX + this.x)
          v.css.y = Math.round(height * v.pidY + this.y)
          v.css.width = Math.round(width * v.pidW)
          // v.css.height = Math.round(height * v.pidH)
        }
      }
      this.lsComponentList = lsComponentList
      this.x = left
      this.y = top
      this.w = width
      this.h = height
      this._correct() // 矫正
    },
    _correct () {
      var lsComponentList = this.lsComponentList
      for (let v of lsComponentList) {
        if (((v.css.y + v.css.height) > this.y + this.h) && v.isSelect) {
          v.css.height = this.y + this.h - v.css.y
        }

        if (((v.css.x + v.css.width) > this.x + this.w) && v.isSelect) {
          v.css.width = this.x + this.w - v.css.x
        }
      }
      this.lsComponentList = lsComponentList
    },
    onResizstop (left, top, width, height) { // 拖拽结束后
      this._calculateXYWH()
      var lsComponentList = this.lsComponentList
      for (let v of lsComponentList) {
        v.pidW = v.css.width / this.w
        v.pidH = v.css.height / this.h
        v.pidX = (v.css.x - this.x) / this.w
        v.pidY = (v.css.y - this.y) / this.h
      }
    },
    onRotateing (rotate) { // 旋转的时候发生的事情
      var lsComponentList = this.lsComponentList
      for (let v of lsComponentList) {
        if (v.isSelect) {
          v.css.r = rotate
        }
      }
      this.r = rotate // 日了狗了
      this.lsComponentList = lsComponentList
    },
    onDragSelecting (s, m) { // 拖拽选框发生的事情 s:开始位置， m移动的位置
      console.log(s, m, '...位置变动...')
      let lsComponentList = this.lsComponentList
      let mousePosition = []
      mousePosition[0] = s
      mousePosition[1] = m
      for (let v of lsComponentList) {
        if (this.isBaohan2Points([v.css.x, v.css.y], [v.css.x + v.css.width, v.css.y + v.css.height], mousePosition)) {
          v.isSelect = 1
        } else {
          v.isSelect = 0
        }
      }
      this.lsComponentList = lsComponentList
      this._calculateIsHasRotate()
      this._calculateXYWH(true)
    },
    _calculateIsHasRotate () {
      let lsComponentList = this.lsComponentList
      let num = 0
      for (let v of lsComponentList) {
        if (v.isSelect) {
          num++
        }
      }
      if (num == 1) { //
        this.isRotate = true
        for (let v of lsComponentList) {
          if (v.isSelect) {
            this.r = v.css.r
          }
        }
      } else {
        this.r = 0
        this.isRotate = false
      }
    },

    // 元素四个点是否都在范围内
    isBaohan2Points (lt, rb, areas) {
      return this.isBaohanPoint(lt, areas) && this.isBaohanPoint(rb, areas)
    },
    // 一个点的xy坐标是否包含在范围之内;[121,321],[[123,456],[321,567]] (x,y) ((x,y),(x,y))
    isBaohanPoint (point, areas) {
      // 取x区域和y区域
      let xArr = [areas[0][0], areas[1][0]]// x的范围值
      let yArr = [areas[0][1], areas[1][1]]// y的范围值
      return this.isBaohanNum(point[0], xArr) && this.isBaohanNum(point[1], yArr)
    },
    // 一个坐标数据是否在区域内; 234 ,[123,456]  x(y)坐标;  x(y)取值范围
    isBaohanNum (position, area) {
      let start = Math.min(area[0], area[1])// 开始位置
      let end = Math.max(area[0], area[1])// 结束为止
      return position >= start && position <= end
    },
    onPositonHandle (e, type) { // 元素对齐
      // this.preventDeactivation = true
      var lsComponentList = this.lsComponentList
      switch (type) {
        case 'left': // 左对齐
          for (let v of lsComponentList) {
            if (v.isSelect) {
              v.css.x = this.x
            }
          }
          break
        case 'top': // 上对齐
          for (let v of lsComponentList) {
            if (v.isSelect) {
              v.css.y = this.y
            }
          }
          break
        case 'right': // 右对齐
          for (let v of lsComponentList) {
            if (v.isSelect) {
              v.css.x = this.x + this.w - v.css.width
            }
          }
          break
        case 'bottom': // 下对齐
          for (let v of lsComponentList) {
            if (v.isSelect) {
              v.css.y = this.y + this.h - v.css.height
            }
          }
          break
        case 'verticalcenter': // 垂直居中
          let verticalcenter_y = this.y + this.h / 2
          for (let v of lsComponentList) {
            if (v.isSelect) {
              v.css.y = verticalcenter_y - v.css.height / 2
            }
          }
          break
        case 'horizontalcenter': // 水平居中
          let horizontalcenter_x = this.x + this.w / 2
          for (let v of lsComponentList) {
            if (v.isSelect) {
              v.css.x = horizontalcenter_x - v.css.width / 2
            }
          }
          break
          this.lsComponentList = lsComponentList
      }
      this._calculateXYWH(true) // 重新计算位置
    }
  }
}
</script>

<style>
  *{padding: 0; margin: 0}
  .ss-ul-box li{padding: 15px 0;}
  .componentItem_box{position: relative; z-index: 1}
  .componentItem_border:after{content:'';position: absolute; left: 0; top: 0; right: 0; bottom: 0; border: 1px solid #1486ff}
  .componentItem{position: absolute;  background: #eee}
  .componentItem:hover:after{content:'';position: absolute; left: 0; top: 0; right: 0; bottom: 0; border: 1px solid #1486ff}
</style>
