### Helpers

####Links

Creating links in Compound is easy with the `linkTo` and `pathTo` helpers. Using them
will help to make your app more portable and easier to maintain.

Say we have a Users model, and an index page listing all our users. We could create
the following link in our view:

`<a href="/users/index">Users</a>`

While that may work for now, we can also use a helper to create the link for us:

`link_to('Users', pathTo.users)`

The benefit of using `pathTo`, is that it will handle the adding the path to page for you,
using the routes configuration file (`/config/routes.js`). 

To see your routes, do the following in the node console:

`compound routes`

And you will get a list of your current routes in the following format:

`users GET    /users.:format?    users#index`

In the example above, from left-to-right:

 * `users GET` - The path, this is what we would use in `pathTo`
 * `/users.:format?` - This tells us what the link looks like, and what paramaters it takes. In this example, the _format_ describes what this contoller action responds to (see controllers for `respondsTo`).    
 * `users#index` - Convention is _controller_ # _action_, so this example will look for the `index` action in the `users` controller. 

#####Customizing Links

`linkTo` also takes optional arguments so that you can add classes, id, etc.

**Examples:**

 * `link_to('Cancel', pathTo.users, { class: 'btn', id: 'cancel' )`
 * `link_to('Add Another', false, { class: 'add-to-cart', data-item: 'WDGT-3000' )`

**Date-Remote:**

The `linkTo` helper also offers a convenient method for sending asynchronous requests back to the server with
only one additional argument:
 
    linkTo('Users index', '/users', { remote: true })
    // <a href="/users" data-remote="true" data-jsonp="renderUsers">Users index</a>

In the example above, the third argument (_{ remote: true }_), adds a data-remote="true" attribute to the `a` tag. 
Clicking on this link will send an asynchronous GET request to /users. The result will be executed as Javascript.

You can also specify a jsonp parameter to handle the response:

    linkTo('Users index', '/users', { remote: true, jsonp: 'renderUsers' })
    // <a href="/users" data-remote="true" data-jsonp="renderUsers">Users index</a>

The server will reply with json response { users: [ {}, {}, {} ] }, and this object will be passed as an argument 
to the _renderUsers_ function (you will need to create this method in your users_controller):

`renderUsers({users: [{},{},{}]});`

You can also specify an anonymous function in the jsonp param:

`{ jsonp: '(function (url) { location.href = url; })' }`

If the server sent you "http://google.com/", the following javascript will be evaluated:

`(function (url) { location.href = url; })("http://google.com");`

####Form Helpers

Forms are easyto create in Compound with the `formFor` helper.

`formFor` takes two arguments: `resource` and `params`, and returns a `form` object. As an added protection layer,
forms also generate a [`csrf token`](http://en.wikipedia.org/wiki/Cross-site_request_forgery) which is verified 
in the conroller when the form is submitted.

The `form` object has the following availble helpers:

 * `begin` - opening `<form>` tag
 * `end` - closing `<form>` tag
 * `input`
 * `label`
 * `textarea`
 * `submit`
 * `checkbox`
 * `select`

Let's see an example of a form:

    <% var form = formFor(user, { action: path_to.users }); %>
    <%- form.begin() %>

    <%- form.label('name', 'Username') %> <%- form.input('name') %>
    <%- form.submit('Save') %>

    <%- form.end() %>

Which ouputs:

```
<form action="/users/1" method="POST">
  <input type="hidden" name="_method" value="PUT" />
  <input type="hidden" name="authenticity_token" value="RANDOM_TOKEN" />
  <p>
    <label for="name">Username</label>
    <input id="name" name="name" value="Anatoliy" />
  </p>
  <p>
    <input type="submit" value="Save" />
  </p>        
</form>
```

#####Quick Forms
 
Forms can also be created without requiring a resource with `formTagBegin`. This is the "light" 
version of the `formFor` helper which expects only one argument: `params`. Use this helper when you 
don't have a resource, but still want to be able to use simple method overriding and csrf protection 
tokens.

**An example:**

    <%- formTagBegin({ action: path_to.users }); %>

    <%- labelTag('First name', { name: 'name'}) %>
    <%- inputTag('name', {value: 'Sascha'}) %>
    <%- submitTag('Save') %>

    <%- formTagEnd() %>

**This will generate:**

    <form action="/users" method="POST">
      <input type="hidden" name="authenticity_token" value="RANDOM_TOKEN" />
      <p>
        <label for="name">Username</label>
        <input id="name" name="name" value="" />
      </p>
      <p>
        <input type="submit" value="Save" />
      </p>        
    </form>

#####Form Elements

######Inputs

Use the `inputTag` helper to create a form <input> in a form where you _don't have a resource_. 

**Example:**

    <%- inputTag({name: 'creditCard', type: 'text', autocomplete: 'off'}) %>

This produces:

    <input type="text" name="creditCard" autocomplete="off" />

When you are creating a form for an object that belongs to a resouce (like a user's name), use
`form.input`.

    <%- form.input('name', {options}) %>

Using the `form.input` helper creates the same html markup as `inputTag`, but it also add the _value_ 
of the resource(in this case `User`) passed to form, and specifies it as a value="" html attribute:

    <input name="name" value="Sascha" />

######Select Boxes / Dropdown Lists

`<select>` boxes are easy to create if you follow this convention:

**If your data is structured like this:**

    var states = [
     ...,
     { name: 'California', _id: 3 },
     { name: 'Texas', _id: 47, selected: true },
     ...
    ]

**...and your form is structured like this:**

    <%- form.select('state', States, { fieldname: 'name', fieldvalue: '_id' }) %>

**...your select list will look like this:**

    <select name="states">
      <option value="3">California</option>
      <option value="47" selected="selected">Texas</option>
    </select>

######Labels

Use the `labelTag` to create labels for your forms. Just like the `inputTag` above, there are two 
variations of the `labelTag`:

    <%- labelTag('Text on label', {'for': 'attachedInput', style: 'font-size: 10px'}) %>
    <%- form.label('attachedInput', 'Text on label', {style: 'font-size: 10px'}) %>

will both generate

    <label for="attachedInput" style="font-size: 10px">Text on label</label>

When you have a resource, and/or if you are using _i18n_, use `form.label`.

**I18N**

When using internationalization (I18N), you can change the language of form labels on the fly. So, in the
following example:

    <%- form.label('name', 'Name', {style: 'font-size: 10px'}) %>

The second argument is omitted if i18n is turned on, and the desired value from locale file is used 
automatically. For example, if we have a es.yml ( _Spanish_ ):

    es:
      models:
        User:
          fields:
            name: nombre

...and form looks like:

    <% var form = formFor(user); %>
    <%- form.label('name') %>

...the result will be:

    <label for="name">nombre</label>

**i18n and DRY**

Instead of manually adding label names, you could just use the locale file to store it for you, and
everywhere you used a form, the label would use whatever you added to the locale file.

Create `en.yml` with the following:

    en:
      models:
        User:
         fields:
          name: Name of user

And everywhere you had `name` in a form just use:

    <%- form.label('name', {options}) %>
    // <label for="name">Name of user</label>

######Submit

Submit tags follow the same conventions as `inputTag` and `form.input`, but produce a submit button: 

    <%- submitTag('Submit data') %>
    <%- form.submit('Submit data', {options}) %

######All Together Now

Let's put all the form helpers together, and create an address form with a select-list for States. Since 
we are using Twitter Bootstrap, let's also give it some markup to make it look pretty, too.

We are going to assume a couple things:

 * You have an `Address` model that has properties that match those of our form
 * That `states` is populated from a model, and passed from your controller to your view as a `states` object
 * You are using `ejs` as your template engine (this can be easily used with `jade` as well)

```
	<% var form= formFor(user, { id: 'nameForm', class: 'form-horizontal'}); %>
	<%- form.begin() %>

	<legend>Add Address</legend>
	
	<div class="control-group">
		<%- form.label( 'line_1', 'Address Line One', { 'class': 'control-label'}) %>
		<div class="controls">
			<%- form.input( 'line_1', { 'placeholder': '14000 Morris Ln', 'class': 'span4' }) %>
		</div>
	</div>
	
	<div class="control-group">
		<%- form.label( 'line_2', 'Address Line Two', { 'class': 'control-label'}) %>
		<div class="controls">
			<%- form.input( 'line_2', { 'placeholder': 'West Tower Plaza', 'class': 'span4' }) %>
		</div>
	</div>
	
	<div class="control-group">
		<%- form.label( 'city', 'City', { 'class': 'control-label'}) %>
		<div class="controls">
			<%- form.input( 'city', { 'placeholder': 'Los Angeles', 'class': 'span4' }) %>
		</div>
	</div>
	
	<div class="control-group">
		<%- form.label( 'states', 'State', { 'class': 'control-label'}) %>
		<div class="controls">
		  <%- form.select('state', States, { fieldname: 'name', fieldvalue: '_id' }) %>
		</div>
	</div>
	
	<div class="control-group">
		<%- form.label( 'county', 'County', { 'class': 'control-label'}) %>
		<div class="controls">
			<%- form.input( 'county', { 'placeholder': 'Jefferson', 'class': 'span4' }) %>
		</div>
	</div>
	
	<div class="control-group">
		<%- form.label( 'postal_code', 'Zip Code', { 'class': 'control-label'}) %>
		<div class="controls">
			<%- form.input( 'postal_code', { 'placeholder': '12345', 'class': 'span1' }) %>
		</div>
	</div>
	
	<div class="form-actions">
		<%- form.submit( '<i class="icon-ok icon-white"></i>', { class: 'btn btn-primary' }) %>
			<%- linkTo( 'Cancel', pathTo.addresses, { class: 'btn cancel' }) %>
	</div>

	<%- form.end() %>
		
```

####JavaScript & CSS

[ ... ]

####Content Tags

[ ... ]

####Custom Helpers

[ ... ]
