<template>
  <div class="Number">
    <div class="inner" :class="generateClasses" @mousedown="onDragStart">
      <div class="number">
        {{ data.number }}
      </div>
    </div>
  </div>
</template>

<script>
import Draggable from 'draggable'
export default {
  name: 'Number',
  props: {
    data: Object
  },
  computed: {
    generateClasses () {
      const { data: { number, slide, dragging } } = this
      return Object.entries({
        invisible: number === null,
        grabby: slide,
        grabbed: dragging
      }).reduce((accumulator, [className, condition]) => {
        if (condition) {
          accumulator.push(className)
        }
        return accumulator
      }, []).join(' ')
    }
  },
  watch: {
    'data.slide' (side) {
      const target = this.$el.firstChild
      const { drag } = this
      if (drag) {
        drag.destroy()
        this.drag = null
        target.style = ''
      }
      if (!side) { return }
      const maxDistance = 1e2
      const startX = target.offsetLeft
      const startY = target.offsetTop
      console.log(startX, startY)
      const options = {
        limit: (x, y) => {
          const diff = {
            bottom: y - startY,
            top: startY - y,
            left: startX - x,
            right: x - startX
          }[side]
          if (diff > maxDistance) {
            this.drag.destroy()
            setTimeout(() => {
              this.$emit('slide', this.data)
            }, 1)
            return { x, y }
          } else {
            switch (side) {
              case 'bottom':
              case 'top':
                return { x: startX, y: diff > 0 ? y : startY }
              case 'left':
              case 'right':
                return { x: diff > 0 ? x : startX, y: startY }
            }
          }
        }
      }
      this.drag = new Draggable(target, options)
    }
  },
  methods: {
    onDragStart () {
      this.data.dragging = true
      document.body.addEventListener('mouseup', this.onDragEnd)
    },
    onDragEnd () {
      this.data.dragging = false
      document.body.removeEventListener('mouseup', this.inDragProcess)
    }
  }
}
</script>

<style lang="scss" scoped>
.Number{
    padding: 10px;
    height: 25%;
    width: 25%;
    float: left;
}
.inner{
    height: 100%;
    width: 100%;
    display: flex !important;
    position: relative;
    &:not(.invisible){
        border: 6px groove $main2;
        z-index:2
    }
    &.grabby{
        cursor: grab;
        background: rgb(203, 255, 203);
        &:hover{
            box-shadow: 0 0 10px 2px black;
        }
        &.grabbed{
            z-index: 9;
            cursor: grabbing;
            background: orange;
        }
    }
}
.number{
    display: inline-flex;
    margin: auto;
    font-weight: bold;
    color: $main2;
    font-size: 80px;
    user-select: none;
}
</style>
