# Pro MERN Stack 2nd Edition book project

This is my repository for the project described in the book Pro MERN Stack (2nd Ed) by Vasan Subramanian.

## Table of Contents

- [Chapter 15: Deployment](#chapter-15)
- [Chapter 14: Authentication](#chapter-14)
- [Chapter 13: Advanced Features](#chapter-13)
- [Chapter 12: Server Rendering](#chapter-12)
- [Chapter 11: React-Bootstrap](#chapter-11)
- [Chapter 10: React Forms](#chapter-10)
- [Chapter 9: React Router](#chapter-9)
- [Chapter 8: Modularization and Webpack](#chapter-8)
- [Chapter 7: Architecture and ESLint](#chapter-7)
- [Chapter 6: MongoDB](#chapter-6)
- [Chapter 5: Express and GraphQL](#chapter-5)
- [Chapter 4: React State](#chapter-4)
- [Chapter 3: React Components](#chapter-3)
- [Chapter 2: Hello World](#chapter-2)

---

# Chapter Notes

## Chapter 15

In this chapter, I deployed the Issue Tracker application using the most popular Platform as a Service (PaaS), Heroku. First, I transferred from the local MongoDB database to MongoDB Atlas, which enables us to have a database on the cloud. Then, we created two GitHub repositories for API and UI, and deployed them to Heroku separately.

My Issue Tracker on Heroku:  
<https://tracker-ui-charlenelyu.herokuapp.com>

My deployment repos can be found here:  
<https://github.com/charlenelyu/tracker-api>  
<https://github.com/charlenelyu/tracker-ui>

![ch15](/readme-images/ch15.png)

## Chapter 14

In this chapter, I implemented authentication and authorization for the Issue Tracker application. Users can view all information without signing in, but only signed-in users are allowed to make changes. I saw how JWT can be used to persist session information in a stateless, yet secure manner. Then, I saw how authorization works with GraphQL APIs and how it can be extended to perform different authorization checks based on the application’s needs. I also learned how CORS and cookie handling restrictions on the browser come into play when the browser accesses the APIs directly.

![ch14](/readme-images/ch14.png)

### notes

<details>
<summary>click for details</summary>

- Sign-In UI
  - In this section, we'll build the necessary user interface for signing in users.
  - In the navigation bar, add an item with the label "Sign In" on the right side. On clicking this, show a modal dialog that lets the user sign in using a button. On successful sign-in, show the user’s name instead of the Sign In menu item, with a dropdown menu that lets the user sign out.
    - Create a new component called `SignInNavItem` which can be placed in the navigation bar.
    - The state variables and some methods for showing the modal are very similar to the component `IssueAddNavItem`.
    - Use a state variable called `user` to save the signed-in status (`signedIn`) and the name of the user (`givenName`).
    - If the state variable indicates that the user is already signed in, the `render()` method returns a dropdown. If the user is not signed in, the `render()` method returns the menu item for signing in as well as a modal dialog to show a Sign In button.
    - In `Page.jsx`, include the item in the navigation bar.
- Google Sign-In
  - In this section, we'll replace the Sign In button with a button to sign in using Google. Once signed in, we’ll use Google to retrieve the user’s name.
  - Create a project and client ID folloing instructions on <https://developers.google.com/identity/sign-in/web/sign-in>. Save the client ID in `.env`.
  - The recommended method for integration listed in the guide doesn't work well with React. We must display the button ourselves following the guide titled "Customize the Sign-In Button".
    - In `template.js`, include the google library.
    - In `uiserver.js`, pass the Google client ID to the UI.
    - With the `SignInNavItem` component, use the library and implement Google Sign-In.
- Verifying the Google Token
  - In this section, we’ll ensure that the credentials are verified at the back-end. The technique to do this is described in the guide "Authenticate with a Backend Server" at <https://developers.google.com/identity/sign-in/web/backend-auth>.
  - Essentially, the client authentication library returns a token on successful authentication, which can be verified at the back-end using Google’s authentication library for Node.js. We’ll need to send this token from the UI to the back-end for it to be verified.
    - Create a new file `auth.js` in the API server to hold all authentication-related functions.
    - In `auth.js`, implement a series of endpoints as Express routes, which will be exported and later mount in the main Express app.
    - To achieve this, we need to install two libraries: `body-parser` that allows us to access the body of POST requests, and the Google authentication library `google-auth-library`.
    - In `server.js`, mount auth routes in the main app at the path `/auth` in order to separate the namespace from `/graphql`. Thus, the full path to access the signin endpoint will be `/auth/signin`.
    - In `uiserver.js`, add the new endpoint prefix `/auth`.
    - In `SignInNavItem.jsx`, obtain the token by a call to `googleUser.getAuthResponse().id_token`, pass this token to the signin API, gather the resultant JSON, and use the `givenName` field to set the state variable `givenName`.
    - Make change to the `.env` file to switch to proxy mode (temporarily).
- JSON Web Tokens
  - Although we verified the token and used the name from the back-end, we didn't persist the information. The information about the sign-in would disappear on a browser refresh. Further, other APIs can't apply any authorization restrictions.
  - One way to persist the authentication information is by creating a session on the back-end, identified by a cookie. This can be done using the middleware `express-session`, which adds a property in the request called `req.session`.
  - JSON Web Tokens (JWT) can encode all the session information that needs to be stored in a token. The token string has all the information, but the information is encrypted so that it cannot be snooped upon or impersonated.
  - In this section, we’ll establish a session that persists even across server restarts. We’ll use JWT to generate a token and send it back to the browser. On every API call that the UI makes, this token will need to be included, identifying the signed-in user.
    - To generate the JWT in the signin API, we need to install `jsonwebtoken` and `cookie-parser`.
    - In `server.js`, include `cookie-parser` globally for all routes.
    - In `auth.js`: generate a JWT in the signin API via a call to the `sign()` function provided by the `jsonwebtoken` package, and set it as a cookie; create a new API to get the current logged-in status.
    - In `SignInNavItem.jsx`, fetch the authentication information via a call to the `/auth/user` API in the `componentDidMount()` method, and set the state.
- Signing Out
  - Signing out requires two things: the JWT cookie in the browser needs to be cleared and the Google authentication needs to be forgotten.
  - In `auth.js`, implement another API under `/auth` to sign out, which will essentially just clear the cookie.
  - In `SignInNavItem.jsx`, replace the trivial `signOut()` with a call to this API. Also call the Google authentication API’s `signOut()` function.
- Authorization
  - For the Issue Tracker application, signed-in users are allowed to make changes, while unauthenticated users can only read the issues.
  - In this section, we’ll prevent unauthorized modification by reporting an error when an unauthorized operation is attempted.
  - Apollo Server provides a mechanism by which a *context* can be passed through to all resolvers.
    - In `api_handler.js`, create the context that holds user information during initialization of the Apollo serve, and pass this context to each resolver as the third argument.
    - Rather than include the context and check for a valid user in each resolver, we'll reuse code by creating a new function in `auth.js`, which takes in a resolver and returns a function that does this check before executing the resolver.
    - Make changes to protected APIs to prevent unauthenticated access.
- Authorization-Aware UI
  - We also need to change the UI to disable operations that are unavailable.
    - In this section, we’ll disable the Create Issue button in the navigation bar when the user is not signed in. We’ll lift the state up to a common ancestor and let the state flow down as props that can be used in the children.
    - In the next section, we’ll use a different technique that’s more suitable for passing props to components very deep in the hierarchy.
  - Convert the `Page` component into a regular component, move the state variable `user` from `SignInNavItem` to `Page`, pass the `user` variable down to the navigation bar, and further to the `NavItem`s include Create Issue and the Sign In.
  - In the `IssueAddNavItem` component, check for the `signedIn` flag and disable the `NavItem` if it is `false`.
  - In the `SignInNavItem` component, remove the state variable and use new props being passed in. Also, remove `loadData()` as this is being done along with the state in the `Page` component.
- React Context
  - In this section, We’ll disable the Close and Delete buttons in the Issue Table as well as the Submit button in the Edit page using the React Context API.
    - The React Context API that can be used to pass properties across the component hierarchy without making intermediate components aware of it.
  - In a new file called `UserContext`, create a user context that will be passed through to all components that need it.
    - A context can be created using the `React.createContext()` method, which takes in an argument, the default value of the context.
  - In `Page.jsx`, provide the user context.
    - The created context exposes a component called `Provider`, which needs to be wrapped around any component hierarchy that needs the context. The component takes in a prop called `value`, which needs to be set to the value that the context will be set to in all descendent components.
  - Now, all descendants can access the user context.
    - Modify `IssueEdit.jsx` to disable the Submit button based on user context.
    - In `IssueTable.jsx`, convert `IssueRow` to regular component for consuming user context.
- CORS with Credentials
  - In this section, we’ll see how we can get the application to work in the non-proxy mode by relaxing the CORS options, at the same time maintaining security.
  - The default configuration of the Apollo Server enabled CORS and allowed requests to `/graphql`. But since it was not done on `/auth`, it was blocked. The `cors` package lets us to enable CORS for the `/auth` set of routes.
    - After installing the `cors` package, import this package and add a middleware in `auth.js`.
  - The default CORS configuration seems to allow requests, but does not allow cookies to be sent for cross-origin requests. To let credentials also be passed to cross-origins, the following must be done:
    - All XHR calls (calls using the API `fetch()`) must include the header `credentials: 'include'`
    - The CORS middleware should include an origin as a configuration option.
    - Another CORS configuration option called `credentials` must be set to true.
- Server Rendering with Credentials
  - Challenge 1: the initial data fetched for the user credentials goes to the `/auth` endpoint rather than the `/graphql` endpoint, but server rendering relies on the fact that all data fetching calls go through `graphQLFetch()`.
  - Solution: use a new GraphQL API for the authenticated user credentials.
  - Challenge 2: when the user data is fetched, the API call made by the UI server must include the cookie. When called from the UI, the browser would have added this cookie automatically.
  - Solution: let all `fetchData()` static functions receive an optional parameter `cookie`, which can be passed through when any function calls `graphQLFetch()`.
  - Challenge 3: the initial data that is fetched needs to be in addition to any other data fetched prior to rendering using `fetchData()` functions in the views.
  - Solution: all global data fetches such as user credentials will have to be hard-coded while rendering on the server.
- Cookie Domain
  - Create a domain and two sub-domains, both of which point to the localhost. This can be done by editing the `/etc/hosts` file and adding the following line: `127.0.0.1 api.promernstack.com ui.promernstack.com`.
    - To open `/etc/hosts` in VSC, first open the `File>Open` dialog, then press the slash key (`/`) to open another dialog that allows us to specify a directory manually. Write `/etc` to navigate to the directory, then select the `hosts` file to edit it.
  - Modify `.env` so that the API endpoint is based on <api.promernstack.com:3000>, then use <ui.promernstack.com:8000> to access the application.

</details>

### troubleshooting

- Page 490 Listing 14-15: In order to sign out correctly, the domain should be included in the clearCookie response from the server. Change line `res.clearCookie('jwt')` to `res.clearCookie('jwt', { domain: process.env.COOKIE_DOMAIN, });`

## Chapter 13

In this chapter, I added some common features to the Issue Tracker application. First, I refactored the UI part to reuse common code across components that display the Toast messages. Following the higher order component (HOC) pattern, I moved most of the repeated code into a new component. Then, I built the Report page using the aggregate function of MongoDB to summarize data fetched from collections. I also added pagination in the Issue List page, which utilized the skip and offset options of find() in MongoDB. Further, I implemented an undo operation, and displayed a search bar in which the users can type keywords to look for issues.

![ch13-1](/readme-images/ch13-1.png)
![ch13-2](/readme-images/ch13-2.png)

### notes

<details>
<summary>click for details</summary>

- Higher Order Component for Toast
  - To reduce code repetition across the main views for showing and managing Toast messages, let’s create a new component called `ToastWrapper` that wraps each of the main views to add the Toast functionality.
  - Move all the state variables relating to the Toast into the this component and the `dismissToast` method.
  - Within the original component, we need a way to show the error, and dismiss it if required. Let's create the `showError` and `showSuccess` methods in `ToastWrapper` and pass them as props. Also include any other props that the parent may want to pass through.
  - What we really need is something that creates a new component class from the existing component classes. Let’s create a function called `withToast` to do just this, like React Router’s `withRouter` function.
  - Now, wherever the original component is referred, we can simply use `withToast(...)` instead. This pattern is called *Higher Order Component (HOC)*.
  - Change `IssueList`, `IssueEdit`, `IssueAddNavItem` components to use the `withToast` HOC.
- MongoDB Aggregate
  - In preparation for implementing the Report page in the next two sections, we'll explore what MongoDB provides in terms of getting summary data of a collection, that is, *aggregates*.
  - Create more issues in the database so that summaries can look meaningful.
    - Add a Mongo shell script `generate_data.mongo.js` to generate some data.
    - Run this script once to populate the database with 100 new issues via `mongo issuetracker scripts/generate_data.mongo.js`.
  - MongoDB provides the collection method `aggregate()` to summarize and perform various other read tasks on the collection using a *pipeline*.
    - The default call to `aggregate()` without any arguments is identical to a call to `find()`, which returns the entire list of documents in the collection without any manipulation.
    - The MongoDB aggregation pipeline consists of stages. Each stage transforms the documents as they pass through the pipeline.
    - Each stage does not have to produce a one-to-one mapping of the previous stage. (e.g. `group` and `unwind`)
    - The `aggregate()` method takes a single parameter, an array of pipeline stage specifications. Each stage specification is an object with a key indicating the type of the stage and the value holding the parameters for the stage.
  - For the Report page, let’s create a pivot table output that shows the counts of issues assigned to various owners, further grouped by statuses.
- Issue Counts API
  - In this section, we'll implement the API that will help build the Report page.
  - In `schema.graphql`, add the new API.
  - In `issue.js`, place the resolver of the API called `count()`, which takes in a filter specification.
    - Process each of the documents returned by the database and update an object called `stats`.
    - Return an array to the caller by simply calling `Object.values(stats)`.
    - Add it as another exported value.
  - In `api_handler.js`, specify the resolver for the endpoint `issueCounts`.
- Report Page
  - For the Report page, We’ll use a format known as a cross-tab or a pivot table: a table with one axis labeled with the statuses and the other axis with owners.
  - In `IssueFilter.jsx`, change the hard-coded navigation of the Apply button by passing in the base URL as props.
  - In `IssueList.jsx`, pass in the new property.
  - Build the `IssueReport` component.
- List API with Pagination
  - In this section, we’ll modify the List API to support pagination on the Issue List page.
  - Modify the schema to add a count of pages in addition to the list of issues.
  - In `issue.js`, add pagination support to List API.
    - Use the new parameter `page` to skip to the given page and limit the number of objects returned.
    - The MongoDB cursor method `skip()` can be used to get the list of documents starting at an offset. The `limit()` cursor method can be used to limit the output to a certain number.
    - Whenever we use an offset, we need to ensure that the list is in the same order when queried multiple times. To guarantee a certain order, include a sort specification using `id` as the sort key.
    - We also need a count of pages, which needs the count of documents matching this filter.  MongoDB allows us to query the cursor itself for the number of documents it matched.
    - Return both the issues list as well as the page count in the return value.
  - Change the `IssueList` component to accommodate the change.
- Pangination UI
  - In this section, we'll use the new API to display a bar of pages. We'll create our own minimalistic pagination bar instead of using React-Bootstrap's pagination.
  - In the `IssueList` component:
    - Modify the data fetcher to include the total count of pages in the query and save it in the state.
    - In the constructor and the `loadData()` method, set state variables using `data.issueList.issues` (list of issues) and `data.issueList.pages` (total count of pages).
    - In the `render()` function, generate a series of page links and lay out the pagination bar.
  - In `Toast.jsx`, set the toast’s z-index so that it always shows on top.
- Undo Delete API
  - The next feature that we’ll implement is an undo action on the delete operation. In this section, we’ll implement the API required for this.
  - In the schema, add a new mutation called `issueRestore`.
  - The actual implementation of the restore API is similar to the Delete API. The difference is that, instead of moving from the `issues` collection to the `deleted_issues` collection, it has to transfer an issue in the reverse direction.
  - In `api_handler.js`, tie the resolver to the API endpoint in the API handler.
- Undo Delete UI
  - The best place to initiate an undo delete operation is in the Toast message that shows that the issue has been deleted.
  - In the `IssueList` component, include a button that can be clicked to initiate the undo. When the button is clicked, we’ll need to call the Restore API.
- Text Index API
  - In the remaining part, we'll implement an autocomplete search bar that finds all issues matching the words typed, and lets the user pick one of them to directly view.
  - MongoDB’s text index let us quickly get to all the documents that contain a certain term.
    - A text index gathers all the words in all the documents and creates a lookup table that returns all documents containing a certain word.
    - The command that creates an index looks like `db.issues.createIndex({ title: "text" })`
    - The command that looks for issues with a certain term looks like `db.issues.find({ $text: {$search: "click" } })`
    - `db.issues.getIndexes()` gets indexes that are currently in existence.
    - `db.issues.dropIndex('title_text')` drops an index.
  - Save the index in `init.mongo.js`
  - In the GraphQL schema, add one more filter option for the search string.
  - In `issue.js`, change the list resolver to use the new parameter to search for documents.
- Search Bar
  - We'll use React Select instead of implementing the search bar ourselves.
    - React Select needs two callbacks to show the options: `loadOptions` and `filterOptions`.
    - The first is an asynchronous method that needs to return an array of options. Each option is an object with the properties `label` and `value`, the `label` being what the user sees and the `value` being a unique identifier.
    - The second callback is expected to be called for each of the returned options to determine whether or not to show the option in the dropdown.
  - Create a new component that will display a React Select and implement the methods required to fetch the documents.
  - Next, we need to take an action when the user selects one of the displayed issues. React Select provides an `onChange` property, which is a callback that is called when the user selects an item, with the value of the selected item as an argument.
  - Add a few more useful properties to the React Select control, including `instanceId`, `DropdownIndicator` and `value`.
  - In case of errors, wrap the component with `withToast` and supply the `showError` function to the GraphQL fetch function.
  - In `Page.jsx`, add the new component between the two `<Nav>`s.
  - To avoid the search control occupying the entire header, wrap the Search component with a `<div>` or something that restricts the width. Instead of a fixed width, use React-Bootstrap’s `Col` component.

</details>

### troubleshooting

- Page 436 Listing 13-5: line `'Lorem ipsum dolor sit amet, ${i}'` should be `` `Lorem ipsum dolor sit amet, ${i}` ``
- Page 467 Listing 13-24: line `history.push('/edit/${value}');` should be ``history.push(`/edit/${value}`);``

## Chapter 12

In this chapter, I explored server rendering, which generates the entire HTML source on the server and sends it to the browser. The book combined server rendering with browser rendering in a way that optimizes the advantages of both. Making sure that the browser render is identical to the server render is the primary complicating factor in this chapter. Both must share data that has been retrieved by the API.

~~Even though I followed every step through the chapter, I couldn't get the initial data and issue-specific description to be displayed within the page source. Similar issues were posted on Piazza several times, some of them still remain unsolved, and I didn't find a solution to this after trying all approaches mentioned on those posts (including checking typos, recompiling source code). I'll go to the next chapter for now. Hope this issue can be resolved in later sections.~~

**[UPDATE]:** Finally fixed the problem after modifying configurations in `.env`. The updated screenshot looks different from the version it should be becuase it's rendered from the code in chapter 13 instead of chapter 12.

![ch12](/readme-images/ch12-update.png)

### notes

<details>
<summary>click for details</summary>

- Introduction
  - Server Rendering: the entire HTML is constructed on the server and sent to the browser.
  - Browser Rendering: fetch data via APIs and construct the DOM on the browser.
  - The need for server rendering arises when the application needs to be indexed by search engines.
    - Search engines typically start from the root URL and then traverse all the hyperlinks present in the resulting HTML. They don't execute JavaScript to fetch data via Ajax calls and look at the changed DOM.
    - To have pages from an application be properly indexed by search engines, the server needs to respond with the same HTML that will result after the Ajax call in `componentDidMount()` methods and subsequent re-rendering of the page.
  - For the Issue Tracker application, we’ll make it work is as follows:
    - use server rendering when any page is opened for the first time (by typing in URL or refreshing);
    - use browser rendering once any page is loaded and the user navigates to another page.
- New Directory Structure
  - Split current source code into three directories
    - keep the shared React components in `src`
    - move the browser-specific file `App.jsx` to `browser`
    - move the server-specific file `uiserver.js` to `server`
  - Change the linting, compiling, and bundling configurations to reflect the new directory structure
    - As for linting, we now need four `.eslintrc` files, one in the base `ui`, and one in each of the sub-directories.
    - Add a `.babelrc` in the `browser` directory, which is a copy of the one in the `src` directory.
    - In `App.jsx` and `uiserver.js`, change the location of the imported/required files.
    - Change the entry points in `package.json` for the new location of `uiserver.js`, and in `webpack.config.js` for the new location of `App.jsx`.
- Basic Server Rendering
  - The `ReactDOMServer.renderToString()` method is used to create an HTML on the server side.
  - In this section, we'll create a simple `About` page to get familiar with the fundamentals.
    - Create a new `About` component, include it in the application.
    - Compile `About.jsx` manually to pure JavaScript via `npx babel src/About.jsx --out-dir server`.
    - Create a new file `template.js` under the `server` directory to build the HTML.
      - It makes a template out of the `index.html` that can accept the contents of the `<div>` and return the complete HTML.
    - Create a new file `render.js` in the `server` directory for importing `About.js` rendering it to string.
      - It will take in a regular request and response like any Express route handler.
      - It will send out the template as a response, with the body replaced by the markup created from `ReactDOMServer.renderToString()`.
    - In `uiserver.js`, add a new route for `/about` to return server-rendered About page,set the `render()` function as the handler for this route.
- Webpack for the Server
  - It's inconvient to compile JSX files manually.
  - It's also not convenient when `import/export` paradigm and `require/module.exports` paradigm are mixed together, since we have to remember adding the `.default` after every `require()` of a file that uses the `import/export` paradigm.
  - It turns out that Webpack can be used for the server to compile JSX on the fly. It will also let us consistently use the `import/export` paradigm in the UI server codebase.
    - Webpack works quite the same as with the front-end code, but for one difference.
    - Many server-side Node packages such as Express are not compatible with Webpack. So, we’ll have to exclude the third-party libraries from the bundle and rely on `node_modules` to be present in the UI server’s file system.
  - First, add server configuration to the `webpack.config.js` file.
    - The `browserConfig` variable contains the original configuration contents, except some changes to the Babel configuration.
    - As for the `browserConfig` variable, we’ll need an output specification. Let’s compile the bundle into a new directory called `dist` and call the bundle `server.js`.
    - We also need to exclude the modules in `node_modules` using the `webpack-node-externals` module.
  - Now, we are ready to convert all `require()` statements to `import` statements.
    - In `ui/server/.eslintrc`, add a rule to disable import extention.
    - Change `template.js` to use the `import/export` paradigm.
    - Change the extension of `render.js` to `render.jsx`, replace `React.createElement()` with JSX, change all `require()` statements to `import` statements.
    - In `uiserver.js`, change all `require()` statements to `import`, change the HMR initialization routine, use `source-map-support` to make error messages more readable.
  - In `package.json`, change the scripts section to use the bundle to start the server instead of the file `uiserver.js`.
  - Remove the manually generated `About.js`.
  - Build the server bundle via `npx webpack`.
- HMR for the Server
  - Although using Webpack for the server simplifies the compilation process, we still need to restart the server for every change.
  - The solution is to automatically reload the modules in the back-end using HMR, but setting this up is quite complex.
  - We’ll only reload changes to modules based on the shared folder. As for changes to `uiserver.js`, we'll restart the server manually and use HMR for the rest of the code that it includes.
    - Create a new Webpack configuration called `webpack. serverHMR.js` that enables HMR for the server.
      - Rather than copy the entire configuration file and make changes to it, we can base on the original configuration and merge the changes required for HMR using package `webpack-merge`.
      - In `webpack. serverHMR.js`, first load up `serverConfig` from `webpack.config.js`, then merge the `serverConfig` with new changes.
    - Run Webpack with this file as the configuration with the `watch` option (`npx webpack -w --config webpack.serverHMR.js`).
    - In `uiserver.js`, make the server to accept changes.
    - In `package.json`, change the script section to add convenience scripts for starting the UI server.
  - Now, a single command `npm run dev-all` will automatically reflect most changes without having to restart this command.
- Server Router
  - On the server, wrapping a Router around the page, or using `Switch` or `NavLinks`, will throw up errors.
  - On the server, React Router recommends that we use a `StaticRouter` in place of a `BrowserRouter`. Also, whereas the `BrowserRouter` looks at the browser’s URL, the `StaticRouter` has to be supplied the URL.
    - `StaticRouter` takes a property called `location`, which is a static URL. It also needs a property called `context`, for now, we'll just supply an empty object for it.
  - Modify `render.jsx` to render the `Page` component instead of the `About` component, but wrapped around by a `StaticRouter`.
  - Now, we will find that both server rendering and browser rendering are identical for the `About` page:  the navigation bar will appear in both cases.
- Hydrate
  - Although the page looks good, what is rendered is pure HTML markup, without any JavaScript code or event handlers. Thus, there is no user interaction possible in the page.
  - In order to attach event handlers, we have to include the source code and let React take control of the rendered page.
    - Add scripts to `template.js`.
    - In `App.jsx`, change `render()` to `hydrate()` as recommended.
  - This step completes the server rendering sequence of events.
- Data from API
  - In reality, messages in the `About` component should come from the API server. This means that we need to be able to make requests to the API server via `graphQLFetch()` from the server as well.
  - To achieve this, we need to replace the `whatwg-fetch` module with a package called `isomorphic-fetch` that can be used both on the browser as well as Node.js.
  - In `graphQLFetch.js`, we currently have the API endpoint specification in `window.ENV.UI_API_ENDPOINT`, which will not work on the server because there is no variable called `window`. We’ll need to use `process.env` variables.
    - To indicate whether the function is being called in the browser or in Node.js, use Webpack’s plugin called `DefinePlugin` to define a global variable called `__isBrowser__`, which is set to `true` in a browser bundle, but `false` in the server bundle.
    - In `uiserver.js`, set `process.env` if it’s not set.
    - In `.env`, add the new environment variable.
  - In `graphQLFetch.js`, use `isomorphic-fetch` to get the correct API endpoint from `process.env` or from `window.ENV`.
  - Now, before calling `renderToString()`, we can make a call to get the data from the About API via `graphQLFetch()`.
    - To pass the information down to the `About` component while it's being rendered, we need to create a global store for all data.
    - Create a global storage module in the `src` directory called `store.js`, which exports an empty object.
    - In `render.jsx`, call `graphQLFetch()` to get the initial data, and save the data to the global store.
  - In `About.jsx`, read data from the global store to display the real API version.
  - When test the About page, there will be an error `Text content did not match. Server: "Issue Tracker API v1.0" Client: "unknown"`
- Syncing Initial Data
  - What we need to do is to make the browser render identical to the server render. The recommended way is to pass the same initial data resulting from the API call to the browser in the form of a script and use that to initialize the global store.
  - Change `template.js` so that it takes an additional argument (the initial data) and sets it in a global variable in a `<script>` section.
  - In `render.jsx`, pass the initial data to the browser.
  - In `app.jsx`, use initial data to initialize the global store.
  - Now, when test the About page, the React error message is no longer shown. But the other pages will still show an error `Expected server HTML to contain a matching <div> in <div>`
- Common Data Fetcher
  - In this section, we'll add a `componentDidMount()` method in the `About` component, which be converted to a regular component from a stateless function.
    - Use a state variable called `apiAbout` to store and display the API version.
    - Initialize this variable in the constructor from the global store if it has the initial data, set it to `null` if the initial data is missing.
    - Use a common static function called `fetchData()` to fetch the data that can be shared by the `About.jsx` as well as by `render.jsx`.
    - In the `componentDidMount()` method, data can be fetched and set in the state if `apiAbout` has not been initialized by the constructor.
    - In the `render()` method, use the state variable rather than the variable from the store.
  - In `render.jsx`, replace the GraphQL query with a call to `About.fetchData()`.
- Generated Routes
  - In this section, we’ll fix the mismatch errors that React is showing for the rest of the pages.
    - First, instead of returning `index.html`, return the server-rendered HTML using the template for all the pages.
    - Remove `index.html`. At this point, we'll find that the React error for mismatched `<div>` for all the pages is no longer seen.
    - Now the main problem is that the data required via API calls needs to be available before rendering is initiated on the server.
    - The only way to do this is by keeping a common source of truth for the list of routes available. Then, match the request’s URL against each route and figure out which component (and therefore, which fetchData() method) will match.
      - store route metadata in a new file `routes.js`.
    - The same source of truth should also be responsible for generating the actual `<Route>` components during a render.
      - change `Contents.jsx` to generate `<Route>` components from `routes.js`).
  - We still need to replace the call to `About.fetchData()` with something more generic.
    - To do that, we need to use a function called `matchPath()` to determine which of the components would match the current URL that is passed in via the request object in `render.jsx`.
- Data Fetcher with Parameters
  - In this section, we’ll make the `IssueEdit` component render from the server with the data that it requires prepopulated.
    - Separate the data fetcher into a static method as we had done in the `About` component. This method relies on the ID of the issue to fetch the data.
    - In the constructor, check if there is any initial data and use that to initialize the state variable `issue`. Set the state variable to `null` to indicate that it was not preloaded from the server. Delete the data from the store once we’ve consumed it.
    - In the `componentDidMount()` method, look for the presence of the state variable. If it is not `null`, it means that it was rendered from the server. If it’s `null`, it means that the user navigated to this component in the browser from a different page. In this case, we can load the data using `fetchData()`.
    - In the `loadData()` method, replace the original call to `graphQLfetch()` with a call to `fetchData()`.
    - In the `render()` method, return `null` if the issue state variable is `null`.
  - In `render.jsx`, pass in the `match` parameter at the time of server rendering.
  - In `template.js`, replace `JSON.stringify()` with the serialize function.
- Data Fetcher with Search
  - In this section, we’ll implement the data fetcher in the `IssueList` component, where the search (query) string is needed for fetching the correct set of issues.
  - In `render.jsx`, pass the query string in addition to the `match` object to `fetchData()`.
  - In `IssueList.jsx`:
    - Create the data fetcher, move the required code from the `loadData()` method into this new static method.
    - In the `loadData()` method, use this data fetcher instead of making the query directly.
    - In the constructor, use the store and the initial data to set the initial set of issues, delete it once we have consumed it.
    - In the `componentDidMount()` method, avoid loading the data if the state variable has a valid set of issues.
    - In the `render()` method, skip rendering if the state variable issues is set to `null`.
- Nested Components
  - In this section, we'll deal with the `IssueDetail` component.
  - Although React Router’s dynamic routing works great when navigating via links in the UI, when it comes to server rendering, we cannot easily deal with nested routes.
    - One option is to add nesting of routes in `routes.js` and pass the nested route object to the containing component.
    - Another alternative is that the route specification for `IssueList` includes an optional Issue ID, and this component deals with the loading of the detail part too.
  - We'll choose to modify the component `IssueList` to render its contained detail as well. This will cause the `IssueDetail` component to be greatly simplified to a stateless component.
  - In `routes.js`, Modify the `/issues` route to specify the ID parameter, which is optional.
  - Pull Up `IssueDetail` into `IssueList`.
- Redirects
  - There's one last thing to take care of: a request to the home page (`/`), returns an HTML from the server that contains an empty page.
  - React Router’s `StaticRouter` handles this by setting a variable called `url` in any context that is passed to it.

</details>

### troubleshooting

- Page 379 Listing 12-3: `rules`, `error`, `always` and `ignorePackages` need to be double quoted.
- Page 401 Listing 12-29: line `process.env.UI_API_ENDPOINT = process.env.UI_API_ENDPOINT;` should be `process.env.UI_SERVER_API_ENDPOINT = process.env.UI_API_ENDPOINT;`
- Page 407 Listing 12-39: line `const resultData = About.fetchData();` should be `const initialData = About.fetchData();`

## Chapter 11

In this chapter, I learned the basics of React-Bootstrap, a UI framework to make a web application look professionally styled and be responsive. With its help, I added some polish to the UI of the Issue Tracker.

![ch11](/readme-images/ch11.png)

### notes

<details>
<summary>click for details</summary>

- Bootstrap Installation
  - Install React-Bootstrap via `npm install react-bootstrap@0`.
  - React-Bootstrap contains a library of React components and has no CSS styles or themes itself. It requires Bootstrap stylesheet to be included in the application to use these components.
  - Install Bootstrap via `npm install bootstrap@3`.
  - The latest version of Bootstrap (Version 4) is not yet supported by React-Bootstrap.
  - Include the Bootstrap stylesheet in the application.
    - We’ll just keep a symbolic link to the Bootstrap distribution under the `public` directory and include the CSS like the other static files such as `index.html`.
    - The command to achieve this is `ln -s ../node_modules/bootstrap/dist public/bootstrap`.
    - In `index.html`, add a link to the main Bootstrap style sheet. (`<link rel="stylesheet" href="/bootstrap/css/bootstrap.min.css">`)
  - To fit the application into the mobile screen, add a meta tag called `view point` in the main page. (`<meta name="viewport" content="width=device-width, initial-scale=1.0">`)
- Buttons
  - In this section, we'll replace some buttons in the Issue Filter with Bootstrap buttons.
  - A simple text-based button can be created using the `<Button>` component, which uses the `bsStyle` property to make buttons look distinct.
  - Apart from the default, which shows the button with a white background, the allowed styles are `primary`, `success`, `info`, `warning`, `danger`, and `link`.
  - To use icons, we need to use the `Glyphicon` component. The `bsSize` property can set the size of the icon.
  - Since the icons’ intended actions are not too obvious, it’s good to have a tooltip that is shown on hovering over the button.
  - In React, the way to do this to us the `OverlayTrigger` component to wrap the button and takes in the `Tooltip` component as a property.
- Navigation Bar
  - In this section, we’ll style the navigation links in the header and add a footer that is visible on all pages.
  - The starting component to create a navigation bar is `Navbar`. Each item is a `NavItem`. These items can be grouped together in a `Nav`.
    - For Issue Tracker, we'll use two `Nav` elements, one for the left side of the navigation bar and another for the right side.
    - The right side `Nav` can be aligned to the right using the `pullRight` property.
  - As for the application title, use `Navbar.Header` and `Navbar.Brand`, which should appear before all the `Nav` elements.
  - As for the extended dropdown menu, use component `NavDropdown`. Each menu item is a `MenuItem` component.
  - Add links to the navigation items. The recommended way is to use the `react-router-bootstrap` package, which provides a wrapper called `LinkContainer` acting as the React Router’s `NavLink`, at the same time letting its children have their own rendering.
    - Install the `react-router-bootstrap` package.
    - Place each `NavItem` on the left-side as a child of the `LinkContainer`.
    - `LinkContainer` supports all properties that the `NavLink` does. Use the `to` property to set the path of the routes.
- Panels
  - In this section, we'll decorate the Filter section using the `Panel` component, which is a great way to show sections separately using a border and an optional heading.
  - The `Panel` component consists of an optional heading (`Panel.Heading`) and a panel body (`Panel.Body`). The panel body is where we’ll place the `<IssueFilter />` instance.
  - To add a heading with the text Filter, wrap the text inside a `Panel.Title` component to make it stand out.
  - To save space, we can collapse the panel by adding the `collapsible` property to the `Panel.Body` and making the panel title control the collapse behavior by setting its `toggle` property.
  - In `Page.jsx`, wrap the body of the page with a `<Grid>` component to add margins.
  - There are two kinds of grid containers in Bootstrap: a fluid one, which fills the entire page, and a fixed one (the default), which has a fixed size, but one that adapts to the screen size.
- Table
  - In this section, we’ll convert the plain table into a Bootstrap table, which looks better, expands to fit the screen, and highlights a row on hover. Further, we’ll make the entire row clickable to select the issue to display its description. We’ll also convert the Edit link into a button.
  - To achieve this, we'll use the `LinkContainer` and `Table` components in `IssueTable.jsx`.
  - Useful Properties of `Table` component
    - `striped`: Highlights alternate rows with a different background.
    - `bordered`: Adds a border around the rows and cells.
    - `condensed`: reduce white space around the text.
    - `hover`: Highlights the row under the cursor.
    - `responsive`: On smaller screens, makes the table horizontally scrollable instead of reducing the width of the columns.
  - Replace `<table>` with `<Table>`
  - To replace the `Select` link with the entire row, use a `LinkContainer` to wrap the entire row and let it navigate to the same location using the `to` property.
    - Note that `LinkContainer` automatically adds the active class to the wrapped element if the route matches the link path. Bootstrap will highlight such rows with a gray background.
  - Use the `Button` and `Glyphicon` components and a `Tooltip` and `OverlayTrigger` to convert the Edit link to a button with an icon. But instead of the `onClick()` event handler, we'll wrap the button with a `LinkContainer`, and set the `to` property to the original link.
  - There's still a minor problem: a click on the Close or Delete buttons will also have the side-effect of selecting the row. This is because we now have an `onClick()` handler on the row.
    - Separate the handlers into explicit functions (as opposed to anonymous functions within the onClick property).
    - Call `e.preventDefault()` in the handlers.
  - In `index.html`, remove all old styles for the table, and give an indication that the table rows are clickable by changing the cursor to a pointer.
- Forms
  - In this section, we’ll use many new React-Bootstrap components to replace the simple `<input>` and `<select>` options in React forms.
  - The common input types are instantiated using a `FormControl`.
    - By default, it uses a regular `<input>` type to render the actual element.
    - The `componentClass` property can change this default to any other element type, (e.g. `select`)
    - The rest of the properties, like `value` and `onChange`, are the same as the `<input>` or `<select>` elements.
  - A label can be associated with `FormControl` using the `ControlLabel` component. The only child of this component is the label text.
  - To keep the label and the control together, they need to be put together under a `FormGroup`.
  - For inputs like Effort that are made up of two inputs, we can use an `InputGroup` to enclose two `FormControls`
    - By default, it will cause the two inputs to appear one below the other.
    - The `InputGroup.Addon` component can be used to display the inputs next to each other, as well as show the dash between the two inputs.
  - For now, we used a space character between the Apply and Reset buttons. A better way is to use the `ButtonToolbar` component.
- The Grid System
  - In this section, we'll use Bootstrap’s grid system to lay out the Issue Filter in a better way, letting each field float horizontally, but one below the other on smaller screens.
  - Grid System Basics
    - The horizontal space is divided into a maximum of 12 columns.
    - A cell (`Col` component) can occupy one or more columns.
    - The cells wrap if there are more than 12 column-space cells within a row (`Row` component). A new row is required if there’s a need to force a break in the flow of cells.
    - In the fluid grid system, rows are more like paragraphs rather than lines. A paragraph (row) can contain multiple lines. As the paragraph width (screen width) reduces, it will need more lines. It’s only when you want to break two sets of sentences (sets of cells) that you really need another paragraph (row).
    - Properties `xs`, `sm`, `md`, and `lg` of the `Col` component denote different screen width from extra small to large. If not specified, the value applicable to the screen size *lesser* than this size will be used (e.g. using `xs` means the same cell widths are used for all screen sizes).
  - As for forms, the best way to use the grid system is to have a single row and specify how many columns each `FormControl` (one cell) occupies at different screen widths.
    - In the filter, we have three cells of roughly equal width: status input (with its label), effort inputs (with their label), and the buttons (together).
- Inline Forms
  - For small forms with two or three inputs that can all fit in one line, we want the form controls next to each other, including the labels. The Issue Add form in our application is a great example.
  - Differences between inline form and the grid-based form:
    - For the grid-based forms, we didn’t have to enclose the controls within a `<Form>`, since the default behavior of the groups was a vertical layout (one below the other, including labels). For inline forms, we need a `<Form>` with the inline property to wrap around the form controls.
    - Unlike the grid-based form, an inline form needs no columns and rows. The `FormGroup` can be placed one after the other.
    - In the inline forms, the button does not need a `FormGroup` around it, and there are no alignment implications if a `ControlLabel` is not given for a button.
    - For inline forms, we need to manually add space using `{' '}` between the label and the control, as well as between form groups.
- Horizontal Forms
  - In this section, we'll change the Issue Edit page to a horizontal form, where the label appears to the left of the input, but each field appears one below the other. We'll also use the validation states that Bootstrap provides to highlight invalid input.
  - First, replace `<form>` with Bootstrap’s `<Form>`. To lay out a horizontal form, we need the `horizontal` property.
  - Enclose the entire form in a `Panel`, with the contents of the original `<h3>` as the header. `Panel.Body` consists of the form and the validation message after it. Next and Prev links can be placed in `Panel.Footer`.
  - To specify how much width the label and the input will occupy, we need to enclose them within different `<Col>`s. And for the label, instead of using the `<ControlLabel>` component, we need to set the `componentClass` property of the `<Col>` to `ControlLabel` to have the intended effect of right-aligning the label.
  - For other `<FormControl>`s, we can specify our own component classes using the `componentClass` property as well.
  - Since the buttons don’t need a label, we can specify an offset to where the column starts.
  - Bootstrap’s form controls support displaying invalid input fields in a distinctive manner, using the `validationState` property of `FormGroup`.
- Validation Alerts
  - Bootstrap provides nicely styled alerts via the `Alert` component.
  - The `Alert` component has different styles for the message like `danger` and `warning`
  - The `Alert` component also has the ability to show a Close icon, by taking in a callback named `onDismiss`. This callback is called when the user clicks the Close icon.
- Toasts
  - In this section, we'll look at result messages and informational alerts, that is, the reporting of successes and failures of an operation.
  - Create a new custom component named `Toast`.
    - The visibility will be controlled by the parent, which passes an `onDismiss` property that can be called to dismiss it.
    - In addition to the Close icon’s click, there will also be a timer that calls this `onDismiss` callback when it expires.
  - In the `render()` method, first add an `Alert` with the required attributes, all of which are passed in from the parent as props.
  - To position the alert message close to the bottom-left corner of the window, we can enclose the alert within a `<div>` that is absolutely positioned using `position: fixed`.
  - To show and hide the alert, we’ll use React-Bootstrap’s `Collapse` component, which takes in a property called `in`. When `in` is set to `true`, the child element shows (fades in) and when is set to `false`, it hides (fades out).
  - To set up an automatic dismiss after five seconds, we can expect a `componentDidUpdate()` call whenever the Toast is being shown. Within this lifecycle method, add a timer and call `onDismiss` on its expiry.
  - Make changes to all the components that need to show a success or error message.
- Modals
  - In this section, we’ll replace the in-page IssueAdd component with a modal dialog that is launched by clicking the Create Issue navigation item in the header.
  - When a modal is rendered, it is rendered outside the main `<div>` of the DOM that holds the page, which can be placed anywhere in the component hierarchy. In order to launch or dismiss the modal, the Create Issue navigation item is the controlling component.
  - Create a new component `IssueAddNavItem` that is self-contained: it displays the navigation item, launches the dialog and controls its visibility, creates the issue, and routes to the Issue Edit page on a successful creation.
    - Move the `NavItem` for Create Issue from the navigation bar to this new component.
    - Add an `onClick()` handler that shows the modal dialog by calling a method `showModal()`.
    - Since the `Modal` component can be placed anywhere, add it right after the `NavItem`.
    - `Modal` requires two properties: `showing` controls the visibility of the modal dialog, and an `onHide()` handler will be called when the user dismisses the dialog.
    - Within the `Modal` component, use `Modal.Header` to show the title of the modal. Add a vertical form (the default) with two fields, Title and Owner in `Modal.Body`. Use `Modal.Footer` to show a button toolbar with a Submit and a Cancel button.
    - When the user clicks on Submit, call `handleSubmit()` and close the Modal dialog. On success, we’ll show the Issue Edit page by pushing the Edit page’s link to the history. When the user clicks on Cancel, hide the modal dialog.
  - In `Page.jsx`, replace `NavItem` with this new component.
  - In `IssueList.jsx`, remove the rendering of `IssueAdd` and the `createIssue` function.

</details>

## Chapter 10

In this chapter, I learned how to deal with forms in React, and started to take in a lot more user input with the help of forms. I converted the hard-coded filter to something more flexible, and filled in the Edit page. I also completed the CRUD paradigm by adding the Update and Delete operation for issues. Another important thing is that I created specialized input components for different data types.

![ch10-1](/readme-images/ch10-1.png)
![ch10-2](/readme-images/ch10-2.png)

### notes

<details>
<summary>click for details</summary>

- Controlled Components
  - To show a value in the form input, the component has to be controlled by the parent via a state variable or props variable.
  - In `issueFilter.jsx`, set the status filter as a controlled component, using `URLSearchParams` to extract its current value during `render()`. After this change, a refresh will show the current value of the filter rather than the default value `All`.
- Controlled Components in Forms
  - In this section, we'll implement a form for the filter to let users make changes and then apply them all using an Apply button. We'll also add a Reset button that gives users an option to reset the filter to the original one.
  - Add an Apply button with an apply handler.
  - Remove all code in the `onChangeStatus()` (which will be part of the method `applyFilter()` later).
  - Create a state variable to store the value of the input that can be updated to reflect the new value in the dropdown.
    - Initialize the state variable to the URL value in the constructor.
    - Use this state variable as the value of the dropdown input during `render()`.
    - In `onChangeStatus()`, use `setState()` to  set the state variable to the new value, which is supplied as part of the event argument as `event.target.value`.
    - Now, the input value can be accessed via `this.state.status`.
  - Implement method `applyFilter()`, which accesses the current value and uses the history to push the new status filter. Bind this new method to `this` in the constructor.
  - To make the new filter reflect when the link is clicked, we need a lifecycle method `componentDidUpdate` that tells us that a property has changed and show the filter again.
  - Add a Reset button. To enable the button only when there are changes, we need a state variable called `changed`, which will be set to `true` within `onChange`, and to `false` in the constructor and when the filter is reset.
  - Implement method `showOriginalFilter()`, and bind it to `this` in the constructor.
- More Filters
  - In this section, we'll add a filter on the Effort field. We’ll need two fields, a minimum and a maximum value to filter on, both of which are optional.
  - Change the API to implement this filter.
    - Change the schema to add two more arguments to the `issueList` API, both integers, called `effortMin` and `effortMax`.
    - In `issue.js`, create a `effort` property of the MongoDB filter and set the `$gte` and `$lte` options.
  - Change the UI to add two inputs for the effort filter.
    - In `IssueList.jsx`: get two extra filter parameters from the URL's search parameters, and use them in a modified GraphQL query to fetch the list of issues. At this point, we’ll be able to test these changes by typing the filter parameters in the URL.
    - In `IssueFilter.jsx`:
      - Add two state variables for the inputs of the new filter in both constructor and `showOriginalFilter()`;
      - Add input fields for these variables in the filter form
      - Add two `onChange` handler for two fields and bind these methods to `this` in the constructor. Remember to check if the text can be converted to a number in the handler.
      - In `applyFilter()`, use the state variables to set the new location in the history. Since there are more variables, let’s use `URLSearchParams` to construct the query string rather than use plain string templates.
- Edit Form
  - In this section, we'll create a complete form for the Edit page in `IssueEdit.jsx`, and also a Submit button. But we will not handle the submission of the form yet.
  - Define the state of the component in the constructor, which is initiated with an empty object for the issue.
  - Replace the empty issue with the issue fetched from the server in a method called `loadData()`.
    - Use the `issue` API to load the data asynchronously.
    - Since all input fields’ contents are strings, we can’t use the result of the API call directly.Convert the natural data types of the fields of issue into strings. Further, add a `null` check for all the optional fields and use the empty string as the value.
    - If the API call failed (indicated by data being `null`), we’ll just use an empty issue in the state.
  - Implement the `render()` method.
    - There are two special cases: when the data has not been fetched yet or the given ID doesn’t exist, we'll have the issue object as an empty one. To handle these two cases, let’s check for the presence of the `id` field in the issue object and avoid rendering the form.
    - If these two conditions did not match, we can render the form. The form will need a Submit button and a submit handler, which we’ll name `handleSubmit()`.
    - For the filter form, we need a `onChange` event handler for each of the inputs. To reduce code duplication, we'll use a common `onChange` event handler for all inputs, taking advantage of the fact that the event’s target has a property called `name`, which will reflect the name of the input field in the form.
  - In the `onChange()` method, get the name of the field from the event’s target, and set the new state. Note that the recommended way is to supply a callback to the `setState` method that takes in the previous state and returns a new state.
  - Add lifecycle methods `componentDidMount()` and `componentDidUpdate()` to load the data.
  - In `handleSubmit()`, let’s just display the contents of the issue on the console for now.
- Specialized Input Component
  - Ideally, we want the form’s state to store the fields in their natural data types. We also want all of the data type conversion routines to be shared.
  - A good solution is to make reusable UI components for the non-string inputs.
    - In all these components, we’ll take the approach of a disjoint state.
    - When the user is not editing the component, the component is controlled and its only function is to display the current value.
    - When the user starts editing, we’ll make it an uncontrolled component. In this state, the value in the parent will not be updated, and the two values (current and edited) will be disjointed. Once the user has finished editing, the two values will be brought back in sync, provided the value is valid.
- Number Input
  - In this section, we'll create a specialized input component for number inputs. We’ll use this for the effort field in the Edit page in place of a plain `<input>` element.
  - Create a new file called `NumInput.jsx` under `ui/src/` directory.
    - Define the conversion functions `format()` and `unformat()` that take in a string and convert to a number and vice versa.
    - In the constructor of the component, set a state variable (which we will use as the value for the `<input>` element) after converting the value passed in as props to a string.
    - In the `onChange()` handler, check for the input containing valid digits and set the state if it is, as we did in the filter form.
    - In the `onBlur()` handler, call the parent’s `onChange()` when the input loses focus. While calling the parent’s `onChange()`, pass the value in the natural data type as a second argument.
    - In the `render()` method, render an `<input>` element with the value set to the state variable and the `onChange()` and `onBlur()` handlers. Further, copy over all other properties that the parent may want to supply as part of props for the actual `<input>` element.
  - In `IssueEdit.jsx`, use this new UI component.
    - Replace `<input>` with `<NumInput>`
    - In the `onChange()` handler, include the value in the natural data type that the component may send us as the second argument.
    - After these changes, if test the application, we'll find that the value in the effort field doesn’t change when we use Next/Prev buttons. To solve this problem, we need to construct the component again with a new initial property. The best way to do this is to assign a `key` property to the component that changes when a new issue is loaded.
    - Remove the replacement of `null` to an empty string for the effort field, as this will be handled by `NumInput`.
  - Now, we should be able to edit the effort field and see that the value changes according to the issue object when you click on Next/Prev. Also, when clicking Submit, the value of effort displayed on the console is a number.
- Date Input
  - Create a new file called `DateInput.jsx` under `ui/src/` directory.
    - For a date, the validity can be determined only when the user has finished typing. Losing focus from the input element can be used as the signal. So, in the `onBlur()` handler, we’ll check for validity of the date, then inform the parent of the new validity use a new optional callback named `onValidityChange()`. We'll also save the focused state and the validity in new state variables called `focused` and `valid`.
    - In `unformat()`, convert string to a date object. For an empty string, which is allowed to be a valid input, we’ll use the `Date(string)` constructor.
    - Rather than have a single `format()` function as in `NumInput`, we’ll have two separate functions for the display format (using `toDateString()`) and the editable format (`YYYY-MM-DD`) of the date.
    - In `onChange()`, check for valid characters and set the state if valid.
    - In `render()` method, display the user typed-in value if it is invalid, or if the user is editing it. Otherwise, display the display format or the editable format converted from the original value.
  - In `IssueEdit.jsx`, change the `due` field to a `DateInput` component.
    - Add a new method `onValidityChange()` to store the validity status of each input in a state variable called `invalidFields`.
    - In the constructor and `loadData()`, initialize `invalidFields` to an empty object.
    - In `render()`, add a new variable which is used to display a message showing the presence of any invalid fields. We’ll initialize this message only if there are any invalid fields (available via `invalidFields`). Use a class called `error` to emphasize error messages.
  - In `index.html`, make some changes in the stylesheet to show error messages in a highlighted manner.
  - Now, if we enter a valid date and click Submit, we'll see the actual date object being stored and displayed in the console. If we enter an invalid date, there will be a red bordered input as well as an error message in red.
- Text Input
  - It's convenient to let a component handle `null` values for text inputs.
  - Create a `TextInput` component with some differences.
    - The `format()` and `unformat()` exists simply for converting to and from null values.
    - In the `onChange()` method, any input will be allowed.
    - In the `render()` method, to handle variations in the HTML element name, instead of hard-coding the element tag, we pass it in as an optional `tag` property default to `input`. Since the tag name is a varialbe, We'll have to use the `React.createElement()` method rather than JSX.
  - In `IssueEdit.jsx`, replace all textual input elements to `TextInput`.
- Update API
  - In this section, we'll implement an Update API for saving the edited issue to the database.
  - Change the schema to reflect this new API.
    - Add a new input data type called `IssueUpdateInputs` with all possible fields that can be changed, and all of them optional.
    - Add a new mutation entry point called `updateIssue`, which will return the new modified issue.
  - In `issue.js`, implement the resolver for this API called `update()`.
    - Validate the issue based on the new inputs when the fields that affect the validity are changing: Title, Status, or Owner.
    - Once the validation succeeds, use the `updateOne()` function with the `$set` operation to save the changes.
    - Export the `update()` function in `module.exports`.
  - In `api_handler.js`, connect the API to its resolver.
- Updating an Issue
  - With the Update API, we can write the `handleSubmit()` method to make a call to the API and save the changes.
    - When using the named query, the query variable called `changes` need to exclude the fields that cannot be changed (`id` and `created`). This can be done using the `rest` operator `...`.
    - After the object is saved using the GraphQL API with the named query, replace the current issue with the returned issue, which requires a `setState()` call with the returned issue.
    - Show an alert message to indicate success of the operation.
    - Return without doing anything if there are invalid fields in the form.
- Updating a Field
  - In this section, we'll use the same API to close an issue in a quick way directly from the Issue List page, by setting its status to `Closed`.
  - In `IssueTable.jsx`:
    - Add a button in every row as part of the Actions column.
    - On click of this button, we’ll initiate a close action, which can be a function passed in as a callback in the props. The callback needs to be passed from `IssueList` via `IssueTable` to `IssueRow`.
    - To identify which issue to close, we have to receive the index of the issue as another value in the props, which can be computed while iterating over the list of issues.
  - In `IssueList.jsx`, implement the `closeIssue()` method.
    - Use a named query called `closeIssue` that takes in an issue ID as a query variable.
    - Call the `issueUpdate` API similar to the regular `update` call, but with the changes hard-coded to setting the status to closed.
    - Since the state is immutable, we have to make a copy of the `issue` state variable. As recommended, we'll use a callback for `this.setState()` that takes in the previous state. If the execution is unsuccessful, just reload the entire data.
- Delete API
  - In the remaining part of this chapter, we'll implement the delete operation to complete all set of CRUD.
  - Modify the schema to include the Delete API, which just takes the ID of the field to be deleted. We’ll return a Boolean value to indicate successful deletion.
  - In `issue.js`, implement the Delete API’s resolver. Note that `delete` is a reserved keyword in JavaScript, so we’ll name the function `remove()` but export it with the name `delete`.
    - Rather than just deleting the record for the given ID, we move it to trash so that we have a chance to recover it.
    - We'll use a new collection called `deleted_issues` to store all deleted issues, and add a `deleted` field to save the date and time of deletion.
    - So the process will be: retrieve the issue based on the given ID from the `issues` collection -> add the `deleted` field -> save it to `deleted_issues` -> delete it from the `issues` collection.
  - In `issue_handler.js`, connect the API to its resolver.
  - Initialize the `deleted_issues` collection in `init.mongo.js`.
- Deleting an Issue
  - The UI changes for deleting an issue is similar to what we did for closing an issue.
  - In `IssueTable.jsx`, add a button and pass the necessary callbacks through `IssueTable` to `IssueRows`.
  - In `IssueList.jsx`, implement the `closeIssue()` method.

</details>

### troubleshooting

- Page 272 Listing 10-2 (also other lists in this chapter): a pair of single quotes `''` is repeatly mistyped as a single double quotes `“`.
- Page 285 Listing 10-7: similar issue to chapter 9, line `const data = await graphQLFetch(query, { id });` should be `const data = await graphQLFetch(query, { id: parseInt(id, 10) });`.

## Chapter 9

In this chapter, I learned how to use React Router to implement client-side routing. I added a report page to the Issue Tracker application, and created a navigation bar so that the user can navigate between different views. I also learned how to connect the URL in the browser with what is shown in the page, and how parameters and query strings can be used to tweak the page contents. Using these facilities, I implemented a filter to display certain issues based on the status field, and added features that let users to edit an issue or explore the detail of an issue.

![ch09](/readme-images/ch09.png)

### notes

<details>
<summary>click for details</summary>

- Simple Routing
  - To navigate between different views of the application, routing is needed. Routing links the state of the page to the URL in the browser.
  - In order to affect routing, any page needs to be connected to something that the browser recognizes. In general, for SPAs, there are two ways to make this connection: hash-based and browser history.
  - In this section, we’ll create two views, one for the issue list and another for a report section. We'll split the main page into two sections: a header section containing a navigation bar with hyperlinks to different views, and a contents section which will switch between the two views depending on the hyperlink selected. The navigation bar will remain regardless of the view that is shown. We’ll also ensure that the home page redirects to the issue list.
    - Install React Router (`react-router-dom`).
    - Create a placeholder component for the report view in a new file `IssueReport.jsx`.
    - Create a component for the contents in a new file `Contents.jsx`. This component will be responsible for switching between views.
      - React Router provides a component called `Route` to achieve routing between different components based on the hyperlink that is clicked. It takes in a property `path` specifying the route needs to match, and a property `component` specifying what needs to be shown when the path matches the URL in the browser.
      - To redirect the home page to the issue list, we can further add a `Redirect` component.
      - Add a message that is shown when none of the routes match. When the property `path` is not specified, it implies that it matches any path.
      - All routes need to be enclosed in a wrapper component, which can be just a `<div>`. But to indicate that only one of these components needs to be shown, they should be enclosed in a `<Switch>` component.
      - Note that the match is a prefix match, and only the first match’s component will be rendered. Therefore, the order of routes is also important.
    - Create a component for the page that shows the navigation bar and the contents component in a new file `Page.jsx`.
      - As for the navigation bar, we are going to use the `HashRouter`. For example, to match the `/issues` path, the URL will be `/#/issues`, where the first `/` is the one and only page of the SPA, `#` is the delimiter for the anchor, and `/issues` is the path of the route.
    - In `App.jsx`, render the `Page` component instead of the original `IssueList` component into the DOM. Further, the page itself needs to be wrapped around a router, as all routing functionality must be within this for routing to work.
    - Now, if we navigate to <localhost:8000>, the URL of the browser changes automatically to <http://localhost:8000/#/issues>. This is because of the `Redirect` route.
- Route Parameters
  - One way of supplying parameters to the component is to add variables as route parameters.
  - Specifying parameters in the route path is similar to that in Express, using the `:` character followed by the name of the property that will receive the value.
  - In this section, we'll use this facility to show a page that lets us edit an issue. For now, we’ll create a placeholder in a new file `IssueEdit.jsx`. Later, we’ll make an Ajax call and fetch the details of the issue and show them as a form for the user to make changes and save.
    - In `Contents.jsx`, add a `Route` component that matches the path `/edit/:id`.
    - In `IssueEdit.jsx`, create the placeholder `Edit` component. We can access the trailing portion of the URL’s path containing the `id` via `match.params.id`.
    - In `IssueTable.jsx`, create a hyperlink in every row that navigates to the edit page.
- Query Parameters
  - The other way of supplying parameters to the component is using the URL’s query string. It's useful for cases where the variables are many and not necessarily in some order.
  - In this section, we'll use this facility implement a simple filter based on the status field.
    - In `schema.graphql`, add a parameter called `status` to the `issueList` query, of type `StatusType`.
    - In `issue.js`, accept this new argument in the function `list()`. Since the argument is optional, we’ll conditionally add a filter on status and pass it to the collection’s `find()` method.
    - In `IssueFilter.jsx`, replace the placeholder for the filter with three hyperlinks: All Issues, New Issues, and Assigned Issues.
    - The query string needs to be handled by the `IssueList` component as part of the `loadData()` function.
    - In the `IssueList` component, implement a lifecycle method `componentDidUpdate()` to reload the data when there's a change in the query string.
- Links
  - Until now, we used `hrefs` to create hyperlinks. React Router provides a better and more convenient way to create links via the `Link` component.
    - The paths in a `Link` are always absolute.
    - The query string and the hash can be supplied as separate properties to the `Link`.
    - `NavLink` is a variation of `Link` that can figure out if the current URL matches the link and adds a class to the link to show it as active.
    - A `Link` works the same between different kinds of routers.
  - In this section, we'll change all `hrefs` to `Links` in order to take advantage of these properties.
    - The `Link` component takes one property, `to`, which can be a string (for simple targets) or an object (for targets with query strings, etc.).
    - In the `IssueRow` component of `IssueTable.jsx`, the target is a simple string. Change the hyperlink element from `<a>` to `<Link>` and the property `href` to `to`, while maintaining the same string as the target.
    - In `IssueFilter.jsx`, the target is an object containing the pathname as `/issues`, and the query string as `?status=New`.
    - As for the navigation bar in `Page.jsx`, we'll use a `NavLink`, which allows us to highlight the currently active navigation link.
    - `NavLink` only adds a class called `active` when the link matches the URL. In order to change the look of active links, we need to define a style for the class in `index.html`.
- Programmatic Navigation
  - Query strings are also typically used for HTML forms. A form requires the query string to be constructed dynamically, as opposed to a predetermined string that we used in the `Link`s.
  - In this section, we’ll add a simple dropdown and set the query string based on the value of the dropdown. All changes will be made in `IssueFilter.jsx`: when the dropdown value changes, it changes the URL’s query string, which in turn applies the filter.
    - First, create this simple dropdown and replace the links with it.
    - Next, trap the event when the dropdown value is changed using the `onChange` property and set this property to a method called `onChangeStatus()`.
    - Implement `onChangeStatus()` to push the new location based on the changed filter.
- Nested Routes
  - For nested routes, the beginning part of the path depicts one section of a page, and based on interaction within that page, the latter part of the path depicts variation, or further definition of what’s shown additionally in the page.
  - In this section, we'll add a description field to an issue. On selecting an issue, the lower half of the page will show the description of that issue. We'll have `/issues` showing a list of issues with no detail, and `/issues/1` showing the the description of issue with ID 1, in addition to the issue list.
    - In `schema.graphql`, add a `description` field both in the type `Issue` and the type `IssueInputs`. We also need to add a new API called `issue` that takes in an integer as an argument to specify the ID of the issue to fetch.
    - In `issue.js`, implement the API to get a single issue. Create a new function `get()` that takes the `id` argument and call `findOne()` on the `issues` collection. Export it along with the other exported functions.
    - In `api_handler.js`, tie up the new function in the resolvers we supply to the Apollo Server.
    - Modify the schema initializer script (`init.mongo.js`) to add a description field to the initial set of issues.
    - Create a new file `IssueDetail.jsx` and implement the `IssueDetail` component.
    - In `IssueList.jsx`, add a `Route` with the actual component being `IssueDetail` after the `IssueAdd` section, whose path is of the form `/issues/<id>`. Instead of hardcoding `/issues`, use `this.props.match.path`, which is the path as matched in the parent component.
    - In `IssueTable.jsx`, create another link beside the Edit link to select an issue.
- Browser History Router
  - Hash-based routing is easy to understand and implement, but the downside of it is when the server needs to respond differently to different URL paths (like support responses to search engine crawlers).
  - To make our application search engine friendly, we’ll switch over to the browser history based router.
    - It can be simply done by changing the import statement and using `BrowserRouter` instead of `HashRouter`. After that, if we navigate to <localhost:8000>, the URL will be without a `#`, like <http://localhost:8000/issues>.
    - However, a refresh on any of the views will fail. That’s because the URL is currently pointing to `/issues` and the browser makes a request to the server for `/issues`, which is not handled by the UI server.
    - To address this, make a change to the UI server, which returns `index.html` for any URL that is not handled otherwise.
    - There is still one change left: change `webpack.config.js` to set the `publicPath` configuration.

</details>

### troubleshooting

- Page 256 Listing 9-21: in the `loadData()` function, line `const data = await graphQLFetch(query, { id });` should be `const data = await graphQLFetch(query, { id: parseInt(id, 10) });`. Otherwise, there will be a `INTERNAL_SERVER_ERROR` when displaying the detail of an issue, becuase an integer is expected but a numerical string is received.

## Chapter 8

In this chapter, I modularized the code by splitting the main file into multiple reusable pieces in both back-end and front-end. For the front-end code, I used Webpack to resolve dependencies, not only for the application’s own dependent modules, but also for third-party libraries, by "packing" these files into a few bundles. Apart from that, I learned how Webpack’s HMR increased productivity by injecting code into the browser incrementally, and refreshing the browser automatically on front-end code changes. Finally, I added support for debugging using source maps.

![ch08](/readme-images/ch08.png)

### notes

<details>
<summary>click for details</summary>

- Back-End Modules
  - In Node.js, there are two key elements to interact with the module system: `require` and `exports`.
  - The `require` element is a function that can be used to import symbols from another module. It takes in an ID as the argument, where the ID is the name of the module.
    - For packages installed using `npm`, this is the same as the name of the package.
    - For modules within the same application, the ID is the path of the file that needs to be imported.
  - The `exports` element defines the symbols that a file or module export. These symbols are set in a global variable called `module.exports` within that file, and that is the one that will be returned by the function call to `require()`.
  - Separate contents in `server.js`
    - Move function `GraphQLDate()` to `graphql_date.js`.
    - Move functions relating to the about message to `about.js`
    - Move all database-related code to `db.js`.
    - Move functions related to the Issue object to `issue.js`. This file need the DB connection from `db.js`.
    - Move contents dealing with Apollo Server, the schema, and the resolvers to `api_handler.js`. For the actual resolver implementations, we’ll import them from three files — `graphql_date.js`, `about.js`, and `issue.js`.
    - what’s left in `server.js` is only the instantiation of the application, applying the Apollo Server middleware and then starting the server.
- Front-End Modules and Webpack
  - Traditionally, the approach to spliting client-side JavaScript code was to use multiple files and include them using `<script>` tags in the main HTML file. This can be hard to manage because the dependency is done by maintaining a certain order of files in the HTML file.
  - With tools such as Webpack and Browserify, dependencies can be defined using statements equivalent to `require()` that we used in Node.js. These tools automatically determines the dependencies of application’s modules as well as third-party libraries, and put these files together into one or just a few bundles of pure JavaScript that can be included in the HTML file.
    - Install Webpack via `npm install --save-dev webpack@4 webpack-cli@3`. We use the option `--save-dev` because the UI server in production has no need for Webpack.
    - Split `App.jsx` into two by taking out the function `graphQLFetch`.
      - Note that for the front-end code, we'll use the newer ES2015-style `import` keyword rather than `require`.
      - Change `.eslintrc` to make an exception that always include extensions in import statements except for packages installed via `npm`.
    - Run `npm run watch` or `npm run compile` to compile all source code with Babel.
    - Pack the `App.js` file and create a bundle called `app.bundle.js` with command `npx webpack public/App.js --output public/app.bundle.js --mode development`. (Note that Webpack cannot handle JSX natively)
    - Replace `App.js` in `index.html` with `app.bundle.js` because the new bundle is the one that contains all the required code.
- Transform and Bundle
  - In the previous section, we had to manually Babel transform the files first, and then put them together in a bundle using Webpack. Webpack is capable of combining these two steps with the help of *loaders*.
  - Install loaders via `npm install --save-dev babel-loader@8`.
  - Create a configuration file for Webpack called `webpack.config.js`, which includes a `module.exports` variable that exports the properties that specify the transformation and bundling process.
    - Set `mode` property to `development` as we did in the command line previously.
    - The `entry` property specifies the file from which all dependencies can be determined. In the Issue Tracker application, this file is `App.jsx`.
    - The `output` property needs to be an object with the `filename` and `path` as two properties. The path has to be an absolute path. The recommended way to supply an absolute path is using the path module and the `path.resolve` function.
    - Loaders are specified under the property `module`, which contains a series of rules as an array. Each rule at the minimum has a `test`, which is a regex to match files and a `use`, which specifies the loader to use on finding a match.
  - Now simply run `npx webpack` without any parameters and see that `app.bundle.js` is created without creating any intermediate files. (To ensure this is hapenning, delete `App.js` and `graphQLFetch.js` in the `public` directory first.)
  - Like the `--watch` option for Babel, Webpack too comes with a `--watch` option, which incrementally builds the bundle, transforming only the changed files.
  - Change the npm scripts for `compile` and `watch` to use the Webpack command instead of the Babel command.
  - Now, we are ready to split the `App.jsx` file and place each React component in its own file. It's recommended to do so especially for stateful components. Stateless components can be combined with other components.
- Libraries Bundle
  - In this section, we’ll use Webpack to create a bundle that includes all third-party libraries rather than rely on CDN services.
  - Install all client-side libraries, which is the same list as the list of `<script>`s in `index.html`.
  - Import these libraries in all the client-side files where they are needed.
  - Now after compilation, the size of `app.bundle.js` will increase from a few KBs to more than 1MB, because it includes all of the libraries. A better alternative is to have two bundles, one for the application code (`app.bundle.js`) and another for all the libraries (`vendor.bundle.js`). We can easily do this in Webpack using an optimization called `splitChunks`.
  - In `index.html`, remove libraries loaded directly from the CDN, and include the new script `vendor.bundle.js`.
- Hot Module Replacement
  - Webpack has a powerful feature called Hot Module Replacement (HMR) that can reduce inconvenience caused by `npm run watch`. It changes modules in the browser while the application is running, removing the need for a refresh altogether.
  - Install `webpack-dev-middleware` and `webpack-hot-middleware`.
  - Import these modules and mount them in the Express server as middleware, but only when explicitly enabled because we don’t want to do this in production.
    - To let us modify the configuration on the fly when HMR is enabled, we need to change the `entry` in `webpack.config.js` to an array so that a new entry point can be pushed easily.
    - In `uiserver.js`: (1) add an option for the Express server to enable HMR using an environment variable called `ENABLE_HMR`; (2) import Webpack, two modules and the configuration file; (3) push a new entry point for Webpack; (4) enable the plugin for HMR; (5)  create a Webpack compiler and create the `dev` middleware and the `hot` middleware.
  - Now, there are different ways to start the UI server. Add these as comments in `package.json` (use properties prefixed with `#`).
  - In `App.jsx`, accept all changes using the `HotModuleReplacementPlugin`’s `accept()` method.
- Debugging
  - Compiling files and creating bundles make debugging hard. Webpack solves this problem by its ability to give us source maps.
  - In the Webpack configuration, enable the source map with a parameter called `devtool`.
  - Now, when look at the browser’s Developer Console, we are able to see the original source code and be able to place breakpoints in it.
  - We can install the React Development Tools add-on, which provides the ability to see the React components in a hierarchical manner like the DOM inspector.
- Production Optimization
  - There are two things that need special attention for Webpack.
  - The first thing to be concerned about is the bundle size.
    - For applications that are frequently used by users such as the Issue Tracker application, the bundle size is not of much concern because it will be cached by the user’s browser.
    - But for applications where there are many infrequent users, it’s important to not just split the bundles into smaller pieces, but also to load the bundles only when required using a strategy called *lazy loading*.
  - The other thing to be concerned about is browser caching, especially when you don’t want it to cache the JavaScript bundle. This happens when the application code has changed and the version in the user’s browser’s cache is the wrong one.
    - Most modern browsers handle this quite well, by checking with the server if the bundle has changed.
    - But older browsers, especially Internet Explorer, aggressively cache script files. The only way around this is to change the name of the script file if the contents have changed.

</details>

## Chapter 7

In this chapter, I made a big architectural change by separating the UI and the API servers. Besides that, to keep the variables flexible, I replaced all hard-coding with environment variables with the help of a package called `dotenv`, which enables the same code to run on different environments using different configurations. Finally, I used ESLint to verify that the code I wrote followed standards and good practices. This would allow us to catch possible bugs earlier in the testing cycle.

![ch07-1](/readme-images/ch07-1.png)
![ch07-2](/readme-images/ch07-2.png)

### notes

<details>
<summary>click for details</summary>

- UI Server
  - Now, the one and only Express server not only serves static content, but also serves API calls. It's better to separate the two functions into two servers - the UI server and the API server.
  - Two servers can be physically different computers, but for development purposes, we’ll run them on the same computer but on different ports.
  - Two servers will use two different Node.js processes, each with its own instance of Express.
  - The API Server: responsible for handling only the API requests.
    - It responds only to URLs matching `/graphql` in the path.
    - Thus, the Apollo server middleware and its requests to the MongoDB database will be the only middleware in this server.
  - The UI Server: contains only the static middleware.
    - In the future, when we introduce server rendering, this server will be responsible for generating HTML pages by calling the API server’s APIs to fetch the necessary data.
    - For the moment, it only serves all static content, which consists of `index.html` and the JavaScript bundle that contains all the React code.
- Multiple Envirionment: remove hard-coding such as the port numbers and MongoDB URL
  - To keep the variables flexible, a typical way is via environment variables.
  - We'll use a package called `dotenv` that help us convert variables stored in a file into environment variables. This package looks for a file called `.env`.
  - In the `api` directory:
    - Install `dotenv` package.
    - `require` this package and immediately call `config()` on it.
    - Use `process.env` properties to let environment variable replace hard-coding.
    - Create a file called `.env`. It is recommended that this file not be checked into any repository, because each developer must specifically set the variables in the environment as per their needs.
    - Better to change the `nodemon` command line so that it watches for changes to this file.
  - In the `ui` directory:
    - Make similar changes to the hard-coded UI server port.
    - The API endpoint has to somehow get to the browser in the form of JavaScript code.
      - Generate a `env.js` file and inject it into `index.html`. This JavaScript file will contain a global variable with the contents of the environment.
      - Within the UI server, generate the contents of this script. It will initialize the global variable `ENV` with one or more properties that are set to environment variables.
      - Initialize a variable for the API endpoint, then create a route in the server that responds to a GET call for `env.js`.
- Proxy-Based Architecture
  - Cross-origin Resource Sharing (CORS)
    - The Same-origin policy exists to prevent malicious websites from gaining unauthorized access to the application.
    - The kind of requests that can be permitted is controlled by the Same-origin policy as well as parameters controlled by the server, which determines if the request can be allowed.
  - The Apollo GraphQL server, by default, allows unauthenticated requests across origins, so we have to disable the default behavior using an environment variable.
    - Add an environment variable `ENABLE_CORS` in `api/.env` and set it to `false`.
    - In `server.js`, look for this environment variable and set an option called `cors` to `true` or `false`, depending on this variable.
    - Now, we need some other mechanism to make API calls.
      - We’ll change the UI to make even API requests to the UI server, where we will install a proxy so that any request to `/graphql` is routed to the API server.
      - Such a proxy can be easily implemented using the `http-proxy-middleware` package.
      - After install this package in the `ui` directory, a proxy can be used as a middleware using `app.use()` mounted on the path `/graphql`.
      - The middleware can be created with just a single option: the target of the proxy, which is the base URL of the host where the requests have to be proxied.
- ESLint: a very flexible linter that lets you define the rules that you want to follow.
  - A *linter* checks for suspicious code that could be bugs. It can also check whether your code adheres to conventions and standards that you want to follow across your team to make the code predictably readable.
  - ESLint looks for a set of rules in the `.eslintrc` file, which is a JSON specification.
    - These are not the definition of rules, but a specification of which rules need to be enabled or disabled.
    - Rules in the configuration file are specified under the property `rules`, which is an object containing one or more rules, identified by the rule name, and the value being the error level.
    - The error levels are `off`, `warning`, and `error`.
    - Common types of error: stylish issues, best practices, possible errors.
  - Create the `.eslintrc` file in the `api` directory, then run `npx eslint .` to see all errors.
  - We can add an npm script in `package.json` that will lint all the files in the `api` directory.
  - Similarly, create the `.eslintrc` file in the `ui` directory as well as the `src` directory, then run `npx eslint . --ext js,jsx --ignore-pattern public` to see all errors.
    - Another way of ignoring patterns of files is to add them as lines to a text file called `.eslintignore`.
    - By default ESLint does not match files with the extension `jsx`. To include this extension, use the command line option `--ext`.
- React PropTypes
  - Similar to how strongly typed languages (like Java) specify the type of parameters as part of the function declaration, the properties being passed from one component to another can also be validated against a specification.
  - This specification is supplied in the form of a static object called `propTypes` in the class, with the name of the property as the key and the validator as the value.
  - The object `PropTypes` is available as a module called `prop-types`, which can be included in `index.html`.

</details>

### troubleshooting

- Page 174 Listing 7-1 `api/package.json`: the dependency of graghql should be `14.6.0` instead of `0.13.2`. Otherwise we will run into a `TypeError: Cannot read property 'filter' of undefined` when starting the api server.
- Page 190 Listing 7-17 `api/.eslintrc`: missing quotes around `rules`; better to leave quotes around `true`.
- Page 193 Listing 7-20 `api/server.js`: lines `errors.push('Field "title" must be at least 3 characters long.');` and `if (issue.status === 'Assigned' && !issue.owner) {` should be in function `issueValidate` instead of function `issueAdd`.
- Page 198 Listing 7-24 `ui/src/.eslintrc`: missing quotes around `rules`.
- Page 198 Listing 7-25 `ui/src/App.jsx`: `function IssueTable(props{ issue }) {` should be `function IssueTable({ issues }) {`.

## Chapter 6

In this chapter, I learned the basics of MongoDB, including the main features and the commonly used methods for CRUD operations. Then I modified the server side of the Issue Tracker application, using some of these methods to read and write to the MongoDB database. Now, instead of having an array of issues in the Express server’s memory, we made the issue list persistent in a real database.

![ch06](/readme-images/ch06.png)

### notes

<details>
<summary>click for details</summary>

- MongoDB Basics
  - Documents: MongoDB is a document database, which means that the equivalent of a record is a document, or an object. In a relational database, we organize data in terms of rows and columns, whereas in a document database, an entire object is written as a document.
    - A document is a data structure composed of field and value pairs, which is similar to a JSON object.
    - Compared to a JSON object, a MongoDB document has support not only for the primitive data types like Boolean, numbers, and strings, but also other common data types such as dates, timestamps, regular expressions, and binary data.
    - In relational databases, we typically need multiple tables to store nested objects and arrays. But in MongoDB, the values of fields can include objects and arrays, as deeply nested as you want it to be. MongoDB encourages denormalization, that is, storing related parts of a document as embedded subdocuments rather than as separate tables in a relational database.
  - Collections: a collection is like a table in a relational database, it is a set of documents.
    - A primary key is mandated in MongoDB, and it has the reserved field name `_id`. MongoDB uses a special data type called the `ObjectId` for the primary key.
    - MongoDB creates this field and auto-generates a unique key for every document.
    - MongoDB does not require to define a schema for a collection. In practice, though, all documents in a collection do have the same fields.
  - Databases: a database is a logical grouping of many collections.
  - Query Language: unlike SQL in a relational database, the MongoDB query language is made up of *methods* to achieve various operations.
    - The main methods for read and write operations are the CRUD methods. Other methods include aggregation, text search, and geospatial queries.
    - All methods operate on a collection and take parameters as JavaScript objects that specify the details of the operation.
  - The Mongo Shell: an interactive JavaScript shell.
    - `show databases`: show available databases.
    - `db`: identify the current database (default is called `test`).
    - `show collections`: see what collections exist in this database.
    - `use [name-of-database]`: switch to another database
    - `db.employees.insertOne({ name: { first: 'John', last: 'Doe' }, age: 44 })`: insert a new document in the `employees` collection.
      - A collection is referenced as a property of the global object `db`, with the same name as the collection.
      - To create a new collection, we creates one document in this collection using the `insertOne()` method.
    - `db.[name-of-collection].find().pretty()`: list all documents in the collection in a pretty format.
      - The result of the `find()` method is a cursor that can be iterated. In the `cursor` object, there is a `toArray()` that reads all documents from the query and places them in an array. Here's an example:
      - `let result = db.employees.find().toArray()`
      - `result.forEach((e) => print('First Name:', e.name.first))`
      - `result.forEach((e) => printjson(e.name))`
      - Note that mongo shell uses `print()` rather than `console.log()` like Node.js. This method can only print strings. Objects need to be converted to strings before printing, using the utility function `tojson()`. There is also another method, called `printjson()`, which can print objects as JSON.
    - `db.[name-of-collection].drop()`: clear the collection.
    - We can see the list of available methods by pressing `tab` twice after typing `db.[name-of-database].`. We can let the mongo shell auto-complete the name of any method by pressing `tab`.
- MongoDB CRUD Operations
  - Create
    - `insertOne()`: create one document.
    - `insertMany()`: create multiple documents in one go.
    - Because the `_id` field generated by MongoDB is not human-readable (it's of type `ObjectID`), we can use a new field like `id` to store a human-readable identifier like a number.
  - Read
    - `find()`: can take in two arguments, the first is a filter to apply to the list, and the second is a projection of which fields to retrieve.
    - `findOne()`: return a single object rather than a cursor.
    - Filter: an object where the property name is the field to filter on, and the value is its value that it needs to match, e.g. `{ id: 1, 'name.last': 'Doe' }`.
      - The filter is actually a shorthand for `{ id: { $eq: 1 } }`, where `$eq` is the operator. In the generic sense, the format of a single element in the filter is `fieldname: { operator: value }`. Other operators for comparison are available, such as `$gt` for greater than, `&gte` for greater than or equal to.
      - Note that we use the dot notation for specifying a field embedded in a nested document. And we use quotes around the field name since it is a regular JavaScript object property.
      - It's a good practice to create an index on a field using `createIndex()` method when filtering on that field is a common occurrence.
    - Projection: an object with field names as the keys and values as 0 or 1, to indicate exclusion or inclusion. We can either start with nothing and include fields using 1s, or start with everything and exclude fields using 0s. The `_id` field is always included unless specified as 0.
  - Update
    - `updateMany()`: takes in two arguments, a query filter, and an update specification if only some fields of the object need to be changed.
    - `updateOne()`: stop after finding and updating the first matching document. The primary key or any unique identifier is normally used in the filter.
    - Update Specification: an object with a series of `$set` properties whose values indicate another object, which specifies the field and its new value.
    - `replaceOne()`: replace the complete document.
    - Note that `_id` cannot be changed via an `updateOne()` or a `replaceOne()`.
  - Delete
    - The delete operation takes a filter and removes the document from the collection.
    - Like other operations, `deleteOne()` and `deleteMany()` are available.
  - Aggregate
    - `count()`: can take a filter and return the number of documents that match a certain criterion.
    - `aggregate()`: works in a pipeline, every stage takes the input from the result of the previous stage and operates on it.
      - The initial input to the pipeline is the entire collection.
      - The pipeline specification is an array of objects, each element being an object with one property that identifies the pipeline stage type and the value specifying the pipeline’s effect.
      - e.g. `db.employees.aggregate([ { $group: { _id: null, total_age: { $sum: '$age' } } } ])` gets the total age of all employees in the entire collection. `db.employees.aggregate([ { $group: { _id: null, count: { $sum: 1 } } } ])` peforms the same as `count()`.
      - To group the aggregate by a field, we need to specify the name of the field (prefixed by a `$`) as the value of `_id`. e.g. `db.employees.aggregate([ { $group: { _id: '$organization', average_age: { $avg: '$age' } } } ])`.
- MongoDB Node.js Driver
  - Install the driver via `npm install mongodb`
  - Make a connection to the database server
    - Import the object `MongoClient` from the driver.
    - Create a new client object using a URL that identifies a database to connect to. The URL should start with `mongodb://` followed by the hostname or the IP address of the server to connect to. The client constructor takes in another argument with more settings for the client.
    - Call `connect()` method on the client object. The `connect()` method is an asynchronous method. It takes a callback funtion with two arguments: an error and the result, where the result is the client object itself.
    - Within the callback, call `db()` on the client object to obtain a connection to the database. To get a handle to a collection, call `collection()` with the collection name on `db`.
  - Perform CRUD operations similar to mongo shell, except that methods are all asynchronous.
    - Besides the regular arguments, each method also takes a callback function that’s called when the operation completes.
  - When we are done, close the connection using `close()` method of the client object.
  - Put all of the above together in a single function so that we can handle errors.
    - Pass a callback to this function, if there are any errors, they can be passed to the callback.
    - For each error, we need to do the following: close the connection to the server, call the callback, return from the call so that no more operations are performed.
- Schema Initialization
  - The mongo shell is not only an interactive shell, but is also a scripting environment.
  - In this section, we'll create a schema initialization script called `init.mongo.js` within the `script` directory.
    - Clear existing issues it by calling a `remove()` with an empty filter (which will match all documents) on the `issues` collection.
    - Copy the array of issues from `server.js` and use it to initialize the `issue` collection.
    - Create a few indexes on useful field.
    - Run the script via `mongo issuetracker scripts/init.mongo.js`
- Reading from MongoDB: in this section, we'll change the List API to read from the MongoDB database rather than the in-memory array of issues in the server.
  - Declare a a global variable in `server.js` to store the connection to the database.
  - Write a function to connect to the database, which initializes this global variable.
  - Change the setup of the server to first connect to the database and then start the Express application.
  - Use the global variale in the resolver `issueList()` to retrieve a list of issues by calling the `find()` method on the issues collection.
  - Since the issues from the database now contain an `_id` in addition to the `id` field, we need to include it in the GraphQL schema of the type `Issue`.
- Writing to MongoDB: in this section, we'll change the Create API to use the MongoDB database.
  - Create a collection with the counter that holds a value for the latest Issue ID generated.
  - Initialize the counter’s value to the count of inserted documents.
  - Create a function in `server.js` to call `findOneAndUpdate()`, which takes the ID of the counter and returns the next sequence.
  - Use this function to generate a new ID field and set it in the `issue` object in the resolver `issueAdd()`.
  - Get rid of the in-memory array of issues in the server.

</details>

## Chapter 5

In this chapter, I spent most of my time integrating with the back-end. The biggest change is that I started fetching and storing the data using APIs from the Express and Node.js server, which replaced the hard-coded array of issues in the browser’s memory. Using GraphQL, I learned how to build the C and R part of CRUD. I also saw how easy some of the validations were to implement and how the strong type system of GraphQL helps avoid errors and makes the APIs self-documenting.

![ch05](/readme-images/ch05.png)

### notes

<details>
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

## Chapter 4

In this chapter, I learned how to use React state and how it can be manipulated on user interactions or other events. I added two text input fields and a button for the Issue Tracker to let users append rows to the initial list of issues. A click of the button will cause the state to change and the components to rerender. One important point during this process is that we use *props* to pass the state and the method around between a child and its parent.

![ch04](/readme-images/ch04.png)

### notes

<details>
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
  - For this project, we'll lift the state up on level to `IssueList`, so that information can be propagated down to `IssueAdd` and to `IssueTable`.
    - First, move the state and other methods that deal with the state to the `IssueList` class. After that, to provide `IssueTable` with access to the array of issues, we need to pass it from the state within `IssueList` to `IssueTable` via `props`. Then in the`IssueTable` class, instead of referring to the array of issues via `this.state`, we’ll need to get it from `props`.
    - Second, move the timer to the constructor of the `IssueAdd` class. At the same time, we have to make `createIssue()` method available in the `IssueAdd` class by passing the method as part of the `props`.
    - Last of all, before passing the method around, we have to bind the method to the `IssueList` component using `.bind()` since we're using arrow functions. It's recommended to do this in the constructor of the class where the method is implemented.
- Event Handling
  - In the book's project, we want to allow users to add an issue interactively on the click of a button.
    - First, create a form with two text inputs and a button used to trigger the event in the `render()` method of `IssueAdd`. At the same time, we can remove the timer that creates an issue from the constructor.
    - When clicking the button, we want it to call `createIssue()` using the input values, and we want to prevent the form from being submitted because we will handle the event ourselves. We’ll create a method called `handleSubmit()` to finish this task.
    - Rewrite the form declaration with a name and an `onSubmit` handler. Then, implement the method `handleSubmit()`. This method calls the `preventDefault()` on the event to prevent the form from being submitted, creates a new issue by calling `createIssue()`, and clears the text input fields.
    - Like the previous section, `handleSubmit()` needs to be bound to the `IssueAdd` class.
- Stateless Components
  - For components that have nothing but a `render()` method (like `IssueRow` and `IssueTable` in the book's project), it is recommended that they are written as functions rather than classes.
  - If a component does not depend on `props`, it can be written as a simple function whose name is the component name.
  - If the rendering depends on the `props` alone, the function can be written with one argument as the `props`.

</details>

## Chapter 3

In this chapter, I started to use React classes to instantiate components. I laid out the main page of the Issue Tracker  by generating components dynamically from data and composing them together. I also learned how to pass data from a parent component to its children, which allows us to reuse a component class and to make it to render differently with different data.

![ch03](/readme-images/ch03.png)

### notes

<details>
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

## Chapter 2

In this chapter, I learned to use React to render a simple page and use Node.js and Express to serve that page from a web server. I used nvm to install Node.js, then practiced with how to use npm not just to install node packages, but also to customize command-line instructions. I also got familiar with Babel, which allows us to transform JSX into pure JavaScript and to convert JavaScript into a backwards compatible version to support older browsers.

![ch02](/readme-images/ch02.png)

### notes

<details>
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
