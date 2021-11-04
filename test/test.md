# Vue Test Utils
## 1.安装
### 1.1 在Jest中处理单文件组件

支持vue文件的处理：moduleFileExtensions
@别名：moduleNameMapper
处理高版本js：transform属性 && babel-jest
测试覆盖率配置：collectCoverage \ collectCoverageFrom

### 1.2 Jest的测试套件
```describe``` + ```test```
 ```describe``` + ```it```

## 2.教程
### 2.1 起步
mount挂载组件：返回一个包裹器——```wrapper```，```wrapper.vm``` 可访问实际的vue示例

对DOM的操作，应该在单元之前 await nextTick函数；
Vue.nextTick中的错误，不会被测试运行期捕获。解决方法如下：
1.测试一开始将Vue的全局错误处理器，设置为done回调
2.在调用nextTick时，不带参数，让其作为一个promise返回

事件测试分三步：
* 获取wrapper
* 触发事件
* 判断结果是否符合预期

### 2.2 常用技巧
浅渲染--shallowMount
* 针对有多个子组件的父组件，shallowMount可以只挂载一个组件，而不渲染其他子组件

生命周期钩子
* mount和shallowMount 可以响应所有的生命周期事件；但destroy事件，需要手动调用 Wrapper.destroy()

涉及DOM变更的地方
* 建议使用await 
* 或 then链式回调

断言触发的事件：可通过wrapper.emitted()方法，取回事件记录；
父组件也可以通过 ```wrapper.vm.$emit('customMethod')```手动触发 子组件上的事件

操作组件的状态：
```javascript
it('manipulates state', async () => {
  await wrapper.setData({ count: 10 })

  await wrapper.setProps({ foo: 'bar' })
})
```

如何测试 transition中被包裹的内容？
* 使用```transitionStub```：通过重写transition组件的默认行为，并在条件发生变化时，立即呈现子元素；
* 或在 mount初始化时，僧面props的值

应用全局的插件和mixins？？？？

仿造注入：比如仿造路由上的参数/path?id=123&name=hxf
此时可以在mount时，声明mocks属性，将$route参数挂载上去


路由测试：最好集成到端到端的测试


## 3.API
## 4.Wrapper
## 5.WrapperArry
## 6.挂载选项
## 7.组件
```RouterLinkStub```组件：用来模拟router-link，可以在stubs中声明它，然后你就可以找到文件中的router-link组件啦