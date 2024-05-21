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
import { onMounted, ref } from 'vue'
import QRCode from 'qrcode'
import JsBarcode from 'jsbarcode'

const props = defineProps<{
    content: Content
}>()


const pdfItemElement = ref()
const qrcodeCanvas = ref()
const barcodeCanvas = ref()

const style = ref({
    spaceWidth: 34,
    spaceHeight: 48.5,

    layoutWidth: 20,
    layoutHeight: 20,
    layoutPaddingLr: 3,
    layoutPaddingUb: 3,

    qrcodeMarignButtom: 1,
    qrcodeWidth: 10
})

const style_qrocde_width = ref(10)

onMounted(() => {
    QRCode.toCanvas(qrcodeCanvas.value, props.content.qrcode, {
        width: mmToPx(style.value.qrcodeWidth),
        margin: 0,
        scale: 1
    })
    JsBarcode(barcodeCanvas.value, props.content.barcode, {
        format: 'CODE128',
        width: mmToPx(0.10),
        height: mmToPx(3),
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

<style>
:root {
    --space-width: 34mm;
    --space-height: 48.5mm;

    --layout-width: 20mm;
    --layout-height: 20mm;
    --layout-padding-lr: 3mm;
    --layout-padding-ub: 3mm;

    --qrcode-width: v-bind(style_qrocde_width+ 'mm');
    --qrcode-marign-buttom: 1mm;

    --content-width: calc(var(--layout-width) - calc(2 * var(--layout-padding-lr)));
    --content-height: calc(var(--layout-height) - calc(2 * var(--layout-padding-ub)));
}
</style>
<style scoped>
.layout {
    width: var(--content-width);
    height: var(--content-height);
    padding: var(--layout-padding-ub) var(--layout-padding-lr);
    margin-right: calc(var(--space-width) - var(--layout-width));
    margin-bottom: calc(var(--space-height) - var(--layout-height));
}

.text-qrcode {
    display: flex;
    height: var(--qrcode-width);
    margin-bottom: var(--qrcode-marign-buttom);
}

.text {
    width: calc(var(--content-width));
    display: block;
}

.text div {
    height: 50%;
    font-size: 2mm;
    text-align: center;
}

.qrcode {
    width: var(--qrcode-width);
    height: 100%;
    display: grid;
    place-items: center;
}

.barcode {
    align-items: center;
    height: calc(var(--content-height) - v-bind(style_qrocde_width+ 'mm') - var(--qrcode-marign-buttom));
    justify-content: center;
    display: flex;
}

.barcode canvas {
    position: absolute;
}
</style>

