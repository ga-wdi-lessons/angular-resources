# Factories and `ngResource`

## Learning Objectives

* Explain the purpose of Factories in Angular.
* Use `ngResource` to pull information from an API.
* Use $stateParams to access query parameters and update the URL.
* Create separate views and routes for each CRUD action.

## Framing

In the last couple of classes, we've been using hardcoded values in our controller to act as our "backend". We probably won't ever do that again. Instead we'll be connecting to an external API using resources and providing an interface to models using factories.

## You Do: Walkthrough of Current App (20 minutes / 0:20)

> With the person next to you, take 15 minutes to walk through the following part of the lesson plan, up to the `Factories` header. Read our descriptions of the different components.

> We'll then take the next 5 minutes for questions.

Run the below commands to clone the starter code. You will not be using the code you created in the `ui-router` class.  

```bash
$ git clone https://github.com/ga-wdi-exercises/grumblr_angular.git
$ cd grumblr_angular
$ git checkout factory-resource-starter
```

Where we're picking up the app, it has...
* A functioning index route that uses grumbles hardcoded into the index controller
* The makings of a show route. We'll need to build this out.

#### index.html

```html
<!DOCTYPE html>
<html data-ng-app="grumblr">
  <head>
    <title>Angular</title>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.5.6/angular.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-router/0.4.2/angular-ui-router.min.js"></script>
    <script src="js/app.js"></script>
  </head>
  <body>
    <h1><a data-ui-sref="grumbleIndex">Grumblr</a></h1>
    <main data-ui-view></main>
  </body>
</html>
```
> **data-ng-app**: Establishes the domain of our Angular application.  
>  
> **data-ui-sref:** This creates a link that, when clicked, directs the user to `#/grumbles` without reloading the page.
>
> We use this instead of the usual `href` because `sref` refers to a state and automatically grabs the URL for that state. It's like link helpers in rails.
>  
> **data-ui-view:** Whichever view is triggered by the user will be displayed in the DOM element with this attribute.  

#### js/ng-views/index.html

```html
<h2>These are all the Grumbles</h2>

<div data-ng-repeat="grumble in vm.grumbles">
  <p>{{grumble.title}}</p>
</div>
```
> **data-ng-repeat:** Allows us to iterate through each item in the array passed in as an argument. In this case, the controller's `grumbles` property.  
>  
> **vm:** Represents the current instance of our index controller.  

#### js/app.js

```js
  angular
  .module("grumblr", [
    "ui.router"
  ])
  .config([
    "$stateProvider",
    RouterFunction
  ]);

  function RouterFunction($stateProvider){
    $stateProvider
    .state("grumbleIndex", {
      url: "/grumbles",
      templateUrl: "js/ng-views/index.html",
      controller: "GrumbleIndexController",
      controllerAs: "vm"
    })
    .state("grumbleShow", {
      url: "/grumbles/:id",
      templateUrl: "js/ng-views/show.html"
    });
  }
```
> **.module:** A module is a container for controllers, directives, services -- all parts of our application. A module can have sub-modules .
>  
> **ui.router:** A 3rd party module that functions as a router.  Allows our application to have multiple states.  
>  
> **$stateProvider:**  A ui-router service that allows us to define states in our application.  
>  
> **.state:** Used to define an individual state in our application. Arguments include (1) state name and (2) an object that contains information about route, template and controller used.  


#### Index Controller

```js
  .controller("GrumbleIndexController", [
    GrumbleIndexControllerFunction
  ]);

  function GrumbleIndexControllerFunction(){
    this.grumbles = [
      {title: "These"},
      {title: "Are"},
      {title: "Hardcoded"},
      {title: "Grumbles"}
    ]
  }
```
> **GrumbleIndexController:** The name of this controller.  
>  
> **GrumbleIndexControllerFunction:** A function that contains this controller's behavior. This is a stylistic decision - we could have passed in an anonymous function to `.controller` if we wanted to.  

You'll notice that, at the moment, we have hardcoded models into the Grumbles controller. Today we'll be learning about `ngResource`, a module that allows us to make calls to that Rails API we'll set up now.

## Set Up Grumblr API (5 minutes / 0:25)

Let's start by cloning and running a Grumblr Rails API in the background. Our front-end Grumblr application will make AJAX calls to this API.

```bash
$ git clone https://github.com/ga-wdi-exercises/grumblr_rails_api.git
$ cd grumblr_rails_api
$ bundle install
$ rails db:drop
$ rails db:create
$ rails db:migrate
$ rails db:seed
$ rails s
```

## Factories (10 minutes / 0:35)

First up, we'll convert the hardcoded data to read from an external API using a factory.

### Factory

A factory is an Angular component that adds functionality to an Angular application. It does this by generating new instances of something. In this case, Grumbles.  

Factories allow us to separate concerns and extract functionality that would otherwise be defined in our controller. We do this by creating an object, attaching properties and methods to it and then returning that object.

Let's start building out a factory in Grumblr.

```js
    .factory( "GrumbleFactory", [
      GrumbleFactoryFunction
    ]);

  function GrumbleFactoryFunction(){
    return {
      helloWorld: function(){
        console.log( "Hello world!" );
      }
    }
  }
```
> Factories can also take dependencies. In that case, the arguments passed into a factory will look a little different. We'll see that in play when we learn about `ng-resource` later today.

Now we can call it in a controller...

```js
    .controller( "GrumbleIndexController", [
      // The factory is passed in as a dependency to our controller.
      "GrumbleFactory",
      GrumbleIndexControllerFunction
    ]);

  function GrumbleIndexControllerFunction( GrumbleFactory ){
    // When `helloWorld` is called, it runs the function we defined in our factory file.Factory
    GrumbleFactory.helloWorld();
  }
```
> This is nice because it keeps our controller clean. We leave the function declaration(s) to our factory.

<!-- <details>
<summary>Bonus! Services!</summary>


A service achieves the same purpose as a factory. It is instantiated, however, using the `new` keyword. Instead of defining an object and returning it, we attach properties and methods to `this`. Let's recreate the above factory using a service...

Our controllers look nearly identical in both examples. The difference is in the content of the factory and service. **What do you notice?**

Which One Should I Use?

The answer is it doesn't really matter. You might take a look at this "cheat sheet" of what should be used when:

[http://demisx.github.io/angularjs/2014/09/14/angular-what-goes-where.html](http://demisx.github.io/angularjs/2014/09/14/angular-what-goes-where.html)

Great article comparing Factories, Services, & Providers:

[http://tylermcginnis.com/angularjs-factory-vs-service-vs-provider/](http://tylermcginnis.com/angularjs-factory-vs-service-vs-provider/)
</div>
</details> -->

### I Do: Create Grumble Factory (15 minutes / 0:50)

> Do not code along. You will have the chance to write all this code in the following exercise.

Let's make a factory that's actually useful. It's purpose: enable us to perform CRUD actions on our Rails Grumblr API.  

By default, Angular does not include a way to interact with APIs. For that, there is a separate module, called [ngResource](https://docs.angularjs.org/api/ngResource). The ngResource module provides interaction support with RESTful services.

Let's include it in our application using a CDN.  

```html
<!-- index.html -->

<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.5.0-beta.2/angular-resource.min.js"></script>
```

Add `ngResource` as a dependency to our application.

```js
angular
  .module("grumblr", [
    "ui.router",
    "ngResource"
  ])
```

And `$resource` as a dependency to our `GrumbleFactory`

```js
    .factory( "GrumbleFactory", [
      "$resource",
      GrumbleFactoryFunction
    ]);

    function GrumbleFactoryFunction( $resource ){
      return $resource( "http://localhost:3000/grumbles/:id" );
    }
```

Out of the box, this gives us several methods for our newly defined `Grumble` factory...

* `GrumbleFactory.get`  
* `GrumbleFactory.save`  
* `GrumbleFactory.query`  
* `GrumbleFactory.remove`  
* `GrumbleFactory.delete`  

> Where's `update`, you ask? We're going to define that ourselves later on.  

When the data is returned from the server, the response object is an instance of the resource class. The actions `save`, `remove` and `delete` are available on it as methods with the `$` prefix. This allows you to easily perform CRUD operations on server-side data like this...  

```js
var Grumble = $resource('/grumbles/:id');
var grumble = Grumble.get( { id:123 }, function(grumble) {
  grumble.abc = true;
  grumble.$save();
});
```

#### Let's Test It Out With `.query`...

Let's update our index controller so that, instead of using hardcoded grumbles, `this.grumbles` is set to the result of making a `GET` request to `http://localhost:3000/grumbles`.

```js
    .controller( "GrumbleIndexController", [
      "GrumbleFactory",
      GrumbleIndexControllerFunction
    ]);

    // Whenever `.grumbles` is called on our ViewModel, it returns the response from `.query()`
    function GrumbleIndexControllerFunction( GrumbleFactory ){
      this.grumbles = GrumbleFactory.query();
    }
```
### You Do: Create Grumble Factory + Update Index Controller (10 minutes / 1:00)

### Break (10 minutes / 1:10)

### You Do: Show (20 minutes / 1:30)

#### Modify the Show `.state()` in `app.js`

* Include values for `controller` and `controllerAs`.  

```js
.state("grumbleShow", {
  url: "/grumbles/:id",
  templateUrl: "js/ng-views/show.html",
  controller: "GrumbleShowController",
  controllerAs: "vm"
});
```

#### Update `index.html`

Each grumble listed here should link to its corresponding show page.

```html
<div data-ng-repeat="grumble in vm.grumbles | orderBy:'-created_at'">
  <p><a data-ui-sref="grumbleShow({id: grumble.id})">{{grumble.title}}</a></p>
</div>
```
> NOTE: The `data-ui-sref` attribute is not only set to route name `grumbleShow`, but `grumbleShow` also takes an `id` as an argument so that it knows which show page it should direct the user to.

#### Create a Show Controller

This will look very similar to the index controller, which a couple exceptions.  
The controller requires access to ui-router's `$stateParams` service. We pass it in the same way we do `GrumbleFactory`.  
* `$stateParams` returns an object containing the information passed into, in this case, `grumbleShow`. If we print it to the console, it looks like this...
```js
Object {id: "6"}
```

The controller will have a `grumble` property. It should be set to the return value of `ngResource`'s `get` method.
* `ngResource`'s `get` method requires an object as an argument, which contains a key-value pair for the grumble's id.  

```js
  .controller("GrumbleShowController", [
    "GrumbleFactory",
    "$stateParams",
    GrumbleShowControllerFunction
  ]);

  function GrumbleShowControllerFunction(GrumbleFactory, $stateParams){
    this.grumble = GrumbleFactory.get({id: $stateParams.id});
  }
```

#### Update `show.html`

Use what you learned on your first day of Angular to create a show view for a Grumble. It should display the Grumble's title, author name, created at date, content and photo.  

### I Do: New/Create (15 minutes / 1:45)

> Do not code along. You will have the chance to write all this code in the following exercise.

#### Add New Link to `index.html`

This link will trigger the `grumbleNew` state when clicked.

```html
<a data-ui-sref="grumbleNew">New Grumble</a>

<h2>These are all the Grumbles</h2>

<div data-ng-repeat="grumble in vm.grumbles">
  <p><a data-ui-sref="grumbleShow({id: grumble.id})">{{grumble.title}}</a></p>
</div>
```

#### Create `grumbleNew` Route

```js
// app.js

$stateProvider
  .state("grumbleIndex", {
    url: "/grumbles",
    templateUrl: "js/ng-views/index.html",
    controller: "GrumbleIndexController",
    controllerAs: "vm"
  })
  .state("grumbleNew", {
    url: "/grumbles/new",
    templateUrl: "js/ng-views/new.html",
    controller: "GrumbleNewController",
    controllerAs: "vm"
  })
  .state("grumbleShow", {
    url: "/grumbles/:id",
    templateUrl: "js/ng-views/show.html",
    controller: "GrumbleShowController",
    controllerAs: "vm"
  });
```

> NOTE: `grumbleNew` is placed before `grumbleShow`. This is important - why? Switching `grumbleNew` and `grumbleShow` may shed some light on this...  

#### Create new controller

In the `GrumbleNewControllerFunction`, let's bring in `GrumbleFactory`, instantiate a `new` instance of it that can be modified by form inputs, and add a `create` method that allows it to be send to our API.

```js
  .controller( "GrumbleNewController", [
    "GrumbleFactory",
    GrumbleNewControllerFunction
  ]);

  function GrumbleNewControllerFunction( GrumbleFactory ){
    this.grumble = new GrumbleFactory();
    this.create = function(){
      this.grumble.$save()
    }
  }
```

#### Create `new.html`

Let's create a form view for creating Grumbles. Notice how `data-ng-model` references what we defined as `this.grumble` in our controller in the previous step.

```html
<!-- js/ng-views/new.html -->

<h2>Create Grumble</h2>

<form>
  <input placeholder="Title" data-ng-model="vm.grumble.title" />
  <input placeholder="Author name" data-ng-model="vm.grumble.authorName" />
  <input placeholder="Photo URL" data-ng-model="vm.grumble.photoUrl" />
  <textarea placeholder="Grumble content" data-ng-model="vm.grumble.content"></textarea>
  <button type="button" data-ng-click="vm.create()">Create</button>
</form>
```
> Fields are matched to grumble properties using the `data-ng-model` directive.  



### You Do: New/Create (10 minutes / 1:55)

### Break (10 minutes / 2:05)

### You Do: Edit/Update (15 minutes / 2:20)

The steps here are pretty similar to those of the last "I Do," with a few exceptions. The biggest one is...

#### Define an `update` method in the Factory

`ngResource` does not come with a native `update` method. We need to define it in the `FactoryFunction` return statement, indicating that `update` corresponds to a `PUT` request.  

```js
    .factory( "GrumbleFactory", [
      "$resource",
      FactoryFunction
    ])

  function FactoryFunction( $resource ){
    return $resource( "http://localhost:3000/grumbles/:id", {}, {
        update: { method: "PUT" }
    });
  }
```

The rest of the steps are a bit more straightforward...  

#### Update `index.html`

Let's update our `ng-repeat` div so that it also displays a link with each Grumble that will direct us to an edit page.

```html
<!-- js/ng-views/index.html -->

<div data-ng-repeat="grumble in vm.grumbles">
  <p><a data-ui-sref="grumbleShow({id: grumble.id})">{{grumble.title}}</a></p>
  <a data-ui-sref="grumbleEdit({id: grumble.id})">Edit</a>
</div>
```

#### Create `grumbleEdit` Route

Follow the same process we did for `grumbleNew`, making sure to use the word `edit` wherever necessary.  

Not sure what URL to use? Think about what the path would look like for an edit form in a Rails app...  

#### Create `GrumbleEditController`

The big addition here is our controller's `update` method. You'll notice that it makes use of `$update`. THIS is the method we defined in the grumble factory. It is preceded by a `$` because this is how `ngResource` indicates it's an instance method.

```js
    .controller( "GrumbleEditController", [
      "GrumbleFactory",
      "$stateParams",
      GrumbleEditControllerFunction
    ]);

  function GrumbleEditControllerFunction( GrumbleFactory, $stateParams ){
    this.grumble = GrumbleFactory.get({id: $stateParams.id});
    this.update = function(){
      this.grumble.$update({id: $stateParams.id})
    }
  }
```

#### Create `edit.html`

The form on this page will look a lot like the one in `new.html`, but you'll need to make some changes to it...
* Reference the proper controller instance. You probably called it `vm`.
* Replace your inputs' `placeholder` attribute with `value` so we have some content to work with in our input fields upon page load.
* Set these value attributes to the contents of the Grumble like so...
```html
<input value="vm.grumble.title" ... >
```
* In the button's `ng-click` directive, reference the controller's `.update` method instead of `.create`.

### You Do: Delete (10 minutes)

>  We may not get to this in-class.  

Contrary to how we've done things for every other RESTful route, we will not be creating a separate controller for `delete`. This is because we want to be able to delete a Grumble simply by clicking a button on each grumble's show page.

#### Add Delete Button to `edit.html`

When clicked, the delete button will trigger a `destroy` method that we have yet to define in `edit.controller.js`.

```html
<form>
  <input value="vm.grumble.title" data-ng-model="vm.grumble.title" />
  <input value="vm.grumble.authorName" data-ng-model="vm.grumble.authorName" />
  <input value="vm.grumble.photoUrl" data-ng-model="vm.grumble.photoUrl" />
  <textarea value="vm.grumble.content" data-ng-model="vm.grumble.content"></textarea>
  <button data-ng-click="vm.update()">Update</button>
  <button data-ng-click="vm.destroy()">Delete</button>
</form>
```

#### Add Destroy Method to Edit controller

```js
    .controller( "GrumbleEditController", [
      "GrumbleFactory",
      "$stateParams",
      GrumbleEditControllerFunction
    ]);

  function GrumbleEditControllerFunction( GrumbleFactory, $stateParams ){
    this.grumble = GrumbleFactory.get({id: $stateParams.id});
    this.update = function(){
      this.grumble.$update({id: $stateParams.id})
    }
    this.destroy = function(){
      this.grumble.$delete({id: $stateParams.id});
    }
  }
```

### Closing/Questions (10 minutes / 2:30)

* Why do we use factories?
* What do factories return?
* What are `ngResource` and `$resource`? What methods do they provide us with?
* How do we use `$resource` and a factory to create and save something to an API/database?

----------

### Grumblr Bonuses

> Only attempt the following once you have set up full CRUD functionality for Grumblr.

#### state.go

Use `state.go` so that when a user creates, edits or deletes something, they are directed to a page that is not the same form.

> We're not covering this in today's class. [Learn more in the `ui-router` documentation](https://github.com/angular-ui/ui-router/wiki/Quick-Reference).

#### Hardcore SPA

Only use a single view and controller (i.e., you should be to execute full CRUD functionality from the index). The user should only be able to see forms for creating and updating after clicking buttons for those respective actions.

> This will require making use of directives we didn't use in today's class, like `ng-show` and `ng-hide`.

### Homework (Optional)

Finish implementing full CRUD functionality for Grumblr using `ngResource`. In other words, finish going through all the code in this lesson plan.  

Links to the starter and solution code can be found in the [`grumblr_angular` repo](https://github.com/ga-wdi-exercises/grumblr_angular).

### Resources

* Angular documentation for [ngResource](https://docs.angularjs.org/api/ngResource) and [$resource](https://docs.angularjs.org/api/ngResource/service/$resource).
* [Angular: What Goes Where?](http://demisx.github.io/angularjs/2014/09/14/angular-what-goes-where.html)
* [Factory vs. Service vs. Provider](http://tylermcginnis.com/angularjs-factory-vs-service-vs-provider/)

## Screencasts
- Dec 16, 2015 (Robin)
  - [Part 1](https://youtu.be/Ni-KnX9eEDI)
  - [Part 2](https://youtu.be/Jm4lmgpQfJ8)
  - [Part 3](https://youtu.be/dP0YsPTnaTU)
  - [Part 4](https://youtu.be/oEFmmQgh4cE)
