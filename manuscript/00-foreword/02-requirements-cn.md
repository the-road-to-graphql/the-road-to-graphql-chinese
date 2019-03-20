## Requirements 基本要求

> To get the most out of this book, you should be familiar with the basics of web development, which includes some knowledge of HTML, CSS and JavaScript. You will also need to be familiar with the term [API](https://www.robinwieruch.de/what-is-an-api-javascript/), because it is discussed frequently. I encourage you to join the official [Slack Group](https://slack-the-road-to-learn-react.wieruch.com/) for the book, to help or get help from others.

要想充分理解本书的内容，你应该熟悉基本的 Web 开发知识，其中包括 HTML，CSS 和 JavaScript。同时你也需要熟悉 [API](https://www.robinwieruch.de/what-is-an-api-javascript/) 这个术语，因为它会经常被提及到。我建议你加入本书的官方 [Slack 讨论组](https://slack-the-road-to-learn-react.wieruch.com/)，在那里你可以帮助他人或者寻求帮助。

### Editor and Terminal 编辑器和终端

> For the development environment, use a running editor or IDE and terminal (command line tool), and [follow my setup guide](https://www.robinwieruch.de/developer-setup/). It is adjusted for MacOS users, but you can find a Windows setup guide, too. There are lots of articles about setting up a web development environment for your OS.

对于开发环境，你需要一个可用的编辑器或者IDE，以及一个终端（命令行工具），然后[参照我的环境搭建指南](https://www.robinwieruch.de/developer-setup/)配置环境。该指南适用于 MacOS 用户，但你也可以在上面找到一个 Windows 的搭建指南。

### React

> On the client-side, this book uses React to teach about GraphQL in JavaScript. My other book called The Road to learn React teaches you all the fundamentals about React. It also teaches you to make the transition from JavaScript ES5 to JavaScript ES6. The book is available for free and after having read The Road to learn React, you should possess all the knowledge to implement the GraphQL client-side application with this book.

在客户端上，本书使用了 React 来讲授 JavaScript 中的 GraphQL。我的另一本书《The Road to learn React》(中文版《React 学习之道》)讲授了所有关于 React 的基础知识。它同时还讲了如何从 JavaScript ES5 迁移到 JavaScript ES6。那本书可以免费阅读，在看完它之后，你应该就掌握了所有本书所需的在客户端实现 GraphQL 的知识。

### Node

> On the server-side, this book uses Node with Express as library to teach about GraphQL in JavaScript. You don't need to know much about those technologies before using them for your first GraphQL powered applications. The book will guide you through the process of setting up a Node application with Express and shows you how to weave GraphQL into the mix. Afterward, you should be able to consume the GraphQL API provided by your server-side application in your client-side application.

至于服务器端上，本书使用了 Node 加 Express 作为库来讲授 JavaScript 中的 GraphQL。在将这些技术用于你的第一个支持 GraphQL 的应用程序之前，你不必了解太多。本书将指导你完成用 Express 设置 Node 应用的过程，并且像你展示如何将 GraphQL 穿插其中。之后，在你的前端应用中就应该能消费你后端应用提供的 GraphQL API 了。

### Node and NPM

> You will need to have [node and npm](https://nodejs.org/en/) installed, which are used to manage the libraries we'll use along the way. In this book, you will install external node packages via npm (node package manager). These node packages can be libraries or whole frameworks. You can verify which node and npm versions you have in the command line. If you don't see output in the terminal, you will need to install node and npm. These are the versions used for this publication:

你需要安装 [node 和 npm](https://nodejs.org/en/)，这两个工具都是用来管理你在本书中所用到的各种库的。本书中提及的 node 包需要通过 npm （node package manager，Node 包管理器）来安装，这些包可能是一些库，或者集成在一起的框架。你可以在命令行验证 node 和 npm 的安装版本。如果没有看到任何输出结果，那就说明你需要安装他们，下面是我在写这本书的时候使用的版本：

{title="Command Line",lang="text"}
~~~~~~~~
node --version
*v10.11.0
npm --version
*v6.4.1
~~~~~~~~

> If you read The Road to learn React, you should be familiar with the setup already, since it introduces the npm ecosystem as well.

如果你看过《React 学习之道》，你应该已经熟悉整个设置过程了，因为那本书中也采用了 npm 生态系统。