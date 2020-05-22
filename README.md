# Pro MERN Stack 2nd Edition book project

This is my repository for the project described in the book Pro MERN Stack (2nd Ed) by Vasan Subramanian.

## Chapter 2

### notes

- Server-Less HelloWorld
  - The React library is available as a JavaScript file that be included in the HTML file using the `<script>` tag.
  - Two scripts should be included in the `<head>` section: (1) the *React core module* that deals with React components; (2) the *ReactDOM module* that converts React components to a DOM that a browser can understand.
  - `React.createElement(type, [props], [...children])` creates a React element. A React element is a JavaScript object that represents what is shown on screen, so it is also called the *virtual DOM*.
  - `ReactDOM.render(element, container[, callback])` transfers React element to the real DOM.
- JSX (JavaScript XML)
  - a markup language used by React to construct an element or an element hierarchy that looks very similar to HTML.
  - Babel compiles JSX into regular JavaScript.
- Project Setup
  - initialize the project with `npm init` (it actually creates the `package.json` file and sets the characteristics of the project)
  - install packages with `npm install <package>`

### troubleshooting

- Listing 2-1 line 15: `<div id="contents">` should be `<div id="content">`.
- To create a default `package.json` using information extracted from the current directory, use the `npm init` command with the `--yes` or `-y` flag.
