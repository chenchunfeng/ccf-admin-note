## 动态切换主题

原理大概就是定义css变量，修改全局变量，组件再渲染新颜色

功能大概分为三部分
- 主题色选取组件
- 自定义组件 主题色切换
- element plus 主题色切换



## 主题色选取组件


1. 组件页面
```html
<!-- theme change -->
<template>
  <el-dialog>
    <el-color-picker v-model="color" />
  </el-dialog>
</template>
```

> el-color-picker 可以使用predefine预定义颜色，方便快速选择


2. 全局css变量定义，一般项目中会存在/src/styles/variables.scss

```scss
$menuText: #bfcbd9;
$menuActiveText: #b4e03a;

export {
  menuText: $menuText;
  menuActiveText: $menuActiveText;
}
```

3. 全局静态常量，定义主题色local storage 的key 和defaultThemeColor 
```javascript
// /src/constant/
export const MAIN_COLOR = 'mainColor'
export const DEFAULT_COLOR = '#409eff'
```

4. vuex 定义数据保存就切换方法

```javascript
// /src/store/modules/theme.js
import { setItem, getItem } from '@/utils/storage'
import { MAIN_COLOR, DEFAULT_COLOR} from '@/constants'

export default {
  namespaced: true,
  state: () => ({
    mainColor: getItem(MAIN_COLOR) || DEFAULT_COLOR,
  }),
  mutations: {
    setMainColor(state, color) {
      state.mainColor = color
      setItem(MAIN_COLOR, color)
    }
  },
  actions: {}
}
```

## 自定义组件 主题色切换

这个相对简单，就是在html里面使用:style="{color: mainColor}"

如果想在css里面使用，vue3.2提供了v-bind(xxx)的方法 https://vuejs.org/api/sfc-css-features.html#v-bind-in-css

```vue
<script setup>
import variables from '@/styles/variables.scss';
</script>
<style lang="scss">
.xxx {
  color: v-bind(variables.mainColor)
}
</style>

```


## element plus 主题色切换

这个有些难度, 官方有调整主题色方案，但哪是预设主题色方案，并不能动态切换。
现在使用的方法，是copy import 'element-plus/dist/index.css' 的内容，把主题色替换成自己的主题色，再插入dom节点，利用优化级覆盖原有的


```javascript

import axios from 'axios'
const getOriginStyle = async () => {
    const version = require('element-plus/package.json').version
    // 通过安装对接的版本获取index.css 源码
    const { data } = await axios(`https://unpkg.com/element-plus@${version}/dist/index.css`)
    return data 
}

// element ui的主题色相关比较多，需要通过主题色计算重新设置，这使用scss-color-function计算，使用rgb-hex   rgg转十六进制
// 这里有特定的配方
// formula.json
{
  "shade-1": "color(primary shade(10%))",
  "light-1": "color(primary tint(10%))",
  "light-2": "color(primary tint(20%))",
  "light-3": "color(primary tint(30%))",
  "light-4": "color(primary tint(40%))",
  "light-5": "color(primary tint(50%))",
  "light-6": "color(primary tint(60%))",
  "light-7": "color(primary tint(70%))",
  "light-8": "color(primary tint(80%))",
  "light-9": "color(primary tint(90%))",
  "subMenuHover": "color(primary tint(70%))",
  "subMenuBg": "color(primary tint(80%))",
  "menuHover": "color(primary tint(90%))",
  "menuBg": "color(primary)"
}

// 使用新颜色替换originStyle 
import color from 'css-color-function'
import rgbHex from 'rgb-hex'
import formula from '@/constant/formula.json'


const generateColors = (mainColor) => {
  const colors = {
    primary: mainColor
  }
  Object.keys(formula).forEach(key => {
    // scss-color-function计算，并使用rgbHex转换成十六进制
    colors[key] = rgbHex(color(formula[key]))
  })

  return colors
}



// 替换origin styles 里面的色值，这个也有一份配方表，element ui 对应的变量色值
const replaceStyle = (oriStyle) => {
    // element-plus 默认色值
  const colorMap = {
    '#3a8ee6': 'shade-1',
    '#409eff': 'primary',
    '#53a8ff': 'light-1',
    '#66b1ff': 'light-2',
    '#79bbff': 'light-3',
    '#8cc5ff': 'light-4',
    '#a0cfff': 'light-5',
    '#b3d8ff': 'light-6',
    '#c6e2ff': 'light-7',
    '#d9ecff': 'light-8',
    '#ecf5ff': 'light-9'
  }

  // 根据默认色值为要替换的色值打上标记
  Object.keys(colorMap).forEach(key => {
    const value = colorMap[key]
    // 全局替换，不区分大小写
    data = data.replace(new RegExp(key, 'ig'), value)
  })

  return data
}


// 最后整合调用
const generateNewStyle = async mainColor => {
    const colors = generateColors(primaryColor)
    let cssText = await getOriginStyle()

    // 遍历生成的样式表，在 CSS 的原样式中进行全局替换
    Object.keys(colors).forEach(key => {
      cssText = cssText.replace(
        new RegExp('(:|\\s+)' + key, 'g'),
        '$1' + colors[key]
      )
    })

  return cssText
}

```


上面的步骤已生成新的index.css内容，后面就插的document即可

```javascript

const newStyle = generateNewStyle()
function writeDocument(newStyle) {
  const style = document.createElement('style');
  style.innerText = newStyle;
  // 在head标签下插下新style节点
  document.head.appendChild(style);
}

```


## 缺陷修补
1. 刷新后，element ui 新主题色不生效
   在加载app.vue组件时，再跑一次替换代码
2. 自定义修改的element ui 组件样式会被覆盖，注意优先级
3. 对于自定义组件，直接使用variables.scss。只能是预设方法，并不能动态切换。

哪里就要使用vuex里面的css变量
```vue
<template>
  <div :style="{backgroundColor: $store.getter.cssVar.bgColor}"></div>
</template>
<script setup>
  import { useStore } from 'vuex'
  const cssVar = store.getter.cssVar
</script>
<style lang="scss">
  .xxx {
    color: v-bind(cssVar.mainColor)
  }
</style>
```

vuex
```javascript
// getters

import { getItem } from '@/utils/storage'
import { generateColors } from '@/utils/theme'

export default {
  cssVar: (state) => {
    // 保证cssVar 依赖值的响应变化
    return {
      ...state.theme.variables,
      ...generateColors(getItem(MAIN_COLOR))
    }
  },
}

// children module theme
import variables from '@/styles/variables.scss'

export default {
  namespaced: true,
  state: () => ({

    variables,
  }),
  mutations: {
    setMainColor(state, mainColor) {
      state.variables.menuBg = newColor
    }
  }
}

```

