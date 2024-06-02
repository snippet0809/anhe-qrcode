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
}>()


const pdfItemElement = ref()
const qrcodeCanvas = ref()
const barcodeCanvas = ref()

const multiple = 10
const style = ref({
  qrcodeWidth: 13 * multiple
})

onMounted(() => {
  console.log(mmToPx(1))
  QRCode.toCanvas(qrcodeCanvas.value, props.content.qrcode, {
    width: mmToPx(style.value.qrcodeWidth),
    margin: 0,
    errorCorrectionLevel: 'L'
  })
  JsBarcode(barcodeCanvas.value, props.content.barcode, {
    width: mmToPx(0.17 * multiple),
    height: mmToPx(3.5 * multiple),
    margin: 0,
    displayValue: false
  })
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

<style lang="scss" scoped>
$multiple: 10;
$space-width: 33.9mm * $multiple;
$space-height: 48.4mm * $multiple;

$layout-width: 20mm * $multiple;
$layout-height: 20mm * $multiple;
$layout-padding-lr: 1.3mm * $multiple;
$layout-padding-ub: 1.3mm * $multiple;

$qrcode-width: 13mm * $multiple;
$qrcode-margin-bottom: 0.3mm * $multiple;

$content-width: $layout-width - $layout-padding-lr*2;
$content-height: $layout-height - $layout-padding-ub*2;

.layout {
  width: $content-width;
  height: $content-height;
  padding: $layout-padding-ub $layout-padding-lr;

  margin-right: $space-width - $layout-width;
  margin-bottom: $space-height - $layout-height;
}

.text-qrcode {
  display: flex;
  height: $qrcode-width;
  margin-bottom: $qrcode-margin-bottom;
}

.text {
  width: $content-width;
  display: block;
}

.text div {
  display: flex;
  height: 50%;
  font-weight: bold;
  font-size: 3.8mm * $multiple;
  justify-content: center;
  align-items: center;
}

.qrcode {
  width: $qrcode-width;
  height: 100%;
  display: grid;
  place-items: center;
}

.barcode {
  align-items: center;
  height: $content-height - $qrcode-width - $qrcode-margin-bottom;
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
