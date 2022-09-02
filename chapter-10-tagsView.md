## tags view

需求： 

1. 记录vue-router history 生成list
2. app main 顶部展示，点击跳转对应页面
3. tag 右键添加关闭当前、关闭全部、关闭右侧
4. route-view 添加过渡动画

### 创建数据源 collection vue-router history


```vue

<!-- 监听route 记录到 vuex -->
<script setup>

import { watch } from 'vue';
import { useRoute } from 'vue-router'
import { useStore } from 'vuex'

const route = useRoute()
const store = useStore()

watch(route, (val) => {
  // val 需要根据展示定好数据格式
  store.commit('addTagsViewList', val)
})
</script>

```

```javascript

export default {
  namespaced: true,
  state: () => ({
    tagsViewList: []
  }),
  mutations: {
    addTagsViewList(state, tag) {
      state.tagsViewList.push(tag)
    }
  },
  actions: {}
}

```


### 创建tags view 组件，展示数据
### 创建context menu组件，根据右键点击位置，定位到该位置
### 过渡动画

https://router.vuejs.org/zh/guide/advanced/transitions.html#%E5%9F%BA%E4%BA%8E%E8%B7%AF%E7%94%B1%E7%9A%84%E5%8A%A8%E6%80%81%E8%BF%87%E6%B8%A1

遇到问题
Component inside <Transition> renders non-element root node that cannot be animated

主要是某些组件没有template 下面没有根标签


