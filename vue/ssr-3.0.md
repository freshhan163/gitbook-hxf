# 服务端渲染相关知识
## Vue3.x SSR官方指南
[官方文档](https://v3.cn.vuejs.org/guide/ssr/introduction.html#%E4%BB%80%E4%B9%88%E6%98%AF%E6%9C%8D%E5%8A%A1%E7%AB%AF%E6%B8%B2%E6%9F%93-ssr)

## 介绍
### 什么是服务端渲染
一个SSR的应用，也被称为“同构”或“通用”应用，一套代码可以运行在两个端上。

## 起步
### 安装
```bash
npm install @vue/server-renderer
```
注意：Node版本需要是12+；

### 渲染一个Vue应用
```createSSRApp```从```vue```库中导入。
```renderToString(app)```从```@vue/server-renderer```库中导入。

## 编写通用的代码
每个请求都应该对应一个新的应用实例，避免跨请求的状态污染。

服务端在发送html之前，数据是被解析好的。

组件生命周期钩子，只有```beforeCreate```和 ```created```会被执行。避免在上面两个钩子中执行有副作用的代码.

### 访问特定平台的API
比如```axios```库，是在服务端和客户端有相同API的库。

针对只存在浏览器中的API，建议在只存在于客户端的声明周期钩子里访问。

如果一个第三方库没有通用的写法，需仿造一些全局变量。

自定义指令：建议使用组件抽象机制，而不是指令。

## 源代码结构

### 避免有状态的单例模式
为每个请求创建一个新的Vue根实例：编写一个工厂函数。

### 介绍构建步骤
```App.vue```：根组件
```entry-client.js```：引用```createSSRApp```创建应用并挂载到DOM中。
```entry-server.js```：引用```createSSRApp```创建应用，并返回创建的应用。

## 构建配置
### 服务端构建和客户端构建的不同
* 生成一个```webpack.manifest```：让webpack追踪所有模块对应到包中的JSON文件。（```webpack-manifest-plugin```）
* 将应用依赖改为外部拓展：减小服务端的包大小，同时需要将webpack处理的依赖如```css```和 ```vue```排除在外。（```webpack-node-externals```）
* 修改webpack的构建目标：commonJs
* 添加环境变量：告诉webpack当前工作是服务端渲染

### 配置示例
* 禁用```cache-loader```：否则客户端构建版本会从服务端构建版本使用缓存过的组件
* 根据环境变量修改入口文件
* 修改target


### 生成clientManifest
有了客户端构建单```clientManifest```，渲染器就同时拥有了服务端和客户端构建版本的信息，这样就可以自动推断并向渲染出来的HTML中注入preload 和 prefetch指令。优点如下：
* 针对```hash```文件，可以替换```html-webpack-plugin```的资源URL
* 针对webpack的按需分割特性，可以确保优化过的代码块是被```preload/prefetch```的???

## 服务器配置

* ```createApp```需要从```manifest```文件中获取；
* 为所有静态资源，定义正确的路径
* 调用```renderToString```
* 将```html```的内容，替换为服务端渲染出来的应用内容。

## 路由和代码分离

每个请求都要有一个干净的路由实例：新增一个```createRouter```方法。
客户端使用路由：```createRouter(createWebHistory())```
服务端使用路由：```createRouter(createMemoryHistory())```

### 代码分离
Vue Router提供了懒加载，允许动态import```() => import()```

客户端和服务端，
客户端要先等路由器解析完成后，再挂载：```router.isReady().then()```
服务端也要等路由器解析完成后，再调用renderToString：```await router.isReady()```

## 客户端Hydration
即将服务端静态的HTML 转换为 客户端动态的DOM：```createSSRApp```

使用时要注意以下两点：
* 确保客户端和服务端的状态一直，不要依赖客户端特有的API
* 确认模板中编写的HTML是有效的，有些浏览器会自动注入一些dom，比如```<tbody>```，可以使用[HTML验证器插件](https://html-validate.org/)

## Vue2 VS Vue3
### API对比
Vue2
* ```createRenderer```、 ```createBundleRenderer```：服务端创建Bundle
* ```new Vue({})```：客户端创建app

Vue3
* ```createSSRApp```：创建客户端/服务端Bundle

等待Vue的Router解析
* Vue2：
```JS
router.push(url);
router.onReady()
```

* Vue3：
```JS
const app = createSSRApp(App);
const router = reateRouter(createWebHistory());
app.use(router);
await router.isReady().then();
```

### 流程对比
