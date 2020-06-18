# Pro MERN Stack 2nd Edition book project

This is my repository for the project described in the book Pro MERN Stack (2nd Ed) by Vasan Subramanian.

# Table of Contents

- [Chapter5] (#Chapter 5)
- [Chapter4] (#Chapter 4)
- [Chapter3] (#Chapter 3)
- [Chapter2] (#Chapter 2)

# Chapter 5

In this chapter, I spent most of my time integrating with the back-end. The biggest change is that I started fetching and storing the data using APIs from the Express and Node.js server, which replaced the hard-coded array of issues in the browser’s memory. Using GraphQL, I learned how to build the C and R part of CRUD. I also saw how easy some of the validations were to implement and how the strong type system of GraphQL helps avoid errors and makes the APIs self-documenting. 

![ch05](/readme-images/ch05.png)

### notes

<datails>
<summary>click for details</summary>
- Express
  - Middleware: an Express application is essentially a series of middleware function calls.
    - Middleware functions are those that have access to the request object (`req`), the response object (`res`), and the next middleware function in the application’s request-response cycle. The next middleware function is commonly denoted by a variable named `next`.
    - Middleware can be at the application level (applies to all requests) or at a specific path level (applies to specific request path patterns).
    - To use a middleware at the application level, we simply supply the function to the application, like `app.use(middlewareFunction);`.
    - To use a middleware at a specific path level, the `app.use()` method have to be called with two arguments, the first one being the path, like `app.use('/public', express.static('public'));`.
  - Routing: Express takes a client request, matches it against any routes that are present, and executes the handler function that is associated with that route.
    - A middleware function deals with any request matching the path specification, regardless of the HTTP method. In contrast, a route can match a request with a specific HTTP method.
    - A route specification consists of an HTTP method, a path specification, and the route handler. The handler is passed in a request object and a response object.
  - Request matching: when a request is received, the first thing that Express does is match the request to one of the routes. Further, the request URL is matched with the path specification, which is the first argument in the route. When a HTTP request matches this specification, the handler function is called.
  - Route parameters: named segments in the path specification that match a part of the URL. If a match occurs, the value in that part of the URL is supplied as a variable in the request object.
    - e.g. for `app.get('/customers/:customerId', ...)`, URLs like `/customers/1234`, `/customers/4567` will match the route specification. The customer ID will be captured and supplied to the handler function as part of the request in `req.params`, with the name of the parameter as the key. Thus, `req.params.customerId` will have the value `1234` or `4567` for each of these URLs.
  - Route lookup: multiple routes can be set up to match different URLs and patterns. The router tries to match all routes in the order in which they are installed, and the first match is used. So, the routes have to be defined in the order of priority.
  - Handler function
    - Any aspect of the request can be inspected using the request object’s properties and methods. Some useful properties and methods are `req.params`, `req.query`, `req.header`, `req.get(header)`, `req.path`, `req.url`, `req.originalURL`, `req.body`.
    - The response object is used to construct and send a response. Note that if no response is sent, the client is left waiting. Some useful methods are `res.send(body)`, `res.status(code)`, `res.json(object)`, `res.sendFile(path)`.
- REST (representational state transfer) API
  - Resource based: resources are accessed based on a Uniform Resource Identifier (URI), also known as an *endpoint*.
  - HTTP methods as actions: to access and manipulate the resources, you use HTTP methods
- GraphQL
  - Field specification: in a REST API, specifying no fields of an object will return the entire object, while in GraphQL, it is invalid to request nothing. The properties of an object that need to be returned must be specified.
  - Graph based: REST APIs are resource based, while GraphQL is graph based. This means that relationships between objects are naturally handled in GraphQL APIs.
  - Single endpoint: GraphQL API servers have a single endpoint in contrast to one endpoint per resource in REST.
  - Strongly typed: all fields and arguments have a type against which both queries and results can be validated. It's also possible to specify which fields and arguments are required and which others are optional. All this is done by the GraphQL schema language.
  - Introspection: a GraphQL server can be queried for the types it supports. This let developers test and learn an API set quickly.
- The About API: in this section, we will implement a simple API called About that returns a string, as well as another API that lets us change the string.
  - Install packages `graphql`, `apollo-server-express`.
  - Define the schema using the `type` keyword.
    - For the About API, the basic data type `String` is enough.
    - GraphQL schema has two special types called `Query` and `Mutation`. All other APIs or fields are defined hierarchically under these two types, which are like the entry points into the API. A schema must have at least the `Query` type.
    - In addition to specifying the type, the schema language has a provision to indicate whether the value is optional or mandatory. By default, all values are optional (i.e., they can be null), and those that require a value are defined by adding an exclamation character `!` after the type.
    - Note that all arguments must be named, and all fields must have a type, and there is no void indicating that the field returns nothing. To overcome this, we can just use any data type and make it optional so that the caller does not expect a value.
  - Define *resolvers* that can be called when fields are accessed.
    - The implementation of resolvers depends on the programming language we use.
    - Each field needs to be resolved using a function of the same name as the field.
  - Initialize the GraphQL server by constructing an `ApolloServer` object. The constructor takes in an object with at least two properties, `typeDefs` and `resolvers`, and returns a GraphQL server object.
  - Install the Apollo Server as a middleware in Express using the `applyMiddleware()` method of the `ApolloServer` object.
  - One tool called `Playground` is available by default as part of the Apollo Server and can be accessed simply by browsing the API endpoint (<http://localhost:3000/graphql>). This tool helps us to explore the API.
- GraphQL Schema File
  - When schema grows bigger, it's useful to separate the schema into a file of its own.
  - Create a new file, `schema.graphql`. To read the contents of the file, use the `fs` module and the `readFileSync()` function.
  - Change the `nodemon` tool by adding an `-e` option specifying all the extensions it needs to watch for. In this case, let’s specify `js` and `graphql` as the two extensions for this option.
- The List API: in this section, we'll implement an API to fetch a list of issues.
  - Define a custom type called `Issue`.
  - Add a new field under `Query` to return a list of issues. The GraphQL way to specify a list of another type is to enclose it within square brackets `[]`.
  - Separate the top-level `Query` and `Mutation` definitions from the custom types using a comment, which beginns with `#`.
  - In the server code, add a resolver under `Query` for the new field.
- List API Integration: in this section, we will modify the `IssueList` React component in order to fetch data from the server.
  - To use the APIs, we need to make asynchronous API calls. Modern browsers support Ajax calls natively via the Fetch API.
  - Within the `loadData()` method, construct a GraphQL query, which is a simple string. Send this query string as the value for the `query` property within a JSON, as part of the body to the `fetch` request. The method we’ll use is POST and we’ll add a header that indicates that the content type is JSON.
  - Once the response arrives, we can get the JSON data converted to a JavaScript object using the `response.json()` method. Finally, call a `setState()` to supply the list of issues to the state variable called `issues`. Note that we need to add the keyword `async` for `loadData()` function.
  - Since a string instead of a `Date` object is used to represent the date, we need to remove the conversions in the `IssueRow` component’s `render()` method. We can also now remove the global variable `initialIssues`.
- Custom Scalar Types (to fix the string-formatted dates)
  - The recommended string format for transferring `Date` objects in a JSON is the ISO 8601 format. It is also the same format used by JavaScript `Date`’s `toJSON()` method. It's easy to convert a date to this string format using either `toJSON()` or `toISOString()` methods of `Date`, as well as to convert it back to a date using `new Date(dateString)`.
  - Although GraphQL does not support dates natively, it has support for custom scalar types. After creating a custom scalar type date, it can be used just as any native scalar type like `String` and `Int`.
    - Define a type for the scalar using the `scalar` keyword.
    - Add a top-level resolver for all scalar types, which handles both serialization (on the way out) as well as parsing (on the way in) via class methods.
  - Now, in `App.jsx`, we can convert the string to the native `Date` type.
    - A better way do this is to pass a *reviver* function to the JSON `parse()` function.
- The Create API: in this section, we will implement an API for creating a new issue in the server, which will be appended to the list of issues in the server’s memory.
  - Define a field in the schema under `Mutation` called `issueAdd`.
    - Since this field need to take multiple arguments, one for each property of the issue being added, we can define a new type called `issueInputs` as an object that has the fields we need for the input.
    - GraphQL needs a different specification when it comes to input types. Instead of using the `type` keyword, we have to use the `input` keyword.
  - Define a resolver for `issueAdd` that takes in an `IssueInput` type and creates a new issue in the in-memory database.
- Create API Integration
  - Before making the API call, we need a query with the values of the fields filled in. Use a template string to generate such a query within the `createIssue()` method in `IssueList`.
  - Use this query to execute `fetch` asynchronously.
- Query Variables
  - Using query strings for dynamic user input can be quite erroe-prone. Instead, we can use *variables* in GraphQL to factor dynamic values out of the query and pass them as a separate dictionary.
  - To use variables, we have to name the operation first, which is done by specifying a name right after the `query` or `mutation` field specification.
  - Next, the input value has to be replaced with a variable name. Variable names start with the `$` character.
  - To supply the value of the variable, we need to send it across in a JSON object that is separate from the query string.
- Input Validations
  - A common way to validate input in GraphQL schema is to use enumeration types (*enums*). These will be dealt with as strings both in the client as well as in the server.
  - GraphQL schema supplies default values in case the input is not given. This can be done by adding an `=` symbol and the default value after the type specification.
  - For programmatic validations, we have to have them before saving a new issue in `server.js`.
- Display Errors
  - Create a common utility function called `graphQLFetch` that handles all API calls and report errors. This will be an async function since we’ll be calling `fetch()` using `await`.
  - All transport errors will be thrown from within the call to `fetch()`, so the call to `fetch()` and the subsequent retrieval of the body should be within a `try-catch` block.
  - Display Errors using `alert` in the `catch` block.
  - Once the `fetch` is complete, we’ll look for errors as part of `result.errors`. The error code can be found within `error.extensions.code`. For `BAD_USER_INPUT`, we’ll need to join all the validation errors together and show it to the user. For all other error codes, we’ll display the code and the message as they are received.
  - Finally, return `result.data` in this utility function.  The caller can check if any data was returned, and if so, use that.
</details>

### troubleshooting

- On **page 92**, the command for installing npm packages should be `npm install graphql@0 apollo-server-express@2.3`, which sets the dependencies for `apollo-server-express` to version 2.3+. Otherwise, there will be a `cannot find module` error when trying to start the server.
- On page 125 Listing 5-17: the function shown as `function validateIssue(_, { issue })` should be `function issueValidate(issue)`.

---

## Chapter 4

In this chapter, I learned how to use React state and how it can be manipulated on user interactions or other events. I added two text input fields and a button for the Issue Tracker to let users append rows to the initial list of issues. A click of the button will cause the state to change and the components to rerender. One important point during this process is that we use *props* to pass the state and the method around between a child and its parent.

![ch04](/readme-images/ch04.png)

### notes

<datails>
<summary>click for details</summary>

- Initial State
  - React uses a data structure called *state* in a component to make it respond to user input and other events.
  - The state of a component is captured in a variable called `this.state` in the component’s class, which should be an object where each key is a state variable name and the value is the current value of that variable.
  - Setting the initial state needs to be done in the constructor of the component.
  - The state essentially holds mutable data, as opposed to the immutable properties in the form of `props` that we saw earlier. This state needs to be used in the `render()` method to build the view. When data or the state changes, React automatically rerenders the view to show the new changed data.
  - It is useful to store in the state anything that affects the rendered view and can change due to any event.
- Async State Initialization
  - Typically, instead of having the initial state available to us, we need to fetch data from the server via an API call.
  - Firstly, the state is assigned a value in the constructor. After that, the state can be modified via a call to `React.Component`’s `this.setState()` method. This method takes in one argument, which is an object containing all the changed state variables and their values.
  - We should not call `this.setState()` within the constructor, because the constructor only constructs the component and does not render the UI. If the initial page is complex and takes time to render, and the Ajax call returns before rendering is finished, you will get an error.
  - React provides methods called *lifecycle methods* to cater to situations where something needs to be done depending on the stage, or changes in the status of the component. We can add one of the methods to the component's class and load the data within it.
    - `componentDidMount()`: This method is called as soon as the component’s representation has been converted and inserted into the DOM.
    - `componentDidUpdate()`: This method is invoked immediately after an update occurs, but it is not called for the initial render. The method also takes the previous props and previous state as arguments, so that it can check the differences between the previous props and state and the current props and state before taking an action.
    - `componentWillUnmount()`: This method is useful for cleanup such as cancelling timers and pending network requests.
    - `shouldComponentUpdate()`: This method can be used to optimize and prevent a rerender in case there is a change in the props or state that really doesn’t affect the output or the view. This method is rarely used.
- Updating State
  - Note that the variable `this.state` in the component is immutable. We can only change it by making a copy of the state variable and then call `this.setState()`.
  - There are libraries called *immutability helpers*, such as `immutable.js`, which can be used to construct the new state object. When a property of the object is modified, the library creates a copy of the object. For cases when we have to make lots of copies because of deep nesting of objects in the state, we may consider using `immutable.js`.
  - For simple cases like we only need append something to the array, we can make a shallow copy of the array using the `slice()` method.
  - React automatically propagates any changes to child components that depend on the parent component’s state, so there's no need for us to explicitly call a `setState()` on the child components.
- Lifting State Up
  - One way that allows sibling classes to communicate with each other is to have the common parent.
  - In the Issue Tracker example, we'll lift the state up on level to `IssueList`, so that information can be propagated down to `IssueAdd` and to `IssueTable`.
    - First, move the state and other methods that deal with the state to the `IssueList` class. After that, to provide `IssueTable` with access to the array of issues, we need to pass it from the state within `IssueList` to `IssueTable` via `props`. Then in the`IssueTable` class, instead of referring to the array of issues via `this.state`, we’ll need to get it from `props`.
    - Second, move the timer to the constructor of the `IssueAdd` class. At the same time, we have to make `createIssue()` method available in the `IssueAdd` class by passing the method as part of the `props`.
    - Last of all, before passing the method around, we have to bind the method to the `IssueList` component using `.bind()` since we're using arrow functions. It's recommended to do this in the constructor of the class where the method is implemented.
- Event Handling
  - In the Issue Tracker example, we want to allow users to add an issue interactively on the click of a button.
    - First, create a form with two text inputs and a button used to trigger the event in the `render()` method of `IssueAdd`. At the same time, we can remove the timer that creates an issue from the constructor.
    - When clicking the button, we want it to call `createIssue()` using the input values, and we want to prevent the form from being submitted because we will handle the event ourselves. We’ll create a method called `handleSubmit()` to finish this task.
    - Rewrite the form declaration with a name and an `onSubmit` handler. Then, implement the method `handleSubmit()`. This method calls the `preventDefault()` on the event to prevent the form from being submitted, creates a new issue by calling `createIssue()`, and clears the text input fields.
    - Like the previous section, `handleSubmit()` needs to be bound to the `IssueAdd` class.
- Stateless Components
  - For components that have nothing but a `render()` method (like `IssueRow` and `IssueTable` in the Issue Tracker example), it is recommended that they are written as functions rather than classes.
  - If a component does not depend on `props`, it can be written as a simple function whose name is the component name.
  - If the rendering depends on the `props` alone, the function can be written with one argument as the `props`.

</details>

---

## Chapter 3

In this chapter, I started to use React classes to instantiate components. I laid out the main page of the Issue Tracker  by generating components dynamically from data and composing them together. I also learned how to pass data from a parent component to its children, which allows us to reuse a component class and to make it to render differently with different data.

![ch03](/readme-images/ch03.png)

### notes

<datails>
<summary>click for details</summary>

- React Classes
  - React classes are used to create real components.
  - React classes are created by extending `React.Component`. Within the body, at least a `render()` method is needed, otherwise the component will have no screen presence.
  - The `render()` function is supposed to return an element, which can be either a native HTML element such as a `<div>` or an instance of another React component.
  - React classes can be instantiated by `<[class-name] />`.
- Composing Components
  - Component composition allows the UI to be split into smaller independent pieces so that each piece can be coded in isolation and can be reused easily.
  - React’s philosophy prefers component composition in preference to inheritance.
- Passing Data Using Properties
  - A component takes inputs (called properties), so we can pass different data from a parent component to a child component and make it render differently on different instances.
  - The easiest way to pass data to child components is using an attribute when instantiating a component. Within the `render()` method of the child, the attribute’s value can be accessed via `this.props.[atrribute]`, where `props` is a special object variable.
  - In most cases, the name of the attribute is the same as the HTML attribute. To avoid conflicts, there are some different naming requirements, like `class` in HTML -> `className` in JSX, hyphen-based-names in HTML-> camelCasedNames in JSX.
  - For the `style` attribute, rather than a CSS kind of string, React needs it to be specified as an object containing JavaScript key-value pairs. The keys are the same as the CSS style name, except that they are camel cased. The values are CSS style values.
- Passing Data Using Children
  - Another way to pass data is to use the contents of the HTML-like node of the component. In the child component, this can be accessed via `this.props.children`.
  - Now, instead of passing data as a property to child components, we can embed it as the child contents like `<[child-class-name] ...> [data] </[child-class-name]>`
- Dynamic Composition
  - In this section, we'll generate a set of components from an array to replace the hard-coded components, and include more fields in the table as well.
  - Modify parent component:
    - Use `map()` method to iterate over the array.
    - The header row in the table will now have more columns. Instead of specifying the style for each cell, We can create a class for the table, and use CSS to style the table and each table-cell. This needs to be part of `index.html`. After that, we can remove the inline `style` attribute from all the table-cells and table-headers.
    - Last of all we need to identify each instance with an attribute called `key`. The value of this key has to uniquely identify a row.
  - Modify child component:
    - Remove all the inline `style` attribute
    - add few more columns

</details>

---

## Chapter 2

In this chapter, I learned to use React to render a simple page and use Node.js and Express to serve that page from a web server. I used nvm to install Node.js, then practiced with how to use npm not just to install node packages, but also to customize command-line instructions. I also got familiar with Babel, which allows us to transform JSX into pure JavaScript and to convert JavaScript into a backwards compatible version to support older browsers.

![ch02](/readme-images/ch02.png)

### notes

<datails>
<summary>click for details</summary>

- Server-Less HelloWorld
  - The React library is available as a JavaScript file that be included in the HTML file using the `<script>` tag.
  - Two scripts should be included in the `<head>` section: (1) the *React core module* that deals with React components; (2) the *ReactDOM module* that converts React components to a DOM that a browser can understand.
  - `React.createElement(type, [props], [...children])` creates a React element. A React element is a JavaScript object that represents what is shown on screen, so it is also called the *virtual DOM*.
  - `ReactDOM.render(element, container[, callback])` transfers React element to the real DOM.
- JSX (JavaScript XML)
  - a markup language used by React to construct an element or an element hierarchy that looks very similar to HTML.
  - Babel compiles JSX into regular JavaScript. It is also available as a JavaScript file.
- Project Setup
  - nvm: node version manager
  - npm: node package manager
  - initialize the project with `npm init` (it actually creates the `package.json` file and sets the characteristics of the project)
  - install packages with `npm install <package>`
- Express
  - Express runs HTTP servers in the Node.js environment. To start using it, import the module using `require` keyword.
  - An Express application is a web server that listens on a specific IP address and port. Multiple applications can be created to listen on different ports.
  - Express gets most of the job done by functions called *middleware*.
    - It takes in HTTP request and response object, responds to requests, or decides to continue with middleware chain by calling the next middleware function.
    - e.g. the built-in `express.static()` function generates a middleware function. It matches the request URL with a file in the directory specified by the parameter. If a file exists, return the contents of the file as the response, if not, chain to the next middleware function.
  - For the application to use the static middleware, we need to mount it with the `use()` method of the application.
  - The `listen()` method of the application starts the server and waits eternally for requests.
  - We can use `node <server-js-file>` command to run the server, or use the `npm start` command after adding the correct entry to the `scripts` section of `package.json`.
- Separate Script File
  - Separate out the JSX and JavaScript and refer to it as an external script.
  - This way, we can keep the HTML as pure HTML and all the script that needs compilation in a separate file.
- JSX Transform
  - Move the transformation of JSX to JavaScript from runtime to build time.
    - First, install required Babel tools (`core`, `cli`, `preset-react`).
    - Then transform JSX into pure JavaScript using `npx babel` command. After this step, we'll see new files with the extension `.js` in the output directory.
    - Finally, make corresponding changes in the html file (change the reference to reflect the new extension, remove the script type specification and the imported runtime transformer library).
- Older Browser Support
  - Install Babel preset called `preset-env`. It can specify the target browsers that we need to support and automatically apply all the transforms and plugins that are required to support those browsers.
  - `.babelrc` is Babel's configuration file that can specify the presets that need to be used. If we contain presets in this file, the `npx babel` command can be run without specifying any presets.
  - Even with Babel's transformation work, code may still not work because of some missing built-ins such as `Array.find()`. *Polyfills* are required to add these new functions’ implementations. Babel provides these polyfills too.
- Automate
  - npm custom commands can be specified in the `scripts` section of `package.json`. These can then be run using `npm run <script>` from the console.
  - To automate transforms, add `"compile": "babel src --out-dir public"`.
  - To automate recompilation upon client-side code changes, add `"start": "nodemon -w server server/server.js"` (requires to install nodemon first).

</details>

### troubleshooting

- Listing 2-1 line 10: `<div id="contents">` should be `<div id="content">`.
- To create a default `package.json` using information extracted from the current directory, use the `npm init` command with the `--yes` or `-y` flag.
