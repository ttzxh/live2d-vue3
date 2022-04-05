<template>
  <div>
    <slot>
      <span v-for="(item, index) in className" :key="index" :class="[item]" @mouseenter="testEnter(index)" @click="testClick(index)"></span>
    </slot>
  </div>
</template>

<script setup>
import { defineProps, ref, defineEmits } from 'vue'
const currentIndex = ref(0)
const $emit = defineEmits(['itemClick', 'itemEnter'])
const props = defineProps(['className'])
const testClick = index => {
  $emit('itemClick', index, 1000)
}
const testEnter = index => {
  $emit('itemEnter', index)
}
</script>

<style scoped="scoped">
span {
  cursor: pointer;
  display: block;
  line-height: 30px;
  text-align: center;
  transition: color 0.3s;
}
span:hover {
  color: #0684bd; /* #34495e */
}
</style>
