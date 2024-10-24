<script lang="ts" setup>
import { onMounted } from "vue"
// 将 Element Plus 的语言设置为中文
import zhCn from "element-plus/es/locale/lang/zh-cn"
import IpcDict from "@/constants/ipc-dict"
import { useTheme } from "@/hooks/useTheme"
import { APP_TITLE } from "@/hooks/useTitle"

const { initTheme } = useTheme()

/** 初始化主题 */
initTheme()

onMounted(() => {
  console.log(`Hello, ${APP_TITLE}! \n`)
  const winState: WinStateDTO = {
    width: 1024,
    height: 768,
    center: true,
    maxable: true,
    resizable: true
  }
  window.vIpcRenderer.send(IpcDict.CODE_01001, winState)
})
</script>

<template>
  <el-config-provider :locale="zhCn">
    <router-view />
  </el-config-provider>
</template>
