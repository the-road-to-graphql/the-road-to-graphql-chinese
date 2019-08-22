> # Node.js with GraphQL and Apollo Server

# 在 GraphQL 和 Apollo 服务端中使用 Node.js 
> In this chapter, you will implement server-side architecture using GraphQL and Apollo Server. The GraphQL query language is implemented as a reference implementation in JavaScript by Facebook, while Apollo Server builds on it to simplify building GraphQL servers in JavaScript. Since GraphQL is a query language, its transport layer and data format is not set in stone. GraphQL isn't opinionated about it, but it is used as alternative to the popular REST architecture for client-server communication over HTTP with JSON.

在本章你将会使用 GraphQL 和 Apollo 服务端构建服务端。FaceBook 用 JavaScript 实现了 GraphQL 查询语言的参考实现，而 Apollo 服务端基于此实现，以便用 JavaScript 构建 GraphQL 服务端变得更简单。由于 GraphQL 是一种查询语言，所以它的传输层和数据格式不是固定的。在通过 HTTP 与 JSON 的客户-服务端传输中，它常常被视为流行的 REST 架构的替代品，虽然 GraphQL 没有刻意提到这一点。
> In the end, you should have a fully working GraphQL server boilerplate project that implements authentication, authorization, a data access layer with a database, domain specific entities such as users and messages, different pagination strategies, and real-time abilities due to subscriptions. You can find a working solution of it, as well as a working client-side application in React, in this GitHub repository: [Full-stack Apollo with React and Express Boilerplate Project](https://github.com/the-road-to-graphql/fullstack-apollo-express-postgresql-boilerplate). I consider it an ideal starter project to realize your own idea.

在章节的最后，你能拥有一个完全工作的 GraphQL 服务端脚手架项目，这个项目包括身份验证，授权，连接着数据库的数据访问层，像用户、消息这样的具体的领域实体，不同的分页策略和由订阅带来的实时推送功能。你可以在这个 GitHub 代码库：[使用 React 和 Express 的全栈 Apollo 脚手架项目](https://github.com/the-road-to-graphql/fullstack-apollo-express-postgresql-boilerplate)找到一个已经实现了这些功能的服务端解决方案和一个由 React 实现的客户端解决方案。我认为这是一个理想的实现自己想法的入门项目。
> While building this application with me in the following sections, I recommend to verify your implementations with the built-in GraphQL client application (e.g. GraphQL Playground). Once you have your database setup done, you can verify your stored data over there as well. In addition, if you feel comfortable with it, you can implement a client application (in React or something else) which consumes the GraphQL API of this server. So let's get started!

当你在和我一起构建这个应用程序时，推荐使用内置的 GraphQL 客户端应用程序（比如 GraphQL Playground）来验证你的实现。完成数据库设置的话，也可以在那里验证存储数据。此外，如果你感觉还有余力的话，还可以实现一个使用这个服务提供的 GraphQL API 的客户端应用程序（使用 React 或者其他什么）。那让我们开始吧！
> ## Apollo Server Setup with Express

## 使用 Express 设置 Apollo 服务端
> There are two ways to start out with this application. You can follow my guidance in [this minimal Node.js setup guide step by step](https://www.robinwieruch.de/minimal-node-js-babel-setup) or you can find a starter project in this [GitHub repository](https://github.com/rwieruch/node-babel-server) and follow its installation instructions.

有两种方法来开始这个应用程序。你可以按照我在这个[手把手教你启动最小 Node.js 的指南](https://www.robinwieruch.de/minimal-node-js-babel-setup)里的指导，或者在这个 [GitHub 代码库](https://github.com/rwieruch/node-babel-server)里找到一个启动项目，并根据它的安装说明进行操作。
> Apollo Server can be used with several popular libraries for Node.js like Express, Koa, Hapi. It is kept library agnostic, so it's possible to connect it with many different third-party libraries in client and server applications. In this application, you will use [Express](https://expressjs.com/), because it is the most popular and common middleware library for Node.js.

Apollo 服务端可以与 Express、Koa、Hapi 等常用的 Node.js 库一起使用。因为它与库无关，所以可以将它与客户端和服务器应用程序的许多不同第三方库结合使用。在这个应用程序中，你将使用 [Express](https://expressjs.com/)，因为它是 Node.js 最流行和最常见的中间件库。
> Install these two dependencies to the *package.json* file and *node_modules* folder:

将这两个依赖包安装到 *package.json* 文件和 *node_modules* 文件夹中

{title="Command Line",lang="json"}
~~~~~~~~
npm install apollo-server apollo-server-express --save
~~~~~~~~
> As you can see by the library names, you can use any other middleware solution (e.g. Koa, Hapi) to complement your standalone Apollo Server. Apart from these libraries for Apollo Server, you need the core libraries for Express and GraphQL:

从库名可以看出，你可以使用任何其他中间件解决方案（例如 Koa、Hapi）来填充你自己的 Apollo 服务端。除了用于 Apollo 服务端的这些库之外，你还需要 Express 和 GraphQL:

{title="Command Line",lang="json"}
~~~~~~~~
npm install express graphql --save
~~~~~~~~
> Now every library is set to get started with the source code in the *src/index.js* file. First, you have to import the necessary parts for getting started with Apollo Server in Express:

现在，每个库都设置为从 *src/index.js* 文件中的源代码开始。为了启动使用 Express 的 Apollo 服务端，首先，你需要导入必要的模块：

{title="src/index.js",lang="javascript"}
~~~~~~~~
import express from 'express';
import { ApolloServer } from 'apollo-server-express';
~~~~~~~~
> Second, use both imports for initializing your Apollo Server with Express:

然后，使用这两个导入的模块通过 Express 来初始化 Apollo 服务端。

{title="src/index.js",lang="javascript"}
~~~~~~~~
import express from 'express';
import { ApolloServer } from 'apollo-server-express';

# leanpub-start-insert
const app = express();

const schema = ...
const resolvers = ...

const server = new ApolloServer({
  typeDefs: schema,
  resolvers,
});

server.applyMiddleware({ app, path: '/graphql' });

app.listen({ port: 8000 }, () => {
  console.log('Apollo Server on http://localhost:8000/graphql');
});
# leanpub-end-insert
~~~~~~~~
> Using Apollo Server's `applyMiddleware()` method, you can opt-in any middleware, which in this case is Express. Also, you can specify the path for your GraphQL API endpoint. Beyond this, you can see how the Express application gets initialized. The only missing items are the definition for the schema and resolvers for creating the Apollo Server instance. We'll implement them first and learn about them after:

使用 Apollo 服务端的 `applyMiddleware()` 方法，你可以添加任意中间件，这里使用了 Express。此外，还可以指定 GraphQL API 端点的路径。而且，你还可以自定义初始化 Express 应用程序。现在惟一缺少的是用于创建 Apollo 服务端实例的 schema 和 resolvers 的定义。我们将先实现它们，之后再来学习它们：

{title="src/index.js",lang="javascript"}
~~~~~~~~
import express from 'express';
# leanpub-start-insert
import { ApolloServer, gql } from 'apollo-server-express';
# leanpub-end-insert

const app = express();

# leanpub-start-insert
const schema = gql`
  type Query {
    me: User
  }

  type User {
    username: String!
  }
`;

const resolvers = {
  Query: {
    me: () => {
      return {
        username: 'Robin Wieruch',
      };
    },
  },
};
# leanpub-end-insert

...
~~~~~~~~
> The **GraphQL schema** provided to the Apollo Server is all the available data for reading and writing data via GraphQL. It can happen from any client who consumes the GraphQL API. The schema consists of **type definitions**, starting with a mandatory top level **Query type** for reading data, followed by **fields** and **nested fields**. In the schema from the Apollo Server setup, you have defined a `me` field, which is of the **object type** `User`. In this case, a User type has only a `username` field, a **scalar type**. There are various scalar types in the GraphQL specification for defining strings (String), booleans (Boolean), integers (Int), and more. At some point, the schema has to end at its leaf nodes with scalar types to resolve everything properly. Think about it as similar to a JavaScript object with objects or arrays inside, except it requires primitives like strings, booleans, or integers at some point.

**GraphQL 模式** 是 Apollo 服务端提供出的所有，能通过 GraphQL 读写的可用数据。它可以提供给任何消费这个 GraphQL API 的客户端。模式 由**类型定义**组成，首先在顶级必须有用于读取数据的 **查询类型**，然后是**字段**和**嵌套字段**。在这个准备设置给 Apollo 服务端的模式中，定义了一个 `me` 字段，它的类型是**对象类型** `User`。在这里，User 类型只有一个 `username` 字段， 它是**标量**类型的。 GraphQL 规范中有各种标量类型，用于定义字符串（String）、布尔值（Boolean）、整数（Int）等。一般情况下， 模式的叶子节点必须以标量类型结束，以便正确地解析所有内容。可以将其类比为 JavaScript 对象，其中包含对象、数组，此外也包括字符串、布尔值和整数等基本类型。

{title="Code Playground",lang="javascript"}
~~~~~~~~
const data = {
  me: {
    username: 'Robin Wieruch',
  },
};
~~~~~~~~
> In the GraphQL schema for setting up an Apollo Server, **resolvers** are used to return data for fields from the schema. The data source doesn't matter, because the data can be hardcoded, can come from a database, or from another (RESTful) API endpoint. You will learn more about potential data sources later. For now, it only matters that the resolvers are agnostic according to where the data comes from, which separates GraphQL from your typical database query language. Resolvers are functions that resolve data for your GraphQL fields in the schema. In the previous example, only a user object with the username "Robin Wieruch" gets resolved from the `me` field.

在这个为了设置 Apollo 服务端而准备的 GraphQL 的模式中，**resolvers** 可以设定返回模式的字段组成的数据。数据源并不重要，数据可以是硬编码的，可以是来自数据库的，也可以是来自其他 （RESTful） API 端点的。你将会在之后学习这些潜在的数据来源。现在只需要知道 resolvers 的数据与数据源无关，这是 GraphQL 和典型的数据库查询语言不同的地方。 Resolvers 是模式中解析 GraphQL 字段的函数。上面的例子中，只有一个 me 字段含有一个 username 为 "Robin Wieruch" 的 User 对象 。
> Your GraphQL API with Apollo Server and Express should be working now. On the command line, you can always start your application with the `npm start` script to verify it works after you make changes. To verify it without a client application, Apollo Server comes with GraphQL Playground, a built-in client for consuming GraphQL APIs. It is found by using a GraphQL API endpoint in a browser at `http://localhost:8000/graphql`. In the application, define your first GraphQL query to see its result:

现在，你通过使用 Express 设置的 Apollo 服务端实现的 GraphQL API 应该已经可以运行了。在命令行中，你可以使用 “npm start” 脚本启动应用程序，以便在你进行更改后验证它是否工作。为了在没有客户端应用程序的情况下验证它，Apollo 服务端附带了 GraphQL Playground，这是一个用于消费 GraphQL API 的内置客户端。可以通过在浏览器访问 http://localhost:8000/graphql 使用 GraphQL API 端点。在应用中定义你的一个 GraphQL 的 查询然后查看结果：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  me {
    username
  }
}
~~~~~~~~
> The result for the query should this or your defined sample data:

查询的结果应该是这个，也可以是你自定义的其他示例数据：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "data": {
    "me": {
      "username": "Robin Wieruch"
    }
  }
}
~~~~~~~~
> I might not mention GraphQL Playground as much moving forward, but I leave it to you to verify your GraphQL API with it after you make changes. It is useful tool to experiment and explore your own API. Optionally, you can also add [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) to your Express middleware. First, install CORS on the command line:

我可能不会过多地提到 GraphQL Playground，但希望你在进行更改之后用它来验证一下你的 GraphQL API。它是一个有用的工具，可以用来试验和探索你的 API。你还可以选择将 [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) 添加到你的 Express 中间件中。

{title="Command Line",lang="json"}
~~~~~~~~
npm install cors --save
~~~~~~~~

> Second, use it in your Express middleware:

然后在你的 Express 中间件中使用它。

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import cors from 'cors';
# leanpub-end-insert
import express from 'express';
import { ApolloServer, gql } from 'apollo-server-express';

const app = express();

# leanpub-start-insert
app.use(cors());
# leanpub-end-insert

...
~~~~~~~~
> CORS is needed to perform HTTP requests from another domain than your server domain to your server. Otherwise you may run into cross-origin resource sharing errors for your GraphQL server.

服务器的 HTTP 跨域请求需要用 CORS 来处理。否则，你的 GraphQL 服务端可能会遇到跨域错误。
> ### Exercises:

### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/b6468a84ad77018bf940d951016b7e2c1e07404f)
* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/b6468a84ad77018bf940d951016b7e2c1e07404f)
> * Read more about [GraphQL](https://graphql.org/learn)
* 延伸阅读：[GraphQL](https://graphql.org/learn)
> * Experiment with the schema and the resolver
* 尝试完善 schema 和 resolver
>   * Add more fields to the user type
  * 为 user 类型添加更多字段
>   * Fulfill the requirements in the resolver
  * 完成 resolver 中的这些字段的解析
>  * Query your fields in the GraphQL Playground
  * 在 GraphQL Playground 中查询出你的字段
> * Read more about [Apollo Server Standalone](https://www.apollographql.com/docs/apollo-server/v2/getting-started.html)
* 延伸阅读：[独立的 Apollo Server](https://www.apollographql.com/docs/apollo-server/v2/getting-started.html)
> * Read more about [Apollo Server in Express Setup](https://www.apollographql.com/docs/apollo-server/v2/essentials/server.html)
* 延伸阅读：[在 Express 中设置 Apollo Server](https://www.apollographql.com/docs/apollo-server/v2/essentials/server.html)
> ## Apollo Server: Type Definitions

## Apollo 服务端： 类型定义
> This section is all about GraphQL type definitions and how they are used to define the overall GraphQL schema. A GraphQL schema is defined by its types, the relationships between the types, and their structure. Therefore GraphQL uses a **Schema Definition Language (SDL)**. However, the schema doesn't define where the data comes from. This responsibility is handled by resolvers outside of the SDL. When you used Apollo Server before, you used a User object type within the schema and defined a resolver which returned a user for the corresponding `me` field.

本节关于 GraphQL 的类型定义以及如何使用它们定义 GraphQL 模式。一个 GraphQL 模式由其类型、类型之间的关系及其结构定义。为此，GraphQL 使用了一种**模式定义语言（SDL）**。不过，模式并不定义数据来自何处。此职责由 SDL 之外的解析器处理。在使用 Apollo 服务端之前，你在模式中使用了一个 User 对象类型，并定义了一个 resolver，该 resolver 为相应的 `me` 字段返回一个 user。
> Note the exclamation point for the `username` field in the User object type. It means that the `username` is a **non-nullable** field. Whenever a field of type User with a `username` is returned from the GraphQL schema, the user has to have a `username`. It cannot be undefined or null. However, there isn't an exclamation point for the user type on the `me` field. Does it mean that the result of the `me` field can be null? That is the case for this particular scenario. There shouldn't be always a user returned for the `me` field, because a server has to know what the field contains before it can respond. Later, you will implement an authentication mechanism (sign up, sign in, sign out) with your GraphQL server. The `me` field is populated with a user object like account details only when a user is authenticated with the server. Otherwise, it remains null. When you define GraphQL type definitions, there must be conscious decisions about the types, relationships, structure and (non-null) fields.

注意 User 对象类型中的 `username` 字段的感叹号。意思是这个 `username` 字段是一个**不能为空**的字段。每当从 GraphQL 模式返回拥有 `username` 的 User 类型时， `username` 字段必须有值，不能是 undefined 或者 null。不过，在 `me` 字段中， User 类型没有感叹号，是否这意味着返回结果中的 `me` 字段可以为空？这是一种特殊的情况。因为服务器必须知道该字段包含什么才能响应，所以 `me` 字段不应该必然有 user 返回。稍后，你将使用 GraphQL 服务端实现身份验证机制(注册、登录、退出)。只有当用户通过服务器进行身份验证时， `me` 字段才会填充 User 对象，比如帐户详细信息。否则，它仍然为空。在定义 GraphQL 类型定义时，必须对类型、关系、结构和(非 null)字段进行特意的设计。
> We extend the schema by extending or adding more type definitions to it, and use **GraphQL arguments** to handle user fields:

我们通过扩展或添加更多类型定义来扩展 schema，并使用 **GraphQL参数** 来处理 user 字段：

{title="src/index.js",lang="javascript"}
~~~~~~~~
const schema = gql`
  type Query {
    me: User
# leanpub-start-insert
    user(id: ID!): User
# leanpub-end-insert
  }

  type User {
    username: String!
  }
`;
~~~~~~~~

> **GraphQL arguments** can be used to make more fine-grained queries because you can provide them to the GraphQL query. Arguments can be used on a per-field level with parentheses. You must also define the type, which in this case is a non-nullable identifier to retrieve a user from a data source. The query returns the User type, which can be null because a user entity might not be found in the data source when providing a non identifiable `id` for it. Now you can see how two queries share the same GraphQL type, so when adding fields to the it, a client can use them implicitly for both queries `id` field:

GraphQL 参数可用于进行更细粒度的查询，因为可以将它们提供给 GraphQL 查询。参数可以在带有括号的每个字段级上使用。你还必须定义类型，在这个例子中，该类型是一个非可空标识符，用于从数据源那里获取 user。 查询返回的 User 类型可以为空，因为在为其提供非法的 `id` 时，可能在数据源中找不到一个 user 实体。现在你可以看到两个查询共享了相同的 GraphQL 类型，因此在向其添加字段时，客户端还可以隐式地使用两个查询的 `id` 字段：

{title="src/index.js",lang="javascript"}
~~~~~~~~
const schema = gql`
  type Query {
    me: User
    user(id: ID!): User
  }

  type User {
# leanpub-start-insert
    id: ID!
# leanpub-end-insert
    username: String!
  }
`;
~~~~~~~~
> You may be wondering about the ID scalar type. The ID denotes an identifier used internally for advanced features like caching or refetching. It is a superior string scalar type. All that's missing from the new GraphQL query is the resolver, so we'll add it to the map of resolvers with sample data:

你可能想知道 ID 这个标量类型。ID 是表示用于诸如缓存或者获取的内部高级特性标识符。它是继承自标量类型 string 的标量类型。新的 GraphQL 查询缺少的只是 resolver，所以我们将为示例数据添加一个叫做 resolvers 的映射:

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  Query: {
    me: () => {
      return {
        username: 'Robin Wieruch',
      };
    },
# leanpub-start-insert
    user: () => {
      return {
        username: 'Dave Davids',
      };
    },
# leanpub-end-insert
  },
};
~~~~~~~~
> Second, make use of the incoming `id` argument from the GraphQL query to decide which user to return. All the arguments can be found in the second argument in the resolver function's signature:

其次，使用来自 GraphQL 查询的传入 `id` 参数来决定返回哪个 user。所有的参数都可以在 resolver 的函数的第二个参数中找到:

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  Query: {
    me: () => {
      return {
        username: 'Robin Wieruch',
      };
    },
# leanpub-start-insert
    user: (parent, args) => {
# leanpub-end-insert
      return {
        username: 'Dave Davids',
      };
    },
  },
};
~~~~~~~~
> The first argument is called `parent` as well, but you shouldn't worry about it for now. Later, it will be showcased where it can be used in your resolvers. Now, to make the example more realistic, extract a map of sample users and return a user based on the `id` used as a key in the extracted map:

第一个参数也称为 `parent`，不过现在你不用担心它。稍后，将展示在 resolvers 中如何使它。现在，为了使示例更加生动，我们创建一个叫做 users 的映射，并根据映射中的键 `id` 返回一个 user：

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
let users = {
  1: {
    id: '1',
    username: 'Robin Wieruch',
  },
  2: {
    id: '2',
    username: 'Dave Davids',
  },
};
# leanpub-end-insert

# leanpub-start-insert
const me = users[1];
# leanpub-end-insert

const resolvers = {
  Query: {
# leanpub-start-insert
    user: (parent, { id }) => {
      return users[id];
# leanpub-end-insert
    },
    me: () => {
# leanpub-start-insert
      return me;
# leanpub-end-insert
    },
  },
};
~~~~~~~~
> Now try out your queries in GraphQL Playground:

现在试着在在 GraphQL Playground 验证一下你的查询结果：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  user(id: "2") {
    username
  }
  me {
    username
  }
}
~~~~~~~~
> It should return this result:

应该拿到这样的结果：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "data": {
    "user": {
      "username": "Dave Davids"
    },
    "me": {
      "username": "Robin Wieruch"
    }
  }
}
~~~~~~~~
> Querying a list of of users will be our third query. First, add the query to the schema again:

查询 users 列表将是我们的第三个查询。首先，再次将 query 添加到 schema 中:

{title="src/index.js",lang="javascript"}
~~~~~~~~
const schema = gql`
  type Query {
# leanpub-start-insert
    users: [User!]
# leanpub-end-insert
    user(id: ID!): User
    me: User
  }

  type User {
    id: ID!
    username: String!
  }
`;
~~~~~~~~
> In this case, the `users` field returns a list of users of type User, which is denoted with the square brackets. Within the list, no user is allowed to be null, but the list itself can be null in case there are no users (otherwise, it could be also `[User!]!`). Once you add a new query to your schema, you are obligated to define it in your resolvers within the Query object:

在本例中，`users` 字段返回一个 User 类型的列表，该列表用方括号表示。在列表中，不允许任何 user 为空，但是如果没有 user，列表本身可以为空（否则，它应该是  `[User!]!` 的）。一旦你在你的模式中添加了一个新的查询，你必须在 Query 对象的解析器中定义它:

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  Query: {
# leanpub-start-insert
    users: () => {
      return Object.values(users);
    },
# leanpub-end-insert
    user: (parent, { id }) => {
      return users[id];
    },
    me: () => {
      return me;
    },
  },
};
~~~~~~~~
> You have three queries that can be used in your GraphQL client (e.g. GraphQL Playground) applications. All of them operate on the same User type to fulfil the data requirements in the resolvers, so each query has to have a matching resolver. All queries are grouped under one unique, mandatory Query type, which lists all available GraphQL queries exposed to your clients as your GraphQL API for reading data. Later, you will learn about the Mutation type, for grouping a GraphQL API for writing data.

现在在GraphQL client（例如 GraphQL Playground）应用程序中，你有三个查询可以使用。它们每个都对相同的 User 类型进行操作，为了满足解析器中的数据需求，每个查询都必须有一个对应的解析器。所有查询都分组在一个惟一的、强制的 Query 类型下，它列出所有公开给客户端的可用 GraphQL 查询，作为读取数据的 GraphQL API。稍后，你将会学到为 GraphQL API 编组写入数据的 Mutation 类型。
> ### Exercises:
 ### 练习：
> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/469080f810a0049442f02393fae746cebc391cc0)
* 完成你的[最后一部分源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/469080f810a0049442f02393fae746cebc391cc0)
> * Read more about [the GraphQL schema with Apollo Server](https://www.apollographql.com/docs/apollo-server/v2/essentials/schema.html)
* 延伸阅读：[Apollo Server 中的 GraphQL 模式](https://www.apollographql.com/docs/apollo-server/v2/essentials/schema.html)
> * Read more about [the GraphQL mindset: Thinking in Graphs](https://graphql.github.io/learn/thinking-in-graphs/)
* 延伸阅读：[GraphQL 思维模式：用图来思考](https://graphql.github.io/learn/thinking-in-graphs/)
> * Read more about [nullability in GraphQL](https://blog.apollographql.com/using-nullability-in-graphql-2254f84c4ed7)
* 延伸阅读：[GraphQL 中的可空](https://blog.apollographql.com/using-nullability-in-graphql-2254f84c4ed7)
> ## Apollo Server: Resolvers

## Apollo 服务端: 解析器
> This section continuous with the GraphQL schema in Apollo Server, but transitions more to the resolver side of the subject. In your GraphQL type definitions you have defined types, their relations and their structure. But there is nothing about how to get the data. That's where the GraphQL resolvers come into play.

本节将继续介绍 Apollo Server 中的 GraphQL 模式，但更多的是过渡到解析器方面。在你的 GraphQL 类型定义中，你已经定义了类型、它们的关系和结构。但还没有涉及到关于如何获取数据的。这就是 GraphQL 的解析器发挥作用的地方。
> In JavaScript, the resolvers are grouped in a JavaScript object, often called a **resolver map**. Each top level query in your Query type has to have a resolver. Now, we'll resolve things on a per-field level.

在 JavaScript 中，如果解析器一个 JavaScript 对象，那么通常将称其为 **解析器映射**。 Query 类型中的每个顶级查询都必须有一个解析器。现在，我们将在每个字段级别上也对它进行处理。

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  Query: {
    users: () => {
      return Object.values(users);
    },
    user: (parent, { id }) => {
      return users[id];
    },
    me: () => {
      return me;
    },
  },

# leanpub-start-insert
  User: {
    username: () => 'Hans',
  },
# leanpub-end-insert
};
~~~~~~~~
> Once you start your application again and query for a list of users, every user should have an identical username.

再次启动应用程序并查询 users 列表，列表中的每个 user 都具有相同的 username。

{title="GraphQL Playground",lang="json"}
~~~~~~~~
// query
{
  users {
    username
    id
  }
}

// query result
{
  "data": {
    "users": [
      {
        "username": "Hans",
        "id": "1"
      },
      {
        "username": "Hans",
        "id": "2"
      }
    ]
  }
}
~~~~~~~~
> The GraphQL resolvers can operate more specifically on a per-field level. You can override the username of every User type by resolving a `username` field. Otherwise, the default `username` property of the user entity is taken for it. Generally this applies to every field. Either you decide specifically what the field should return in a resolver function or GraphQL tries to fallback for the field by retrieving the property automatically from the JavaScript entity.

GraphQL 解析器可以在每个字段上进行更具体地操作。你可以通过解析一个 `username` 字段来覆盖每个 User 类型的 username。如果没有这么做的话，就会获取 User 实体的默认 `username` 属性。一般来说，这适用于每个字段。返回什么字段既可以是你在解析器函数中具体决定，也可以由 GraphQL 从 JavaScript 实体自动检索属性来尝试获取。
> Let's evolve this a bit by diving into the function signatures of resolver functions. Previously, you have seen that the second argument of the resolver function is the incoming arguments of a query. That's how you were able to retrieve the `id` argument for the user from the Query. The first argument is called the parent or root argument, and always returns the previously resolved field. Let's check this for the new username resolver function.

让我们通过深入研究解析器函数的参数来进一步改进它。在前面，你已经知道解析器函数的第二个参数是一次查询的传入参数。就是如何从查询中获取 user 的 `id` 参数。第一个参数称为父参数或根参数，并且总是返回以前解析过的字段。让我们看一下新的 username 解析器函数。

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  Query: {
    users: () => {
      return Object.values(users);
    },
    user: (parent, { id }) => {
      return users[id];
    },
    me: () => {
      return me;
    },
  },

  User: {
# leanpub-start-insert
    username: parent => {
      return parent.username;
    }
# leanpub-end-insert
  },
};
~~~~~~~~
> When you query your list of users again in a running application, all usernames should complete correctly. That's because GraphQL first resolves all users in the `users` resolver, and then goes through the User's `username` resolver for each user. Each user is accessible as the first argument in the resolver function, so they can be used to access more properties on the entity. You can rename your parent argument to make it more explicit:

当你在运行中的应用程序中再次查询 users 列表时，所有 username 都应该已经正确填写。这是因为 GraphQL 首先解析 `users` 解析器中的所有 users，然后为每个 user 遍历所有 User 的 `username` 解析器。每个 user 都可以作为访问解析器函数中的第一个参数，因此可以使用它们访问实体上的更多属性。你也可以显式地重命名父参数：

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  Query: {
    ...
  },

  User: {
# leanpub-start-insert
    username: user => {
      return user.username;
# leanpub-end-insert
    }
  },
};
~~~~~~~~
> In this case, the `username` resolver function is redundant, because it only mimics the default behavior of a GraphQL resolver. If you leave it out, the username would still resolves with its correct property. However, this fine control over the resolved fields opens up powerful possibilities. It gives you the flexibility to add data mapping without worrying about the data sources behind the GraphQL layer. Here, we expose the full username of a user, a combination of its first and last name by using template literals:

由于 `username` 解析器函数只是仿写了 GraphQL 解析器的默认行为，所以在本例中它是冗余的。即使省略掉它，username 属性仍然会正确地解析。不过，这种对已解析字段的精细控制为你提供了强大的可能性。它为你提供了添加数据映射的灵活性，而无需担心在 GraphQL 背后的数据来源。在这里，我们暴露一个 user 的完整的 username，它由 firstname 和 lastname 的字面量组合而成:

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  ...

  User: {
# leanpub-start-insert
    username: user => `${user.firstname} ${user.lastname}`,
# leanpub-end-insert
  },
};
~~~~~~~~
> For now, we are going to leave out the `username` resolver, because it only mimics the default behavior with Apollo Server. These are called **default resolvers**, because they work without explicit definitions. Next, look to the other arguments in the function signature of a GraphQL resolver:

因为 `username` 解析器只是在仿写 Apollo 服务端的默认行为，所以现在我们将省略掉它。这样的写法被称为**默认解析器**，因为它们被没有显式地定义。接下来，我们再看看 GraphQL 解析器中的其他参数:

{title="Code Playground",lang="javascript"}
~~~~~~~~
(parent, args, context, info) => { ... }
~~~~~~~~
> The context argument is the third argument in the resolver function used to inject dependencies from the outside to the resolver function. Assume the signed-in user is known to the outside world of your GraphQL layer because a request to your GraphQL server is made and the authenticated user is retrieved from elsewhere. You might decide to inject this signed in user to your resolvers for application functionality, which is done with the `me` user for the `me` field. Remove the declaration of the `me` user (`let me = ...`) and pass it in the context object when Apollo Server gets initialized instead:

上下文参数是解析器函数中的第三个参数，用于将依赖项从外部注入解析器函数。因为从其他地方取得的经过身份验证的用户也可以向 GraphQL 服务器发出请求，因此我们假设已登录 user 对 GraphQL 的外部世界是已知的。你可能想要将这个已登录的用户注入到应用程序的 resolvers 的方法中，这是可以通过 `me` 字段的 `me` user 完成。移除 `me` user 的声明 （`let me = ...`），并在 Apollo 服务端初始化时将其传递给上下文对象:

{title="src/index.js",lang="javascript"}
~~~~~~~~
const server = new ApolloServer({
  typeDefs: schema,
  resolvers,
# leanpub-start-insert
  context: {
    me: users[1],
  },
# leanpub-end-insert
});
~~~~~~~~
> Next, access it in the resolver's function signature as a third argument, which gets destructured into the `me` property from the context object.

接下来，在解析器函数中的第三个参数可以访问到它，该参数从上下文对象分解出 `me` 属性。

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  Query: {
    users: () => {
      return Object.values(users);
    },
    user: (parent, { id }) => {
      return users[id];
    },
# leanpub-start-insert
    me: (parent, args, { me }) => {
      return me;
    },
# leanpub-end-insert
  },
};
~~~~~~~~
> The context should be the same for all resolvers now. Every resolver that needs to access the context, or in this case the `me` user, can do so using the third argument of the resolver function.

现在所有解析器的上下文应该是相同的。每个解析器所需要访问的上下文，比如在本例中命名为 `me` 的 user，都可以使用解析器函数的第三个参数来访问到。
> The fourth argument in a resolver function, the info argument, isn't used very often, because it only gives you internal information about the GraphQL request. It can be used for debugging, error handling, advanced monitoring, and tracking. You don't need to worry about it for now.

解析器函数中的第四个参数，信息参数，不常使用，因为它只是提供关于 GraphQL 请求的内部信息。它可以用于调试、错误处理、高级监视和跟踪。你现在不用关心。
> A couple of words about the resolver's return values: a resolver can return arrays, objects and scalar types, but it has to be defined in the matching type definitions. The type definition has to define an array or non-nullable field to have the resolvers working appropriately. What about JavaScript promises? Often, you will make a request to a data source (database, RESTful API) in a resolver, returning a JavaScript promise in the resolver. GraphQL can deal with it, and waits for the promise to resolve. That's why you don't need to worry about asynchronous requests to your data source later.

关于解析器的返回值的几句话：解析器可以返回数组、对象和标量类型，但是必须在匹配的类型定义中定义它。类型定义必须定义数组或非空字段，以便解析器能正常工作。关于 JavaScript 的 promises 有什么要说的么？通常，你可以在解析器中向数据源(数据库、RESTful API)发出请求，并在解析器中返回 JavaScript 的 promise。GraphQL 可以处理它，并等待 promise 的 resolve。因此不需要担心对数据源的异步请求。
> ### Exercises:

### 练习：
> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/5d8ebc22260455ac6803af20838cbc1f2636be8f)
* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/5d8ebc22260455ac6803af20838cbc1f2636be8f)
> * Read more about [GraphQL resolvers in Apollo](https://www.apollographql.com/docs/apollo-server/v2/essentials/data.html)
* 延伸阅读：[Apollo 中的 GraphQL 解析器](https://www.apollographql.com/docs/apollo-server/v2/essentials/data.html)

>## Apollo Server: Type Relationships

## Apollo 服务端: 关系类型

>You started to evolve your GraphQL schema by defining queries, mutations, and type definitions. In this section, let's add a second GraphQL type called Message and see how it behaves with your User type. In this application, a user can have messages. Basically, you could write a simple chat application with both types. First, add two new top level queries and the new Message type to your GraphQL schema:

你需要通过定义查询，变更和类型定义以开始逐步构建你的 GraphQL 模式。在本节中，让我们添加第二个名为 Message 的 GraphQL 类型，并查看它如何处理你的 User 类型。在这个应用里，一个用户可以拥有多个消息，基本上，你可以使用这两种类型编写一个简单的聊天应用。首先，向你的 GraphQL 模式添加两个新的顶级查询和新的 Message 类型：


{title="src/index.js",lang="javascript"}
~~~~~~~~
const schema = gql`
  type Query {
    users: [User!]
    user(id: ID!): User
    me: User

# leanpub-start-insert
    messages: [Message!]!
    message(id: ID!): Message!
# leanpub-end-insert
  }

  type User {
    id: ID!
    username: String!
  }

# leanpub-start-insert
  type Message {
    id: ID!
    text: String!
  }
# leanpub-end-insert
`;
~~~~~~~~

>Second, you have to add two resolvers for Apollo Server to match the two new top level queries:

然后，你需要向 Apollo 服务端添加两个解析器来匹配这两个新的顶级查询:

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
let messages = {
  1: {
    id: '1',
    text: 'Hello World',
  },
  2: {
    id: '2',
    text: 'By World',
  },
};
# leanpub-end-insert

const resolvers = {
  Query: {
    users: () => {
      return Object.values(users);
    },
    user: (parent, { id }) => {
      return users[id];
    },
    me: (parent, args, { me }) => {
      return me;
    },
# leanpub-start-insert
    messages: () => {
      return Object.values(messages);
    },
    message: (parent, { id }) => {
      return messages[id];
    },
# leanpub-end-insert
  },
};
~~~~~~~~

>Once you run your application again, your new GraphQL queries should work in GraphQL playground. Now we'll add relationships to both GraphQL types. Historically, it was common with REST to add an identifier to each entity to resolve its relationship.

再次运行你的应用，新的 GraphQL 查询应该可以在 GraphQL playground 中工作了。现在我们将为这两种 GraphQL 类型添加关系。过去 REST 通常为每一个实体添加一个标志符来解析它们的关系。

{title="src/index.js",lang="javascript"}
~~~~~~~~
const schema = gql`
  type Query {
    users: [User!]
    user(id: ID!): User
    me: User

    messages: [Message!]!
    message(id: ID!): Message!
  }

  type User {
    id: ID!
    username: String!
  }

  type Message {
    id: ID!
    text: String!
# leanpub-start-insert
    userId: ID!
# leanpub-end-insert
  }
`;
~~~~~~~~

>With GraphQL, Instead of using an identifier and resolving the entities with multiple waterfall requests, you can use the User entity within the message entity directly:

在 GraphQL 中，你可以直接在消息实体中使用用户实体，而不需要使用标志符和多个瀑布式请求解析实体:

{title="src/index.js",lang="javascript"}
~~~~~~~~
const schema = gql`
  ...

  type Message {
    id: ID!
    text: String!
# leanpub-start-insert
    user: User!
# leanpub-end-insert
  }
`;
~~~~~~~~

>Since a message doesn't have a user entity in your model, the default resolver doesn't work. You need to set up an explicit resolver for it.

因为在你的模型中一个消息没有用户实体，默认的解析器无法工作，你需要为它设置一个显式的解析器。

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  Query: {
    users: () => {
      return Object.values(users);
    },
    user: (parent, { id }) => {
      return users[id];
    },
    me: (parent, args, { me }) => {
      return me;
    },
    messages: () => {
      return Object.values(messages);
    },
    message: (parent, { id }) => {
      return messages[id];
    },
  },

# leanpub-start-insert
  Message: {
    user: (parent, args, { me }) => {
      return me;
    },
  },
# leanpub-end-insert
};
~~~~~~~~

>In this case, every message is written by the authenticated `me` user. If you query the following about messages, you will get this result:

在本例中，每条消息都是由经过身份验证的 `me` 用户编写。如果你查询以下关于消息的内容，将得到以下结果:

{title="GraphQL Playground",lang="json"}
~~~~~~~~
// query
{
  message(id: "1") {
    id
    text
    user {
      id
      username
    }
  }
}

// query result
{
  "data": {
    "message": {
      "id": "1",
      "text": "Hello World",
      "user": {
        "id": "1",
        "username": "Robin Wieruch"
      }
    }
  }
}
~~~~~~~~

>Let's make the behavior more like in a real world application. Your sample data needs keys to reference entities to each other, so the message passes a `userId` property:

让我们使它的行为更像现实中的应用。你的示例数据需要键来让实体互相引用，所以消息传递了一个 `userId` 属性:

{title="src/index.js",lang="javascript"}
~~~~~~~~
let messages = {
  1: {
    id: '1',
    text: 'Hello World',
# leanpub-start-insert
    userId: '1',
# leanpub-end-insert
  },
  2: {
    id: '2',
    text: 'By World',
# leanpub-start-insert
    userId: '2',
# leanpub-end-insert
  },
};
~~~~~~~~

>The parent argument in your resolver function can be used to get a message's `userId`, which can then be used to retrieve the appropriate user.

解析器中的 parent 参数可用于获取消息的 `userId`，然后用于检索对应的用户。

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  ...

  Message: {
# leanpub-start-insert
    user: message => {
      return users[message.userId];
# leanpub-end-insert
    },
  },
};
~~~~~~~~

>Now every message has its own dedicated user. The last steps were crucial for understanding GraphQL. Even though you have default resolver functions or this fine-grained control over the fields by defining your own resolver functions, it is up to you to retrieve the data from a data source. The developer makes sure every field can be resolved. GraphQL lets you group those fields into one GraphQL query, regardless of the data source.

现在每条消息都拥有自己独有的用户。最后一步对于理解 GraphQL 非常重要。即使你有默认的解析器函数，或者通过定义自己的解析器函数对字段进行细粒度控制，你也可以从数据源检索数据。开发者确保每一个字段都会被解析。GraphQL 允许你将这些字段组合到一个 GraphQL 查询中，而不在意数据源是什么。

>Let's recap this implementation detail again with another relationship that involves user messages. In this case, the relationships go in the other direction.

让我们再次回顾一下涉及用户消息的另一种关系的实现细节。 在本例中，关系将走向另一个方向。

{title="src/index.js",lang="javascript"}
~~~~~~~~
let users = {
  1: {
    id: '1',
    username: 'Robin Wieruch',
# leanpub-start-insert
    messageIds: [1],
# leanpub-end-insert
  },
  2: {
    id: '2',
    username: 'Dave Davids',
# leanpub-start-insert
    messageIds: [2],
# leanpub-end-insert
  },
};
~~~~~~~~

>This sample data could come from any data source. The important part is that it has a key that defines a relationship to another entity. All of this is independent from GraphQL, so let's define the relationship from users to their messages in GraphQL.

这个示例数据可以来源于任何数据源。重要的部分是它通过一个键来定义与另一个实体的关系。所有这些都独立于 GraphQL ，因此让我们在 GraphQL 中定义用户与他们的消息之间的关系。

{title="src/index.js",lang="javascript"}
~~~~~~~~
const schema = gql`
  type Query {
    users: [User!]
    user(id: ID!): User
    me: User

    messages: [Message!]!
    message(id: ID!): Message!
  }

  type User {
    id: ID!
    username: String!
# leanpub-start-insert
    messages: [Message!]
# leanpub-end-insert
  }

  type Message {
    id: ID!
    text: String!
    user: User!
  }
`;
~~~~~~~~

>Since a user entity doesn't have messages, but message identifiers, you can write a custom resolver for the messages of a user again. In this case, the resolver retrieves all messages from the user from the list of sample messages.

由于用户实体没有消息，只有消息标识符，因此可以再次为用户的消息编写自定义解析器。在本例中，解析器从示例消息列表中检索用户的所有消息。

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  ...

# leanpub-start-insert
  User: {
    messages: user => {
      return Object.values(messages).filter(
        message => message.userId === user.id,
      );
    },
  },
# leanpub-end-insert

  Message: {
    user: message => {
      return users[message.userId];
    },
  },
};
~~~~~~~~

>This section has shown you how to expose relationships in your GraphQL schema. If the default resolvers don't work, you have to define your own custom resolvers on a per field level for resolving the data from different data sources.

本节向你展示了如何在 GraphQL 模式中公开关系。如果默认解析器不起作用，则必须在每个字段级别上定义自己的自定义解析器，以便解析来自不同数据源的数据。

>### Exercises:

>* Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/491d93a90f4ee3413d9226e0a18c10b7407949ef)
>* Query a list of users with their messages
>* Query a list of messages with their user
>* Read more about [the GraphQL schema](https://graphql.github.io/learn/schema/)

### 练习：

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/491d93a90f4ee3413d9226e0a18c10b7407949ef)
* 查询用户列表及其消息
* 查询消息列表及其用户
* 延伸阅读：[GraphQL 模式](https://graphql.github.io/learn/schema/)

>## Apollo Server: Queries and Mutations

## Apollo 服务端: 查询与变更

>So far, you have only defined queries in your GraphQL schema using two related GraphQL types for reading data. These should work in GraphQL Playground, because you have given them equivalent resolvers. Now we'll cover GraphQL mutations for writing data. In the following, you create two mutations: one to create a message, and one to delete it. Let's start with creating a message as the currently signed in user (the `me` user).

到目前为止，你只使用两个 GraphQL 相关的类型在 GraphQL 模式中定义了查询来读取数据。这些应该可以在 GraphQL Playground 中使用，因为你已经为它们提供了对应的解析器。现在我们将介绍用于编写数据的 GraphQL 变更。接下来，你将创建两个变更：一个用于创建消息，另一个用于删除消息。让我们从创建当前登录用户(`me` 用户)的消息开始 。

{title="src/index.js",lang="javascript"}
~~~~~~~~
const schema = gql`
  type Query {
    users: [User!]
    user(id: ID!): User
    me: User

    messages: [Message!]!
    message(id: ID!): Message!
  }

# leanpub-start-insert
  type Mutation {
    createMessage(text: String!): Message!
  }
# leanpub-end-insert

  ...
`;
~~~~~~~~

>Apart from the Query type, there are also Mutation and Subscription types. There, you can group all your GraphQL operations for writing data instead of reading it. In this case, the `createMessage` mutation accepts a non-nullable `text` input as an argument, and returns the created message. Again, you have to implement the resolver as counterpart for the mutation the same as with the previous queries, which happens in the mutation part of the resolver map:

除了查询类型之外，还有变更和订阅类型。现在，你可以对所有 GraphQL 操作进行分组，以便编写数据而不是读取数据。在本例中，`createMessage` 变更接受一个不可空的 `text` 输入作为参数，并返回创建的消息。同样，你必须为变更实现与前面查询相同的对应的解析器，这发生在解析器映射的变更部分:

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  Query: {
    ...
  },

# leanpub-start-insert
  Mutation: {
    createMessage: (parent, { text }, { me }) => {
      const message = {
        text,
        userId: me.id,
      };

      return message;
    },
  },
# leanpub-end-insert

  ...
};
~~~~~~~~

>The mutation's resolver has access to the text in its second argument. It also has access to the signed-in user in the third argument, used to associate the created message with the user. The parent argument isn't used. The one thing missing to make the message complete is an identifier. To make sure a unique identifier is used, install this neat library in the command line:

变更的解析器可以从第二个参数中获取 text。 它还可以从第三个参数中获取已登录用户，用于将创建的消息与用户相关联。parent 参数未被使用。要使消息完整还缺少一个标志符。要使标志符唯一的话，请确保在命令行中安装这个整洁的库:

{title="Command Line",lang="json"}
~~~~~~~~
npm install uuid --save
~~~~~~~~

And import it to your file:

{title="src/index.js",lang="javascript"}
~~~~~~~~
import uuidv4 from 'uuid/v4';
~~~~~~~~

>Now you can give your message a unique identifier:

现在你可以为你的消息提供一个唯一标志符:

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  Query: {
    ...
  },

  Mutation: {
    createMessage: (parent, { text }, { me }) => {
# leanpub-start-insert
      const id = uuidv4();
# leanpub-end-insert
      const message = {
# leanpub-start-insert
        id,
# leanpub-end-insert
        text,
        userId: me.id,
      };

      return message;
    },
  },

  ...
};
~~~~~~~~

>So far, the mutation creates a message object and returns it to the API. However, most mutations have side-effects, because they are writing data to your data source or performing another action. Most often, it will be a write operation to your database, but in this case, you only need to update your `users` and `messages` variables. The list of available messages needs to be updated, and the user's reference list of `messageIds` needs to have the new message `id`.

截至目前，变更会创建一个消息对象并返回给 API。但是，大多数变更都有副作用，因为它们正在将数据写入数据源或执行其他操作。通常，这是对数据库的写入操作，但在这种情况下，你只需要更新 `users` 和 `messages` 变量。需要更新可用消息列表，并且用户的 `messageIds` 参考列表应当包含新消息的 `id` 。

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  Query: {
    ...
  },

  Mutation: {
    createMessage: (parent, { text }, { me }) => {
      const id = uuidv4();
      const message = {
        id,
        text,
        userId: me.id,
      };

# leanpub-start-insert
      messages[id] = message;
      users[me.id].messageIds.push(id);
# leanpub-end-insert

      return message;
    },
  },

  ...
};
~~~~~~~~

>That's it for the first mutation. You can try it right now in GraphQL Playground:

这是第一次变更。你可以立即在 GraphQL Playground 中尝试：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
mutation {
  createMessage (text: "Hello GraphQL!") {
    id
    text
  }
}
~~~~~~~~

>The last part is essentially your writing operation to a data source. In this case, you have only updated the sample data, but it would most likely be a database in practical use. Next, implement the mutation for deleting messages:

最后一部分实际上是对数据源的写入操作。在本例中，你只更新了示例数据，但在实际中，它很可能是使用的数据库。接下来，让我们实现删除消息的变更:

{title="src/index.js",lang="javascript"}
~~~~~~~~
const schema = gql`
  type Query {
    users: [User!]
    user(id: ID!): User
    me: User

    messages: [Message!]!
    message(id: ID!): Message!
  }

  type Mutation {
    createMessage(text: String!): Message!
# leanpub-start-insert
    deleteMessage(id: ID!): Boolean!
# leanpub-end-insert
  }

  ...
`;
~~~~~~~~

>The mutation returns a boolean that tells if the deletion was successful or not, and it takes an identifier as input to identify the message. The counterpart of the GraphQL schema implementation is a resolver:

这个变更返回一个布尔值告诉我们是否删除成功，并接受一个标识符作为输入来标识消息。这在 GraphQL 模式的实现中对应着一个解析器:

{title="src/index.js",lang="javascript"}
~~~~~~~~
const resolvers = {
  Query: {
    ...
  },

  Mutation: {
    ...

# leanpub-start-insert
    deleteMessage: (parent, { id }) => {
      const { [id]: message, ...otherMessages } = messages;

      if (!message) {
        return false;
      }

      messages = otherMessages;

      return true;
    },
# leanpub-end-insert
  },

  ...
};
~~~~~~~~

>The resolver finds the message by id from the messages object using destructuring. If there is no message, the resolver returns false. If there is a message, the remaining messages without the deleted message are the updated versions of the messages object. Then, the resolver returns true. Otherwise, if no message is found, the resolver returns false. Mutations in GraphQL and Apollo Server aren't much different from GraphQL queries, except they write data.

解析器使用从消息对象中解构的 id 查找消息。如果没有消息，解析器将返回 false 。如果有消息，解析器返回 true ，消息对象更新为不包含删除内容的消息。否则，如果没有找到消息，解析器返回 false 。除了写入数据，GraphQL 和 Apollo 服务端中的变更与 GraphQL 查询没有太大区别。

>There is only one GraphQL operation missing for making the messages features complete. It is possible to read, create, and delete messages, so the only operation left is updating them as an exercise.

要使消息功能完整只缺少一个 GraphQL 操作。可以读取、创建和删除消息，因此剩下的就是将它们作为练习。

>### Exercises:

### 练习：

>* Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/a10c54ec1b82043d98fcff2a6395fcd8e405bfda)
>* Create a message in GraphQL Playground with a mutation
>  * Query all messages
>  * Query the `me` user with messages
>* Delete a message in GraphQL Playground with a mutation
>  * Query all messages
>  * Query the me user with messages
>* Implement an `updateMessage` mutation for completing all CRUD operations for a message in GraphQL
>* Read more about [GraphQL queries and mutations](https://graphql.github.io/learn/queries/)

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/a10c54ec1b82043d98fcff2a6395fcd8e405bfda)
* 使用一个变更在 GraphQL Playground 中创建消息
  * 查询所有消息
  * 通过消息查询 `me` 用户
* 使用一个变更在 GraphQL Playground 中删除消息
  * 查询所有消息
  * 通过消息查询 `me` 用户
* 实现一个 `updateMessage` 变更，用于在 GraphQL 中完成消息的所有 CRUD 操作
* 延伸阅读：[GraphQL 查询与变更](https://graphql.github.io/learn/queries/)

>## GraphQL Schema Stitching with Apollo Server

## Apollo 服务端中的 GraphQL 模式拼接

>Schema stitching is a powerful feature in GraphQL. It's about merging multiple GraphQL schemas into one schema, which may be consumed in a GraphQL client application. For now, you only have one schema in your application, but there may come a need for more complicated operations that use multiple schemas and schema stitching. For instance, assume you have a GraphQL schema you want to modularize based on domains (e.g. user, message). You may end up with two schemas, where each schema matches one type (e.g. User type, Message type). The operation requires merging both GraphQL schemas to make the entire GraphQL schema accessible with your GraphQL server's API. That's one of the basic motivations behind schema stitching.

模式拼接是 GraphQL 中的一个强大的功能。它可以将多个 GraphQL 模式合并到一个模式中，以便可以在 GraphQL 客户端应用中使用。目前，你的应用程序中只有一个模式，但是可能需要使用多个模式和模式拼接进行更复杂的操作。例如，假设你有一个需要基于域 (例如用户、消息) 模块化的 GraphQL 模式。
最终可能会有两个模式，其中每个模式都匹配一种类型（例如用户类型，消息类型）。 该操作需要合并两个 GraphQL 模式，使整个 GraphQL 模式可以通过 GraphQL 服务器的 API 访问。 这是模式拼接的基本理念之一。

>But you can take this one step further: you may end up with microservices or third-party platforms that expose their dedicated GraphQL APIs, which then can be used to merge them into one GraphQL schema, where schema stitching becomes a single source of truth. Then again, a client can consume the entire schema, which is composed out of multiple domain-driven microservices.

但是你可以更进一步: 你可能最终会使用微服务或第三方平台，这些平台会公开其专用的 GraphQL API 将它们合并到一个 GraphQL 模式中，其中模式拼接成为一个单一的数据来源。同样，客户端可以使用由多个域驱动的微服务组成的整个模式。

>In our case, let's start with a separation by technical concerns for the GraphQL schema and resolvers. Afterward, you will apply the separation by domains that are users and messages.

在我们的例子中，让我们从 GraphQL 模式和解析器的技术关注点开始分离。之后，你需要按用户和消息的域分隔。

> ### Technical Separation
### 技术分离
> Let's take the GraphQL schema from the application where you have a User type and Message type. In the same step, split out the resolvers to a dedicated place. The *src/index.js* file, where the schema and resolvers are needed for the Apollo Server instantiation, should only import both things. It becomes three things when outsourcing data, which in this case is the sample data, now called models.
让我们从包含一种用 User 类型和 Message 类型的应用程序中获取 GraphQL 模式。在同一步骤中，拆出解析器到特定位置。 在 *src/index.js* 中，应该只导入 Apollo 服务端实例化所需要的模式和解析器。分发数据会变成三件事，在这种情况下是示例数据，现在称为模型。

{title="src/index.js",lang="javascript"}
~~~~~~~~
import cors from 'cors';
import express from 'express';
# leanpub-start-insert
import { ApolloServer } from 'apollo-server-express';
# leanpub-end-insert

# leanpub-start-insert
import schema from './schema';
import resolvers from './resolvers';
import models from './models';
# leanpub-end-insert

const app = express();

app.use(cors());

const server = new ApolloServer({
  typeDefs: schema,
  resolvers,
  context: {
# leanpub-start-insert
    models,
    me: models.users[1],
# leanpub-end-insert
  },
});

server.applyMiddleware({ app, path: '/graphql' });

app.listen({ port: 8000 }, () => {
  console.log('Apollo Server on http://localhost:8000/graphql');
});
~~~~~~~~

> As an improvement, models are passed to the resolver function's as context. The models are your data access layer, which can be sample data, a database, or a third-party API. It's always good to pass those things from the outside to keep the resolver functions pure. Then, you don't need to import the models in each resolver file. In this case, the models are the sample data moved to the *src/models/index.js* file:
作为改进，把模型当做上下文传递给解析器函数。模型是你的数据访问层，可以是示例数据，数据库或第三方 API。从外部传递模型来保持解析器函数的纯净总是好的。然后，你不需要在每个解析器文件中导入模型。在这个例子中，模型是放到 *src/models/index.js* 中的示例数据。

{title="src/models/index.js",lang="javascript"}
~~~~~~~~
let users = {
  1: {
    id: '1',
    username: 'Robin Wieruch',
    messageIds: [1],
  },
  2: {
    id: '2',
    username: 'Dave Davids',
    messageIds: [2],
  },
};

let messages = {
  1: {
    id: '1',
    text: 'Hello World',
    userId: '1',
  },
  2: {
    id: '2',
    text: 'By World',
    userId: '2',
  },
};

export default {
  users,
  messages,
};
~~~~~~~~

> Since you have passed the models to your Apollo Server context, they are accessible in each resolver. Next, move the resolvers to the *src/resolvers/index.js* file, and adjust the resolver's function signature by adding the models when they are needed to read/write users or messages.
由于你已将模型传递到 Apollo 服务端上下文，因此可在每个解析器中访问它们。接下来，将解析器移动到 *src/resolvers/index.js* 文件，同时，在需要读/写 users 或 messages 时，添加模型到解析器函数的签名。

{title="src/resolvers/index.js",lang="javascript"}
~~~~~~~~
import uuidv4 from 'uuid/v4';

export default {
  Query: {
# leanpub-start-insert
    users: (parent, args, { models }) => {
      return Object.values(models.users);
# leanpub-end-insert
    },
# leanpub-start-insert
    user: (parent, { id }, { models }) => {
      return models.users[id];
# leanpub-end-insert
    },
    me: (parent, args, { me }) => {
      return me;
    },
# leanpub-start-insert
    messages: (parent, args, { models }) => {
      return Object.values(models.messages);
# leanpub-end-insert
    },
# leanpub-start-insert
    message: (parent, { id }, { models }) => {
      return models.messages[id];
# leanpub-end-insert
    },
  },

  Mutation: {
# leanpub-start-insert
    createMessage: (parent, { text }, { me, models }) => {
# leanpub-end-insert
      const id = uuidv4();
      const message = {
        id,
        text,
        userId: me.id,
      };

# leanpub-start-insert
      models.messages[id] = message;
      models.users[me.id].messageIds.push(id);
# leanpub-end-insert

      return message;
    },

# leanpub-start-insert
    deleteMessage: (parent, { id }, { models }) => {
      const { [id]: message, ...otherMessages } = models.messages;
# leanpub-end-insert

      if (!message) {
        return false;
      }

# leanpub-start-insert
      models.messages = otherMessages;
# leanpub-end-insert

      return true;
    },
  },

  User: {
# leanpub-start-insert
    messages: (user, args, { models }) => {
      return Object.values(models.messages).filter(
# leanpub-end-insert
        message => message.userId === user.id,
      );
    },
  },

  Message: {
# leanpub-start-insert
    user: (message, args, { models }) => {
      return models.users[message.userId];
# leanpub-end-insert
    },
  },
};
~~~~~~~~

> The resolvers receive all sample data as models in the context argument rather than operating directly on the sample data as before. As mentioned, it keeps the resolver functions pure. Later, you will have an easier time testing resolver functions in isolation. Next, move your schema's type definitions in the *src/schema/index.js* file:
解析器在上下文参数中接收所有示例数据作为模型，而不像之前直接操作示例数据。如上所述，它使解析器函数保持纯净。稍后，你可以更轻松地单独测试解析器函数。接下来，把 schema 的类型定义移动到 *src/schema/index.js* 文件中：

{title="src/schema/index.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  type Query {
    users: [User!]
    user(id: ID!): User
    me: User

    messages: [Message!]!
    message(id: ID!): Message!
  }

  type Mutation {
    createMessage(text: String!): Message!
    deleteMessage(id: ID!): Boolean!
  }

  type User {
    id: ID!
    username: String!
    messages: [Message!]
  }

  type Message {
    id: ID!
    text: String!
    user: User!
  }
`;
~~~~~~~~

> The technical separation is complete, but the separation by domains, where schema stitching is needed, isn't done yet. So far, you have only outsourced the schema, resolvers and data (models) from your Apollo Server instantiation file. Everything is separated by technical concerns now. You also made a small improvement for passing the models through the context, rather than importing them in resolver files.
技术分离已经完成，但是需要进行 schema 组合的域分离还没有完成。到目前为止，你只从 Apollo 服务端实例化文件中拆分了模式，解析器和数据（模型）。现在一切都被技术关注点分开。你也通过上下文传递模型做了一些改进，而不是在解析器文件中导入它们。

> ### Domain Separation
### 域分离
> In the next step, modularize the GraphQL schema by domains (user and message). First, separate the user-related entity in its own schema definition file called *src/schema/user.js*:
在下一步，按域（user 和 message）模块化 GraphQL 模式。首先，将用户相关实体 schema 分离到名为 *src/schema/user.js*的 schema 定义文件中：

{title="src/schema/user.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
# leanpub-start-insert
  extend type Query {
# leanpub-end-insert
    users: [User!]
    user(id: ID!): User
    me: User
  }

  type User {
    id: ID!
    username: String!
    messages: [Message!]
  }
`;
~~~~~~~~

> The same applies for the message schema definition in *src/schema/message.js*:
这同样适用于 *src/schema/message.js* 中的消息 schema 定义：

{title="src/schema/message.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
# leanpub-start-insert
  extend type Query {
# leanpub-end-insert
    messages: [Message!]!
    message(id: ID!): Message!
  }

# leanpub-start-insert
  extend type Mutation {
# leanpub-end-insert
    createMessage(text: String!): Message!
    deleteMessage(id: ID!): Boolean!
  }

  type Message {
    id: ID!
    text: String!
    user: User!
  }
`;
~~~~~~~~

> Each file only describes its own entity, with a type and its relations. A relation can be a type from a different file, such as a Message type that still has the relation to a User type even though the User type is defined somewhere else. Note the `extend` statement on the Query and Mutation types. Since you have more than one of those types now, you need to extend the types. Next, define shared base types for them in the *src/schema/index.js*:
每个文件只描述自己的实体，具有类型及其关系。关系可以是来自不同文件的类型，例如一个 Message 类型与一个 User 类型存在关系，即使 User 类型是在其他位置定义的。请注意 Query 和 Mutation 类型的 `extend` 语句，由于你拥有多个这样类型，因此需要使用 extend 语句去扩展这些类型。接下来，在 *src / schema / index.js* 中为它们定义共享基类型。

{title="src/schema/index.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

import userSchema from './user';
import messageSchema from './message';

const linkSchema = gql`
  type Query {
    _: Boolean
  }

  type Mutation {
    _: Boolean
  }

  type Subscription {
    _: Boolean
  }
`;

export default [linkSchema, userSchema, messageSchema];
~~~~~~~~

> In this file, both schemas are merged with the help of a utility called `linkSchema`. The `linkSchema` defines all types shared within the schemas. It already defines a Subscription type for GraphQL subscriptions, which may be implemented later. As a workaround, there is an empty underscore field with a Boolean type in the merging utility schema, because there is no official way of completing this action yet. The utility schema defines the shared base types, extended with the `extend` statement in the other domain-specific schemas.
在此文件中，两个模式都在名为 `linkSchema` 的实用程序的帮助下合并。`linkSchema` 定义了模式中共享的所有类型。它已经为 GraphQL 订阅集定义了一个 Subscription 类型，可以在之后实现。因为还没有正式的方法来完成此操作，作为一种变通方法，在合并通用 schema 中存在一个带有 Boolean 类型的空下划线字段。通用 schema 定义共享基类型，在其他特定域 schemas 中使用 extend 进行扩展。

> This time, the application runs with a stitched schema instead of one global schema. What's missing are the domain separated resolver maps. Let's start with the user domain again in file in the *src/resolvers/user.js* file, whereas I leave out the implementation details for saving space here:
这次，应用程序使用组合 schema 而不是一个全局 schema 运行。缺少的是域分离的解析器映射。让我们再次从 *src/resolvers/user.js* 文件中的用户域开始，而这里我为了节省空间省略了实现细节：

{title="src/resolvers/user.js",lang="javascript"}
~~~~~~~~
export default {
  Query: {
    users: (parent, args, { models }) => {
      ...
    },
    user: (parent, { id }, { models }) => {
      ...
    },
    me: (parent, args, { me }) => {
      ...
    },
  },

  User: {
    messages: (user, args, { models }) => {
      ...
    },
  },
};
~~~~~~~~

> Next, add the message resolvers in the *src/resolvers/message.js* file:
接下来，在 *src/resolvers/message.js* 文件中添加消息解析器：

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
import uuidv4 from 'uuid/v4';

export default {
  Query: {
    messages: (parent, args, { models }) => {
      ...
    },
    message: (parent, { id }, { models }) => {
      ...
    },
  },

  Mutation: {
    createMessage: (parent, { text }, { me, models }) => {
      ...
    },

    deleteMessage: (parent, { id }, { models }) => {
      ...
    },
  },

  Message: {
    user: (message, args, { models }) => {
      ...
    },
  },
};
~~~~~~~~

> Since the Apollo Server accepts a list of resolver maps too, you can import all of your resolver maps in your *src/resolvers/index.js* file, and export them as a list of resolver maps again:
由于 Apollo 服务端也接受解析器映射列表，你可以在 *src/resolvers/index.js* 文件中导入所有解析器映射，并再次将它们导出为解析器映射列表。

{title="src/resolvers/index.js",lang="javascript"}
~~~~~~~~
import userResolvers from './user';
import messageResolvers from './message';

export default [userResolvers, messageResolvers];
~~~~~~~~

> Then, the Apollo Server can take the resolver list to be instantiated. Start your application again and verify that everything is working for you.
然后，Apollo 服务端可以将解析器列表实例化。再次启动你的应用程序并确认一切正常。

> In the last section, you extracted schema and resolvers from your main file and separated both by domains. The sample data is placed in a *src/models* folder, where it can be migrated to a database-driven approach later. The folder structure should look similar to this:
在上一节中，你从主文件中提取了模式和解析器，并按域分隔。示例数据放在 *src/models* 文件夹中，以后可以将其迁移到数据库驱动的方式。文件夹结构应该与此类似：

* src/
  * models/
    * index.js
  * resolvers/
    * index.js
    * user.js
    * message.js
  * schema/
    * index.js
    * user.js
    * message.js
  * index.js

> You now have a good starting point for a GraphQL server application with Node.js. The last implementations gave you a universally usable GraphQL boilerplate project to serve as a foundation for your own software development projects. As we continue, the focus becomes connecting GraphQL server to databases, authentication and authorization, and using powerful features like pagination.
现在，你通过 Node.js 获取了 GraphQL 服务端应用的良好起点。最后的实现为你提供了一个通用的 GraphQL 样板项目作为你自己开发项目的基础。我们之后的内容，重点是放在 GraphQL 服务器的数据库连接，身份验证和授权，并使用像分页这样的强大功能。

> ### Exercises:
### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/953ef4b2ac8edc7c6338fb73ecdc1446e9cbdc4d)
* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/953ef4b2ac8edc7c6338fb73ecdc1446e9cbdc4d)
> * Read more about [schema stitching with Apollo Server](https://www.apollographql.com/docs/graphql-tools/schema-stitching.html)
* 延伸阅读：[使用 Apollo Server 进行模式组合](https://www.apollographql.com/docs/graphql-tools/schema-stitching.html)
> * Schema stitching is only a part of **schema delegation**
* schema 组合只是 ** schema 委派** 的一部分
  > * Read more about [schema delegation](https://www.apollographql.com/docs/graphql-tools/schema-delegation.html)
  延伸阅读[模式委派](https://www.apollographql.com/docs/graphql-tools/schema-delegation.html)
  > * Familiarize yourself with the motivation behind **remote schemas** and **schema transforms**
  * 熟悉 **远程模式** 和 **模式转换** 背后的动机

> ## PostgreSQL with Sequelize for a GraphQL Server
## PostgreSQL 与 Sequelize 的 GraphQL 服务器

> To create a full-stack GraphQL application, you'll need to introduce a sophisticated data source. Sample data is fluctuant, while a database gives persistent data. In this section, you'll set up PostgreSQL with Sequelize ([ORM](https://en.wikipedia.org/wiki/Object-relational_mapping)) for Apollo Server. [PostgreSQL](https://www.postgresql.org/) is a SQL database whereas an alternative would be the popular NoSQL database called [MongoDB](https://www.mongodb.com/) (with Mongoose as ORM). The choice of tech is always opinionated. You could choose MongoDB or any other SQL/NoSQL solution over PostgreSQL, but for the sake of this application, let's stick to PostgreSQL.
要创建一个全栈 GraphQL 应用程序，你需要引入一个持久化的数据源。样本数据是波动的，而数据库则提供持久数据。在本节中，你将使用 Sequelize（[ORM](https://en.wikipedia.org/wiki/Object-relational_mapping)）为 Apollo 服务端设置 PostgreSQL。[PostgreSQL](https://www.postgresql.org/) 是一个 SQL 数据库，而另一种选择是流行的 NoSQL 数据库 [MongoDB](https://www.mongodb.com/) （用 Mongoose 作为 ORM）。技术选型始终是自由的。你可以选择 MongoDB 或任何其他 SQL/NoSQL 解决方案代替 PostgreSQL，但是为了这个应用程序，让我们坚持使用 PostgreSQL。

>This [setup guide](https://www.robinwieruch.de/postgres-express-setup-tutorial/) will walk you through the basic PostgreSQL setup, including installation, your first database, administrative database user setup, and essential commands. These are the things you should have accomplished after going through the instructions:
这个安装指南将引导你完成基本的 PostgreSQL 设置，包括安装，创建你的第一个数据库，管理数据库用户设置和必要命令。这些是你在阅读完说明后应该完成的事情：

> * A running installation of PostgreSQL
* PostgreSQL 的运行安装
> * A database super user with username and password
* 具有用户名和密码的数据库超级用户
> * A database created with `createdb` or `CREATE DATABASE`
* 使用 `createdb` 或 `CREATE DATABASE` 创建数据库

> You should be able to run and stop your database with the following commands:
你应该能够使用如下命令运行和停止数据库：

* pg_ctl -D /usr/local/var/postgres start
* pg_ctl -D /usr/local/var/postgres stop

> Use the `psql` command to connect to your database in the command line, where you can list databases and execute SQL statements against them. You should find a couple of these operations in the PostgreSQL setup guide, but this section will also show some of them. Consider performing these in the same way you've been completing GraphQL operations with GraphQL Playground. The `psql` command line interface and GraphQL Playground are effective tools for testing applications manually.
使用 `psql` 命令在命令行中连接到数据库，你可以查看到数据库并对其执行 SQL 语句。你应该在 PostgreSQL 设置指南中找到这些操作，但本节还会展示其中的一些。考虑以使用 GraphQL Playground 完成 GraphQL 操作相同的方式执行。`psql` 命令行接口和 GraphQL Playground 是手动测试应用程序的有效工具。

> Once you have installed PostgreSQL on your local machine, you'll also want to acquire [PostgreSQL for Node.js](https://github.com/brianc/node-postgres) and [Sequelize (ORM)](https://github.com/sequelize/sequelize) for your project. I highly recommend you keep the Sequelize documentation open, as it will be useful for reference when you connect your GraphQL layer (resolvers) with your data access layer (Sequelize).
在你本地计算机上安装 PostgreSQL 后，您还需要为你的项目安装 [PostgreSQL for Node.js](https://github.com/brianc/node-postgres) 和 [Sequelize (ORM)](https://github.com/sequelize/sequelize)。我强烈建议你打开 Sequelize 文档，因为当你将 GraphQL 层（解析器）与数据访问层（Sequelize）连接时，它将非常有用。

{title="Command Line",lang="json"}
~~~~~~~~
npm install pg sequelize --save
~~~~~~~~

> Now you can create models for the user and message domains. Models are usually the data access layer in applications. Then, set up your models with Sequelize to make read and write operations to your PostgreSQL database. The models can then be used in GraphQL resolvers by passing them through the context object to each resolver. These are the essential steps:
现在，你可以为 user 和 message 域创建模型。模型通常是应用程序中的数据访问层。然后，使用 Sequelize 设置模型，对 PostgreSQL 数据库进行读写操作。然后通过上下文文传递模型到每个 GraphQL 解析器中来使用它们。这些是必不可少的步骤：

> * Creating a model for the user domain
* 为用户域创建模型
> * Creating a model for the message domain
* 为消息域创建模型
> * Connecting the application to a database
* 连接应用程序到数据库
  > * Providing super user's username and password
  * 提供超级用户的用户名和密码
  > * Combining models for database use
  * 合并使用数据库的模型
> * Synchronizing the database once application starts
* 应用程序启动后同步数据库

> First, implement the *src/models/user.js* model:
首先，实现 *src/models/user.js* 模型：
{title="src/models/user.js",lang="javascript"}
~~~~~~~~
const user = (sequelize, DataTypes) => {
  const User = sequelize.define('user', {
    username: {
      type: DataTypes.STRING,
    },
  });

  User.associate = models => {
    User.hasMany(models.Message, { onDelete: 'CASCADE' });
  };

  return User;
};

export default user;
~~~~~~~~

> Next, implement the *src/models/message.js* model:
下一步，实现 *src/models/message.js* 模型：

{title="src/models/message.js",lang="javascript"}
~~~~~~~~
const message = (sequelize, DataTypes) => {
  const Message = sequelize.define('message', {
    text: {
      type: DataTypes.STRING,
    },
  });

  Message.associate = models => {
    Message.belongsTo(models.User);
  };

  return Message;
};

export default message;
~~~~~~~~

> Both models define the shapes of their entities. The message model has a database column with the name text of type string. You can add multiple database columns horizontally to your model. All columns of a model make up a table row in the database, and each row reflects a database entry, such as a message or user. The database table name is defined by an argument in the Sequelize model definition. The message domain has the table "message". You can define relationships between entities with Sequelize using associations. In this case, a message entity belongs to one user, and that user has many messages. That's a minimal database setup with two domains, but since we're focusing on server-side GraphQL, you should consider reading more about databases subjects outside of these applications to fully grasp the concept.
两种模型都定义了它们的实体形式。message 模型有一个叫做 text 的字符串类型的数据库列。你可以将多个数据库列水平添加到模型中。一个模型的所有列组成数据库中的表行，每行反映数据库条目，例如 message 或 user。数据库表名称由 Sequelize 模型定义中的参数决定。消息域中有 "message" 表。你可以使用 Sequelize 和 association 定义实体之间的关系。在这个例子中，message 实体属于一个 user，并且该 user 具有许多 message。这是一个包含两个域的最简单的数据库设置，但是因为我们专注于 GraphQL 的服务器端实现 ，你应该考虑阅读更多本应用之外的关于数据库的信息，来完全掌握这些概念。

> Next, connect to your database from within your application in the *src/models/index.js* file. We'll need the database name, a database super user, and the user's password. You may also want to define a database dialect, because Sequelize supports other databases as well.
接下来，从应用程序中的 *src/models/index.js* 文件连接到你的数据库。我们需要数据库名称，数据库超级用户和用户密码。你可能还想定义数据库 dialect，因为 Sequelize 也支持其他数据库。

{title="src/models/index.js",lang="javascript"}
~~~~~~~~
import Sequelize from 'sequelize';

const sequelize = new Sequelize(
  process.env.DATABASE,
  process.env.DATABASE_USER,
  process.env.DATABASE_PASSWORD,
  {
    dialect: 'postgres',
  },
);

export { sequelize };
~~~~~~~~

> In the same file, you can physically associate all your models with each other to expose them to your application as data access layer (models) for the database.
在同一文件中，你可以将所有模型彼此物理关联，来将它们作为数据库的数据访问层（模型）公开给应用程序。

{title="src/models/index.js",lang="javascript"}
~~~~~~~~
import Sequelize from 'sequelize';

const sequelize = new Sequelize(
  process.env.DATABASE,
  process.env.DATABASE_USER,
  process.env.DATABASE_PASSWORD,
  {
    dialect: 'postgres',
  },
);

# leanpub-start-insert
const models = {
  User: sequelize.import('./user'),
  Message: sequelize.import('./message'),
};

Object.keys(models).forEach(key => {
  if ('associate' in models[key]) {
    models[key].associate(models);
  }
});
# leanpub-end-insert

export { sequelize };

# leanpub-start-insert
export default models;
# leanpub-end-insert
~~~~~~~~

> The database credentials--database name, database super user name, database super user password--can be stored as environment variables. In your *.env* file, add those credentials as key value pairs. My defaults for local development are:
数据库的凭据--数据库名，数据库超级用户名，数据库超级用户密码可以存储在环境变量中。在 *.env* 文件中将这些凭据添加为键值对。 我对本地开发的默认值是：

{title=".env",lang="javascript"}
~~~~~~~~
DATABASE=postgres
DATABASE_USER=postgres
DATABASE_PASSWORD=postgres
~~~~~~~~

> You set up environment variables when you started creating this application. If not, you can also leave credentials in the source code for now. Finally, the database needs to be migrated/synchronized once your Node.js application starts. To complete this operation in your *src/index.js* file:
你可以在创建此应用程序时设置环境变量。如果没有，你也可以在源代码中保留这些凭据。最后，一旦 Node.js 应用程序启动，就需要迁移/同步数据库。在 *src/index.js* 文件中完成这些操作：

{title="src/index.js",lang="javascript"}
~~~~~~~~
import express from 'express';
import { ApolloServer } from 'apollo-server-express';

import schema from './schema';
import resolvers from './resolvers';
# leanpub-start-insert
import models, { sequelize } from './models';
# leanpub-end-insert

...

# leanpub-start-insert
sequelize.sync().then(async () => {
# leanpub-end-insert
  app.listen({ port: 8000 }, () => {
    console.log('Apollo Server on http://localhost:8000/graphql');
  });
# leanpub-start-insert
});
# leanpub-end-insert
~~~~~~~~

> We've completed the database setup for a GraphQL server. Next, you'll replace the business logic in your resolvers, because that is where Sequelize is used to access the database instead the sample data. The application isn't quite complete, because the resolvers don't use the new data access layer.
我们已经完成了 GraphQL 服务器的数据库设置。接下来，你将替换解析器中的业务逻辑，因为这是使用 Sequelize 访问数据库而不是示例数据的地方。应用程序还不完整，因为解析器没有使用新的数据访问层。

> ### Exercises:
### 练习：
> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/a1927fc375a62a9d7d8c514f8bf7f576587cca93)
* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/a1927fc375a62a9d7d8c514f8bf7f576587cca93)
> * Familiarize yourself with databases
* 熟悉数据库
  > * Try the `psql` command-line interface to access a database
  * 尝试使用 `psql` 命令行界面来访问数据库
  > * Check the Sequelize API by reading through their documentation
  * 通过阅读他们的文档来检查 Sequelize API
  > * Look up any unfamiliar database jargon mentioned here.
  * 查看此处提到的任何不熟悉的数据库术语。

> ## Connecting Resolvers and Database

## 连接解析器和数据库

> Your PostgreSQL database is ready to connect to a GraphQL server on startup. Now, instead of using the sample data, you will use data access layer (models) in GraphQL resolvers for reading and writing data to and from a database. In the next section, we will cover the following:

既然你已经准备好了在 GraphQL 服务器启动时用来连接的 PostgreSQL 数据库。现在，我们将在 GraphQL 解析器中使用数据访问层（模型）来读写数据库中的数据而不是简单的使用示例数据。在接下来的一节中我们将涵盖以下内容：

> * Use the new models in your GraphQL resolvers
* 在 GraphQL 解析器中使用新的模型

> * Seed your database with data when your application starts
* 在应用启动时准备好种子数据

> * Add a user model method for retrieving a user by username
* 增加一个 user 模型方法，通过 username 检索 user

> * Learn the essentials about `psql` for the command line
* 学习 `psql` 命令行的一些基本操作

> Let's start by refactoring the GraphQL resolvers. You passed the models via Apollo Server's context object to each GraphQL resolver earlier. We used sample data before, but the Sequelize API is necessary for our real-word database operations. In the *src/resolvers/user.js* file, change the following lines of code to use the Sequelize API:

我们开始重构 GraphQL 解析器吧。我们之前通过 Apollo Server 的 context 对象将模型传递给了每一个 GraphQL 解析器。而且我们之前采用的是示例数据，但是对于的真实数据库操作来说，使用 Sequelize API 是很有必要的。在 *src/resolvers/user.js* 文件中，我们使用 Sequelize API 来修改下面的代码：

{title="src/resolvers/user.js",lang="javascript"}
~~~~~~~~
export default {
  Query: {
# leanpub-start-insert
    users: async (parent, args, { models }) => {
      return await models.User.findAll();
# leanpub-end-insert
    },
# leanpub-start-insert
    user: async (parent, { id }, { models }) => {
      return await models.User.findById(id);
# leanpub-end-insert
    },
# leanpub-start-insert
    me: async (parent, args, { models, me }) => {
      return await models.User.findById(me.id);
# leanpub-end-insert
    },
  },

  User: {
# leanpub-start-insert
    messages: async (user, args, { models }) => {
      return await models.Message.findAll({
        where: {
          userId: user.id,
        },
      });
# leanpub-end-insert
    },
  },
};
~~~~~~~~

> The `findAll()` and `findById()` are commonly used Sequelize methods for database operations. Finding all messages for a specific user is more specific, though. Here, you used the `where` clause to narrow down messages by the `userId` entry in the database. Accessing a database will add another layer of complexity to your application's architecture, so be sure to reference the Sequelize API documentation as much as needed going forward.

`findAll() `和 `findById()` 是 Sequelize 常用的数据库操作方法。但是，查找一个特定 user 的所有 message 相对来说有点太细节了。这里我们使用 `where` 语句，通过 `userId` 来缩小 message 的搜索范围。由于访问数据库的操作将使得应用的架构变得更复杂，因此请务必尽可能多地参考 Sequelize API 文档。

> Next, return to the *src/resolvers/message.js* file and perform adjustments to use the Sequelize API:

接下来我们回到 *src/resolvers/message.js* 文件，使用 Sequelize API 来做一些调整：

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
export default {
  Query: {
# leanpub-start-insert
    messages: async (parent, args, { models }) => {
      return await models.Message.findAll();
# leanpub-end-insert
    },
# leanpub-start-insert
    message: async (parent, { id }, { models }) => {
      return await models.Message.findById(id);
# leanpub-end-insert
    },
  },

  Mutation: {
# leanpub-start-insert
    createMessage: async (parent, { text }, { me, models }) => {
      return await models.Message.create({
        text,
        userId: me.id,
# leanpub-end-insert
      });
    },

# leanpub-start-insert
    deleteMessage: async (parent, { id }, { models }) => {
      return await models.Message.destroy({ where: { id } });
# leanpub-end-insert
    },
  },

  Message: {
# leanpub-start-insert
    user: async (message, args, { models }) => {
      return await models.User.findById(message.userId);
# leanpub-end-insert
    },
  },
};
~~~~~~~~

> Apart from the `findById()` and `findAll()` methods, you are creating and deleting a message in the mutations as well. Before, you had to generate your own identifier for the message, but now Sequelize takes care of adding a unique identifier to your message once it is created in the database.

除了 `findById()` 和 `findAll()` 方法以外，我们还将用到一些创建和删除 message 等这类变更操作。之前我们不得不为 message 生成一个唯一标识，但是现在当 message 在数据库中创建后，Sequelize 会为其添加唯一标识。

> There was one more crucial change in the two files: [async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function). Sequelize is a JavaScript promise-based ORM, so it always returns a JavaScript promise when operating on a database. That's where async/await can be used as a more readable version for asynchronous requests in JavaScript. You learned about the returned results of GraphQL resolvers in Apollo Server in a previous section. A result can be a JavaScript promise as well, because the resolvers are waiting for its actual result. In this case, you can also get rid of the async/await statements and your resolvers would still work. Sometimes it is better to be more explicit, however, especially when we add more business logic within the resolver's function body later, so we will keep the statements for now.

这两个文件还有一个重要的变化：[async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)。Sequelize 是一个基于 promise 的 JavaScript ORM，因此在做数据库操作时始终会返回一个 promise。async/await 的使用能够大大提高JavaScript 异步请求代码的可读性。在上一节中我们知道了在 Apollo 客户端中 GraphQL 解析器的返回结果。由于解析器会一直等待真实结果的返回，所以返回结果也可以是一个 promise。在这种情况下，你也可以删除 async/await 语句，当然你的解析器仍然可以工作。然而，有时候表达明确一点更好，特别是当我们稍后在解析器函数中添加更多业务逻辑的时候，因此我们先保留现在的语句。

> Now we'll shift to seeding the database with sample data when your applications starts with `npm start`. Once your database synchronizes before your server listens, you can create two user records manually with messages in your database. The following code for the *src/index.js* file shows how to perform these operations with async/await. Users will have a `username` with associated `messages`.

现在我们将运行 `npm start` 在启动应用时，将一些示例数据作为种子数据录入到数据库中。当数据库在服务器侦听前同步好后，你可以在数据库中手动创建两条 user 记录。在下面的 *src/index.js* 文件中的代码显示了如何使用 async/await 来执行这些操作。User 将拥有一个 `username` 属性，还有一个关联的 `messages` 属性。

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const eraseDatabaseOnSync = true;
# leanpub-end-insert

# leanpub-start-insert
sequelize.sync({ force: eraseDatabaseOnSync }).then(async () => {
  if (eraseDatabaseOnSync) {
    createUsersWithMessages();
  }
# leanpub-end-insert

  app.listen({ port: 8000 }, () => {
    console.log('Apollo Server on http://localhost:8000/graphql');
  });
});

# leanpub-start-insert
const createUsersWithMessages = async () => {
  await models.User.create(
    {
      username: 'rwieruch',
      messages: [
        {
          text: 'Published the Road to learn React',
        },
      ],
    },
    {
      include: [models.Message],
    },
  );

  await models.User.create(
    {
      username: 'ddavids',
      messages: [
        {
          text: 'Happy to release ...',
        },
        {
          text: 'Published a complete ...',
        },
      ],
    },
    {
      include: [models.Message],
    },
  );
};
# leanpub-end-insert
~~~~~~~~

> The `force` flag in your Sequelize `sync()` method can be used to seed the database on every application startup. You can either remove the flag or set it to `false` if you want to keep accumulated database changes over time. The flag should be removed for your production database at some point.

在 Sequelize 的 `sync()` 方法中 `force` 标记可用于在每次应用启动时插入种子数据。如果要随时保持累积的数据库改动，可以删除该标记或将其设置为 `false`。当应用于生产数据库时请记得删除这个标记。

> Next, we have to handle the `me` user. Before, you used one of the users from the sample data; now, the user will come from a database. It's a good opportunity to write a custom method for your user model in the *src/models/user.js* file:

接下来，我们必须要处理 `me` 这个 user 类型数据了。之前，我们使用了示例数据中的一个 user；现在，我们将从数据库中读取它。现在是时候在 *src/models/user.js* 文件中为 user 模型编写一个自定义方法了：

{title="src/models/user.js",lang="javascript"}
~~~~~~~~
const user = (sequelize, DataTypes) => {
  const User = sequelize.define('user', {
    username: {
      type: DataTypes.STRING,
    },
  });

  User.associate = models => {
    User.hasMany(models.Message, { onDelete: 'CASCADE' });
  };

# leanpub-start-insert
  User.findByLogin = async login => {
    let user = await User.findOne({
      where: { username: login },
    });

    if (!user) {
      user = await User.findOne({
        where: { email: login },
      });
    }

    return user;
  };
# leanpub-end-insert

  return User;
};

export default user;
~~~~~~~~

> The `findByLogin()` method on your user model retrieves a user by `username` or by `email` entry. You don't have an `email` entry on the user yet, but it will be added when the application has an authentication mechanism. The `login` argument is used for both `username` and `email`, for retrieving the user from the database, and you can see how it is used to sign in to an application with username or email.

在 user 模型的 `findByLogin()` 方法中，我们通过 `username` 或 `email` 条目来检索 user。虽然现在还没有用户的 `email` 条目，但是当应用具有身份验证机制时，我们会添加上。`username` 和 `email` 都可以作为 `login` 方法的参数来从数据库中检索 user，你也可以了解到如何使用 username 或 email 进行登录应用。

> You have introduced your first custom method on a database model. It is always worth considering where to put this business logic. When giving your model these access methods, you may end up with a concept called *fat models*. An alternative would be writing separate services like functions or classes for these data access layer functionalities.

我们已经在数据库模型上引入了第一个自定义方法，接下来我们要考虑的是把这个业务逻辑放在哪里。当为模型提供这些访问方法时，你可能最终会得到一个名为 *fat models* 的概念。另一种方法是为这些数据访问层功能编写单独的服务，如函数或类。

> The new model method can be used to retrieve the `me` user from the database. Then you can put it into the context object when the Apollo Server is instantiated in the *src/index.js* file:

新的模型方法可用于从数据库中检索出 `me`。然后在 *src/index.js* 文件中，当实例化 Apollo 服务端时我们可以将其放入上下文对象中：

{title="src/index.js",lang="javascript"}
~~~~~~~~
const server = new ApolloServer({
  typeDefs: schema,
  resolvers,
  context: {
    models,
# leanpub-start-insert
    me: models.User.findByLogin('rwieruch'),
# leanpub-end-insert
  },
});
~~~~~~~~

> However, this cannot work yet, because the user is read asynchronously from the database, so `me` would be a JavaScript promise rather than the actual user; and because you may want to retrieve the `me` user on a per-request basis from the database. Otherwise, the `me` user has to stay the same after the Apollo Server is created. Instead, use a function that returns the context object rather than an object for the context in Apollo Server. This function uses the async/await statements. The function is invoked every time a request hits your GraphQL API, so the `me` user is retrieved from the database with every request.

但是这还不行，因为 user 是从数据库中异步读取的，因此 `me` 将是一个 promise 而不是实际的 user；而且你可能希望在每次请求中都从数据库中检索出 `me`。否则，一旦 Apollo 服务端创建后 `me` 会保持不变。相反，这里使用一个返回上下文对象的 async/await 异步函数而不是 Apollo 服务端中一个直接的上下文对象。由于每次请求到 GraphQL API 时都会调用该函数，因此每次请求都会从数据库中检索出 `me`。

{title="src/index.js",lang="javascript"}
~~~~~~~~
const server = new ApolloServer({
  typeDefs: schema,
  resolvers,
# leanpub-start-insert
  context: async () => ({
# leanpub-end-insert
    models,
# leanpub-start-insert
    me: await models.User.findByLogin('rwieruch'),
  }),
# leanpub-end-insert
});
~~~~~~~~

> You should be able to start your application again. Try out different GraphQL queries and mutations in GraphQL Playground, and verify that everything is working for you. If there are any errors regarding the database, make sure that it is properly connected to your application and that the database is running on the command line too.

重新启动应用，在 GraphQL Playground 中尝试不同的 GraphQL 查询和变更操作，并验证这些都能正常工作。如果出现任何关于数据的错误，请确保它正确连接到你的应用，并且数据库能够在命令行正确运行。

> Since you have introduced a database now, GraphQL Playground is not the only manual testing tool anymore. Whereas GraphQL Playground can be used to test your GraphQL API, you may want to use the `psql` command line interface to query your database manually. For instance, you may want to check user message records in the database or whether a message exists there after it has been created with a GraphQL mutation. First, connect to your database on the command line:

由于现在已经引入了数据库，因此 GraphQL Playground 不再是唯一的手动测试工具。虽然 GraphQL Playground 可用于测试 GraphQL API，但你可能更希望使用 `psql` 命令行界面手动查询数据库。例如，你可能希望检查数据库中 user 关联的 message 记录，或者在通过 GraphQL 创建 message 之后检查其是否存在数据库中。首先，通过命令行连接到你的数据库：

{title="Command Line",lang="json"}
~~~~~~~~
psql mydatabasename
~~~~~~~~

> And second, try the following SQL statements. It's the perfect opportunity to learn more about SQL itself:

然后，尝试执行下面的 SQL 语句。趁这个机会熟悉更多的 SQL 操作：

{title="psql",lang="sql"}
~~~~~~~~
SELECT * from users;
SELECT text from messages;
~~~~~~~~

> Which leads to:

这里应该输出：

{title="psql",lang="sql"}
~~~~~~~~
mydatabase=# SELECT * from users;
 id | username |         createdAt          |         updatedAt
----+----------+----------------------------+----------------------------
  1 | rwieruch | 2018-08-21 21:15:38.758+08 | 2018-08-21 21:15:38.758+08
  2 | ddavids  | 2018-08-21 21:15:38.786+08 | 2018-08-21 21:15:38.786+08
(2 rows)

mydatabase=# SELECT text from messages;
               text
-----------------------------------
 Published the Road to learn React
 Happy to release ...
 Published a complete ...
(3 rows)
~~~~~~~~

> Every time you perform GraphQL mutations, it is wise to check your database records with the `psql` command-line interface. It is a great way to learn about [SQL](https://en.wikipedia.org/wiki/SQL), which is normally abstracted away by using an ORM such as Sequelize.

每次你执行完 GraphQL 变更后，最好使用 `psql` 命令行界面检查下数据库中记录。这是学习 [SQL](https://en.wikipedia.org/wiki/SQL) 的好方法，只是我们通常会使用像 Sequelize 这样的 ORM 来抽象。

> In this section, you have used a PostgreSQL database as data source for your GraphQL server, using Sequelize as the glue between your database and your GraphQL resolvers. However, this was only one possible solution. Since GraphQL is data source agnostic, you can opt-in any data source to your resolvers. It could be another database (e.g. MongoDB, Neo4j, Redis), multiple databases, or a (third-party) REST/GraphQL API endpoint. GraphQL only ensures all fields are validated, executed, and resolved when there is an incoming query or mutation, regardless of the data source.

在本节中，我们使用了 PostgreSQL 数据库作为 GraphQL 服务器的数据源，使用 Sequelize 来桥接数据库和 GraphQL 解析器。但是，这只是一种可能的解决方案。由于 GraphQL 与数据源无关，因此你可以选择将任何数据源添加到解析器中。它可以是另一个数据库（例如 MongoDB，Neo4j，Redis），多个数据库或（第三方）REST/GraphQL API endpoint。GraphQL 仅在存在进行查询或修改操作时确保所有字段都经过验证，执行和解析，而不管数据源如何。

> ### Exercises:

### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/27a1372264760879e86be377e069da738270c4f3)

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/27a1372264760879e86be377e069da738270c4f3)

> * Experiment with psql and the seeding of your database

* 尝试使用 psql 插入种子数据到数据库中

> * Experiment with GraphQL playground and query data which comes from a database now

* 现在尝试使用 GraphQL playground 来查询来自数据库中数据

> * Remove and add the async/await statements in your resolvers and see how they still work

* 在解析器中删除和添加 async/await 语句，看看它们是如何工作的

> * Read more about [GraphQL execution](https://graphql.github.io/learn/execution/)

* 延伸阅读：[GraphQL 执行](https://graphql.github.io/learn/execution/)

> ## Apollo Server: Validation and Errors

## Apollo 服务端：校验和错误处理

> Validation, error, and edge case handling are not often verbalized in programming. This section should give you some insights into these topics for Apollo Server and GraphQL. With GraphQL, you are in charge of what returns from GraphQL resolvers. It isn't too difficult inserting business logic into your resolvers, for instance, before they read from your database.

我们通常不会在编码过程中明确提出对于校验，错误以及边缘场景的处理。这节中会介绍一些关于 Apollo 服务端和 GraphQL 在这方面的内容。通常使用 GraphQL 过程中，你只需要负责 GraphQL 解析器的返回值。例如，在从数据库读取数据之前，将业务逻辑插入到解析器中并不太困难。

{title="src/resolvers/user.js",lang="javascript"}
~~~~~~~~
export default {
  Query: {
    users: async (parent, args, { models }) => {
      return await models.User.findAll();
    },
    user: async (parent, { id }, { models }) => {
      return await models.User.findById(id);
    },
    me: async (parent, args, { models, me }) => {
# leanpub-start-insert
      if (!me) {
        return null;
      }
# leanpub-end-insert

      return await models.User.findById(me.id);
    },
  },

  ...
};
~~~~~~~~

> It may be a good idea keeping the resolvers surface slim but adding business logic services on the side. Then it is always simple to reason about the resolvers. In this application, we keep the business logic in the resolvers to keep everything at one place and avoid scattering logic across the entire application.

保持解析器层轻量同时添加业务逻辑到 service 层是一个不错的方法，这样解析器就变得足够简单。但是在我们这个应用中，我们将业务逻辑都保留在解析器中，是为了将所有代码都保存在同一个地方，避免在整个应用中逻辑变得分散。

> Let's start with the validation, which will lead to error handling. GraphQL isn't directly concerned about validation, but it operates between tech stacks that are: the client application (e.g. showing validation messages) and the database (e.g. validation of entities before writing to the database).

我们先从校验开始，这将使得我们需要对错误进行处理。GraphQL 和校验并没有直接关系，但是它在客户端应用（例如显示校验信息）和数据库（例如在写到数据库之前对实体进行校验）之间发挥着作用。

> Let's add some basic validation rules to your database models. This section gives an introduction to the topic, as it would become too verbose to cover all uses cases in this application. First, add validation to your user model in the *src/models/user.js* file:

我们先来为数据库模型添加一些基本的校验规则。本节只会进行一些简单的介绍，因为如果我们在应用中涵盖所有的场景将会变得过于冗长。首先，我们在 *src/models/user.js* 文件中为 user 模型添加如下校验：

{title="src/models/user.js",lang="javascript"}
~~~~~~~~
const user = (sequelize, DataTypes) => {
  const User = sequelize.define('user', {
    username: {
      type: DataTypes.STRING,
# leanpub-start-insert
      unique: true,
      allowNull: false,
      validate: {
        notEmpty: true,
      },
# leanpub-end-insert
    },
  });

  ...

  return User;
};

export default user;
~~~~~~~~

> Next, add validation rules to your message model  in the *src/models/message.js* file:

接下来，在 *src/models/message.js* 文件中为 message 模型添加校验规则：

{title="src/models/message.js",lang="javascript"}
~~~~~~~~
const message = (sequelize, DataTypes) => {
  const Message = sequelize.define('message', {
    text: {
      type: DataTypes.STRING,
# leanpub-start-insert
      validate: { notEmpty: true },
# leanpub-end-insert
    },
  });

  Message.associate = models => {
    Message.belongsTo(models.User);
  };

  return Message;
};

export default message;
~~~~~~~~

> Now, try to create a message with an empty text in GraphQL Playground. It still requires a non-empty text for your message in the database. The same applies to your user entities, which now require a unique username. GraphQL and Apollo Server can handle these cases. Let's try to create a message with an empty text. You should see a similar input and output:

现在试下在 GraphQL Playground 中使用空的 text 字段创建一条 message。对数据库来说，也需要为 message 提供一个非空的 text 字段。这同样适用于 user 实体，同时它的 username 字段还有唯一性约束。GraphQL 和 Apollo Server 能够轻松应对这些场景。我们尝试使用空的 text 创建一条 message。你会看到一个类似的输入和输出：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
// mutation
mutation {
  createMessage(text: "") {
    id
  }
}

// mutation error result
{
  "data": null,
  "errors": [
    {
      "message": "Validation error: Validation notEmpty on text failed",
      "locations": [],
      "path": [
        "createMessage"
      ],
      "extensions": { ... }
    }
  ]
}
~~~~~~~~

> It seems like Apollo Server's resolvers make sure to transform [JavaScript errors](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) into valid GraphQL output. It is already possible to use this common error format in your client application without any additional error handling.

看起来 Apollo 服务端的解析器一定能将 [JavaScript errors](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) 转换为有效的 GraphQL 输出。现在已经可以在客户端应用中使用这种常见错误格式，而无需任何其他错误处理。

> If you want to add custom error handling to your resolver, you always can add the commonly try/catch block statements for async/await:

如果你想在解析器中添加自定义异常处理，你还可通过为 async/await 语句添加 try/catch 进行异常捕获。

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
export default {
  Query: {
    ...
  },

  Mutation: {
    createMessage: async (parent, { text }, { me, models }) => {
# leanpub-start-insert
      try {
# leanpub-end-insert
        return await models.Message.create({
          text,
          userId: me.id,
        });
# leanpub-start-insert
      } catch (error) {
        throw new Error(error);
      }
# leanpub-end-insert
    },

    ...
  },

  ...
};
~~~~~~~~

> The error output for GraphQL should stay the same in GraphQL Playground, because you used the same error object to generate the Error instance. However, you could also use your custom message here with `throw new Error('My error message.');`.

因为你使用了相同的错误对象来生成错误实例，所以在 GraphQL Playground 中的错误输出应该保持不变。当然你也可以使用自定义的错误消息 `throw new Error('My error message.');`。

> Another way of adjusting your error message is in the database model definition. Each validation rule can have a custom validation message, which can be defined in the Sequelize model:

另外一个调整错误消息的方法是在对数据库模型定义的时候。你可以在定义 Sequelize 模型的时候为每一条校验规则添加一个自定义的校验消息：

{title="src/models/message.js",lang="javascript"}
~~~~~~~~
const message = (sequelize, DataTypes) => {
  const Message = sequelize.define('message', {
    text: {
      type: DataTypes.STRING,
      validate: {
# leanpub-start-insert
        notEmpty: {
          args: true,
          msg: 'A message has to have a text.',
        },
# leanpub-end-insert
      },
    },
  });

  Message.associate = models => {
    Message.belongsTo(models.User);
  };

  return Message;
};

export default message;
~~~~~~~~

> This would lead to the following error(s) when attempting to create a message with an empty text. Again, it is straightforward in your client application, because the error format stays the same:

当我们尝试使用空的 text 创建一条 message 的时候将产生以下错误。当然，因为错误格式保持不变，所以输出还是非常简单：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "data": null,
  "errors": [
    {
# leanpub-start-insert
      "message": "SequelizeValidationError: Validation error: A message has to have a text.",
# leanpub-end-insert
      "locations": [],
      "path": [
        "createMessage"
      ],
      "extensions": { ... }
    }
  ]
}
~~~~~~~~

> That's one of the main benefits of using Apollo Server for GraphQL. Error handling is often free, because an error--be it from the database, a custom JavaScript error or another third-party--gets transformed into a valid GraphQL error result. On the client side, you don't need to worry about the error result's shape, because it comes in a common GraphQL error format where the data object is null but the errors are captured in an array. If you want to change your custom error, you can do it on a resolver per-resolver basis. Apollo Server comes with a solution for global error handling:

这也是在 GraphQL 中使用 Apollo 服务端最主要的原因之一。错误处理通常是不受约束的，因为错误可能来源于数据库、自定义抛出的 JavaScript 错误或者其他第三方，这些错误都将被转换成有效的 GraphQL 错误。在客户端，你不必担心具体的错误长什么样子，因为它有一个常见的 GraphQL 错误格式，其中数据对象为 null 但错误却是在数组中被捕获。如果你想更改自定义错误，你可以在解析器的基础上挨个处理。Apollo 服务端还提供了一种全局错误处理的解决方案：

{title="src/index.js",lang="javascript"}
~~~~~~~~
const server = new ApolloServer({
  typeDefs: schema,
  resolvers,
# leanpub-start-insert
  formatError: error => {
    // remove the internal sequelize error message
    // leave only the important validation error
    const message = error.message
      .replace('SequelizeValidationError: ', '')
      .replace('Validation error: ', '');

    return {
      ...error,
      message,
    };
  },
# leanpub-end-insert
  context: async () => ({
    models,
    me: await models.User.findByLogin('rwieruch'),
  }),
});
~~~~~~~~

> These are the essentials for validation and error handling with GraphQL in Apollo Server. Validation can happen on a database (model) level or on a business logic level (resolvers). It can happen on a directive level too (see exercises). If there is an error, GraphQL and Apollo Server will format it to work with GraphQL clients. You can also format errors globally in Apollo Server.

这些是在 Apollo 服务端中进行 GraphQL 校验和错误处理的基本要素。校验可以在数据库（模型）层或者业务逻辑（解析器）层，也可以在 directive 层（见练习）。如果出现错误，GraphQL 和 Apollo 服务端 、将对其进行格式化以兼容 GraphQL 客户端。你也可以在 Apollo 服务端中进行全局的错误格式化处理。

> ### Exercises:
### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/83b2a288ccd65c574ac3f2083c4ceee3197700e7)

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/83b2a288ccd65c574ac3f2083c4ceee3197700e7)

> * Add more validation rules to your database models

* 在你的数据库模型上添加更多的校验规则

> * Read more about validation in the Sequelize documentation

* 延伸阅读： Sequelize 校验规则的文档

> * Read more about [Error Handling with Apollo Server](https://www.apollographql.com/docs/apollo-server/v2/features/errors.html)

* 延伸阅读：[Apollo 服务端的错误处理](https://www.apollographql.com/docs/apollo-server/v2/features/errors.html)

> * Get to know the different custom errors in Apollo Server

* 了解在 Apollo Server 中不同的自定义错误

> * Read more about [GraphQL field level validation with custom directives](https://blog.apollographql.com/graphql-validation-using-directives-4908fd5c1055)

* 延伸阅读：[GraphQL 字段级校验以及自定义 directive](https://blog.apollographql.com/graphql-validation-using-directives-4908fd5c1055)

> * Read more about [custom schema directives](https://www.apollographql.com/docs/apollo-server/v2/features/directives.html)

* 延伸阅读：[自定义模式 directive](https://www.apollographql.com/docs/apollo-server/v2/features/directives.html)

> ## Apollo Server: Authentication

## Apollo 服务端: 认证

> Authentication in GraphQL is a popular topic. There is no opinionated way of doing it, but many people need it for their applications. GraphQL itself isn't opinionated about authentication since it is only a query language. If you want authentication in GraphQL, consider using GraphQL mutations. In this section, we use a minimalistic approach to add authentication to your GraphQL server. Afterward, it should be possible to register (sign up) and login (sign in) a user to your application. The previously used `me` user will be the authenticated user.

在 GraphQL 中认证是一个很热门的话题。没有固定的方法去做认证，但大多数人的应用都需要认证。GraphQL 本身并没有固定的认证机制，因为它只是一种查询语言。如果你想要在 GraphQL 中实现认证，考虑用 GraphQL 变更操作。在这一部分中，我们用最简化的方法去给你的 GraphQL 服务器添加认证。之后，应该可以通过服务器注册和登录一个用户到你的应用。之前用过的 `me` 用户将会做为一个认证过的用户。

> In preparation for the authentication mechanism with GraphQL, extend the user model in the _src/models/user.js_ file. The user needs an email address (as unique identifier) and a password. Both email address and username (another unique identifier) can be used to sign in to the application, which is why both properties were used for the user's `findByLogin()` method.

在准备用 GraphQL 实现认证机制时，在 _src/models/user.js_ 文件中扩展用户模型。用户需要一个电子邮件地址（作为唯一标识）和一个密码。电子邮件地址和用户名（另一种唯一标识）都可以被用于登录到应用，这就是为什么这两个属性都被用于用户的 `findByLogin()` 方法。

{title="src/models/user.js",lang="javascript"}

```
...

const user = (sequelize, DataTypes) => {
  const User = sequelize.define('user', {
    username: {
      type: DataTypes.STRING,
      unique: true,
      allowNull: false,
      validate: {
        notEmpty: true,
      },
    },
# leanpub-start-insert
    email: {
      type: DataTypes.STRING,
      unique: true,
      allowNull: false,
      validate: {
        notEmpty: true,
        isEmail: true,
      },
    },
    password: {
      type: DataTypes.STRING,
      allowNull: false,
      validate: {
        notEmpty: true,
        len: [7, 42],
      },
    },
# leanpub-end-insert
  });

  ...

  return User;
};

export default user;
```

> The two new entries for the user model have their own validation rules, same as before. The password of a user should be between 7 and 42 characters, and the email should have a valid email format. If any of these validations fails during user creation, it generates a JavaScript error, transforms and transfers the error with GraphQL. The registration form in the client application could display the validation error then.

同以前一样，用户模型的两个新字段拥有自己的验证规则。用户密码应该是 7 到 42 位的字符串，并且电子邮件应该具有合法的电子邮件格式。如果在用户创建期间任何一个验证失败了，则会生成一个 JavaScript 错误，并用 GraphQL 转换和传输错误。在客户端应用程序中的注册表单可能会显示验证错误。

> You may want to add the email, but not the password, to your GraphQL user schema in the _src/schema/user.js_ file too:

你也可能想添加电子邮件而不是密码到文件 _src/schema/user.js_ 中的 GraphQL 用户模式:

{title="src/schema/user.js",lang="javascript"}

```
import { gql } from 'apollo-server-express';

export default gql`
  ...

  type User {
    id: ID!
    username: String!
# leanpub-start-insert
    email: String!
# leanpub-end-insert
    messages: [Message!]
  }
`;
```

> Next, add the new properties to your seed data in the _src/index.js_ file:

然后，添加新的属性到文件 `src/index.js` 中的种子数据：

{title="src/index.js",lang="javascript"}

```
const createUsersWithMessages = async () => {
  await models.User.create(
    {
      username: 'rwieruch',
# leanpub-start-insert
      email: 'hello@robin.com',
      password: 'rwieruch',
# leanpub-end-insert
      messages: [ ... ],
    },
    {
      include: [models.Message],
    },
  );

  await models.User.create(
    {
      username: 'ddavids',
# leanpub-start-insert
      email: 'hello@david.com',
      password: 'ddavids',
# leanpub-end-insert
      messages: [ ... ],
    },
    {
      include: [models.Message],
    },
  );
};
```

> That's the data migration of your database to get started with GraphQL authentication.

这就是数据库的数据迁移以便开始 GraphQL 认证

### Registration (Sign Up) with GraphQL

### 用 GraphQL 实现注册

> Now, let's examine the details for GraphQL authentication. You will implement two GraphQL mutations: one to register a user, and one to log in to the application. Let's start with the sign up mutation in the _src/schema/user.js_ file:

现在，让我们来考察 GraphQL 认证的具体细节。你将会实现两个 GraphQL 变更操作：一个用于注册用户，另一个用于登录到应用程序。让我们从 _src/schema/user.js_ 文件中的注册变更操作开始：

{title="src/schema/user.js",lang="javascript"}

```
import { gql } from 'apollo-server-express';

export default gql`
  extend type Query {
    users: [User!]
    user(id: ID!): User
    me: User
  }

# leanpub-start-insert
  extend type Mutation {
    signUp(
      username: String!
      email: String!
      password: String!
    ): Token!
  }
# leanpub-end-insert

# leanpub-start-insert
  type Token {
    token: String!
  }
# leanpub-end-insert

  type User {
    id: ID!
    username: String!
    messages: [Message!]
  }
`;
```

> The `signUp` mutation takes three non-nullable arguments: username, email, and password. These are used to create a user in the database. The user should be able to take the username or email address combined with the password to enable a successful login.

`signUP` 变更操作接受三个不为空的参数：用户名，电子邮件和密码。这些参数是用于在数据库中创建用户。用户应该能够使用用户名或者电子邮件组合密码来成功登录。

> Now we'll consider the return type of the `signUp` mutation. Since we are going to use a token-based authentication with GraphQL, it is sufficient to return a token that is nothing more than a string. However, to distinguish the token in the GraphQL schema, it has its own GraphQL type. You will learn more about tokens in the following, because the token is all about the authentication mechanism for this application.

现在我们将会考虑 `signUp` 变更操作的返回值类型。由于我们准备使用 GraphQL 基于 token 的认证，因此仅仅返回一个字符串 token 就足够了。然而，为了在 GraphQL 模式中区分 token，它具有自己的 GraphQL 类型。你将会在如下学习到更多关于 token 的知识，因为这个应用所有关于认证机制的都跟 token 有关。

> First, add the counterpart for your new mutation in the GraphQL schema as a resolver function. In your _src/resolvers/user.js_ file, add the following resolver function that creates a user in the database and returns an object with the token value as string.

首先，在 GraphQL 模式中添加一份新的变更操作做为一个解析函数。在你的 _src/resolvers/user.js_ 文件中，添加如下解析函数用于在数据库中创建一个用户并且返回一个对象包含对应的字符串 token。

{title="src/resolvers/user.js",lang="javascript"}

```
# leanpub-start-insert
const createToken = async (user) => {
  ...
};
# leanpub-end-insert

export default {
  Query: {
    ...
  },

# leanpub-start-insert
  Mutation: {
    signUp: async (
      parent,
      { username, email, password },
      { models },
    ) => {
      const user = await models.User.create({
        username,
        email,
        password,
      });

      return { token: createToken(user) };
    },
  },
# leanpub-end-insert

  ...
};
```

> That's the GraphQL framework around a token-based registration. You created a GraphQL mutation and resolver for it, which creates a user in the database based on certain validations and its incoming resolver arguments. It creates a token for the registered user. For now, the set up is sufficient to create a new user with a GraphQL mutation.

这就是 GraphQL 框架关于创建一个基于 token 的注册。你已经为注册创建了一个 GraphQL 变更操作和一个解析器，它基于某些验证和传入的解析器参数在数据库中创建用户。它为注册的用户创建一个 token。目前的设置足以创建具有GraphQL 变更操作的新用户。

### Securing Passwords with Bcrypt

### 用 Bcrypt 保护密码

There is one major security flaw in this code: the user password is stored in plain text in the database, which makes it much easier for third parties to access it. To remedy this, we use add-ons like [bcrypt](https://github.com/kelektiv/node.bcrypt.js) to hash passwords. First, install it on the command line:

> 这段代码中有一个主要的安全的漏洞：用户密码是直接以文本形式存储在数据库中，这样会让第三方很容易的获取密码。为了补救这个问题，我们用 [bcrypt](https://github.com/kelektiv/node.bcrypt.js) 库来哈希密码。首先，通过命令行将其安装：

{title="Command Line",lang="json"}

```
npm install bcrypt --save
```

> Note: If you run into any problems with bcrypt on Windows while installing it, you can try out a substitute called [bcrypt.js](https://github.com/dcodeIO/bcrypt.js). It is slower, but people reported that it works on their machine.

注意：如果你在 Windows 上安装 bcrypt 的过程中出现任何问题，你可以尝试用一个叫 [bcrypt.js](https://github.com/dcodeIO/bcrypt.js) 的代替方案。尽管它有点慢，但是有人报告过它可以在他们的机器上运行。

> Now it is possible to hash the password with bcrypt in the user's resolver function when it gets created on a `signUp` mutation. There is also an alternative way with Sequelize. In your user model, define a hook function that is executed every time a user entity is created:

到此，当用户被 signUp 变更操作创建的时候可以用 bcrypt 在用户的解析函数中哈希密码。还有一种代替的方法是通过 Sequelize。在你的用户模型中，定义一个钩子函数每当用户实例被创建的时候被调用。

{title="src/models/user.js",lang="javascript"}

```
const user = (sequelize, DataTypes) => {
  const User = sequelize.define('user', {
    ...
  });

  ...

# leanpub-start-insert
  User.beforeCreate(user => {
    ...
  });
# leanpub-end-insert

  return User;
};

export default user;
```

> In this hook function, add the functionalities to alter your user entity's properties before they reach the database. Let's do it for the hashed password by using bcrypt.

在这个钩子函数中，添加修改用户实例属性的功能在其被存储到数据库前。让我们使用 bcrypt 来对密码进行哈希运算。

{title="src/models/user.js",lang="javascript"}

```
# leanpub-start-insert
import bcrypt from 'bcrypt';
# leanpub-end-insert

const user = (sequelize, DataTypes) => {
  const User = sequelize.define('user', {
    ...
  });

  ...

# leanpub-start-insert
  User.beforeCreate(async user => {
    user.password = await user.generatePasswordHash();
  });
# leanpub-end-insert

# leanpub-start-insert
  User.prototype.generatePasswordHash = async function() {
    const saltRounds = 10;
    return await bcrypt.hash(this.password, saltRounds);
  };
# leanpub-end-insert

  return User;
};

export default user;
```

> The bcrypt `hash()` method takes a string--the user's password--and an integer called salt rounds. Each salt round makes it more costly to hash the password, which makes it more costly for attackers to decrypt the hash value. A common value for salt rounds nowadays ranged from 10 to 12, as increasing the number of salt rounds might cause performance issues both ways.

bcrypt 的 `hash()` 方法接受一个字符串--用户密码和一个叫 salt rounds 的整数。每次 salt 循环使得密码哈希成本更高，同时这也使得攻击者解密哈希值的成本更高。现在一般的 salt rounds 的范围是 10 到 12，因为增加 salt rounds 的范围可能会同时在哈希和解密的过程中导致性能问题。

> In this implementation, the `generatePasswordHash()` function is added to the user's prototype chain. That's why it is possible to execute the function as method on each user instance, so you have the user itself available within the method as `this`. You can also take the user instance with its password as an argument, which I prefer, though using JavaScript's prototypal inheritance a good tool for any web developer. For now, the password is hashed with bcrypt before it gets stored every time a user is created in the database,.

在这个实现中，`generatePasswordHash()` 函数被添加到用户的原型链中。这就是为什么我们可以在每一个用户的实例中执行这个函数，所以你可以在这个函数中通过 `this` 访问到这个用户本身。你也可以把包含密码的用户实例作为参数，我更喜欢这样做，尽管对于任何 Web 开发者来说使用 JavaScript 的原型链继承是一个好的工具。现在，每一个用户在数据库中被创建的时候，他的密码都是通过 bcrypt 哈希运算过的。

### Token based Authentication in GraphQL

### 在 GraphQL 中实现基于 Token 的认证

> We still need to implement the token based authentication. So far, there is only a placeholder in your application for creating the token that is returned on a sign up and sign in mutation. A signed in user can be identified with this token, and is allowed to read and write data from the database. Since a registration will automatically lead to a login, the token is generated in both phases.

我们仍然需要实现基于 token 的认证。目前为止，在你的应用中只有一个占位符用于创建被注册和登录变更操作返回的 token。一个已经登录的用户可以被这个 token 标识，并被允许从数据库读写数据。因为用户注册后会自动登录，所以在两个阶段中都会生成 token。

> Next are the implementation details for the token-based authentication in GraphQL. Regardless of GraphQL, you are going to use a [JSON web token (JWT)](https://jwt.io/) to identify your user. The definition for a JWT from the official website says: _JSON Web Tokens are an open, industry standard RFC 7519 method for representing claims securely between two parties._ In other words, a JWT is a secure way to handle the communication between two parties (e.g. a client and a server application). If you haven't worked on security related applications before, the following section will guide you through the process, and you'll see the token is just a secured JavaScript object with user information.

接下来是在 GraphQL 中基于 token 认证的实现细节。与 GraphQL 无关，你将会用 [JSON web token (JWT)](https://jwt.io/) 去识别你的用户。JWT 官方网站给出的定义说：_JSON Web Tokens 是一种开放的，行业标准为 RFC 7519 的，为了在双方之间安全的表示请求的方法。_ 换句话说，JWT 是一种处理两端之间通信的安全方法（例如客户端和服务端）。如果你之前没有在安全相关的应用上工作过，如下部分将会引导你完成整个过程，同时你也会明白 token 只是一种安全的带有用户信息的 JavaScript 对象。

> To create JWT in this application, we'll use the popular [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) node package. Install it on the command line:

为了在这个应用中创建 JWT，你将会使用流行的 [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) 的 node 包。在命令行中将其安装。

{title="Command Line",lang="json"}

```
npm install jsonwebtoken --save
```

> Now, import it in your _src/resolvers/user.js_ file and use it to create the token:

现在，将其倒入到 _src/resolvers/user.js_ 文件并用它来创建 token：

{title="src/resolvers/user.js",lang="javascript"}

```
# leanpub-start-insert
import jwt from 'jsonwebtoken';
# leanpub-end-insert

const createToken = async user => {
# leanpub-start-insert
  const { id, email, username } = user;
  return await jwt.sign({ id, email, username });
# leanpub-end-insert
};

...
```

> The first argument to "sign" a token can be any user information except sensitive data like passwords, because the token will land on the client side of your application stack. Signing a token means putting data into it, which you've done, and securing it, which you haven't done yet. To secure your token, pass in a secret (**any** long string) that is **only available to you and your server**. No third-party entities should have access, because it is used to encode (sign) and decode your token.

“签署” token 的第一个参数可以是除敏感数据之外的任何用户信息（例如密码），因为 token 将会被客户端获取。签署一个 token 意味着将数据放入其中，这个你已经做了，并保护它，这个你尚未完成。为了保护你的 token，传入一个密钥（**任意** 长字符串）**只能被你和你的服务器使用**。任何第三方实体都不应该具有访问权限，因为这个密钥将会被用来加密（签署）和解密你的 token。

> Add the secret to your environment variables in the *.env* file:

在 *.env* 文件中，将密钥添加到你的环境变量：

{title=".env",lang="javascript"}

```
DATABASE=postgres
DATABASE_USER=postgres
DATABASE_PASSWORD=postgres

# leanpub-start-insert
SECRET=wr3r23fwfwefwekwself.2456342.dawqdq
# leanpub-end-insert
```

> Then, in the _src/index.js_ file, pass the secret via Apollo Server's context to all resolver functions:

然后，在文件 _src/index.js_ 中，通过 Apollo 服务端的上下文将密钥传入到所有的解析函数：

{title="src/index.js",lang="javascript"}

```
const server = new ApolloServer({
  typeDefs: schema,
  resolvers,
  ...
  context: async () => ({
    models,
    me: await models.User.findByLogin('rwieruch'),
# leanpub-start-insert
    secret: process.env.SECRET,
# leanpub-end-insert
  }),
});
```

> Next, use it in your `signUp` resolver function by passing it to the token creation. The `sign` method of JWT handles the rest. You can also pass in a third argument for setting an expiration time or date for a token. In this case, the token is only valid for 30 minutes, after which a user has to sign in again.

接下来，通过将其传递给 token 的创建方法，在 `signUp` 解析函数中使用它。JWT 的 `sign` 方法处理剩下的事情。你也可以传入第三个参数来设置 token 的过期时间或者日期。在这个例子中，token 只有 30 分钟的合法时间，之后用户必须再次登录。

{title="src/resolvers/user.js",lang="javascript"}

```
# leanpub-start-insert
import jwt from 'jsonwebtoken';
# leanpub-end-insert

# leanpub-start-insert
const createToken = async (user, secret, expiresIn) => {
# leanpub-end-insert
  const { id, email, username } = user;
# leanpub-start-insert
  return await jwt.sign({ id, email, username }, secret, {
    expiresIn,
  });
# leanpub-end-insert
};

export default {
  Query: {
    ...
  },

  Mutation: {
    signUp: async (
      parent,
      { username, email, password },
# leanpub-start-insert
      { models, secret },
# leanpub-end-insert
    ) => {
      const user = await models.User.create({
        username,
        email,
        password,
      });

# leanpub-start-insert
      return { token: createToken(user, secret, '30m') };
# leanpub-end-insert
    },
  },

  ...
};
```

> Now you have secured your information in the token as well. If you would want to decode it, in order to access the secured data (the first argument of the `sign` method), you would need the secret again. Furthermore, the token is only valid for 30 minutes.

现在你已经将你的信息保护在了 token 中。如果你想解密它，为了获取保护的数据（`sign` 方法的第一个参数），你将再次需要这个密钥。此外，这个 token 的合法时间只有三十分钟。

> That's it for the registration: you are creating a user and returning a valid token that can be used from the client application to authenticate the user. The server can decode the token that comes with every request and allows the user to access sensitive data. You can try out the registration with GraphQL Playground, which should create a user in the database and return a token for it. Also, you can check your database with `psql` to test if the use was created and with a hashed password.

这就是注册：你正在创建一个用户并返回一个合法的 token，可以从客户端应用使用该 token 来认证用户。服务器可以解密每一个从请求附带的 token 并允许用户访问敏感的数据。你可以尝试使用 GraphQL Playground 注册，这个应该可以在数据库中创建一个用户并为其返回一个 token。此外，你可以使用 `psql` 检查你的数据库来判断用户是否被创建并且包含一个经过哈希计算过的密码。

### Login (Sign In) with GraphQL

### 用 GraphQL 实现登录

> Before you dive into the authorization with the token on a per-request basis, let's implement the second mutation for the authentication mechanism: the `signIn` mutation (or login mutation). Again, first we add the GraphQL mutation to your user's schema in the _src/schema/user.js_ file:

在你深入将基于 token 的认证应用于每一个请求之前，让我们为了认证机制实现第二个变更操作：`signIn` 变更操作（或者登录变更）。同样，首先我们添加 GraphQL 变更操作到你用户的模式，在文件 _src/schema/user.js_ 中：

{title="src/schema/user.js",lang="javascript"}

```
import { gql } from 'apollo-server-express';

export default gql`
  ...

  extend type Mutation {
    signUp(
      username: String!
      email: String!
      password: String!
    ): Token!

# leanpub-start-insert
    signIn(login: String!, password: String!): Token!
# leanpub-end-insert
  }

  type Token {
    token: String!
  }

  ...
`;
```

> Second, add the resolver counterpart to your _src/resolvers/user.js_ file:

然后，添加解析副本到你的 _src/resolvers/user.js_ 文件：

{title="src/resolvers/user.js",lang="javascript"}

```
import jwt from 'jsonwebtoken';
# leanpub-start-insert
import { AuthenticationError, UserInputError } from 'apollo-server';
# leanpub-end-insert

...

export default {
  Query: {
    ...
  },

  Mutation: {
    signUp: async (...) => {
      ...
    },

# leanpub-start-insert
    signIn: async (
      parent,
      { login, password },
      { models, secret },
    ) => {
      const user = await models.User.findByLogin(login);

      if (!user) {
        throw new UserInputError(
          'No user found with this login credentials.',
        );
      }

      const isValid = await user.validatePassword(password);

      if (!isValid) {
        throw new AuthenticationError('Invalid password.');
      }

      return { token: createToken(user, secret, '30m') };
    },
# leanpub-end-insert
  },

  ...
};
```

> Let's go through the new resolver function for the login step by step. As arguments, the resolver has access to the input arguments from the GraphQL mutation (login, password) and the context (models, secret). When a user tries to sign in to your application, the login, which can be either the unique username or unique email, is taken to retrieve a user from the database. If there is no user, the application throws an error that can be used in the client application to notify the user. If there is an user, the user's password is validated. You will see this method on the user model in the next example. If the password is not valid, the application throws an error to the client application. If the password is valid, the `signIn` mutation returns a token identical to the `signUp` mutation. The client application either performs a successful login or shows an error message for invalid credentials. You can also see specific Apollo Server Errors used over generic JavaScript Error classes.

让我们逐步完成登录的新解析函数。作为参数，解析函数拥有访问 GraphQL 变更（login，password）和上下文（models，secret）的输入参数。当用户尝试登录你的应用，登录，可以是唯一的用户名或者是唯一的电子邮件，将会被用于从数据库中获取用户。如果没有该用户，应用将会抛出一个错误用于客户端提示用户。如果用户存在，用户的密码将会被验证。你将会在下一个例子的用户模型中看到这个方法。如果密码不合法，应用抛出一个错误给客户端应用。如果密码合法，`signIn` 变更操作返回一个同 `signUp` 变更操作一样的 token。客户端应用要么登录成功要么给不合法的凭据显示一个错误信息。你还可以看到在通用 JavaScript Error 类上使用特殊的 Apollo 服务端错误。

> Next, we want to implement the `validatePassword()` method on the user instance. Place it in the _src/models/user.js_ file, because that's where all the model methods for the user are stored, same as the `findByLogin()` method.

接下来，我们要在用户实例上实现 `validatePassword()` 方法。将其放在 _src/models/user.js_ 文件中，因为这是所有用户模型方法放置的地方，同 `findByLogin()` 方法一样。

{title="src/models/user.js",lang="javascript"}

```
import bcrypt from 'bcrypt';

const user = (sequelize, DataTypes) => {
  ...

  User.findByLogin = async login => {
    let user = await User.findOne({
      where: { username: login },
    });

    if (!user) {
      user = await User.findOne({
        where: { email: login },
      });
    }

    return user;
  };

  User.beforeCreate(async user => {
    user.password = await user.generatePasswordHash();
  });

  User.prototype.generatePasswordHash = async function() {
    const saltRounds = 10;
    return await bcrypt.hash(this.password, saltRounds);
  };

# leanpub-start-insert
  User.prototype.validatePassword = async function(password) {
    return await bcrypt.compare(password, this.password);
  };
# leanpub-end-insert

  return User;
};

export default user;
```

> Again, it's a prototypical JavaScript inheritance for making a method available in the user instance. In this method, the user (this) and its password can be compared with the incoming password from the GraphQL mutation using bcrypt, because the password on the user is hashed, and the incoming password is plain text. Fortunately, bcrypt will tell you whether the password is correct or not when a user signs in.

同样，它是一个典型的 JavaScript 继承让一个方法可以在用户实例上可用。在这个方法中，用户（this）和其密码可以和从 GraphQL 变更操作传入的密码一起使用 bcrypt 进行比较。因为用户的密码是加过密的，传入的密码是普通文本。幸运的是，在用户登录的时候，bcrypt 将会告诉你密码是否是正确的。

> Now you have set up registration (sign up) and login (sign in) for your GraphQL server application. You used bcrypt to hash and compare a plain text password before it reaches the database with a Sequelize hook function, and you used JWT to encrypt user data with a secret to a token. Then the token is returned on every sign up and sign in. Then the client application can save the token (e.g. local storage of the browser) and send it along with every GraphQL query and mutation as authorization.

现在你已经给你的 GraphQL 服务器应用设置好了注册和登录。你通过使用 Sequelize 钩子函数对即将到达数据库的明文密码使用 bcrypt 进行哈希和比较，同时你使用 JWT 和一个密钥将用户数据加密到一个 token。然后在每次注册和登录的时候返回这个 token。然后客户端应用可以保存 token（例如浏览器本地存储）并做为认证随同每一个 GraphQL 查询操作和变更操作一起发送。

> The next section will teach you about authorization in GraphQL on the server-side, and what should you do with the token once a user is authenticated with your application after a successful registration or login.

下一节将会向你介绍关于服务器端的 GraphQL 授权，和当成功注册或者登录并得到应用的认证过后，可以用 token 做什么。

> ### Exercises:

### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/831ab566f0b5c5530d9270a49936d102f7fdf73c)
> * Register (sign up) a new user with GraphQL Playground
> * Check your users and their hashed passwords in the database with `psql`
> * Read more about [JSON web tokens (JWT)](https://jwt.io/)
> * Login (sign in) a user with GraphQL Playground
>  * copy and paste the token to the interactive token decoding on the JWT website (conclusion: the information itself isn't secure, that's why you shouldn't put a password in the token)

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/831ab566f0b5c5530d9270a49936d102f7fdf73c)
* 用 GraphQL Playground 注册一个新用户
* 用 `psql` 在数据库中检查你的用户和其加密的密码
* 延伸阅读: [JSON web tokens (JWT)](https://jwt.io/)
* 用 GraphQL Playground 登录一个用户
  * 复制和粘贴 token 到 JWT 网站上的交互式 token 解密（总结：信息本身并没有被保护，这就是为什么你不能将你的密码放在你的 token 中）

> ## Authorization with GraphQL and Apollo Server
## 使用 GraphQL 和 Apollo 服务端进行授权

> In the last section, you set up GraphQL mutations to enable authentication with the server. You can register a new user with bcrypt hashed passwords and you can login with your user's credentials. Both GraphQL mutations related to authentication return a token (JWT) that secures non-sensitive user information with a secret.

在上一节中，你已经设置了 GraphQL 变更操作以启用服务器的身份验证。你可以使用 bcrypt 哈希密码注册新用户，并且用你的用户凭证进行登录。这两个涉及身份认证的 GraphQL 变更都返回了一个使用密钥来保护非敏感用户信息的 token（JWT）

> The token, whether its obtained on registration or login, is returned to the client application after a successful GraphQL `signIn` or `signUp` mutation. The client application must store the token somewhere like [the browser's session storage](https://www.robinwieruch.de/local-storage-react). Every time a request is made to the GraphQL server, the token has to be attached to the HTTP header of the HTTP request. The GraphQL server can then validate the HTTP header, verify its authenticity, and perform a request like a GraphQL operation. If the token is invalid, the GraphQL server must return an error for the GraphQL client. If the client still has a token locally stored, it should remove the token and redirect the user to the login page.

无论是在注册还是登录时获得的 token，在 GraphQL signIn 或 signUp 变更（操作）成功后都会被返回给客户端。客户端必须存储 token，例如[浏览器的会话存储](https://www.robinwieruch.de/local-storage-react)。每次向 GraphQL server 发起请求时，必须将 token 附加到 HTTP 请求头里面。GraphQL server 接收到请求后，可以校验 HTTP 请求头，验证其真实性，并执行类似 GraphQL 操作的请求。如果 token 无效，则 GraphQL server 必须向客户端返回一个错误。如果客户端仍然本地存储着 token，则应删除 token 并重定向到登录页面。

> Now we just need to perform the server part of the equation. Let's do it in the *src/index.js* file by adding a global authorization that verifies the incoming token before the request hits the GraphQL resolvers.

现在我们只需要执行方法的服务器部分。让我们在 *src/index.js* 文件中添加一个全局授权，在请求到达 GraphQL 解析器之前先验证传入的 token。

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import jwt from 'jsonwebtoken';
# leanpub-end-insert
import {
  ApolloServer,
# leanpub-start-insert
  AuthenticationError,
# leanpub-end-insert
} from 'apollo-server-express';
...

# leanpub-start-insert
const getMe = async req => {
  const token = req.headers['x-token'];

  if (token) {
    try {
      return await jwt.verify(token, process.env.SECRET);
    } catch (e) {
      throw new AuthenticationError(
        'Your session expired. Sign in again.',
      );
    }
  }
};
# leanpub-end-insert

const server = new ApolloServer({
  typeDefs: schema,
  resolvers,
  ...
# leanpub-start-insert
  context: async ({ req }) => {
    const me = await getMe(req);

    return {
      models,
      me,
      secret: process.env.SECRET,
    };
  },
# leanpub-end-insert
});

...
~~~~~~~~

> In this general authorization on the server-side, you are injecting the `me` user, the authenticated user from the token, with every request to your Apollo Server's context. The `me` user is encoded in the token in the `createToken()` function. It's not a user from the database anymore, which spares the additional database request.

在服务端的全局授权中，你将把通过 token 身份校验的用户 `me` 注入向 Apollo 服务端发起的每个请求中。在 token 中的用户 `me` 会被 createToken() 函数编码。它不再是数据库中的一个用户，这可以避免额外的数据库请求。

> In the `getMe()` function, you extract the HTTP header for the authorization called "x-token" from the incoming HTTP request. The GraphQL client application sends the token obtained from the registration or login with every other request in an HTTP header, along with the payload of the HTTP request (e.g. GraphQL operation). It can then be checked to see if there is such an HTTP header in the function or not. If not, the function continues with the request, but the `me` user is undefined. If there is a token, the function verifies the token with its secret and retrieves the user information that was stored when you created the token. If the verification fails because the token was invalid or expired, the GraphQL server throws a specific Apollo Server Error. If the verification succeeds, the function continues with the `me` user defined.

在 `getMe()` 函数内，你从传入的 HTTP 请求中提取名为 "x-token" 的请求头以便进行授权校验 。在每次请求时，GraphQL client 都会把在注册或登录时获取的 token 及其他有效数据（例如 GraphQL 操作）一起放在 HTTP 请求头中发送。然后可以检查在函数中是否存在这样的 HTTP 请求头。如果没有，该函数继续往下执行，但用户 `me` 是未定义的。如果 token 存在，则该函数会使用其密钥验证 token，并检索创建 token 时存储的用户信息。如果因 token 无效或过期导致验证失败，GraphQL 服务端会抛出特定的 Apollo Server 错误。如果验证成功，则函数将继续使用经过验证的用户 `me`。

> The function returns an error when the client application sends an HTTP header with an invalid or expired token. Otherwise, the function waves the request through, because users must be checked at the resolver level to see if they're allowed to perform certain actions. A non-authenticated user--where the `me` user is undefined--might be able to retrieve messages but not create new ones. The application is now protected against invalid and expired tokens.

当 GraphQL 客户端发送带有无效或过期 token 的 HTTP 请求时，该函数返回错误。否则，该函数会通过请求，因为必须在解析器层校验是否允许用户执行某些操作。一个未经身份验证的用户 -- 此处用户 `me` 是未定义的 -- 也许能够查看消息，但不能创建新消息。现在，我们的 GraphQL 服务端可以防止无效和过期的 token 了。

> That's the most high-level authentication for your GraphQL server application. You are able to authenticate with your GraphQL server from a GraphQL client application with the `signUp` and `signIn` GraphQL mutations, and the GraphQL server only allows valid, non-expired tokens from the GraphQL client application.

这是 GraphQL 服务端应用的最高级别身份验证。你可以使用 GraphQL 服务端对具有 `signUp` 和 `signIn` 变更操作的 GraphQL 客户端进行身份验证，并且 GraphQL 服务端仅允许来自 GraphQL 客户端的有效的，未过期的 token 通过验证。

> ### GraphQL Authorization on a Resolver Level
### 解析器层的 GraphQL 授权

> A GraphQL HTTP request comes through the `getMe()` function, even if it has no HTTP header for a token. This is good default behavior, because you want to register new users and login to the application without a token for now. You might want to query messages or users without being authenticated with the application. It is acceptable and sometimes necessary to wave through some requests without authorization token, to grant different levels of access to different user types. There will be an error only when the token becomes invalid or expires.

一个 GraphQL HTTP 请求，即使它的请求头里没有 token，也会经过 getMe() 函数。这是一个很好的默认行为，因为你现在想要注册新用户并在没有 token 的情况下登录应用程序。你可能希望在未经授权验证的情况下查询消息或用户。在没有授权 token 的情况下允许用户的某些请求是可接受的，有时甚至是必要的，这样可以对不同用户类型授予不同级别的访问权限，仅当 token 失效或过期时 getMe() 函数才会返回错误。

> However, certain GraphQL operations should have more specific authorizations. Creating a message should only be possible for authorized users. Otherwise, or there would be no way to track the messages' authors. The `createMessage` GraphQL mutation can be protected, or "guarded", on a GraphQL resolver level. The naive approach of protecting the GraphQL operation is to guard it with an if-else statement in the *src/resolvers/message.js* file:

但是，某些 GraphQL 操作应具有更多特定授权，比如，只有授权用户才能创建消息。否则无法追踪消息的创建者。我们可以在 GraphQL 解析器层保护 `createMessage` GraphQL 变更操作，最简单的方法是在 *src/resolvers/message.js* 文件中的使用 if-else 语句：

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { ForbiddenError } from 'apollo-server';
# leanpub-end-insert

export default {
  Query: {
    ...
  },

  Mutation: {
    createMessage: async (parent, { text }, { me, models }) => {
# leanpub-start-insert
      if (!me) {
        throw new ForbiddenError('Not authenticated as user.');
      }
# leanpub-end-insert

      return await models.Message.create({
        text,
        userId: me.id,
      });
    },

    ...
  },

  ...
};
~~~~~~~~

> You can imagine how this becomes  and error prone if it is userepetitived for all GraphQL operations that are accessible to an authenticated user, as it mixes lots of authorization logic into the resolver functions. To remedy this, we introduce an authorization abstraction layer for protecting GraphQL operations, with solutions called **combined resolvers** or **resolver middleware**. Let's install this node package:

你可以想象，如果它被用于授权所有经过身份验证的用户才可访问的 GraphQL 操作，将会变得多么重复且容易出错，因为它将大量授权逻辑混合到解析函数中。为了解决这个问题，我们引入了一个授权抽象层来保护 GraphQL 操作，使用名为 **combined resolvers** 或 **resolver middleware** 的解决方案。我们来安装这个包：

{title="Command Line",lang="json"}
~~~~~~~~
npm install graphql-resolvers --save
~~~~~~~~

> Let's implement a protecting resolver function with this package in a new *src/resolvers/authorization.js* file. It should only check whether there is a `me` user or not.

让我们在新建的 *src/resolvers/authorization.js* 文件中，使用这个包来实现一个保护解析函数。它只用来检验是否是当前用户 `me`。

{title="src/resolvers/authorization.js",lang="javascript"}
~~~~~~~~
import { ForbiddenError } from 'apollo-server';
import { skip } from 'graphql-resolvers';

export const isAuthenticated = (parent, args, { me }) =>
  me ? skip : new ForbiddenError('Not authenticated as user.');
~~~~~~~~

> The `isAuthenticated()` resolver function acts as middleware, either continuing with the next resolver (skip), or performing another action, like returning an error. In this case, an error is returned when the `me` user is not available. Since it is a resolver function itself, it has the same arguments as a normal resolver. A guarding resolver can be used when a message is created in the *src/resolvers/message.js* file. Import it with the `combineResolvers()` from the newly installed node package. The new resolver is used to protect the resolvers by combining them.

这里的解析函数 `isAuthenticated()` 充当中间件，要么执行 skip，要么执行其他操作比如返回错误。在这种情况下，当用户 `me` 不可用时会返回错误。由于它本身就是一个解析函数，因此它具有与普通解析函数相同的参数。所以在 *src/resolvers/message.js* 文件中创建消息时，也可以使用这个保护解析函数。引入并使用新安装的包中的 `combineResolvers()`，通过组合一个新的解析器来保护它们。

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { combineResolvers } from 'graphql-resolvers';
# leanpub-end-insert

# leanpub-start-insert
import { isAuthenticated } from './authorization';
# leanpub-end-insert

export default {
  Query: {
    ...
  },

  Mutation: {
# leanpub-start-insert
    createMessage: combineResolvers(
      isAuthenticated,
      async (parent, { text }, { models, me }) => {
        return await models.Message.create({
          text,
          userId: me.id,
        });
      },
    ),
# leanpub-end-insert

    ...
  },

  ...
};
~~~~~~~~

> Now the `isAuthenticated()` resolver function always runs before the resolver that creates the message associated with the authenticated user in the database. The resolvers get chained to each other, and you can reuse the protecting resolver function wherever you need it. It only adds a small footprint to your actual resolvers, which can be changed in the *src/resolvers/authorization.js* file.

现在，`isAuthenticated()` 解析函数始终在创建消息的解析器之前运行。这些解析器相互链接，你可以在任何需要的地方重用这个保护性质的解析器函数。而所需要做的仅仅是在 *src/resolvers/authorization.js* 文件中，在真正执行任务的解析器前做了一点小小的改动。

> ### Permission-based GraphQL Authorization
### 基于权限的 GraphQL 授权

> The previous resolver only checks if a user is authenticated or not, so it is only applicable to the higher level. Cases like permissions require another protecting resolver that is more specific than the one in the *src/resolvers/authorization.js* file:

之前的解析器仅用于检查用户是否通过身份验证，因此它只适用于更高级别的的授权。像权限这样的情况需要另一个保护解析器，它比 *src/resolvers/authorization.js* 文件中的解析器更具体：

{title="src/resolvers/authorization.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
export const isMessageOwner = async (
  parent,
  { id },
  { models, me },
) => {
  const message = await models.Message.findById(id, { raw: true });

  if (message.userId !== me.id) {
    throw new ForbiddenError('Not authenticated as owner.');
  }

  return skip;
};
# leanpub-end-insert
~~~~~~~~

> This resolver checks whether the authenticated user is the message owner. It's a useful check before deleting a message, since you only want the message creator to be able to delete it. The guarding resolver retrieves the message by id, checks the message's associated user with the authenticated user, and either throws an error or continues with the next resolver.

这个解析器用于检查已认证用户是否为当前消息的创建者。鉴于当前信息只能被它的创建者删除，所以在删除信息之前，这是一个必要的检查。保护解析器通过 id 检索消息，并检查与消息关联的已认证用户信息，之后抛出错误或继续执行下一个解析程序。

Let's protect a resolver with this fine-tuned authorization permission resolver in the *src/resolvers/message.js* file:

让我们在 *src/resolvers/message.js*文件中使用这个经过微调的授权权限解析器来保护解析器：

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
import { combineResolvers } from 'graphql-resolvers';

# leanpub-start-insert
import { isAuthenticated, isMessageOwner } from './authorization';
# leanpub-end-insert

export default {
  Query: {
    ...
  },

  Mutation: {
    ...

# leanpub-start-insert
    deleteMessage: combineResolvers(
      isMessageOwner,
      async (parent, { id }, { models }) => {
        return await models.Message.destroy({ where: { id } });
      },
    ),
# leanpub-end-insert
  },

  ...
};
~~~~~~~~

> The `deleteMessage` resolver is protected by an authorization resolver now. Only the message owner, i.e. the message creator, is allowed to delete a message. If the user isn't authenticated, you can stack your protecting resolvers onto each other:

`deleteMessage` 现在被授权解析程序保护起来了，它仅允许消息所有者，也就是消息创建者，删除消息。如果用户未经过身份验证，会跳过 isAuthenticated 继续执行其他保护解析器：

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
import { combineResolvers } from 'graphql-resolvers';

import { isAuthenticated, isMessageOwner } from './authorization';

export default {
  Query: {
    ...
  },

  Mutation: {
    ...

    deleteMessage: combineResolvers(
# leanpub-start-insert
      isAuthenticated,
# leanpub-end-insert
      isMessageOwner,
      async (parent, { id }, { models }) => {
        return await models.Message.destroy({ where: { id } });
      },
    ),
  },

  ...
};
~~~~~~~~

> As an alternate tactic, you can also use the `isAuthenticated` resolver directly in the `isMessageOwner` resolver; then, you can avoid handling it in the actual resolver for deleting a message. I find being explicit to be more practical than hiding knowledge within the authorization resolver. The alternative route is still explained in the role-based authorization section, however.

作为替代策略，你还可以直接在 `isMessageOwner` 解析器中使用 `isAuthenticated` 解析器；你可以避免在当前的解析器中处理它。我发现在授权解析器中，显式调用比隐式调用更实际。不过, 另一种路线仍在基于角色的授权部分进行了解释。

> The second combined resolver is for permission checks, because it decides whether or not the user has permission to delete the message. This is just one way of doing it, though. In other cases, the message could carry a boolean flag that decides if the active user has certain permissions.

第二个组合解析器用于权限检查，因为它决定用户是否有删除消息的权限。不过，这只是一种方法。在其他情况下，消息可以携带一个布尔值的标志，该标志决定当前用户是否具有某些权限。

> ### Role-based GraphQL Authorization
### 基于角色的 GraphQL 授权

> We went from a high-level authorization to a more specific authorization with permission-based resolver protection. Now we'll cover yet another way to enable authorization called **roles**. The next code block is a GraphQL mutation that requires role-based authorization, because it has the ability to delete a user. This allows you to create users with admin roles.

我们从高级授权转变为更具体的基于权限的解析程序保护的授权。现在我们将介绍另一种称为 **角色** 的方法来进行授权。下个代码块是基于角色授权的 GraphQL 变更操作，因为它具有删除用户的能力。这允许你创建具有管理角色的用户。

> Let's implement the new GraphQL mutation first, followed by the role-based authorization. You can start in your *src/resolvers/user.js* file with a resolver function that deletes a user in the database by identifier:

首先，让我们来实现新的 GraphQL 变更操作，然后是基于角色的授权。你可以在 *src/resolvers/user.js* 中开始写一个解析函数，该函数通过标识符删除数据库中的用户：

{title="src/resolvers/user.js",lang="javascript"}
~~~~~~~~
...

export default {
  Query: {
    ...
  },

  Mutation: {
    ...

# leanpub-start-insert
    deleteUser: async (parent, { id }, { models }) => {
      return await models.User.destroy({
        where: { id },
      });
    },
# leanpub-end-insert
  },

  ...
};
~~~~~~~~

> New GraphQL operations must be implemented in the resolvers and schema. Next, we'll add the new mutation in the *src/schema/user.js* file. It returns a boolean that tells you whether the deletion was successful or not:

新的 GraphQL 操作必须在解析器和模式中实现。接下来，我们将在 *src/schema/user.js* 文件中添加新的变更操作。它返回一个布尔值，告诉你是否删除成功：

{title="src/schema/user.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  extend type Query {
    ...
  }

  extend type Mutation {
    signUp(
      username: String!
      email: String!
      password: String!
    ): Token!

    signIn(login: String!, password: String!): Token!
# leanpub-start-insert
    deleteUser(id: ID!): Boolean!
# leanpub-end-insert
  }

  ...
`;
~~~~~~~~

> Before you can implement role-based protections for it, you must introduce the actual roles for the user entities. Add a `role` entry to your user's entity in the *src/models/user.js* file:

在实现基于角色的保护之前，你必须为用户实体引入实际的角色。你需要在 *src/models/user.js* 文件中为用户的实体添加一个 `角色` 入口：

{title="src/models/user.js",lang="javascript"}
~~~~~~~~
...

const user = (sequelize, DataTypes) => {
  const User = sequelize.define('user', {
    ...
    password: {
      type: DataTypes.STRING,
      allowNull: false,
      validate: {
        notEmpty: true,
        len: [7, 42],
      },
    },
# leanpub-start-insert
    role: {
      type: DataTypes.STRING,
    },
# leanpub-end-insert
  });

  ...

  return User;
};

export default user;
~~~~~~~~

> Add the role to your GraphQL user schema in the *src/schema/user.js* file too:

将角色也添加到 *src/schema/user.js* 文件中的 GraphQL 用户模式中：

{title="src/schema/user.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  ...

  type User {
    id: ID!
    username: String!
    email: String!
# leanpub-start-insert
    role: String
# leanpub-end-insert
    messages: [Message!]
  }
`;
~~~~~~~~

> Since you already have seed data in your *src/index.js* file for two users, you can give one of them a role. The admin role used in this case will be checked if the user attempts a delete operation:

*src/index.js* 文件中，已经设置了两个用户数据，你可以为其中一个用户提供一个角色。如果用户尝试进行删除，将会校验他是否具有管理员角色：

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

const createUsersWithMessages = async () => {
  await models.User.create(
    {
      username: 'rwieruch',
      email: 'hello@robin.com',
      password: 'rwieruch',
# leanpub-start-insert
      role: 'ADMIN',
# leanpub-end-insert
      messages: [
        {
          text: 'Published the Road to learn React',
        },
      ],
    },
    {
      include: [models.Message],
    },
  );

  ...
};
~~~~~~~~

> Because you are not retrieving the actual `me` user from the database in the *src/index.js* file, but the user from the token instead, you must add the role information of the user for the token when it's created in the *src/resolvers/user.js* file:

你不是从 *src/index.js* 关联的数据库中检索实际的 `me` 用户，而是从 token 中检索用户，所以在 *src/resolvers/user.js* 中创建 token 时，必须为用户添加角色信息：

{title="src/resolvers/user.js",lang="javascript"}
~~~~~~~~
const createToken = async (user, secret, expiresIn) => {
# leanpub-start-insert
  const { id, email, username, role } = user;
  return await jwt.sign({ id, email, username, role }, secret, {
# leanpub-end-insert
    expiresIn,
  });
};
~~~~~~~~

> Next, protect the new GraphQL mutation with a role-based authorization. Create a new guarding resolver in your *src/resolvers/authorization.js* file:

下一步，使用基于角色的授权保护新的 GraphQL 变更操作。在 *src/resolvers/authorization.js* 文件中创建一个新的保护解析器：

{title="src/resolvers/authorization.js",lang="javascript"}
~~~~~~~~
import { ForbiddenError } from 'apollo-server';
# leanpub-start-insert
import { combineResolvers, skip } from 'graphql-resolvers';
# leanpub-end-insert

export const isAuthenticated = (parent, args, { me }) =>
  me ? skip : new ForbiddenError('Not authenticated as user.');

# leanpub-start-insert
export const isAdmin = combineResolvers(
  isAuthenticated,
  (parent, args, { me: { role } }) =>
    role === 'ADMIN'
      ? skip
      : new ForbiddenError('Not authorized as admin.'),
);
# leanpub-end-insert

export const isMessageOwner = async (
  parent,
  { id },
  { models, me },
) => {
  const message = await models.Message.findById(id, { raw: true });

  if (message.userId !== me.id) {
    throw new ForbiddenError('Not authenticated as owner.');
  }

  return skip;
};
~~~~~~~~

> The new resolver checks to see if the authenticated user has the `ADMIN` role. If it doesn't, the resolver returns an error; if it does, the next resolver is called. Unlike the `isMessageOwner` resolver, the `isAdmin` resolver is already combined, using the `isAuthenticated` resolver. Put this check in your actual resolver, which you are going to protect in the next step:

新的解析器将校验用户是否具有 `ADMIN` 角色。如果没有，则解析器返回错误；如果有，则继续调用下一个解析器。与`isMessageOwner` 解析器不同，`isAdmin` 解析器已经使用 `isAuthenticated` 解析器进行组合。将这个校验放入你的实际解析器中，你将在下一步中保护它：

{title="src/resolvers/user.js",lang="javascript"}
~~~~~~~~
import jwt from 'jsonwebtoken';
# leanpub-start-insert
import { combineResolvers } from 'graphql-resolvers';
# leanpub-end-insert
import { AuthenticationError, UserInputError } from 'apollo-server';

# leanpub-start-insert
import { isAdmin } from './authorization';
# leanpub-end-insert

...

export default {
  Query: {
    ...
  },

  Mutation: {
    ...

# leanpub-start-insert
    deleteUser: combineResolvers(
      isAdmin,
      async (parent, { id }, { models }) => {
        return await models.User.destroy({
          where: { id },
        });
      },
    ),
# leanpub-end-insert
  },

  ...
};
~~~~~~~~

> That's the basics of role-based authorization in GraphQL with Apollo Server. In this example, the role is only a string that needs to be checked. In a more elaborate role-based architecture, the role might change from a string to an array that contains many roles. It eliminates the need for an equal check, since you can check to see if the array includes a targeted role. Using arrays with roles is the foundation for a sophisticated role-based authorization setup.

以上就是使用 GraphQL Apollo 服务端基于角色授权的基础知识。在示例中，角色只是需要检查的字符串。在更复杂的基于角色的体系结构中，角色可能会从字符串更改为包含许多角色的数组。这样就不用进行同等检查，而是需要检查数组里面是否包含目标角色。把角色设置为数组，便于处理复杂的基于角色的授权配置。

> ### Setting Headers in GraphQL Playground
### 在 GraphQL Playground 中设置 Headers

> You set up authorization for your GraphQL application, and now you just need to verify that it works. The simplest way to test this type of application is to use GraphQL Playground to run through different scenarios. The user deletion scenario will be used as an example, but you should test all the remaining scenarios for practice.

你已经为 GraphQL 应用设置好了授权，现在您只需要验证它能否工作。测试 GraphQL 应用最简单的方法是使用 GraphQL Playground 来运行不同的场景。这里我们将删除用户作示范，但你应该亲自练习运行所有剩余的实际场景。

> Before a user can perform a delete action, there must be a sign-in, so we execute a `signIn` mutation in GraphQL Playground with a non admin user. Consider trying this tutorial with an admin user later to see how it performs differently.

在用户执行删除操作之前，必须先登录，因此我们先用非管理员用户在 GraphQL Playground 中执行 `signIn` 变更操作，稍后我们再用管理员用户进行操作一遍，以区分它们的不同。

{title="GraphQL Playground",lang="json"}
~~~~~~~~
mutation {
  signIn(login: "ddavids", password: "ddavids") {
    token
  }
}
~~~~~~~~

> You should receive a token after logging into GraphQL Playground. The token needs to be set in the HTTP header for the next GraphQL operation. GraphQL Playground has a panel to add HTTP headers. Since your application is checking for an x-token, set the token as one:

登录 GraphQL Playground 后，你会收到一个 token，在接下来的 GraphQL 操作中都需要把这个 token 附到 HTTP 请求头里。GraphQL Playground 有一个用于添加 HTTP 请求头的面板。由于你的应用程序将检查 x-token 是否存在，请将 token 设置为：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "x-token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MiwiZW1haWwiOiJoZWxsb0BkYXZpZC5jb20iLCJ1c2VybmFtZSI6ImRkYXZpZHMiLCJpYXQiOjE1MzQ5MjM4NDcsImV4cCI6MTUzNDkyNTY0N30.ViGU6UUY-XWpWDJGfXqES2J1lEr-Uye8XDQ79lAvByE"
}
~~~~~~~~

> Your token will be different than the one above, but of a similar format. Since the token is set as an HTTP header now, you should be able to delete a user with the following GraphQL mutation in GraphQL Playground. The HTTP header with the token will be sent with the GraphQL operation:

你的 token 应该与上面的 token 不同，但格式类似。设置好 token，我们现在可以在 GraphQL Playground 上通过下面的 GraphQL 变更操作删除某个用户了。带有 token 的 HTTP 请求头将与 GraphQL 操作一起发送：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
mutation {
  deleteUser(id: "2")
}
~~~~~~~~

> Instead of a successful request, you will see the following GraphQL error after executing the GraphQL mutation for deleting a user. That's because you haven't logged in as a user with an admin role.

执行删除用户的 GraphQL 变更操作后，你看到的是 GraphQL 返回的错误，而不是成功的请求。那是因为你没有用管理员角色的用户登录。

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "data": null,
  "errors": [
    {
      "message": "Not authorized as admin.",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "path": [
        "deleteUser"
      ],
      "extensions": { ... }
    }
  ]
}
~~~~~~~~

> If you follow the same sequence as an admin user, you can delete a user entity successfully.

如果你以管理员角色登录，再按照之前的步骤操作，则可以成功删除一个用户实体。

![](images/authorization_1024.jpg)

> We've added basic authorization for this application. It has the global authorization before every request hits the GraphQL resolvers; and authorization at the resolver level with protecting resolvers. They check whether a user is authenticated, whether the user is able to delete a message (permission-based authorization), and whether a user is able to delete a user (role-based authorization).

我们已为此应用程序添加了基本授权。在每个请求到达 GraphQL 解析器之前，它具有全局授权；并在解析器层进行权限保护。它们会检查用户是否已通过身份验证，用户是否能够删除消息（基于权限的授权），以及一个用户是否能够删除另外一个用户（基于角色的授权）。

> If you want to be even more exact than resolver level authorization, check out **directive-based authorization** or **field level authorization** in GraphQL. You can apply authorization at the data-access level with an ORM like Sequelize, too. Your application's requirements decide which level is most effective for authorization.

如果你想要比解析器层授权更精确，请在 GraphQL 中查看 **基于指令的授权** 或 **字段级别的授权**。你也可以使用像 Sequelize 这样的 ORM，在数据访问级别进行权限验证。你可以基于应用程序的需求决定使用哪种级别的授权策略。

> ### Exercises:
### 练习：
> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/4f6e8e6e7b899faca13e1c8354fe59637e7e23a6)
* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/4f6e8e6e7b899faca13e1c8354fe59637e7e23a6)
> * Read more about [GraphQL authorization](https://graphql.github.io/learn/authorization/)
* 延伸阅读：[GraphQL 授权](https://graphql.github.io/learn/authorization/)的更多信息
> * Work through the different authorization scenarios with GraphQL Playground
* 通过 GraphQL Playground 测试不同的授权场景
> * Find out more about field level authorization with Apollo Server and GraphQL
* 了解 GraphQL Apollo 服务端中字段级别授权的更多信息
> * Find out more about data access level authorization with Apollo Server and GraphQL
* 了解 GraphQL Apollo 服务端中数据访问级别授权的更多信息

> ## GraphQL Custom Scalars in Apollo Server
## Apollo 服务端中的 GraphQL 自定义标量

> So far, you have used a couple of scalars in your GraphQL application, because each field resolves eventually to a scalar type. Let's add a String scalar for the date when a message got created. First, we'll extend the *src/schema/message.js* which uses this field for a message:

目前为止，你已经在你的 GraphQL 应用中使用过很多标量了，因为事实上每个字段最后都会被解析为一个标量类型。让我们来为消息的创建日期设置 String 标量类型。首先，我们要扩展文件 *src/schema/message.js*，为消息增加 createdAt 字段：

{title="src/schema/message.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  extend type Query {
    messages(cursor: String, limit: Int): [Message!]!
    message(id: ID!): Message!
  }

  extend type Mutation {
    createMessage(text: String!): Message!
    deleteMessage(id: ID!): Boolean!
  }

  type Message {
    id: ID!
    text: String!
# leanpub-start-insert
    createdAt: String!
# leanpub-end-insert
    user: User!
  }
`;
~~~~~~~~

> Second, adjust the seed data in the *src/index.js* file. At the moment, all seed data is created at once, which applies to the messages as well. It would be better to have each message created in one second intervals. The creation date should differ for each message.

然后，修改 *src/index.js* 文件中的种子数据。目前，所有的种子数据都是一次性生成的，消息也是。如果能够做到每条消息都是隔一秒再创建的就更好了，这样每条消息的创建时间就会不一样。

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

sequelize.sync({ force: eraseDatabaseOnSync }).then(async () => {
  if (eraseDatabaseOnSync) {
# leanpub-start-insert
    createUsersWithMessages(new Date());
# leanpub-end-insert
  }

  app.listen({ port: 8000 }, () => {
    console.log('Apollo Server on http://localhost:8000/graphql');
  });
});

# leanpub-start-insert
const createUsersWithMessages = async date => {
# leanpub-end-insert
  await models.User.create(
    {
      username: 'rwieruch',
      email: 'hello@robin.com',
      password: 'rwieruch',
      role: 'ADMIN',
      messages: [
        {
          text: 'Published the Road to learn React',
# leanpub-start-insert
          createdAt: date.setSeconds(date.getSeconds() + 1),
# leanpub-end-insert
        },
      ],
    },
    {
      include: [models.Message],
    },
  );

  await models.User.create(
    {
      username: 'ddavids',
      email: 'hello@david.com',
      password: 'ddavids',
      messages: [
        {
          text: 'Happy to release ...',
# leanpub-start-insert
          createdAt: date.setSeconds(date.getSeconds() + 1),
# leanpub-end-insert
        },
        {
          text: 'Published a complete ...',
# leanpub-start-insert
          createdAt: date.setSeconds(date.getSeconds() + 1),
# leanpub-end-insert
        },
      ],
    },
    {
      include: [models.Message],
    },
  );
};
~~~~~~~~

> Now you should be able to query the `createdAt` of a message in your GraphQL Playground:

现在，你应该可以在 GraphQL Playground 中查询消息的 `createdAt` 字段：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
query {
  message(id: "1") {
    id
    createdAt
    user {
      username
    }
  }
}

// query result
{
  "data": {
    "message": {
      "id": "1",
      "createdAt": "1540978531448",
      "user": {
        "username": "rwieruch"
      }
    }
  }
}
~~~~~~~~

> You may have noticed something odd: While the date returned from a GraphQL Playground has a unix timestamp (e.g. 1540978531448), the date the database for a message (and other entities) has another format (e.g. 2018-10-31 17:35:31.448+08). Check it yourself with psql. That's the internal working of GraphQL which uses its internal formatting rules for dates. You can change this behavior by adding a custom scalar. First, install a popular GraphQL node package for custom date scalars.

你可能已经注意到了这里的奇怪之处：从 GraphQL Playground 返回的数据中，有一个 unix 时间戳（例如：1540978531448）；而在数据库中，消息（以及其他实体）的时间有另外一个格式（例如：2018-10-31 17:35:31.448+08）。你可以自己用 psql 查看。这是 GraphQL 的内部机制，使用它自己内部的格式化日期的规则。你可以通过增加自定义标量来改变这种行为。首先，我们需要安装一个主流的 GraphQL 自定义时间标量的 node 依赖包。

{title="Command Line",lang="json"}
~~~~~~~~
npm install graphql-iso-date --save
~~~~~~~~

> Second, introduce a `Date` scalar in your schema in the *src/schema/index.js* file:

然后，在 *src/schema/index.js* 文件中的 schema 中引入一个 `Date` 标量：

{title="src/schema/index.js",lang="javascript"}
~~~~~~~~
const linkSchema = gql`
# leanpub-start-insert
  scalar Date
# leanpub-end-insert

  type Query {
    _: Boolean
  }

  type Mutation {
    _: Boolean
  }

  type Subscription {
    _: Boolean
  }
`;
~~~~~~~~

> Third, define the scalar with the help of the installed node package in your *src/resolvers/index.js* file:

接着，在 *src/resolvers/index.js* 文件中，使用这个安装好的依赖包来定义这个标量。

{title="src/resolvers/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { GraphQLDateTime } from 'graphql-iso-date';
# leanpub-end-insert

import userResolvers from './user';
import messageResolvers from './message';

# leanpub-start-insert
const customScalarResolver = {
  Date: GraphQLDateTime,
};
# leanpub-end-insert

export default [
# leanpub-start-insert
  customScalarResolver,
# leanpub-end-insert
  userResolvers,
  messageResolvers,
];
~~~~~~~~

> And last but not least, change the scalar type from String to Date for your message schema in the *src/schema/message.js*:

最后，在 *src/schema/message.js* 文件中，把消息 schema 中的 createdAt 字段的标量类型从 String 变为 Date

{title="src/schema/message.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  extend type Query {
    messages(cursor: String, limit: Int): [Message!]!
    message(id: ID!): Message!
  }

  extend type Mutation {
    createMessage(text: String!): Message!
    deleteMessage(id: ID!): Boolean!
  }

  type Message {
    id: ID!
    text: String!
# leanpub-start-insert
    createdAt: Date!
# leanpub-end-insert
    user: User!
  }
`;
~~~~~~~~

> Now, query again your messages. The output for the `createdAt` date should be different.

现在，重新查询一遍消息列表，`createdAt` 日期的格式已经改变了。

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "data": {
    "message": {
      "id": "1",
# leanpub-start-insert
      "createdAt": "2018-10-31T11:57:53.043Z",
# leanpub-end-insert
      "user": {
        "username": "rwieruch"
      }
    }
  }
}
~~~~~~~~

> It's in a readable format now. You can dive deeper into the date formatting that can be adjusted with this library by checking out their [documentation](https://github.com/excitement-engineer/graphql-iso-date).

现在它已经是一个可读的格式了。你可以通过查询[文档](https://github.com/excitement-engineer/graphql-iso-date)研究这个库能把日期转换成哪些格式。

> ### Exercises:
> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-express-postgresql-boilerplate/tree/709a406a8a94e15779d2e93cfb847d49de5aa6ca)
> * Read more about [custom scalars in GraphQL](https://www.apollographql.com/docs/apollo-server/features/scalars-enums.html)

### 练习：
* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-express-postgresql-boilerplate/tree/709a406a8a94e15779d2e93cfb847d49de5aa6ca)
* 延伸阅读：[GraphQL 中的自定义标量](https://www.apollographql.com/docs/apollo-server/features/scalars-enums.html)


> ## Pagination in GraphQL with Apollo Server

## Apollo 服务端中 GrapgQL 的分页

> Using GraphQL, you will almost certainly encounter a feature called **pagination** for applications with lists of items. Stored user messages in a chat application become long lists, and when the client application request messages for the display, retrieving all messages from the database at once can lead to severe performance bottlenecks. Pagination allows you to split up a list of items into multiple lists, called pages. A page is usually defined with a limit and an offset. That way, you can request one page of items, and when a user wants to see more, request another page of items.

使用 GraphQL 必然会遇到列表应用的 **分页** 功能。聊天应用中保存的用户消息会变得越来越长，当客户端请求消息记录用于展示时，一次性从数据库中获取所有的消息会导致服务器性能瓶颈。分页允许你把一个列表分成多个列表，即分页。一页通常会有大小限制和偏移量。通过分页，你可以只请求一页的展示数据，如果用户想查看更多的数据，就再请求一页。

> You will implement pagination in GraphQL with two different approaches in the following sections. The first approach will be the most naive approach, called **offset/limit-based pagination**. The advanced approach is **cursor-based pagination**, one of many sophisticated ways to allow pagination in an application.

接下来，你将会通过两种方式在 GraphQL 中实现分页。第一种方法是最原始的方法，叫做**偏移/限制分页**。更复杂的方式是**游标分页**，这是在应用中允许分页的众多经典实践的中一种。

> ### Offset/Limit Pagination with Apollo Server and GraphQL

### Apollo Server 中 GrapgQL 的偏移/限制分页

> Offset/limit-based pagination isn't too difficult to implement. The limit states how many items you want to retrieve from the entire list, and the offset states where to begin in the whole list. Using different offsets, you can shift through the entire list of items and retrieve a sublist (page) of it with the limit.

偏移/限制分页并不是很难实现。限制规定了你每次想要从整个列表中获取多少条数据，偏移量指定了应该从整个列表的什么位置开始。使用不同的偏移量，你可以筛选整个列表并从中获取一个固定条数的子列表（页）。

> We set the message schema in the *src/schema/message.js* file to consider the two new arguments:

我们在文件 *src/schema/message.js* 中修改消息的 schema，增加两个新的参数：

{title="src/schema/message.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  extend type Query {
# leanpub-start-insert
    messages(offset: Int, limit: Int): [Message!]!
# leanpub-end-insert
    message(id: ID!): Message!
  }

  extend type Mutation {
    createMessage(text: String!): Message!
    deleteMessage(id: ID!): Boolean!
  }

  type Message {
    id: ID!
    text: String!
    createdAt: Date!
    user: User!
  }
`;
~~~~~~~~

> Then you can adjust the resolver in the *src/resolvers/message.js* file to handle the new arguments:

然后，你可以修改 *src/resolvers/message.js* 文件中的解析器，以便处理这两个新的参数：

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
...

export default {
  Query: {
# leanpub-start-insert
    messages: async (
      parent,
      { offset = 0, limit = 100 },
      { models },
    ) => {
      return await models.Message.findAll({
        offset,
        limit,
      });
# leanpub-end-insert
    },
    message: async (parent, { id }, { models }) => {
      return await models.Message.findById(id);
    },
  },

  Mutation: {
    ...
  },

  ...
};
~~~~~~~~

> Fortunately, your ORM (Sequelize) gives you everything you need for internal offset and limit functionality. Try it in GraphQL Playground yourself by adjusting the limit and offset.

幸运的是，你的 ORM (Sequelize) 为内部偏移和限制功能提供了支持。你可以在 GraphQL playground 上尝试修改限制和偏移量，体验分页功能。

{title="GraphQL Playground",lang="json"}
~~~~~~~~
query {
  messages(offset: 1, limit: 2){
    text
  }
}
~~~~~~~~

> Even though this approach is simpler, it comes with a few disadvantages. When your offset becomes very long, the database query takes longer, which  can lead to a poor client-side performance while the UI waits for the next page of data. Also, offset/limit pagination cannot handle deleted items in between queries. For instance, if you query the first page and someone deletes an item, the offset would be wrong on the next page because the item count is off by one. You cannot easily overcome this problem with offset/limit pagination, which is why cursor-based pagination might be necessary.

虽然这个方法很简单，但是它也有缺点。当偏移量变得非常长时，数据库查询时间会变长，客户端在等待下一页数据时的性能会比较差。同时，偏移/限制分页不能处理在查询之前删除一个数据的情况。例如，你查询了第一页，同时有人删除了第一页中的一条数据，下一页的偏移量就会出错，因为数据已经减少一个了。在偏移/限制分页中，这个问题并不容易解决，所以游标分页就有必要了。

> ### Cursor-based Pagination with Apollo Server and GraphQL

### Apollo 服务端中 GraphQL 的游标分页

> In cursor-based pagination, the offset is given an identifier called a **cursor** rather counting items like offset/limit pagination. The cursor can be used to express "give me a limit of X items from cursor Y". A common approach to use dates (e.g. creation date of an entity in the database) to identify an item in the list. In our case, each message already has a `createdAt` date that is assigned to the entity when it is written to the database and we expose it already in the schema of the message entity. That's the creation date of each message that will be the cursor.

与偏移/限制分页中用数据量的个数标记偏移量不同，在游标分页中，我们用**游标**标记偏移量。游标可用于表达：”给我从游标 Y 开始的 X 个数据”。一个常用的方法是用时间（例如：一个实体在数据库中的创建时间）来标识一个列表中的某条数据。在我们的例子中，每条消息已经有一个 `createdAt` 时间，即这个实体被写入数据库的时间，而且我们已经在消息实体的 schema 中暴露了这个字段。这个消息的创建时间就是游标。

> Now we have to change the original pagination to cursor-based in the *src/schema/message.js* file. You only need to exchange the offset with the cursor. Instead of an offset that can only be matched implicitly to an item in a list and changes once an item is deleted from the list, the cursor has a stable position within, because the message creation dates won't change.

现在，我们在 *src/schema/message.js* 中将之前的分页改为游标分页。你只需要修改将偏移量变为游标（将 `offset` 变为 `cursor`）。与偏移量只能明确指向列表中的一条数据，而且一旦从列表中删除一条数据，偏移量就会发生变化这种现象不同，游标指向的位置是固定不变的，因为消息创建的时间不会改变。

{title="src/schema/message.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  extend type Query {
# leanpub-start-insert
    messages(cursor: String, limit: Int): [Message!]!
# leanpub-end-insert
    message(id: ID!): Message!
  }

  extend type Mutation {
    createMessage(text: String!): Message!
    deleteMessage(id: ID!): Boolean!
  }

  type Message {
    id: ID!
    text: String!
    createdAt: Date!
    user: User!
  }
`;
~~~~~~~~

> Since you adjusted the schema for the messages, reflect these changes in your *src/resolvers/message.js* file as well:

因为你修改了消息的 schema，你也需要相应地修改 *src/resolvers/message.js* 文件：

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import Sequelize from 'sequelize';
# leanpub-end-insert

...

export default {
  Query: {
# leanpub-start-insert
    messages: async (parent, { cursor, limit = 100 }, { models }) => {
# leanpub-end-insert
      return await models.Message.findAll({
        limit,
# leanpub-start-insert
        where: {
          createdAt: {
            [Sequelize.Op.lt]: cursor,
          },
        },
# leanpub-end-insert
      });
    },
    message: async (parent, { id }, { models }) => {
      return await models.Message.findById(id);
    },
  },

  Mutation: {
    ...
  },

  ...
};
~~~~~~~~

> Instead of the offset, the cursor is the `createdAt` property of a message. With Sequelize and other ORMs it is possible to add a clause to find all items in a list by a starting property (`createdAt`) with less than (`lt`) or greater than (`gt`, which is not used here) values for this property. Using a date as a cursor, the where clause finds all messages **before** this date, because there is an `lt` Sequelize operator. There are two more things to make it work:

与偏移量不同，游标是消息的 `createdAt` 属性。通过 Sequelize 或其他 ORM 框架，我们可以添加一个查询子句来查找从开始值（`createdAt`）开始列表中所有小于（`lt`）或者大于（`gt`，并没有再此处使用）这个值的所有数据。将日期当做游标，配合 `lt` Sequelize 操作符，where 子句将查找所有在这个日前**之前**的消息。为了让它可以工作，这里有两件额外的事情需要做。

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
...

export default {
  Query: {
    messages: async (parent, { cursor, limit = 100 }, { models }) => {
      return await models.Message.findAll({
# leanpub-start-insert
        order: [['createdAt', 'DESC']],
# leanpub-end-insert
        limit,
# leanpub-start-insert
        where: cursor
          ? {
# leanpub-end-insert
              createdAt: {
                [Sequelize.Op.lt]: cursor,
              },
            }
# leanpub-start-insert
          : null,
# leanpub-end-insert
      });
    },
    message: async (parent, { id }, { models }) => {
      return await models.Message.findById(id);
    },
  },

  Mutation: {
    ...
  },

  ...
};
~~~~~~~~

> First, the list should be ordered by `createdAt` date, otherwise the cursor won't help. However, you can be sure that requesting the first page of messages without a cursor will lead to the most recent messages when the list is ordered. When you request the next page with a cursor based on the previous page's final creation date, you get the next page of messages ordered by creation date. That's how you can move page by page through the list of messages.

首先，这个列表必须是通过 `createdAt` 排序的，否则游标是无用的。无论如何，当列表是有序时，都可以确保不带游标请求消息列表的第一页也可以返回最近的消息。当你将前一页的最后一个创建时间当做游标用于请求下一页时，你可以获得通过创建时间排序的下一页消息。这就是你怎么一页一页访问整个消息列表的。

> Second, the ternary operator for the cursor makes sure the cursor isn't needed for the first page request. As mentioned, the first page only retrieves the most recent messages in the list, so you can use the creation date of the last message as a cursor for the next page of messages.

其次，游标的三元操作符确保了第一页请求不需要游标。之前提到过，第一页只获取列表中最近的消息，所以你可以利用最后一条消息的创建时间当做游标来请求下一页消息。

> You can also extract the where clause from the database query:

你也可以将数据库查询中的 where 语句抽出来：

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
...

export default {
  Query: {
    messages: async (parent, { cursor, limit = 100 }, { models }) => {
# leanpub-start-insert
      const cursorOptions = cursor
        ? {
            where: {
              createdAt: {
                [Sequelize.Op.lt]: cursor,
              },
            },
          }
        : {};
# leanpub-end-insert

      return await models.Message.findAll({
        order: [['createdAt', 'DESC']],
        limit,
# leanpub-start-insert
        ...cursorOptions,
# leanpub-end-insert
      });
    },
    message: async (parent, { id }, { models }) => {
      return await models.Message.findById(id);
    },
  },

  Mutation: {
    ...
  },

  ...
};
~~~~~~~~

> Now you can test what you've learned in GraphQL Playground to see it in action. Make the first request for the most recent messages:

现在你可以在 GraphQL playground 中试试你刚刚学到的知识，看看它的真实效果。发送第一个请求去获取最近的消息列表：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
query {
  messages(limit: 2) {
    text
    createdAt
  }
}
~~~~~~~~

> Which may lead to something like this (be careful, dates should be different from your dates):

这可能会得到下面这样的数据（注意，你的时间应该和此处不一样）：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "data": {
    "messages": [
      {
        "text": "Published a complete ...",
        "createdAt": "2018-10-25T08:22:02.484Z"
      },
      {
        "text": "Happy to release ...",
        "createdAt": "2018-10-25T08:22:01.484Z"
      }
    ]
  }
}
~~~~~~~~

> Now you can use the `createdAt` date from the last page to request the next page of messages with a cursor:

现在，你可以将最近一页的 `createdAt` 时间当做游标去请求下一页消息：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
query {
  messages(limit: 2, cursor: "2018-10-25T08:22:01.484Z") {
    text
    createdAt
  }
}
~~~~~~~~

> The result gives the last message from the seed data, but the limit is set to 2 messages. This happens because there are only 3 messages in the database and you already have retrieved 2 in the last pagination action:

返回结果为种子数据中的最后一条消息，然而此时限制是 2 条消息。出现这种情况是因为在数据库中只有 3 条数据，而你已经在上一次分页请求中获取了 2 条数据：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "data": {
    "messages": [
      {
        "text": "Published the Road to learn React",
        "createdAt": "2018-10-25T08:22:00.484Z"
      }
    ]
  }
}
~~~~~~~~

> That's a basic implementation of a cursor-based pagination using the creation date of an item as a stable identifier. The creation date is a common approach, but there are alternatives you should explore as well.

这就是使用创建时间作为一条数据的稳定标识符的游标分页的一个基础的应用。创建时间是一个通用的方法，然而这里还有另外的值得探索的方法。

> ### Cursor-based Pagination: Page Info, Connections and Hashes

### 基于游标的分页:页面信息、连接和哈希

> In this last section about pagination in GraphQL, we advance the cursor-based pagination with a few improvements. Currently, you have to query all creation dates of the messages to use the creation date of the last message for the next page as a cursor. GraphQL connections add only a structural change to your list fields in GraphQL that allow you to pass meta information. Let's add a GraphQL connection in the *src/schema/message.js* file:

在这 GraphQL 分页的最后一节中，我们对基于游标的分页进行一些改进。目前，你必须查询所有消息的创建时间，以便将最后一条消息的创建时间用作查询下一页的游标。GraphQL 连接只修改列表字段结构，就能支持传递元信息。让我们在 *src/schema/message.js* 文件中添加一个 GraphQL 链接:

{title="src/schema/message.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  extend type Query {
# leanpub-start-insert
    messages(cursor: String, limit: Int): MessageConnection!
# leanpub-end-insert
    message(id: ID!): Message!
  }

  extend type Mutation {
    createMessage(text: String!): Message!
    deleteMessage(id: ID!): Boolean!
  }

# leanpub-start-insert
  type MessageConnection {
    edges: [Message!]!
    pageInfo: PageInfo!
  }
# leanpub-end-insert

# leanpub-start-insert
  type PageInfo {
    endCursor: Date!
  }
# leanpub-end-insert

  type Message {
    id: ID!
    text: String!
    createdAt: Date!
    user: User!
  }
`;
~~~~~~~~

> You introduced an intermediate layer that holds meta information with the PageInfo type with the list of items in an edges field. In the intermediate layer, you can introduce the new information such as an  `endCursor`( `createdAt` of the last message in the list). Then, you won't need to query every `createdAt` date of every message, only the `endCursor`. Place these changes in the *src/resolvers/message.js* file:

你引入了一个拥有元信息的中间层，其中包含 PageInfo 类型，以及代表列项的 edges 字段。在中间层，你可以引入新的信息，比如 `endCursor` (列表中最后一条消息的 `createdAt`)。然后，你将不需要查询每条消息的 `createdAt` 时间，只需要查询 `endCursor`。在 *src/resolvers/message.js* 文件中进行以下调整:

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
...

export default {
  Query: {
    messages: async (parent, { cursor, limit = 100 }, { models }) => {
      const cursorOptions = cursor
        ? {
            where: {
              createdAt: {
                [Sequelize.Op.lt]: cursor,
              },
            },
          }
        : {};

# leanpub-start-insert
      const messages = await models.Message.findAll({
# leanpub-end-insert
        order: [['createdAt', 'DESC']],
        limit,
        ...cursorOptions,
      });

# leanpub-start-insert
      return {
        edges: messages,
        pageInfo: {
          endCursor: messages[messages.length - 1].createdAt,
        },
      };
# leanpub-end-insert
    },
    message: async (parent, { id }, { models }) => {
      return await models.Message.findById(id);
    },
  },

  Mutation: {
    ...
  },

  ...
};
~~~~~~~~

> You gave the result a new structure with the intermediate `edges` and `pageInfo` fields. The `pageInfo` field now has the cursor of the last message in the list, and you should be able to query the first page the following way:

你返回了一个带有 `edges` 和 `pageInfo` 中间字段的新结构。`pageInfo` 字段现在拥有列表中最后一条消息的游标，你应该能够通过以下方式查询第一页:

{title="GraphQL Playground",lang="json"}
~~~~~~~~
query {
  messages(limit: 2) {
    edges {
      text
    }
    pageInfo {
      endCursor
    }
  }
}
~~~~~~~~

> The result may look like the following:

结果如下：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "data": {
    "messages": {
      "edges": [
        {
          "text": "Published a complete ..."
        },
        {
          "text": "Happy to release ..."
        }
      ],
      "pageInfo": {
        "endCursor": "2018-10-25T08:29:56.771Z"
      }
    }
  }
}
~~~~~~~~

> Use the last cursor to query the next page:

通过最后一个游标查询下一页信息

{title="GraphQL Playground",lang="json"}
~~~~~~~~
query {
  messages(limit: 2, cursor: "2018-10-25T08:29:56.771Z") {
    edges {
      text
    }
    pageInfo {
      endCursor
    }
  }
}
~~~~~~~~

> Again, this will only return the remaining last message in the list. You are no longer required to query the creation date of every message, only to query the cursor for the last message. The client application doesn't need the details for the cursor of the last message, as it just needs `endCursor` now.:

同样，这将返回列表中剩余的最后一条消息。你不再需要查询每条消息的创建时间，只需要查询最后一条消息的游标。客户端应用程序不需要最后一条消息的游标的详细信息，因为它现在只需要 `endCursor`。

> You can add relevant information in the intermediate GraphQL connection layer. Sometimes, a GraphQL client needs to know whether there are more pages of a list to query, because every list is finite. Let's add this information to the schema for the message's connection in the *src/schema/message.js* file

你可以在 GraphQL 连接层中添加相关信息。由于每个列表数目都是有限的，有时，GraphQL 客户端需要知道列表中是否还有更多可以查询的页面。让我们在 *src/schema/message.js* 文件中将这个信息添加到消息连接的模式中:

{title="src/schema/message.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  extend type Query {
    messages(cursor: String, limit: Int): MessageConnection!
    message(id: ID!): Message!
  }

  extend type Mutation {
    createMessage(text: String!): Message!
    deleteMessage(id: ID!): Boolean!
  }

  type MessageConnection {
    edges: [Message!]!
    pageInfo: PageInfo!
  }

  type PageInfo {
# leanpub-start-insert
    hasNextPage: Boolean!
# leanpub-end-insert
    endCursor: Date!
  }

  ...
`;
~~~~~~~~

> In the resolver in the *src/resolvers/message.js* file, you can find this information with the following:

在 *src/resolvers/message.js* 文件的解析器中，你可以看到以下信息

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
...

export default {
  Query: {
    messages: async (parent, { cursor, limit = 100 }, { models }) => {
      ...

      const messages = await models.Message.findAll({
        order: [['createdAt', 'DESC']],
# leanpub-start-insert
        limit: limit + 1,
# leanpub-end-insert
        ...cursorOptions,
      });

# leanpub-start-insert
      const hasNextPage = messages.length > limit;
      const edges = hasNextPage ? messages.slice(0, -1) : messages;
# leanpub-end-insert

      return {
# leanpub-start-insert
        edges,
# leanpub-end-insert
        pageInfo: {
# leanpub-start-insert
          hasNextPage,
          endCursor: edges[edges.length - 1].createdAt,
# leanpub-end-insert
        },
      };
    },
    message: async (parent, { id }, { models }) => {
      return await models.Message.findById(id);
    },
  },

  Mutation: {
    ...
  },

  ...
};
~~~~~~~~

> You only retrieve one more message than defined in the limit. If the list of messages is longer than the limit, there is a next page; otherwise, there is no next page. You return the limited messages, or all messages if there is no next page. Now you can include the `hasNextPage` field in the `pageInfo` field. If you query messages with a limit of 2 and no cursor, you get true for the `hasNextPage` field. If query messages with a limit of more than 2 and no cursor, the `hasNextPage` field becomes false. Then, your GraphQL client application knows that the list has reached its end.

你只比定义的 limit 多检索一条消息。如果消息列表数量大于 limit，则有下一页;否则，就没有下一页。要么返回限定的 limit 数量的消息，要么没有下一页时，返回所有消息。现在，你可以在 `pageInfo` 类型中添加 `hasNextPage` 字段。如果查询消息的条件为 limit = 2，且没有游标，返回结果中，`hasNextPage` 为 true。如果查询消息的条件为 limit > 2 且没有游标，返回结果中，`hasNextPage` 为 false。据此，GraphQL 客户端应用程序知道列表已经查询到最后了。

> The last improvements gave your GraphQL client application a more straightforward GraphQL API. The client doesn't need to know about the cursor being the last creation date of a message in a list. It only uses the `endCursor` as a `cursor` argument for the next page. However, the cursor is still a creation date property, which may lead to confusion on the GraphQL client side. The client shouldn't care about the format or the actual value of the cursor, so we'll ask the cursor with a hash function that uses a base64 encoding:

最后的改进是给 GraphQL 客户端应用程序提供一个更加直观的 GraphQL API。客户端不需要知道游标代表列表中最后一条消息的创建时间。它仅使用 `endCursor` 作为下一页的 `cursor` 参数。但是，游标仍然是一个创建时间属性，这可能会引起 GraphQL 客户端的困惑。客户端不应该关心游标的格式或真实值，所以我们将用一个使用 base64 编码的哈希函数来处理游标:

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const toCursorHash = string => Buffer.from(string).toString('base64');
# leanpub-end-insert

# leanpub-start-insert
const fromCursorHash = string =>
  Buffer.from(string, 'base64').toString('ascii');
# leanpub-end-insert

export default {
  Query: {
    messages: async (parent, { cursor, limit = 100 }, { models }) => {
      const cursorOptions = cursor
        ? {
            where: {
              createdAt: {
# leanpub-start-insert
                [Sequelize.Op.lt]: fromCursorHash(cursor),
# leanpub-end-insert
              },
            },
          }
        : {};

      ...

      return {
        edges,
        pageInfo: {
          hasNextPage,
# leanpub-start-insert
          endCursor: toCursorHash(
            edges[edges.length - 1].createdAt.toString(),
          ),
# leanpub-end-insert
        },
      };
    },
    message: async (parent, { id }, { models }) => {
      return await models.Message.findById(id);
    },
  },

  Mutation: {
    ...
  },

  ...
};
~~~~~~~~

> The returned cursor as meta information is hashed by the new utility function. Remember to stringify the date before hashing it. In addition, the `endCursor` in the *src/schema/message.js* file isn't a Date anymore, but a String scalar again.

作为元信息返回的游标被这个新的通用函数哈希化。记住，在哈希化之前要把时间转化为字符串格式。此外，在 *src/schema/message.js* 文件中 `endCursor` 不再是一个时间，而是一个字符串标量。

{title="src/schema/message.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  ...

  type MessageConnection {
    edges: [Message!]!
    pageInfo: PageInfo!
  }

  type PageInfo {
    hasNextPage: Boolean!
# leanpub-start-insert
    endCursor: String!
# leanpub-end-insert
  }

  ...
`;
~~~~~~~~

> The GraphQL client receives a hashed `endCursor` field. The hashed value can be used as a cursor to query the next page. In the resolver, the incoming cursor is reverse hashed to the actual date, which is used for the database query.

GraphQL 客户端接收哈希化的 `endCursor` 字段。这个哈希值可以用作查询下一页的游标。在解析器中，用于数据库查询时，需要把传入的游标反哈希化成真实时间。

> Hashing the cursor is a common approach for cursor-based pagination because it hides the details from the client. The (GraphQL) client application only needs to use the hash value as a cursor to query the next paginated page.

哈希化游标是基于游标的分页的一种常见方法，因为它对客户端隐藏了详细信息。（GraphQL）客户端应用程序只需要使用哈希值作为查询下一页的游标。

> ### Exercises:

### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/810907cde43b460231b9ed3a2172e62528f81ba4)
> * Read more about [GraphQL pagination](https://graphql.github.io/learn/pagination/)

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/810907cde43b460231b9ed3a2172e62528f81ba4)
* 延伸阅读：[GraphQL 分页](https://graphql.github.io/learn/pagination/)

> ## GraphQL Subscriptions

## GraphQL 订阅

> So far, you used GraphQL to read and write data with queries and mutations. These are the two essential GraphQL operations to get a GraphQL server ready for CRUD operations. Next, you will learn about GraphQL Subscriptions for real-time communication between GraphQL client and server.

到目前为止，你使用 GraphQL 的查询和变更来读写数据。这是使 GraphQL 服务端为增删改查操作做好准备的两个基本 GraphQL 操作。接下来，你将了解 GraphQL 订阅，以便在 GraphQL 客户端和服务端之间进行实时通信。

> Next, you will implement real-time communication for created messages. If a user creates a message, another user should get this message in a GraphQL client application as a real-time update. To start, we add the Subscription root level type to the *src/schema/message.js* schema:

接下来，你将为创建的消息实现实时通信。如果一个用户创建了一条消息，那么在 GraphQL 客户端应用程序中，另一个用户应当实时获得这条新消息。首先，我们在 *src/schema/message.js* 文件中，添加根级别的订阅类型:

{title="src/schema/message.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  extend type Query {
    ...
  }

  extend type Mutation {
    ...
  }

  ...

  type Message {
    id: ID!
    text: String!
    createdAt: Date!
    user: User!
  }

# leanpub-start-insert
  extend type Subscription {
    messageCreated: MessageCreated!
  }
# leanpub-end-insert

# leanpub-start-insert
  type MessageCreated {
    message: Message!
  }
# leanpub-end-insert
`;
~~~~~~~~

> As a naive GraphQL consumer, a subscription works like a GraphQL query. The difference is that the subscription emits changes (events) over time. Every time a message is created, the subscribed GraphQL client receives the created message as payload. A subscription from a GraphQL client for the schema would look like this:

作为一个简单的 GraphQL 用户，订阅的工作方式类似于 GraphQL 查询。不同之处在于订阅会实时触发变更(事件)。每次创建消息时，订阅的 GraphQL 客户端都会将创建的消息作为有效负载接收。来自 GraphQL 客户端的订阅模式将如下所示:

{title="GraphQL Playground",lang="json"}
~~~~~~~~
subscription {
  messageCreated {
    message {
      id
      text
      createdAt
      user {
        id
        username
      }
    }
  }
}
~~~~~~~~

> In the first part, you'll set up the subscription architecture for your application; then, you'll add the implementation details for the created message subscription. The first step need only be completed once, but the latter will be a recurring when more GraphQL subscriptions are added to your application.

在第一部分中，你将为应用程序设置订阅结构；然后，你将为创建的消息订阅添加实现细节。第一步只需要做一次，但是当向应用程序添加更多的 GraphQL 订阅时，这一步将需要再次设置。

> ### Apollo Server Subscription Setup

### Apollo 服务端订阅配置

> Because we are using Express as middleware, expose the subscriptions with an advanced HTTP server setup in the *src/index.js* file:

由于我们使用 Express 作为中间件，所以在 *src/index.js* 文件中可以使用高级 HTTP 服务设置公开订阅:

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import http from 'http';
# leanpub-end-insert

...

server.applyMiddleware({ app, path: '/graphql' });

# leanpub-start-insert
const httpServer = http.createServer(app);
server.installSubscriptionHandlers(httpServer);
# leanpub-end-insert

const eraseDatabaseOnSync = true;

sequelize.sync({ force: eraseDatabaseOnSync }).then(async () => {
  if (eraseDatabaseOnSync) {
    createUsersWithMessages(new Date());
  }

# leanpub-start-insert
  httpServer.listen({ port: 8000 }, () => {
# leanpub-end-insert
    console.log('Apollo Server on http://localhost:8000/graphql');
  });
});

...
~~~~~~~~

> For the context passed to the resolvers, you can distinguish between HTTP requests (GraphQL mutations and queries) and subscriptions in the same file. HTTP requests come with a req and res object, but the subscription comes with a connection object, so you can pass the models as a data access layer for the subscription's context.

根据传递给解析器的上下文，你可以在同一个文件中区分 HTTP 请求(GraphQL 变更和查询）和订阅。HTTP 请求附带一个 req 和 res 对象，但是订阅附带一个连接对象，因此可以将模型作为订阅上下文的数据访问层。

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

const server = new ApolloServer({
  typeDefs: schema,
  resolvers,
  ...
# leanpub-start-insert
  context: async ({ req, connection }) => {
    if (connection) {
      return {
        models,
      };
    }
# leanpub-end-insert

# leanpub-start-insert
    if (req) {
# leanpub-end-insert
      const me = await getMe(req);

      return {
        models,
        me,
        secret: process.env.SECRET,
      };
# leanpub-start-insert
    }
# leanpub-end-insert
  },
});

...
~~~~~~~~

> To complete the subscription setup, you'll need to use one of the available [PubSub engines](https://www.apollographql.com/docs/apollo-server/v2/features/subscriptions.html#PubSub-Implementations) for publishing and subscribing to events. Apollo Server comes with its own by default, but there are links for other options should you find it lacking. In a new *src/subscription/index.js* file, add the following:

要完成订阅设置，你需要使用一个可用的 [PubSub 引擎](https://www.apollographql.com/docs/apollo-server/v2/features/subscriptions.html#PubSub-Implementations)来发布和订阅事件。默认情况下，Apollo Server 自带 PubSub 引擎，但是你可能会发现它缺少一些其他的配置。新建一个 *src/subscription/index.js* 文件，配置以下内容:

{title="src/subscription/index.js",lang="javascript"}
~~~~~~~~
import { PubSub } from 'apollo-server';

export default new PubSub();
~~~~~~~~

> This PubSub instance is your API which enables subscriptions in your application. The overarching setup for subscriptions is done now.

这个 PubSub 实例是你的 API，它支持应用程序中的订阅。现在我们完成了订阅的总体设置。

> ### Subscribing and Publishing with PubSub

### 使用 PubSub 实现订阅及发布

> Let's implement the specific subscription for the message creation. It should be possible for another GraphQL client to listen to message creations. For instance, in a chat application it should be possible to see a message of someone else in real-time. Therefore, extend the previous *src/subscription/index.js* file with the following implementation:

让我们实现一个针对即时消息的订阅服务，它可以使 GraphQL 客户端监听到新消息。例如，在聊天应用中实时的收到他人发来的信息。接下来，继续完成之前的 *src/subscription/index.js* ：

{title="src/subscription/index.js",lang="javascript"}
~~~~~~~~
import { PubSub } from 'apollo-server';

# leanpub-start-insert
import * as MESSAGE_EVENTS from './message';
# leanpub-end-insert

# leanpub-start-insert
export const EVENTS = {
  MESSAGE: MESSAGE_EVENTS,
};
# leanpub-end-insert

export default new PubSub();
~~~~~~~~

> And add your first event in a new *src/subscription/message.js* file, which we used earlier:

在 *src/subscription/message.js* 中添加你的第一个事件常量：

{title="src/subscription/message.js",lang="javascript"}
~~~~~~~~
export const CREATED = 'CREATED';
~~~~~~~~

> This folder structure allows you to separate your events at the domain level. By exporting all events with their domains, you can import all events elsewhere and make use of the domain-specific events.

通过按领域划分并导出相关事件常量，便于你在其他地方导入和使用。这样的目录结构让你能在领域层分离不同种类的事件。

> The only piece missing is using the event and the PubSub instance in your message resolver. In the beginning of this section, you added the new subscription to the message schema. Now you have to implement its counterpart in the *src/resolvers/message.js* file:

在本节的开始，你已经添加了新的订阅操作到 message 模式中。唯一缺失的就是在 message 解析器中处理订阅事件和 PubSub 实例。现在你需要在 *src/resolvers/message.js*  实现对应的部分：

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
import pubsub, { EVENTS } from '../subscription';
# leanpub-end-insert

...

export default {
  Query: {
    ...
  },

  Mutation: {
    ...
  },

  Message: {
    ...
  },

# leanpub-start-insert
  Subscription: {
    messageCreated: {
      subscribe: () => pubsub.asyncIterator(EVENTS.MESSAGE.CREATED),
    },
  },
# leanpub-end-insert
};
~~~~~~~~

> The subscribe's function signature has access to the same arguments as the other resolver functions. Models from the context can be accessed here, but it isn't necessary for this application.

subscribe 函数签名和其他解析函数相同。上下文提供的模型对象可以在这里访问，但是在本例中用不到。

> The subscription as resolver provides a counterpart for the subscription in the  . However, since it uses a publisher-subscriber mechanism (PubSub) for events, you have only implemented the subscribing, not the publishing. It is possible for a GraphQL client to listen for changes, but there are no changes published yet. The best place for publishing a newly created message is in the same file as the created message:

Subscription 里的解析器处理了 message 模式中所对应的订阅。上面的代码完成了消息的订阅。然而，在发布订阅模型中，你还需要实现发布。在订阅解析器的同一个文件中放置发布函数是最好的。

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
...

import pubsub, { EVENTS } from '../subscription';

...

export default {
  Query: {
    ...
  },

  Mutation: {
    createMessage: combineResolvers(
      isAuthenticated,
      async (parent, { text }, { models, me }) => {
# leanpub-start-insert
        const message = await models.Message.create({
# leanpub-end-insert
          text,
          userId: me.id,
        });

# leanpub-start-insert
        pubsub.publish(EVENTS.MESSAGE.CREATED, {
          messageCreated: { message },
        });
# leanpub-end-insert

# leanpub-start-insert
        return message;
# leanpub-end-insert
      },
    ),

    ...
  },

  Message: {
    ...
  },

  Subscription: {
    messageCreated: {
      subscribe: () => pubsub.asyncIterator(EVENTS.MESSAGE.CREATED),
    },
  },
};
~~~~~~~~

> You implemented your first subscription in GraphQL with Apollo Server and PubSub. To test it, create a new message with a logged in user. You can try both these GraphQL operations in two separate tabs in GraphQL Playground to compare their output. In the first tab, execute the subscription:

使用 Apollo 服务端和 PubSub ，你在 GraphQL 中实现了第一个订阅服务。为了测试是否奏效，可以使用一个已登录的用户发送一条信息。你可以在两个不同的浏览器标签中打开 GraphQL Playground 分别执行订阅发布操作，对比输出结果。在第一个标签中，执行订阅操作。

{title="GraphQL Playground",lang="json"}
~~~~~~~~
subscription {
  messageCreated {
    message {
      id
      text
      createdAt
      user {
        id
        username
      }
    }
  }
}
~~~~~~~~

> Results will indicate the tab is listening for changes. In the second tab, log in a user:

执行订阅后，页面会提示正在监听。在第二个标签中，登录一个账户：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
mutation {
  signIn(login: "rwieruch", password: "rwieruch") {
    token
  }
}
~~~~~~~~

> Copy the token from the result, and then paste it to the HTTP headers panel in the same tab:

从结果中复制 token 并粘贴到当前标签的 HTTP headers 面板中：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "x-token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwiZW1haWwiOiJoZWxsb0Byb2Jpbi5jb20iLCJ1c2VybmFtZSI6InJ3aWVydWNoIiwicm9sZSI6IkFETUlOIiwiaWF0IjoxNTM0OTQ3NTYyLCJleHAiOjE1MzQ5NDkzNjJ9.mg4M6SfYPJkGf_Z2Zr7ztGNbDRDLksRWdhhDvTbmWbQ"
}
~~~~~~~~

> Then create a message in the second tab:

然后在第二个标签中发送一条信息：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
mutation {
  createMessage(text: "Does my subscription work?") {
    text
  }
}
~~~~~~~~

> Afterward, check your first tab again. It should show the created message:

之后，检查第一个标签。应该会看到刚才的信息：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "data": {
    "messageCreated": {
      "message": {
        "id": "4",
        "text": "Does my subscription work?",
        "createdAt": "2018-10-25T08:56:04.786Z",
        "user": {
          "id": "1",
          "username": "rwieruch"
        }
      }
    }
  }
}
~~~~~~~~

> You have implemented GraphQL subscriptions. It can be a challenge to wrap your head around them, but once you've worked through some basic operations, you can use these as a foundation to create real-time GraphQL applications.

你已经实现了 GraphQL 订阅。虽然要完全理清头绪还需要更多的挑战，不过一旦你完成了一些基本操作，你就可以用这些知识去作为 GraphQL 实时应用的基础了。

> ### Exercises:

### 练习:

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/eeb50f34a2569fa85141bf8ec3f8e9baaf670170)
> * Read more about [Subscriptions with Apollo Server](https://www.apollographql.com/docs/apollo-server/v2/features/subscriptions.html)
> * Watch a talk about [GraphQL Subscriptions](http://youtu.be/bn8qsi8jVew)

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/eeb50f34a2569fa85141bf8ec3f8e9baaf670170)
* 延伸阅读：[Apollo 服务端订阅](https://www.apollographql.com/docs/apollo-server/v2/features/subscriptions.html)
* 观看相关演讲 [GraphQL 订阅](http://youtu.be/bn8qsi8jVew)

> ## Testing a GraphQL Server

## 测试 GraphQL 服务器

> Testing often get overlooked in programming instruction, so this section will focus on to end-to-end (E2E) testing of a GraphQL server. While unit and integration tests are the fundamental pillars of the popular testing pyramid, covering all standalone functionalities of your application, E2E tests cover user scenarios for the entire application. An E2E test will assess whether a user is able to sign up for your application, or whether an admin user can delete other users. You don't need to write as many E2E tests, because they cover larger and more complex user scenarios, not just basic functionality. Also, E2E tests cover all the technical corners of your application, such as the GraphQL API, business logic, and databases.

在编程开发中测试经常被忽视，因此本节将重点介绍 GraphQL 服务器的端到端（E2E）测试。虽然单元和集成测试是测试金字塔的基本支柱，覆盖了应用程序的所有独立功能，但 E2E 测试则覆盖了整个应用程序的用户场景。在本例中，E2E 测试将确定用户是否能够注册你的应用程序，或管理员用户是否可以删除其他用户。你不需要编写尽可能多的 E2E 测试，因为它们覆盖的是更大，更复杂的用户场景，而不仅仅是基本功能。 此外，E2E 测试覆盖了应用程序的所有技术角落，例如 GraphQL API ，业务逻辑和数据库。

> ### GraphQL Server E2E Test Setup

### GraphQL 服务器 E2E 测试配置

> Programs called Mocha and Chai are really all you need to test the application we've created. Mocha is a test runner that lets you execute tests from an npm script, while providing an organized testing structure; Chai gives you all the functionalities to make assertions, e.g. "Expect X to be equal to Y" based on real-world scenarios and run through them.

这里使用测试组件 Mocha 和 Chai 用来测试我们的程序。Mocha 是一个测试运行器，它允许你从 npm 脚本执行测试，同时提供有组织的测试结构；Chai为你提供很多断言方法，例如： 基于真实场景，“期望 X 等于 Y ”。

{title="Command Line",lang="json"}
~~~~~~~~
npm install mocha chai --save-dev
~~~~~~~~

> To use these programs, you must first install a library called [axios](https://github.com/axios/axios) for making requests to the GraphQL API. When testing user sign-up, you can send a GraphQL mutation to the GraphQL API that creates a user in the database and returns their information.

要使用这些程序，必须先安装 [axios](https://github.com/axios/axios) ，以便向 GraphQL API 发出请求。在测试用户注册时，你可以将 GraphQL 变更操作发送到 GraphQL API，然后在数据库中创建用户并返回其信息。

{title="Command Line",lang="json"}
~~~~~~~~
npm install axios --save-dev
~~~~~~~~

> Mocha is run using npm scripts in your *package.json* file. The pattern used here matches all test files with the suffix *.spec.js* within the *src/* folder.

Mocha 使用 *package.json* 文件中的 npm 脚本运行。 此模式会匹配 *src/* 文件夹中带有 *.spec.js* 后缀的所有测试文件。

{title="package.json",lang="javascript"}
~~~~~~~~
{
  ...
  "scripts": {
    "start": "nodemon --exec babel-node src/index.js",
# leanpub-start-insert
    "test": "mocha --require @babel/register 'src/**/*.spec.js'"
# leanpub-end-insert
  },
  ...
}
~~~~~~~~

> Don't forget to install the babel node package with `npm install @babel/register --save-dev`. That should be sufficient to run your first test. Add a *src/tests/user.spec.js* to your application. and write your first test there:

不要忘记使用 `npm install @babel/register --save-dev` 安装 babel 包，它应该足以让我们运行第一次测试。新建 *src/tests/user.spec.js* 文件，在里面写上你的第一个测试：

{title="src/tests/user.spec.js",lang="javascript"}
~~~~~~~~
import { expect } from 'chai';

describe('users', () => {
  it('user is user', () => {
# leanpub-start-insert
    expect('user').to.eql('user');
# leanpub-end-insert
  });
});
~~~~~~~~

> The test is executed by typing `npm test` into the command line. While it doesn't test any logic of your application, the test will verify that Mocha, Chai, and your new npm script are working.

在命令行中键入 `npm test` 来执行测试。虽然它不测试应用程序的任何逻辑，但将验证 Mocha、Chai 和你的 npm 脚本是否正常工作。

> Before you can write end-to-end tests for the GraphQL server, the database must be addressed. Since the tests run against the actual GraphQL server, so you only need to run against a test database rather than the production database. Add an npm script in the *package.json* to start the GraphQL server with a test database:

在为 GraphQL 服务器编写E2E 测试之前，必须先解决数据库问题。由于测试需要在真实的 GraphQL 服务器上运行，所以需要使用测试数据库而不是生产数据库。 在 *package.json* 中添加 npm 脚本以使用测试数据库启动 GraphQL 服务器：

{title="package.json",lang="javascript"}
~~~~~~~~
{
  ...
  "scripts": {
    "start": "nodemon --exec babel-node src/index.js",
# leanpub-start-insert
    "test-server": "TEST_DATABASE=mytestdatabase npm start",
# leanpub-end-insert
    "test": "mocha --require @babel/register 'src/**/*.spec.js'"
  },
  ...
}
~~~~~~~~

> The script must be started before the E2E GraphQL server tests. If the `TEST_DATABASE` environment flag is set, you have to adjust the database setup in the *src/models/index.js* file to use the test database instead:

这个脚本需要在 E2E 测试启动前执行。上面设置了 `TEST_DATABASE` 环境变量，还须调整 *src/models/index.js* 文件中的数据库配置以使用测试数据库：

{title="src/models/index.js",lang="javascript"}
~~~~~~~~
import Sequelize from 'sequelize';

const sequelize = new Sequelize(
# leanpub-start-insert
  process.env.TEST_DATABASE || process.env.DATABASE,
# leanpub-end-insert
  process.env.DATABASE_USER,
  process.env.DATABASE_PASSWORD,
  {
    dialect: 'postgres',
  },
);

...
~~~~~~~~

> You also need to make sure to create such a database. Mine is called *mytestdatabase* in the npm script, which I added in the command line with `psql` and `createdb` or `CREATE DATABASE`.

你还需要确保刚才 npm 脚本中的 *mytestdatabase* 数据库已经创建，在命令行中执行 `psql` 然后执行 `createdb` 或 `CREATE DATABASE` 来创建数据库.

> Finally, you must start with a seeded and consistent database every time you run a test server. To do this, set the database re-seeding flag to depend on the set test database environment variable in the *src/index.js* file:

最后，每次运行测试服务器时，必须从一个初始一致的数据库开始。 为此，请将数据库 re-seeding 设置为依赖 *src/index.js* 文件中配置的测试数据库环境变量：

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const isTest = !!process.env.TEST_DATABASE;
# leanpub-end-insert

# leanpub-start-insert
sequelize.sync({ force: isTest }).then(async () => {
  if (isTest) {
# leanpub-end-insert
    createUsersWithMessages(new Date());
  }

  httpServer.listen({ port: 8000 }, () => {
    console.log('Apollo Server on http://localhost:8000/graphql');
  });
});

...
~~~~~~~~

> Now you are ready to write tests against an actual running test sever (`npm run test-server`) that uses a consistently seeded test database. If you want to use async/await in your test environment, adjust your *.babelrc* file:

现在，你已准备好搭配种子测试数据库的测试服务器（`npm run test-server`）来编写测试了。 如果要在测试环境中使用 async/await ，还需修改 *.babelrc* 文件：

{title=".babelrc",lang="javascript"}
~~~~~~~~
{
  "presets": [
    [
      "@babel/preset-env", {
        "targets": {
          "node": "current"
        }
      }
    ]
  ]
}
~~~~~~~~

> Now you can write tests with asynchronous business logic with async/await.

现在你可以使用 async/await 来测试异步逻辑。

> ### Testing User Scenarios with E2E Tests

### 使用 E2E 来测试用户场景

> Every E2E test sends an actual request with axios to the API of the running GraphQL test server. Testing your `user` GraphQL query would look like the following in the *src/tests/user.spec.js* file:

每个 E2E 测试都使用 axios 将真实的请求发送到正在运行的 GraphQL 测试服务器。接下来在 *src/tests/user.spec.js* 文件中编写 `user` 查询操作的测试：

{title="src/tests/user.spec.js",lang="javascript"}
~~~~~~~~
import { expect } from 'chai';

describe('users', () => {
# leanpub-start-insert
  describe('user(id: String!): User', () => {
    it('returns a user when user can be found', async () => {
      const expectedResult = {
        data: {
          user: {
            id: '1',
            username: 'rwieruch',
            email: 'hello@robin.com',
            role: 'ADMIN',
          },
        },
      };

      const result = await userApi.user({ id: '1' });

      expect(result.data).to.eql(expectedResult);
    });
  });
# leanpub-end-insert
});
~~~~~~~~

> Each test should be as straightforward as this one. You make a GraphQL API request with axios, expecting a query/mutation result from the API. Behind the scenes, data is read or written from or to the database. The business logic such as authentication, authorization, and pagination works in between. A request goes through the whole GraphQL server stack from API to database. An end-to-end test doesn't test an isolated unit (unit test) or a smaller composition of units (integration test), but the entire pipeline.

每个测试都应该像这个一样简单直接。使用 axios 发出 GraphQL API 请求，期望来自 API 的查询/变更结果。 在幕后，数据从数据库读取或写入，身份验证、授权和分页等业务逻辑在期间执行。 请求会走完整个从 API 到数据库的 GraphQL 服务器流程。 E2E 测试不会测试隔离单元（单元测试）或较小的单元组合（集成测试），而是整个管道（流程）。

> The `userApi` function is the final piece needed to set up effective testing for this application. It's not implemented in the test, but in another *src/tests/api.js* file for portability. In this file, you will find all your functions which can be used to run requests against your GraphQL test server.

`userApi` 函数是确保此测试有效的最后一部分。它没有在这个测试中实现，而是在 *src/tests/api.js* 文件中测试的，在此文件中，你将找到用于请求 GraphQL 测试服务器的所有函数（的测试）。

{title="src/tests/api.js",lang="javascript"}
~~~~~~~~
import axios from 'axios';

const API_URL = 'http://localhost:8000/graphql';

export const user = async variables =>
  axios.post(API_URL, {
    query: `
      query ($id: ID!) {
        user(id: $id) {
          id
          username
          email
          role
        }
      }
    `,
    variables,
  });
~~~~~~~~

> You can use basic HTTP to perform GraphQL operations across the network layer. It only needs a payload, which is the query/mutation and the variables. Beyond that, the URL of the GraphQL server must be known. Now, import the user API in your actual test file:

你可以使用基本的 HTTP 作为 GraphQL 的网络层。它需要一个有效载荷，即查询/变更和变量。 除此之外，还必须提供 GraphQL 服务器的 URL。 现在，在实际测试文件中导入用户 API：

{title="src/tests/user.spec.js",lang="javascript"}
~~~~~~~~
import { expect } from 'chai';

# leanpub-start-insert
import * as userApi from './api';
# leanpub-end-insert

describe('users', () => {
  describe('user(id: String!): User', () => {
    it('returns a user when user can be found', async () => {
      const expectedResult = {
        ...
      };

      const result = await userApi.user({ id: '1' });

      expect(result.data).to.eql(expectedResult);
    });
  });
});
~~~~~~~~

> To execute your tests now, run your GraphQL test server in the command line with `npm run test-server`, and execute your tests in another command line tab with `npm test`. The output should appear as such:

要立即执行测试，请在命令行中执行 `npm run test-server` 来运行 GraphQL 测试服务器，并在另一个命令行选项卡中键入 `npm test` 来执行测试。 输出应如下所示：

{title="Command Line",lang="javascript"}
~~~~~~~~
users
  user(id: ID!): User
    ✓ returns a user when user can be found (69ms)

1 passing (123ms)
~~~~~~~~

> If your output is erroneous, the console logs may help you figure out what went wrong. Another option is to take the query from the axios request and put it into GraphQL Playground. The error reporting in Playground might make it easier to find problems.

如果输出错误，控制台日志可能会帮助你找出问题所在。另一种选择是从 axios 请求中复制查询代码并将其放入GraphQL Playground 执行。 Playground 中的错误报告可以使查找问题变得更容易。

> That's your first E2E test against a GraphQL server. The next one uses the same API, and you can see how useful it is to extract the API layer as reusable functions. In your *src/tests/user.spec.js* file add another test:

这是你针对 GraphQL 服务器的第一次 E2E 测试。 下一个测试会使用相同的 API，你可以看到将 API 层提取为可重用函数是多么有用。在你的 *src/tests/user.spec.js* 文件中添加另一个测试：

{title="src/tests/user.spec.js",lang="javascript"}
~~~~~~~~
import { expect } from 'chai';

import * as userApi from './api';

describe('users', () => {
  describe('user(id: ID!): User', () => {
    it('returns a user when user can be found', async () => {
      const expectedResult = {
        ...
      };

      const result = await userApi.user({ id: '1' });

      expect(result.data).to.eql(expectedResult);
    });

# leanpub-start-insert
    it('returns null when user cannot be found', async () => {
      const expectedResult = {
        data: {
          user: null,
        },
      };

      const result = await userApi.user({ id: '42' });

      expect(result.data).to.eql(expectedResult);
    });
  });
# leanpub-end-insert
});
~~~~~~~~

> It is valuable to test the common path, but also less common edge cases. In this case, the uncommon path didn't return an error, but null for the user.

测试常见路径是有价值的，不太常见的边界场景也值得被测试。在这种边界场景下，查询不存在的用户不会返回错误，而是返回 null。

> Let's add another test that verifies non-admin user authorization related to deleting messages. Here you will implement a complete scenario from login to user deletion. First, implement the sign in and delete user API in the *src/tests/api.js* file:

让我们添加另一个测试来验证仅管理员才能删除用户。 在这里，你将实现从登录到删除用户的完整场景。 首先，在 *src/tests/api.js*  文件中实现登录和删除用户的 API：

{title="src/tests/api.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
export const signIn = async variables =>
  await axios.post(API_URL, {
    query: `
      mutation ($login: String!, $password: String!) {
        signIn(login: $login, password: $password) {
          token
        }
      }
    `,
    variables,
  });
# leanpub-end-insert

# leanpub-start-insert
export const deleteUser = async (variables, token) =>
  axios.post(
    API_URL,
    {
      query: `
        mutation ($id: ID!) {
          deleteUser(id: $id)
        }
      `,
      variables,
    },
    {
      headers: {
        'x-token': token,
      },
    },
  );
# leanpub-end-insert
~~~~~~~~

> The `deleteUser` mutation needs the token from the `signIn` mutation's result. Next, you can test the whole scenario by executing both APIs in your new E2E test:

`deleteUser` 需要使用 `signIn` 拿到的 token 。接下来，你可以通过在新的 E2E 测试中执行两个 API 来测试整个场景：

{title="src/tests/user.spec.js",lang="javascript"}
~~~~~~~~
import { expect } from 'chai';

import * as userApi from './api';

describe('users', () => {
  describe('user(id: ID!): User', () => {
    ...
  });

# leanpub-start-insert
  describe('deleteUser(id: String!): Boolean!', () => {
    it('returns an error because only admins can delete a user', async () => {
      const {
        data: {
          data: {
            signIn: { token },
          },
        },
      } = await userApi.signIn({
        login: 'ddavids',
        password: 'ddavids',
      });

      const {
        data: { errors },
      } = await userApi.deleteUser({ id: '1' }, token);

      expect(errors[0].message).to.eql('Not authorized as admin.');
    });
  });
# leanpub-end-insert
});
~~~~~~~~

> First, you are using the `signIn` mutation to login a user to the application. The login is fulfilled once the token is returned. The token can then be used for every other GraphQL operation. In this case, it is used for the `deleteUser` mutation. The mutation still fails, however, because the current user is not admin. You can try the same scenario on your own with an admin to test the simple path for reusing APIs.

首先，使用 `signIn` 登录用户，登录成功后返回 token。然后，此 token 可用于其他每个 GraphQL 操作。在这种场景下，它用于 `deleteUser` 。 但是，这个操作仍然失败了，因为当前用户不是管理员。你可以自行使用管理员账户并通过同样的 API 来再次尝试这个的场景。

{title="Command Line",lang="javascript"}
~~~~~~~~
users
  user(id: String!): User
    ✓ returns a user when user can be found (81ms)
    ✓ returns null when user cannot be found
  deleteUser(id: String!): Boolean!
    ✓ returns an error because only admins can delete a user (109ms)

3 passing (276ms)
~~~~~~~~

> These E2E tests cover scenarios for user domains, going through the GraphQL API over business logic to the database access. However, there is still plenty of room for alternatives. Consider testing other user domain-specific scenarios such as a user sign up (registration), providing a wrong password on sign in (login), or requesting one and another page of paginated messages for the message domain.

这些 E2E 测试涵盖了用户领域的场景，从 GraphQL API 业务逻辑到数据库访问。但是，仍有可测试的空间。考虑测试其他用户特定领域的场景，例如用户注册，在登录时提供错误的密码，或者在消息领域中请求一页和另一页的带分页的消息。

> This section only covered E2E tests. With Chai and Mocha at your disposal, you can also add smaller unit and integration tests for your different application layers (e.g. resolvers). If you need a library to spy, stub, or mock something, I recommend [Sinon](https://sinonjs.org) as a complementary testing library.

本节仅涉及 E2E 测试。使用 Chai 和 Mocha ，你还可以为不同的应用层（例如：解析器层）添加更小的单元和集成测试。如果你需要一个库来实现测试替身、测试桩或模拟某些东西，我建议使用 [Sinon](https://sinonjs.org) 作为补充测试库。

> ### Exercises:

### 练习:

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/d11e0487085e014170146ec7479d0154c4a6fce4)
> * Implement tests for the message domain similar to the user domain
> * Write more fine-granular unit/integration tests for both domains
> * Read more about [GraphQL and HTTP](https://graphql.github.io/learn/serving-over-http/)
> * Read more about [Mocking with Apollo Server](https://www.apollographql.com/docs/apollo-server/v2/features/mocking.html)

* 查看 [本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/d11e0487085e014170146ec7479d0154c4a6fce4)
* 为消息领域实现与用户领域类似的测试
* 为这两个领域编写更精细的单元/集成测试
* 延伸阅读： [GraphQL and HTTP](https://graphql.github.io/learn/serving-over-http/)
* 延伸阅读： [Mocking with Apollo Server](https://www.apollographql.com/docs/apollo-server/v2/features/mocking.html)

> ## Batching and Caching in GraphQL with Data Loader

## 在 GraphQL 中使用批处理和缓存

> The section is about improving the requests to your database. While only one request (e.g. a GraphQL query) hits your GraphQL API, you may end up with multiple database reads and writes to resolve all fields in the resolvers. Let's see this problem in action using the following query in GraphQL Playground:

这一部分内容介绍了如何优化数据库请求。每当有请求 (比如一个 GraphQL 查询) 调用 GraphQL API，可能在 resolver 层需要有多数据库执行读和写操作。我们在 GraphQL Playground 中使用以下的查询来看看有什么问题：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
query {
  messages {
    edges {
      user {
        username
      }
    }
  }
}
~~~~~~~~

> Keep the query open, because you use it as a case study to make improvements. Your query result should be similar to the following:

为了把这个实例用于学习优化，请保持这个查询是打开的状态。查询的返回结果如下:

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "data": {
    "messages": {
      "edges": [
        {
          "user": {
            "username": "ddavids"
          }
        },
        {
          "user": {
            "username": "ddavids"
          }
        },
        {
          "user": {
            "username": "rwieruch"
          }
        }
      ]
    }
  }
}
~~~~~~~~

> In the command line for the running GraphQL server, four requests were made to the database:

在启动 GraphQL 服务的命令行里，数据库接收到了四个请求：

{title="Command Line",lang="javascript"}
~~~~~~~~
Executing (default): SELECT "id", "text", "createdAt", "updatedAt", "userId" FROM "messages" AS "message" ORDER BY "message"."createdAt" DESC LIMIT 101;

Executing (default): SELECT "id", "username", "email", "password", "role", "createdAt", "updatedAt" FROM "users" AS "user" WHERE "user"."id" = 2;

Executing (default): SELECT "id", "username", "email", "password", "role", "createdAt", "updatedAt" FROM "users" AS "user" WHERE "user"."id" = 2;

Executing (default): SELECT "id", "username", "email", "password", "role", "createdAt", "updatedAt" FROM "users" AS "user" WHERE "user"."id" = 1;
~~~~~~~~

> There is one request made for the list of messages, and three requests for each individual user. That's the nature of GraphQL. Even though you can nest your GraphQL relationships and query structure, there will still be database requests. Check the resolvers for the message user in your *src/resolvers/message.js* file to see where this is happening. At some point, you may run into performance bottlenecks when nesting GraphQL queries or mutations too deeply, because a lot of items need to be retrieved from your database.

有一个请求来获取 message 列表，三个请求分别获取每个用户信息。这是 GraphQL 的特性。尽管可以嵌套地使用 GraphQL 关系和 query 结构, 数据库请求还是会存在. 在 *src/resolvers/message.js* 文件中查看 resolever 可以找到这些请求。在使用深层内嵌查询或者更改的时候，你可能会遇到性能瓶颈，那是因为有特别多的数据需要从数据库读取。

> In the following, you will optimize these database accesses with batching. It's a strategy used for a GraphQL server and its database, but also for other programming environments. Compare the query result in GraphQL Playground and your database output in the command line.

下面这个示例演示了如何用批处理来优化数据库操作。这是个用于优化 GraphQL 服务器和数据库的策略，同时也适用于其他编程环境。在命令行里可以对比 GraqhQL Playground 和数据库的查询结果。

> There are two improvements that can be made with batching. First, one author of a message is retrieved twice from the database, which is redundant. Even though there are multiple messages, the author of some of these messages can be the same person. Imagine this problem on a larger scale for 100 messages between two authors in a chat application. There would be one request for the 100 messages and 100 requests for the 100 authors of each message, which would lead to 101 database accesses. If duplicated authors are retrieved only once, it would only need one request for the 100 messages and 2 requests for the authors, which reduces the 101 database hits to just 3. Since you know all the identifiers of the authors, these identifiers can be batched to a set where none are repeated. In this case, the two authors a list of [2, 2, 1] identifiers become a set of [2, 1] identifiers.

使用批处理会有两点可以得到改善。首先，有一个 message 的 user 数据从数据库中获取了两次，这就是多余的操作。即使是有多条 message，但是有一些可能来自于相同作者。设想一个更大数据的聊天应用场景，假如现在有在两个用户之间有 100 条消息，那么就需要 1 个请求来查询 100 条消息和 100 个请求来为每条消息查询作者，一共就需要 101 个数据库查询。假如重复的作者只查询一次，那就只需要 1 条查询来查找消息和 2 条查询来查找作者，这样一来总查询就只有 3 条。由于每个作者的 id 是可知的，这些 id 就可以批处理到没有重复数据的集合中存放。在当前的这个示例中，作者的 id 就从 [2, 2, 1] 简化为 [2, 1]。

> Second, every author is read from the database individually, even though the list is purged from its duplications. Reading all authors with only one database request should be possible, because at the time of the GraphQL API request with all messages at your disposal, you know all the identifiers of the authors. This decreases your database accesses from 3 to 2, because now you only request the list of 100 messages and its 2 authors in two requests.

另外，即使我们现在消除了重复的读取，每一个作者都是单独从数据库中读取的。因为提交 GraphQL 的时候携带了所有信息，包括所有作者的 id，那么只用一个数据库请求读取所有的作者信息是可行的。这样就可以把 3 个数据库请求缩减到 2 个，其中一个请求用来拉取 100 条消息，另一个请求用来拉取所有的作者信息。

> The same two principals can be applied to the 4 database accesses which should be decreased to 2. On a smaller scale, it might not have much of a performance impact, but for 100 messages with the 2 authors, it reduces your database accesses significantly. That's where Facebook's open source [dataloader](https://github.com/facebook/dataloader) becomes a vital tool. You can install it via npm on the command line:

这两个准则可以应用在示例的 4 个数据库请求上，这样就可以缩减到 2 个数据库请求。在小规模请求上，影响可能比较小，但是在有 100 条消息和 2 个作者的情况下，性能提升就很明显了。这就是 Facebook 的开源工具 [dataloader](https://github.com/facebook/dataloader) 成为重要工具的原因。可以使用 npm 借助如下的命令安装：

{title="Command Line",lang="json"}
~~~~~~~~
npm install dataloader --save
~~~~~~~~

> Now, in your *src/index.js* file you can import and make use of it:

然后在 *src/index.js* 里这样导入并使用：

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import DataLoader from 'dataloader';
# leanpub-end-insert

...

# leanpub-start-insert
const batchUsers = async (keys, models) => {
  const users = await models.User.findAll({
    where: {
      id: {
        $in: keys,
      },
    },
  });

  return keys.map(key => users.find(user => user.id === key));
};
# leanpub-end-insert

const server = new ApolloServer({
  typeDefs: schema,
  resolvers,
  ...
  context: async ({ req, connection }) => {
    if (connection) {
      ...
    }

    if (req) {
      const me = await getMe(req);

      return {
        models,
        me,
        secret: process.env.SECRET,
# leanpub-start-insert
        loaders: {
          user: new DataLoader(keys => batchUsers(keys, models)),
        },
# leanpub-end-insert
      };
    }
  },
});

...
~~~~~~~~

> The loaders act as abstraction on top of the models, and can be passed as context to the resolvers. The user loader in the following example is used instead of the models directly.

loaders 实际上是 models 的上层抽象，可以作为上下文传递给 resolver，下列示例中直接用 user loader 代替了 model。

> Now we'll consider the function as argument for the DataLoader instantiation. The function gives you access to a list of keys in its arguments. These keys are your set of identifiers, purged of duplication, which can be used to retrieve items from a database. That's why keys (identifiers) and models (data access layer) are passed to the `batchUser()` function. The function then takes the keys to retrieve the entities via the model from the database. By the end of the function, the keys are mapped in the same order as the retrieved entities. Otherwise, it's possible to return users right after their retrieval from the database, though they have a different order than the incoming keys. As a result, users need to be returned in the same order as their incoming identifiers (keys).

现在我们将该函数视为 DataLoader 实例化的参数。该函数使你可以访问其参数中的 keys 数组。这些键就是已清除重复的标识符集合，可用于从数据库中拉取数据。这就是将标识符（id）和模型（数据访问层）传递给 `batchUser()` 函数的原因。然后，该函数使用 keys 在数据库中的模型拉取实体。在函数结束时，keys 数组会按顺序映射成获得的数据实体。否则，如果在从数据库中拉取数据实体之后立即返回，就会导致它们的顺序与传入 keys 列表不同。因此，数据实体需要以传入的 keys 列表的相同顺序返回。

> That's the setup for the loader, an improved abstraction on top of the model. Now, since you are passing the loader for the batched user retrieval as context to the resolvers, you can make use of it in the *src/resolvers/message.js* file:
 Now, s you can make use of it in the *src/resolvers/message.js* file:

以上就是 loader 的设置方法，一种在模型上层有效的抽象。由于把 loader 作为上下文传递给了 resolver，现在可以在 *src/resolvers/message.js* 中这样使用它：

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
...

export default {
  Query: {
    ...
  },

  Mutation: {
    ...
  },

  Message: {
# leanpub-start-insert
    user: async (message, args, { loaders }) => {
      return await loaders.user.load(message.userId);
# leanpub-end-insert
    },
  },

  Subscription: {
    ...
  },
};
~~~~~~~~

> While the `load()` function takes each identifier individually, it will batch all these identifiers into one set and request all users at the same time. Try it by executing the same GraphQL query in GraphQL Playground. The result should stay the same, but you should only see 2 instead of 4 requests to the database in your command-line output for the GraphQL server:

当  `load()` 函数每次调用，获取 id 时，它会在同时把这些 id 批处理到一个 set 里面，然后一次性请求所有数据。你可以在 GraphQL Playground 里使用同样的查询语句来进行尝试。查询结果应该是相同的，但是你应该在 GraphQL 服务器的命令行输出里看到 2 个而不是 4 个数据库请求：

{title="Command Line",lang="javascript"}
~~~~~~~~
Executing (default): SELECT "id", "text", "createdAt", "updatedAt", "userId" FROM "messages" AS "message" ORDER BY "message"."createdAt" DESC LIMIT 101;

Executing (default): SELECT "id", "username", "email", "password", "role", "createdAt", "updatedAt" FROM "users" AS "user" WHERE "user"."id" IN (2, 1);
~~~~~~~~

> That's the benefit of the batching improvement: instead of fetching each (duplicated) user on its own, you fetch them all at once in one batched request with the dataloader package.

这就是使用批处理带来的好处： 使用 dataloader 这个三方库，一次性地请求所有需要的数据而不是单独请求重复的数据。

> Now let's get into caching. The dataloader package we installed before also gives the option to cache requests. It doesn't work yet, though; try to execute the same GraphQL query twice and you should see the database accesses twice on your command line.

下面介绍缓存原理。我们刚刚安装的 dataloader 包同时也提供了缓存请求的选项，虽然它目前还没有生效。尝试执行两次相同的 GraphQL 查询，你可以在命令行里看到两次数据库请求。

{title="Command Line",lang="javascript"}
~~~~~~~~
Executing (default): SELECT "id", "text", "createdAt", "updatedAt", "userId" FROM "messages" AS "message" ORDER BY "message"."createdAt" DESC LIMIT 101;
Executing (default): SELECT "id", "username", "email", "password", "role", "createdAt", "updatedAt" FROM "users" AS "user" WHERE "user"."id" IN (2, 1);

Executing (default): SELECT "id", "text", "createdAt", "updatedAt", "userId" FROM "messages" AS "message" ORDER BY "message"."createdAt" DESC LIMIT 101;
Executing (default): SELECT "id", "username", "email", "password", "role", "createdAt", "updatedAt" FROM "users" AS "user" WHERE "user"."id" IN (2, 1);
~~~~~~~~

> That's happening because a new instance of the dataloader is created within the GraphQL context for every request. If you move the dataloader instantiation outside, you get the caching benefit of dataloader for free:

这是因为每次请求都会用 GraqhQL 上下文创建新的 loader 实例，但是如果把 dataloader 初始化语句移到外面，你就可以体验到缓存带来的好处了：

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const userLoader = new DataLoader(keys => batchUsers(keys, models));
# leanpub-end-insert

const server = new ApolloServer({
  typeDefs: schema,
  resolvers,
  ...
  context: async ({ req, connection }) => {
    if (connection) {
      ...
    }

    if (req) {
      const me = await getMe(req);

      return {
        models,
        me,
        secret: process.env.SECRET,
        loaders: {
# leanpub-start-insert
          user: userLoader,
# leanpub-end-insert
        },
      };
    }
  },
});

...
~~~~~~~~

> Try to execute the same GraphQL query twice again. This time you should see only a single database access, for the places where the loader is used; the second time, it should be cached.

再次使用相同的 GraphQL 查询两遍，这次你可以观察到，对于使用 loader 的地方，只会有一次数据库请求，因为它被缓存起来了。

{title="Command Line",lang="javascript"}
~~~~~~~~
Executing (default): SELECT "id", "text", "createdAt", "updatedAt", "userId" FROM "messages" AS "message" ORDER BY "message"."createdAt" DESC LIMIT 101;
Executing (default): SELECT "id", "username", "email", "password", "role", "createdAt", "updatedAt" FROM "users" AS "user" WHERE "user"."id" IN (2, 1);

Executing (default): SELECT "id", "text", "createdAt", "updatedAt", "userId" FROM "messages" AS "message" ORDER BY "message"."createdAt" DESC LIMIT 101;
~~~~~~~~

> In this case, the users are not read from the database twice, only the messages, because they are not using a dataloader yet. That's how you can achieve caching in GraphQL with dataloaders. Choosing a caching strategy isn't quite as simple. For example, if a cached user is updated in between actions, the GraphQL client application still queries the cached user.

在这个示例中，user 并没有分两次从数据库读取，只有 message 是的，因为它没有使用 dataloader。这就是如何在 GraphQL 中使用缓存的方法了。选择缓存策略不是一件简单的事情，比如说，如果两次查询中间 user 得到了更新，那么客户端应用依然会得到被缓存的 user。

> It's difficult to find the right timing for invalidating the cache, so I recommended performing the dataloader instantiation with every incoming GraphQL request. You lose the benefit of caching over multiple GraphQL requests, but still use the cache for every database access with one incoming GraphQL request. The dataloader package expresses it like this: *"DataLoader caching does not replace Redis, Memcache, or any other shared application-level cache. DataLoader is first and foremost a data loading mechanism, and its cache only serves the purpose of not repeatedly loading the same data in the context of a single request to your Application."* If you want to get into real caching on the database level, give [Redis](https://redis.io/) a shot.


选择合适的时机清除缓存比较困难，所以我建议在每个 GraphQL 请求的时候重新初始化 dataloader。你将会失去在多个请求之间缓存数据的能力，但是还是可以在一个 GraphQL 请求里缓存所有的数据库请求。在 dataloader 包里这样描述： *“Dataloader 缓存不是为了替代 Redis，Memcache，或者是别的任何应用层缓存组件。Dataloader 首先是一种数据加载机制，它的缓存目的在于同一个请求上下文中，避免重复请求相同的数据。”*  假如你需要真正的数据库级别缓存，可以试试[Redis](https://redis.io/) 。

> Outsource the loaders into a different folder/file structure. Put the batching for the individual users into a new *src/loaders/user.js* file:

将 loader 封装到不同的文件夹/文件结构里。把各个用户的批处理放入新建的 *src/loaders/user.js* file: 文件中：

{title="src/loaders/user.js",lang="javascript"}
~~~~~~~~
export const batchUsers = async (keys, models) => {
  const users = await models.User.findAll({
    where: {
      id: {
        $in: keys,
      },
    },
  });

  return keys.map(key => users.find(user => user.id === key));
};
~~~~~~~~

> And in a new *src/loaders/index.js* file export all the functions:

创建新的 *src/loaders/index.js* 文件导出所有函数：

{title="src/loaders/index.js",lang="javascript"}
~~~~~~~~
import * as user from './user';

export default { user };
~~~~~~~~

> Finally, import it in your *src/index.js* file and use it:

最后导入 *src/index.js* 使用：

{title="src/index.js",lang="javascript"}
~~~~~~~~
...
import DataLoader from 'dataloader';

...
# leanpub-start-insert
import loaders from './loaders';
# leanpub-end-insert

...

const server = new ApolloServer({
  typeDefs: schema,
  resolvers,
  ...
  context: async ({ req, connection }) => {
    if (connection) {
      ...
    }

    if (req) {
      const me = await getMe(req);

      return {
        models,
        me,
        secret: process.env.SECRET,
        loaders: {
# leanpub-start-insert
          user: new DataLoader(keys =>
            loaders.user.batchUsers(keys, models),
          ),
# leanpub-end-insert
        },
      };
    }
  },
});

...
~~~~~~~~

> Remember to add the loader to your subscriptions, in case you use them there:

别忘了把 loader 添加到订阅，那里可能会用到：

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

const server = new ApolloServer({
  typeDefs: schema,
  resolvers,
  ...
  context: async ({ req, connection }) => {
    if (connection) {
      return {
        models,
# leanpub-start-insert
        loaders: {
          user: new DataLoader(keys =>
            loaders.user.batchUsers(keys, models),
          ),
        },
# leanpub-end-insert
      };
    }

    if (req) {
      ...
    }
  },
});

...
~~~~~~~~

> Feel free to add more loaders on your own, maybe for the message domain. The practice can provide useful abstraction on top of your models to allow batching and request-based caching.

请随意在 message 域添加你自己的 loader。这个实践提供有用的 model 上层抽象，用来支持批处理和基于请求的缓存。

> ### Exercises:

### 练习:

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/9ff0542f620a0d9939c1adcbd21951f8fc1693f4)
> * Read more about [GraphQL and Dataloader](https://www.apollographql.com/docs/graphql-tools/connectors.html#dataloader)
> * Read more about [GraphQL Best Practices](https://graphql.github.io/learn/best-practices/)

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/9ff0542f620a0d9939c1adcbd21951f8fc1693f4)
* 阅读更多关于[GraphQL 和 Dataloader](https://www.apollographql.com/docs/graphql-tools/connectors.html#dataloader)
* 阅读更多关于[GraphQL 最佳实践](https://graphql.github.io/learn/best-practices/)

> ## GraphQL Server + PostgreSQL Deployment to Heroku

## GraphQL 服务器 + PostgreSQL 部署到 Heroku

> Eventually you want to deploy the GraphQL server online, so it can be used in production. In this section, you learn how to deploy a GraphQL server to Heroku, a platform as a service for hosting applications. Heroku allows PostgreSQL as well.

最后你想要把 GraphQL 部署到线上环境，用以满足生产环境需求。在这一章节你将学到如何把 GraphQL 服务器部署到 Heroku，一个用来托管应用的平台即服务应用。Heroku 同时也支持 PostgreSQL。

> This section guides you through the process in the command line. For the visual approach check this [GraphQL server on Heroku deployment tutorial](https://www.apollographql.com/docs/apollo-server/deployment/heroku.html) which, however, doesn't include the PostgreSQL database deployment.

这一章节提供基于命令行工具的快速上手教程。这里可以查看视频教程[Heroku 部署 GraphQL 服务器教程](https://www.apollographql.com/docs/apollo-server/deployment/heroku.html)。视频里没有包含 PostgreSQL 数据库有关内容。

> Initially you need to complete three requirements to use Heroku:

使用 Heroku 之前请确保满足三个条件：

> * [Install git for your command line and push your project to GitHub](https://www.robinwieruch.de/git-essential-commands/)
> * Create an account for [Heroku](https://www.heroku.com/)
> * Install the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) for accessing Heroku's features on the command line

* [安装 git 命令行工具并把项目上传到 GitHub](https://www.robinwieruch.de/git-essential-commands/)
* 在 [Heroku](https://www.heroku.com/) 上创建帐号
* 安装 [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) 从而在命令行里使用 Heroku 功能


> In the command line, verify your Heroku installation with `heroku version`. If there is a valid installation, sign in to your Heroku account with `heroku login`. That's it for the general Heroku setup. In your project's folder, create a new Heroku application and give it a name:

在命令行里使用 `heroku version` 来检测 Heroku 是否安装成功。如果成功安装，使用 `heroku login` 来登入你的帐号。这就完成了基本的 Heroku 设置。在你的项目文件夹下，创建新的 Heroku 应用并命名：

{title="Command Line",lang="javascript"}
~~~~~~~~
heroku create graphql-server-node-js
~~~~~~~~

> Afterward, you can also install the PostgreSQL add-on for Heroku on the command line for your project:

然后你也可以为你的项目在 Heroku 上安装 PostgreSQL 插件：

{title="Command Line",lang="javascript"}
~~~~~~~~
heroku addons:create heroku-postgresql:hobby-dev
~~~~~~~~

> It uses the [hobby tier](https://devcenter.heroku.com/articles/heroku-postgres-plans#hobby-tier), a free application that can be upgraded as needed. Output for the PostgreSQL add-on installation should be similar to:

它使用 [hobby tier](https://devcenter.heroku.com/articles/heroku-postgres-plans#hobby-tier)，一个可以按需升级的免费软件。PostgreSQL 插件的安装输出应该是像这样：

{title="Command Line",lang="javascript"}
~~~~~~~~
Creating heroku-postgresql:hobby-dev on ⬢ graphql-server-node-js... free
Database has been created and is available
 ! This database is empty. If upgrading, you can transfer
 ! data from another database with pg:copy
Created postgresql-perpendicular-34121 as DATABASE_URL
Use heroku addons:docs heroku-postgresql to view documentation
~~~~~~~~

> Check the [Heroku PostgreSQL documentation](https://devcenter.heroku.com/articles/heroku-postgresql) for more in depth instructions for your database setup.

查看 [Heroku PostgreSQL 文档](https://devcenter.heroku.com/articles/heroku-postgresql) 以获得关于数据库安装的更多信息。

> You are ready to take your application online. With the PostgreSQL add-on, you received a database URL as well. You can find it with `heroku config`. Now, let's step into your GraphQL server's code to make a couple of adjustments for production. In your *src/models/index.js*, you need to decide between development (coding, testing) and production (live) build. Because you have a new environment variable for your database URL, you can use this to make the decision:

你已经准备好将自己的应用部署上线了。使用 PostgreSQL 插件，你应该也有个数据库 URL。可以通过 `heroku config` 找到它。现在需要检查 GraphQL 服务器的代码，为生产环境做一些改动。在 *src/models/index.js* 文件里，你需要在 development(coding, testing) 和 production(live) 构建中选择一个。由于你有一个新的环境变量来保存数据库 URL，可以这样来修改环境：

{title="src/models/index.js",lang="javascript"}
~~~~~~~~
import Sequelize from 'sequelize';

# leanpub-start-insert
let sequelize;
if (process.env.DATABASE_URL) {
  sequelize = new Sequelize(process.env.DATABASE_URL, {
    dialect: 'postgres',
  });
} else {
# leanpub-end-insert
  sequelize = new Sequelize(
    process.env.TEST_DATABASE || process.env.DATABASE,
    process.env.DATABASE_USER,
    process.env.DATABASE_PASSWORD,
    {
      dialect: 'postgres',
    },
  );
# leanpub-start-insert
}
# leanpub-end-insert

...
~~~~~~~~

> If you check your *.env* file, you will see the `DATABASE_URL` environment variable isn't there. But you should see that it is set as Heroku environment variable with `heroku config:get DATABASE_URL`. Once your application is live on Heroku, your environment variables are merged with Heroku's environment variables, which is why the `DATABASE_URL` isn't applied for your local development environment.

查看 *.env* 文件，你可以看到 `DATABASE_URL` 环境变量是不存在的。但是使用 `heroku config:get DATABASE_URL` 可以看到它被设置为 Heroku 环境变量了。一旦你的项目在 Heroku 上运行，你的环境变量是和 Heroku 的环境变量整合到一起的。这就是 `DATABASE_URL` 在你的本地开发环境不可见的原因。

> Another environment variable used in the *src/index.js* file is called *SECRET* for your authentication strategy. If you haven't included an *.env* file in your project's version control (see .gitignore), you need to set the `SECRET` for your production code in Heroku using `heroku config:set SECRET=wr3r23fwfwefwekwself.2456342.dawqdq`. The secret is just made up and you can choose your own custom string for it.

另一个在 *src/index.js* 文件中使用的环境变量是用于授权策略的 *SECRET*。如果你还没有在版本控制中添加 *.env* 文件(查看 .gitignore)，你就需要在 Heroku 的项目中使用 `heroku config:set SECRET=wr3r23fwfwefwekwself.2456342.dawqdq` 来设置 `SECRET`。这里的密钥是胡编的，你可以用自己的密钥代替。

> Also, consider the application's port in the *src/index.js* file. Heroku adds its own `PORT` environment variable, and you should use the port from an environment variable as a fallback.

另外需要在 *src/index.js* 里配置应用的端口。Heroku 有自己的 `PORT` 环境变量，并且你应该使用环境变量中的端口作为默认值。

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const port = process.env.PORT || 8000;
# leanpub-end-insert

sequelize.sync({ force: isTest }).then(async () => {
  if (isTest) {
    createUsersWithMessages(new Date());
  }

# leanpub-start-insert
  httpServer.listen({ port }, () => {
    console.log(`Apollo Server on http://localhost:${port}/graphql`);
# leanpub-end-insert
  });
});

...
~~~~~~~~

> Finally, decide whether you want to start with a seeded database or an empty database on Heroku PostgreSQL. If it is to be seeded, add an extra flag to the seeding:

最后，决定是否要在 Heroku PostgreSQL 上使用种子数据库或空数据库。如果使用种子数据库，需要添加额外的标记：

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

const isTest = !!process.env.TEST_DATABASE;
# leanpub-start-insert
const isProduction = !!process.env.DATABASE_URL;
# leanpub-end-insert
const port = process.env.PORT || 8000;

# leanpub-start-insert
sequelize.sync({ force: isTest || isProduction }).then(async () => {
  if (isTest || isProduction) {
# leanpub-end-insert
    createUsersWithMessages(new Date());
  }

  httpServer.listen({ port }, () => {
    console.log(`Apollo Server on http://localhost:${port}/graphql`);
  });
});

...
~~~~~~~~

> Remember to remove the flag after, or the database will be purged and seeded with every deployment. Depending on development or production, you are choosing a database, seeding it (or not), and selecting a port for your GraphQL server. Before pushing your application to Heroku, push all recent changes to your GitHub repository. After that, push all the changes to your Heroku remote repository as well, since you created a Heroku application before: `git push heroku master`. Open the application with `heroku open`, and add the `/graphql` suffix to your URL in the browser to open up GraphQL Playground. If it doesn't work, check the troubleshoot area below.

记得之后删除 flag，否者每次部署数据库都会被清空然后用种子数据填充。根据应用处在 development 或者 production 环境，你都要选择一个数据库，使用数据填充 (也可以用空数据库)，并且为 GraphQL 选择一个端口。在推送到 Heroku 之前，把所有的改动 push 到 github 仓库。然后，由于之前已经创建了 Heroku 应用，使用 `git push heroku master` 把所有改动也推送到 Heroku 远程仓库。使用 `heroku open` 打开 Heroku 应用，添加 `/graphql` 后缀到浏览器的 URL 中来打开 GraphQL Playground。如果打开失败，请阅读下面的故障排除部分。

> Depending on your seeding strategy, your database will either be empty or contain seeded data. If its empty, register a user and create messages via GraphQL mutations. If its seeded, request a list of messages with a GraphQL query.

根据你选择的数据填充策略，你的数据库将为空或包含种子数据。如果为空，则使用 GraphQL mutation 注册用户并且创建消息。如果有种子数据，则使用 GraphQL 查询请求消息列表。

> Congratulations, your application should be live now. Not only is your GraphQL server running on Heroku, but your PostgreSQL database. Follow the exercises to learn more about Heroku.

恭喜，你的应用已经正常上线了。现在 GraphQL 服务器和 PostgreSQL 都已经在Heroku上成功运行。按照下面的练习了解有关 Heroku 的更多信息。

> ### Heroku Troubleshoot

### Heroku 故障筛查

> It can happen that the GraphQL schema is not available in GraphQL Playground for application in production. It's because the `introspection` flag for Apollo Server is disabled. In order to fix it, you can set it to true. Another improvement to add may be the `playground` flag to enable GraphQL Playground for Heroku:

生产环境下可能会发生 GraphQL schema 在 GraphQL Playground 中不可用的情况，这是因为禁用了 Apollo Serve 的 `introspection` 标志。你可以将其设置为 true 来修复。另一个改进是添加 `playground` 标志来为 Heroku 启用 GraphQL Playground：

{title="src/index.js",lang="javascript"}
~~~~~~~~
const server = new ApolloServer({
# leanpub-start-insert
  introspection: true,
  playground: true,
# leanpub-end-insert
  typeDefs: schema,
  resolvers,
  ...
});
~~~~~~~~

> Another issue may be that Heroku doesn't install the dev dependencies for production. Although it does install the dev dependencies for building the application on Heroku, it purges the dev dependencies afterward. However, in our case, in order to start the application (npm start script), we rely on a few dev dependencies that need to be available in production. [You can tell Heroku to keep the dev dependencies:](https://devcenter.heroku.com/articles/nodejs-support#package-installation)

另一个问题可能是 Heroku 没有为生产环境安装 dev 依赖库。虽然在 Heroku 上构建应用时确实安装了 dev 依赖库，但在之后会被自动清除。但是在我们的示例中，为了启动应用程序 (npm start script)，在生产环境中需要的几个dev依赖库。参考：[配置 Heroku 保存 dev 依赖库:](https://devcenter.heroku.com/articles/nodejs-support#package-installation)

{title="Command Line",lang="javascript"}
~~~~~~~~
heroku config:set NPM_CONFIG_PRODUCTION=false YARN_PRODUCTION=false
~~~~~~~~

> In a real world scenario, you would want to use something else to start your application and not rely on any dev dependencies.

在现实环境下，一般使用其他东西来启动应用程序，而是不依赖于任何 dev 依赖库。

> ### Exercises:

### 练习:


> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/9dbfb30226cdc4843adbcc09d16871b2a902a4d3)
> * Feedback whether the troubleshooting area for Heroku was useful is very appreciated
> * Create sample data in your production database with GraphQL Playground
> * Get familiar with the [Heroku Dashboard](https://dashboard.heroku.com/apps)
>   * Find your application's logs
>   * Find your application's environment variables
> * access your PostgreSQL database on Heroku with `heroku pg:psql`

* 查看[本章源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/9dbfb30226cdc4843adbcc09d16871b2a902a4d3)
* 欢迎给我们关于 Heroku 的故障排除区域是否有用的反馈
* 使用 GraphQL Playground 创建生产环境测试数据
* 熟悉[Heroku Dashboard](https://dashboard.heroku.com/apps)
  * 找到应用日志
  * 找到应用环境变量
* 使用`heroku pg:psql` 访问 PostgreSQL 数据库

<hr class="section-divider">

> You built a sophisticated GraphQL server boilerplate project with Express and Apollo Server. You should have learned that GraphQL isn't opinionated about various things, and about authentication, authorization, database access, and pagination. Most of the operations we learned were more straightforward because of Apollo Server over the GraphQL reference implementation in JavaScript. That's okay, because many people are using Apollo Server to build GraphQL servers. Use this application as a starter project to realize your own ideas, or find my starter project with a GraphQL client built in React in [this GitHub repository](https://github.com/the-road-to-graphql/fullstack-apollo-express-postgresql-boilerplate).

你已经使用 Express 和 Apollo Server 构建了一个复杂的 GraphQL 服务器示例项目。也应该已经了解到 GraphQL 不会强制要求身份验证，授权，数据库访问和分页等等。由于 Apollo Server 使用了基于 JavaScript 实现的 GraphQL，因此我们学到的大多数操作都更直接。没关系，因为很多人都在使用 Apollo Server 来构建 GraphQL 服务器。使用此应用作为入门项目来实现你自己的想法，或者使用我的入门项目，也就是 [这个 GitHub 仓库](https://github.com/the-road-to-graphql/fullstack-apollo-express-postgresql-boilerplate) 中用 React 实现的 GraphQL 客户端。
