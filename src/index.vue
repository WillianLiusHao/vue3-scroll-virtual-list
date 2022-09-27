<script lang="ts">
import { PropType, onMounted, ref, nextTick } from "vue";
export default {
  name: 'VirtualList'
}
</script>
<script lang="ts" setup>
const props = defineProps({
  data: {
    type: Array as PropType<any>,
    required: true,
  },
  // 列表项初始条数，用于预先计算可视区域的显示项数
  initSize: {
    type: Number,
    default: 8,
  },
  // 列表每一项的key
  key: {
    type: String,
  },
  // 在可见区域的上/下方 分别预渲染条数，避免快速滑动出现闪烁
  bufferScale: {
    type: Number,
    default: 1
  }
})


/* 计算方法 */
// 可视区域条数 = 可视区域高度 / 每条高度 
// 可视区域开始索引 = Math.ceil(scrollTop / 每条高度) 向下取整，保证最前面的那条及时部分展示也可展示

const virtualDom = ref<Element>() // 可是区域容器
const viewData = ref<any>([]) // 渲染数据
const startIndex = ref<number>(0) // 渲染数据开始索引
const viewNums = ref<number>(0) // 可视区域数据条数
const itemH = ref<number>(0) 
const contentH = ref<number>(0)


const scrollView = () => {
  const scrollTop = virtualDom.value?.scrollTop as number
  startIndex.value = Math.floor(scrollTop / itemH.value) 
  viewData.value = props.data.slice(
    startIndex.value >= props.bufferScale ? startIndex.value - props.bufferScale : startIndex.value,
    startIndex.value + 1 + viewNums.value + props.bufferScale
  )
}
onMounted(() => {
  const viewH = virtualDom.value?.clientHeight as number
  viewData.value = props.data.slice(0, props.initSize + props.bufferScale)
  nextTick(() => {
    itemH.value = document.querySelector('.virtual-list-item')?.clientHeight as number
    contentH.value = props.data.length * itemH.value
    viewNums.value = Math.ceil(viewH / itemH.value)
  })
})

const itemStyle = (index: number) => {
  return `
    top: ${startIndex.value >= props.bufferScale ? (startIndex.value - props.bufferScale + index) * itemH.value : (startIndex.value + index) * itemH.value}px;
    left: 0px;
  `
}
</script>

<template>
  <div class="virtual-list-view" ref="virtualDom" @scroll="scrollView">
    <div class="virtual-list-content" :style='`height:${contentH}px;width: 200px;`'>
      <div
        class="virtual-list-item"
        v-for="(item, index) in viewData"
        :key="key ? item[key] : index"
        :style="itemStyle(index)"
      >
        <slot name="default" :item="item"></slot>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.virtual-list {
  &-view {
    position: relative;
    width: 100%;
    overflow-x:hidden;
    overflow-y: auto;

    display: flex;
    justify-content: center;
    align-items: center;
    border: 1px solid #ccc;
  }

  &-content {
    position: absolute;
    top: 0;
  }
  &-item {
    width: 100%;
    height: 120px;
    display: flex;
    justify-content: center;
    align-items: center;
    position: absolute;
  }
}
::-webkit-scrollbar {
  width: 5px;
  height: 5px;
  background-color: #F5F5F5;
}

::-webkit-scrollbar-thumb{
  border-radius: 10px;
  -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,.3);
  background-color: #bdbdbd;
}

::-webkit-scrollbar-thumb:hover{
  border-radius: 5px;
  -webkit-box-shadow: inset 0 0 5px rgba(0,0,0,0.2);
  background: rgba(0,0,0,0.2);
}
</style>
