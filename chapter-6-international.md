## vue3.2国际化


###  前言
项目使用的为
- vue3.2
- element plus
- vue-i18n@next

国际化的基本可理解为下面代码
```javascript
const message = {
  zh: {
    hello: '你好'
  },
  en: {
    hello: 'hello'
  }
}

const locale = "zh"

function t(kye) {
  return message[locale][key] || ""
}

console.log(t(hello)) // "你好"
```

由上面demo可清楚，国际化有三个要点

- message  语言包
- locale 当时语言
- t 函数

###  安装初始化vue-i18n

```javascript

npm install vue-i18n@next

//  新建 /src/i18n/index.js

import { createI18n } from "vue-i18n"

const message = {
  zh: {
    hello: '你好'
  },
  en: {
    hello: 'hello'
  }
}

const locale = "zh"

const i18n = createI18n({
  // 使用 Composition API 模式，则需要将其设置为false
  legacy: false,
  // 全局注入 $t 函数
  globalInjection: true,
  locale,
  messages
})


export default i18n


//  main.js
import i18n from "@/i18n"
app.use(i18n)


```
### localStorage vuex 数据保存
```javascript
import { getItem, setItem } from "@/utils/storage"
const LANG = 'language'


export default {
  namespaced: true,
  state: () => ({
    language: getItem(LANG) || 'zh'   // 防止刷新丢失问题
  }),
  mutations: {
    setLanguage(state, lang) {
      state.language = lang
      setItem(LANG, lang)
    }
  },
}

```

### 切换语言的button组件

```javascript
// 其实就是调一下vuex,  具体ui因项目而异
import { useI18n } from 'vue-i18n'
import { useStore } from 'vuex'
import { ElMessage } from 'element-plus'

const store = useStore()
const i18n = useI18n()

// 切换语言的方法
const handleSetLanguage = lang => {
  i18n.locale.value = lang
  store.commit('app/setLanguage', lang)
  ElMessage.success(i18n.t('msg.toast.switchLangSuccess'))
}
```


> el-tooltip 里面直接放svg会报错，需要用div包裹



###  第三方组件 element plus 国际化

根据当时的语言变量，导入不同的语言包

```javascript
import ElementPlus from 'element-plus'
import zh from 'element-plus/es/locale/lang/zh-cn'
import en from "element-plus/es/locale/lang/en"
import store from '@/store'


app.use(ElementPlus, {
  locale: store.getters.language === 'zh' ? zh : en,
})


```

> 低版本的element plus还不能自动切换，需要刷新页面解决


###  具体页面替换

1. template 中 可以直接使用 {{ $t('xxxxx') }}
2. script 中 需要使用 i18n.t('xxxx') 这里的i18n  由 const i18n = useI18n() 函数生成
3. js 文件中，需要使用 i18n.global.t('xxxx')   由 import i18n from '@/i18n' 导入


> 在我的项目中，有不少保存constants的js文件，里面直接导出常量，这里有两个方法
> 1. 引入import i18n from '@/i18n' 使用i18n.global.t替换，但切换lang里，使用window.reload才能切换，用户体验不好。但代码修改量不大。
> 2. 把导出的常量数据，变成函数，传入组件的I18n，使用i18n.t 替换即可。优化是不需要刷新页面就可以切换，但导入数据的地方全部使用修改，如果是旧项目，修改量较大。




###  接口请求添加指定语言请求头 
```javascript
// axios请求拦截器
service.interceptors.request.use(
  config => {
    // 配置接口国际化
    config.headers['Accept-Language'] = store.getters.language
    return config
  error => {
    return Promise.reject(error)
  }
)
```
