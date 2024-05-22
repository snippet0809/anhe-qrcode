<template>
  <el-form>
    <el-form-item>

    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="generateImage" :loading=status_render>预览内容</el-button>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="saveImage">保存图片</el-button>
    </el-form-item>
  </el-form>

  <!-- 遍历生成 -->
  <div class="result" ref="result">
    <!--换行-->
    <div v-for="row in Math.ceil(count / rowLen)" style="display: flex">
      <!--每行20个-->
      <CodeItem v-for="data in dataList.slice((row - 1) * rowLen, Math.min(row * rowLen, dataList.length))"
                :content="data"/>
    </div>
  </div>
</template>
<script setup lang="ts">
import {type Ref, ref, nextTick} from 'vue'
import html2canvas from 'html2canvas'
import CodeItem, {Content} from '@/components/CodeItem.vue'

const qrcode_prefix = ref('')

const count = ref(20)
const rowLen = ref(4)

const dataList: Ref<Array<Content>> = ref([])

const result = ref()

const status_render = ref(false)

async function generateImage() {
  status_render.value = true
  await new Promise(resolve => setTimeout(resolve, 100))
  dataList.value = []
  let timestamp = Date.now()
  for (let i = 0; i < count.value; i++) {
    let t = Date.now()
    while (t === timestamp) {
      t = Date.now()
    }
    const qrcode = qrcode_prefix.value + t
    const barcode = t.toString()
    dataList.value.push(new Content('安和', qrcode, "123456781234"))
    timestamp = t
  }
  await nextTick()
  status_render.value = false
}


function saveImage() {
  html2canvas(result.value, {scale: 0.25}).then((canvas) => {
    const imgData = canvas.toDataURL('image/png', 1.0)

    const a = document.createElement("a");
    a.href = imgData;
    a.download = `安和-${getDateTimeString()}.png`;
    a.click();
  })
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
.result {
  width: calc(34mm * 4 * v-bind(rowLen));
  padding: 10mm 10mm;
}
</style>
