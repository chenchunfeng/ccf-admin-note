## 全屏方案

F11 ?

浏览器给出了对应的[api](https://developer.mozilla.org/zh-CN/docs/Web/API/Fullscreen_API) 
1. Document.exitFullscreen()：该方法用于请求从全屏模式切换到窗口模式
2. Element.requestFullscreen()：该方法用于请求浏览器（user agent）将特定元素（甚至延伸到它的后代元素）置为全屏模式

原生api有一些问题，比如切换全屏时，窗口会变黑

document.getElementById('app').requestFullscreen()

这里我们使用它的包装库 screenfull

```vue
<!-- 安装screenfull   https://www.npmjs.com/package/screenfull-->
npm install screenfull@5.1.0

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import screenfull from 'screenfull'

// 是否全屏
const isFullscreen = ref(false)

// 监听变化
const change = () => {
  isFullscreen.value = screenfull.isFullscreen
}

// 切换事件
const onToggle = () => {
  screenfull.toggle()
}

// 设置侦听器
onMounted(() => {
  screenfull.on('change', change)
})

// 删除侦听器
onUnmounted(() => {
  screenfull.off('change', change)
})
</script>
```