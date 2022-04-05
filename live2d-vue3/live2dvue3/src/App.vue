<template>
  <div id="app">
    <!-- <ttzxh-live2d
      @live2dItemClick="live2dItemClick"
      @live2dItemEnter="live2dItemEnter"
      ref="live2d"
      :className="className"
      TipWidth="150px"
      TipHeight="50px"
      ModelHeight="200px"
      ModelWidth="200px"
    ></ttzxh-live2d>-->
    <live2d-v3
      @live2dItemClick="live2dItemClick"
      @live2dItemEnter="live2dItemEnter"
      ref="live2d"
      :className="className"
      TipWidth="150px"
      TipHeight="50px"
      ModelHeight="200px"
      ModelWidth="200px"
    ></live2d-v3>
  </div>
</template>

<script setup>
// import ttzxhLive2d from './package/index.js'
import { ref } from 'vue'
const className = ['fa fa-lg fa-times', 'fa fa-lg fa-info-circle']
const live2d = ref(null)
//index为当前选中的icon的索引
const live2dItemClick = index => {
  //第一个icon被点击时触发
  if (index === 0) {
    live2d.value.loadOtherModel()
  }
  if (index === 1) {
    live2d.value.loadRandModel()
  }
}
const live2dItemEnter = index => {
  live2d.value.showMessage('我是live2d', 2000, 1000)
}
</script>

<style scoped>
.fa {
  color: blue;
}
</style>
