<template>
  <div class="app-container">
    <el-card v-loading="loading" shadow="never">
      <div class="toolbar-wrapper">
        <div>
          <el-button type="primary" :icon="Upload" @click="showUpload = true">导入</el-button>
          <el-button type="primary" :icon="Download" @click="exportToExcelSelected">导出选中数据</el-button>
          <el-button type="primary" :icon="Download" @click="exportToExcel">导出全部数据</el-button>
        </div>
      </div>
      <div class="table-wrapper">
        <el-table
          ref="multipleTableRef"
          :row-key="
            (row) => {
              return row.id
            }
          "
          :data="tableData"
          @selection-change="handleSelectionChange"
        >
          <el-table-column type="selection" :reserve-selection="true" width="50" align="center" />
          <el-table-column prop="code" label="FNSKU" align="center" />
          <el-table-column prop="title" label="简化标题" align="center" />
        </el-table>
      </div>
      <div class="pager-wrapper">
        <el-pagination
          background
          :layout="paginationData.layout"
          :page-sizes="paginationData.pageSizes"
          :total="paginationData.total"
          :page-size="paginationData.pageSize"
          :currentPage="paginationData.currentPage"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        />
      </div>
    </el-card>
    <UploadImport title="数据导入" v-model:show="showUpload" @success="successBack" />
  </div>
</template>
<script lang="ts" setup>
import { ref } from "vue"

import * as XLSX from "xlsx"
import { ElMessage } from "element-plus"
import { Upload, Download } from "@element-plus/icons-vue"
import { usePagination } from "@/hooks/usePagination"
import UploadImport from "@/components/UploadImport/index.vue"

const loading = ref<boolean>(false)
const { paginationData } = usePagination()

const fileName = ref<string>("")
const showUpload = ref<boolean>(false)
const tableData = ref<any[]>([])
const allList = ref<any[]>([])

const multipleTableRef = ref()
const multipleSelection = ref<any[]>([])
const handleSelectionChange = (val: any[]) => {
  multipleSelection.value = val
}
const successBack = async (obj: any) => {
  multipleTableRef.value?.clearSelection()
  multipleSelection.value = []
  allList.value = obj.list
  fileName.value = obj.fileName
  showUpload.value = false
  paginationData.currentPage = 1
  paginationData.pageSize = 20
  paginationData.total = 0
  tableData.value = []
  initData()
}
const initData = () => {
  loading.value = true
  setTimeout(() => {
    const startIndex = (paginationData.currentPage - 1) * paginationData.pageSize
    const endIndex = startIndex + paginationData.pageSize
    const list = allList.value.slice(startIndex, endIndex)
    const result = {
      total: allList.value.length,
      records: list
    }
    loading.value = false
    const { total, records } = result
    paginationData.total = total
    tableData.value = records
  }, 500)
}

// 改变页数
const handleSizeChange = (val: any) => {
  paginationData.pageSize = val
  initData()
}
// 改变页数
const handleCurrentChange = (page: any) => {
  paginationData.currentPage = page
  initData()
}

const exportExcelTable = (json: any[], name: string, titleArr: string[], sheetName: string, fields: string[]) => {
  let data: any = []
  if (!Array.isArray(json)) return console.warn("数据请传入数组")
  if (!Array.isArray(titleArr)) return console.warn("标题请传入数组")
  if (!Array.isArray(fields)) return console.warn("字段请传入数组")
  data = json.map((obj) => {
    return fields.map((field) => {
      return obj[field]
    })
  })
  data.splice(0, 0, titleArr)
  // fields为英文字段表头,一般不需要，需要直接把下面注释打开即可
  // data.splice(0, 0, fields);

  const ws = XLSX.utils.aoa_to_sheet(data)
  const wch = [{ wch: 12 }, { wch: 50 }] // 设置列宽
  const wb = XLSX.utils.book_new()
  // 此处隐藏英文字段表头
  const wsrows = [{ hidden: false }]
  ws["!rows"] = wsrows // ws - worksheet
  ws["!cols"] = wch // 设置列宽
  XLSX.utils.book_append_sheet(wb, ws, sheetName)
  /* generate file and send to client */
  XLSX.writeFile(wb, name + ".xlsx")
}
const exportToExcelSelected = () => {
  if (allList.value.length === 0) {
    ElMessage.warning("请先导入数据")
    return
  }
  if (multipleSelection.value.length === 0) {
    ElMessage.warning("请至少选择一条数据")
    return
  }
  const titleArr = ["FNSKU", "简化标题"] //表头第一行从左到右
  const fields = ["code", "title"] //标题对应的字段名
  exportExcelTable(multipleSelection.value, fileName.value, titleArr, "sheetName", fields) //列表变量名、文件名、第一行标题、表名、字段名
}
const exportToExcel = () => {
  if (allList.value.length === 0) {
    ElMessage.warning("请先导入数据")
    return
  }
  const titleArr = ["FNSKU", "简化标题"] //表头第一行从左到右
  const fields = ["code", "title"] //标题对应的字段名
  exportExcelTable(allList.value, fileName.value, titleArr, "sheetName", fields) //列表变量名、文件名、第一行标题、表名、字段名
}
</script>

<style lang="scss" scoped>
.search-wrapper {
  margin-bottom: 20px;
  :deep(.el-card__body) {
    padding-bottom: 2px;
  }
}

.toolbar-wrapper {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.table-wrapper {
  margin-bottom: 20px;
}

.pager-wrapper {
  display: flex;
  justify-content: flex-end;
}
</style>
