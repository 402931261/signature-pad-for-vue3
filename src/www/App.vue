<template>
  <div class="pages">
    <h2>签字板 For vue3</h2>
    <div class="signature-pad-container">
      <SignaturePad
        ref="signaturePadRef"
        :bgImageUrl="canvasBgUrl"
        :penColor="color"
        :watermark="watermarkOptions"
        @beginStroke="handleBeginStroke"
        @endStroke="handleEndStroke"
      />
    </div>
    <div class="signature-pad-buttons">
      <button @click="handleResetSignatureClick">重置（清空笔画）</button>
      <button @click="handleResetBgColorClick">重置（并清空背景颜色）</button>
      <button @click="handleResetBgAndSignatureClick">重置（并清空背景）</button>
      <button @click="handleResetWatermarkAndSignatureClick">重置（并清空水印）</button>
      <button @click="handleResetAllClick">重置（并清空背景和水印）</button>
      <button @click="handleGetBase64Click">获取签字base64</button>
      <button @click="handleGetImageFileClick">获取签字文件下载</button>
      <button @click="checkSignatureIsEmpty">检查是画板是否有内容</button>
      <div class="pen-color">
        <span>画笔颜色：</span>
        <input type="color" v-model="color" />
      </div>
    </div>
  </div>
</template>
<script setup>
import SignaturePad from '@/components/signaturePad/index.vue'
import { ref, onMounted } from 'vue'

// 以下是签字板相关
const signaturePadRef = ref(null)
const canvasBgUrl = ref('')
const color = ref('#223388')
const watermarkOptions = ref({
  text: '这是一段水印文字this is the watermark text',
  color: 'rgba(255, 0, 0, 0.3)',
})

setTimeout(() => {
  // canvasBgUrl.value = ''
  signaturePadRef.value.setBgImage(
    'https://img0.baidu.com/it/u=4032653794,3881997789&fm=253&fmt=auto&app=138&f=JPEG?w=715&h=500',
  )
}, 1500)

// 画笔开始绘制
const handleBeginStroke = () => {
  console.log('handleBeginStroke')
}
// 画笔结束绘制
const handleEndStroke = () => {
  console.log('handleEndStroke')
}

// 只清空签名
const handleResetSignatureClick = () => {
  signaturePadRef.value.clear()
}

// 清空签名和背景颜色
const handleResetBgColorClick = () => {
  signaturePadRef.value.clear(['background-color'])
}

// 清空签名和背景
const handleResetBgAndSignatureClick = () => {
  signaturePadRef.value.clear(['background'])
}

// 清空签名和水印
const handleResetWatermarkAndSignatureClick = () => {
  signaturePadRef.value.clear(['watermark'])
}

// 清空签名、背景和水印
const handleResetAllClick = () => {
  signaturePadRef.value.clear(['background', 'watermark'])
}

// 获取签字base64
const handleGetBase64Click = () => {
  console.log(signaturePadRef.value.isCanvasEmpty())
  console.log('base64 data: ', signaturePadRef.value.getBase64Data())
}

// 获取签字文件下载
const handleGetImageFileClick = () => {
  signaturePadRef.value.getImageFile()
}

// 检查签字板是否是空的
const checkSignatureIsEmpty = () => {
  console.log(signaturePadRef.value.isCanvasEmpty())
}

onMounted(() => {
  console.log('signaturePadRef', signaturePadRef.value)
})
</script>

<style scoped>
.pages {
  width: 80%;
  height: 100%;
  margin: 0 auto;
  padding: 40px 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  color: #333;
}

h2 {
  text-align: center;
  font-size: 24px;
  font-weight: 600;
  margin-bottom: 30px;
  color: #2c3e50;
}

.signature-pad-container {
  width: 100%;
  height: 400px;
  border: 2px solid #e1e8ed;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
  background-color: #fff;
  margin-bottom: 30px;
}

.signature-pad-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  justify-content: center;
  align-items: center;
}

button {
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  background-color: #3498db;
  color: white;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 2px 8px rgba(52, 152, 219, 0.3);
}

button:hover {
  background-color: #2980b9;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(52, 152, 219, 0.4);
}

button:active {
  transform: translateY(0);
}

.pen-color {
  display: flex;
  align-items: center;
  gap: 8px;
  border: 1px solid #e1e8ed;
  border-radius: 6px;
  padding: 8px 12px;
}

input[type='color'] {
  width: 50px;
  height: 24px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  background-color: transparent;
  padding: 2px;
}

input[type='color']::-webkit-color-swatch-wrapper {
  padding: 0;
}

input[type='color']::-webkit-color-swatch {
  border: none;
  border-radius: 4px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}
</style>
