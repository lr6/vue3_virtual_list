<script setup>
import { ref } from 'vue'
import { faker } from '@faker-js/faker'
import VirtualListGrade from './components/VirtualListGrade.vue'
const list = ref([])
const count = 10_000
for(let i = 0; i < count; i++) {
  list.value.push(faker.lorem.sentences())
}
</script>

<template>
  <VirtualListGrade :list_data="list" />
</template>

<style scoped>
</style>
