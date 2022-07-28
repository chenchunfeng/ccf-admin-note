## 面包屑

一般不使用静态面包屑，原因

- 每个页面都要写一个
- 如果数据结构发生改变，还要修改
  

所以就要按照一定的规则，添加动态路由

这里主要使用route.matched 来获取点击的路由

```javascript

const breadcrumbData = ref([])

const getBreadCrumb = () => {
  breadcrumbData.value = route.matched.filter(
    (item) => item.meta && item.meta.title
  )
}

```

