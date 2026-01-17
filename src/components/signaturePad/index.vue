<template>
  <div class="signature-canvas-container" ref="containerRef">
    <canvas ref="canvasRef" class="signature-canvas-pad"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch, nextTick } from 'vue'
import SignaturePad from 'signature_pad'

const emits = defineEmits(['beginStroke', 'endStroke'])
const props = defineProps({
  // 笔画最小宽度
  penMinWidth: {
    type: Number,
    default: 2,
  },
  // 笔画最大宽度
  penMaxWidth: {
    type: Number,
    default: 2,
  },
  // 笔画颜色
  penColor: {
    type: String,
    default: '#000000',
  },
  // 背景色
  backgroundColor: {
    type: String,
    default: '#F3F3F4',
  },
  // 是否有背景图（用于回显）
  bgImageUrl: {
    type: String,
    default: '',
  },
})

// 背景url改变监听
watch(
  () => props.bgImageUrl,
  () => {
    initBgImage()
  },
)

// 笔画颜色改变监听
watch(
  () => props.penColor,
  () => {
    if (signaturePadRef.value) {
      signaturePadRef.value.penColor = props.penColor
    }
  },
)

// 笔画最小宽度监听
watch(
  () => props.penMinWidth,
  (val) => {
    if (signaturePadRef.value) {
      signaturePadRef.value.minWidth = val
    }
  },
)

// 笔画最大宽度监听
watch(
  () => props.penMaxWidth,
  (val) => {
    if (signaturePadRef.value) {
      signaturePadRef.value.maxWidth = val
    }
  },
)

const containerRef = ref(null) // 画板容器
const canvasRef = ref(null)
const signaturePadRef = ref(null)
const bgImage = ref(null)
const isEmpty = ref(true)
const canvasData = ref(null) // 保存画板数据

let ratio = window.devicePixelRatio || 1

// 初始化背景图
const initBgImage = () => {
  if (props.bgImageUrl) {
    const img = new Image()
    img.crossOrigin = 'anonymous'
    img.onload = () => {
      bgImage.value = img
      if (signaturePadRef.value) {
        signaturePadRef.value.clear()
        if (canvasData.value) {
          signaturePadRef.value.fromData(canvasData.value)
        }
      }
    }
    img.src = props.bgImageUrl
  } else {
    bgImage.value = null
    if (signaturePadRef.value) {
      signaturePadRef.value.clear()
      if (canvasData.value) {
        signaturePadRef.value.fromData(canvasData.value)
      }
    }
  }
}

// 初始化画板
const initSignaturePad = () => {
  if (!canvasRef.value || !containerRef.value) return

  const canvas = canvasRef.value
  const container = containerRef.value

  const width = container.clientWidth
  const height = container.clientHeight

  canvas.width = width * ratio
  canvas.height = height * ratio
  canvas.style.width = `${width}px`
  canvas.style.height = `${height}px`

  const ctx = canvas.getContext('2d')
  ctx.scale(ratio, ratio)

  const canvasPad = new SignaturePad(canvas, {
    minWidth: props.penMinWidth,
    maxWidth: props.penMaxWidth,
    penColor: props.penColor,
    backgroundColor: props.backgroundColor,
  })

  removeEvents(canvasPad)
  addEvents(canvasPad)

  const originClear = canvasPad.clear
  canvasPad.clear = (isClearBg) => {
    originClear.call(canvasPad) // 调用原始方法
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    if (bgImage.value && !isClearBg) {
      ctx.drawImage(bgImage.value, 0, 0, width, height)
    } else {
      ctx.fillStyle = props.backgroundColor
      ctx.fillRect(0, 0, width, height)
    }
  }

  signaturePadRef.value = canvasPad
  canvasPad.clear()

  // 恢复之前的画板
  if (canvasData.value) {
    canvasPad.fromData(canvasData.value)
  }
}

// 解除事件绑定
const removeEvents = (canvasPad) => {
  canvasPad.removeEventListener('beginStroke')
  canvasPad.removeEventListener('endStroke')
}

// 添加事件绑定
const addEvents = (canvasPad) => {
  canvasPad.addEventListener('beginStroke', () => {
    emits('beginStroke')
  })
  canvasPad.addEventListener('endStroke', () => {
    emits('endStroke')
  })
}

// 清空画板 是否清除背景图
const clear = (isClearBg = false) => {
  signaturePadRef.value?.clear(isClearBg)
  canvasData.value = null
  isEmpty.value = true
}

// 保存画板 格式、质量、是否以文件导出
const getBase64Data = (format = 'png', quality = 0.95) => {
  if (signaturePadRef.value && !signaturePadRef.value.isEmpty()) {
    return signaturePadRef.value.toDataURL(`image/${format}`, quality)
  } else {
    return null
  }
}

// 导出画板图片
const getImageFile = (format = 'png', quality = 0.95) => {
  const link = document.createElement('a')
  link.href = signaturePadRef.value.toDataURL(`image/${format}`, quality)
  link.download = `signature.${format}`
  link.click()
}

// 设置背景图
const setBgImage = (url) => {
  const img = new Image()
  img.crossOrigin = 'anonymous'
  img.onload = () => {
    bgImage.value = img
    if (signaturePadRef.value) {
      signaturePadRef.value.clear()
      if (canvasData.value) {
        signaturePadRef.value.fromData(canvasData.value)
      }
    }
  }
  img.src = url
}

const handleResize = () => {
  ratio = window.devicePixelRatio || 1
  // 保存当前画板
  if (signaturePadRef.value) {
    canvasData.value = signaturePadRef.value.toData()
  }
  // 重新初始化画布
  initSignaturePad()
}

onMounted(() => {
  nextTick(() => {
    initSignaturePad()
    initBgImage()
    window.addEventListener('resize', handleResize)
  })
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})

// 画板是否空白
const isCanvasEmpty = () => {
  isEmpty.value = signaturePadRef.value.isEmpty()
  return isEmpty.value
}

defineExpose({
  clear,
  getBase64Data,
  getImageFile,
  setBgImage,
  isCanvasEmpty,
  signaturePadRef,
})
</script>

<style scoped>
.signature-canvas-container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}
</style>
