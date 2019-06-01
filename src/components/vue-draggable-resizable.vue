<template>
  <div>
    <div
      :style="style"
      v-if="w||h"
      :class="[{
        [classNameActive]: enabled,
        [classNameDragging]: dragging,
        [classNameResizing]: resizing,
        [classNameDraggable]: draggable,
        [classNameResizable]: resizable
      }, className]"
      @mousedown.stop.prevent="elementDown"
    >
      <div>
        <!--旋转手柄-->
        <div
          v-if="isRotate"
          class="handle handle-rotate"
          @mousedown.stop.prevent="rotateDown($event)">
          <svg width="14" height="14" xmlns="http://www.w3.org/2000/svg"><path d="M10.536 3.464A5 5 0 1 0 11 10l1.424 1.425a7 7 0 1 1-.475-9.374L13.659.34A.2.2 0 0 1 14 .483V5.5a.5.5 0 0 1-.5.5H8.483a.2.2 0 0 1-.142-.341l2.195-2.195z" fill="#1486ff" fill-rule="nonzero"></path></svg>
        </div>
        <div

          v-for="handle in actualHandles"
          :key="handle"
          :class="[classNameHandle, classNameHandle + '-' + handle]"
          :style="{display: enabled ? 'block' : 'none'}"
          @mousedown.stop.prevent="handleDown(handle, $event)"
        >
          <slot :name="handle"></slot>
        </div>
      </div>
      <!--默认插槽-->
      <slot></slot>
    </div>
    <div
      :style="dragSelectStyle"
      v-if="dragSelecting"
      class="ss-dragSelect-box"
    ></div>
  </div>
</template>

<script>
  import { matchesSelectorToParentElements, addEvent, removeEvent } from '../utils/dom'
  const events = {
    mouse: {
      start: 'mousedown',
      move: 'mousemove',
      stop: 'mouseup'
    },
    touch: {
      start: 'touchstart',
      move: 'touchmove',
      stop: 'touchend'
    }
  }

  const userSelectNone = {
    userSelect: 'none',
    MozUserSelect: 'none',
    WebkitUserSelect: 'none',
    MsUserSelect: 'none'
  }

  const userSelectAuto = {
    userSelect: 'auto',
    MozUserSelect: 'auto',
    WebkitUserSelect: 'auto',
    MsUserSelect: 'auto'
  }
  let eventsFor = events.mouse

  export default {
    replace: true,
    name: 'vue-draggable-resizable',
    props: {
      itemData: {
        type: Object,
        default: null
      },
      fathterClass: { // 父标签的class
        type: String,
        default: ''
      },
      isCanDragSelect: { // 是否可以拖拽
        type: Boolean,
        default: false
      },
      debug: {
        type: Boolean,
        default: false
      },
      className: {
        type: String,
        default: 'ss-vdr'
      },
      classNameDraggable: {
        type: String,
        default: 'draggable'
      },
      classNameResizable: {
        type: String,
        default: 'resizable'
      },
      classNameDragging: {
        type: String,
        default: 'dragging'
      },
      classNameResizing: {
        type: String,
        default: 'resizing'
      },
      classNameActive: {
        type: String,
        default: 'active'
      },
      classNameHandle: {
        type: String,
        default: 'ss-handle'
      },
      isRotate: { // 是否要旋转
        type: Boolean,
        default: true
      },
      disableUserSelect: {
        type: Boolean,
        default: true
      },
      enableNativeDrag: {
        type: Boolean,
        default: false
      },
      preventDeactivation: {
        type: Boolean,
        default: false
      },
      active: { // 是否按下选中元素
        type: Boolean,
        default: false
      },
      draggable: {
        type: Boolean,
        default: true
      },
      resizable: {
        type: Boolean,
        default: true
      },
      lockAspectRatio: { // 是否锁住长宽
        type: Boolean,
        default: false
      },
      w: { // 宽度
        type: Number,
        default: 0
      },
      h: { // 高度
        type: Number,
        default: 0
      },
      x: { // x 轴
        type: Number,
        default: 0,
        validator: (val) => typeof val === 'number'
      },
      y: { // y 轴
        type: Number,
        default: 0,
        validator: (val) => typeof val === 'number'
      },

      r: { // y 轴
        type: Number,
        default: 0,
        validator: (val) => typeof val === 'number'
      },
      z: { // 层级关系
        type: [String, Number],
        default: 10000,
        validator: (val) => (typeof val === 'string' ? val === 'auto' : val >= 0)
      },
      minWidth: { // 最小宽度
        type: Number,
        default: 5,
        validator: (val) => val >= 0
      },
      minHeight: { // 最小高度
        type: Number,
        default: 5,
        validator: (val) => val >= 0
      },
      maxWidth: {
        type: Number,
        default: null,
        validator: (val) => val >= 0
      },
      maxHeight: {
        type: Number,
        default: null,
        validator: (val) => val >= 0
      },
      handles: {
        type: Array,
        default: () => ['tl', 'tm', 'tr', 'mr', 'br', 'bm', 'bl', 'ml'],
        validator: (val) => {
          const s = new Set(['tl', 'tm', 'tr', 'mr', 'br', 'bm', 'bl', 'ml'])
          return new Set(val.filter(h => s.has(h))).size === val.length
        }
      },
      dragHandle: {
        type: String,
        default: null
      },
      dragCancel: {
        type: String,
        default: null
      },
      axis: {
        type: String,
        default: 'both',
        validator: (val) => ['x', 'y', 'both'].includes(val)
      },
      grid: { // 网格
        type: Array,
        default: () => [1, 1]
      },
      parent: {
        type: [Boolean, String],
        default: false
      },
      onDragStart: {
        type: Function,
        default: null
      },
      onResizeStart: {
        type: Function,
        default: null
      }
    },

    data: function () {
      return {
        mouseClickPosition: null,
        rawWidth: this.w,
        rawHeight: this.h,
        rawLeft: this.x,
        rawTop: this.y,
        rawRight: null,
        rawBottom: null,
        left: this.x,
        top: this.y,
        rotate: this.r,
        right: null,
        bottom: null,
        aspectFactor: this.w / this.h, // 宽度和长度比例关系
        parentWidth: null, // 父级元素的宽度
        parentHeight: null, // 父级元素的高度
        minW: this.minWidth,
        minH: this.minHeight,
        maxW: this.maxWidth,
        maxH: this.maxHeight,
        handle: null,
        enabled: this.active,
        resizing: false, // 正在放大缩小中
        dragging: false,  // 正在拖拽中
        rotateing: false, // 正在旋转中
        dragSelecting: false, //正在拖拽选择框选元素
        zIndex: this.z,
        dragSelectDate:{left:0, top:0, width:0, height:0}, // 拖拽选择框的大小
      }
    },

    created: function () {
      // eslint-disable-next-line
      if (this.maxWidth && this.minWidth > this.maxWidth) console.warn('[Vdr warn]: Invalid prop: minWidth cannot be greater than maxWidth')
      // eslint-disable-next-line
      if (this.maxWidth && this.minHeight > this.maxHeight) console.warn('[Vdr warn]: Invalid prop: minHeight cannot be greater than maxHeight')
      this.resetBoundsAndMouseState()
    },
    mounted: function () {
      if (!this.enableNativeDrag) {
        this.$el.ondragstart = () => false
      }
      [this.parentWidth, this.parentHeight] = this.getParentSize()
      this.rawRight = this.parentWidth - this.rawWidth - this.rawLeft
      this.rawBottom = this.parentHeight - this.rawHeight - this.rawTop
      addEvent(document.documentElement, 'mousedown', this.deselect)
      addEvent(document.documentElement, 'mousedown', this.dragSelectDown) // 拖拽选择元素
      addEvent(document.documentElement, 'touchend touchcancel', this.deselect)
      addEvent(window, 'resize', this.checkParentSize)
    },
    beforeDestroy: function () {
      removeEvent(document.documentElement, 'mousedown', this.deselect)
      removeEvent(document.documentElement, 'touchstart', this.handleUp)
      removeEvent(document.documentElement, 'mousemove', this.move)
      removeEvent(document.documentElement, 'touchmove', this.move)
      removeEvent(document.documentElement, 'mouseup', this.handleUp)
      removeEvent(document.documentElement, 'touchend touchcancel', this.deselect)
      removeEvent(window, 'resize', this.checkParentSize)
    },

    methods: {
      resetBoundsAndMouseState () {
        this.mouseClickPosition = { mouseX: 0, mouseY: 0, x: 0, y: 0, w: 0, h: 0, o: [0, 0], m: [0, 0], s:[0, 0], e:[0, 0] }
        this.bounds = {
          minLeft: null,
          maxLeft: null,
          minRight: null,
          maxRight: null,
          minTop: null,
          maxTop: null,
          minBottom: null,
          maxBottom: null
        }
      },
      checkParentSize () {
        if (this.parent) {
          const [newParentWidth, newParentHeight] = this.getParentSize()
          const deltaX = this.parentWidth - newParentWidth
          const deltaY = this.parentHeight - newParentHeight
          this.rawRight -= deltaX
          this.rawBottom -= deltaY
          this.parentWidth = newParentWidth
          this.parentHeight = newParentHeight
        }
      },
      getParentSize () {
        const parent = this.parent
        if (parent === true) {
          const style = window.getComputedStyle(document.querySelector(`.${this.fathterClass}`), null)
          console.log(style.getPropertyValue('height'))
          return [
            parseInt(style.getPropertyValue('width'), 10),
            parseInt(style.getPropertyValue('height'), 10)
          ]
        }
        if (typeof parent === 'string') {
          const parentNode = document.querySelector(parent)
          if (!(parentNode instanceof HTMLElement)) {
            throw new Error(`The selector ${parent} does not match any element`)
          }
          return [parentNode.offsetWidth, parentNode.offsetHeight]
        }
        return [null, null]
      },
      calcDragLimits () { // 拖拽或者移动的时候的限制
        return {
          minLeft: (this.parentWidth + this.left) % this.grid[0],
          maxLeft: Math.floor((this.parentWidth - this.width - this.left) / this.grid[0]) * this.grid[0] + this.left,
          minRight: (this.parentWidth + this.right) % this.grid[0],
          maxRight: Math.floor((this.parentWidth - this.width - this.right) / this.grid[0]) * this.grid[0] + this.right,
          minTop: (this.parentHeight + this.top) % this.grid[1],
          maxTop: Math.floor((this.parentHeight - this.height - this.top) / this.grid[1]) * this.grid[1] + this.top,
          minBottom: (this.parentHeight + this.bottom) % this.grid[1],
          maxBottom: Math.floor((this.parentHeight - this.height - this.bottom) / this.grid[1]) * this.grid[1] + this.bottom
        }
      },
      deselect (e) {
        const target = e.target || e.srcElement
        const regex = new RegExp(this.className + '-([trmbl]{2})', '')
        if (!this.$el.contains(target) && !regex.test(target.className)) {
          if (this.enabled && !this.preventDeactivation) {
            this.enabled = false
            this.$emit('deactivated', this.itemData)
            this.$emit('update:active', false, this.itemData)
          }
          removeEvent(document.documentElement, eventsFor.move, this.handleMove)
        }
        this.resetBoundsAndMouseState()
      },
      elementDown (e, target) {
        this.checkParentSize()
        var target = target || e.target || e.srcElement
        if (this.$el.contains(target)) {
          if (this.onDragStart && this.onDragStart(e) === false) {
            return
          }
          if (
            (this.dragHandle && !matchesSelectorToParentElements(target, this.dragHandle, this.$el)) ||
            (this.dragCancel && matchesSelectorToParentElements(target, this.dragCancel, this.$el))
          ) {
            return
          }
          if (!this.enabled) {
            this.enabled = true
            this.$emit('activated', this.itemData)
            this.$emit('update:active', true, this.itemData)
          }
          if (this.draggable) {
            this.dragging = true
          }
          this.mouseClickPosition.mouseX = e.touches ? e.touches[0].pageX : e.pageX
          this.mouseClickPosition.mouseY = e.touches ? e.touches[0].pageY : e.pageY
          this.mouseClickPosition.left = this.left
          this.mouseClickPosition.right = this.right
          this.mouseClickPosition.top = this.top
          this.mouseClickPosition.bottom = this.bottom
          if (this.parent) {
            this.bounds = this.calcDragLimits()
          }
          addEvent(document.documentElement, eventsFor.move, this.move)
          addEvent(document.documentElement, eventsFor.stop, this.handleUp)
        }
      },
      handleDown (handle, e) { // 手柄移动时候发生的事情
        this.checkParentSize()
        if (this.onResizeStart && this.onResizeStart(handle, e) === false) {
          return
        }
        if (e.stopPropagation) e.stopPropagation()
        if (this.lockAspectRatio && !handle.includes('m')) {
          this.handle = 'm' + handle.substring(1)
        } else {
          this.handle = handle
        }
        this.resizing = true
        this.mouseClickPosition.mouseX = e.touches ? e.touches[0].pageX : e.pageX
        this.mouseClickPosition.mouseY = e.touches ? e.touches[0].pageY : e.pageY
        this.mouseClickPosition.left = this.left
        this.mouseClickPosition.right = this.right
        this.mouseClickPosition.top = this.top
        this.mouseClickPosition.bottom = this.bottom
        this.bounds = this.calcResizeLimits()
        addEvent(document.documentElement, eventsFor.move, this.handleMove)
        addEvent(document.documentElement, eventsFor.stop, this.handleUp)
      },

      rotateDown (e) { // 旋转元素发发生的事情
        if (this.onResizeStart && this.onResizeStart(handle, e) === false) {
          return
        }
        if (e.stopPropagation) e.stopPropagation()
        this.rotateing = true
        this.mouseClickPosition.o[0] = this.left+this.width/2 // 元素中心点的x值
        this.mouseClickPosition.o[1] = this.top+this.height/2 //元素中心点的y值
        addEvent(document.documentElement, eventsFor.move, this.rotateMove)
        addEvent(document.documentElement, eventsFor.stop, this.handleUp)
      },
      dragSelectDown (e) { // 旋转元素发发生的事情
        if (this.onResizeStart && this.onResizeStart(handle, e) === false) {
          return
        }
        this.mouseClickPosition.s[0] = e.touches ? e.touches[0].pageX : e.pageX
        this.mouseClickPosition.s[1] = e.touches ? e.touches[0].pageY : e.pageY
        let s_x = this.mouseClickPosition.s[0];
        let s_y = this.mouseClickPosition.s[1];
        setTimeout(()=>{
          if( (s_x < this.left && s_y < this.top && !this.resizing && !this.rotateing && this.isCanDragSelect) || (s_x > (this.left+this.width) && (s_y > (this.top + this.height )) && !this.resizing && !this.rotateing  && this.isCanDragSelect)){
            if (e.stopPropagation) e.stopPropagation()
            this.dragSelecting = true
            addEvent(document.documentElement, eventsFor.move, this.dragSelectMove)
            addEvent(document.documentElement, eventsFor.stop, this.handleUp)
          }
        },20)
      },

      calcResizeLimits () {
        let minW = this.minW
        let minH = this.minH
        let maxW = this.maxW
        let maxH = this.maxH
        const aspectFactor = this.aspectFactor
        const [gridX, gridY] = this.grid
        const width = this.width
        const height = this.height
        const left = this.left
        const top = this.top
        const right = this.right
        const bottom = this.bottom
        if (this.lockAspectRatio) {
          if (minW / minH > aspectFactor) {
            minH = minW / aspectFactor
          } else {
            minW = aspectFactor * minH
          }
          if (maxW && maxH) {
            maxW = Math.min(maxW, aspectFactor * maxH)
            maxH = Math.min(maxH, maxW / aspectFactor)
          } else if (maxW) {
            maxH = maxW / aspectFactor
          } else if (maxH) {
            maxW = aspectFactor * maxH
          }
        }
        maxW = maxW - (maxW % gridX)
        maxH = maxH - (maxH % gridY)
        const limits = {
          minLeft: null,
          maxLeft: null,
          minTop: null,
          maxTop: null,
          minRight: null,
          maxRight: null,
          minBottom: null,
          maxBottom: null
        }
        if (this.parent) {
          limits.minLeft = (this.parentWidth + left) % gridX
          limits.maxLeft = left + Math.floor((width - minW) / gridX) * gridX
          limits.minTop = (this.parentHeight + top) % gridY
          limits.maxTop = top + Math.floor((height - minH) / gridY) * gridY
          limits.minRight = (this.parentWidth + right) % gridX
          limits.maxRight = right + Math.floor((width - minW) / gridX) * gridX
          limits.minBottom = (this.parentHeight + bottom) % gridY
          limits.maxBottom = bottom + Math.floor((height - minH) / gridY) * gridY
          if (maxW) {
            limits.minLeft = Math.max(limits.minLeft, this.parentWidth - right - maxW)
            limits.minRight = Math.max(limits.minRight, this.parentWidth - left - maxW)
          }
          if (maxH) {
            limits.minTop = Math.max(limits.minTop, this.parentHeight - bottom - maxH)
            limits.minBottom = Math.max(limits.minBottom, this.parentHeight - top - maxH)
          }
          if (this.lockAspectRatio) {
            limits.minLeft = Math.max(limits.minLeft, left - top * aspectFactor)
            limits.minTop = Math.max(limits.minTop, top - left / aspectFactor)
            limits.minRight = Math.max(limits.minRight, right - bottom * aspectFactor)
            limits.minBottom = Math.max(limits.minBottom, bottom - right / aspectFactor)
          }
        } else {
          limits.minLeft = null
          limits.maxLeft = left + Math.floor((width - minW) / gridX) * gridX
          limits.minTop = null
          limits.maxTop = top + Math.floor((height - minH) / gridY) * gridY
          limits.minRight = null
          limits.maxRight = right + Math.floor((width - minW) / gridX) * gridX
          limits.minBottom = null
          limits.maxBottom = bottom + Math.floor((height - minH) / gridY) * gridY
          if (maxW) {
            limits.minLeft = -(right + maxW)
            limits.minRight = -(left + maxW)
          }
          if (maxH) {
            limits.minTop = -(bottom + maxH)
            limits.minBottom = -(top + maxH)
          }
          if (this.lockAspectRatio && (maxW && maxH)) {
            limits.minLeft = Math.min(limits.minLeft, -(right + maxW))
            limits.minTop = Math.min(limits.minTop, -(maxH + bottom))
            limits.minRight = Math.min(limits.minRight, -left - maxW)
            limits.minBottom = Math.min(limits.minBottom, -top - maxH)
          }
        }
        return limits
      },
      move (e) {
        e.preventDefault();
        if (this.resizing) {
          this.handleMove(e)
        } else if (this.dragging) {
          this.elementMove(e)
        }else if (this.rotateing) {
          this.rotateMove(e)
        }else if (this.dragSelecting) {
          this.dragSelectMove(e)
        }
      },
      elementMove (e) { // 元素移动发生的事情
        const axis = this.axis
        const grid = this.grid
        const mouseClickPosition = this.mouseClickPosition
        const tmpDeltaX = axis && axis !== 'y' ? mouseClickPosition.mouseX - (e.touches ? e.touches[0].pageX : e.pageX) : 0
        const tmpDeltaY = axis && axis !== 'x' ? mouseClickPosition.mouseY - (e.touches ? e.touches[0].pageY : e.pageY) : 0
        const [deltaX, deltaY] = this.snapToGrid(this.grid, tmpDeltaX, tmpDeltaY)
        if (!deltaX && !deltaY) return
        this.rawTop = mouseClickPosition.top - deltaY
        this.rawBottom = mouseClickPosition.bottom + deltaY
        this.rawLeft = mouseClickPosition.left - deltaX
        this.rawRight = mouseClickPosition.right + deltaX
        setTimeout(()=>{ // 定时器解决这个问题奇葩
          this.$emit('dragging', this.left, this.top, this.itemData)
        },5)

      },
      handleMove (e) { // 拖拽8个角发生的事情
        const handle = this.handle
        const mouseClickPosition = this.mouseClickPosition
        const tmpDeltaX = mouseClickPosition.mouseX - (e.touches ? e.touches[0].pageX : e.pageX)
        const tmpDeltaY = mouseClickPosition.mouseY - (e.touches ? e.touches[0].pageY : e.pageY)
        const [deltaX, deltaY] = this.snapToGrid(this.grid, tmpDeltaX, tmpDeltaY)
        if (!deltaX && !deltaY) return
        if (handle.includes('b')) {
          this.rawBottom = mouseClickPosition.bottom + deltaY
        } else if (handle.includes('t')) {
          this.rawTop = mouseClickPosition.top - deltaY
        }
        if (handle.includes('r')) {
          this.rawRight = mouseClickPosition.right + deltaX
        } else if (handle.includes('l')) {
          this.rawLeft = mouseClickPosition.left - deltaX
        }
        this.$emit('resizing', this.left, this.top, this.width, this.height, this.itemData)
      },
      rotateMove (e) { // 拖拽8个角发生的事情
        this.mouseClickPosition.m[0] = e.touches ? e.touches[0].pageX-7 : e.pageX-7 // 移动点的x值
        this.mouseClickPosition.m[1] = e.touches ? e.touches[0].pageY-7 : e.pageY-7 // 移动点的y值
        let o = this.mouseClickPosition.o
        let m = this.mouseClickPosition.m
        var atan_o =  Math.atan((m[0]-o[0])/(o[1]-m[1]))*180/Math.PI
        if (m[1]>o[1]) {
          atan_o = atan_o + 180
        }
        if(atan_o<3 && atan_o>-3){
          this.rotate=0
        } else if(atan_o<93 && atan_o>87){
          this.rotate=90
        } else if(atan_o<183 && atan_o>177){
          this.rotate=180
        } else if(atan_o<273 && atan_o>267){
          this.rotate=270
        } else {
          this.rotate = atan_o
        }
        this.$emit('rotateing', this.rotate )
      },

      dragSelectMove (e) {
        this.mouseClickPosition.m[0] = e.touches ? e.touches[0].pageX - 10 : e.pageX - 10// 移动点的x值
        this.mouseClickPosition.m[1] = e.touches ? e.touches[0].pageY- 10 : e.pageY- 10 // 移动点的y值
        let s = this.mouseClickPosition.s
        let m = this.mouseClickPosition.m
        let dragSelectDate = this.dragSelectDate
        if(s[0]<m[0] && s[1]<m[1]){ //开始点在结束点的左上角
          dragSelectDate.left = s[0]
          dragSelectDate.top = s[1]
        } else if (s[0]>m[0] && s[1]<m[1]){
          dragSelectDate.left = m[0]
          dragSelectDate.top = s[1]
        }else if (s[0]>m[0] && s[1]>m[1]){
          dragSelectDate.left = m[0]
          dragSelectDate.top = m[1]
        }else if (s[0]<m[0] && s[1]>m[1]){
          dragSelectDate.left = s[0]
          dragSelectDate.top = m[1]
        }
        dragSelectDate.width = Math.abs(m[0]-s[0])
        dragSelectDate.height = Math.abs(m[1]-s[1])
        this.dragSelectDate = dragSelectDate
        this.$emit('dragSelecting', s, m)
      },


      handleUp (e) { // 所有的鼠标抬起都会走这里
        this.handle = null
        this.resetBoundsAndMouseState()
        this.rawTop = this.top
        this.rawBottom = this.bottom
        this.rawLeft = this.left
        this.rawRight = this.right
        if (this.resizing) {
          this.resizing = false
          this.$emit('resizestop', this.left, this.top, this.width, this.height, this.itemData)
        }
        if (this.dragging) {
          this.dragging = false
          this.$emit('dragstop', this.left, this.top, this.itemData)
        }
        if (this.rotateing) {
          this.rotateing = false
          this.$emit('rotatestop', this.rotate)
        }if (this.dragSelecting) {
          this.dragSelecting = false
          let dragSelectDate = this.dragSelectDate
          dragSelectDate.left = 0
          dragSelectDate.top = 0
          dragSelectDate.width = 0
          dragSelectDate.height = 0
          this.dragSelectDate = dragSelectDate
          this.$emit('dragselectstop')
        }
        removeEvent(document.documentElement, eventsFor.move, this.handleMove)
        removeEvent(document.documentElement, eventsFor.move, this.rotateMove)
        removeEvent(document.documentElement, eventsFor.move, this.dragSelectMove)
      },
      snapToGrid (grid, pendingX, pendingY) {
        const x = Math.round(pendingX / grid[0]) * grid[0]
        const y = Math.round(pendingY / grid[1]) * grid[1]
        return [x, y]
      }
    },
    computed: {
      style () {
        return {
          position: 'absolute',
          top: this.top + 'px',
          left: this.left + 'px',
          width: this.width + 'px',
          height: this.height + 'px',
          transform:`rotate(${this.rotate}deg)`,
          zIndex: this.zIndex,
          ...(this.dragging && this.disableUserSelect ? userSelectNone : userSelectAuto)
        }
      },
      dragSelectStyle () {
        return {
          top: this.dragSelectDate.top + 'px',
          left: this.dragSelectDate.left + 'px',
          width: this.dragSelectDate.width + 'px',
          height: this.dragSelectDate.height + 'px',
        }
      },
      actualHandles () {
        if (!this.resizable) return []
        return this.handles
      },
      width () {
        return this.parentWidth - this.left - this.right
      },
      height () {
        return this.parentHeight - this.top - this.bottom
      },
      resizingOnX () { // 是否是中间横向的拉伸或者缩放
        return (Boolean(this.handle) && (this.handle.includes('l') || this.handle.includes('r')))
      },
      resizingOnY () { // 是否是纵向的拉伸或者收缩
        return (Boolean(this.handle) && (this.handle.includes('t') || this.handle.includes('b')))
      },
      isCornerHandle () { // 是否拉的是四个拐角
        return (Boolean(this.handle) && ['tl', 'tr', 'br', 'bl'].includes(this.handle))
      }
    },
    watch: {
      active (val) {
        this.enabled = val
        if (val) {
          this.$emit('activated', this.itemData)
        } else {
          this.$emit('deactivated', this.itemData)
        }
      },
      z (val) {
        if (val >= 0 || val === 'auto') {
          this.zIndex = val
        }
      },
      rawLeft (newLeft) {
        const bounds = this.bounds
        const aspectFactor = this.aspectFactor
        const lockAspectRatio = this.lockAspectRatio
        const left = this.left
        const top = this.top
        if (bounds.minLeft !== null && newLeft < bounds.minLeft) {
          newLeft = bounds.minLeft
        } else if (bounds.maxLeft !== null && bounds.maxLeft < newLeft) {
          newLeft = bounds.maxLeft
        }
        if (lockAspectRatio && this.resizingOnX) {
          this.rawTop = top - (left - newLeft) / aspectFactor
        }

        this.left = newLeft
      },
      rawRight (newRight) {
        const bounds = this.bounds
        const aspectFactor = this.aspectFactor
        const lockAspectRatio = this.lockAspectRatio
        const right = this.right
        const bottom = this.bottom
        if (bounds.minRight !== null && newRight < bounds.minRight) {
          newRight = bounds.minRight
        } else if (bounds.maxRight !== null && bounds.maxRight < newRight) {
          newRight = bounds.maxRight
        }
        if (lockAspectRatio && this.resizingOnX) {
          this.rawBottom = bottom - (right - newRight) / aspectFactor
        }
        this.right = newRight
      },
      rawTop (newTop) {
        const bounds = this.bounds
        const aspectFactor = this.aspectFactor
        const lockAspectRatio = this.lockAspectRatio
        const left = this.left
        const top = this.top
        if (bounds.minTop !== null && newTop < bounds.minTop) {
          newTop = bounds.minTop
        } else if (bounds.maxTop !== null && bounds.maxTop < newTop) {
          newTop = bounds.maxTop
        }
        if (lockAspectRatio && this.resizingOnY) {
          this.rawLeft = left - (top - newTop) * aspectFactor
        }
        this.top = newTop
      },
      rawBottom (newBottom) {
        const bounds = this.bounds
        const aspectFactor = this.aspectFactor
        const lockAspectRatio = this.lockAspectRatio
        const right = this.right
        const bottom = this.bottom
        if (bounds.minBottom !== null && newBottom < bounds.minBottom) {
          newBottom = bounds.minBottom
        } else if (bounds.maxBottom !== null && bounds.maxBottom < newBottom) {
          newBottom = bounds.maxBottom
        }
        if (lockAspectRatio && this.resizingOnY) {
          this.rawRight = right - (bottom - newBottom) * aspectFactor
        }
        this.bottom = newBottom
      },
      x () {
        if (this.resizing || this.dragging) {
          return
        }
        if (this.parent) {
          this.bounds = this.calcDragLimits()
        }
        const delta = this.x - this.left
        if (delta % this.grid[0] === 0) {
          this.rawLeft = this.x
          this.rawRight = this.right - delta
        }
      },
      y () {
        if (this.resizing || this.dragging) {
          return
        }
        if (this.parent) {
          this.bounds = this.calcDragLimits()
        }
        const delta = this.y - this.top
        if (delta % this.grid[1] === 0) {
          this.rawTop = this.y
          this.rawBottom = this.bottom - delta
        }
      },
      r (newRotate) {
        this.rotate = newRotate
      },
      lockAspectRatio (val) {
        if (val) {
          this.aspectFactor = this.width / this.height
        } else {
          this.aspectFactor = undefined
        }
      },
      minWidth (val) {
        if (val > 0 && val <= this.width) {
          this.minW = val
        }
      },
      minHeight (val) {
        if (val > 0 && val <= this.height) {
          this.minH = val
        }
      },
      maxWidth (val) {
        this.maxW = val
      },
      maxHeight (val) {
        this.maxH = val
      },
      w () {
        if (this.resizing || this.dragging) {
          return
        }
        if (this.parent) {
          this.bounds = this.calcResizeLimits()
        }
        const delta = this.width - this.w
        if (delta % this.grid[0] === 0) {
          this.rawRight = this.right + delta
        }
      },
      h () {
        if (this.resizing || this.dragging) {
          return
        }
        if (this.parent) {
          this.bounds = this.calcResizeLimits()
        }
        const delta = this.height - this.h

        if (delta % this.grid[1] === 0) {
          this.rawBottom = this.bottom + delta
        }
      }
    }
  }
</script>
