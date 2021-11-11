# 服务端渲染相关知识
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
```createSSRApp()```从```vue```库中导入。

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

```entry-server.js```：引用```createSSRApp```创建应用，并返回创建的应用app。

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
* 保证客户端和服务端的状态一致，不要依赖客户端特有的API。
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
const router = createRouter(createWebHistory());
app.use(router);
await router.isReady().then();
```

## Vue SSR demo配置
* 客户端
    * app.js：createSSRApp、createRouter、createStore ==> createApp（避免状态污染）
    * client-entry.js
        * 调用createApp
        * router.isReady
        * App.mount()
    * server-entry.js
        * 调用createApp：每次访问都生成一个实例
        * 修改url，await router.isReady
        * 修改ssrContext的 state参数
        * 返回app
* 构建配置—webpack
    * 不同的入口
    * webpack.base.js
        * mode
        * resolve | module | optimizaiton
        * Plugins
            * VueLoaderPlugin，现在直接使用16.x即可，不用再用vue-loader-v16
    * webpack.server.js
        * server整体打包为一个server-bundle
        * Webpack-node-externals
        * target：node
        * plugins
            * 生成server-bundle.json（webpack-client-manifest）
    * webpack.client.js
        * entry
        * optimization
        * Plugins：添加VueSSRPlugin（生成client-manifest.json）
            * Workbox-webpack-plugin：提供service-worker服务、资源的预缓存（prod模式添加）
* development环境下（setup-dev-server.js配置）
    * 客户端打包
    * 客户端添加devMiddleware、hotMiddleware
    * 客户端打包成功后，从内存中读取```client-manifest.json```（memory-fs）
    * express添加```hot-middleware```
    * 服务端打包
    * 从内存中读取```server-bundle.json```（memory-fs）
    * 服务端监控文件变化：然后update()
* server启动
    * development环境（用```vue-bundle-renderer```库）
    * ```createBundleRenderer(bundle, options) ```依赖于 ```renderOptions.bundleRunner.createBundle(bundle, options)```
        * ```bundle```
            * ```string```
            * ```object```：包含basedir、entry、files、maps参数，其中files包含具体的文件内容
        * ```options```：具体请看[官方文档](https://github.com/nuxt-contrib/vue-bundle-renderer#readme)
    * production环境（用vue库提供的API```createSSRApp```）
        * manifest是指server的manifest
        * 加载createSSRApp函数
        ```js
        const appPath = path.join(__dirname, "../dist", manifest["app.js"]);
        const createApp = require(appPath).default;

        // 生成html
        const app = createApp();
        const page = await renderToString(app);
        ```

## Vue3.0 SSR渲染用到的库
* ```webpack-manifest-plugin```：生成json文件（客户端、服务端通用），也可以自定义plugin（获取json，生成```createBundleRenderer```API需要的options即可）
* ```vue```
    * ```createSSRApp```：生成SSRApp
    * ```vue/server-renderer```：从3.2.13+版本开始，```@vue/server-renderer```包已经被集成到了```vue```库中
        * ```renderToString```：根据传入的内容，渲染生成html
* ```vue-bundle-renderer```：为vue3.0提供的 bundle renderer，依赖于 bundle-runner
    * ```createRenderer```
    * ```createBundleRenderer(bundle, options)```
        * ```renderToString```：根据传入的内容，渲染生成html
* ```bundle-runner```：返回bundle
    * ```createBundle(bundle)```：bundle参数需要有```basedir、entry、files、maps```等参数
* ```vue-server-renderer```：适用于vue2.x，vue3.x中改用上面的```vue-bundle-renderer```
