<template>
  <el-form label-width="auto" :rules="rules" :model="formData" ref="ruleFormRef">
    <el-form-item label="二维码前缀" prop="qrcode_prefix">
      <el-input v-model="formData.qrcode_prefix">
        <template #prepend>{{ FormData.qrcode_protocol }}</template>
      </el-input>
    </el-form-item>
    <el-form-item label="条形码起始值" prop="barcode_suffix">
      <el-input v-model="formData.barcode_suffix">
        <template #prepend>{{ FormData.barcode_prefix }}</template>
      </el-input>
    </el-form-item>
    <el-form-item label="总个数">
      <el-input-number v-model="formData.count" :min="1" :max="1200" :step="100"></el-input-number>
    </el-form-item>
    <el-form-item label="每行个数">
      <el-input-number v-model="rowLen"></el-input-number>
    </el-form-item>
    <el-text type="warning">1、当生成较多数量的条码时，所需要的时间会比较长，请耐心等待</el-text>
    <br/>
    <el-text type="warning">2、等待过程中，若浏览器提示【页面无响应】，请选择【等待】，千万不要点【退出网页】</el-text>
    <el-form-item>
      <el-button type="primary" @click="generateImage()" :loading=status_render>预览内容</el-button>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="saveImage" :loading="status_download" v-show="dataList.length>0">保存PDF
      </el-button>
    </el-form-item>
  </el-form>

  <!-- 遍历生成 -->
  <div class="result" ref="result">
    <!--换行-->
    <div v-for="row in Math.ceil(formData.count / rowLen)" style="display: flex">
      <!--每行20个-->
      <CodeItem v-for="data in dataList.slice((row - 1) * rowLen,
          Math.min(row * rowLen, dataList.length))" :content="data"/>
    </div>
  </div>
</template>
<script setup lang="ts">
import {type Ref, ref, nextTick, reactive} from 'vue'
import html2canvas from 'html2canvas'
import CodeItem, {Content} from '@/components/CodeItem.vue'
import {ElMessage, type FormInstance, type FormRules} from 'element-plus'
import jsPDF from 'jspdf'

class FormData {
  static qrcode_protocol = 'http://'
  static barcode_prefix = '007'

  qrcode_prefix: string = 'trace.lyanhe.com/t/'
  barcode_suffix: string = ''
  count: number = 100
}

const formData = ref(new FormData())
const rowLen = ref(20)
const scale = ref(0.2)
const itemWidth = 33.9, itemHeight = 48.4
const ruleFormRef = ref<FormInstance>()
const rules = reactive<FormRules<FormData>>({
  qrcode_prefix: [{required: true, message: '请输入二维码前缀', trigger: 'blur'}],
  barcode_suffix: [{required: true, message: '请输入起始条形码的后9位', pattern: /^\d{9}$/, trigger: 'blur'}]
})


const dataList: Ref<Array<Content>> = ref([])
const result = ref()

const status_render = ref(false)
const status_download = ref(false)

async function generateImage() {
  let success = false
  await ruleFormRef.value!.validate((valid) => {
    if (valid) success = true
  })
  if (!success) {
    ElMessage.error('表单验证未通过')
    return
  }
  dataList.value = []
  status_render.value = true
  await new Promise(resolve => setTimeout(resolve, 100))
  dataList.value = []
  for (let i = 0; i < formData.value.count; i++) {
    let index = parseInt(formData.value.barcode_suffix) + i
    const indexStr = String(index).padStart(9, '0')
    const barcode = FormData.barcode_prefix + indexStr
    const qrcode = FormData.qrcode_protocol + formData.value.qrcode_prefix + barcode
    dataList.value.push(new Content('安和', qrcode, barcode))
  }
  await nextTick()
  status_render.value = false
}

async function saveImage() {
  status_download.value = true
  await new Promise(resolve => setTimeout(resolve, 100))
  html2canvas(result.value, {scale: scale.value}).then((canvas) => {
    const quotient = Math.floor(formData.value.count / rowLen.value)
    const remainder = formData.value.count % rowLen.value
    const colLen = remainder == 0 ? quotient : quotient + 1
    const width = itemWidth * rowLen.value + 20, height = itemHeight * colLen + 20
    let orientation: 'p' | 'l' = 'p', format = [width, height]
    if (rowLen.value > colLen) {
      orientation = 'l'
      format = [height, width]
    }
    const pdf = new jsPDF(orientation, 'mm', format)
    try {
      const imgData = canvas.toDataURL('image/png', 1.0)
      // const a = document.createElement("a");
      // a.href = imgData;
      // a.download = `安和-${getDateTimeString()}.png`;
      // a.click();
      pdf.addImage(imgData, 'png', 0, 0, itemWidth * rowLen.value + (20 * scale.value),
          itemHeight * colLen + (20 * scale.value))
      pdf.save(`安和-${getDateTimeString()}.pdf`)
    } catch {
      ElMessage.error('生成PDF失败')
    }
    status_download.value = false
  })
}

function pxToMm(px: number) {
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
.result {
  width: calc(33.9mm * 10 * v-bind(rowLen));
  padding: 10mm 10mm;
}

.el-input {
  width: 500px;
}
</style>
