# React with GraphQL

In this client-sided GraphQL application we'll build together, you will learn how to combine React with GraphQL. There is no clever library like [Apollo Client](https://github.com/apollographql/apollo-client) or [Relay](https://github.com/facebook/relay) to help you get started yet, so instead, you will perform GraphQL queries and mutations with basic HTTP requests. Later, in the next application we are going to build together, I'll introduce Apollo as a GraphQL client for your React.js application. For now, the application we build should only show how to use GraphQL in React with HTTP.

Along the way, you will build a simplified GitHub client, basically an issue tracker for GitHub, that consumes [GitHub's GraphQL API](https://developer.github.com/v4/). You will perform GraphQL queries and mutations to read and write data, and by the end, you should be able to showcase a GraphQL in React example that can be used by other developers as a learning tool. The final application you are going to build can be found in this [repository on GitHub](https://github.com/rwieruch/react-graphql-github-vanilla).

## Writing your first React GraphQL Client

After the last sections, you should be ready to use queries and mutations in your React application. In this section, you will create a React application that consumes the GitHub GraphQL API. The application should show open issues in a GitHub repository, making it a simple issue tracker. Again, if you lack experience with React, see [The Road to learn React](https://www.robinwieruch.de/the-road-to-learn-react) to learn more about it. After that you should be well set up for the following section.

For this application, no elaborate React setup is needed. You will simply use [create-react-app](https://github.com/facebook/create-react-app) to create your React application with zero-configuration. Install it with npm by typing the following instructions on the command line: `npm install -g create-react-app`. If you want to have an elaborated React setup instead, read this [setup guide for using Webpack with React](https://www.robinwieruch.de/minimal-react-webpack-babel-setup/).

Now, let's create the application with create-react-app. In your general projects folder, type the following instructions:

{title="Command Line",lang="json"}
~~~~~~~~
create-react-app react-graphql-github-vanilla
cd react-graphql-github-vanilla
~~~~~~~~

After your application has been created, you can test it with `npm start` and `npm test`. Again, after you have learned about plain React in *the Road to learn React*, you should be familiar with npm, create-react-app, and React itself.

The following application will focus on the *src/App.js* file. It's up to you to split out components, configuration, or functions to their own folders and files. Let's get started with the App component in the mentioned file. In order to simplify it, you can change it to the following content:

{title="src/App.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';

const TITLE = 'React GraphQL GitHub Client';

class App extends Component {
  render() {
    return (
      <div>
        <h1>{TITLE}</h1>
      </div>
    );
  }
}

export default App;
~~~~~~~~

The component only renders a `title` as a headline. Before implementing any more React components, let's install a library to handle GraphQL requests, executing queries and mutations, using a HTTP POST method. For this, you will use [axios](https://github.com/axios/axios). On the command line, type the following command to install axios in the project folder:

{title="Command Line",lang="json"}
~~~~~~~~
npm install axios --save
~~~~~~~~

Afterward, you can import axios next to your App component and configure it. It's perfect for the following application, because somehow you want to configure it only once with your personal access token and GitHub's GraphQL API.

First, define a base URL for axios when creating a configured instance from it. As mentioned before, you don't need to define GitHub's URL endpoint every time you make a request because all queries and mutations point to the same URL endpoint in GraphQL. You get the flexibility from your query and mutation structures using objects and fields instead.

{title="src/App.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
# leanpub-start-insert
import axios from 'axios';

const axiosGitHubGraphQL = axios.create({
  baseURL: 'https://api.github.com/graphql',
});
# leanpub-end-insert

...

export default App;
~~~~~~~~

Second, pass the personal access token as header to the configuration. The header is used by each request made with this axios instance.

{title="src/App.js",lang="javascript"}
~~~~~~~~
...

const axiosGitHubGraphQL = axios.create({
  baseURL: 'https://api.github.com/graphql',
# leanpub-start-insert
  headers: {
    Authorization: 'bearer YOUR_GITHUB_PERSONAL_ACCESS_TOKEN',
  },
# leanpub-end-insert
});

...
~~~~~~~~

Replace the `YOUR_GITHUB_PERSONAL_ACCESS_TOKEN` string with your personal access token. To avoid cutting and pasting your access token directly into the source code, you can create a *.env* file to hold all your environment variables on the command line in your project folder. If you don't want to share the personal token in a public GitHub repository, you can add the file to your *.gitignore*.

{title="Command Line",lang="json"}
~~~~~~~~
touch .env
~~~~~~~~

Environment variables are defined in this *.env* file. Be sure to follow the correct naming constraints when using create-react-app, which uses `REACT_APP` as prefix for each key. In your *.env* file, paste the following key value pair. The key has to have the `REACT_APP` prefix, and the value has to be your personal access token from GitHub.

{title=".env",lang="javascript"}
~~~~~~~~
REACT_APP_GITHUB_PERSONAL_ACCESS_TOKEN=xxxXXX
~~~~~~~~

Now, you can pass the personal access token as environment variable to your axios configuration with string interpolation ([template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)) to create a configured axios instance.

{title="src/App.js",lang="javascript"}
~~~~~~~~
...

const axiosGitHubGraphQL = axios.create({
  baseURL: 'https://api.github.com/graphql',
  headers: {
# leanpub-start-insert
    Authorization: `bearer ${
      process.env.REACT_APP_GITHUB_PERSONAL_ACCESS_TOKEN
    }`,
# leanpub-end-insert
  },
});

...
~~~~~~~~

The initial axios setup is essentially the same as we completed using the GraphiQL application before to access GitHub's GraphQL API, when you had to set a header with a personal access token and endpoint URL as well.

Next, set up a form for capturing details about a GitHub organization and repository from a user. It should be possible to fill out an input field to request a paginated list of issues for a specific GitHub repository. First, there needs to be a form with an input field to enter the organization and repository. The input field has to update React's local state. Second, the form needs a submit button to request data about the organization and repository that the user provided in the input field, which are located in the component's local state. Third, it would be convenient to have an initial local state for the organization and repository to request initial data when the component mounts for the first time.

Let's tackle implementing this scenario in two steps. The render method has to render a form with an input field. The form has to have an `onSubmit` handler, and the input field needs an `onChange` handler. The input field uses the `path` from the local state as a value to be a controlled component. The `path` value in the local state from the `onChange` handler updates in the second step.

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  render() {
    return (
      <div>
        <h1>{TITLE}</h1>

        <form onSubmit={this.onSubmit}>
          <label htmlFor="url">
            Show open issues for https://github.com/
          </label>
          <input
            id="url"
            type="text"
            onChange={this.onChange}
            style={{ width: '300px' }}
          />
          <button type="submit">Search</button>
        </form>

        <hr />

        {/* Here comes the result! */}
      </div>
    );
  }
}
~~~~~~~~

Declare the class methods to be used in the render method. The `componentDidMount()` lifecycle method can be used to make an initial request when the App component mounts. There needs to be an initial state for the input field to make an initial request in this lifecycle method.

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  state = {
    path: 'the-road-to-learn-react/the-road-to-learn-react',
  };

  componentDidMount() {
    // fetch data
  }

  onChange = event => {
    this.setState({ path: event.target.value });
  };

  onSubmit = event => {
    // fetch data

    event.preventDefault();
  };

  render() {
    ...
  }
}
~~~~~~~~

The previous implementation uses a React class component syntax you might have not used before. If you are not familiar with it, check this [GitHub repository](https://github.com/the-road-to-learn-react/react-alternative-class-component-syntax) to gain more understanding. Using **class field declarations** lets you omit the constructor statement for initializing the local state, and eliminates the need to bind class methods. Instead, arrow functions will handle all the binding.

Following a best practice in React, make the input field a controlled component. The input element shouldn't be used to handle its internal state using native HTML behavior; it should be React.

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

  render() {
# leanpub-start-insert
    const { path } = this.state;
# leanpub-end-insert

    return (
      <div>
        <h1>{TITLE}</h1>

        <form onSubmit={this.onSubmit}>
          <label htmlFor="url">
            Show open issues for https://github.com/
          </label>
          <input
            id="url"
            type="text"
# leanpub-start-insert
            value={path}
# leanpub-end-insert
            onChange={this.onChange}
            style={{ width: '300px' }}
          />
          <button type="submit">Search</button>
        </form>

        <hr />

        {/* Here comes the result! */}
      </div>
    );
  }
}
~~~~~~~~

The previous setup for the form--using input field(s), a submit button, `onChange()` and `onSubmit()` class methods--is a common way to implement forms in React. The only addition is the initial data fetching in the `componentDidMount()` lifecycle method to improve user experience by providing an initial state for the query to request data from the backend. It is a useful foundation for [fetching data from a third-party API in React](https://www.robinwieruch.de/react-fetching-data/).

When you start the application on the command line, you should see the initial state for the `path` in the input field. You should be able to change the state by entering something else in the input field, but nothing happens with `componentDidMount()` and submitting the form yet.

You might wonder why there is only one input field to grab the information about the organization and repository. When opening up a repository on GitHub, you can see that the organization and repository are encoded in the URL, so it becomes a convenient way to show the same URL pattern for the input field. You can also split the `organization/repository` later at the `/` to get these values and perform the GraphQL query request.

### Exercises:

* Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/ca7b278b8f602c46dfac64a1304d39a8e8e0006b)
* If you are unfamiliar with React, check out *The Road to learn React*

## GraphQL Query in React

In this section, you are going to implement your first GraphQL query in React, fetching issues from an organization's repository, though not all at once. Start by fetching only an organization. Let's define the query as a variable above of the App component.

{title="src/App.js",lang="javascript"}
~~~~~~~~
const GET_ORGANIZATION = `
  {
    organization(login: "the-road-to-learn-react") {
      name
      url
    }
  }
`;
~~~~~~~~

Use template literals in JavaScript to define the query as string with multiple lines. It should be identical to the query you used before in GraphiQL or GitHub Explorer. Now, you can use axios to make a POST request to GitHub's GraphiQL API. The configuration for axios already points to the correct API endpoint and uses your personal access token. The only thing left is passing the query to it as payload during a POST request. The argument for the endpoint can be an empty string, because you defined the endpoint in the configuration. It will execute the request when the App component mounts in `componentDidMount()`. After the promise from axios has been resolved, only a console log of the result remains.

{title="src/App.js",lang="javascript"}
~~~~~~~~
...

const axiosGitHubGraphQL = axios.create({
  baseURL: 'https://api.github.com/graphql',
  headers: {
    Authorization: `bearer ${
      process.env.REACT_APP_GITHUB_PERSONAL_ACCESS_TOKEN
    }`,
  },
});

const GET_ORGANIZATION = `
  {
    organization(login: "the-road-to-learn-react") {
      name
      url
    }
  }
`;

class App extends Component {
  ...

  componentDidMount() {
# leanpub-start-insert
    this.onFetchFromGitHub();
# leanpub-end-insert
  }

  onSubmit = event => {
    // fetch data

    event.preventDefault();
  };

# leanpub-start-insert
  onFetchFromGitHub = () => {
    axiosGitHubGraphQL
      .post('', { query: GET_ORGANIZATION })
      .then(result => console.log(result));
  };
# leanpub-end-insert

  ...
}
~~~~~~~~

You used only axios to perform a HTTP POST request with a GraphQL query as payload. Since axios uses promises, the promise resolves eventually and you should have the result from the GraphQL API in your hands. There is nothing magical about it. It's an implementation in plain JavaScript using axios as HTTP client to perform the GraphQL request with plain HTTP.

Start your application again and verify that you have got the result in your developer console log. If you get a [401 HTTP status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes), you didn't set up your personal access token properly. Otherwise, if everything went fine, you should see a similar result in your developer console log.

{title="Developer Tools",lang="json"}
~~~~~~~~
{
  "config": ...,
  "data":{
    "data":{
      "organization":{
        "name":"The Road to learn React",
        "url":"https://github.com/the-road-to-learn-react"
      }
    }
  },
  "headers": ...,
  "request": ...,
  "status": ...,
  "statusText": ...
}
~~~~~~~~

The top level information is everything axios returns you as meta information for the request. It's all axios, and nothing related to GraphQL yet, which is why most of it is substituted with a placeholder. Axios has a `data` property that shows the result of your axios request. Then again comes a `data` property which reflects the GraphQL result. At first, the `data` property seems redundant in the first result, but once you examine it you will know that one `data` property comes from axios, while the other comes from the GraphQL data structure. Finally, you find the result of the GraphQL query in the second `data` property. There, you should find the organization with its resolved name and url fields as string properties.

In the next step, you're going to store the result holding the information about the organization in React's local state. You will also store potential errors in the state if any occur.

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  state = {
    path: 'the-road-to-learn-react/the-road-to-learn-react',
# leanpub-start-insert
    organization: null,
    errors: null,
# leanpub-end-insert
  };

  ...

  onFetchFromGitHub = () => {
    axiosGitHubGraphQL
      .post('', { query: GET_ORGANIZATION })
      .then(result =>
# leanpub-start-insert
        this.setState(() => ({
          organization: result.data.data.organization,
          errors: result.data.errors,
        })),
# leanpub-end-insert
      );
  }

  ...

}
~~~~~~~~

In the second step, you can display the information about the organization in your App component's `render()` method:

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

  render() {
# leanpub-start-insert
    const { path, organization } = this.state;
# leanpub-end-insert

    return (
      <div>
        <h1>{TITLE}</h1>

        <form onSubmit={this.onSubmit}>
          ...
        </form>

        <hr />

# leanpub-start-insert
        <Organization organization={organization} />
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~~

Introduce the Organization component as a new functional stateless component to keep the render method of the App component concise. Because this application is going to be a simple GitHub issue tracker, you can already mention it in a short paragraph.

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...
}

# leanpub-start-insert
const Organization = ({ organization }) => (
  <div>
    <p>
      <strong>Issues from Organization:</strong>
      <a href={organization.url}>{organization.name}</a>
    </p>
  </div>
);
# leanpub-end-insert
~~~~~~~~

In the final step, you have to decide what should be rendered when nothing is fetched yet, and what should be rendered when errors occur. To solve these edge cases, you can use [conditional rendering](https://www.robinwieruch.de/conditional-rendering-react/) in React. For the first edge case, simply check whether an `organization` is present or not.

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

  render() {
# leanpub-start-insert
    const { path, organization, errors } = this.state;
# leanpub-end-insert

    return (
      <div>
        ...

        <hr />

# leanpub-start-insert
        {organization ? (
          <Organization organization={organization} errors={errors} />
        ) : (
          <p>No information yet ...</p>
        )}
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~~

For the second edge case, you have passed the errors to the Organization component. In case there are errors, it should simply render the error message of each error. Otherwise, it should render the organization. There can be multiple errors regarding different fields and circumstances in GraphQL.

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const Organization = ({ organization, errors }) => {
  if (errors) {
    return (
      <p>
        <strong>Something went wrong:</strong>
        {errors.map(error => error.message).join(' ')}
      </p>
    );
  }

  return (
    <div>
      <p>
        <strong>Issues from Organization:</strong>
        <a href={organization.url}>{organization.name}</a>
      </p>
    </div>
  );
};
# leanpub-end-insert
~~~~~~~~

You performed your first GraphQL query in a React application, a plain HTTP POST request with a query as payload. You used a configured axios client instance for it. Afterward, you were able to store the result in React's local state to display it later.

### GraphQL Nested Objects in React

Next, we'll request a nested object for the organization. Since the application will eventually show the issues in a repository, you should fetch a repository of an organization as the next step. Remember, a query reaches into the GraphQL graph, so we can nest the `repository` field in the `organization` when the schema defined the relationship between these two entities.

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const GET_REPOSITORY_OF_ORGANIZATION = `
# leanpub-end-insert
  {
    organization(login: "the-road-to-learn-react") {
      name
      url
# leanpub-start-insert
      repository(name: "the-road-to-learn-react") {
        name
        url
      }
# leanpub-end-insert
    }
  }
`;

class App extends Component {
  ...

  onFetchFromGitHub = () => {
    axiosGitHubGraphQL
# leanpub-start-insert
      .post('', { query: GET_REPOSITORY_OF_ORGANIZATION })
# leanpub-end-insert
      .then(result =>
          ...
      );
  };

  ...
}
~~~~~~~~

In this case, the repository name is identical to the organization. That's okay for now. Later on, you can define an organization and repository on your own dynamically. In the second step, you can extend the Organization component with another Repository component as child component. The result for the query should now have a nested repository object in the organization object.

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Organization = ({ organization, errors }) => {
  if (errors) {
    ...
  }

  return (
    <div>
      <p>
        <strong>Issues from Organization:</strong>
        <a href={organization.url}>{organization.name}</a>
      </p>
# leanpub-start-insert
      <Repository repository={organization.repository} />
# leanpub-end-insert
    </div>
  );
};

# leanpub-start-insert
const Repository = ({ repository }) => (
  <div>
    <p>
      <strong>In Repository:</strong>
      <a href={repository.url}>{repository.name}</a>
    </p>
  </div>
);
# leanpub-end-insert
~~~~~~~~

The GraphQL query structure aligns perfectly with your component tree. It forms a natural fit to continue extending the query structure like this, by nesting other objects into the query, and extending the component tree along the structure of the GraphQL query. Since the application is an issue tracker, we need to add a list field of issues to the query.

If you want to follow the query structure more thoughtfully, open the "Docs" sidebar in GraphiQL to learn out about the types `Organization`, `Repository`, `Issue`. The paginated issues list field can be found there as well. It's always good to have an overview of the graph structure.

Now let's extend the query with the list field for the issues. These issues are a paginated list in the end. We will cover these more later; for now, nest it in the `repository` field with a `last` argument to fetch the last items of the list.

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const GET_ISSUES_OF_REPOSITORY = `
# leanpub-end-insert
  {
    organization(login: "the-road-to-learn-react") {
      name
      url
      repository(name: "the-road-to-learn-react") {
        name
        url
# leanpub-start-insert
        issues(last: 5) {
          edges {
            node {
              id
              title
              url
            }
          }
        }
# leanpub-end-insert
      }
    }
  }
`;
~~~~~~~~

You can also request an id for each issue using the `id` field on the issue's `node` field, to use a `key` attribute for your list of rendered items in the component, which is considered [best practice in React](https://reactjs.org/docs/lists-and-keys.html). Remember to adjust the name of the query variable when its used to perform the request.

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

  onFetchFromGitHub = () => {
    axiosGitHubGraphQL
# leanpub-start-insert
      .post('', { query: GET_ISSUES_OF_REPOSITORY })
# leanpub-end-insert
      .then(result =>
          ...
      );
  };

  ...
}
~~~~~~~~

The component structure follows the query structure quite naturally again. You can add a list of rendered issues to the Repository component. It is up to you to extract it to its own component as a refactoring to keep your components concise, readable, and maintainable.

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Repository = ({ repository }) => (
  <div>
    <p>
      <strong>In Repository:</strong>
      <a href={repository.url}>{repository.name}</a>
    </p>

# leanpub-start-insert
    <ul>
      {repository.issues.edges.map(issue => (
        <li key={issue.node.id}>
          <a href={issue.node.url}>{issue.node.title}</a>
        </li>
      ))}
    </ul>
# leanpub-end-insert
  </div>
);
~~~~~~~~

That's it for the nested objects, fields, and list fields in a query. Once you run your application again, you should see the last issues of the specified repository rendered in your browser.

### GraphQL Variables and Arguments in React

Next we'll make use of the form and input elements. They should be used to request the data from GitHub's GraphQL API when a user fills in content and submits it. The content is also used for the initial request in `componentDidMount()` of the App component. So far, the organization `login` and repository `name` were inlined arguments in the query. Now, you should be able to pass in the `path` from the local state to the query to define dynamically an organization and repository. That's where variables in a GraphQL query came into play, do you remember?

First, let's use a naive approach by performing string interpolation with JavaScript rather than using GraphQL variables. To do this, refactor the query from a template literal variable to a function that returns a template literal variable. By using the function, you should be able to pass in an organization and repository.

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const getIssuesOfRepositoryQuery = (organization, repository) => `
# leanpub-end-insert
  {
# leanpub-start-insert
    organization(login: "${organization}") {
# leanpub-end-insert
      name
      url
# leanpub-start-insert
      repository(name: "${repository}") {
# leanpub-end-insert
        name
        url
        issues(last: 5) {
          edges {
            node {
              id
              title
              url
            }
          }
        }
      }
    }
  }
`;
~~~~~~~~

Next, call the `onFetchFromGitHub()` class method in the submit handle, but also when the component mounts in `componentDidMount()` with the initial local state of the `path` property. These are the two essential places to fetch the data from the GraphQL API on initial render, and on every other manual submission from a button click.

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  state = {
    path: 'the-road-to-learn-react/the-road-to-learn-react',
    organization: null,
    errors: null,
  };

  componentDidMount() {
# leanpub-start-insert
    this.onFetchFromGitHub(this.state.path);
# leanpub-end-insert
  }

  onChange = event => {
    this.setState({ path: event.target.value });
  };

  onSubmit = event => {
# leanpub-start-insert
    this.onFetchFromGitHub(this.state.path);
# leanpub-end-insert

    event.preventDefault();
  };

  onFetchFromGitHub = () => {
    ...
  }

  render() {
    ...
  }
}
~~~~~~~~

Lastly, call the function that returns the query instead of passing the query string directly as payload. Use the [JavaScript's split method on a string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split) to retrieve the prefix and suffix of the `/` character from the path variable where the prefix is the organization and the suffix is the repository.

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

# leanpub-start-insert
  onFetchFromGitHub = path => {
    const [organization, repository] = path.split('/');
# leanpub-end-insert

    axiosGitHubGraphQL
      .post('', {
# leanpub-start-insert
        query: getIssuesOfRepositoryQuery(organization, repository),
# leanpub-end-insert
      })
      .then(result =>
        this.setState(() => ({
          organization: result.data.data.organization,
          errors: result.data.errors,
        })),
      );
  };

  ...
}
~~~~~~~~

Since the split returns an array of values and it is assumed that there is only one slash in the path, the array should consist of two values: the organization and the repository. That's why it is convenient to use a JavaScript array destructuring to pull out both values from an array in the same line.

Note that the application is not built for to be robust, but is intended only as a learning experience. It is unlikely anyone will ask a user to input the organization and repository with a different pattern than *organization/repository*, so there is no validation included yet. Still, it is a good foundation for you to gain experience with the concepts.

If you want to go further, you can extract the first part of the class method to its own function, which uses axios to send a request with the query and return a promise. The promise can be used to resolve the result into the local state, using `this.setState()` in the `then()` resolver block of the promise.

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const getIssuesOfRepository = path => {
  const [organization, repository] = path.split('/');

  return axiosGitHubGraphQL.post('', {
    query: getIssuesOfRepositoryQuery(organization, repository),
  });
};
# leanpub-end-insert

class App extends Component {
  ...

  onFetchFromGitHub = path => {
# leanpub-start-insert
    getIssuesOfRepository(path).then(result =>
# leanpub-end-insert
      this.setState(() => ({
        organization: result.data.data.organization,
        errors: result.data.errors,
      })),
    );
  };

  ...
}
~~~~~~~~

You can always split your applications into parts, be they functions or components, to make them concise, readable, reusable and [testable](https://www.robinwieruch.de/react-testing-tutorial/). The function that is passed to `this.setState()` can be extracted as higher-order function. It has to be a higher-order function, because you need to pass the result of the promise, but also provide a function for the `this.setState()` method.

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const resolveIssuesQuery = queryResult => () => ({
  organization: queryResult.data.data.organization,
  errors: queryResult.data.errors,
});
# leanpub-start-insert

class App extends Component {
  ...

  onFetchFromGitHub = path => {
# leanpub-start-insert
    getIssuesOfRepository(path).then(queryResult =>
      this.setState(resolveIssuesQuery(queryResult)),
# leanpub-end-insert
    );
  };

  ...
}
~~~~~~~~

Now you've made your query flexible by providing dynamic arguments to your query. Try it by starting your application on the command line and by filling in a different organization with a specific repository (e.g. *facebook/create-react-app*).

It's a decent setup, but there was nothing to see about variables yet. You simply passed the arguments to the query using a function and string interpolation with template literals. Now we'll use GraphQL variables instead, to refactor the query variable again to a template literal that defines inline variables.

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const GET_ISSUES_OF_REPOSITORY = `
  query ($organization: String!, $repository: String!) {
    organization(login: $organization) {
# leanpub-end-insert
      name
      url
# leanpub-start-insert
      repository(name: $repository) {
# leanpub-end-insert
        name
        url
        issues(last: 5) {
          edges {
            node {
              id
              title
              url
            }
          }
        }
      }
    }
  }
`;
~~~~~~~~

Now you can pass those variables as arguments next to the query for the HTTP POST request:

{title="src/App.js",lang="javascript"}
~~~~~~~~
const getIssuesOfRepository = path => {
  const [organization, repository] = path.split('/');

  return axiosGitHubGraphQL.post('', {
# leanpub-start-insert
    query: GET_ISSUES_OF_REPOSITORY,
    variables: { organization, repository },
# leanpub-end-insert
  });
};
~~~~~~~~

Finally, the query takes variables into account without detouring into a function with string interpolation. I strongly suggest practicing with the exercises below before continuing to the next section. We've yet to discuss features like fragments or operation names, but we'll soon cover them using Apollo instead of plain HTTP with axios.

### Exercises:

* Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/c08126a9ec91dde4198ae85bb2f194fa7767c683)
* Explore and add fields to your organization, repository and issues
  * Extend your components to display the additional information
* Read more about [serving a GraphQL API over HTTP](http://graphql.org/learn/serving-over-http/)

## GraphQL Pagination in React

## React 中的 GraphQL 分页

Last section you implemented a list field in your GraphQL query, which fit into the flow of structuring the query with nested objects and a list responsible for showing partial results of the query in React.

上一节中，你在 React 应用上实现了一个用于 GraphQL 查询的列表字段，它能得到一个包含部分查询结果的列表，其中的元素满足指定的嵌套结构。

In this section, you will explore pagination with list fields with GraphQL in React in more detail. Initially, you will learn more about the arguments of list fields. Further, you will add one more nested list field to your query. Finally, you will fetch another page of the paginated `issues` list with your query.

本节将继续在使用 React 的基础上，更为深入地探索基于 GraphQL 列表字段实现的分页功能。你将首先了解列表字段的参数使用，而后在查询中添加另一个嵌套列表字段，从而能够在查询中实现获取下一页 `issues` 的功能。

Let's start by extending the `issues` list field in your query with one more argument:

我们首先为这个 `issues` 列表字段添加一个额外参数：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const GET_ISSUES_OF_REPOSITORY = `
  query ($organization: String!, $repository: String!) {
    organization(login: $organization) {
      name
      url
      repository(name: $repository) {
        name
        url
# leanpub-start-insert
        issues(last: 5, states: [OPEN]) {
# leanpub-end-insert
          edges {
            node {
              id
              title
              url
            }
          }
        }
      }
    }
  }
`;
~~~~~~~~

If you read the arguments for the `issues` list field using the "Docs" sidebar in GraphiQL, you can explore which arguments you can pass to the field. One of these is the `states` argument, which defines whether or not to fetch open or closed issues. The previous implementation of the query has shown you how to refine the list field, in case you only want to show open issues. You can explore more arguments for the `issues` list field, but also for other list fields, using the documentation from Github's API.

如果你在 GraphiQL 工具中通过 "Docs" 边栏查看过这个 `issues` 列表字段的参数信息，就能找到可供使用的参数信息。其中的一个参数便是 `states`，用于决定获取处在开启状态或关闭状态的 issues，亦或两者同时。如果仅需要展示处于开启状态的 issues，那么可以进一步提炼这个列表字段，方法如上述代码所示。通过 GitHub 的 API 文档，你能够查看关于这个 `issues` 列表字段的更多参数信息，其它的列表字段也是一样。

Now we'll implement another nested list field that could be used for pagination. Each issue in a repository can have reactions, essentially emoticons like a smiley or a thumbs up. Reactions can be seen as another list of paginated items. First, extend the query with the nested list field for reactions:

现在我们将实现另一个列表字段，以便用于完善分页功能。我们知道，仓库中的每个 issue 都可能具有 reactions，也就是像笑脸或者大拇指之类的表情。这些 reactions 可以被视作另一项分页内容，为了展示它们，首先在查询中添加一个嵌套的 reactions 列表字段：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const GET_ISSUES_OF_REPOSITORY = `
  query ($organization: String!, $repository: String!) {
    organization(login: $organization) {
      name
      url
      repository(name: $repository) {
        name
        url
        issues(last: 5, states: [OPEN]) {
          edges {
            node {
              id
              title
              url
# leanpub-start-insert
              reactions(last: 3) {
                edges {
                  node {
                    id
                    content
                  }
                }
              }
# leanpub-end-insert
            }
          }
        }
      }
    }
  }
`;
~~~~~~~~

Second, render the list of reactions in one of your React components again. Implement dedicated List and Item components, such as ReactionsList and ReactionItem for it. As an exercise, try keeping the code for this application readable and maintainable.

之后，我们继续通过 React 组件来渲染 reactions 列表。这里本应当为列表和条目定义专门的组件，例如叫做 ReactionsList 和 ReactionItem，可以将此当作课后练习，从而提升应用代码的可读性和可维护性。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Repository = ({ repository }) => (
  <div>
    ...

    <ul>
      {repository.issues.edges.map(issue => (
        <li key={issue.node.id}>
          <a href={issue.node.url}>{issue.node.title}</a>

# leanpub-start-insert
          <ul>
            {issue.node.reactions.edges.map(reaction => (
              <li key={reaction.node.id}>{reaction.node.content}</li>
            ))}
          </ul>
# leanpub-end-insert
        </li>
      ))}
    </ul>
  </div>
);
~~~~~~~~

You extended the query and React's component structure to render the result. It's a straightforward implementation when you are using a GraphQL API as your data source which has a well defined underlying schema for these field relationships.

通过扩展查询结构以及 React 组件内容，你已经成功渲染出相应结果。由于作为数据源的 GraphQL API 为其字段定义了清晰的结构和关联性，这里的实现方式直截了当。

Lastly, you will implement real pagination with the `issues` list field, as there should be a button to fetch more issues from the GraphQL API to make it a function of a completed application. Here is how to implement a button:

最后，为了做出完整的应用，你需要实现一个对 `issues` 列表字段的实际分页功能，通过点击一个按钮从 GraphQL API 获取更多的 issues。以下是按钮部分的实现代码：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Repository = ({
  repository,
# leanpub-start-insert
  onFetchMoreIssues,
# leanpub-end-insert
}) => (
  <div>
    ...

    <ul>
      ...
    </ul>

# leanpub-start-insert
    <hr />
# leanpub-end-insert

# leanpub-start-insert
    <button onClick={onFetchMoreIssues}>More</button>
# leanpub-end-insert
  </div>
);
~~~~~~~~

The handler for the button passes through all the components to reach the Repository component:

该按钮的处理函数经由更外层传入 Repository 组件：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Organization = ({
  organization,
  errors,
# leanpub-start-insert
  onFetchMoreIssues,
# leanpub-end-insert
}) => {
  ...

  return (
    <div>
      <p>
        <strong>Issues from Organization:</strong>
        <a href={organization.url}>{organization.name}</a>
      </p>
      <Repository
        repository={organization.repository}
# leanpub-start-insert
        onFetchMoreIssues={onFetchMoreIssues}
# leanpub-end-insert
      />
    </div>
  );
};
~~~~~~~~

Logic for the function is implemented in the App component as class method. It passes to the Organization component as well.

该函数在 App 组件中作为类方法实现，同样也作为属性传入 Organization 组件。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

# leanpub-start-insert
  onFetchMoreIssues = () => {
    ...
  };
# leanpub-end-insert

  render() {
    const { path, organization, errors } = this.state;

    return (
      <div>
        ...

        {organization ? (
          <Organization
            organization={organization}
            errors={errors}
# leanpub-start-insert
            onFetchMoreIssues={this.onFetchMoreIssues}
# leanpub-end-insert
          />
        ) : (
          <p>No information yet ...</p>
        )}
      </div>
    );
  }
}
~~~~~~~~

Before implementing the logic for it, there needs to be a way to identify the next page of the paginated list. To extend the inner fields of a list field with fields for meta information such as the `pageInfo` or the `totalCount` information, use `pageInfo` to define the next page on button-click. Also, the `totalCount` is only a nice way to see how many items are in the next list:

开始实现功能之前，首先需要考虑如何标明这个分页列表中的下一页。在列表字段中，可以通过添加额外的内部字段来获取相关元信息，例如 `pageInfo` 和 `totalCount`，前者可用于按钮点击的事件处理中定义下一页的相关信息。此外，`totalCount` 可辅助判断下一页中的条目数量：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const GET_ISSUES_OF_REPOSITORY = `
  query ($organization: String!, $repository: String!) {
    organization(login: $organization) {
      name
      url
      repository(name: $repository) {
        ...
        issues(last: 5, states: [OPEN]) {
          edges {
            ...
          }
# leanpub-start-insert
          totalCount
          pageInfo {
            endCursor
            hasNextPage
          }
# leanpub-end-insert
        }
      }
    }
  }
`;
~~~~~~~~

Now, you can use this information to fetch the next page of issues by providing the cursor as a variable to your query. The cursor, or the `after` argument, defines the starting point to fetch more items from the paginated list.

通过这些信息，现在你能将这个游标（Cursor）作为变量传入查询中，从而获取下一页的 issues 内容。这个游标，也就是 `after` 参数，在分页列表中定义了获取更多条目所需的起始位置。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

  onFetchMoreIssues = () => {
# leanpub-start-insert
    const {
      endCursor,
    } = this.state.organization.repository.issues.pageInfo;
# leanpub-end-insert

# leanpub-start-insert
    this.onFetchFromGitHub(this.state.path, endCursor);
# leanpub-end-insert
  };

  ...
}
~~~~~~~~

The second argument wasn't introduced  to the `onFetchFromGitHub()` class method yet. Let's see how that turns out.

名为 `onFetchFromGitHub` 的类方法尚未定义第二个参数，现在让我们来定义它。

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const getIssuesOfRepository = (path, cursor) => {
# leanpub-end-insert
  const [organization, repository] = path.split('/');

  return axiosGitHubGraphQL.post('', {
    query: GET_ISSUES_OF_REPOSITORY,
# leanpub-start-insert
    variables: { organization, repository, cursor },
# leanpub-end-insert
  });
};

class App extends Component {
  ...

# leanpub-start-insert
  onFetchFromGitHub = (path, cursor) => {
    getIssuesOfRepository(path, cursor).then(queryResult =>
      this.setState(resolveIssuesQuery(queryResult, cursor)),
# leanpub-end-insert
    );
  };

  ...
}
~~~~~~~~

The argument is simply passed to the `getIssuesOfRepository()` function, which makes the GraphQL API request, and returns the promise with the query result. Check the other functions that call the `onFetchFromGitHub()` class method, and notice how they don't make use of the second argument, so the cursor parameter will be `undefined` when it's passed to the GraphQL API call. Either the query uses the cursor as argument to fetch the next page of a list, or it fetches the initial page of a list by having the cursor not defined at all:

新的参数被直接传入 `getIssuesOfRepository()` 函数，后者用于构造 GraphQL 的 API 请求，并返回对应查询结果的 Promise。通过观察其它对 `onFetchFromGitHub()` 类方法的调用，能够注意到它们并未使用第二个参数，因此实际传入 GraphQL API 调用中的参数是 `undefined`。最终，这个查询要么使用了传入的游标信息作为参数来获取下一页的列表，要么由于游标未定义获取到第一页的列表。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const GET_ISSUES_OF_REPOSITORY = `
  query (
    $organization: String!,
    $repository: String!,
# leanpub-start-insert
    $cursor: String
# leanpub-end-insert
  ) {
    organization(login: $organization) {
      name
      url
      repository(name: $repository) {
        ...
# leanpub-start-insert
        issues(first: 5, after: $cursor, states: [OPEN]) {
# leanpub-end-insert
          edges {
            ...
          }
          totalCount
          pageInfo {
            endCursor
            hasNextPage
          }
        }
      }
    }
  }
`;
~~~~~~~~

In the previous template string, the `cursor` is passed as variable to the query and used as `after` argument for the list field. The variable is not enforced though, because there is no exclamation mark next to it, so it can be `undefined`. This happens for the initial page request for a paginated list, when you only want to fetch the first page. Further, the argument `last` has been changed to `first` for the `issues` list field, because there won't be another page after you fetched the last item in the initial request. Thus, you have to start with the first items of the list to fetch more items until you reach the end of the list.

上面的模板字符串中，`cursor` 作为变量传入到该列表字段的 `after` 参数中。因为没有后置的的感叹号声明强制要求，所以可以是 `undefined`，这种场景出现在分页列表中获取第一页的情况下。此外，`issues` 列表字段中的 `last` 参数改成了 `first`，因为获取到最后一个条目之后就不存在下一页了，因此必须从第一个条目开始不断获取更多内容，直到到达列表中的最后一个条目为止。

That's it for fetching the next page of a paginated list with GraphQL in React, except one final step. Nothing updates the local state of the App component about a page of issues yet, so there are still only the issues from the initial request. You want to merge the old pages of issues with the new page of issues in the local state of the App component, while keeping the organization and repository information in the deeply nested state object intact. The perfect time for doing this is when the promise for the query resolves. You already extracted it as a function outside of the App component, so you can use this place to handle the incoming result and return a result with your own structure and information. Keep in mind that the incoming result can be an initial request when the App component mounts for the first time, or after a request to fetch more issues happens, such as when the "More" button is clicked.

以上就基本是在 React 应用中使用 GraphQL 获取分页列表所需的过程，不过还差最后一步。目前为止，App 组件中关于 issues 的状态并未得到更新，因此仅显示第一个请求中的结果。为了在保持组织和仓库基本信息不变的情况下合并 issues 的分页列表，最佳的操作时间就是查询得到的 Promise 完成时。由于这个过程已经被提取为 App 组件之外的独立函数，因此可以在该函数中对输入数据进行必要的数据处理。需要注意的是，这里的输入数据既可以是应用挂载时的第一次请求结果，也可以是点击 "More" 按钮时获取的后续结果。

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const resolveIssuesQuery = (queryResult, cursor) => state => {
  const { data, errors } = queryResult.data;

  if (!cursor) {
    return {
      organization: data.organization,
      errors,
    };
  }

  const { edges: oldIssues } = state.organization.repository.issues;
  const { edges: newIssues } = data.organization.repository.issues;
  const updatedIssues = [...oldIssues, ...newIssues];

  return {
    organization: {
      ...data.organization,
      repository: {
        ...data.organization.repository,
        issues: {
          ...data.organization.repository.issues,
          edges: updatedIssues,
        },
      },
    },
    errors,
  };
};
# leanpub-end-insert
~~~~~~~~

The function is a complete rewrite, because the update mechanism is more complex now. First, you passed the `cursor` as an argument to the function, which determines whether it was an initial query or a query to fetch another page of issues. Second, if the `cursor` is `undefined`, the function can return early with the state object that encapsulates the plain query result, same as before. There is nothing to keep intact in the state object from before, because it is an initial request that happens when the App component mounts or when a user submits another request which should overwrite the old state anyway. Third, if it is a fetch more query and the cursor is there, the old and new issues from the state and the query result get merged in an updated list of issues. In this case, a JavaScript destructuring alias is used to make naming both issue lists more obvious. Finally, the function returns the updated state object. Since it is a deeply nested object with multiple levels to update, use the JavaScript spread operator syntax to update each level with a new query result. Only the `edges` property should be updated with the merged list of issues.

由于更新机制变得更复杂，该函数也经历了全面重写。首先，`cursor` 被作为函数参数传入，用于确定是首次查询还是获取下一页内容的查询；随后，当 `cursor` 是 `undefined` 时函数会提前结束并将查询结果简单包装后直接返回，与原先的逻辑相同，这里对应于 App 组件挂载或者用户提交新的输入的场景，应当覆盖之前的状态；其次，当 `cursor` 存在也就是加载更多的查询时，查询结果中的 issues 列表会与已有的列表进行合并，为了显得更为直观而使用了 JavaScript 解构别名语法；最后，该函数返回了更新后的对象，由于存在多层嵌套而使用了 JavaScript 扩展（Spread）语法逐层更新，不过只有 `edges` 属性需要更新为合并后的 issues 列表。

Next, use the `hasNextPage` property from the `pageInfo` that you requested to show a "More" button (or not). If there are no more issues in the list, the button should disappear.

之后，根据 `pageInfo` 对象中的 `hasNextPage` 来决定是否显示 "More" 按钮，当没有更多页面时按钮消失。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Repository = ({ repository, onFetchMoreIssues }) => (
  <div>
    ...

    <hr />

# leanpub-start-insert
    {repository.issues.pageInfo.hasNextPage && (
# leanpub-end-insert
      <button onClick={onFetchMoreIssues}>More</button>
# leanpub-start-insert
    )}
# leanpub-end-insert
  </div>
);
~~~~~~~~

Now you've implemented pagination with GraphQL in React. For practice, try more arguments for your issues and reactions list fields on your own. Check the "Docs" sidebar in GraphiQL to find out about arguments you can pass to list fields. Some arguments are generic, but have arguments that are specific to lists. These arguments should show you how finely-tuned requests can be with a GraphQL query.

至此你已经成功地在 React 应用中实现了基于 GraphQL 的分页功能，如果需要，可以自行练习更多关于 issues 和 reactions 的参数使用。通过 GraphiQL 中地 "Docs" 边栏可以查看列表字段的参数信息，有些参数是公共的，而有些是部分列表所特有的，这些参数将帮助你实现精致的 GraphQL 查询。

### Exercises:

### 练习：

* Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/060677346e8955fb1a6c7579859ce92e62e1f406)
* 确认[上一节的源代码](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/060677346e8955fb1a6c7579859ce92e62e1f406)
* Explore further arguments, generic or specific for the type, on the `issues` and `reactions` list fields
  * Think about ways to beautify the updating mechanism of deeply nested state objects and [contribute your thoughts to it](https://github.com/rwieruch/react-graphql-github-apollo/pull/14)
* 探索更多关于 `issues` 和 `reactions` 列表字段的参数信息，包括公共和类型独有的参数
  * 思考对深度嵌套状态对象的更新机制的优化方案，并[在此贡献你的想法](https://github.com/rwieruch/react-graphql-github-apollo/pull/14)

## GraphQL Mutation in React

## React 中的 GraphQL 变更操作

You fetched a lot of data using GraphQL in React, the larger part of using GraphQL. However, there are always two sides to such an interface: read and write. That's where GraphQL mutations complement the interface. Previously, you learned about GraphQL mutations using GraphiQL without React. In this section, you will implement such a mutation in your React GraphQL application.

你已经尝试了在 React 应用中通过 GraphQL 获取大量数据，这也是 GraphQL 的主要应用场景。不过，这种接口永远存在两个方向的操作：读取和写入。在 GraphQL 中写入操作被称为变更。此前，你已经在没有 React 的情况下通过 GraphiQL 使用过 GraphQL 变更操作，本节中你将在 React 应用中实现 GraphQL 变更操作。

You have executed GitHub's `addStar` mutation before in GraphiQL. Now, let's implement this mutation in React. Before implementing the mutation, you should query additional information about the repository, which is partially required to star the repository in a mutation.

你已经在 GraphiQL 执行过 GitHub 的 `addStar` 变更操作了，现在将通过 React 实现同样的操作。开始实现 star 某仓库这个变更操作之前，你需要查询关于这个仓库所需的部分额外信息。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const GET_ISSUES_OF_REPOSITORY = `
  query (
    $organization: String!,
    $repository: String!,
    $cursor: String
  ) {
    organization(login: $organization) {
      name
      url
      repository(name: $repository) {
# leanpub-start-insert
        id
# leanpub-end-insert
        name
        url
# leanpub-start-insert
        viewerHasStarred
# leanpub-end-insert
        issues(first: 5, after: $cursor, states: [OPEN]) {
          ...
        }
      }
    }
  }
`;
~~~~~~~~

The `viewerHasStarred` field returns a boolean that tells whether the viewer has starred the repository or not. This boolean helps determine whether to execute a `addStar` or `removeStar` mutation in the next steps. For now, you will only implement the `addStar` mutation. The `removeStar` mutation will be left off as part of the exercise. Also, the `id` field in the query returns the identifier for the repository, which you will need to clarify the target repository of your mutation.

通过 `viewerHasStarred` 字段可以知晓当前用户是否已经 star 了这个仓库，从而帮助确定下一步的变更操作是 `addStar` 还是 `removeStar`。现在你只需要实现 `addStar` 变更操作，而 `removeStar` 部分将作为本节的练习。另外，查询操作中返回的 `id` 字段返回了该仓库的标识符，能够用于在后续变更操作中指明目标。

The best place to trigger the mutation is a button that stars or unstars the repository. That's where the `viewerHasStarred` boolean can be used for a conditional rendering to show either a "Star" or "Unstar" button. Since you are going to star a repository, the Repository component is the best place to trigger the mutation.

触发这个变更操作的最佳场景，莫过于一个对仓库 star 或取消 star 的按钮，通过 `viewerHasStarred` 这个布尔值进行条件渲染，得到一个显示内容为 "Star" 或 "Unstar" 的按钮。鉴于 star 的目标是一个仓库，因此在 Repository 组件中触发该变更操作最合适不过。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Repository = ({
  repository,
  onFetchMoreIssues,
# leanpub-start-insert
  onStarRepository,
# leanpub-end-insert
}) => (
  <div>
    ...

# leanpub-start-insert
    <button
      type="button"
      onClick={() => onStarRepository()}
    >
      {repository.viewerHasStarred ? 'Unstar' : 'Star'}
    </button>
# leanpub-end-insert

    <ul>
      ...
    </ul>
  </div>
);
~~~~~~~~

To identify the repository to be starred, the mutation needs to know about the `id` of the repository. Pass the `viewerHasStarred` property as a parameter to the handler, since you'll use the parameter to determine whether you want to execute the star or unstar mutation later.

为了确定用户 star 的是哪一个仓库，变更操作必须知晓该仓库的 `id` 信息，这里通过 `viewerHasStarred` 属性作为参数传入，以便后续能够区分 star 或取消 star 的场景。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Repository = ({ repository, onStarRepository }) => (
  <div>
    ...

    <button
      type="button"
      onClick={() =>
# leanpub-start-insert
        onStarRepository(repository.id, repository.viewerHasStarred)
# leanpub-end-insert
      }
    >
      {repository.viewerHasStarred ? 'Unstar' : 'Star'}
    </button>

    ...
  </div>
);
~~~~~~~~

The handler should be defined in the App component. It passes through each component until it reaches the Repository component, also reaching through the Organization component on its way.

对应的事件处理函数应该定义在 App 组件中，向下传递至 Repository 组件，中途也经过了 Organization 组件。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Organization = ({
  organization,
  errors,
  onFetchMoreIssues,
# leanpub-start-insert
  onStarRepository,
# leanpub-end-insert
}) => {
  ...

  return (
    <div>
      ...
      <Repository
        repository={organization.repository}
        onFetchMoreIssues={onFetchMoreIssues}
# leanpub-start-insert
        onStarRepository={onStarRepository}
# leanpub-end-insert
      />
    </div>
  );
};
~~~~~~~~

Now it can be defined in the App component. Note that the `id` and the `viewerHasStarred` information can be destructured from the App's local state, too. This is why you wouldn't need to pass this information in the handler, but use it from the local state instead. However, since the Repository component knew about the information already, it is fine to pass the information in the handler, which also makes the handler more explicit. It's also good preparation for dealing with multiple repositories and repository components later, since the handler will need to be more specific in these cases.

现在可以尝试在 App 组件中定义这个方法，很容易发现 `id` 和 `viewerHasStarred` 的信息可以从应用的本地状态中解构得到，因此无需在函数中传入便可直接使用。不过，鉴于 Repository 已经知晓这些信息，作为函数参数中传入仍然是合理的选择，并且能够使得数据流更加直观。特别是考虑到之后有通过多个 Repository 组件处理多个仓库的需求，这种场景下函数需要具备更加充分的信息。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

# leanpub-start-insert
  onStarRepository = (repositoryId, viewerHasStarred) => {
    ...
  };
# leanpub-end-insert

  render() {
    const { path, organization, errors } = this.state;

    return (
      <div>
        ...

        {organization ? (
          <Organization
            organization={organization}
            errors={errors}
            onFetchMoreIssues={this.onFetchMoreIssues}
# leanpub-start-insert
            onStarRepository={this.onStarRepository}
# leanpub-end-insert
          />
        ) : (
          <p>No information yet ...</p>
        )}
      </div>
    );
  }
}
~~~~~~~~

Now, you can implement the handler. The mutation can be outsourced from the component. Later, you can use the `viewerHasStarred` boolean in the handler to perform a `addStar` or `removeStar` mutation. Executing the mutation looks similar to the GraphQL query from before. The API endpoint is not needed, because it was set in the beginning when you configured axios. The mutation can be sent in the `query` payload, which we'll cover later. The `variables` property is optional, but you need to pass the identifier.

现在便可以开始实现这个事件处理函数，整个变更操作的执行过程可以从组件外包出去作为独立函数，之后就可以通过 `viewerHasStarred` 这个布尔值判断是执行 `addStar` 还是 `removeStar` 变更操作。GraphQL 中变更操作的执行十分类似于之前使用过的查询操作，API 地址的配置已经在最初的章节通过 axios 完成，并不需要重复进行。变更的内容可以通过 `query` 属性传输，这点我们会在之后详述，虽然 `variables` 属性在 API 中是可选的，但当前需要用于传入仓库标识符。

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const addStarToRepository = repositoryId => {
  return axiosGitHubGraphQL.post('', {
    query: ADD_STAR,
    variables: { repositoryId },
  });
};
# leanpub-end-insert

class App extends Component {
  ...

  onStarRepository = (repositoryId, viewerHasStarred) => {
# leanpub-start-insert
    addStarToRepository(repositoryId);
# leanpub-end-insert
  };

  ...
}
~~~~~~~~

Before you define the `addStar` mutation, check GitHub's GraphQL API again. There, you will find all information about the structure of the mutation, the required arguments, and the available fields for the result. For instance, you can include the `viewerHasStarred` field in the returned result to get an updated boolean of a starred or unstarred repository.

定义 `addStar` 变更操作之前，先再次检查 GitHub 的 GraphQL API，从中你可以找到它的所有结构信息以及必须参数，还有可用的结果字段。例如，你可以在返回结果中包含 `viewerHasStarred` 字段来取得是否 star 该仓库的更新后结果。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const ADD_STAR = `
  mutation ($repositoryId: ID!) {
    addStar(input:{starrableId:$repositoryId}) {
      starrable {
        viewerHasStarred
      }
    }
  }
`;
~~~~~~~~

You could already execute the mutation in the browser by clicking the button. If you haven't starred the repository before, it should be starred after clicking the button. You can visit the repository on GitHub to get visual feedback, though you won't see any results reflected yet. The button still shows the "Star" label when the repository wasn't starred before, because the `viewerHasStarred` boolean wasn't updated in the local state of the App component after the mutation. That's the next thing you are going to implement. Since axios returns a promise, you can use the `then()` method on the promise to resolve it with your own implementation details.

现在已经能够通过在浏览器中点击该按钮来执行变更操作，如果之前你没有 star 过这个仓库，在点击按钮之后就会进行 star。虽然可以在 GitHub 网站上得到可视化的反馈，不过在应用中并不会产生任何反应，即便已经进行 star 之后该按钮仍然会显示 "Star" 标签，这是由于 App 组件中的 `viewerHasStarred` 状态在变更操作后并未得到更新。这也是你下一步的工作，鉴于 axios 返回了一个 Promise，你可以在 `then()` 方法的回调中添加相关实现细节。

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const resolveAddStarMutation = mutationResult => state => {
  ...
};
# leanpub-end-insert

class App extends Component {
  ...

  onStarRepository = (repositoryId, viewerHasStarred) => {
# leanpub-start-insert
    addStarToRepository(repositoryId).then(mutationResult =>
      this.setState(resolveAddStarMutation(mutationResult)),
    );
# leanpub-end-insert
  };

  ...
}
~~~~~~~~

When resolving the promise from the mutation, you can find out about the `viewerHasStarred` property in the result. That's because you defined this property as a field in your mutation. It returns a new state object for React's local state, because you used the function in `this.setState()`. The spread operator syntax is used here, to update the deeply nested data structure. Only the `viewerHasStarred` property changes in the state object, because it's the only property returned by the resolved promise from the successful request. All other parts of the local state stay intact.

在变更操作的 Promise 结果中，你能看到一个 `viewerHasStarred` 属性，这是由于它被定义为变更操作中的一个字段。调用 `resolveAddStarMutation` 函数后产生了一个新的状态对象，用于 `this.setState()` 方法中。这里同样使用了扩展语法来更新深度嵌套结构，这个函数中除了 `viewerHasStarred` 属性之外的内容都不会发生变化，因为它是从响应中唯一能够得到的内容。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const resolveAddStarMutation = mutationResult => state => {
# leanpub-start-insert
  const {
    viewerHasStarred,
  } = mutationResult.data.data.addStar.starrable;

  return {
    ...state,
    organization: {
      ...state.organization,
      repository: {
        ...state.organization.repository,
        viewerHasStarred,
      },
    },
  };
# leanpub-end-insert
};
~~~~~~~~

Now try to star the repository again. You may have to go on the GitHub page and unstar it first. The button label should adapt to the updated `viewerHasStarred` property from the local state to show a "Star" or "Unstar" label. You can use what you've learned about starring repositories to implement a `removeStar` mutation.

现在可以重新尝试 star 这个仓库，虽然可能需要先到 GitHub 页面上取消 star。按钮中的文本将根据本地状态中 `viewerHasStarred` 更新后的值来决定显示 "Star" 或是 "Unstar" 标签，你可以根据目前习得的成果实现出 `removeStar` 变更操作。

 We also want to show the current number of people who have starred the repository, and update this count in the `addStar` and `removeStar` mutations. First, retrieve the total count of stargazers by adding the following fields to your query:

我们想要更进一步显示出 star 了该仓库的当前人数，并在 `addStar` 或 `removeStar` 操作后更新，为此，首先需要在查询操作中添加 stargazers 字段来获取总人数：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const GET_ISSUES_OF_REPOSITORY = `
  query (
    $organization: String!,
    $repository: String!,
    $cursor: String
  ) {
    organization(login: $organization) {
      name
      url
      repository(name: $repository) {
        id
        name
        url
# leanpub-start-insert
        stargazers {
          totalCount
        }
# leanpub-end-insert
        viewerHasStarred
        issues(first: 5, after: $cursor, states: [OPEN]) {
          ...
        }
      }
    }
  }
`;
~~~~~~~~

Second, you can show the count as a part of your button label:

随后，你可以将人数作为按钮标签的一部分：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Repository = ({
  repository,
  onFetchMoreIssues,
  onStarRepository,
}) => (
  <div>
    ...

    <button
      type="button"
      onClick={() =>
        onStarRepository(repository.id, repository.viewerHasStarred)
      }
    >
# leanpub-start-insert
      {repository.stargazers.totalCount}
      {repository.viewerHasStarred ? ' Unstar' : ' Star'}
# leanpub-end-insert
    </button>

    <ul>
      ...
    </ul>
  </div>
);
~~~~~~~~

Now we want the count to update when you star (or unstar) a repository. It is the same issue as the missing update for the `viewerHasStarred` property in the local state of the component after the `addStar` mutation succeeded. Return to your mutation resolver and update the total count of stargazers there as well. While the stargazer object isn't returned as a result from the mutation, you can increment and decrement the total count after a successful mutation manually using a counter along with the `addStar` mutation.

现在我们希望这个数量随着 star（或取消 star）一个仓库而更新，而目前的问题就和在没有更新 `addStar` 变更操作成功后没有更新 `viewerHasStarred` 属性一样，为此在回调函数中一同更新本地的 stargazers 总数。虽然 stargazer 对象并没有作为变更结果返回，但仍然可以在 `addStar` 变更操作成功后人工增加或者减少计数。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const resolveAddStarMutation = mutationResult => state => {
  const {
    viewerHasStarred,
  } = mutationResult.data.data.addStar.starrable;

# leanpub-start-insert
  const { totalCount } = state.organization.repository.stargazers;
# leanpub-end-insert

  return {
    ...state,
    organization: {
      ...state.organization,
      repository: {
        ...state.organization.repository,
        viewerHasStarred,
# leanpub-start-insert
        stargazers: {
          totalCount: totalCount + 1,
        },
# leanpub-end-insert
      },
    },
  };
};
~~~~~~~~

You have implemented your first mutation in React with GraphQL. So far, you have just implemented the `addStar` mutation. Even though the button already reflects the `viewerHasStarred` boolean by showing a "Star" or "Unstar" label, the button showing "Unstar" should still execute the `addStar` mutation. The `removeStar` mutation to unstar the repository is one of the practice exercises mentioned below.

你已经在 React 应用中实现了首个 GraphQL 的变更操作，虽然目前为止，只有 `addStar` 一个操作。尽管按钮仍然能够通过显示 "Star" 或 "Unstar" 标签来反映 `viewerHasStarred` 的值，但在按钮显示为 "Unstar" 时仍然执行的是 `addStar` 操作，而用来取消 star 某仓库的 `removeStar` 变更操作就作为下面的练习之一。

### Exercises:

### 练习：

* Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/3dcd95e32ef24d9e716a1e8ac144b62c0f41ca3c)
* 确认[上一节的源代码](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/3dcd95e32ef24d9e716a1e8ac144b62c0f41ca3c)
* Implement the `removeStar` mutation, which is used analog to the `addStar` mutation.
  * The `onStarRepository` class method has already access to the `viewerHasStarred` property.
  * Conditionally execute a `addStar` or `removeStar` mutation in the class handler.
  * Resolve the new state after removing a star from a repository.
  * Align your final thoughts with [this implementation](https://github.com/rwieruch/react-graphql-github-vanilla).
* 实现 `removeStar` 变更操作，应当与 `addStar` 操作类似。
  * 仍然通过 `onStarRepository` 类方法访问 `viewerHasStarred` 属性。
  * 在类方法中根据条件执行 `addStar` 或 `removeStar` 变更操作。
  * 取消 star 一个仓库后更新本地状态。
  * 保持你的最终思路与[该实现](https://github.com/rwieruch/react-graphql-github-vanilla)一致。
* Implement the `addReaction` mutation for an issue
* 为特定 issue 实现 `addReaction` 变更操作
* Implement more fine-grained components (e.g. IssueList, IssueItem, ReactionList, ReactionItem)
  * Extract components to their own files and use import and export statements to use them again in the App or other extracted components
* 将组件进一步细分（例如 IssueList、IssueItem、ReactionList 和 ReactionItem）

## Shortcomings of GraphQL in React without Apollo

## React 中不借助 Apollo 使用 GraphQL 的缺点

We implemented a simple GitHub issue tracker that uses React and GraphQL without a dedicated library for GraphQL, using only axios to communicate with the GraphQL API with HTTP POST methods. I think it is important to work with raw technologies, in this case GraphQL, using plain HTTP methods, before introducing another abstraction. The Apollo library offers an abstraction that makes using GraphQL in React much easier, so you will use Apollo for your next application. For now, using GraphQL with HTTP has shown you two important things before introducing Apollo:

我们基于 React 和 GraphQL 实现了一个简单的 GitHub issue 追踪器，并未使用任何专用的 GraphQL 封装库，而是仅仅通过 axios 发送 HTTP POST 请求实现与 GraphQL API 的通信。我认为掌握底层技术是十分重要的，也就是能够在不引入额外抽象的情况下，使用普通 HTTP 请求实现 GraphQL 交互。不过，Apollo 库的抽象能够极大简化在 React 中使用 GraphQL 的成本，因此在下一个应用将使用 Apollo 进行开发。引入 Apollo 之前，使用 HTTP 与 GraphQL 的过程体现出了两项重要内容：

* How GraphQL works when using a puristic interface such as HTTP.
* The shortcomings of using no sophisticated GraphQL Client library in React, because you have to do everything yourself.

* GraphQL 如何结合 HTTP 这样简洁的接口工作。
* 缺乏成熟的 GraphQL 客户端工具情况下的缺点，因为你要将一切从头做起。

Before we move on, I want to address the shortcomings of using puristic HTTP methods to read and write data to your GraphQL API in a React application:

在继续下一章内容之前，我希望总结一下现有实现存在的缺点，即 React 应用中，直接使用 HTTP 方法来实现对 GraphQL API 读写数据的方式：

* **Complementary:** To call a GraphQL API from your client application, use HTTP methods. There are several quality libraries out there for HTTP requests, one of which is axios. That's why you have used axios for the previous application. However, using axios (or any other HTTP client library) doesn't feel like the best fit to complement a GraphQL centred interface. For instance, GraphQL doesn't use the full potential of HTTP. It's just fine to default to HTTP POST and only one API endpoint. It doesn't use resources and methods on those resources like a RESTful interface, so it makes no sense to specify a HTTP method and an API endpoint with every request, but to set it up once in the beginning instead. GraphQL comes with its own constraints. You could see it as a layer on top of HTTP when it's not as important for a developer to know about the underlying HTTP.

* **互补性：**为了从客户端应用中通过 HTTP 请求调用 GraphQL API，有很多优质的库可供选择，其中之一就是 axios，这也是为什么在上述应用中使用它的原因。然而，使用 axios（或者任何其它 HTTP 客户端）并不能很好填补 GraphQL 接口的需求。例如，GraphQL 并不需要使用全部 HTTP 功能，仅需要自动应用 POST 方法和唯一 API 地址即可。由于不需要 RESTful 接口中那样的资源路径和方法定义，因此对每个请求重复输入 HTTP 方法和 API 地址是毫无意义的，而是应当作为一劳永逸的配置。GraphQL 有着自己的约束，可以将其视作 HTTP 的上层抽象，因此底层的 HTTP 实现对于开发人员并不是必要的。

* **Declarative:** Every time you make a query or mutation when using plain HTTP requests, you have to make a dedicated call to the API endpoint using a library such as axios. It's an imperative way of reading and writing data to your backend. However, what if there was a declarative approach to making queries and mutations? What if there was a way to co-locate queries and mutations to your view-layer components? In the previous application, you experienced how the query shape aligned perfectly with your component hierarchy shape. What if the queries and mutations would align in the same way? That's the power of co-locating your data-layer with your view-layer, and you will find out more about it when you use a dedicated GraphQL client library for it.

* **声明式：**每当需要使用 HTTP 请求实现一个查询或变更，你都需要使用 axios 之类的库对专门的 API 地址进行调用，这样是命令式地对后端读取或写入数据。然而，要是存在声明式的手段来构造查询和变更呢？要是可以在视图层组件的协同定义查询和变更呢？上述应用中，你亲身体验了组件的层次结构与查询的结构高度是如何得高度相似，要是查询和变更也同样能够达到此等一致呢？这就是协同定义数据层与视图层带来的强大力量，通过使用一个专用的 GraphQL 客户端库，你将发现更多的简化方式。

* **Feature Support:** When using plain HTTP requests to interact with your GraphQL API, you are not leveraging the full potential of GraphQL. Imagine you want to split your query from the previous application into multiple queries that are co-located with their respective components where the data is used. That's when GraphQL would be used in a declarative way in your view-layer. But when you have no library support, you have to deal with multiple queries on your own, keeping track of all of them, and trying to merge the results in your state-layer. If you consider the previous application, splitting up the query into multiple queries would add a whole layer of complexity to the application. A GraphQL client library deals with aggregating the queries for you.

* **功能支持：**使用普通 HTTP 请求与 GraphQL API 交互时，你将无法发掘 GraphQL 的全部可能性。设想你希望把上述应用中的查询拆分，并且协同定义于实际使用数据的相应组件中，这时 GraphQL 将在视图层中以声明式的方式出现。但是当你缺少工具支持，只能人工处理多个查询，追踪各个查询的关系并且在状态层中合并查询结果。以上述应用为例，拆分查询内容会极大提升应用复杂度，而一个 GraphQL 客户端工具可以高效地实现查询间的级联。

* **Data Handling:** The naive way for data handling with puristic HTTP requests is a subcategory of the missing feature support for GraphQL when not using a dedicated library for it. There is no one helping you out with normalizing your data and caching it for identical requests. Updating your state-layer when resolving fetched data from the data-layer becomes a nightmare when not normalizing the data in the first place. You have to deal with deeply nested state objects which lead to the verbose usage of the JavaScript spread operator. When you check the implementation of the application in the GitHub repository again, you will see that the updates of React's local state after a mutation and query are not nice to look at. A normalizing library such as [normalizr](https://github.com/paularmstrong/normalizr) could help you to improve the structure of your local state. You learn more about normalizing your state in the book [Taming the State in React](https://roadtoreact.com). In addition to a lack of caching and normalizing support, avoiding libraries means missing out on functionalities for pagination and optimistic updates. A dedicated GraphQL library makes all these features available to you.

* **数据处理：**使用基础 HTTP 请求时的原生数据处理可以认为是缺少 GraphQL 功能支持的子集，没有专用工具的情况下，没有其他人会帮你进行数据规范化或者缓存相同请求。如果没有在第一时间对数据进行规范化，那么整个数据层的更新都将变成噩梦，你将不得不面对深层的嵌套对象，导致大量不必要的使用 JavaScript 的展开语法。当你在 GitHub 仓库中查看该应用的实现，你会发现在变更或者查询后 React 本地状态的更新过程并不美观，可以使用像 [normalizr](https://github.com/paularmstrong/normalizr) 这样强大的规范化工具来帮助你改善本地状态结构，在 [Taming the State in React](https://roadtoreact.com) 中可以了解更多关于规范化状态的信息。除了缺乏缓存于规范化过程之外，缺少工具意味着缺乏像分页和乐观更新这样的功能，而一个专用的 GraphQL 工具能够保证所有这些功能可用。

* **GraphQL Subscriptions:** While there is the concept of a query and mutation to read and write data with GraphQL, there is a third concept of a GraphQL **subscription** for receiving real-time data in a client-sided application. When you would have to rely on plain HTTP requests as before, you would have to introduce [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) next to it. It enables you to introduce a long-lived connection for receiving results over time. In conclusion, introducing GraphQL subscriptions would add another tool to your application. However, if you would introduce a GraphQL library for it on the client-side, the library would probably implement GraphQL subscriptions for you.

* **GraphQL 订阅：** GraphQL 中读写数据的概念除了查询和变更外，还存在第三个概念——**订阅**——用于在客户端应用中接收实时数据。当你依赖于基础 HTTP 请求进行读写时，如果需要实现订阅的等效功能，还需要依赖于 [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) 来接收实时数据。后者引入了长链接机制来持续接收结果。总而言之，使用 GraphQL 订阅能够为你的应用再添加一个工具，通过在客户端使用 GraphQL 工具，订阅功能可能已经被工具所自动实现。

I am looking forward to introducing Apollo as a GraphQL client library to your React application. It will help with the aforementioned shortcomings. However, I do strongly believe it was good to learn about GraphQL in React without a GraphQL library in the beginning.

我十分期待能够为这个 React 应用引入 Apollo 作为 GraphQL 客户端工具库，它会帮助解决上述提到的所有缺点。不过，我仍然坚信在不使用工具库的情况下入门 GraphQL 是一个很好的学习方式。

| |

You can find the final [repository on GitHub](https://github.com/rwieruch/react-graphql-github-vanilla). The repository showcases most of the exercise tasks too. The application is not feature complete since it doesn't cover all edge cases and isn't styled. However, I hope the implementation walkthrough with plain GraphQL in React has helped you to understand using only GraphQL client-side in React using HTTP requests. I feel it's important to take this step before using a sophisticated GraphQL client library such as Apollo or Relay.

你能在[这个 GitHub 仓库](https://github.com/rwieruch/react-graphql-github-vanilla)中找到应用的最终版本，它还包含了绝大多数练习的答案。鉴于仍有大量边界场景未覆盖，同时缺少样式，这个应用并不能算开发完成。不过，我希望在 React 中通过 HTTP 手动实现能够帮助你了解客户端的 GraphQL 概念，我认为在使用一个像 Apollo 和 Relay 这样成熟的 GraphQL 客户端工具之前这个步骤是十分重要的。

I've shown how to implement a React application with GraphQL and HTTP requests without using a library like Apollo. Next, you will continue learning about using GraphQL in React using Apollo instead of basic HTTP requests with axios. The Apollo GraphQL Client makes caching your data, normalizing it, performing optimistic updates, and pagination effortless. That's not all by a long shot, so stay tuned for the next applications you are are going to build with GraphQL.

我已经演示了如何在 React 应用中通过普通的 HTTP 请求实现 GraphQL 交互，并未用到 Apollo 等工具库。之后，你将继续学习在 React 应用中通过 Apollo 来使用 GraphQL，而非基于 axios 的基础 HTTP 请求。借助 Apollo GraphQL 客户端能够零成本地实现数据缓存、数据规范化、乐观更新以及分页，甚至远不止这些，使用希望你能为后续 GraphQL 应用的开发做好准备。
