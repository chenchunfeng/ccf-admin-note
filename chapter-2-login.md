## 项目架构之搭建登录架构解决方案与实现

### 安装element ui plus 

可以通过vue-cli来安装，也可以按[官方文档](https://element-plus.gitee.io/zh-CN/guide/installation.html#%E7%89%88%E6%9C%AC)操作

- vue add element-plus

- 报 UNABLE_TO_VERIFY_LEAF_SIGNATURE
- 把 npm config set strict-ssl false
- 安装成功后设回true


## svg icon 组件


#### require.context
```javascript

// https://webpack.docschina.org/guides/dependency-management/#requirecontext
require.context(
  directory,
  (useSubdirectories = true),
  (regExp = /^\.\/.*$/),
  (mode = 'sync')
);

// demo
require.context('./test', false, /\.test\.js$/);
//（创建出）一个 context，其中文件来自 test 目录，request 以 `.test.js` 结尾。
```

此导出函数有三个属性：resolve, keys, id。

resolve 是一个函数，它返回 request 被解析后得到的模块 id。
keys 也是一个函数，它返回一个数组，由所有可能被此 context module 处理的请求（译者注：参考下面第二段代码中的 key）组成。

如果想引入一个文件夹下面的所有文件，或者引入能匹配一个正则表达式的所有文件，这个功能就会很有帮助，例如：

```javascript
function importAll(r) {
  r.keys().forEach(r);
}

importAll(require.context('../components/', true, /\.js$/));
```


#### aria-hidden=“true”

为了避免屏幕识读设备抓取非故意的和可能产生混淆的输出内容（尤其是当图标纯粹作为装饰用途时），我们为这些图标设置了 aria-hidden=“true” 属性。

#### svg-sprite-loader

> npm install svg-sprite-loader@6.0.9 --save-dev
```javascript
// vue.config.js

// 去掉原有的svg rule
    config.module
      // 规则
      .rule('svg')
      // 忽略
      .exclude.add(resolve('src/icons'))
      // 结束
      .end()
// 使用svg-sprite-loader 处理icons里面的svg
    config.module
      // 规则
      .rule('icons')
      // 正则，解析 .svg 格式文件
      .test(/\.svg$/)
      // 解析的文件
      .include.add(resolve('src/icons'))
      // 结束
      .end()
      // 新增了一个解析的loader
      .use('svg-sprite-loader')
      // 具体的loader
      .loader('svg-sprite-loader')
      // loader 的配置
      .options({
        symbolId: 'icon-[name]'
      })
      // 结束
      .end()

```

大概原理就是， 使用svg-sprite-loader处理 require进来的所有svg文件，并他们转成symbol 追加到body下面，这个就可以 全局use #icon-xxxx使用对于的svg icon 


#### 添加远程路径方法

```javascript
export function isExternal(path) {
  return /^(https?:|mailto:|tel:)/.test(path)
} 
```
```html
  <div
    v-if="isExternal"
    :style="styleExternalIcon"
    class="svg-external-icon svg-icon"
    :class="className"
  />
```

## 通用后台登录方案解析

- axios       /utils/request
- 接口封装    /api/xxx
- 登录请求动作   fetchLogin  method 点击按钮
- token缓存     headers.token = xxxx
- 登录鉴权      401 403 axios 拦截 路由全局前置守卫

ref="xxxx"
跟vue2区别，没有this.$ref.xxx

vue3 demo
```html
 <el-form ref="loginFormRef"></el-form>
```
```javascript
    
   const loginFormRef = ref(null) 
   loginFormRef.value.validate()
```
虽然声明  xxxref = ref(null)  


> ref reactive 怎样选，统一使用ref ，为什么？ 性能高
[文档] (https://blog.vuejs.org/posts/vue-3.2.html)


由于课程api限制，需要自己mock数据
```javascript
// vue.config.js
  devServer: {
    // 配置反向代理
    proxy: {
      // 当地址中有/api的时候会触发代理机制
      '/api': {
        // 要代理的服务器地址  这里不用写 api
        target: 'xxx',
        changeOrigin: true, // 是否跨域
        bypass: function (req, res) {
          if (req.headers.accept.indexOf('html') !== -1) {
            return '/index.html'
          } else if (process.env.MOCK === 'yes') {
            console.log('req.path', req.path)
            const name = req.path.split('/api/')[1].split('/').join('_')

            try {
              const mock = require(`./mock/${name}`)
              const result = mock(req.method)
              delete require.cache[require.resolve(`./mock/${name}`)]
              return res.send(result)
            } catch (e) {}
          }
        }
      }
    }
  },

// 根据目录再新建mock文件夹，一个接口一个文件
```


## 使用axios 拦截器，对响应的数据做简化处理
```javascript

axios.interceptors.request.use(config => {
  if (config)
}, err => {
  console.log(err)
})

// 接口响应结构
// code: 200
// data: {token: "afea405c-9ff0-4b32-9fcd-3cad593586f5"}
// message: "登录成功"
// success: true

import { ElMessage } from 'element-plus'

axios.interceptors.response.use(response => {
  const { success, message, data } = response
  if (success) {
    return data
  } else {
    ElMessage.
    ElMessage.error(message)
    return Promise.resolve(new Error(message))
  }
}, err => {
  ElMessage.error(err.message)
  return Promise.resolve(new Error(err))
})
```

## 封装localStorage

1. 扩展value 值可支持复杂类型 array object
2. 统一入口，后面可做一个统一处理，劫持数据等数据

## 全局权限router.beforeEach 不在放在store里面处理，单独新增permission文件处理，单一职责


- -! 搞错参数顺序，搞了很久

to from next   个人搞成from to next



