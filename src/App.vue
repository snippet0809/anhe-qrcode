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
    <el-form-item label="横向间距">
      <el-input-number v-model="itemWidth" :step="0.1"></el-input-number>
    </el-form-item>
    <el-form-item label="纵向间距">
      <el-input-number v-model="itemHeight" :step="0.1"></el-input-number>
    </el-form-item>
    <el-form-item label="扩放比例">
      <el-space>
        <el-input-number v-model="multiple" :min="1" :max="20" :step="1"></el-input-number>
        <el-text type="success">值越大，图片越清晰（原理：显示器DPI固定的情况下，1mm对应的像素数量也是固定的。
          物理距离小的图片，对应的像素数量也少，图片会模糊，所以要做放大处理）
        </el-text>
      </el-space>
    </el-form-item>
    <el-form-item label="缩放比例">
      <el-space>
        <el-input-number v-model="scale" :min="0.1" :max="1" :step="0.1"></el-input-number>
        <el-text type="success">值越大，图片越清晰（原理：可以简单地理解为图片的压缩比例，等于1时表示不压缩直接输出原图）
        </el-text>
      </el-space>
    </el-form-item>

    <el-form-item label="条形码竖条间距">
      <el-space>
        <el-input-number v-model="barcodeWidth" :step="0.01"></el-input-number>
        <el-text type="success">注意：若改变扩放或缩放参数，条形码竖条间距或需手动调整
        </el-text>
      </el-space>
    </el-form-item>
    <el-form-item label="条形码高度">
      <el-space>
        <el-input-number v-model="barcodeHeight" :step="0.1"></el-input-number>
        <el-text type="success">注意：暂未做自适应处理，若改变扩放或缩放参数，条形码高度或需手动调整
        </el-text>
      </el-space>
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
  <el-text type="success">v1.1.0</el-text>

  <!-- 遍历生成 -->
  <div class="result" ref="result">
    <!--换行-->
    <div v-for="row in Math.ceil(formData.count / rowLen)" style="display: flex">
      <!--每行20个-->
      <CodeItem v-for="data in dataList.slice((row - 1) * rowLen,
          Math.min(row * rowLen, dataList.length))" :content="data" :itemStyle="style"/>
    </div>
  </div>
</template>
<script setup lang="ts">
import {type Ref, ref, nextTick, reactive, onMounted} from 'vue'
import html2canvas from 'html2canvas'
import CodeItem, {Content} from '@/components/CodeItem.vue'
import {ElMessage, type FormInstance, type FormRules} from 'element-plus'
import jsPDF from 'jspdf'

class FormData {
  static qrcode_protocol = 'http://'
  static barcode_prefix = '007'

  qrcode_prefix: string = 'trace.lyanhe.com/t/'
  barcode_suffix: string = '000000000'
  count: number = 100
}

const formData = ref(new FormData())
const ruleFormRef = ref<FormInstance>()
const rules = reactive<FormRules<FormData>>({
  qrcode_prefix: [{required: true, message: '请输入二维码前缀', trigger: 'blur'}],
  barcode_suffix: [{required: true, message: '请输入起始条形码的后9位', pattern: /^\d{9}$/, trigger: 'blur'}]
})

const rowLen = ref(20)
const multiple = ref(10)
const scale = ref(0.5)
const itemWidth = ref(34)
const itemHeight = ref(48.5)
const barcodeWidth = ref(0.17)
const barcodeHeight = ref(3.5)

const dataList: Ref<Array<Content>> = ref([])
const result = ref()

const status_render = ref(false)
const status_download = ref(false)

const style = {
  itemWidth: itemWidth.value, itemHeight: itemHeight.value, scale: scale.value, multiple: multiple.value,
  barcodeWidth: barcodeWidth.value, barcodeHeight: barcodeHeight.value
}

onMounted(() => {
  const resultWidth = style.itemWidth * style.multiple * rowLen.value
  result.value.style.width = `${resultWidth}mm`
})

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
  html2canvas(result.value, {scale: style.scale}).then((canvas) => {
    const quotient = Math.floor(formData.value.count / rowLen.value)
    const remainder = formData.value.count % rowLen.value
    const colLen = remainder == 0 ? quotient : quotient + 1
    const width = style.itemWidth * rowLen.value, height = style.itemHeight * colLen
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
      pdf.addImage(imgData, 'png', 0, 0, pxToMm(canvas.width / (style.multiple * style.scale)),
          pxToMm(canvas.height / (style.multiple * style.scale)))
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
.el-input {
  width: 500px;
}
</style>
