<script setup>
import { ref, defineProps, computed, onMounted } from 'vue'

let start_offset = ref(0)
let start = ref(0)
let end = ref(null)
const container_dom = ref(null)

const list_height = computed(() => {
  const { list_data, item_size } = props
  return list_data.length * item_size
})
const visible_list = computed(() => {
  console.log(props.list_data.slice(start.value, Math.min(end.value, props.list_data.length)))
  return props.list_data.slice(start.value, Math.min(end.value, props.list_data.length))
})
const visible_count = computed(() => {
  return Math.ceil(800 / props.item_size)
})
const get_transform = computed(() => {
  return `translate3d(0,${start_offset.value}px,0)`;
})

const props = defineProps({
  list_data: {
    type: Array,
    default: []
  },
  item_size: {
    type: Number,
    default: 200
  }
})

onMounted(() => {
  end.value = start.value + visible_count.value
})

function scroll_event() {
  const scrollTop = container_dom.value.scrollTop
  start.value = Math.floor(scrollTop / props.item_size)
  end.value = start.value + visible_count.value
  start_offset.value = scrollTop - (scrollTop % props.item_size)
}

</script>

<template>
  <div class="container" ref="container_dom" @scroll="scroll_event">
    <div class="phantom" :style="{ height: list_height + 'px' }"></div>
    <div class="list" :style="{ transform: get_transform }">
      <div v-for="(item, index) in visible_list"
           :key="index"
           class="list_item"
           :style="{ height: props.item_size + 'px' }">
        {{ item }}
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
  color: #555;
  box-sizing: border-box;
  border-bottom: 1px solid #999;
  display: flex;
  align-items: center;
  justify-content: center;
}

</style>
