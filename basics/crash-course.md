## Crash Course to CompoundJS

The guide demonstrates how to create simple compound app using generators.

Goal: working application without knowledge of express, learn structure and
tools coming with compound and get quick overview of main features.

### What is compound

Before we start I have to define compound framework. Compound's formula:
[Express][express] + structure + extensions. Where **structure** is standard
layout of directories, and **extensions** are node modules adding functionality to
the framework. Compound's goal: provide obvious and well-organized interface for
express application development. That means everything working with express will
work with compound.

Now we can start building app. Let's create todo-list app with REST API and web
interface.

### First steps: install compound and generate app

1. install compound using npm

    npm install compound

2. generate app

    compound init todo-list-app

3. install dependencies

    cd todo-list-app && npm install

Now we have initial compound app structure, we can run application and see
what will happen. Let's run `node .` command and open http://localhost:3000/ in
browser.

> **NOTE** `node .` command is simple way to run application in current
> directory. You also can run `node server.js` or `coffee server.coffee` or even
> `compound server`: result will be the same, we will learn differences later.

After opening localhost:3000 in browser we will see welcome page with some links
and debug info (appers after clicking on corresponding link). This is static
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

Now we have some stuff in out application to explore structure, let's do brief
overview of files we have to get lists CRUD (create-read-update-delete) working.

### Explore app structure

#### Router

Let's start with routes because this is first place in compound stack where
request handling happens. In other words, when you open
http://localhost:3000/lists in your browser router should decide what part of
application should handle this request. Routes are configuration rules explains
to application what path your application can handle.

Routes listed in file `config/routes.js` which look like:

```javascript
exports.routes = function (map) {
    map.resources('lists');
    map.all(':controller/:action');
    map.all(':controller/:action/:id');
};
```

Line `map.resources('lists');` was added by list scaffold generator. We can see
which express routes was created by this line using `compound routes` command:

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

First column is route helper name, second - method, third - route, last -
controller#action to handle request. Helper name should be used to generate
path, all route helpers available as methods on `pathTo` object. Examples:

    pathTo.lists() // '/lists'
    pathTo.lists({format: 'json'}); // '/lists.json'
    pathTo.list(1); // '/lists/1'
    pathTo.list(1, {format: 'json'}); // '/lists/1.json'
    pathTo.list({id: 1, format: 'json'}); // '/lists/1.json'
    pathTo.new_list(); // '/lists/new'
    pathTo.edit_list(1); // '/lists/1/edit'
    pathTo.edit_list('my-list'); // '/lists/my-list/edit'

> By default path helpers generated with underscores as words separators, this
> behavior will be changed in future versions and it's highly recommended to add
> line map.camelCaseHelperNames = true; to the top of file to generate names in
> camelCase: `newList` and `editList` instead of `new_list` and `edit_list`

#### Controller

When route matched, request handling passed to corresponding controller#action.
Controllers located in `./app/controllers`. For example, our list controller
described in `./app/controllers/lists.js`.

> Please note that the name of controller you've generated using scaffold
> generator ends with <code>_controller</code>. This is just different format of
> controller which means that controller code will be `eval`ed in
> `controllerContext`, it is so-called 'eval-controller'. When file doesn't ends
> with _controller - it's 'noeval controller', all differences between eval and
> noeval controllers explained in API docs for
> [controller](http://compoundjs.com/man/controller.3.html)

Controller consists of set of actions and hooks. Both actions should handle
request or call `next()` if request could not be handled. In case if `next()`
called control will be returned to router which will try to match next route to
handle this request. If action could not handle request because of error, you
can call `next(err)` and control will be passed to error handler express
middleware (skipping all routes).

Hooks allows you to prepare environment before action or do something additional
after action. For example, load list before edit, update, show actions.

The most common results of action: render, send or redirect. And of course
next(err) - this is most often used result of any action.

The most important thing about controller: it should not contain business logic.
Of course you able validate your model, do some other things with model before
create and update. But it will ruine your controller and application code. Just
do everything in model, use proper ORM which ships validation and hooks.

#### View

Controller action may decide to render view. All views in compound application
located in `./app/views`. There are also special directory for layouts:
`./app/views/layouts`. Every view or layout inside app/views could be rendered
using `render` method of controller context. One thing: you should not point
view extension. And the last rule: be lazy, do not specify view name when it is
the same as action name. Example (lists_controller.js):

    action('index', function() {
        render();
    });

It will render view 'lists/index' located in `./app/views/lists/index.ejs`.
