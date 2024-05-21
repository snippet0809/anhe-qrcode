<template>
    <el-form>
        <el-form-item>
            <el-button type="primary" @click="generateImage">生成图片</el-button>
        </el-form-item>
    </el-form>

    <!-- 遍历生成 -->
    <div id="result" ref="result">
        <div v-for="row in Math.ceil(count/20)">
            <CodeItem v-for="data in dataList.slice(row*20,Math.min((row+1)*20,dataList.length))" :content="data" />
        </div>
    </div>
</template>
<script setup lang="ts">
import { type Ref, ref, onMounted, nextTick, watch } from 'vue'
import html2canvas from 'html2canvas'
import CodeItem, { Content } from '@/components/CodeItem.vue'

const qrcode_prefix = ref('')

const count = ref(50)
//const rowCount = ref(0)

const dataList: Ref<Array<Content>> = ref([])

const result = ref()

onMounted(() => {
})


async function generateImage() {
    dataList.value = []
    let timestamp = Date.now()
    for (let i = 0; i < count.value; i++) {
        let t = Date.now()
        while (t === timestamp) {
            t = Date.now()
        }
        const qrcode = qrcode_prefix.value + t
        const barcode = t.toString()
        dataList.value.push(new Content('安和', qrcode, barcode))
        timestamp = t
    }
//    await nextTick()
//    html2canvas(result.value, { scale: 1 }).then((canvas) => {
//        const imgData = canvas.toDataURL('image/png')
//    })
}

function pxToMm(px: number): number {
    const dom = document.createElement('div')
    dom.style.height = '1in'
    document.body.appendChild(dom)
    const dpi = dom.offsetHeight
    document.body.removeChild(dom)
    return (px * 25.4) / dpi
}

function getDateTimeString(): string {
    const now = new Date()
    const year = now.getFullYear()
    const month = (now.getMonth() + 1).toString().padStart(2, '0')
    const day = now.getDate().toString().padStart(2, '0')
    const hours = now.getHours().toString().padStart(2, '0')
    const minutes = now.getMinutes().toString().padStart(2, '0')
    const seconds = now.getSeconds().toString().padStart(2, '0')
    return `${year}${month}${day}${hours}${minutes}${seconds}`
}
</script>
<style scoped>
</style>
