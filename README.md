# @npm_lx/signature-pad-for-vue3

A Vue3 signature pad component based on [signature_pad](https://github.com/szimek/signature_pad), supporting custom styles and multiple export formats, easy to use, suitable for user signatures, contract signatures, etc. Background images can be changed at any time.

## Other Languages Document
- [中文文档](https://github.com/402931261/signature-pad-for-vue3/blob/master/README.zhCN.md)

## Installation

```bash
npm install @npm_lx/signature-pad-for-vue3
```

## Component Preview
To preview the component, you need to clone the project from GitHub and run the following commands in the project root directory:
```bash
pnpm install
pnpm run dev
```

## Basic Usage

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
      <button @click="clearSignature">Clear</button>
      <button @click="saveSignature">Save</button>
      <button @click="exportImage">Export Image</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import SignaturePad from '@npm_lx/signature-pad-for-vue3'

const signaturePadRef = ref(null)

const handleBeginStroke = () => {
  console.log('Begin signing')
}

const handleEndStroke = () => {
  console.log('End signing')
}

const clearSignature = () => {
  signaturePadRef.value.clear()
}

const saveSignature = () => {
  const base64Data = signaturePadRef.value.getBase64Data()
  if (base64Data) {
    console.log('Signature data:', base64Data)
    // You can send base64Data to the server for storage
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

## Props

| Prop Name | Type | Default Value | Description |
| --- | --- | --- | --- |
| penMinWidth | Number | 2 | Minimum width of the pen stroke |
| penMaxWidth | Number | 2 | Maximum width of the pen stroke |
| penColor | String | '#000000' | Color of the pen stroke |
| backgroundColor | String | '#F3F3F4' | Background color of the signature pad |
| bgImageUrl | String | '' | URL of the background image (for signature preview) |
| watermark | Object | {} | Watermark configuration parameters, default values see [Watermark Configuration](#default-watermark-parameters) |

## Default Watermark Parameters
When you want to add a watermark, remember to pass a string for the text parameter to enable the watermark, otherwise no watermark will be added if the text value is empty.
```javascript
{
  // Watermark text
  text: '', 
  // Font size
  fontSize: 20,
  // Line height
  lineHeight: 24,
  // Font family
  fontFamily: 'Arial',
  // Font weight
  fontWeight: 'bold',
  // Font color
  color: 'rgba(0, 0, 0, 0.1)',
  // Rotation angle
  rotate: -45,
  // Watermark starting x coordinate
  x: 100,
  // Watermark starting y coordinate
  y: 100,
  // Distance between watermarks when repeating x y
  textDistanceX: 0,
  textDistanceY: 0,
  // Whether to repeat the watermark across the entire canvas
  repeat: true,
}
```
## Methods

| Method Name | Parameters | Return Value | Description |
| --- | --- | --- | --- |
| clear | isClearBg: Boolean (default: false) | None | Clear the signature pad, if isClearBg is true, also clear the background image |
| getBase64Data | format: String (default: 'png'), quality: Number (default: 0.95) | String | Get the signature as Base64 data |
| getImageFile | format: String (default: 'png'), quality: Number (default: 0.95) | None | Export the signature as an image file |
| setBgImage | url: String | None | Set the background image |
| isCanvasEmpty | None | Boolean | Check if the canvas is empty |

## Events

| Event Name | Description |
| --- | --- |
| beginStroke | Triggered when starting to sign |
| endStroke | Triggered when finishing signing |

## Notes

1. The component needs a container with fixed width and height to display
2. Use ref to call the component's methods
3. Using background images requires cross-origin support from the source
4. Exported image formats support common formats like png, jpg, etc.
