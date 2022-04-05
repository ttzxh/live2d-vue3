<template>
  <div
    id="waifu"
    style="position: absolute"
    :style="[
      { left: positionLeft },
      { right: positionRight },
      { top: positionTop },
      { botton: positionBottom },
    ]"
  >
    <div>
      <div id="waifu-tips" :style="[{ width: TipWidth }, { height: TipHeight }]"></div>
      <canvas id="live2d" width="800" height="800" :style="[{ width: ModelWidth }, { height: ModelHeight }]"></canvas>
      <div
        id="waifu-tool"
        style="position: absolute; width: 20px"
        :style="[
          { left: toolPositionLeft },
          { right: toolPositionRight },
          { top: toolPositionTop },
          { botton: toolPositionBottom },
        ]"
      >
        <live2d-item :className="className" @itemClick="itemClick" @itemEnter="itemEnter"></live2d-item>
      </div>
    </div>
  </div>
</template>

<script setup>
import live2dItem from './live2d-item.vue'
import live2dmin from './live2d.min.js'
import { onMounted, ref, onUnmounted, defineProps, defineEmits, nextTick, defineExpose } from 'vue'
const props = defineProps({
  className: {
    type: Array,
    default: () => []
  },
  ModelWidth: {
    type: String,
    default: ''
  },
  ModelHeight: {
    type: String,
    default: ''
  },
  TipWidth: {
    type: String,
    default: ''
  },
  TipHeight: {
    type: String,
    default: ''
  },
  positionLeft: {
    type: String,
    default: ''
  },
  positionRight: {
    type: String,
    default: ''
  },
  positionBottom: {
    type: String,
    default: ''
  },
  positionTop: {
    type: String,
    default: ''
  },
  toolPositionLeft: {
    type: String,
    default: '0px'
  },
  toolPositionRight: {
    type: String,
    default: ''
  },
  toolPositionBottom: {
    type: String,
    default: ''
  },
  toolPositionTop: {
    type: String,
    default: ''
  }
})
const modelList = ref('')
const text = ref('')
const now = ref(0)
const messageTimer = ref(null)
const userActionTimer = ref(null)
const waifuPath = ref('')
const modelId = ref(1)
const modelTexturesId = ref(0)
const cdnPath = ref('')
const $emit = defineEmits(['live2dItemEnter', 'live2dItemClick'])
onMounted(() => {
  getHours()
  nextTick(() => {
    loadlimit(),
      window.addEventListener('copy', () => {
        showMessage('你都复制了些什么呀，转载要记得加上出处哦！', 6000, 9)
      })
    window.addEventListener('visibilitychange', () => {
      if (!document.hidden) showMessage('哇，你终于回来了～', 6000, 9)
    })
  })
})
onUnmounted(() => {
  window.removeEventListener('copy', () => {})
  window.removeEventListener('visibilitychange', () => {})
})
const itemEnter = index => {
  $emit('live2dItemEnter', index)
}
const itemClick = index => {
  $emit('live2dItemClick', index)
}
const devtool = () => {
  const devtools = () => {}
  console.log('%c', devtools)
  devtools.toString = () => {
    showMessage('哈哈，你打开了控制台，是想要看看我的小秘密吗？', 6000, 9)
  }
}
// const view = () => {
//   loadRandModel()
// }
// const circle = () => {
//   loadOtherModel()
// }
const loadWidget = config => {
  cdnPath.value = config.cdnPath
  sessionStorage.removeItem('waifu-text')
  let userAction = false,
    messageArray = ['好久不见，日子过得好快呢……', '大坏蛋！你都多久没理人家了呀，嘤嘤嘤～', '嗨～快来逗我玩吧！', '拿小拳拳锤你胸口！']
  window.addEventListener('mousemove', () => (userAction = true))
  window.addEventListener('keydown', () => (userAction = true))
  setInterval(() => {
    if (userAction) {
      userAction = false
      clearInterval(userActionTimer.value)
      userActionTimer.value = null
    } else if (!userActionTimer.value) {
      userActionTimer.value = setInterval(() => {
        showMessage(randomSelection(messageArray), 6000, 9)
      }, 20000)
    }
  }, 1000)

  welcomeMessage()
  showHitokoto()
  loadModelList()
  // initModel()
  loadModel(modelId.value, modelTexturesId, '你好呀')
}
const loadlimit = () => {
  if (screen.width >= 768) {
    loadWidget({
      cdnPath: 'https://cdn.jsdelivr.net/gh/fghrsh/live2d_api/'
    })
  }
}
const getHours = () => {
  setInterval(() => {
    now.value = new Date().getHours()
  }, 1000)
}
const randomSelection = obj => {
  return Array.isArray(obj) ? obj[Math.floor(Math.random() * obj.length)] : obj
}
const welcomeMessage = () => {
  const now = new Date().getHours()
  if (now > 5 && now <= 7) text.value = '早上好！一日之计在于晨，美好的一天就要开始了。'
  else if (now > 7 && now <= 11) text.value = '上午好！工作顺利嘛，不要久坐，多起来走动走动哦！'
  else if (now > 11 && now <= 13) text.value = '中午了，工作了一个上午，现在是午餐时间！'
  else if (now > 13 && now <= 17) text.value = '午后很容易犯困呢，今天的运动目标完成了吗？'
  else if (now > 17 && now <= 19) text.value = '傍晚了！窗外夕阳的景色很美丽呢，最美不过夕阳红～'
  else if (now > 19 && now <= 21) text.value = '晚上好，今天过得怎么样？'
  else if (now > 21 && now <= 23) text.value = ['已经这么晚了呀，早点休息吧，晚安～', '深夜时要爱护眼睛呀！']
  else text.value = '你是夜猫子呀？这么晚还不睡觉，明天起的来嘛？'
  showMessage(text.value, 7000, 8)
}
const showHitokoto = () => {
  fetch('https://v1.hitokoto.cn')
    .then(response => response.json())
    .then(result => {
      const text = `这句一言来自 <span>「${result.from}」</span>，是 <span>${result.creator}</span> 在 hitokoto.cn 投稿的。`
      showMessage(result.hitokoto, 6000, 9)
      setTimeout(() => {
        showMessage(text, 4000, 9)
      }, 6000)
    })
}
const showMessage = (text, timeout, priority) => {
  if (!text || (sessionStorage.getItem('waifu-text') && sessionStorage.getItem('waifu-text') > priority)) return
  if (messageTimer.value) {
    clearTimeout(messageTimer.value)
    messageTimer.value = null
  }
  text = randomSelection(text)
  sessionStorage.setItem('waifu-text', priority)
  const tips = document.getElementById('waifu-tips')
  tips.innerHTML = text
  tips.classList.add('waifu-tips-active')
  messageTimer.value = setTimeout(() => {
    sessionStorage.removeItem('waifu-text')
    tips.classList.remove('waifu-tips-active')
  }, timeout)
}

const loadModelList = async () => {
  const response = await fetch(`${cdnPath.value}model_list.json`)
  modelList.value = await response.json()
}
const loadModel = async (modelId, modelTexturesId, message) => {
  localStorage.setItem('modelId', modelId)
  localStorage.setItem('modelTexturesId', modelTexturesId)
  showMessage(message, 4000, 10)
  if (!modelList.value) await loadModelList()
  const target = randomSelection(modelList.value.models[modelId])
  loadlive2d('live2d', `${cdnPath.value}model/${target}/index.json`)
}
const loadRandModel = async () => {
  const modelId = localStorage.getItem('modelId'),
    modelTexturesId = localStorage.getItem('modelTexturesId')
  if (!modelList.value) await loadModelList()
  const target = randomSelection(modelList.value.models[modelId])
  loadlive2d('live2d', `${cdnPath.value}model/${target}/index.json`)
  showMessage('我的新衣服好看嘛？', 4000, 10)
}
const loadOtherModel = async () => {
  let modelId = localStorage.getItem('modelId')
  if (!modelList.value) await loadModelList()
  const index = ++modelId >= modelList.value.models.length ? 0 : modelId
  loadModel(index, 0, modelList.value.messages[index])
}
defineExpose({ loadOtherModel, loadRandModel, showMessage })
</script>

<style scoped="scoped">
#waifu-toggle {
  background-color: #fa0;
  border-radius: 5px;
  bottom: 66px;
  color: #fff;
  cursor: pointer;
  font-size: 12px;
  left: 0;
  margin-left: -100px;
  padding: 5px 2px 5px 5px;
  position: fixed;
  transition: margin-left 1s;
  width: 60px;
  writing-mode: vertical-rl;
}
html,
body {
  height: 100%;
  width: 100%;
}
#waifu-toggle.waifu-toggle-active {
  margin-left: -50px;
}

#waifu-toggle.waifu-toggle-active:hover {
  margin-left: -30px;
}

#waifu {
  right: 0;
  bottom: 0;
  line-height: 0;
  position: fixed;
  z-index: 1;
}

#waifu-tips {
  animation: shake 50s ease-in-out 5s infinite;
  background-color: rgba(236, 217, 188, 0.5);
  border: 1px solid rgba(224, 186, 140, 0.62);
  border-radius: 12px;
  box-shadow: 0 3px 15px 2px rgba(191, 158, 118, 0.2);
  font-size: 14px;
  line-height: 24px;
  margin: -30px 20px;
  min-height: 70px;
  opacity: 0;
  overflow: hidden;
  padding: 5px 10px;
  position: absolute;
  text-overflow: ellipsis;
  transition: opacity 1s;
  width: 250px;
  word-break: break-all;
}

#waifu-tips.waifu-tips-active {
  opacity: 1;
  transition: opacity 0.2s;
}
#waifu #live2d {
  cursor: grab;
  height: 300px;
  position: relative;
  width: 300px;
}

#waifu #live2d:active {
  cursor: grabbing;
}

#waifu-tool {
  opacity: 0;
  position: absolute;
  right: 290px;
  top: 70px;
  transition: opacity 1s;
}

#waifu:hover #waifu-tool {
  opacity: 1;
}
@keyframes shake {
  2% {
    transform: translate(0.5px, -1.5px) rotate(-0.5deg);
  }

  4% {
    transform: translate(0.5px, 1.5px) rotate(1.5deg);
  }

  6% {
    transform: translate(1.5px, 1.5px) rotate(1.5deg);
  }

  8% {
    transform: translate(2.5px, 1.5px) rotate(0.5deg);
  }

  10% {
    transform: translate(0.5px, 2.5px) rotate(0.5deg);
  }

  12% {
    transform: translate(1.5px, 1.5px) rotate(0.5deg);
  }

  14% {
    transform: translate(0.5px, 0.5px) rotate(0.5deg);
  }

  16% {
    transform: translate(-1.5px, -0.5px) rotate(1.5deg);
  }

  18% {
    transform: translate(0.5px, 0.5px) rotate(1.5deg);
  }

  20% {
    transform: translate(2.5px, 2.5px) rotate(1.5deg);
  }

  22% {
    transform: translate(0.5px, -1.5px) rotate(1.5deg);
  }

  24% {
    transform: translate(-1.5px, 1.5px) rotate(-0.5deg);
  }

  26% {
    transform: translate(1.5px, 0.5px) rotate(1.5deg);
  }

  28% {
    transform: translate(-0.5px, -0.5px) rotate(-0.5deg);
  }

  30% {
    transform: translate(1.5px, -0.5px) rotate(-0.5deg);
  }

  32% {
    transform: translate(2.5px, -1.5px) rotate(1.5deg);
  }

  34% {
    transform: translate(2.5px, 2.5px) rotate(-0.5deg);
  }

  36% {
    transform: translate(0.5px, -1.5px) rotate(0.5deg);
  }

  38% {
    transform: translate(2.5px, -0.5px) rotate(-0.5deg);
  }

  40% {
    transform: translate(-0.5px, 2.5px) rotate(0.5deg);
  }

  42% {
    transform: translate(-1.5px, 2.5px) rotate(0.5deg);
  }

  44% {
    transform: translate(-1.5px, 1.5px) rotate(0.5deg);
  }

  46% {
    transform: translate(1.5px, -0.5px) rotate(-0.5deg);
  }

  48% {
    transform: translate(2.5px, -0.5px) rotate(0.5deg);
  }

  50% {
    transform: translate(-1.5px, 1.5px) rotate(0.5deg);
  }

  52% {
    transform: translate(-0.5px, 1.5px) rotate(0.5deg);
  }

  54% {
    transform: translate(-1.5px, 1.5px) rotate(0.5deg);
  }

  56% {
    transform: translate(0.5px, 2.5px) rotate(1.5deg);
  }

  58% {
    transform: translate(2.5px, 2.5px) rotate(0.5deg);
  }

  60% {
    transform: translate(2.5px, -1.5px) rotate(1.5deg);
  }

  62% {
    transform: translate(-1.5px, 0.5px) rotate(1.5deg);
  }

  64% {
    transform: translate(-1.5px, 1.5px) rotate(1.5deg);
  }

  66% {
    transform: translate(0.5px, 2.5px) rotate(1.5deg);
  }

  68% {
    transform: translate(2.5px, -1.5px) rotate(1.5deg);
  }

  70% {
    transform: translate(2.5px, 2.5px) rotate(0.5deg);
  }

  72% {
    transform: translate(-0.5px, -1.5px) rotate(1.5deg);
  }

  74% {
    transform: translate(-1.5px, 2.5px) rotate(1.5deg);
  }

  76% {
    transform: translate(-1.5px, 2.5px) rotate(1.5deg);
  }

  78% {
    transform: translate(-1.5px, 2.5px) rotate(0.5deg);
  }

  80% {
    transform: translate(-1.5px, 0.5px) rotate(-0.5deg);
  }

  82% {
    transform: translate(-1.5px, 0.5px) rotate(-0.5deg);
  }

  84% {
    transform: translate(-0.5px, 0.5px) rotate(1.5deg);
  }

  86% {
    transform: translate(2.5px, 1.5px) rotate(0.5deg);
  }

  88% {
    transform: translate(-1.5px, 0.5px) rotate(1.5deg);
  }

  90% {
    transform: translate(-1.5px, -0.5px) rotate(-0.5deg);
  }

  92% {
    transform: translate(-1.5px, -1.5px) rotate(1.5deg);
  }

  94% {
    transform: translate(0.5px, 0.5px) rotate(-0.5deg);
  }

  96% {
    transform: translate(2.5px, -0.5px) rotate(-0.5deg);
  }

  98% {
    transform: translate(-1.5px, -1.5px) rotate(-0.5deg);
  }

  0%,
  100% {
    transform: translate(0, 0) rotate(0);
  }
}
</style>
