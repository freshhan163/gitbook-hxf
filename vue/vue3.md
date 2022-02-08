# Vue3文档学习
[官方文档](https://v3.cn.vuejs.org/guide/introduction.html)

## 基础
### 安装
单文件组件的配套工具：```@vue/compiler-sfc```
命令行工具：```@vue/cli```
新的开发构建工具：```vite```

### 介绍
1.指令：会在渲染的DOM上应用特殊的响应行为。

### 应用&组件实例
1.创建一个应用实例

```Vue.createApp()```：创建一个新的应用实例，支持链式调用。
```app.mount('#app')```：挂载应用，```mount```不返回应用本身，返回的是根组件实例。

Vue没有完全遵照```MVVM```模型，但Vue的设计是受到它的启发。

根组件和其他组件没什么不同，配置选项是一样的，对应的组件实例行为也是一样的。

2.组件实例property

组件中的```data```定义的```property```，可通过组件实例```vm[key]```暴露。

Vue还暴露了组件实例的一些内置属性，如```$attr```，```$emit```。

3.生命周期钩子

生命周期钩子中的```this```指向调用它的当前活动实例。

不要在生命周期钩子、或回调上使用箭头函数，因为箭头函数没有```this```，```this```会作为变量一直向上级词法作用域查找，直至找到为止，会经常导致报错。

生命周期：
```beforeCreate``` ==> ```created``` ==> ```beforeMount``` ==> ```mounted``` ==> ```beforeUnmount``` ==> ```umounted```

```beforeUpdate``` ==> ```updated```

### 模板语法
底层实现：Vue将模板编译为虚拟DOM渲染函数。

展示原始html：```v-html```（不要渲染任意的html，容易导致XSS攻击）

绑定属性Attribute：

* 如果绑定的值是```null``` 或 ```undefined``` 或 ```false```，则该attribute不会被包含在渲染的元素上。

* 如果值是```trusy```，或值是空字符串，该attribute会被渲染。


指令Directive:

* 职责：当表达式的值改变时，其将产生的连带影响，响应式的作用域DOM。
* ```v-bind```：接受动态参数（动态属性、或动态处理函数）。
* 指令可以和修饰符一起使用

JS表达式：模板中的表达式只能访问一个受限的列表，自定义的全局变量它无法访问。


### Data Property 和方法

#### Data Property
```data```选项是一个函数，返回一个对象，Vue在创建组件实例的时候，会调用它，并以```$data```的形式存储在实例中。
可通过```vm.count```和 ```vm.$data.count```访问```count```属性。

不在```data```中声明的数据，Vue的响应式系统，不会自动跟踪它。

#### 方法
methods方法会自定绑定this，指向vue实例。
在定义方法时，应避免使用箭头函数，这会影响this指向。

防抖和节流：建议使用```Lodash```库。为了使组件彼此独立，请在created时，注册防抖函数，在mounted移除组件时，取消注册防抖函数。


### 计算属性和侦听器watch

#### 计算属性缓存 VS 方法

计算属性是基于依赖关系缓存数据，只有在相关响应式依赖发生改变时，才会重新求值。

方法在每次重新触发渲染时，都会再次执行函数。

#### 计算属性的setter
计算属性默认只有```get()```，需要的话可以手动提供 ```set()```

#### 侦听器
应用场景：在数据变化的时候执行异步 或 开销较大的操作

### class和style绑定

```class```可以是对象、数组、字符串，```class```也可以接收data、或 计算属性computed。

#### 在组件上使用class

当在使用的子组件上添加```class```时，会和组件本身的根节点class合并，组件本身的class在最前面，根组件的class有更高的优先级。

当有多个根节点时，可用 ```$attrs.class```决定将class绑定在哪个节点上。

```style```属性接收一个对象、数组。style的属性也可以接收多个值，用数组即可。
```html
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

Vue会通过运行时检测来确定哪些样式的property被浏览器支持。会一直测试直到找到它的前缀。

### 条件渲染
```v-show```：始终会被渲染，并被保留在DOM中。只是简单的切换css属性```display```。但它不支持```<template>```。

```v-if```：只有值为```truthy```才会被渲染。切换时，条件块内的事件监听器和子组件会适当的被销毁和重建。
```v-if```和```v-for```一起使用时，```v-for```的优先级更高。

### 列表渲染
可接受```for...of```和 ```for...in```两种语法。
```for..in```遍历结果和 ```Object.keys()```一致。

#### 维护状态
采用“就地更新”的策略，如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序，而是就地更新每个元素，并且确保它们在每个索引位置正确渲染。

#### 数组更新检测

### 事件处理
支持内联处理器，支持多个内联处理器。

#### 事件修饰符
目的：方法只有存粹的数据逻辑，而不是去处理DOM事件细节。

使用捕获模式：```.capture```

修饰符可以串联。

使用时需要注意修饰符的顺序。

事件也可以只有修饰符。

```.passive```修饰符，可以提高移动端的性能。

#### 按键修饰符

### 按键别名

#### 系统修饰符
* .ctrl
* .alt
* .shift
* .meta
#### .exact修饰符
用于精准的控制系统修饰符组合触发的事件。

#### 鼠标按钮修饰符
.left | .right | .middle

### 表单输入绑定

#### 修饰符
* .lazy：默认输入框的值会在```input```事件变更时更新，用了```.lazy```后，输入框的值就会在```change```事件后更新
* .number：将输入的值自动转换为```number```
* .trim：过滤用户输入的首位空白


### 组件基础
1.需要先注册

2.props传递数据

3.监听子组件事件
父组件可以通过```v-on```监听子组件实例的任意事件。
子组件调用```$emit('parentFuncName')```触发一个事件。

4.在组件上使用v-model
```v-model```的语法糖，为
```html
<input :value="searchText" @input="searchText = $event.target.value" />
```

当在组件上使用v-model时，等价于 ```@update```事件。

5.动态组件
```html
<!-- 组件会在 `currentTabComponent` 改变时改变 -->
<component :is="currentTabComponent"></component>
```

6.元素位置受限

针对```<table>```等类型的元素，其内部只能出现```<tr>```等，如果出现自定义的组件名，会被视为无效内容提升到外部，导致最终渲染结果出错。

此时建议使用动态组件```is="vue:component-name"```作为一个变通方法。

7.大小写不敏感

HTML中的属性名不区分大小写，因此浏览器将所有大写字符解释为小写。

## 深入组件
### 组件注册
1.命名
可用```kebab-case``` 或 ```PascalCase```命名规则。

2.全局注册和局部注册
没啥可说的，局部注册就是指在```components```属性中注册

### Props

1.传入布尔值的情况，需要注意以下两种情况

```html
<!-- 1. 包含该 prop 没有值的情况在内，都意味着 `true`。          -->
<!-- 2. 如果没有在 props 中把 is-published 的类型设置为 Boolean，则这里的值为空字符串，而不是“true” 。 -->
<blog-post is-published></blog-post>
```

2.传入一个对象的所有属性

以下两行代码等价

```html
<blog-post v-bind="post"></blog-post>

<blog-post v-bind:id="post.id" v-bind:title="post.title"></blog-post>
```

3.单向数据流

Vue中数据的流动是单项的，只能从父级流向 子组件，这是为了防止子组件意外变更父组件的状态。

4.Prop验证
支持的属性：type \ default \ required \ validator

type：也可以是一个自定义的类型检查

5.prop的大小写

建议统一用驼峰命名法（camelCase），子组件中使用的时候，用短横线分割命名（kebab-case）

### 非Prop的Attribute

组件会继承使用它时在其上定义的一些属性，且在父组件上定义的属性有更高的优先级。

禁用Attribute继承：在子组件中声明属性```inheritAttrs: false```，然后将传递来的所有属性绑定到其他DOM上。

针对有多个根节点的组件（默认没有子组件继承的属性），需要显式绑定```$attrs```，否则会发出运行时警告。

### 自定义事件

定义自定义事件：可通过组件的```emits```选项，定义自定义事件。

自定义事件，也可以验证，该函数接收传递给```$emit```的参数，并返回一个```Boolean```值，表示事件是否有效。

#### v-model
1.在组件中使用```v-model```参数，是```update```的语法糖。

需要手动触发```$emit('update:title', $event.target.value)```，来触发 ```@update:title="updateTitle"```事件。

自定义修饰符：自定义的修饰符，会保存在实例的```modelModifiers``属性中。

### 插槽
1.渲染作用域

插槽可以访问和模板其余部分相同的实例Property。但插槽不能访问子组件中的实例。

2.为插槽设置备用内容

```html
<slot>备用内容</slot>
```

3.具名插槽

```v-slot```只能添加在```<template>```上。

4.作用域插槽

如果我们想让插槽内容能够访问子组件中的数据，就需要用到作用域插槽。

1）定义插槽的地方需要修改一下，将数据绑定到插槽中。
```html
<slot :item="item"></slot>
```
2）使用插槽的地方，将数据绑定到插槽上

```html
<template v-slot:default="slotProps">
    <span>{{ slotProps.item }}</span>
</template>
```

如果不带插槽名称的话，会被认定为默认插槽，也可以写成这样```v-slot="slotProps"```

但是当出现多个插槽的时候，请不要使用上面的语法，会导致作用域不明确，请使用完整的基于```<template>```的语法。

5.解构插槽Prop

可以将作用域插槽的参数解构、重命名、赋予默认值等。

6.动态插槽名

插槽名称也支持动态的

7.具名插槽的缩写（v-slot: ==> #）

### Provide / Inject（嵌套较深的组件的数据传递）

无论嵌套多深，父组件可以作为其所有子组件的依赖提供者。

父组件用```provide```提供数据，子组件用```inject```使用数据。

provide：可以是一个对象、一个函数。是和```data```同级的实例属性。

inject：是一个数组,是和```data```同级的实例属性。

#### 处理响应性
默认情况下，provide、inject属性不是响应式的，此时，我们就需要用```ref```和```reactive```包裹对象，来改变这种行为。

也可以采用```Computed```属性的方法。

### 动态组件&异步组件
动态组件之间的切换，Vue都创建一个新的组件实例，不会保留之前的状态。

如果想缓存动态组件的状态，需要用```<keep-alive>```包裹动态组件


#### 异步组件（defineAsyncComponent）

```defineAsyncComponent```接受一个```promise```函数，可返回resolve 或 reject。

也可以返回一个promise。

#### 和Suspense一起使用
可由```<Suspense>```控制组件的加载状态，也可通过设置```suspensible:false```，让组件控制自己的加载状态。
### 模板引用--ref

访问ref可以直接获取模板DOM，可通过```this.$refs.refName```访问。

可向组件添加ref，就可以从父组件触发子组件的事件啦。

注意：```$refs```只有在组件完全渲染完成之后才会生效。

### 处理边界情况

1.强制更新($forceUpdate)：但请尽量不要使用该属性。

2.低级静态组件和v-once

v-once应用场景：包含很多静态内容的组件。

```v-once```：只对其求值一次，然后进行缓存。

Vue中渲染纯HTML元素的速度非常快。

## 过渡&动画
## 可复用&组合

### 组合式API

#### 介绍

要解决的问题：逻辑关注点分离，导致遇到大型组件的时候，我们必须不断地“跳转”相关代码的选项块。

解决方案：利用```setup```将有关系的逻辑放在一起，提取为一个函数，然后引入组件的```setup```中。

#### 组合式API基础

执行时机：组件创建之前执行（```beforeCreate```生命周期之前被执行），一旦```props```被解析，就将作为组合式API的入口。

1.```setup```中无法使用```this```，且无法获取```data```、```computed```、```methods```。

2.```setup```中暴露的属性，一定要用响应式API包裹```ref```、```reactive```、```toRef(props, attributeName)```、```toRefs(props)```

如果想侦听```props```属性的变化，在```setup```中需要使用```toRefs(props)```。

将暴露的属性封装在对象的必要性：保持数据类型行为的统一，如```Number```、```String```等基本类型是值传递的，对象是引用类型，不会在某个地方失去它的响应性。

3.seup内注册的生命周期钩子，会在原生命周期钩子调用之前执行。

生命周期钩子命名：```onMounted```

参数：接受一个箭头回调函数

4.watch属性

```watch(atttibuteName, (newValue, oldValue) => {}, options)```

5.computed属性

接受一个箭头回调函数，访问计算属性时，也需要用```.value```


#### Setup

1.参数props

```setup```中的props是响应式的，props变化时，它也将被更新。由于是响应式的，所以不能在参数中解构。

解构方法：```toRefs(props)```

解构单个可能不存在的参数：```toRef(props, 'title')```

2.参数context

一个普通对象，不是响应式的，可以被解构。

包含```attrs```、```slots```、```emit```、```expose```。

其中```attrs```、```slots```会随着组件更新而更新，因此使用的时候不要解构。

3.结合模板使用

```setup```中返回的对象和```props```参数，可以被模板直接访问。

```setup```中返回的```refs```会被自动浅解析，不要在目标中使用```.value```

4.返回渲染函数

```setup```可以返回一个渲染模板的函数。这时就不能返回其他任何东西了。

```expose```可以将要返回的对象暴露出来，被外部组件实例访问。

#### 生命周期钩子

```setup```中没有```onBeforeCreate```和 ```create```生命周期钩子，如果想在这些钩子中触发事件，请直接在```setup```中触发即可。

#### Provide/Inject

```setup```中提供```inject```和```provide```钩子，但此时需要一个个定义。

增加响应性：先将数据用```ref```或 ```reactive```包裹，然后再传入```provide```和```inject```。

如果需要修改```provide```传递来的值，最好通过```provide```传递进来一个函数。

为了防止子组件修改```provide```传递的值，最好添加 ```readonly```属性。

#### 模板引用ref
如果想在```setup```中访问对DOM节点的引用，需要返回一个```rootName```，该```rootName```被```ref()```包裹。

模板引用ref只会在初始渲染之后（```onMounted```生命周期中）获得赋值。

```ref```属性是响应式的。

在```v-for```中的使用，和单个使用类似。需要```setup```返回一个```ref```数组。但请确保在每次更新之前```onBeforeUpdate```重置ref

侦听模板引用```watchEffect```：注意该方法是在DOM更新之前被调用的，所以该函数运行时，DOM还未更新。如果想访问更新之后的DOM，需要添加属性```flush: 'post'```

### Mixin

#### 基础
可以包含组件的任意选项，```mixin```对象的所有选项都将被混入组件中。

#### 选项合并
当有同名属性的时候，分包按以下几种方式合并：
* data：会合并data，有同名冲突是，组件 > mixin。
* 生命周期钩子函数：合并为数组，都会执行，执行顺序：mixin > 组件。
* 值为对象的选项，如```methods```、```components```、```directives```，被合并为一个对象，有同名冲突是，组件 > mixin。

#### 全局Mixin
直接为```app```添加```mixin```方法。但是要慎用！！！

大多数情况下，全局Mixin只适用于自定义选项（```createApp```时传递的选项）。这时推荐将其作为插件发布。

#### 自定义合并策略

可以修改```app.config.optionMergeStrategies.custom```，来以自定义的逻辑合并。

### 自定义指令
#### 简介
使用场景：对普通DOM进行底部操作。

定义方法有2种：

* ```app.directive('name', {})```
* 在组件中声明```directive```属性

#### 钩子函数
```created``` ==> ```beforeMount``` ==> ```mounted``` ==> ```beforeUpdate``` ==> ```updated``````beforeUnmount``` ==> ```umounted```

接收的参数：```el```（DOM本身）、```binding```（指令绑定的属性）、```vnode```、```prevNode```

#### 动态指令参数
```v-mydirective:[argument]="value"```

在自定义指令中，可以通过```binding```获取到```argument```参数(biding.arg)，和 ```value```(binding.value)

其中```vlaue```也可以是一个对象。

#### 函数简写
如果指令中没有声明周期钩子，则其中的方法，都会在```mounted```和```updated```事件被触发时执行。

#### 在组件中使用
在组件中使用的指令，会被应用在组件的根节点上。

对于有多个根节点的组件，指令会被忽略，并抛出一个警告。

### Teleport
作用：允许我们控制在DOM中哪个父节点下渲染了TMML，而不用求助于全局状态，或将其拆分为两个组件。

```<teleport to="body"></teleport>```：该组件的HTML会被应用到```body```标签。

其中可以嵌套子组件，此时子组件的父组件仍然是```<teleport>```所在的组件。

#### 在同一个目标上使用多个teleport
比如```<Modal>```组件，有多个实例处于活动状态，此时可以将多个```teleport```挂载到同一个目标中，顺序是页面中的追加顺序。

### 渲染函数（用render函数返回页面）

### 插件
应用场景：向Vue添加全局级的功能。

应用举例：```vue-router```；

可以创建指令、全局方法、mixin、添加全局实例方法、提供一个库等。

#### 编写插件
* 对象：会调用install方法
* 函数：函数本身会被调用。
* 参数：```app```、```options```


#### 使用插件
```app.use(pluginName, options)```

## 高阶指南
### Vue与Web Components
HTML中允许创建自定义的dom节点[Web Components MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components)，Vue也提供了类似的功能。

### 响应性
#### 深入响应性原理
1.什么是响应性
* 值被读取时进行追踪
* 值改变时检测
* 重新运行代码读取原来的值

#### 响应性基础
1.声明响应式状态
```reactive```：相当于Vue2.x中的```Vue.observable```API。返回一个响应式的对象状态，是深度转换。

Vue中```data```返回的对象，其实就是将属性交给```reactive()```使其成为响应式对象。

2.创建独立的响应式值作为```refs```

```ref```包裹的值，返回的对象，只包含value属性。

```ref```在模板中使用时，会自欧东被解包，不用手动添加```.value```访问。如果```ref```被包裹在一个对象中，被```setup```返回了，则在模板中访问时，需要添加```.value```属性。

```ref()```创建的值，可以传进```reactive()```中。

```ref()```解包，仅发生在```reactive```的对象中，如果```reactive```中的是数组或Map，```ref```不会被解包，访问时需要手动添加```.value```。

3.响应式状态结构（```reactive```）

```reactive```创建的对象，如果解构的话，会丧失响应性。因此需要先用```toRefs```转换，然后再解构。

4.使用```readonly```防止更高响应式对象

#### 响应式计算和侦听（computed、watchEffect、watch）

### 渲染机制和优化
1.将HTML生成了虚拟DOM。

优点：JS直接操作DOM的计算成本很高，虽然JS执行更新很快，但查找DOM节点并用JS更新他们成本很高，因此虚拟DOM采用批量处理，一次性更改DOM。

### Vue2中的更改检测警告