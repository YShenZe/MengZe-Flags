<script setup>
import { ref } from 'vue'

defineProps({
  msg: String,
})

const count = ref(0)
</script>

<template>
  <h4>{{ msg }}</h4>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
