# 在 GraphQL 和 Apollo 服务端中使用 Node.js 

在本章你将会使用 GraphQL 和 Apollo 服务端构建服务端。FaceBook 用 JavaScript 实现了 GraphQL 查询语言的参考实现，而 Apollo 服务端基于此实现，以便用 JavaScript 构建 GraphQL 服务端变得更简单。由于 GraphQL 是一种查询语言，所以它的传输层和数据格式不是固定的。在通过 HTTP 与 JSON 的客户-服务端传输中，它常常被视为流行的 REST 架构的替代品，虽然 GraphQL 没有刻意提到这一点。

在章节的最后，你能拥有一个完全工作的 GraphQL 服务端脚手架项目，这个项目包括身份验证，授权，连接着数据库的数据访问层，像用户、消息这样的具体的领域实体，不同的分页策略和由订阅带来的实时推送功能。你可以在这个 GitHub 代码库：[使用 React 和 Express 的全栈 Apollo 脚手架项目](https://github.com/the-road-to-graphql/fullstack-apollo-express-postgresql-boilerplate)找到一个已经实现了这些功能的服务端解决方案和一个由 React 实现的客户端解决方案。我认为这是一个理想的实现自己想法的入门项目。

当你在和我一起构建这个应用程序时，推荐使用内置的 GraphQL 客户端应用程序（比如 GraphQL Playground）来验证你的实现。完成数据库设置的话，也可以在那里验证存储数据。此外，如果你感觉还有余力的话，还可以实现一个使用这个服务提供的 GraphQL API 的客户端应用程序（使用 React 或者其他什么）。那让我们开始吧！

## 使用 Express 设置 Apollo 服务端

有两种方法来开始这个应用程序。你可以按照我在这个[手把手教你启动最小 Node.js 的指南](https://www.robinwieruch.de/minimal-node-js-babel-setup)里的指导，或者在这个 [GitHub 代码库](https://github.com/rwieruch/node-babel-server)里找到一个启动项目，并根据它的安装说明进行操作。

Apollo 服务端可以与 Express、Koa、Hapi 等常用的 Node.js 库一起使用。因为它与库无关，所以可以将它与客户端和服务器应用程序的许多不同第三方库结合使用。在这个应用程序中，你将使用 [Express](https://expressjs.com/)，因为它是 Node.js 最流行和最常见的中间件库。

将这两个依赖包安装到 *package.json* 文件和 *node_modules* 文件夹中

{title="Command Line",lang="json"}
~~~~~~~~
npm install apollo-server apollo-server-express --save
~~~~~~~~

从库名可以看出，你可以使用任何其他中间件解决方案（例如 Koa、Hapi）来填充你自己的 Apollo 服务端。除了用于 Apollo 服务端的这些库之外，你还需要 Express 和 GraphQL:

{title="Command Line",lang="json"}
~~~~~~~~
npm install express graphql --save
~~~~~~~~

现在，每个库都设置为从 *src/index.js* 文件中的源代码开始。为了启动使用 Express 的 Apollo 服务端，首先，你需要导入必要的模块：

{title="src/index.js",lang="javascript"}
~~~~~~~~
import express from 'express';
import { ApolloServer } from 'apollo-server-express';
~~~~~~~~

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

使用 Apollo 服务端的 `applyMiddleware()` 方法，你可以添加任意中间件，这里使用了 Express。此外，还可以指定 GraphQL API 端点的路径。而且，你还可以自定义初始化 Express 应用程序。现在惟一缺少的是用于创建 Apollo 服务端实例的模式（schema）和解析器（resolvers）的定义。我们将先实现它们，之后再来学习它们：

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

**GraphQL 模式** 是 Apollo 服务端提供出的所有，能通过 GraphQL 读写的可用数据。它可以提供给任何消费这个 GraphQL API 的客户端。模式 由**类型定义**组成，首先在顶级必须有用于读取数据的 **查询类型**，然后是**字段**和**嵌套字段**。在这个准备设置给 Apollo 服务端的模式中，定义了一个 `me` 字段，它的类型是**对象类型** `User`。在这里，User 类型只有一个 `username` 字段， 它是**标量**类型的。 GraphQL 规范中有各种标量类型，用于定义字符串（String）、布尔值（Boolean）、整数（Int）等。一般情况下， 模式的叶子节点必须以标量类型结束，以便正确地解析所有内容。可以将其类比为 JavaScript 对象，其中包含对象、数组，此外也包括字符串、布尔值和整数等基本类型。

{title="Code Playground",lang="javascript"}
~~~~~~~~
const data = {
  me: {
    username: 'Robin Wieruch',
  },
};
~~~~~~~~

在这个为了设置 Apollo 服务端而准备的 GraphQL 的模式中，**resolvers** 可以设定返回模式的字段组成的数据。数据源并不重要，数据可以是硬编码的，可以是来自数据库的，也可以是来自其他 （RESTful） API 端点的。你将会在之后学习这些潜在的数据来源。现在只需要知道 resolvers 的数据与数据源无关，这是 GraphQL 和典型的数据库查询语言不同的地方。 Resolvers 是模式中解析 GraphQL 字段的函数。上面的例子中，只有一个 me 字段含有一个 username 为 "Robin Wieruch" 的 User 对象 。

现在，你通过使用 Express 设置的 Apollo 服务端实现的 GraphQL API 应该已经可以运行了。在命令行中，你可以使用 “npm start” 脚本启动应用程序，以便在你进行更改后验证它是否工作。为了在没有客户端应用程序的情况下验证它，Apollo 服务端附带了 GraphQL Playground，这是一个用于消费 GraphQL API 的内置客户端。可以通过在浏览器访问 http://localhost:8000/graphql 使用 GraphQL API 端点。在应用中定义你的一个 GraphQL 的 查询然后查看结果：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  me {
    username
  }
}
~~~~~~~~

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

我可能不会过多地提到 GraphQL Playground，但希望你在进行更改之后用它来验证一下你的 GraphQL API。它是一个有用的工具，可以用来试验和探索你的 API。你还可以选择将 [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) 添加到你的 Express 中间件中。

{title="Command Line",lang="json"}
~~~~~~~~
npm install cors --save
~~~~~~~~

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

服务器的 HTTP 跨域请求需要用 CORS 来处理。否则，你的 GraphQL 服务端可能会遇到跨域错误。

### 练习：

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/b6468a84ad77018bf940d951016b7e2c1e07404f)
* 延伸阅读：[GraphQL](https://graphql.org/learn)
* 尝试完善模式和解析器
  * 为 user 类型添加更多字段
  * 完成 resolver 中的这些字段的解析
  * 在 GraphQL Playground 中查询出你的字段
* 延伸阅读：[独立的 Apollo 服务器](https://www.apollographql.com/docs/apollo-server/v2/getting-started.html)
* 延伸阅读：[在 Express 中设置 Apollo 服务器](https://www.apollographql.com/docs/apollo-server/v2/essentials/server.html)

## Apollo 服务端： 类型定义

本节介绍了 GraphQL 的类型定义以及如何使用它们定义 GraphQL 模式。一个 GraphQL 模式由其类型、类型之间的关系及其结构定义。为此，GraphQL 使用了一种**模式定义语言（SDL）**。不过，模式并不定义数据来自何处。此职责由 SDL 之外的解析器处理。在使用 Apollo 服务端之前，你在模式中使用了一个 User 对象类型，并定义了一个 resolver，该 resolver 为相应的 `me` 字段返回一个 user。

注意 User 对象类型中的 `username` 字段的感叹号。意思是这个 `username` 字段是一个**不能为空**的字段。每当从 GraphQL 模式返回拥有 `username` 的 User 类型时， `username` 字段必须有值，不能是 undefined 或者 null。不过，在 `me` 字段中， User 类型没有感叹号，是否这意味着返回结果中的 `me` 字段可以为空？这是一种特殊的情况。因为服务器必须知道该字段包含什么才能响应，所以 `me` 字段不应该必然有 user 返回。稍后，你将使用 GraphQL 服务端实现身份验证机制(注册、登录、退出)。只有当用户通过服务器进行身份验证时， `me` 字段才会填充 User 对象，比如帐户详细信息。否则，它仍然为空。在定义 GraphQL 类型定义时，必须对类型、关系、结构和(非 null)字段进行特意的设计。

我们通过扩展或添加更多类型定义来扩展模式，并使用 **GraphQL参数** 来处理 user 字段：

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

查询 users 列表将是我们的第三个查询。首先，再次将 query 添加到模式中:

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

现在在GraphQL client（例如 GraphQL Playground）应用程序中，你有三个查询可以使用。它们每个都对相同的 User 类型进行操作，为了满足解析器中的数据需求，每个查询都必须有一个对应的解析器。所有查询都分组在一个惟一的、强制的 Query 类型下，它列出所有公开给客户端的可用 GraphQL 查询，作为读取数据的 GraphQL API。稍后，你将会学到为 GraphQL API 编组写入数据的 Mutation 类型。

### 练习：
* 完成你的[最后一部分源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/469080f810a0049442f02393fae746cebc391cc0)
* 延伸阅读：[Apollo 服务器中的 GraphQL 模式](https://www.apollographql.com/docs/apollo-server/v2/essentials/schema.html)
* 延伸阅读：[GraphQL 思维模式：用图来思考](https://graphql.github.io/learn/thinking-in-graphs/)
* 延伸阅读：[GraphQL 中的可空](https://blog.apollographql.com/using-nullability-in-graphql-2254f84c4ed7)

## Apollo 服务端: 解析器

本节将继续介绍 Apollo Server 中的 GraphQL 模式，但更多的是过渡到解析器方面。在你的 GraphQL 类型定义中，你已经定义了类型、它们的关系和结构。但还没有涉及到关于如何获取数据的。这就是 GraphQL 的解析器发挥作用的地方。

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

GraphQL 解析器可以在每个字段上进行更具体地操作。你可以通过解析一个 `username` 字段来覆盖每个 User 类型的 username。如果没有这么做的话，就会获取 User 实体的默认 `username` 属性。一般来说，这适用于每个字段。返回什么字段既可以是你在解析器函数中具体决定，也可以由 GraphQL 从 JavaScript 实体自动检索属性来尝试获取。

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

因为 `username` 解析器只是在仿写 Apollo 服务端的默认行为，所以现在我们将省略掉它。这样的写法被称为**默认解析器**，因为它们被没有显式地定义。接下来，我们再看看 GraphQL 解析器中的其他参数:

{title="Code Playground",lang="javascript"}
~~~~~~~~
(parent, args, context, info) => { ... }
~~~~~~~~

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

现在所有解析器的上下文应该是相同的。每个解析器所需要访问的上下文，比如在本例中命名为 `me` 的 user，都可以使用解析器函数的第三个参数来访问到。

解析器函数中的第四个参数，信息参数，不常使用，因为它只是提供关于 GraphQL 请求的内部信息。它可以用于调试、错误处理、高级监视和跟踪。你现在不用关心。

关于解析器的返回值的几句话：解析器可以返回数组、对象和标量类型，但是必须在匹配的类型定义中定义它。类型定义必须定义数组或非空字段，以便解析器能正常工作。关于 JavaScript 的 promises 有什么要说的么？通常，你可以在解析器中向数据源(数据库、RESTful API)发出请求，并在解析器中返回 JavaScript 的 promise。GraphQL 可以处理它，并等待 promise 的 resolve。因此不需要担心对数据源的异步请求。

### 练习：
* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/5d8ebc22260455ac6803af20838cbc1f2636be8f)
* 延伸阅读：[Apollo 中的 GraphQL 解析器](https://www.apollographql.com/docs/apollo-server/v2/essentials/data.html)


## Apollo 服务端: 关系类型

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

现在每条消息都拥有自己独有的用户。最后一步对于理解 GraphQL 非常重要。即使你有默认的解析器函数，或者通过定义自己的解析器函数对字段进行细粒度控制，你也可以从数据源检索数据。开发者确保每一个字段都会被解析。GraphQL 允许你将这些字段组合到一个 GraphQL 查询中，而不在意数据源是什么。

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

本节向你展示了如何在 GraphQL 模式中公开关系。如果默认解析器不起作用，则必须在每个字段级别上定义自己的自定义解析器，以便解析来自不同数据源的数据。

### 练习：

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/491d93a90f4ee3413d9226e0a18c10b7407949ef)
* 查询用户列表及其消息
* 查询消息列表及其用户
* 延伸阅读：[GraphQL 模式](https://graphql.github.io/learn/schema/)

## Apollo 服务端: 查询与变更

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

解析器使用从消息对象中解构的 id 查找消息。如果没有消息，解析器将返回 false 。如果有消息，解析器返回 true ，消息对象更新为不包含删除内容的消息。否则，如果没有找到消息，解析器返回 false 。除了写入数据，GraphQL 和 Apollo 服务端中的变更与 GraphQL 查询没有太大区别。

要使消息功能完整只缺少一个 GraphQL 操作。可以读取、创建和删除消息，因此剩下的就是将它们作为练习。

### 练习：

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/a10c54ec1b82043d98fcff2a6395fcd8e405bfda)
* 使用一个变更在 GraphQL Playground 中创建消息
  * 查询所有消息
  * 通过消息查询 `me` 用户
* 使用一个变更在 GraphQL Playground 中删除消息
  * 查询所有消息
  * 通过消息查询 `me` 用户
* 实现一个 `updateMessage` 变更，用于在 GraphQL 中完成消息的所有 CRUD 操作
* 延伸阅读：[GraphQL 查询与变更](https://graphql.github.io/learn/queries/)

## Apollo 服务端中的 GraphQL 模式拼接

模式拼接是 GraphQL 中的一个强大的功能。它可以将多个 GraphQL 模式合并到一个模式中，以便可以在 GraphQL 客户端应用中使用。目前，你的应用程序中只有一个模式，但是可能需要使用多个模式和模式拼接进行更复杂的操作。例如，假设你有一个需要基于域 (例如用户、消息) 模块化的 GraphQL 模式。
最终可能会有两个模式，其中每个模式都匹配一种类型（例如用户类型，消息类型）。 该操作需要合并两个 GraphQL 模式，使整个 GraphQL 模式可以通过 GraphQL 服务器的 API 访问。 这是模式拼接的基本理念之一。

但是你可以更进一步: 你可能最终会使用微服务或第三方平台，这些平台会公开其专用的 GraphQL API 将它们合并到一个 GraphQL 模式中，其中模式拼接成为一个单一的数据来源。同样，客户端可以使用由多个域驱动的微服务组成的整个模式。

在我们的例子中，让我们从 GraphQL 模式和解析器的技术关注点开始分离。之后，你需要按用户和消息的域分隔。

### 技术分离
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

解析器在上下文参数中接收所有示例数据作为模型，而不像之前直接操作示例数据。如上所述，它使解析器函数保持纯净。稍后，你可以更轻松地单独测试解析器函数。接下来，把模式的类型定义移动到 *src/schema/index.js* 文件中：

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

技术分离已经完成，但是需要进行模式组合的域分离还没有完成。到目前为止，你只从 Apollo 服务端实例化文件中拆分了模式，解析器和数据（模型）。现在一切都被技术关注点分开。你也通过上下文传递模型做了一些改进，而不是在解析器文件中导入它们。

### 域分离
在下一步，按域（user 和 message）模块化 GraphQL 模式。首先，将用户相关实体模式分离到名为 *src/schema/user.js*的模式定义文件中：

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

这同样适用于 *src/schema/message.js* 中的消息模式定义：

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

每个文件只描述自己的实体，具有类型及其关系。关系可以是来自不同文件的类型，例如一个 Message 类型与一个 User 类型存在关系，即使 User 类型是在其他位置定义的。请注意 Query 和 Mutation 类型的 `extend` 语句，由于你拥有多个这样类型，因此需要使用 extend 语句去扩展这些类型。接下来，在 *src/schema/index.js* 中为它们定义共享基类型。

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

在此文件中，两个模式都在名为 `linkSchema` 的实用程序的帮助下合并。`linkSchema` 定义了模式中共享的所有类型。它已经为 GraphQL 订阅集定义了一个 Subscription 类型，可以在之后实现。因为还没有正式的方法来完成此操作，作为一种变通方法，在合并通用模式中存在一个带有 Boolean 类型的空下划线字段。通用模式定义共享基类型，在其他特定域模式中使用 extend 进行扩展。

这次，应用程序使用组合模式而不是一个全局模式运行。缺少的是域分离的解析器映射。让我们再次从 *src/resolvers/user.js* 文件中的用户域开始，而这里我为了节省空间省略了实现细节：

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

由于 Apollo 服务端也接受解析器映射列表，你可以在 *src/resolvers/index.js* 文件中导入所有解析器映射，并再次将它们导出为解析器映射列表。

{title="src/resolvers/index.js",lang="javascript"}
~~~~~~~~
import userResolvers from './user';
import messageResolvers from './message';

export default [userResolvers, messageResolvers];
~~~~~~~~

然后，Apollo 服务端可以将解析器列表实例化。再次启动你的应用程序并确认一切正常。

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

现在，你通过 Node.js 获取了 GraphQL 服务端应用的良好起点。最后的实现为你提供了一个通用的 GraphQL 样板项目作为你自己开发项目的基础。我们之后的内容，重点是放在 GraphQL 服务器的数据库连接，身份验证和授权，并使用像分页这样的强大功能。

### 练习：

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/953ef4b2ac8edc7c6338fb73ecdc1446e9cbdc4d)
* 延伸阅读：[使用 Apollo 服务器进行模式组合](https://www.apollographql.com/docs/graphql-tools/schema-stitching.html)
* schema 组合只是 ** schema 委派** 的一部分
  延伸阅读[模式委派](https://www.apollographql.com/docs/graphql-tools/schema-delegation.html)
  * 熟悉 **远程模式** 和 **模式转换** 背后的动机

## PostgreSQL 与 Sequelize 的 GraphQL 服务器

要创建一个全栈 GraphQL 应用程序，你需要引入一个持久化的数据源。样本数据是波动的，而数据库则提供持久数据。在本节中，你将使用 Sequelize（[ORM](https://en.wikipedia.org/wiki/Object-relational_mapping)）为 Apollo 服务端设置 PostgreSQL。[PostgreSQL](https://www.postgresql.org/) 是一个 SQL 数据库，而另一种选择是流行的 NoSQL 数据库 [MongoDB](https://www.mongodb.com/) （用 Mongoose 作为 ORM）。技术选型始终是自由的。你可以选择 MongoDB 或任何其他 SQL/NoSQL 解决方案代替 PostgreSQL，但是为了这个应用程序，让我们坚持使用 PostgreSQL。

这个安装指南将引导你完成基本的 PostgreSQL 设置，包括安装，创建你的第一个数据库，管理数据库用户设置和必要命令。这些是你在阅读完说明后应该完成的事情：

* PostgreSQL 的运行安装
* 具有用户名和密码的数据库超级用户
* 使用 `createdb` 或 `CREATE DATABASE` 创建数据库

你应该能够使用如下命令运行和停止数据库：

* pg_ctl -D /usr/local/var/postgres start
* pg_ctl -D /usr/local/var/postgres stop

使用 `psql` 命令在命令行中连接到数据库，你可以查看到数据库并对其执行 SQL 语句。你应该在 PostgreSQL 设置指南中找到这些操作，但本节还会展示其中的一些。考虑以使用 GraphQL Playground 完成 GraphQL 操作相同的方式执行。`psql` 命令行接口和 GraphQL Playground 是手动测试应用程序的有效工具。

在你本地计算机上安装 PostgreSQL 后，您还需要为你的项目安装 [PostgreSQL for Node.js](https://github.com/brianc/node-postgres) 和 [Sequelize (ORM)](https://github.com/sequelize/sequelize)。我强烈建议你打开 Sequelize 文档，因为当你将 GraphQL 层（解析器）与数据访问层（Sequelize）连接时，它将非常有用。

{title="Command Line",lang="json"}
~~~~~~~~
npm install pg sequelize --save
~~~~~~~~

现在，你可以为 user 和 message 域创建模型。模型通常是应用程序中的数据访问层。然后，使用 Sequelize 设置模型，对 PostgreSQL 数据库进行读写操作。然后通过上下文文传递模型到每个 GraphQL 解析器中来使用它们。这些是必不可少的步骤：

* 为用户域创建模型
* 为消息域创建模型
* 连接应用程序到数据库
  * 提供超级用户的用户名和密码
  * 合并使用数据库的模型
* 应用程序启动后同步数据库

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

两种模型都定义了它们的实体形式。message 模型有一个叫做 text 的字符串类型的数据库列。你可以将多个数据库列水平添加到模型中。一个模型的所有列组成数据库中的表行，每行反映数据库条目，例如 message 或 user。数据库表名称由 Sequelize 模型定义中的参数决定。消息域中有 "message" 表。你可以使用 Sequelize 和 association 定义实体之间的关系。在这个例子中，message 实体属于一个 user，并且该 user 具有许多 message。这是一个包含两个域的最简单的数据库设置，但是因为我们专注于 GraphQL 的服务器端实现 ，你应该考虑阅读更多本应用之外的关于数据库的信息，来完全掌握这些概念。

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

数据库的凭据--数据库名，数据库超级用户名，数据库超级用户密码可以存储在环境变量中。在 *.env* 文件中将这些凭据添加为键值对。 我对本地开发的默认值是：

{title=".env",lang="javascript"}
~~~~~~~~
DATABASE=postgres
DATABASE_USER=postgres
DATABASE_PASSWORD=postgres
~~~~~~~~

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

我们已经完成了 GraphQL 服务器的数据库设置。接下来，你将替换解析器中的业务逻辑，因为这是使用 Sequelize 访问数据库而不是示例数据的地方。应用程序还不完整，因为解析器没有使用新的数据访问层。

### 练习：
* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/a1927fc375a62a9d7d8c514f8bf7f576587cca93)
* 熟悉数据库
  * 尝试使用 `psql` 命令行界面来访问数据库
  * 通过阅读他们的文档来检查 Sequelize API
  * 查看此处提到的任何不熟悉的数据库术语。

## 连接解析器和数据库

既然你已经准备好了在 GraphQL 服务器启动时用来连接的 PostgreSQL 数据库。现在，我们将在 GraphQL 解析器中使用数据访问层（模型）来读写数据库中的数据而不是简单的使用示例数据。在接下来的一节中我们将涵盖以下内容：

* 在 GraphQL 解析器中使用新的模型

* 在应用启动时准备好种子数据

* 增加一个 user 模型方法，通过 username 检索 user

* 学习 `psql` 命令行的一些基本操作

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

`findAll() `和 `findById()` 是 Sequelize 常用的数据库操作方法。但是，查找一个特定 user 的所有 message 相对来说有点太细节了。这里我们使用 `where` 语句，通过 `userId` 来缩小 message 的搜索范围。由于访问数据库的操作将使得应用的架构变得更复杂，因此请务必尽可能多地参考 Sequelize API 文档。

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

除了 `findById()` 和 `findAll()` 方法以外，我们还将用到一些创建和删除 message 等这类变更操作。之前我们不得不为 message 生成一个唯一标识，但是现在当 message 在数据库中创建后，Sequelize 会为其添加唯一标识。

这两个文件还有一个重要的变化：[async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)。Sequelize 是一个基于 promise 的 JavaScript ORM，因此在做数据库操作时始终会返回一个 promise。async/await 的使用能够大大提高JavaScript 异步请求代码的可读性。在上一节中我们知道了在 Apollo 客户端中 GraphQL 解析器的返回结果。由于解析器会一直等待真实结果的返回，所以返回结果也可以是一个 promise。在这种情况下，你也可以删除 async/await 语句，当然你的解析器仍然可以工作。然而，有时候表达明确一点更好，特别是当我们稍后在解析器函数中添加更多业务逻辑的时候，因此我们先保留现在的语句。

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

在 Sequelize 的 `sync()` 方法中 `force` 标记可用于在每次应用启动时插入种子数据。如果要随时保持累积的数据库改动，可以删除该标记或将其设置为 `false`。当应用于生产数据库时请记得删除这个标记。

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

在 user 模型的 `findByLogin()` 方法中，我们通过 `username` 或 `email` 条目来检索 user。虽然现在还没有用户的 `email` 条目，但是当应用具有身份验证机制时，我们会添加上。`username` 和 `email` 都可以作为 `login` 方法的参数来从数据库中检索 user，你也可以了解到如何使用 username 或 email 进行登录应用。

我们已经在数据库模型上引入了第一个自定义方法，接下来我们要考虑的是把这个业务逻辑放在哪里。当为模型提供这些访问方法时，你可能最终会得到一个名为 *fat models* 的概念。另一种方法是为这些数据访问层功能编写单独的服务，如函数或类。

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

重新启动应用，在 GraphQL Playground 中尝试不同的 GraphQL 查询和变更操作，并验证这些都能正常工作。如果出现任何关于数据的错误，请确保它正确连接到你的应用，并且数据库能够在命令行正确运行。

由于现在已经引入了数据库，因此 GraphQL Playground 不再是唯一的手动测试工具。虽然 GraphQL Playground 可用于测试 GraphQL API，但你可能更希望使用 `psql` 命令行界面手动查询数据库。例如，你可能希望检查数据库中 user 关联的 message 记录，或者在通过 GraphQL 创建 message 之后检查其是否存在数据库中。首先，通过命令行连接到你的数据库：

{title="Command Line",lang="json"}
~~~~~~~~
psql mydatabasename
~~~~~~~~

然后，尝试执行下面的 SQL 语句。趁这个机会熟悉更多的 SQL 操作：

{title="psql",lang="sql"}
~~~~~~~~
SELECT * from users;
SELECT text from messages;
~~~~~~~~

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

每次你执行完 GraphQL 变更后，最好使用 `psql` 命令行界面检查下数据库中记录。这是学习 [SQL](https://en.wikipedia.org/wiki/SQL) 的好方法，只是我们通常会使用像 Sequelize 这样的 ORM 来抽象。

在本节中，我们使用了 PostgreSQL 数据库作为 GraphQL 服务器的数据源，使用 Sequelize 来桥接数据库和 GraphQL 解析器。但是，这只是一种可能的解决方案。由于 GraphQL 与数据源无关，因此你可以选择将任何数据源添加到解析器中。它可以是另一个数据库（例如 MongoDB，Neo4j，Redis），多个数据库或（第三方）REST/GraphQL API endpoint。GraphQL 仅在存在进行查询或修改操作时确保所有字段都经过验证，执行和解析，而不管数据源如何。

### 练习：

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/27a1372264760879e86be377e069da738270c4f3)
* 尝试使用 psql 插入种子数据到数据库中
* 现在尝试使用 GraphQL playground 来查询来自数据库中数据
* 在解析器中删除和添加 async/await 语句，看看它们是如何工作的
* 延伸阅读：[GraphQL 执行](https://graphql.github.io/learn/execution/)

## Apollo 服务端：校验和错误处理

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

保持解析器层轻量同时添加业务逻辑到 service 层是一个不错的方法，这样解析器就变得足够简单。但是在我们这个应用中，我们将业务逻辑都保留在解析器中，是为了将所有代码都保存在同一个地方，避免在整个应用中逻辑变得分散。

我们先从校验开始，这将使得我们需要对错误进行处理。GraphQL 和校验并没有直接关系，但是它在客户端应用（例如显示校验信息）和数据库（例如在写到数据库之前对实体进行校验）之间发挥着作用。

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

看起来 Apollo 服务端的解析器一定能将 [JavaScript errors](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) 转换为有效的 GraphQL 输出。现在已经可以在客户端应用中使用这种常见错误格式，而无需任何其他错误处理。

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

因为你使用了相同的错误对象来生成错误实例，所以在 GraphQL Playground 中的错误输出应该保持不变。当然你也可以使用自定义的错误消息 `throw new Error('My error message.');`。

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

这些是在 Apollo 服务端中进行 GraphQL 校验和错误处理的基本要素。校验可以在数据库（模型）层或者业务逻辑（解析器）层，也可以在 directive 层（见练习）。如果出现错误，GraphQL 和 Apollo 服务端 、将对其进行格式化以兼容 GraphQL 客户端。你也可以在 Apollo 服务端中进行全局的错误格式化处理。

### 练习：

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/83b2a288ccd65c574ac3f2083c4ceee3197700e7)
* 在你的数据库模型上添加更多的校验规则
* 延伸阅读： Sequelize 校验规则的文档
* 延伸阅读：[Apollo 服务端的错误处理](https://www.apollographql.com/docs/apollo-server/v2/features/errors.html)
* 了解在 Apollo Server 中不同的自定义错误
* 延伸阅读：[GraphQL 字段级校验以及自定义 directive](https://blog.apollographql.com/graphql-validation-using-directives-4908fd5c1055)
* 延伸阅读：[自定义模式 directive](https://www.apollographql.com/docs/apollo-server/v2/features/directives.html)

## Apollo 服务端: 认证

在 GraphQL 中认证是一个很热门的话题。没有固定的方法去做认证，但大多数人的应用都需要认证。GraphQL 本身并没有固定的认证机制，因为它只是一种查询语言。如果你想要在 GraphQL 中实现认证，考虑用 GraphQL 变更操作。在这一部分中，我们用最简化的方法去给你的 GraphQL 服务器添加认证。之后，应该可以通过服务器注册和登录一个用户到你的应用。之前用过的 `me` 用户将会做为一个认证过的用户。

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

同以前一样，用户模型的两个新字段拥有自己的验证规则。用户密码应该是 7 到 42 位的字符串，并且电子邮件应该具有合法的电子邮件格式。如果在用户创建期间任何一个验证失败了，则会生成一个 JavaScript 错误，并用 GraphQL 转换和传输错误。在客户端应用程序中的注册表单可能会显示验证错误。

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

这就是数据库的数据迁移以便开始 GraphQL 认证

### 用 GraphQL 实现注册

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

`signUP` 变更操作接受三个不为空的参数：用户名，电子邮件和密码。这些参数是用于在数据库中创建用户。用户应该能够使用用户名或者电子邮件组合密码来成功登录。

现在我们将会考虑 `signUp` 变更操作的返回值类型。由于我们准备使用 GraphQL 基于 token 的认证，因此仅仅返回一个字符串 token 就足够了。然而，为了在 GraphQL 模式中区分 token，它具有自己的 GraphQL 类型。你将会在如下学习到更多关于 token 的知识，因为这个应用所有关于认证机制的都跟 token 有关。

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

这就是 GraphQL 框架关于创建一个基于 token 的注册。你已经为注册创建了一个 GraphQL 变更操作和一个解析器，它基于某些验证和传入的解析器参数在数据库中创建用户。它为注册的用户创建一个 token。目前的设置足以创建具有GraphQL 变更操作的新用户。

### 用 Bcrypt 保护密码

 这段代码中有一个主要的安全的漏洞：用户密码是直接以文本形式存储在数据库中，这样会让第三方很容易的获取密码。为了补救这个问题，我们用 [bcrypt](https://github.com/kelektiv/node.bcrypt.js) 库来哈希密码。首先，通过命令行将其安装：

{title="Command Line",lang="json"}

```
npm install bcrypt --save
```

注意：如果你在 Windows 上安装 bcrypt 的过程中出现任何问题，你可以尝试用一个叫 [bcrypt.js](https://github.com/dcodeIO/bcrypt.js) 的代替方案。尽管它有点慢，但是有人报告过它可以在他们的机器上运行。

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

bcrypt 的 `hash()` 方法接受一个字符串--用户密码和一个叫 salt rounds 的整数。每次 salt 循环使得密码哈希成本更高，同时这也使得攻击者解密哈希值的成本更高。现在一般的 salt rounds 的范围是 10 到 12，因为增加 salt rounds 的范围可能会同时在哈希和解密的过程中导致性能问题。

在这个实现中，`generatePasswordHash()` 函数被添加到用户的原型链中。这就是为什么我们可以在每一个用户的实例中执行这个函数，所以你可以在这个函数中通过 `this` 访问到这个用户本身。你也可以把包含密码的用户实例作为参数，我更喜欢这样做，尽管对于任何 Web 开发者来说使用 JavaScript 的原型链继承是一个好的工具。现在，每一个用户在数据库中被创建的时候，他的密码都是通过 bcrypt 哈希运算过的。

### 在 GraphQL 中实现基于 Token 的认证

我们仍然需要实现基于 token 的认证。目前为止，在你的应用中只有一个占位符用于创建被注册和登录变更操作返回的 token。一个已经登录的用户可以被这个 token 标识，并被允许从数据库读写数据。因为用户注册后会自动登录，所以在两个阶段中都会生成 token。

接下来是在 GraphQL 中基于 token 认证的实现细节。与 GraphQL 无关，你将会用 [JSON web token (JWT)](https://jwt.io/) 去识别你的用户。JWT 官方网站给出的定义说：_JSON Web Tokens 是一种开放的，行业标准为 RFC 7519 的，为了在双方之间安全的表示请求的方法。_ 换句话说，JWT 是一种处理两端之间通信的安全方法（例如客户端和服务端）。如果你之前没有在安全相关的应用上工作过，如下部分将会引导你完成整个过程，同时你也会明白 token 只是一种安全的带有用户信息的 JavaScript 对象。

为了在这个应用中创建 JWT，你将会使用流行的 [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) 的 node 包。在命令行中将其安装。

{title="Command Line",lang="json"}

```
npm install jsonwebtoken --save
```

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

“签署” token 的第一个参数可以是除敏感数据之外的任何用户信息（例如密码），因为 token 将会被客户端获取。签署一个 token 意味着将数据放入其中，这个你已经做了，并保护它，这个你尚未完成。为了保护你的 token，传入一个密钥（**任意** 长字符串）**只能被你和你的服务器使用**。任何第三方实体都不应该具有访问权限，因为这个密钥将会被用来加密（签署）和解密你的 token。

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

现在你已经将你的信息保护在了 token 中。如果你想解密它，为了获取保护的数据（`sign` 方法的第一个参数），你将再次需要这个密钥。此外，这个 token 的合法时间只有三十分钟。

这就是注册：你正在创建一个用户并返回一个合法的 token，可以从客户端应用使用该 token 来认证用户。服务器可以解密每一个从请求附带的 token 并允许用户访问敏感的数据。你可以尝试使用 GraphQL Playground 注册，这个应该可以在数据库中创建一个用户并为其返回一个 token。此外，你可以使用 `psql` 检查你的数据库来判断用户是否被创建并且包含一个经过哈希计算过的密码。

### 用 GraphQL 实现登录

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

让我们逐步完成登录的新解析函数。作为参数，解析函数拥有访问 GraphQL 变更（login，password）和上下文（models，secret）的输入参数。当用户尝试登录你的应用，登录，可以是唯一的用户名或者是唯一的电子邮件，将会被用于从数据库中获取用户。如果没有该用户，应用将会抛出一个错误用于客户端提示用户。如果用户存在，用户的密码将会被验证。你将会在下一个例子的用户模型中看到这个方法。如果密码不合法，应用抛出一个错误给客户端应用。如果密码合法，`signIn` 变更操作返回一个同 `signUp` 变更操作一样的 token。客户端应用要么登录成功要么给不合法的凭据显示一个错误信息。你还可以看到在通用 JavaScript Error 类上使用特殊的 Apollo 服务端错误。

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

同样，它是一个典型的 JavaScript 继承让一个方法可以在用户实例上可用。在这个方法中，用户（this）和其密码可以和从 GraphQL 变更操作传入的密码一起使用 bcrypt 进行比较。因为用户的密码是加过密的，传入的密码是普通文本。幸运的是，在用户登录的时候，bcrypt 将会告诉你密码是否是正确的。

现在你已经给你的 GraphQL 服务器应用设置好了注册和登录。你通过使用 Sequelize 钩子函数对即将到达数据库的明文密码使用 bcrypt 进行哈希和比较，同时你使用 JWT 和一个密钥将用户数据加密到一个 token。然后在每次注册和登录的时候返回这个 token。然后客户端应用可以保存 token（例如浏览器本地存储）并做为认证随同每一个 GraphQL 查询操作和变更操作一起发送。

下一节将会向你介绍关于服务器端的 GraphQL 授权，和当成功注册或者登录并得到应用的认证过后，可以用 token 做什么。

### 练习：

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/831ab566f0b5c5530d9270a49936d102f7fdf73c)
* 用 GraphQL Playground 注册一个新用户
* 用 `psql` 在数据库中检查你的用户和其加密的密码
* 延伸阅读: [JSON web tokens (JWT)](https://jwt.io/)
* 用 GraphQL Playground 登录一个用户
  * 复制和粘贴 token 到 JWT 网站上的交互式 token 解密（总结：信息本身并没有被保护，这就是为什么你不能将你的密码放在你的 token 中）

## 使用 GraphQL 和 Apollo 服务端进行授权

在上一节中，你已经设置了 GraphQL 变更操作以启用服务器的身份验证。你可以使用 bcrypt 哈希密码注册新用户，并且用你的用户凭证进行登录。这两个涉及身份认证的 GraphQL 变更都返回了一个使用密钥来保护非敏感用户信息的 token（JWT）

无论是在注册还是登录时获得的 token，在 GraphQL signIn 或 signUp 变更（操作）成功后都会被返回给客户端。客户端必须存储 token，例如[浏览器的会话存储](https://www.robinwieruch.de/local-storage-react)。每次向 GraphQL server 发起请求时，必须将 token 附加到 HTTP 请求头里面。GraphQL server 接收到请求后，可以校验 HTTP 请求头，验证其真实性，并执行类似 GraphQL 操作的请求。如果 token 无效，则 GraphQL server 必须向客户端返回一个错误。如果客户端仍然本地存储着 token，则应删除 token 并重定向到登录页面。

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

在服务端的全局授权中，你将把通过 token 身份校验的用户 `me` 注入向 Apollo 服务端发起的每个请求中。在 token 中的用户 `me` 会被 createToken() 函数编码。它不再是数据库中的一个用户，这可以避免额外的数据库请求。

在 `getMe()` 函数内，你从传入的 HTTP 请求中提取名为 "x-token" 的请求头以便进行授权校验 。在每次请求时，GraphQL client 都会把在注册或登录时获取的 token 及其他有效数据（例如 GraphQL 操作）一起放在 HTTP 请求头中发送。然后可以检查在函数中是否存在这样的 HTTP 请求头。如果没有，该函数继续往下执行，但用户 `me` 是未定义的。如果 token 存在，则该函数会使用其密钥验证 token，并检索创建 token 时存储的用户信息。如果因 token 无效或过期导致验证失败，GraphQL 服务端会抛出特定的 Apollo Server 错误。如果验证成功，则函数将继续使用经过验证的用户 `me`。

当 GraphQL 客户端发送带有无效或过期 token 的 HTTP 请求时，该函数返回错误。否则，该函数会通过请求，因为必须在解析器层校验是否允许用户执行某些操作。一个未经身份验证的用户 -- 此处用户 `me` 是未定义的 -- 也许能够查看消息，但不能创建新消息。现在，我们的 GraphQL 服务端可以防止无效和过期的 token 了。

这是 GraphQL 服务端应用的最高级别身份验证。你可以使用 GraphQL 服务端对具有 `signUp` 和 `signIn` 变更操作的 GraphQL 客户端进行身份验证，并且 GraphQL 服务端仅允许来自 GraphQL 客户端的有效的，未过期的 token 通过验证。

### 解析器层的 GraphQL 授权

一个 GraphQL HTTP 请求，即使它的请求头里没有 token，也会经过 getMe() 函数。这是一个很好的默认行为，因为你现在想要注册新用户并在没有 token 的情况下登录应用程序。你可能希望在未经授权验证的情况下查询消息或用户。在没有授权 token 的情况下允许用户的某些请求是可接受的，有时甚至是必要的，这样可以对不同用户类型授予不同级别的访问权限，仅当 token 失效或过期时 getMe() 函数才会返回错误。

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

你可以想象，如果它被用于授权所有经过身份验证的用户才可访问的 GraphQL 操作，将会变得多么重复且容易出错，因为它将大量授权逻辑混合到解析函数中。为了解决这个问题，我们引入了一个授权抽象层来保护 GraphQL 操作，使用名为 **combined resolvers** 或 **resolver middleware** 的解决方案。我们来安装这个包：

{title="Command Line",lang="json"}
~~~~~~~~
npm install graphql-resolvers --save
~~~~~~~~

让我们在新建的 *src/resolvers/authorization.js* 文件中，使用这个包来实现一个保护解析函数。它只用来检验是否是当前用户 `me`。

{title="src/resolvers/authorization.js",lang="javascript"}
~~~~~~~~
import { ForbiddenError } from 'apollo-server';
import { skip } from 'graphql-resolvers';

export const isAuthenticated = (parent, args, { me }) =>
  me ? skip : new ForbiddenError('Not authenticated as user.');
~~~~~~~~

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

现在，`isAuthenticated()` 解析函数始终在创建消息的解析器之前运行。这些解析器相互链接，你可以在任何需要的地方重用这个保护性质的解析器函数。而所需要做的仅仅是在 *src/resolvers/authorization.js* 文件中，在真正执行任务的解析器前做了一点小小的改动。

### 基于权限的 GraphQL 授权

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

作为替代策略，你还可以直接在 `isMessageOwner` 解析器中使用 `isAuthenticated` 解析器；你可以避免在当前的解析器中处理它。我发现在授权解析器中，显式调用比隐式调用更实际。不过, 另一种路线仍在基于角色的授权部分进行了解释。

第二个组合解析器用于权限检查，因为它决定用户是否有删除消息的权限。不过，这只是一种方法。在其他情况下，消息可以携带一个布尔值的标志，该标志决定当前用户是否具有某些权限。

### 基于角色的 GraphQL 授权

我们从高级授权转变为更具体的基于权限的解析程序保护的授权。现在我们将介绍另一种称为 **角色** 的方法来进行授权。下个代码块是基于角色授权的 GraphQL 变更操作，因为它具有删除用户的能力。这允许你创建具有管理角色的用户。

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

以上就是使用 GraphQL Apollo 服务端基于角色授权的基础知识。在示例中，角色只是需要检查的字符串。在更复杂的基于角色的体系结构中，角色可能会从字符串更改为包含许多角色的数组。这样就不用进行同等检查，而是需要检查数组里面是否包含目标角色。把角色设置为数组，便于处理复杂的基于角色的授权配置。

### 在 GraphQL Playground 中设置 Headers

你已经为 GraphQL 应用设置好了授权，现在您只需要验证它能否工作。测试 GraphQL 应用最简单的方法是使用 GraphQL Playground 来运行不同的场景。这里我们将删除用户作示范，但你应该亲自练习运行所有剩余的实际场景。

在用户执行删除操作之前，必须先登录，因此我们先用非管理员用户在 GraphQL Playground 中执行 `signIn` 变更操作，稍后我们再用管理员用户进行操作一遍，以区分它们的不同。

{title="GraphQL Playground",lang="json"}
~~~~~~~~
mutation {
  signIn(login: "ddavids", password: "ddavids") {
    token
  }
}
~~~~~~~~

登录 GraphQL Playground 后，你会收到一个 token，在接下来的 GraphQL 操作中都需要把这个 token 附到 HTTP 请求头里。GraphQL Playground 有一个用于添加 HTTP 请求头的面板。由于你的应用程序将检查 x-token 是否存在，请将 token 设置为：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "x-token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MiwiZW1haWwiOiJoZWxsb0BkYXZpZC5jb20iLCJ1c2VybmFtZSI6ImRkYXZpZHMiLCJpYXQiOjE1MzQ5MjM4NDcsImV4cCI6MTUzNDkyNTY0N30.ViGU6UUY-XWpWDJGfXqES2J1lEr-Uye8XDQ79lAvByE"
}
~~~~~~~~

你的 token 应该与上面的 token 不同，但格式类似。设置好 token，我们现在可以在 GraphQL Playground 上通过下面的 GraphQL 变更操作删除某个用户了。带有 token 的 HTTP 请求头将与 GraphQL 操作一起发送：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
mutation {
  deleteUser(id: "2")
}
~~~~~~~~

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

如果你以管理员角色登录，再按照之前的步骤操作，则可以成功删除一个用户实体。

![](images/authorization_1024.jpg)

我们已为此应用程序添加了基本授权。在每个请求到达 GraphQL 解析器之前，它具有全局授权；并在解析器层进行权限保护。它们会检查用户是否已通过身份验证，用户是否能够删除消息（基于权限的授权），以及一个用户是否能够删除另外一个用户（基于角色的授权）。

如果你想要比解析器层授权更精确，请在 GraphQL 中查看 **基于指令的授权** 或 **字段级别的授权**。你也可以使用像 Sequelize 这样的 ORM，在数据访问级别进行权限验证。你可以基于应用程序的需求决定使用哪种级别的授权策略。

### 练习：
* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/4f6e8e6e7b899faca13e1c8354fe59637e7e23a6)
* 延伸阅读：[GraphQL 授权](https://graphql.github.io/learn/authorization/)的更多信息
* 通过 GraphQL Playground 测试不同的授权场景
* 了解 GraphQL Apollo 服务端中字段级别授权的更多信息
* 了解 GraphQL Apollo 服务端中数据访问级别授权的更多信息

## Apollo 服务端中的 GraphQL 自定义标量

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

你可能已经注意到了这里的奇怪之处：从 GraphQL Playground 返回的数据中，有一个 unix 时间戳（例如：1540978531448）；而在数据库中，消息（以及其他实体）的时间有另外一个格式（例如：2018-10-31 17:35:31.448+08）。你可以自己用 psql 查看。这是 GraphQL 的内部机制，使用它自己内部的格式化日期的规则。你可以通过增加自定义标量来改变这种行为。首先，我们需要安装一个主流的 GraphQL 自定义时间标量的 node 依赖包。

{title="Command Line",lang="json"}
~~~~~~~~
npm install graphql-iso-date --save
~~~~~~~~

然后，在 *src/schema/index.js* 文件中的模式中引入一个 `Date` 标量：

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

最后，在 *src/schema/message.js* 文件中，把消息模式中的 createdAt 字段的标量类型从 String 变为 Date

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

现在它已经是一个可读的格式了。你可以通过查询[文档](https://github.com/excitement-engineer/graphql-iso-date)研究这个库能把日期转换成哪些格式。

### 练习：
* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-express-postgresql-boilerplate/tree/709a406a8a94e15779d2e93cfb847d49de5aa6ca)
* 延伸阅读：[GraphQL 中的自定义标量](https://www.apollographql.com/docs/apollo-server/features/scalars-enums.html)

## Apollo 服务端中 GrapgQL 的分页

使用 GraphQL 必然会遇到列表应用的 **分页** 功能。聊天应用中保存的用户消息会变得越来越长，当客户端请求消息记录用于展示时，一次性从数据库中获取所有的消息会导致服务器性能瓶颈。分页允许你把一个列表分成多个列表，即分页。一页通常会有大小限制和偏移量。通过分页，你可以只请求一页的展示数据，如果用户想查看更多的数据，就再请求一页。

接下来，你将会通过两种方式在 GraphQL 中实现分页。第一种方法是最原始的方法，叫做**偏移/限制分页**。更复杂的方式是**游标分页**，这是在应用中允许分页的众多经典实践的中一种。

### Apollo Server 中 GrapgQL 的偏移/限制分页

偏移/限制分页并不是很难实现。限制规定了你每次想要从整个列表中获取多少条数据，偏移量指定了应该从整个列表的什么位置开始。使用不同的偏移量，你可以筛选整个列表并从中获取一个固定条数的子列表（页）。

我们在文件 *src/schema/message.js* 中修改消息的模式，增加两个新的参数：

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

幸运的是，你的 ORM (Sequelize) 为内部偏移和限制功能提供了支持。你可以在 GraphQL playground 上尝试修改限制和偏移量，体验分页功能。

{title="GraphQL Playground",lang="json"}
~~~~~~~~
query {
  messages(offset: 1, limit: 2){
    text
  }
}
~~~~~~~~

虽然这个方法很简单，但是它也有缺点。当偏移量变得非常长时，数据库查询时间会变长，客户端在等待下一页数据时的性能会比较差。同时，偏移/限制分页不能处理在查询之前删除一个数据的情况。例如，你查询了第一页，同时有人删除了第一页中的一条数据，下一页的偏移量就会出错，因为数据已经减少一个了。在偏移/限制分页中，这个问题并不容易解决，所以游标分页就有必要了。

> ### Cursor-based Pagination with Apollo Server and GraphQL

### Apollo 服务端中 GraphQL 的游标分页

与偏移/限制分页中用数据量的个数标记偏移量不同，在游标分页中，我们用**游标**标记偏移量。游标可用于表达：”给我从游标 Y 开始的 X 个数据”。一个常用的方法是用时间（例如：一个实体在数据库中的创建时间）来标识一个列表中的某条数据。在我们的例子中，每条消息已经有一个 `createdAt` 时间，即这个实体被写入数据库的时间，而且我们已经在消息实体的模式中暴露了这个字段。这个消息的创建时间就是游标。

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

因为你修改了消息的模式，你也需要相应地修改 *src/resolvers/message.js* 文件：

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

首先，这个列表必须是通过 `createdAt` 排序的，否则游标是无用的。无论如何，当列表是有序时，都可以确保不带游标请求消息列表的第一页也可以返回最近的消息。当你将前一页的最后一个创建时间当做游标用于请求下一页时，你可以获得通过创建时间排序的下一页消息。这就是你怎么一页一页访问整个消息列表的。

其次，游标的三元操作符确保了第一页请求不需要游标。之前提到过，第一页只获取列表中最近的消息，所以你可以利用最后一条消息的创建时间当做游标来请求下一页消息。

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

这就是使用创建时间作为一条数据的稳定标识符的游标分页的一个基础的应用。创建时间是一个通用的方法，然而这里还有另外的值得探索的方法。

### 基于游标的分页:页面信息、连接和哈希

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

同样，这将返回列表中剩余的最后一条消息。你不再需要查询每条消息的创建时间，只需要查询最后一条消息的游标。客户端应用程序不需要最后一条消息的游标的详细信息，因为它现在只需要 `endCursor`。

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

你只比定义的 limit 多检索一条消息。如果消息列表数量大于 limit，则有下一页;否则，就没有下一页。要么返回限定的 limit 数量的消息，要么没有下一页时，返回所有消息。现在，你可以在 `pageInfo` 类型中添加 `hasNextPage` 字段。如果查询消息的条件为 limit = 2，且没有游标，返回结果中，`hasNextPage` 为 true。如果查询消息的条件为 limit > 2 且没有游标，返回结果中，`hasNextPage` 为 false。据此，GraphQL 客户端应用程序知道列表已经查询到最后了。

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

GraphQL 客户端接收哈希化的 `endCursor` 字段。这个哈希值可以用作查询下一页的游标。在解析器中，用于数据库查询时，需要把传入的游标反哈希化成真实时间。

哈希化游标是基于游标的分页的一种常见方法，因为它对客户端隐藏了详细信息。（GraphQL）客户端应用程序只需要使用哈希值作为查询下一页的游标。

### 练习：

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/810907cde43b460231b9ed3a2172e62528f81ba4)
* 延伸阅读：[GraphQL 分页](https://graphql.github.io/learn/pagination/)

## GraphQL 订阅


到目前为止，你使用 GraphQL 的查询和变更来读写数据。这是使 GraphQL 服务端为增删改查操作做好准备的两个基本 GraphQL 操作。接下来，你将了解 GraphQL 订阅，以便在 GraphQL 客户端和服务端之间进行实时通信。

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

在第一部分中，你将为应用程序设置订阅结构；然后，你将为创建的消息订阅添加实现细节。第一步只需要做一次，但是当向应用程序添加更多的 GraphQL 订阅时，这一步将需要再次设置。

### Apollo 服务端订阅配置

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

要完成订阅设置，你需要使用一个可用的 [PubSub 引擎](https://www.apollographql.com/docs/apollo-server/v2/features/subscriptions.html#PubSub-Implementations)来发布和订阅事件。默认情况下，Apollo 服务器自带 PubSub 引擎，但是你可能会发现它缺少一些其他的配置。新建一个 *src/subscription/index.js* 文件，配置以下内容:

{title="src/subscription/index.js",lang="javascript"}
~~~~~~~~
import { PubSub } from 'apollo-server';

export default new PubSub();
~~~~~~~~

这个 PubSub 实例是你的 API，它支持应用程序中的订阅。现在我们完成了订阅的总体设置。

### 使用 PubSub 实现订阅及发布

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

在 *src/subscription/message.js* 中添加你的第一个事件常量：

{title="src/subscription/message.js",lang="javascript"}
~~~~~~~~
export const CREATED = 'CREATED';
~~~~~~~~

通过按领域划分并导出相关事件常量，便于你在其他地方导入和使用。这样的目录结构让你能在领域层分离不同种类的事件。

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

subscribe 函数签名和其他解析函数相同。上下文提供的模型对象可以在这里访问，但是在本例中用不到。

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

执行订阅后，页面会提示正在监听。在第二个标签中，登录一个账户：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
mutation {
  signIn(login: "rwieruch", password: "rwieruch") {
    token
  }
}
~~~~~~~~

从结果中复制 token 并粘贴到当前标签的 HTTP headers 面板中：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "x-token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwiZW1haWwiOiJoZWxsb0Byb2Jpbi5jb20iLCJ1c2VybmFtZSI6InJ3aWVydWNoIiwicm9sZSI6IkFETUlOIiwiaWF0IjoxNTM0OTQ3NTYyLCJleHAiOjE1MzQ5NDkzNjJ9.mg4M6SfYPJkGf_Z2Zr7ztGNbDRDLksRWdhhDvTbmWbQ"
}
~~~~~~~~

然后在第二个标签中发送一条信息：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
mutation {
  createMessage(text: "Does my subscription work?") {
    text
  }
}
~~~~~~~~

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

你已经实现了 GraphQL 订阅。虽然要完全理清头绪还需要更多的挑战，不过一旦你完成了一些基本操作，你就可以用这些知识去作为 GraphQL 实时应用的基础了。

### 练习:

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/eeb50f34a2569fa85141bf8ec3f8e9baaf670170)
* 延伸阅读：[Apollo 服务端订阅](https://www.apollographql.com/docs/apollo-server/v2/features/subscriptions.html)
* 观看相关演讲 [GraphQL 订阅](http://youtu.be/bn8qsi8jVew)

## 测试 GraphQL 服务器

在编程开发中测试经常被忽视，因此本节将重点介绍 GraphQL 服务器的端到端（E2E）测试。虽然单元和集成测试是测试金字塔的基本支柱，覆盖了应用程序的所有独立功能，但 E2E 测试则覆盖了整个应用程序的用户场景。在本例中，E2E 测试将确定用户是否能够注册你的应用程序，或管理员用户是否可以删除其他用户。你不需要编写尽可能多的 E2E 测试，因为它们覆盖的是更大，更复杂的用户场景，而不仅仅是基本功能。 此外，E2E 测试覆盖了应用程序的所有技术角落，例如 GraphQL API ，业务逻辑和数据库。

### GraphQL 服务器 E2E 测试配置

这里使用 Mocha 和 Chai 用来测试我们的程序。Mocha 是一个测试运行器，它允许你从 npm 脚本执行测试，同时提供有组织的测试结构；Chai 为你提供很多断言方法，例如： 在真实场景中通过“期望 X 等于 Y” 这样的测试。

{title="Command Line",lang="json"}
~~~~~~~~
npm install mocha chai --save-dev
~~~~~~~~

要使用这些程序，必须先安装 [axios](https://github.com/axios/axios)，以便向 GraphQL API 发出请求。在测试用户注册时，你可以将 GraphQL 变更操作发送到 GraphQL API，然后在数据库中创建用户并返回其信息。

{title="Command Line",lang="json"}
~~~~~~~~
npm install axios --save-dev
~~~~~~~~

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

在命令行中键入 `npm test` 来执行测试。虽然它不测试应用程序的任何逻辑，但将验证 Mocha、Chai 和你的 npm 脚本是否正常工作。

在为 GraphQL 服务器编写 E2E 测试之前，必须先解决数据库问题。由于测试需要在真实的 GraphQL 服务器上运行，所以需要使用测试数据库而不是生产数据库。 在 *package.json* 中添加 npm 脚本以使用测试数据库启动 GraphQL 服务器：

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

你还需要确保刚才 npm 脚本中的 *mytestdatabase* 数据库已经创建，在命令行中执行 `psql` 然后执行 `createdb` 或 `CREATE DATABASE` 来创建数据库.

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

现在你可以使用 async/await 来测试异步逻辑。

### 使用 E2E 来测试用户场景

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

每个测试都应该像这个一样简单直接。使用 axios 发出 GraphQL API 请求，期望来自 API 的查询/变更结果。 在幕后，数据从数据库读取或写入，身份验证、授权和分页等业务逻辑在期间执行。 请求会走完整个从 API 到数据库的 GraphQL 服务器流程。 E2E 测试不会测试隔离单元（单元测试）或较小的单元组合（集成测试），而是整个流程。

`userApi` 函数是确保此测试有效的最后一部分。它没有在这个测试中实现，而是在 *src/tests/api.js* 文件中测试的，在此文件中，你将找到用于请求 GraphQL 测试服务器的所有函数的测试。

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

要立即执行测试，请在命令行中执行 `npm run test-server` 来运行 GraphQL 测试服务器，并在另一个命令行选项卡中键入 `npm test` 来执行测试。 输出应如下所示：

{title="Command Line",lang="javascript"}
~~~~~~~~
users
  user(id: ID!): User
    ✓ returns a user when user can be found (69ms)

1 passing (123ms)
~~~~~~~~

如果输出错误，控制台日志可能会帮助你找出问题所在。另一种选择是从 axios 请求中复制查询代码并将其放入 GraphQL Playground 执行。Playground 中的错误报告可以使查找问题变得更容易。

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

测试常见路径是有价值的，不太常见的边界场景也值得被测试。在这种边界场景下，查询不存在的用户不会返回错误，而是返回 null。

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

首先，使用 `signIn` 登录用户，登录成功后返回 token。然后，此 token 可用于其他每个 GraphQL 操作。在这种场景下，它用于 `deleteUser`。 但是，这个操作仍然失败了，因为当前用户不是管理员。你可以自行使用管理员账户并通过同样的 API 来再次尝试这个的场景。

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

这些 E2E 测试涵盖了用户领域的场景，从 GraphQL API 业务逻辑到数据库访问。但是，仍有可测试的空间。考虑测试其他用户特定领域的场景，例如用户注册，在登录时提供错误的密码，或者在消息领域中请求一页和另一页的带分页的消息。

本节仅涉及 E2E 测试。使用 Chai 和 Mocha，你还可以为不同的应用层（例如：解析器层）添加更小的单元和集成测试。如果你需要一个库来实现测试替身、测试桩或模拟某些东西，我建议使用 [Sinon](https://sinonjs.org) 作为补充测试库。

### 练习:

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/d11e0487085e014170146ec7479d0154c4a6fce4)
* 为消息领域实现与用户领域类似的测试
* 为这两个领域编写更精细的单元/集成测试
* 延伸阅读：[GraphQL 和 HTTP](https://graphql.github.io/learn/serving-over-http/)
* 延伸阅读：[Apollo 服务端的 mock](https://www.apollographql.com/docs/apollo-server/v2/features/mocking.html)

## 在 GraphQL 中使用批处理和缓存

这一部分内容介绍了如何优化数据库请求。每当有请求 (比如一个 GraphQL 查询) 调用 GraphQL API，解析器可能需要执行多次数据库读和写操作。我们在 GraphQL Playground 中使用以下的查询来看看有什么问题：

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

在启动 GraphQL 服务的命令行里，数据库接收到了四个请求：

{title="Command Line",lang="javascript"}
~~~~~~~~
Executing (default): SELECT "id", "text", "createdAt", "updatedAt", "userId" FROM "messages" AS "message" ORDER BY "message"."createdAt" DESC LIMIT 101;

Executing (default): SELECT "id", "username", "email", "password", "role", "createdAt", "updatedAt" FROM "users" AS "user" WHERE "user"."id" = 2;

Executing (default): SELECT "id", "username", "email", "password", "role", "createdAt", "updatedAt" FROM "users" AS "user" WHERE "user"."id" = 2;

Executing (default): SELECT "id", "username", "email", "password", "role", "createdAt", "updatedAt" FROM "users" AS "user" WHERE "user"."id" = 1;
~~~~~~~~

有一个请求来获取 message 列表，三个请求分别获取每个用户信息。这是 GraphQL 的特性。尽管可以嵌套地使用 GraphQL 关系和 query 结构, 数据库请求还是会存在. 在 *src/resolvers/message.js* 文件中查看解析器可以找到这些请求。在使用深层内嵌查询或者更改的时候，你可能会遇到性能瓶颈，那是因为有特别多的数据需要从数据库读取。

下面这个示例演示了如何用批处理来优化数据库操作。这是个用于优化 GraphQL 服务器和数据库的策略，同时也适用于其他编程环境。在命令行里可以对比 GraqhQL Playground 和数据库的查询结果。

使用批处理会有两点可以得到改善。首先，有一个 message 的 user 数据从数据库中获取了两次，这就是多余的操作。即使是有多条 message，但是有一些可能来自于相同作者。设想一个更大数据的聊天应用场景，假如现在有在两个用户之间有 100 条消息，那么就需要 1 个请求来查询 100 条消息和 100 个请求来为每条消息查询作者，一共就需要 101 个数据库查询。假如重复的作者只查询一次，那就只需要 1 条查询来查找消息和 2 条查询来查找作者，这样一来总查询就只有 3 条。由于每个作者的 id 是可知的，这些 id 就可以批处理到没有重复数据的集合中存放。在当前的这个示例中，作者的 id 就从 [2, 2, 1] 简化为 [2, 1]。

另外，即使我们现在消除了重复的读取，每一个作者都是单独从数据库中读取的。因为提交 GraphQL 的时候携带了所有信息，包括所有作者的 id，那么只用一个数据库请求读取所有的作者信息是可行的。这样就可以把 3 个数据库请求缩减到 2 个，其中一个请求用来拉取 100 条消息，另一个请求用来拉取所有的作者信息。

这两个准则可以应用在示例的 4 个数据库请求上，这样就可以缩减到 2 个数据库请求。在小规模请求上，影响可能比较小，但是在有 100 条消息和 2 个作者的情况下，性能提升就很明显了。这就是 Facebook 的开源工具 [dataloader](https://github.com/facebook/dataloader) 成为重要工具的原因。可以使用 npm 借助如下的命令安装：

{title="Command Line",lang="json"}
~~~~~~~~
npm install dataloader --save
~~~~~~~~

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

loaders 实际上是模型的上层抽象，可以作为上下文传递给解析器，下列示例中直接用 user loader 代替了模型。

现在我们将该函数视为 DataLoader 实例化的参数。该函数使你可以访问其参数中的 keys 数组。这些键就是已清除重复的标识符集合，可用于从数据库中拉取数据。这就是将标识符（id）和模型（数据访问层）传递给 `batchUser()` 函数的原因。然后，该函数使用 keys 在数据库中的模型拉取实体。在函数结束时，keys 数组会按顺序映射成获得的数据实体。否则，如果在从数据库中拉取数据实体之后立即返回，就会导致它们的顺序与传入 keys 列表不同。因此，数据实体需要以传入的 keys 列表的相同顺序返回。

以上就是 loader 的设置方法，一种在模型上层有效的抽象。由于把 loader 作为上下文传递给了解析器，现在可以在 *src/resolvers/message.js* 中这样使用它：

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

当 `load()` 函数每次调用，获取 id 时，它会在同时把这些 id 批处理到一个 set 里面，然后一次性请求所有数据。你可以在 GraphQL Playground 里使用同样的查询语句来进行尝试。查询结果应该是相同的，但是你应该在 GraphQL 服务器的命令行输出里看到 2 个而不是 4 个数据库请求：

{title="Command Line",lang="javascript"}
~~~~~~~~
Executing (default): SELECT "id", "text", "createdAt", "updatedAt", "userId" FROM "messages" AS "message" ORDER BY "message"."createdAt" DESC LIMIT 101;

Executing (default): SELECT "id", "username", "email", "password", "role", "createdAt", "updatedAt" FROM "users" AS "user" WHERE "user"."id" IN (2, 1);
~~~~~~~~

这就是使用批处理带来的好处： 使用 dataloader 这个三方库，一次性地请求所有需要的数据而不是单独请求重复的数据。

下面介绍缓存原理。我们刚刚安装的 dataloader 包同时也提供了缓存请求的选项，虽然它目前还没有生效。尝试执行两次相同的 GraphQL 查询，你可以在命令行里看到两次数据库请求。

{title="Command Line",lang="javascript"}
~~~~~~~~
Executing (default): SELECT "id", "text", "createdAt", "updatedAt", "userId" FROM "messages" AS "message" ORDER BY "message"."createdAt" DESC LIMIT 101;
Executing (default): SELECT "id", "username", "email", "password", "role", "createdAt", "updatedAt" FROM "users" AS "user" WHERE "user"."id" IN (2, 1);

Executing (default): SELECT "id", "text", "createdAt", "updatedAt", "userId" FROM "messages" AS "message" ORDER BY "message"."createdAt" DESC LIMIT 101;
Executing (default): SELECT "id", "username", "email", "password", "role", "createdAt", "updatedAt" FROM "users" AS "user" WHERE "user"."id" IN (2, 1);
~~~~~~~~

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

再次使用相同的 GraphQL 查询两遍，这次你可以观察到，对于使用 loader 的地方，只会有一次数据库请求，因为它被缓存起来了。

{title="Command Line",lang="javascript"}
~~~~~~~~
Executing (default): SELECT "id", "text", "createdAt", "updatedAt", "userId" FROM "messages" AS "message" ORDER BY "message"."createdAt" DESC LIMIT 101;
Executing (default): SELECT "id", "username", "email", "password", "role", "createdAt", "updatedAt" FROM "users" AS "user" WHERE "user"."id" IN (2, 1);

Executing (default): SELECT "id", "text", "createdAt", "updatedAt", "userId" FROM "messages" AS "message" ORDER BY "message"."createdAt" DESC LIMIT 101;
~~~~~~~~

在这个示例中，user 并没有分两次从数据库读取，只有 message 是的，因为它没有使用 dataloader。这就是如何在 GraphQL 中使用缓存的方法了。选择缓存策略不是一件简单的事情，比如说，如果两次查询中间 user 得到了更新，那么客户端应用依然会得到被缓存的 user。

选择合适的时机清除缓存比较困难，所以我建议在每个 GraphQL 请求的时候重新初始化 dataloader。你将会失去在多个请求之间缓存数据的能力，但是还是可以在一个 GraphQL 请求里缓存所有的数据库请求。在 dataloader 包里这样描述： *“Dataloader 缓存不是为了替代 Redis、Memcache 或者是别的任何应用层缓存组件。Dataloader 首先是一种数据加载机制，它的缓存目的在于同一个请求上下文中，避免重复请求相同的数据。”*  假如你需要真正的数据库级别缓存，可以试试 [Redis](https://redis.io/) 。

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
创建新的 *src/loaders/index.js* 文件导出所有函数：

{title="src/loaders/index.js",lang="javascript"}
~~~~~~~~
import * as user from './user';

export default { user };
~~~~~~~~

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

请随意在 message 域添加你自己的 loader。这个实践提供有用的 model 上层抽象，用来支持批处理和基于请求的缓存。

### 练习:

* 查看[本节源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/9ff0542f620a0d9939c1adcbd21951f8fc1693f4)
* 延伸阅读：[GraphQL 和 Dataloader](https://www.apollographql.com/docs/graphql-tools/connectors.html#dataloader)
* 延伸阅读：[GraphQL 最佳实践](https://graphql.github.io/learn/best-practices/)

## GraphQL 服务器 + PostgreSQL 部署到 Heroku

最后你想要把 GraphQL 部署到线上环境，用以满足生产环境需求。在这一章节你将学到如何把 GraphQL 服务器部署到 Heroku，一个用来托管应用的平台即服务应用。Heroku 同时也支持 PostgreSQL。

这一章节提供基于命令行工具的快速上手教程。这里可以查看视频教程 [Heroku 部署 GraphQL 服务器教程](https://www.apollographql.com/docs/apollo-server/deployment/heroku.html)。视频里没有包含 PostgreSQL 数据库有关内容。

使用 Heroku 之前请确保满足三个条件：

* [安装 git 命令行工具并把项目上传到 GitHub](https://www.robinwieruch.de/git-essential-commands/)
* 在 [Heroku](https://www.heroku.com/) 上创建帐号
* 安装 [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) 从而在命令行里使用 Heroku 功能

在命令行里使用 `heroku version` 来检测 Heroku 是否安装成功。如果成功安装，使用 `heroku login` 来登入你的帐号。这就完成了基本的 Heroku 设置。在你的项目文件夹下，创建新的 Heroku 应用并命名：

{title="Command Line",lang="javascript"}
~~~~~~~~
heroku create graphql-server-node-js
~~~~~~~~

然后你也可以为你的项目在 Heroku 上安装 PostgreSQL 插件：

{title="Command Line",lang="javascript"}
~~~~~~~~
heroku addons:create heroku-postgresql:hobby-dev
~~~~~~~~

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

查看 [Heroku PostgreSQL 文档](https://devcenter.heroku.com/articles/heroku-postgresql)以获得关于数据库安装的更多信息。

你已经准备好将自己的应用部署上线了。使用 PostgreSQL 插件，你应该也有个数据库 URL。可以通过 `heroku config` 找到它。现在需要检查 GraphQL 服务器的代码，为生产环境做一些改动。在 *src/models/index.js* 文件里，你需要在 development（coding, testing）和 production（live）构建中选择一个。由于你有一个新的环境变量来保存数据库 URL，可以这样来修改环境：

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

查看 *.env* 文件，你可以看到 `DATABASE_URL` 环境变量是不存在的。但是使用 `heroku config:get DATABASE_URL` 可以看到它被设置为 Heroku 环境变量了。一旦你的项目在 Heroku 上运行，你的环境变量是和 Heroku 的环境变量整合到一起的。这就是 `DATABASE_URL` 在你的本地开发环境不可见的原因。

另一个在 *src/index.js* 文件中使用的环境变量是用于授权策略的 *SECRET*。如果你还没有在版本控制中添加 *.env* 文件（查看 .gitignore），你就需要在 Heroku 的项目中使用 `heroku config:set SECRET=wr3r23fwfwefwekwself.2456342.dawqdq` 来设置 `SECRET`。这里的密钥是胡编的，你可以用自己的密钥代替。

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

记得之后删除 flag，否则每次部署数据库都会被清空然后用种子数据填充。根据应用处在 development 或者 production 环境，你都要选择一个数据库，使用数据填充（也可以用空数据库），并且为 GraphQL 选择一个端口。在推送到 Heroku 之前，把所有的改动 push 到 github 仓库。然后，由于之前已经创建了 Heroku 应用，使用 `git push heroku master` 把所有改动也推送到 Heroku 远程仓库。使用 `heroku open` 打开 Heroku 应用，添加 `/graphql` 后缀到浏览器的 URL 中来打开 GraphQL Playground。如果打开失败，请阅读下面的故障排除部分。

根据你选择的数据填充策略，你的数据库将为空或包含种子数据。如果为空，则使用 GraphQL 变更来注册用户并且创建消息。如果有种子数据，则使用 GraphQL 查询请求消息列表。

恭喜，你的应用已经正常上线了。现在 GraphQL 服务器和 PostgreSQL 都已经在 Heroku 上成功运行。按照下面的练习了解有关 Heroku 的更多信息。

### Heroku 故障筛查

生产环境下可能会发生 GraphQL 模式在 GraphQL Playground 中不可用的情况，这是因为禁用了 Apollo 服务器的 `introspection` 标志。你可以将其设置为 true 来修复。另一个改进是添加 `playground` 标志来为 Heroku 启用 GraphQL Playground：

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

另一个问题可能是 Heroku 没有为生产环境安装 dev 依赖库。虽然在 Heroku 上构建应用时确实安装了 dev 依赖库，但在之后会被自动清除。但是在我们的示例中，为了启动应用程序（npm start script），在生产环境中需要的几个 dev 依赖库。参考：[配置 Heroku 保存 dev 依赖库:](https://devcenter.heroku.com/articles/nodejs-support#package-installation)

{title="Command Line",lang="javascript"}
~~~~~~~~
heroku config:set NPM_CONFIG_PRODUCTION=false YARN_PRODUCTION=false
~~~~~~~~

在现实环境下，一般使用其他东西来启动应用程序，而是不依赖于任何 dev 依赖库。

### 练习：

* 查看[本章源码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/9dbfb30226cdc4843adbcc09d16871b2a902a4d3)
* 欢迎给我们关于 Heroku 的故障排除区域是否有用的反馈
* 使用 GraphQL Playground 创建生产环境测试数据
* 熟悉 [Heroku Dashboard](https://dashboard.heroku.com/apps)
  * 找到应用日志
  * 找到应用环境变量
* 使用 `heroku pg:psql` 访问 PostgreSQL 数据库

<hr class="section-divider">

你已经使用 Express 和 Apollo 服务器构建了一个复杂的 GraphQL 服务器示例项目。也应该已经了解到 GraphQL 不会强制要求身份验证，授权，数据库访问和分页等等。由于 Apollo 服务端使用了基于 JavaScript 实现的 GraphQL，因此我们学到的大多数操作都更直接。没关系，因为很多人都在使用 Apollo 服务端来构建 GraphQL 服务器。使用此应用作为入门项目来实现你自己的想法，或者使用我的入门项目，也就是[这个 GitHub 仓库](https://github.com/the-road-to-graphql/fullstack-apollo-express-postgresql-boilerplate) 中用 React 实现的 GraphQL 客户端。
