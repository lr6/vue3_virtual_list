<script setup>
import { ref } from 'vue'
import VirtualList from './components/VirtualList.vue'
const list = ref([])
const count = 100
for(let i = 0; i < count; i++) {
  list.value.push(i)
}
</script>

<template>
  <VirtualList :list_data="list" :item_size="50" />
</template>

<style scoped>
</style>
