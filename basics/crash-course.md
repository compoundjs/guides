## Crash Course to CompoundJS

The guide demonstrates how to creating simple compound app using generators. 

Goal: working application without knowledge of express, learn
structure and tools coming with compound and get quick overview of main features.

### What is compound

Before we start I have define compound framework. Compound's formula:
[Express][express] + structure + extensions. Where **structure** is standard
layout of directories, and **extensions** are node modules adding functionality to
the framework. Compound's goal: provide obvious and well-organized interface for
express application development. That means everything working with express will
work with compound.

Now we can start building app. Let's create a new app with REST API and web
interface and call it "todo-list".

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

### Generate scaffold (new entities)

Run this command

    compound generate scaffold list name

It will generate all necessary files for `List` model. Then stop server using
`CTRL+C` hotkey and run it again to hook up changes.

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
request handling happens. Routes listed in file `config/routes.js` which look
like:

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
Controllers located in `./app/controllers`. For example, the list controller
described in `./app/controllers/lists\_controller.js`

[express]: https://github.com/visionmedia/express
