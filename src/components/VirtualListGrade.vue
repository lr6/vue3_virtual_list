<script setup>
import { ref, defineProps, computed, onMounted, onUpdated } from 'vue'

let start_offset = ref(0)
let start = ref(0)
let end = ref(0)
const container_dom = ref(null)
const item_dom = ref(null)
const position = ref([])
let list = ref([])
const props = defineProps({
  list_data: {
    type: Array,
    default: []
  },
  estimated_item_size: {
    type: Number,
    default: 50
  },
  buffer_scale: {
    type: Number,
    default: 1
  }
})
list = props.list_data.map((item, index) => {
  return {
    _index: index, item
  }
})

const list_height = computed(() => {
  return position.value[position.value.length - 1].bottom
})
const visible_list = computed(() => {
  const start_index = start.value - above_count.value
  const end_index = Math.min(end.value + below_count.value, list.length)
  return list.slice(start_index, end_index)
})
const visible_count = computed(() => {
  return Math.ceil(800 / props.estimated_item_size)
})
const get_transform = computed(() => {
  return `translate3d(0,${start_offset.value}px,0)`;
})
const above_count = computed(() => {
  return Math.min(start.value, props.buffer_scale * visible_count.value)
})
const below_count = computed(() => {
  return Math.min(position.value.length - end.value, props.buffer_scale * visible_count.value)
})

onMounted(() => {
  end.value = start.value + visible_count.value
})

onUpdated(() => {
  item_dom.value.forEach((node => {
    let rect = node.getBoundingClientRect()
    let height = rect.height
    let index = +node.id
    let estimated_height = position.value[index].height
    let D_value = estimated_height - height
    if(D_value) {
      position.value[index].height = height
      position.value[index].bottom -= D_value
      for(let k = index + 1; k < position.value.length; k++) {
        position.value[k].top = position.value[k - 1].bottom
        position.value[k].bottom -= D_value
      }
    }
  }))
})

function scroll_event() {
  const scrollTop = container_dom.value.scrollTop
  start.value = binarySearch(position.value, scrollTop)
  end.value = start.value + visible_count.value
  if(start.value) {
    start_offset.value = position.value[start.value - 1].bottom
  } else {
    start_offset.value = 0
  }
}
function init_position() {
  position.value = list.map(({}, index) => {
    const { estimated_item_size } = props
    return {
      index,
      height: estimated_item_size,
      top: index * estimated_item_size,
      bottom: (index + 1) * estimated_item_size
    }
  })
}
function binarySearch(list,value){
  let start = 0;
  let end = list.length - 1;
  let tempIndex = null;
  while(start <= end){
    let midIndex = parseInt((start + end)/2);
    let midValue = list[midIndex].bottom;
    if(midValue === value){
      return midIndex + 1;
    }else if(midValue < value){
      start = midIndex + 1;
    }else if(midValue > value){
      if(tempIndex === null || tempIndex > midIndex){
        tempIndex = midIndex;
      }
      end = end - 1;
    }
  }
  return tempIndex;
}

init_position()
</script>

<template>
  <div class="container" ref="container_dom" @scroll="scroll_event">
    <div class="phantom" :style="{ height: list_height + 'px' }"></div>
    <div class="list" :style="{ transform: get_transform }">
      <div v-for="(item, index) in visible_list"
           class="list_item"
           ref="item_dom"
           :id="item._index"
           :key="index">
        {{ item._index }}: {{ item.item }}
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  height: 100%;
  overflow: auto;
  position: relative;
  -webkit-overflow-scrolling: touch;
  scrollbar-width: none;
  -ms-overflow-style: none;
}
.container::-webkit-scrollbar {
  display: none;
}
.phantom {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  z-index: -1;
}
.list {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  text-align: center;
}
.list_item {
  padding: 20px;
  color: #555;
  box-sizing: border-box;
  border-bottom: 1px solid #999;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
