# 服务端渲染相关知识
## Vue2.x SSR官方指南
[官方文档](https://ssr.vuejs.org/zh/#%E4%BB%80%E4%B9%88%E6%98%AF%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF%E6%B8%B2%E6%9F%93-ssr-%EF%BC%9F)

[官方demo](https://github.com/vuejs/vue-hackernews-2.0/)

## 介绍
### 服务端渲染的缺点
1.服务端渲染只能在某些生命周期的钩子函数中使用；一些外部库可能需要特殊处理；

2.服务端渲染需要处于Node.js Server运行环境；

3.服务端渲染会导致服务端负载加重，可以采用缓存策略；

### 服务端渲染和预渲染的区别

预渲染不需要web服务器动态实时生成Html，webpack就可以完成（```prerender-spa-plugin```）。

## 基本用法
### 注意点
```<!--vue-ssr-outlet-->```注释不能去掉，这里是HTML标记插入的地方；

使用方法：

```ssr.createRenderer().renderToString(app, context, callback)```

```createRenderer(options)```可传入options参数，参数信息如下：
* ```template```：页面模板，支持基本插值语法；
* ```clientManifest```：客户端构建的manifest对象，该对象包含整个构建过程的信息。提供给bundleRenderer使用
* ```inject```：使用template时，是否执行自动注入，默认为```true```。
* ```shouldPreload```：是一个函数，用来控制什么文件生成 ```<link rel="preload">```
* ```shouldPrefetch```：是一个函数，同上
* ```runInNewContext```：只用于```createBundleRenderer```，默认情况下，bundle renderer创建一个新的V8上下文，并重新执行整个renderer，但这样开销很大。因此提供了 false | once选项。
* ```basedir```：只用于```createBundleRenderer```。
* ```cache```：建议使用```lru-cache```库；此外缓存对象至少要实现```get```和 ```set``属性。
* ```directives```：允许服务端实现自定义指令。

```renderToString(app, context, callback)```如果没有参数callback，则返回一个promise；
* ```app```是一个Vue实例；
* ```context```是要插入到html中的字段值；

### 模板插值

```createRenderer()```的template需要的几个参数，可以通过```renderToString(app, context)```的context传进去。


## 编写通用代码
1.服务端渲染时，每次请求都应该是全新的、独立的应用程序；

为了避免vuex、vue-router的全局污染，需要提供一个```createApp```函数，保证每次访问时都返回一个全新的实例。

2.生命周期函数只能用```beforeCreate``` 和 ```created```。且不要在这两个函数中，做有副作用的操作。

3.浏览器和Node服务器 使用的API，不是完全相同的；针对适用于特定平台的API，在打包时一定要注意：
* 操作DOM的API
* window | document 相关的API

针对仅浏览器可用的API，需要惰性访问他们；

集成第三方库时，也要注意是否符合以上要求，否则需要mock一些全局变量来使用他们，但这仍然会比较棘手。

4.Vue的自定义指令

针对直接操作DOM的自定义指令，在服务端渲染时，会导致错误。推荐两种解决方法：
* 使用组件作为抽象机制？？？
* 在创建server renderer时，用```directives```选项，提供服务器版本；


## 源码结构
### 如何避免污染
为了避免多个请求间共享一个应用实例，导致数据的交叉污染，我们最好将App的实例化方法，提取为一个函数；
对vuex、router、event-bus等的处理，都要抽象为一个函数哦！

### 为什么要在服务器上，使用webpack打包vue？
webpack的许多功能，比如css-loader、file-loader在Node.js中不能运行，所以服务端、客户端需要各打包一份。

### webpack的目录结构
由于两端都要打包，所以写打包配置时要有一个通用的app.js文件（new一个Vue实例），然后客户端（挂载实例）、服务端（返回app）都各有一个入口文件。

## 路由和代码分割

### 服务端路由
路由也要注意给每个请求一个实例，防止路由污染。

动态加载组件：```() => import()```

客户端也需要 ```router.onReady()```

## 数据预取和状态

为保证数据的一致性，服务端预加载的数据 和 客户端的数据，必须完全一致，否则会因为状态不同导致混合失败。

* 数据需要位于视图组件之外，存放在Vuex容器中。
* 在通用的app.js文件中，引入store；然后调用 ```vuex-router-sync``` 将router 和 store数据同步。

服务端获取数据的时间：渲染之前，预取和解析数据完成，并存放到strore中。然后序列化数据 ，再内置数据，方便客户端直接从store获取内联预置状态。
客户端获取数据的实际：在被挂载之前。

### 如何根据路由，确定组件，然后确定需要预渲染的数据呢？

* 自定义静态函数```asyncData```：无法访问```this```,需要将store和路由信息作为参数传进去；

* 服务端获取数据：路由 ==> 获取匹配到的组件 ==> 判断组件是否有asyncData ==> 调用asyncData传递数据 ==> 将store.state挂载到  context.state

### 客户端数据预取
有两种处理方法：
* 在路由导航之前解析数据：获取到所有数据且全部解析后，再处理当前视图。需要在全局路由钩子函数之前，处理```asyncData```
    * 缺点：如果解析数据时间较长，会有较长时间的白屏。
* 匹配到对应的视图后，再获取数据：将客户端的预处理数据逻辑，放在每个组件的```beforeMount```中。
    * 优点：路由导航被触发时，程序的响应速度更快。
    * 缺点：每个组件都要有个loading条。
使用建议：无论哪种策略，当路由组件可重用时，都应该调用```asyncData```函数，也可以通过纯客户端的```mixin```来处理。

### vuex状态区分
将每个模块对应的数据，分割到各自的模块中：```store.registerModule```

注意要在destroyed时，```unregisterModule```，避免在客户端重复注册模块。


## 客户端激活
刚开始，页面是由服务端的Html提供服务的，那后来怎么切换为客户端的bundle呢？

服务端的数据是已经渲染好的，我们不需要丢弃服务端的数据重新渲染，而是将静态的HTML 激活为 动态的。
服务端的html有```data-server-rendered=true```标记，它会让客户端Vue知道这部分HTML是Vue在服务端渲染的，并且应该以激活模式来挂载。

针对没有上述标记的html中，客户端也可以想```app.$mount('#app', true)```传入第二个参数强制使用激活模式。

### 需要注意的点
请确保在模板中填入有效的HTML。

浏览器可能会在<table>中自动插入```<tbody></tbody>``` 导致HTML不匹配。
```HTML
<table>
    <tr><td>表格内容</td></tr>
</table>
```

## Bundle Renderer指引
```createBundleRenderer(serverBundle, options)```将服务端的代码和客户端的代码相结合，提供代码的热更新、关键css注入等。

```serverBundle```：可以是一个绝地路径，指向已经构建好的bundle文件（js/json）。也可以是由webpack + ```vue-server-renderer/server-plugin```生成的bundle对象。

```bundlre renderer```相比```renderer```有以下优点：
* 内置sourcemap支持
* 支持热重载
* 支持关键css注入
* 使用 clientManifest进行资源注入。
注意：使用时推荐将 runInNewContext选项设置为```false```或```once```;

## 构建配置
基础配置：```base```

客户端配置：```client```

服务端配置：```server```
* ```VueSSRServerPlugin```：可以将服务器的整个输出构建为单个的JSON文件，默认文件名为```vue-ssr-server-bundle.json```，可通过参数```filename```修改；
* ```externals```：需要外置化处理的模块，可以提高构建速度；```whitelist```属性可以排除需要处理的依赖模块

### 生成clientManifest
renderer需要有服务器和客户端的构建信息：server bundle 和 客户端清单（client manifest）

### 手动资源注入
```renderToString```的```context```对象会暴露多个方法，可用于手动控制模板。


## CSS管理

```.vue```文件内部的css，可以通过```vue-loader```配置```extractCSS```选项提取，如果想从```JS```中导入```CSS```，需要使用合适的```loader```。

从依赖模块中导入CSS：注意在服务器端构建过程中，不应该外置化提取。

## Head管理

```this.$ssrContext```可以直接访问组件中的服务器端渲染上下文。

## 缓存

为什么要用缓存？
```SSR```虽然速度较快，但创建组件实例和虚拟DOM节点，仍然会有开销，无法和纯基于字符串拼接的模板性能相比较。

缓存能够改善响应时间、减少服务器负载。

### 页面级别缓存

如果服务端渲染的页面是静态的（对所有用户来说内容不是特定的），那我们可以用```micro-caching```的缓存策略。

### 组件级别缓存
```lru-cache```库，在创建renderer时，就传入cache属性，然后通过```serverCacheKey```函数来缓存组件。
每个缓存组件还需要定义一个唯一的name，每个cache key对应一个name。
每个```serverCacheKey```返回的key应该包含足够的信息，表示渲染结果的具体情况。
如果一个页面的渲染结果由```id```决定，则可以。
如果渲染结果还依赖其他的prop，则需要修改```serverCacheKey```的实现，来考虑其他变量。

一般适用于 纯静态组件。

### 什么时候不要使用组件缓存
* 依赖于全局状态的子组件。
* 对渲染上下文产生副作用的子组件。

## 流式渲染```renderToStream```
```renderToStream```返回的是数据流，在Node中可用 ```stream```来处理。
* 优点：流式渲染会尽快发送数据
* 缺点：但可能第一个数据chunk被发出时，子组件甚至可能不被实例化，生命周期钩子也不会被调用。因此如果依赖由生命周期钩子函数填充上下文数据，则不建议用流式传输模式。

## 非Node.js环境中使用
非Node环境相关的构建，都编译到了 ```vue-server-renderer/basic.js```中
