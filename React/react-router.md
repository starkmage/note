[React Router 官方文档翻译](https://juejin.cn/post/6854573220470013966)

## 两种模式

* <BrowserRouter>，即为history模式，默认的
* <HashRouter>，哈希模式

## 路由匹配

* 当渲染一个`<Switch>`时，它会遍历所有子组件，也就是`<Route>`组件，找到第一个path匹配当前URL的组件就会停止查找，然后渲染这个`<Route>`组件，忽略其他所有的。因此**应该把路径更具体的路由组件放在前面**。
* 通常会将`<Route path="/">`放在`<Switch>`的最后。

## react-router中的exact和strict

* exact：不允许匹配子路由，对`/`不会进行控制
* strict：不关心子路由，但path结尾有`/`时，url也必须有才能匹配

https://juejin.cn/post/6844903841217904648

## Hooks

* useHistory()
* useLocation()
* useParams()
* useRouteMatch()