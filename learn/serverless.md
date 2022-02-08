# 记录Serverless相关知识

* [腾讯云 serveless framework](https://cloud.tencent.com/document/product/1154/40217)
* [Serveless SSR 一篇就够了](https://mp.weixin.qq.com/s/xcxoCUzzruVSm15TLLOs-Q)
    * 为啥要SSR
        * SEO、 白屏
    * 为HTML文件，添加适当的meta标签，表明标题、内容等
    * 浏览器渲染页面的几种方式
        * CSR —> CSR Prerender —> SSR withhydration —> static SSR  —> Server Rendering
* Serveless是什么
    * Faas：Function as a service
    * Baas：Backend as a service
    * 缺点：
        * 需要冷启动时间，用完即休眠，因此不适用于长时间运行的服务
        * 缺乏适当的调试工具
        * 完全依赖于第三方服务：一旦选择某个云服务提供商，再想换就比较麻烦了。
        * 构建很复杂
    * 实际应用：Lambda的CloudFormation
* serverless
    * 页面渲染的发展历程
        * 模板渲染、ajax渲染、node.js的前端开发、node.js的全栈开发—>serverless
    * 概念：
        * 构建和运行不需要服务器管理的应用程序
        * serverless = Faas（运行函数的平台）+ Baas（提供后端云服务，如对象存储、云数据库、消息队列等。）
    * 框架：
        * 框架不统一，不便于迁移
    * 基于serverless的开发模式
        * 不用再维护BFF层（接口集成层）
    * serverless的优点
        * 不用关心后端的实现，开发更便捷
        * 服务端渲染
    * serverless如何测试？
        * 单元测试>继承测试>UI测试
        * 要多做单元测试；测试时，将函数中的逻辑和业务分开，便于测试
    * 如何提高serverless的性能？
        * 下载代码、开辟空间、启动环境，渲染模板
        * 冷启动（会保存一段时间）、热启动
        * 内存满了后，会自动开辟一块新的空间