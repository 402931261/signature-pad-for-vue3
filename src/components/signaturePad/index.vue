<template>
  <div class="signature-canvas-container" ref="containerRef">
    <canvas ref="canvasRef" class="signature-canvas-pad"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch, nextTick } from 'vue'
import SignaturePad from 'signature_pad'
import { loadImage, debounce } from './utils'

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
  watermark: {
    type: Object,
    default: () => ({
      text: '',
      fontSize: 20,
      lineHeight: 24,
      fontFamily: 'Arial',
      fontWeight: 'bold',
      color: 'rgba(0, 0, 0, 0.1)',
      rotate: -45,
      x: 100,
      y: 100,
      repeat: true,
    }),
  },
})

const containerRef = ref(null) // 画板容器
const canvasRef = ref(null) // canvas 元素
const signaturePadRef = ref(null) // 实例
const bgImage = ref(null) // 背景图片
const isEmpty = ref(true) // 是否为空
const canvasData = ref(null) // 保存画板数据

let ratio = window.devicePixelRatio || 1

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

// 初始化背景图
const initBgImage = () => {
  if (props.bgImageUrl) {
    loadImage(props.bgImageUrl).then((img) => {
      bgImage.value = img
      if (signaturePadRef.value) {
        signaturePadRef.value.clear()
        if (canvasData.value) {
          signaturePadRef.value.fromData(canvasData.value)
        }
      }
    })
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

  // 如果signature_pad实例不存在，就创建一个新实例
  if (!signaturePadRef.value) {
    const canvasPad = new SignaturePad(canvas, {
      minWidth: props.penMinWidth,
      maxWidth: props.penMaxWidth,
      penColor: props.penColor,
      backgroundColor: props.backgroundColor,
    })

    const originClear = canvasPad.clear
    // 重写clear方法，调用必定会清除笔画，可选额外清除内容参数clearList：background-color、background 和 watermark
    canvasPad.clear = (clearList = []) => {
      originClear.call(canvasPad) // 调用原始方法

      const isNeedToClearBgColor = clearList.includes('background-color')
      const isNeedToClearBgImage = clearList.includes('background')
      const isNeedToClearWatermark = clearList.includes('watermark')

      ctx.clearRect(0, 0, canvas.width, canvas.height)

      // 清空背景颜色
      if (!isNeedToClearBgColor && props.backgroundColor) {
        ctx.fillStyle = props.backgroundColor
        ctx.fillRect(0, 0, canvas.width, canvas.height)
      }

      // 清空背景图片
      if (!isNeedToClearBgImage && bgImage.value) {
        ctx.drawImage(bgImage.value, 0, 0, width, height)
      }

      // 清空水印
      if (!isNeedToClearWatermark) {
        addWatermark()
      }
    }

    signaturePadRef.value = canvasPad
    addEvents()
    canvasPad.clear()

    // 恢复之前的画板
    if (canvasData.value) {
      canvasPad.fromData(canvasData.value)
    }
  } else {
    signaturePadRef.value.clear()
    if (canvasData.value) {
      signaturePadRef.value.fromData(canvasData.value)
    }
  }
}

// 解除事件绑定
const removeEvents = () => {
  signaturePadRef.value.removeEventListener('beginStroke', emitBeginStroke)
  signaturePadRef.value.removeEventListener('endStroke', emitEndStroke)
}

// 添加事件绑定
const addEvents = () => {
  signaturePadRef.value.addEventListener('beginStroke', emitBeginStroke)
  signaturePadRef.value.addEventListener('endStroke', emitEndStroke)
}

const emitBeginStroke = () => {
  emits('beginStroke')
}

const emitEndStroke = () => {
  emits('endStroke')
}

// 清空画板 是否清除背景图
const clear = (clearList) => {
  signaturePadRef.value?.clear(clearList)
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
  loadImage(url).then((img) => {
    bgImage.value = img
    if (signaturePadRef.value) {
      signaturePadRef.value.clear()
      if (canvasData.value) {
        signaturePadRef.value.fromData(canvasData.value)
      }
    }
  })
}

const handleResize = debounce(() => {
  ratio = window.devicePixelRatio || 1
  // 保存当前画板
  if (signaturePadRef.value) {
    canvasData.value = signaturePadRef.value.toData()
  }
  // 重新初始化画布
  initSignaturePad()
}, 100)

// 加水印功能
const addWatermark = () => {
  const options = Object.assign(
    {
      text: '',
      fontSize: 20,
      lineHeight: 24,
      fontFamily: 'Arial',
      fontWeight: 'bold',
      color: 'rgba(0, 0, 0, 0.1)',
      rotate: -45,
      x: 100,
      y: 100,
      repeat: true,
    },
    props.watermark,
  )

  if (!options.text) {
    return
  }

  const ctx = canvasRef.value.getContext('2d')
  if (options.repeat) {
    const canvasWidth = canvasRef.value.width / ratio
    const canvasHeight = canvasRef.value.height / ratio
    const textWidth = ctx.measureText(options.text).width
    const textHeight = options.fontSize

    const diagonal = Math.sqrt(canvasWidth * canvasWidth + canvasHeight * canvasHeight)
    const stepX = textWidth + 100
    const stepY = textHeight + 100

    for (let x = -diagonal; x < diagonal; x += stepX) {
      for (let y = -diagonal; y < diagonal; y += stepY) {
        ctx.save()
        ctx.font = `${options.fontWeight} ${options.fontSize}px/${options.lineHeight}px ${options.fontFamily}`
        ctx.fillStyle = options.color
        ctx.translate(x, y)
        ctx.rotate((Math.PI / 180) * options.rotate)
        ctx.fillText(options.text, 0, 0)
        ctx.restore()
      }
    }
  } else {
    ctx.save()
    ctx.font = `${options.fontWeight} ${options.fontSize}px/${lineHeight}px ${options.fontFamily}`
    // 水印颜色
    ctx.fillStyle = options.color
    // 水印位置
    ctx.translate(options.x, options.y)
    // 水印旋转
    ctx.rotate((Math.PI / 180) * options.rotate)
    // 设置水印文本
    ctx.fillText(options.text, 0, 0)
    ctx.restore()
  }
}

onMounted(() => {
  nextTick(() => {
    initSignaturePad()
    initBgImage()
    addWatermark()
    window.addEventListener('resize', handleResize)
  })
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  removeEvents()
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
