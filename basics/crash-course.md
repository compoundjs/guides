## Crash Course to CompoundJS

This guide demonstrates how to create a simple compound app using generators.

Goal: to build a working application without any knowledge of the express framework, 
learn how to structure a compoundjs app, use the tools that come with the compoundjs 
framework, and get quick overview of its main features.

### What is compound

Before we start, let's take a look at the compound framework. Compound's formula is
[Express][express] + structure + extensions. Where **structure** is  the standard
layout of directories, and **extensions** are node modules adding functionality to
the framework. Compound's goal is to provide an obvious and well-organized interface for
express compatible application development. This means that everything that works with 
express will work with compound.

Now we can start building our first app. Let's create a todo-list app with a RESTful API and a web
interface.

### First steps: install compound and generate app

0. if not already installed, install nodejs (v 0.8.0+) - http://nodejs.org/download/.  If you have node installed you can test your version with:
    `node --version`

1. install compound using npm.  Doing this globally enables the `compound` command line tool.  You might need to sudo.

    `npm install compound -g`

2. generate the todo-list app using the `compound` command line.  This creates an application with the default compoundjs structure.

    `compound init todo-list-app`

3. install dependencies (see package.json for default dependencies.  They will be downloaded to the node_modules dir)

    `cd todo-list-app && npm install`
    

Now that we have the initial compound app structure, we can run your new compoundjs application. 

Let's run the  command `node .` and  then open 
[http://localhost:3000/](http://localhost:3000/) in a browser.

> **NOTE** the `node .` command is simple way to run the application in the current
> directory. You also can run `node server.js` or `coffee server.coffee` or even
> `compound server`. the result will be the same, we will learn the differences later.

After opening localhost:3000 in a browser we will see a welcome page with some links
and debug info (this appears after you click on the corresponding link). This is a static
file located `./public/index.html`. Everything in the `./public` dir will be
available to your app as static content so client-side javascripts and stylesheets should be saved here.

> **NOTE** some javascripts and stylesheets could be generated from coffee or
> sass / less / stylus. Sources for these files located at `./app/assets` and
> compiled automatically using `co-assets-compiler` extension module.

### Generate scaffolding for Lists

Run this command:

    compound generate scaffold list name

It will generate all necessary files for the `List` model. 

if you haven't already done so, stop the server using the `CTRL+C` hotkey and restart it using `node .` so that you can review the changes in your browser.

> In development mode every modification of an existing model, controller or view 
> file will be updated automatically, but when you modify routes or schema you
> have to restart the server manually. Alternatively use the `node-dev` command (`npm install
> node-dev` first) to restart server automatically on every file change.

Now, visit http://localhost:3000/lists to see the new version of your app.

Now that we have some files in our application, it's time to explore the structure. Let's get a 
brief overview of the files we have to get our new lists CRUD functionality (create-read-update-delete) working.

### Explore app structure

#### Router

Let's start with routes because this is the first place in the compound stack where
request handling happens. In other words, when you open http://localhost:3000/lists 
in your browser, the router should decide what part of the application should handle this 
request. Routes are configuration rules that explain to the application what paths your application 
can handle.

Routes are listed in file `config/routes.js` which look like:

```javascript
exports.routes = function (map) {
    map.resources('lists');
    map.all(':controller/:action');
    map.all(':controller/:action/:id');
};
```

The line `map.resources('lists');` was added by the list scaffolding generator. The command `compound routes` will generate a table that shows all the routes that are available to the app.  
Note how the line "map.resources('lists');" generates a collection of predefined routes.

```text
     lists GET    /lists.:format?          lists#index
     lists POST   /lists.:format?          lists#create
  new_list GET    /lists/new.:format?      lists#new
 edit_list GET    /lists/:id/edit.:format? lists#edit
      list DELETE /lists/:id.:format?      lists#destroy
      list PUT    /lists/:id.:format?      lists#update
      list GET    /lists/:id.:format?      lists#show
           ALL    /:controller/:action     undefined#undefined
           ALL    /:controller/:action/:id undefined#undefined
```

In this table the first column shows the _route helper_ name, the second describes the _method_ (aka _verb_) for the route, 
the third describes the _route_ itself, and the last column shows the _controller#action_ that handles 
the request.   As you can see, a route is a pattern that defines what a resource call will look like in a browser.  When a browser asks the server
for a route like /lists/1/ the router will test it against these patterns to determine how to handle it.

To make the process of including routes in pages easier, Compound includes router helpers. The helper name should be used to generate paths, and all route helpers are available 
as methods on the `pathTo` object. These next examples show the output of calling the list router helpers:

    pathTo.lists() // '/lists'
    pathTo.lists({format: 'json'}); // '/lists.json'
    pathTo.list(1); // '/lists/1'
    pathTo.list(1, {format: 'json'}); // '/lists/1.json'
    pathTo.list({id: 1, format: 'json'}); // '/lists/1.json'
    pathTo.new_list(); // '/lists/new'
    pathTo.edit_list(1); // '/lists/1/edit'
    pathTo.edit_list('my-list'); // '/lists/my-list/edit'

> By default path helpers are generated with underscores as word separators. This
> behavior will be changed in future versions and it's highly recommended to add
> the line map.camelCaseHelperNames = true; to the top of the router file to generate the names in
> camelCase: `newList` and `editList` instead of `new_list` and `edit_list`

#### Controller

When a route is matched request handling is passed to the corresponding controller#action.
Controllers are located in `./app/controllers`. For example, our list controller is
described in `./app/controllers/lists.js`.

> Please note that the name of the controller you've generated using the scaffolding
> generator ends with <code>_controller</code>. This is just a different format of
> a controller name.  What this alternate name means is that the controller code will be `eval`ed in
> `controllerContext`, this is called a 'eval-controller'. When the file doesn't end
> with _controller it is a 'noeval controller'. ControllerContext and the differences between eval and
> noeval controllers are explained in API docs for
> [controller](http://compoundjs.com/man/controller.3.html)

The controller consists of a set of actions and hooks. Both actions and hooks should handle
the request or call `next()` if the request could not be handled. If `next()` is
called, control will be returned to the router which will then try to match next route to
handle this request. Otherwise, if the action could not handle the request because of an error, 
you can call `next(err)` and then control will be passed to error handler express
middleware (skipping all routes).

Hooks allow you to prepare the environment before an action or do something additional
after an action. For example, load a list before the edit, update, show actions.  
This tool allows for DRYer controllers.

The most common results of an action are `render`, `send` and `redirect`. And of course
`next(err)` - this is most often used result of any action.

The most important thing to remember about controllers is that controllers should not contain any 
business logic. While you can validate your data, and do things to it before calling_create_ and 
_update_ on your model, all other business logic is better left to the model. Best practice is to do everything in model, 
keep your controllers skinny and use a proper ORM module which ships validation and hooks.

#### View

In a controller, an action may decide to render a view. All the views in a compound application
are located in `./app/views`. There is also a special directory for layouts:
`./app/views/layouts`. Every view or layout inside app/views could be rendered
using the `render` method within controllerContext. Note that you should omit the
extention when calling render(). The last rule is be lazy; do not specify the view name 
when it is the same as the action name. For example, from within lists_controller.js a call like:

    action('index', function() {
        render();
    });

Will render view the view 'lists/index' located in `./app/views/lists/index.ejs`
within the layout. The convention is that unless you are calling a view with a name
other than the action name, you can omit it.

##### Layouts

A layout is a view wrapping the target view. By convention, the layout called will
share the same name as the controller. A rendered view is passed into a layout as 
the `body` variable. To render a view without layout you can specify 
`{layout: false}` as a param of the `render` method, or just `this.layout = false`
inside controllerContext. You also can specify what layout you want to render
using `c.layout('name');` within controllerContext:

    action('landing', function(c) {
        c.layout('welcome');
        render();
    });

##Authors
Anatoliy Chakkaev
[Daniel Lochrie](https://github.com/dlochrie)

[express]: http://expressjs.com/
