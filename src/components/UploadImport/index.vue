<template>
  <el-dialog v-model="showValue" :title="title" width="600px" destroy-on-close>
    <div class="mt-1 w100 upload-box">
      <el-upload
        drag
        :show-file-list="false"
        :auto-upload="false"
        :limit="1"
        accept=".xls,.xlsx"
        action="#"
        :before-upload="beforeUpload"
        :on-change="customRequest"
      >
        <el-icon class="el-icon--upload"><Plus /></el-icon>
        <div class="el-upload__text">点击或拖拽文件到此处上传</div>
        <div class="el-upload__text">支持'xls','xlsx'</div>
        <template #tip>
          <div class="el-upload__tip">单次导入≤2万条数据，完成后请注意右上角消息提示。</div>
        </template>
      </el-upload>
    </div>
  </el-dialog>
</template>

<script setup lang="ts">
import { computed } from "vue"
import * as XLSX from "xlsx"
import { Plus } from "@element-plus/icons-vue"
import { ElMessage } from "element-plus"
import type { UploadProps } from "element-plus"
import { v4 as uuidv4 } from "uuid"

import { cloneDeep } from "lodash-es"
const emit = defineEmits(["update:show", "success"])

const props = withDefaults(
  defineProps<{
    title?: string
    show: boolean
    url?: string
    callbackSuccess?: (key?: string) => void
  }>(),
  {
    title: "",
    show: false,
    url: ""
  }
)
const showValue = computed({
  get() {
    return props.show
  },
  set(value: boolean) {
    emit("update:show", value)
  }
})
const beforeUpload: UploadProps["beforeUpload"] = (rawFile) => {
  console.log(rawFile, "rawFile")
  if (!["xls", "xlsx"].includes(rawFile.type)) {
    ElMessage.error("文件格式不支持")
    return false
  } else if (rawFile.size > 200 * 1024 * 1024) {
    ElMessage.error("文件超出大小限制!")
    return false
  }
  return true
}
const mapping = {
  name: "名称",
  content: "内容",
  num: "数量"
}
const formatData = (list: any) => {
  let arr = []
  const _list = cloneDeep(list || [])
  arr = _list.map((item: any) => ({
    name: item[mapping.name],
    content: item[mapping.content],
    num: item[mapping.num]
  }))
  return arr
}
const handelData = (list: any) => {
  let arr = <any>[]
  const _list = cloneDeep(list || [])
  _list.forEach((item: any) => {
    const _arr: any = []
    if (item.num > 0) {
      for (let i = 0; i < item.num; i++) {
        _arr.push({
          id: uuidv4(),
          name: item.name,
          content: item.content
        })
      }
    }
    arr = arr.concat(_arr)
  })
  return arr
}
const customRequest: UploadProps["onChange"] = (uploadFile) => {
  const reader = new FileReader()
  reader.onload = (e: any) => {
    const data = new Uint8Array(e.target.result)
    const workbook = XLSX.read(data, { type: "array" })
    const firstSheetName = workbook.SheetNames[0]
    const worksheet = workbook.Sheets[firstSheetName]
    const jsonData = XLSX.utils.sheet_to_json(worksheet)
    const list = formatData(jsonData)
    const result = handelData(list)
    console.log(result, "result")
    const names = uploadFile.name.split(".")
    names.pop()
    console.log(names, "names")
    emit("success", {
      list: result,
      fileName: names.join(".")
    })
  }
  if (uploadFile.raw) {
    reader.readAsArrayBuffer(uploadFile.raw)
  }
}
</script>

<style lang="scss" scoped></style>
