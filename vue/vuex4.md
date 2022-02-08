[4.x官方文档](https://next.vuex.vuejs.org/zh/#%E4%BB%80%E4%B9%88%E6%98%AF%E2%80%9C%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86%E6%A8%A1%E5%BC%8F%E2%80%9D%EF%BC%9F)

# Vuex状态管理
Vue Components --> dispatch(Actions) --> commit(Mutaions) --> Mutate(State)

修改state：通过提交mutation的方式，而非直接改变store.state，因为这样可以让我们明确追踪到状态的变化。

## 核心概念呢
### State
Vuex是单一状态树。

Vuex的状态存储是响应式的，因此从store中读取实例最简单的方法是在计算属性中返回某个状态。

实现：利用Vue插件机制，将store实例，从根组件注入到所有的子组件里，每个子组件都能通过this.$store访问到。


```mapState()```辅助函数：获取state的多个状态；参数：数组、对象，返回一个对象

对象展开运算符：将需要的store属性，合并到computed属性中。

### Getter

问题：从store中的state派生出一些新的状态

解决方法：在store中定义```getter```（可以认为是store的计算属性）。

```createStore({ state: {}, getters: {}})```时，```getters```也可以作为对象的一个属性传入，可通过```store.getters.getName```访问。

```getters```：接收```state```、```getters```

```mapGetters```辅助函数：将store中的getter映射到计算属性

### Mutation

定义：
```js
createStore({ state: {}, mutations: {}, getters: {} })
```

使用：不能直接调用，通过```commit```提交一个mutation。此处更像是一个事件注册。

传参payload：参数应该是一个对象。

提交方式：
* 提交payload
* 对象风格的提交方式（传递type、数据）

声明MUTATION事件常量：类似Redux的Type，方便多人协作。

Mutation必须是同步函数。

在组件中提交Mutation：
* ``` this.$store.commit('xxx')```
* ```mapMutations()```：将commit的提交，映射为组件的方法，调用组件方法，就可提交对应的commit

### Action
问题：Mutation只能是同步调用，那如果遇到异步调用，我们应该怎么处理呢？

解决方法：```Action```

```Action``` vs ```Mutation```：Action提交的是Mutation。Action可以包含异步操作。

接收的参数：```context```对象，和store实例有相同的方法和属性，但不是store。context包含commit、getters、state等属性。

#### 分发Action

```store.dispatch```：Action为什么不直接分发mutation呢？因为Mutation只支持同步操作，但Action内部支持异步操作。

路径：定义mutation --> 定义Action --> Action中commit 一个mutation --> 发出异步请求 --> 再commit一个Mutation。

此处：mutation用来记录action产生的副作用，类似Redux。

#### 在组件中分发Action
```this.$store.dispatch('xxx')```

```mapActions```辅助函数，将组件的methods映射为```store.dispatch```。使用时需要先在根节点注入store。

#### 组合多个Action

```store.dipsatch```返回的是```Promise```，因此可以直接用```.then```后续处理调用，也可以用```async/await```，组合action。

### Module

Vuex允许将store分割成模块，注册模块：
```js
createStore({modules: {}})
```

在模块中访问全局state，用 ```rootState```。

默认情况下，模块内部的action和 mutation是注册在全局命名空间的，因此不要在不同的、无命名空间的模块中定义两个getter导致错误。

可通过```namespaced```使模块成为有命名空间的模块。

* 在带命名空间的模块内访问全局内容：```rootState```、```rootGetters```，若需要在全局空间内分发action，、提交commit，可使用```{root: true}```


* 在带命名空间的模块注册全局action：声明```root: true```

* 带命名空间的绑定函数：
    * ```mapState(nameSpace, {})``` 
    * ```createNamespacedHelpers```

* 模块动态注册```store.registerModule```
    * 也可注册嵌套模块，传递一个数组['nested', 'module']

* 模块卸载```store.unregisterModule```

* 模块查询```store.hasModule```

如库```vuex-router-sync```插件：通过动态注册模块将Vuex和Router结合在一起，实现应用的路由状态管理。

* 保留state：```preserverState: true```

* 模块重用：如多个store中公用同一个模块，或 在一个store中多次注册同一个模块。
    * 对象形式会

## 进阶
## API
