GraphQL的快速使用
---

今天介绍的是一门查询语言，`GraphQL`很多人不是很了解，那么先了解一下它的历史

---
GraphQL 名为但不仅仅是一种查询语言。它是对连接现代应用程序到云服务这一问题的全面解决方案。因此，它构成了现代应用程序开发堆栈中一个新的重要层的基础：数据图。这个新的层将公司的所有应用程序数据和服务集中在同一个地方，具有一致、安全且易于使用的界面，因此任何人都能够通过最小的磨合利用它。

在 Apollo，我们自 2015 年以来一直在构建业界领先的数据图技术，且我们估计我们的软件现已用于超过 90％ 的 GraphQL 实现。多年来，我们与来自不同规模公司的开发人员进行了关于实现 GraphQL 的数千次对话。我们希望分享我们所学到的知识，因此我们将他们的经验提炼成一套创建、维护和操作数据图的最佳实践。在这里我们将它们呈现为 10 大 GraphQL 原则，分为三类，其格式灵感来自于 Twelve Factor App。

---

那么他的优势呢？[GraphQL的优势](https://graphql.cn/)

了解优势后还要熟知使用开发原则[了解更多开发原则](https://principles.graphql.cn/)

关于查询和变更包含以下知识点：

+ 字段（Fields）
+ 参数（Arguments）
+ 别名（Aliases）
+ 片段（Fragments）
+ 操作名称（Operation Name）
+ 变量（Variables）
+ 指令（Directives）
+ 变更（Mutations）
+ 内联片段（Inline Fragments）

那么了解了GraphQL是前端查询语言，那么对应的后端解析就需要使用Schema了，关于Schema的知识点如下：

+ 类型系统（Type System）
+ 类型语言（Type Language）
+ 对象类型和字段（Object Types and Fields）
+ 参数（Arguments）
+ 查询和变更类型（The Query and Mutation Types）
+ 标量类型（Scalar Types）
+ 枚举类型（Enumeration Types）
+ 列表和非空（Lists and Non-Null）
+ 接口（Interfaces）
+ 联合类型（Union Types）
+ 输入类型（Input Types）

那么本文介绍的前端应用，目前大多数前端应用都是前后分离的，直接引入到前端工程去执行查询还需要集成很多，`工具类、fetch、响应式的触发封装、订阅和变更等`，直接使用或者操作GraphQL在
实际应用中比较费力。那么有没有辅助工具？这时候**Apollo**可以帮助你，如果使用的`vue`或者`react`都有对应的中间件

今天以一个第三方中间件为例`Vue Apollo`，相对于`React Apollo`名气还是比较小的，但是实际使用还是比较重要的。

在说之前，先打开链接了解一下[Vue Apollo](https://vue-apollo.netlify.app/zh-cn/guide/)的使用方法。

具体使用方法和快速接入请参考以上文档，只要总体了解基本用法即可

+ 在 Vue 组件中的用法
+ 查询
+ 变更
+ 订阅
+ 使用 fetchMore 实现分页
+ 特殊选项



