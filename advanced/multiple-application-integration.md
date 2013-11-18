Multiple applications in a single application
==============

This guide presents how to integrate multiple "child" Compound applications in a "parent" application.

After the integration, the child applications can still be used as standalone for easier development.

This guide is divided in two sections: how to prepare the child applications for the integration and how to do the integration in the parent.

Integration guide
==============

Modifications to the child application before the integration:
--------------

- At the beginning of “config/environment.js” add the following code :
	(Where UNIQUE_NAME is a name to serve as a unique identifier of your application)

		if(typeof compound.subRoot === 'undefined'){
		 	compound.subRoot = [];
		 	compound.subPublic = [];
		 	compound.subRoot['UNIQUE_NAME'] = compound.root;
		 	compound.subPublic['UNIQUE_NAME'] = "";
		}

- Change all **compound.root** and **compound.app.root** in the project that directs toward a local file by **compound.subRoot['UNIQUE_NAME'];**.
- For the whole project, add the compound.subPublic['UNIQUE_NAME'] everywhere an element of the public folder is called without the Compound.subRoot['UNIQUE_NAME']
	(i.e. those that uses a relative path to web host instead of the file system).
		
		(NB: subPublic will have the / at the end once integrated)
		Example: "/"+compound.subPublic[UNIQUE_NAME]+ "images/images.jpg"

- Rename the “application_controller.js” and “application_layout.js” to a unique name.
- Change the load('application'); in the controllers to use the new name.
- If applicable, change the route to change the “application” route to use the new name.
- In the application controller, add **this.publicPath = compound.subPublic['PRT'];** in a "before" statement and change the javascript and style inclusion in the child’s layout.
	
		<%- stylesheetLinkTag('../'+publicPath+'stylesheets/bootstrap')%>
		<%- javascriptIncludeTag('../'+publicPath+'javascripts/external/jquery.min')%>

- In “route.js”, add **if(map.app.compound.subPublic['UNIQUE_NAME'] === "")** to “root” or "/" routes.
- To limit the changes, it is highly recommended to split the layout in multiple files, taking the menu out of the layout page and instead include it with **<% include FILE %>**. This will simplify the integration by only modifing the parent's menu instead of every childs.

**If using AngularJS (for other front-end module, do something similar):**

- In “…_layout.ejs”, add: **<script>var publicRoot = "<%=publicPath%>";</script>** after the title.
- Add **publicRoot** to the routing provider and in the **$rootScope** (in app.run) to make it available in the html files for relative paths as an angular variable.
- Do not use a “/” route in the routing provider, instead set a named route for your default action (ex.: “/default”) and set the “.otherwise” to redirect to that route.
		
		app.config(['$routeProvider',function($routeProvider ) {
		 	$routeProvider.when('/FILE', {
				templateUrl: publicRoot+'PATH_TO_VIEW/FILE.html',
		 	});
		 	.otherwise( { redirectTo: '/default'});
		}]);
		app.run(['$rootScope', '$location', function($rootScope, $location){
		 	$rootScope.publicRoot = publicRoot;	//Make it available to html
		}]);

Even with these modifications, the child application should still be able to run as a standalone.

Integration in the parent application:
--------------

- Copy the whole child project in the parent’s “node_modules”. The name of this module will be noted MODULE_NAME in the following code.
- Create a symbolic link in the parent’s public that points to the child’s public directory.
	The name of that symlink will be noted SUBDIR_NAME in the following code.
- Add the module in the parent’s “config/autoload.js”: **require('MODULE_NAME ')**.
- Remove the "server.js" file in the child application (in MODULE_NAME’s root).
- Add a “index.js” file at MODULE_NAME’s root containing: 

		exports.init = function(compound) {
			//To keep original root
				var oldRoot = compound.root;
			//Change compound root to current module
				compound.root = compound.root+"/node_modules/MODULE_NAME";
				compound.app.root = compound.root;
			//Create the sub arrays if they have not been created by another sub-app
				if(typeof compound.subRoot === 'undefined'){
					compound.subRoot = [];
					compound.subPublic = [];
				}
			//Change the location of the resource files to the subdirectories
				compound.subRoot['UNIQUE_NAME'] = compound.root;
				compound.subPublic['UNIQUE_NAME'] = "SUBDIR_NAME/";
			//Load the application in compound
				compound.init(compound.root);
			//Set back the original root
				compound.root = oldRoot;
				compound.app.root = oldRoot;
		};

- Add the parent application in the child’s application controller: **load('application');** and remove the “protectFromForgery”.
- Create a symlink in the child layout to the parent’s included files (ex. menu, header, etc) and include it in the child’s layout : **<% include FILE %>** (delete conflicting ones).
- Add the parent’s stylesheet in the child’s layout.
- Modify the menu in “/app/views/layouts/header.ejs” to add child application’s links.

**NB:**	The child’s controllers are always overridden by the parent’s controllers and might be overridden by other child’s depending on the order they are loaded. So you can use that by making “fake” controllers in the child that gets overridden with the parent.

Ex.: The child module can use a fake “authentication_controller” without validation for the standalone, and it will use the parent’s “authentication_controller” when integrated.

You must also be careful to not have modules, controller, models or other elements using the same name; it will either be overridden or cause an error when starting the server. If some elements need to be used by multiple childs, you can include it in the parent instead, as for the authentication module in the example above.
