# @npm_lx/signature-pad-for-vue3

一个基于[signature_pad](https://github.com/szimek/signature_pad)的签名板Vue3组件，支持自定义样式和多种导出格式，简单易用，可用于用户签名、合同签名等场景，可随时更换背景图片。  


## 安装

```bash
npm install @npm_lx/signature-pad-for-vue3
```

## 组件预览
需要你到本项目github上面clone项目，然后在项目根目录下执行以下命令，才能预览组件运行效果
```bash
pnpm install
pnpm run dev
```

## 基本使用

```vue
<template>
  <div class="signature-container">
    <SignaturePad
      ref="signaturePadRef"
      :penMinWidth="2"
      :penMaxWidth="5"
      :penColor="'#000000'"
      :backgroundColor="'#F3F3F4'"
      @beginStroke="handleBeginStroke"
      @endStroke="handleEndStroke"
    />
    <div class="signature-actions">
      <button @click="clearSignature">清除</button>
      <button @click="saveSignature">保存</button>
      <button @click="exportImage">导出图片</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import SignaturePad from '@npm_lx/signature-pad-for-vue3'

const signaturePadRef = ref(null)

const handleBeginStroke = () => {
  console.log('开始签名')
}

const handleEndStroke = () => {
  console.log('结束签名')
}

const clearSignature = () => {
  signaturePadRef.value.clear()
}

const saveSignature = () => {
  const base64Data = signaturePadRef.value.getBase64Data()
  if (base64Data) {
    console.log('签名数据:', base64Data)
    // 可以将base64Data发送到服务器保存
  }
}

const exportImage = () => {
  signaturePadRef.value.getImageFile()
}
</script>

<style scoped>
.signature-container {
  width: 400px;
  height: 300px;
  border: 1px solid #ccc;
  margin: 0 auto;
}

.signature-actions {
  margin-top: 20px;
  text-align: center;
}

button {
  margin: 0 10px;
  padding: 8px 16px;
  cursor: pointer;
}
</style>
```

## 配置项

| 参数名 | 类型 | 默认值 | 描述 |
| --- | --- | --- | --- |
| penMinWidth | Number | 2 | 笔画的最小宽度 |
| penMaxWidth | Number | 2 | 笔画的最大宽度 |
| penColor | String | '#000000' | 笔画的颜色 |
| backgroundColor | String | '#F3F3F4' | 画板的背景颜色 |
| bgImageUrl | String | '' | 背景图片的URL（用于回显签名） |

## 方法

| 方法名 | 参数 | 返回值 | 描述 |
| --- | --- | --- | --- |
| clear | isClearBg: Boolean (默认: false) | 无 | 清空画板，isClearBg为true时同时清除背景图片 |
| getBase64Data | format: String (默认: 'png'), quality: Number (默认: 0.95) | String | 获取签名的Base64数据 |
| getImageFile | format: String (默认: 'png'), quality: Number (默认: 0.95) | 无 | 导出签名为图片文件 |
| setBgImage | url: String | 无 | 设置背景图片 |
| isCanvasEmpty | 无 | Boolean | 检查画板是否为空 |

## 事件

| 事件名 | 说明 |
| --- | --- |
| beginStroke | 开始签名时触发 |
| endStroke | 结束签名时触发 |

## 注意事项

1. 组件需要一个有固定宽高的容器来显示
2. 使用ref来调用组件的方法
3. 使用背景图片需要来源支持跨域设置
4. 导出的图片格式支持png、jpg等常见格式
