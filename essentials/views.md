### Views

#### Views Overview

Views are the presentation layer of you code, and the Compound framework lets 
you choose the templating engine that will drive your views. For the sake of
this guide, we will be using [ejs](https://github.com/visionmedia/ejs) and 
[jade](https://github.com/visionmedia/jade), two of the most popular templating
engines around.

You can choose to use any view/templating engine that you want, and if you ever
want to change it, it's as as easy as changing one line in you app's _environment_ file.

**Ex. ./config/environment.js**

    app.configure(function() {
      ...
      app.set('view engine', 'ejs');
      ...
    }

#### View Locations and Conventions

Views are stored under the `apps` directory, and by convention, should be located inside
directories named after the controller that will be calling them, and they should be named
after the action they perform:

**Ex. posts_controller:**

    app/
      views/
        posts/
          index.ejs
          show.ejs

> Views used/shared in partials should be named with an underscore `_`, like `_form.jade`.
> See the Partials section below for more information.

#### Rendering a View

Views are rendered from the controller. In the Express framework, views are rendered in
the following way:

    res.render(view, locals, callback)

...where _view_ is the name of the view, _locals_ is an object containing data you are
sending to the view, and the _callback_, of course, is the callback method you would like
to call upon error, etc.

In keeping things simple, views in Compound can be rendered as easy as this:

    action(function index() {
      this.title = 'Users index';
      User.all(function (err, users) {
        render('index', { users: users });
      });
    });
    
In Compound, the convention is to _omit_ the 'view' parameter, since the action that invokes
it can call it for us: 

    render('index', { users: users });

And, for even more convenience, anything attached to `this` in the `render` context will be available
to your view, so you can simplify the view above like so:

    render(); 
    // Will render the "index" action, with "users" available to the view

> Tip: Keep it simple, let the action's name determine the name of your view.
> Tip: Don't add the extension to the your view when you are rendering it - Compound is aware of the 
templating/view engine, and will know what extension to add.

### Partials - Including a View Inside Another View

_Partials_, also called _includes_, are views that are called by other views. Partials are very 
helpful for forms (`edit` and `add` for most resources share _most_ of the form), header files, 
rendering rows in a table, and more. 

#### Using Partials/Includes

In `ejs`, we can include a view inside of another view by following this pattern:

    <%- include _form %>

... and in `jade` by the following pattern:

    include _form

As you can see, the two templating engines are nearly identical in how they call _partials_ 
(aka _includes_). 

They are also alike in the following ways:

 * Neither `ejs` nor `jade` require the use of the file's extension (_.ejs_, _.jade_).
 * Both call the included file by looking for it _relative_ to the calling file. So if you don't
specify a path, it will look in the same directory. If you want the directory above use "../", and
so on.
 * In Compound, as in Express, neither templating engine requires "locals", so that means that _any_
variable/array/object available to the parent are fully accessible to the partial, and that goes for
partials calling partials as well.

### Views Best Practices

Sometimes we find that we have a ton of information that we need to show in our views, and we find that our 
views are doing a lot of the work for us, and performing logic. While views can do a lot to make our app do 
more, if you find that views are doing the logic for you, then chances are that you need to refactor.

#### Rule #1 - Views are for Presentation - Not Logic

As a rule of thumb, if your view does not look like rendering `<html>` markup is its main function, then you 
might have too much logic in your view.

`forEach` loops can save you, but make sure that you are looping for presentation, and _not logic_.

**Ex. Good View Logic**

    <h1>Listing Users</h1>
    <ul>
      <% users.forEach(function(user) { %>
        <li><%- user.name %></li>
      <% }); %>
    </ul>

**Ex. Bad View Logic**

    <% var players = []; %>
    <% users.forEach(function(user) { %>
      <% if (user.status === 'uninjured') { %>
        <% players.push(user); %>
      <% } %>
    <% }); %>

    <h1>Tonight's Starting Players</h1>
    <ul>
    <% players.forEach(function(player) { %>
      <li><%- player.name %></li>
    <% }); %>
    </ul>

#### Rule #2 - Views are NOT Asynchronous, and Cannot Perform Asynchronous Actions

Fetching model data, and looping through data that requires asynchronous calls or error-handling, should be
left your controller and models. 

Remember to ask yourself, is this data available to my view? If not, then most likely you will need to go 
back to your controller/model and refactor.
