# Graph QL 学习总结
## 推荐阅读
1.[GraphQL官方文档](https://graphql.cn/learn/)
2.[GraphQL开发原则](https://principles.graphql.cn/)
3.[30分钟了解GraphQL核心概念](https://segmentfault.com/a/1190000014131950)
## 【30分钟了解GraphQL核心概念】学习总结
### 1.什么是GraphQL
* GraphQL是一种查询语言，可由前端通过一张Schema和声明一些Type，拿到我们想要的数据和格式；
* * Type：数据模型的抽象（一个Type有多个Field，每个Field又分别指向某个Type）
* * * Scalar Type（标量类型）：类似JS的基础类型，针对要上传的文件，要用Upload标量，DateTime--日期格式，ID--主键
* * * Object Type（对象类型）：类似对象，也类似TS的interface
* * * Type Modifier（类型修饰符）：类型修饰符可以组合使用
* * * * List：[Type]
* * * * Required：Type!

* * Schema：描述接口获取数据的逻辑，可类似REST中的uri标识符，Schema是多个Query组成的一张表；Query表示数据的查询逻辑，一共有3种查询逻辑：
* * * query（查询）：获取数据时，用query
* * * mutation（更改）：修改数据时，用mutation
* * * subscription（订阅）：应用较少，比较适合实时推送，即数据更改时可进行消息推送；
* * * 注意：在Query查询字段时，是并行执行的；但在Mutation变更的时候，是线性执行，一个接着一个；

* * Resolver（解析函数）
* * * Qery和对应的Resolver是同名的，这样GraphQL才能把他们对应起来；
```javascript
/*
* @param parent：当前上一个Resolver返回的值
* @param args：传入某个Query中的函数
* @param ctx：在Resolver解析链中，不断传递的中间变量（类似context）
* @param info：当前Query的AST对象
*/
function(parent, args, ctx, info) {
    ...
}
```
* * * Resolve实际使用：可将Resolver作为一个中间层，最终的数据获取是由Resolver封装，但实际数据获取的实现可以基于REST\RPC\SQL等；

* * GraphQL的内部工作机制
* * * 解析Query的类型、名字A
* * * 获取对应名字A的Resolver（解析函数），获取解析数据data
* * * 对解析数据data，继续解析；标量类型即可解析结束，对象类型则继续解析；
* * * 整个过程是递归的，直到解析Field的类型是标量类型；

* * GraphQL的请求，如何发送？
* * * 最常见的是通过HTTP发送请求，比如针对如下的请求，如何执行GraphQL查询呢？
```javascript
query {
    me {
        name
    }
}

//  Get 方式
// http://myapi/graphql?query={me{name}}

// # Post 方式的请求体
{
  "query": "...",
  "operationName": "...",
  "variables": { "myVariable": "someValue", ... }
}
```
