## Crash Course to CompoundJS

This guide demonstrates how to create a simple compound app using generators.

Goal: to build a working application without any knowledge of the express framework, 
learn how to structure a compoundjs app, use the tools that come with the compoundjs 
framework, and get quick overview of its main features.

### What is compound

Before we start, let's take a look at the compound framework. Compound's formula:
[Express][express] + structure + extensions. Where **structure** is  the standard
layout of directories, and **extensions** are node modules adding functionality to
the framework. Compound's goal: to provide an obvious and well-organized interface for
express compatible application development. This means that everything that works with 
express will work with compound.

Now we can start building our first app. Let's create todo-list app with REST API and web
interface.

### First steps: install compound and generate app

1. install compound using npm  

    `npm install compound`

2. generate app  

    `compound init todo-list-app`

3. install dependencies  

    `cd todo-list-app && npm install`

Now that we have the initial compound app structure, we can run application and see
what will happen. Let's run `node .` command and open 
[http://localhost:3000/](http://localhost:3000/) in browser.

> **NOTE** `node .` command is simple way to run application in current
> directory. You also can run `node server.js` or `coffee server.coffee` or even
> `compound server`: result will be the same, we will learn differences later.

After opening localhost:3000 in browser we will see welcome page with some links
and debug info (appears after clicking on corresponding link). This is a static
file located `./public/index.html`. Everything in `./public` dir will be
available as static content: client-side javascripts and stylesheets located here.

> **NOTE** some javascripts and stylesheets could be generated from coffee or
> sass / less / stylus. Sources for these files located at `./app/assets` and
> compiled automatically using `co-assets-compiler` extension module.

### Generate scaffold for List

Run this command

    compound generate scaffold list name

It will generate all necessary files for the `List` model. Then stop server
using `CTRL+C` hotkey and run it again to hook up changes.

> In development mode every modification of existing model, controller or view 
> file will be updated automatically, but when you modify routes or schema you
> have to restart server manually or just use `node-dev` command (`npm install
> node-dev` first) to restart server automatically on every file change.

Then visit http://localhost:3000/lists to check how it works.

Now that we have some files in out application to explore its structure, let's get a 
brief overview of the files we have to get lists CRUD (create-read-update-delete) working.

### Explore app structure

#### Router

Let's start with routes, because this is first place in the compound stack where
request handling happens. In other words, when you open http://localhost:3000/lists 
in your browser, the router should decide what part of the application should handle this 
request. Routes are configuration rules explains to application what path your application 
can handle.

Routes listed in file `config/routes.js` which look like:

```javascript
exports.routes = function (map) {
    map.resources('lists');
    map.all(':controller/:action');
    map.all(':controller/:action/:id');
};
```

The line `map.resources('lists');` was added by list scaffold generator. We can see
which express routes were created by this line using `compound routes` command:

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

The first column shows the _route helper_ name, the second describes the _method_ (aka _verb_), 
the third describes the _route_, and the last column shows the _controller#action_ that handles 
the request. The helper name should be used to generate paths, and all route helpers are available 
as methods on `pathTo` object. Examples:

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
> line map.camelCaseHelperNames = true; to the top of file to generate names in
> camelCase: `newList` and `editList` instead of `new_list` and `edit_list`

#### Controller

When a route is matched, request handling is passed to the corresponding controller#action.
Controllers are located in `./app/controllers`. For example, our list controller is
described in `./app/controllers/lists.js`.

> Please note that the name of controller you've generated using the scaffold
> generator ends with <code>_controller</code>. This is just a different format of
> controller, which means that controller code will be `eval`ed in
> `controllerContext`, it is so-called 'eval-controller'. When the file doesn't end
> with _controller - it's 'noeval controller'. The differences between eval and
> noeval controllers are explained in API docs for
> [controller](http://compoundjs.com/man/controller.3.html)

The controller consists of set of actions and hooks. Both actions and hooks should handle
the request or call `next()`, if the request could not be handled. If `next()` is
called, control will be returned to the router which will try to match next route to
handle this request. Otherwise, if the action could not handle request because of an error, 
you can call `next(err)` and then control will be passed to error handler express
middleware (skipping all routes).

Hooks allows you to prepare the environment before an action or do something additional
after an action. For example, load a list before edit, update, show actions.

The most common results of action are: `render`, `send` and `redirect`. And of course
`next(err)` - this is most often used result of any action.

The most important thing to remember about controllers: controllers should not contain any 
business logic. While you can validate your model, and do things to it before _create_ and 
_update_, this logic is better left to the model. Best practice is to do everything in model, 
keep your controllers skinny, and use proper ORM, which ships validation and hooks.

#### View

In a controller, an action may decide to render a view. All views in a compound application
are located in `./app/views`. There is also a special directory for layouts:
`./app/views/layouts`. Every view or layout inside app/views could be rendered
using `render` method of controller context. One thing to note: you can omit the
extention when rendering views. And the last rule: be lazy, do not specify view name 
when it is the same as action name. Example (lists_controller.js):

    action('index', function() {
        render();
    });

It will render view 'lists/index' located in `./app/views/lists/index.ejs`
within the layout. The convention is that unless you are calling a view with name
other than the action name, you can omit it.

##### Layouts

A layout is a view wrapping the target view. By convention, the layout called will
share the same name as the controller. A rendered view is passed into a layout as 
the `body` variable. To render a view without layout you can specify 
`{layout: false}` as param or `render` method, or just `this.layout = false`
inside controller action. You also can specify what layout you want to render
using `c.layout('name');` method of controller context:

    action('landing', function(c) {
        c.layout('welcome');
    });
