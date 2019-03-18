# 最后的一些想法

本书的最后几章希望能鼓励你去应用你所学到的. 到目前为止, 本书已经教了你如何在 JavaScript 的客户端和服务端中使用 GraphQL. 你可以在客户端用 React, 在服务端用 Express. 你也可以在两端用 Apollo 作为复杂的 GraphQL 库. 但是, 这个生态还有很多待开发的地方. 如果你还没完成本书的所有练习, 读完本书所有的内容, 你应该先从它们开始. 此外, 我们看看还可以做什么来提升 GraphQL 技能.

## 下一步学习路径

你已经构建几个书里的应用程序. 当你读那些章节的时候, 你在程序上运用高级技术(基于权限的论证),工具(GraphQL Playground) 或特性(GitHub commenting). 甚至你可能紧紧地跟上了书上的所有练习. 但还没完. 还有许多特性, 技术, 和工具是你可以运用在那些应用上的. 有创造性并且挑战自我去实现它们. 我很期待看到你能想出什么, 所以新创意随时可以告诉我.

我们在客户端用 React,在服务端用 Express. 因为 GraphQL 和 Apollo 库都跨框架的, 你可以把它们和任何其它解决方案结合起来. 如果你熟悉 Angelar 或者 Vue, 你可以把你所学的 GraphQL 和 Apollo 知识, 迁移到用这些框架构建客户端程序. 如果你对在服务端用 Koa 或 Hapi 更感兴趣, 你可以用这些中间件替代 Express 来做之前你在书中所做的. GraphQL 服务程序所使用的数据库也同样适用. 书中, 你用 PostgreSQL 和 Sequelize ORM 去把你的 GraphQL 解析器连接到 PostgreSQL 数据库上. 取决于你要不要用其它的比如 MongoDB 和 Mongoose 来替代 PostgreSQL 和 Sequelize. 同样, Apollo 客户端和服务端的库也可以用生态里的相应的其它库来替代. 你可以在本书的简介的章节里阅读它们. 你也可以用[Yoga GraphQL](https://github.com/prisma/graphql-yoga) 和 [Prisma](https://www.prisma.io/) 来启动和运行你的 GraphQL 服务器. 本书里用到的所有技术都不是一成不变的, 你可以替换它们来用 JavaScript 构建 GraphQL 应用程序. 
 
我最大的建议是继续使用你在本书中所使用的 Apollo Server. 因为它是一个理想的入门工具来实现你自己的想法. 正如我之前提到的, 你可以自由地替换封装(在引擎盖下)的技术,但最好根据你的应用特性有选择地聚焦. 既然用户管理已经帮你实现了, 你就可以开始添加你自己的特性了.

