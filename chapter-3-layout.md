## layout 架构

一个后台管理系统,一般可拆分为 mene navBar main 三大部分

涉及的核心解决功能：

1. 用户退出方案
2. 动态侧边栏方案
3. 动态面包屑方案


### 拆分组件

- SideBar
- NavBar
- AppMain

### 创建全局css

统一css变量, 可应用到各组件的css 或者js中

新增

- styles/variable.scss
- styles/mixin.scss

```scss
定义scss变量,并导致

// demo
$primaryColor: red;
$sideBarWidth: 210px


:export {
  primaryColor: $primaryColor;
  sideBarWidth: $sideBarWidth
}
```

```vue
<script>
// 应用

import variable from 'xxxx';

variable.primaryColor
</script>
<style>
@import '~@/xxxxx';
.xxx {
  color: $primaryColor
  width: calc(100% - #{$sideBarWidth});    
}
</style>
```


scss-#{}插值
一般我们定义的变量都为属性值，可直接使用，但是如果变量作为属性或在某些特殊情况下则必须要以 #{$variables} 形式使用。

```scss
$borderDirection:       top !default; 
$baseFontSize:          12px !default;
$baseLineHeight:        1.5 !default;
// 应用于 class 和属性
.border-#{$borderDirection} {
    border-#{$borderDirection}: 1px solid #ccc;
}
// 应用于复杂的属性值
body {
    font:#{$baseFontSize}/#{$baseLineHeight};
}
```


### 因为navBar 需要显示用户信息，获取用户基本信息

三步走
- 定义用户信息api
- 定义接口调用方法
- 定义接口触发时机

代码分析
1. 用户基本信息是全局的，需要保存在vuex中
2. 触发时机可能是多个，比如说登录成功后，更新用户信息，所以请求的方法也写在vuex actions里面

> 本案例里面permission里面统一管理权限，需要用到用户信息里面的权限列表，所有统一在里面触发用户信息


### 退出登录

按上面的页面，一个功能可分为三块

1. 定义退出登录api
2. 定义接口调用方法，统一放vuex
3. 定义接口触法时机


- 用户主动点击退出
- 前端token超时（后端也要做）
- token被动失效，一般是后端设置超时，或者单点登录被挤下线

退出登录后，需要做哪里动作

- 清除token
- 清除用户信息
- 清除用户权限
- 返回login页


该项目中，在请求拦截器里面主动处理token是否超时
在响应拦截器里面处理401 再logout

- 401 一般是没权限，或者token超时 失效  单点被挤
- 403 权限不足，token有效，只是权限不足