# Pro MERN Stack 2nd Edition book project

This is my repository for the project described in the book Pro MERN Stack (2nd Ed) by Vasan Subramanian.

## Chapter 4

In this chapter, I learned how to use React state and how it can be manipulated on user interactions or other events. I added two text input fields and a button for the Issue Tracker to let users append rows to the initial list of issues. A click of the button will cause the state to change and the components to rerender. One important point during this process is that we use *props* to pass the state and the method around between a child and its parent.

![ch04](/readme-images/ch04.png)

### notes

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

---

## Chapter 3

In this chapter, I started to use React classes to instantiate components. I laid out the main page of the Issue Tracker  by generating components dynamically from data and composing them together. I also learned how to pass data from a parent component to its children, which allows us to reuse a component class and to make it to render differently with different data.

![ch03](/readme-images/ch03.png)

### notes

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

---

## Chapter 2

In this chapter, I learned to use React to render a simple page and use Node.js and Express to serve that page from a web server. I used nvm to install Node.js, then practiced with how to use npm not just to install node packages, but also to customize command-line instructions. I also got familiar with Babel, which allows us to transform JSX into pure JavaScript and to convert JavaScript into a backwards compatible version to support older browsers.

![ch02](/readme-images/ch02.png)

### notes

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

### troubleshooting

- Listing 2-1 line 10: `<div id="contents">` should be `<div id="content">`.
- To create a default `package.json` using information extracted from the current directory, use the `npm init` command with the `--yes` or `-y` flag.
