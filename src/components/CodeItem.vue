<template>
  <div class="layout" ref="pdfItemElement">
    <!-- 文字和二维码-->
    <div class="text-qrcode">
      <!-- 文字 -->
      <div class="text">
        <div><span>{{ props.content.text.charAt(0) }}</span></div>
        <div><span>{{ props.content.text.charAt(1) }}</span></div>
      </div>
      <!-- 二维码 -->
      <div class="qrcode">
        <canvas ref="qrcodeCanvas"></canvas>
      </div>
    </div>

    <!-- 条形码 -->
    <div class="barcode">
      <canvas ref="barcodeCanvas"></canvas>
    </div>
  </div>
</template>
<script lang="ts">

export class Content {
  text: string
  qrcode: string
  barcode: string

  constructor(text: string, qrcode: string, barcode: string) {
    this.text = text
    this.qrcode = qrcode
    this.barcode = barcode
  }
}
</script>
<script setup lang="ts">
import {onMounted, ref} from 'vue'
import QRCode from 'qrcode'
import JsBarcode from 'jsbarcode'

const props = defineProps<{
  content: Content
  itemStyle: {
    itemWidth: number,
    itemHeight: number,
    scale: number,
    multiple: number,
    barcodeWidth: number,
    barcodeHeight: number
  }
}>()

const pdfItemElement = ref()
const qrcodeCanvas = ref()
const barcodeCanvas = ref()

// 像素翻10倍
const space_width = props.itemStyle.itemWidth * props.itemStyle.multiple
const space_height = props.itemStyle.itemHeight * props.itemStyle.multiple

const layout_width = 20 * props.itemStyle.multiple
const layout_height = 20 * props.itemStyle.multiple
const layout_padding = 1.3 * props.itemStyle.multiple

const qrcode_width = 13 * props.itemStyle.multiple
const qrcode_margin_bottom = 0.3 * props.itemStyle.multiple

const content_width = layout_width - layout_padding * 2
const content_height = layout_height - layout_padding * 2

const text_font_size = 3.8 * props.itemStyle.multiple

onMounted(() => {
  console.log("1mm对应的px值", mmToPx(1))
  QRCode.toCanvas(qrcodeCanvas.value, props.content.qrcode, {
    width: mmToPx(qrcode_width),
    margin: 0,
    errorCorrectionLevel: 'L'
  })
  JsBarcode(barcodeCanvas.value, props.content.barcode, {
    width: mmToPx(props.itemStyle.barcodeWidth * props.itemStyle.multiple),
    height: mmToPx(props.itemStyle.barcodeHeight * props.itemStyle.multiple),
    margin: 0,
    displayValue: false
  })

  document.documentElement.style.setProperty("--space-width", `${space_width}mm`)
  document.documentElement.style.setProperty("--space-height", `${space_height}mm`)

  document.documentElement.style.setProperty("--layout-width", `${layout_width}mm`)
  document.documentElement.style.setProperty("--layout-height", `${layout_height}mm`)
  document.documentElement.style.setProperty("--layout-padding", `${layout_padding}mm`)

  document.documentElement.style.setProperty("--qrcode-width", `${qrcode_width}mm`)
  document.documentElement.style.setProperty("--qrcode-margin-bottom", `${qrcode_margin_bottom}mm`)

  document.documentElement.style.setProperty("--content-width", `${content_width}mm`)
  document.documentElement.style.setProperty("--content-height", `${content_height}mm`)

  document.documentElement.style.setProperty("--text-font-size", `${text_font_size}mm`)
})

function mmToPx(mm: number): number {
  const dom = document.createElement('div')
  dom.style.width = '1in'
  document.body.appendChild(dom)
  const dpi = dom.offsetWidth
  document.body.removeChild(dom)
  return (mm * dpi) / 25.4
}
</script>

<style scoped>
:root {
  --space-width: ;
  --space-height: ;

  --layout-width: ;
  --layout-height: ;
  --layout-padding: ;

  --qrcode-width: ;
  --qrcode-margin-bottom: ;

  --content-width: ;
  --content-height: ;

  --text-font-size: ;
}

/* 不要用layout-width，css的尺寸都是真实尺寸，不包括padding和margin */
.layout {
  width: var(--content-width);
  height: var(--content-height);
  padding: var(--layout-padding);

  margin-right: calc(var(--space-width) - var(--layout-width));
  margin-bottom: calc(var(--space-height) - var(--layout-height));
}

.text-qrcode {
  display: flex;
  height: var(--qrcode-width);
  margin-bottom: var(--qrcode-margin-bottom);
}

.text {
  width: var(--content-width);
  display: block;
}

.text div {
  display: flex;
  height: 50%;
  font-size: var(--text-font-size);
  font-weight: bold;
  justify-content: center;
  align-items: center;
}

.qrcode {
  width: var(--qrcode-width);
  height: 100%;
  display: grid;
  place-items: center;
}

.barcode {
  align-items: center;
  height: calc(var(--content-height) - var(--qrcode-width) - var(--qrcode-margin-bottom));
  justify-content: center;
  display: flex;
}

.barcode canvas {
  position: absolute;
}

.text span {
  margin-right: 5mm;
}
</style>

