---
layout: post
title:  "View Controllers and Routers"
categories: [architecture]
---
If you read my [Architecting Complexity](http://www.functionalgibberish.com/blog/2015/1/30/architecting) post, you know I feel pretty strongly about the need for Software Architecture [^0] in software development in general, but especially in "app" development.  So far, I've not gone in to very much detail into my approach to applying Software Architecture to software development.  To be fair, I'm going to focus only on app development, but many of the principles here apply much more broadly.  It just happens that developing apps is something I do a lot, so I've developed a consistent strategy regarding it. 

I can't put all this in one post, so I'll be splitting it out into several posts.  

<!--more-->

## Why?
Software Architecture helps manage complexity, declare dependencies, formally structure the app and in general makes it easier to change the app and Unit Test it.  At first glance, applying architecture to software can make the app appear more complicated, but if you understand the architectural paradigm it will make the app easier to understand.  Morevoer, you know exactly *where* to make changes because the types of tasks are explicitly contained in specific objects.

## Goals
Anybody that does Apple development will be familiar with the MVC (Model View Controller) paradigm.  This is a Software Architecture that defines three essential "types" of objects in an app and how those objects interact with each other.  It is my belief, however warranted, that the MVC is *the barest absolute minimum* architecture that should be applied to an app.  It is the sad, unfortunate state that many software projects often don't meet this standard.  However, to truly build a robust app there should be many more layers of architecture than just the Model, View and Controller.

Let's begin with with some goals, first:
 - Software should be modular.  Meaning the components should be easily rearranged in order to adjust to changes in requirements, new features or even bug fixes.
 - Software components should, as much as possible, conform to the SRP: The Single Responsibility Principle.  This essentially means that any software component should be responsible for one, and only one "thing" (or responsibility).
 - Implicit interdependencies between software components should be eliminated or reduced as much as possible.  This means that component `A` shouldn't *require* component `B` in order to work.  However, component `A` can require a component *like* `B` (via a protocol).
 - Interdependencies should be explicit and swappable.  The best way to do this is through protocols.
 - Dependencies should be injected whenever possible.
 - Never use a singleton if you can avoid it.  Instead, use an injector of some kind.  Singletons create hidden dependencies that make changes unpredictable.  Also, be careful about "practical singletons" in form of some kind of globally referenced state.  For example: storing a class in the app delegate for "easy" reference.  That is a "singleton in practice" and should be avoided.

It's important to keep in mind that these are goals.  Sometimes it just doesn't make sense to go "all the way" given budget and time constraints.  But they should be goals.

The key in software architecture is identifying the *types* of tasks you need to accomplish and specializing certain objects in those types of tasks.  For instance, it's very common for a View Controller to initiate a segue, instantiate another view controller, pop itself off a navigation stack, etc.  But all of these things really have nothing to do with view controllers.  Instead, they're *routing* tasks and as such they should be handled by a separate object that handles routing.  Oddly, I call these "routers".  By using routers, you separate the routing functionality from the view controllers, allowing you to change your routes in a single place.  This makes your codebase much more flexible and easy to change.

## The "Players"
In this part, I'm going over my approach to Routing and handling View Controllers.  I'll explain how I separate application logic from the view controllers and handle dependency injection.  First, lets introduce our players:

- **Router**: The router is the conductor of the app, it controls and managers the transitions between view controllers, injects them with the appropriate delegates and ensures all the "pieces" of the app are where they should be.  Generally, a single router is paired with a storyboard.  This gives the router access to the view controllers it needs.  Alternately, the router can simple instantiate the view controller directly and inject a view layout engine/controller.  For the sake of simplicity, I'm going to assume you're using storyboards as layout engines are a topic to themselves. 
- **View Controller**:  The view controller controls views.  This includes passing data to the views, managing view state (not layouts) and communicating with the router and a dataSource for it's routing and data needs, respectively.
- **Wire Frame**:  The wire frame defines view layouts.  They're often paired with view controllers, but are independent of those View Controllers.  Most commonly, a Storyboard item or a nib file is used, but it can also be a separate class item responsible for handling layouts.
- **Logic Controller** (data source): The basic definition of a logic controller is that it encapsulates a piece of business/app logic.  In this case we're use logic controllers as a data source for the view controllers.  They're job is to provide data for the view controllers and respond to requests for changes made by the view controller.
- **Dependency Injector**:  A dependency injector ranges from something as simple as a class container to more complex classes that dynamically instantiate class types on request.


## Overview
The routers are the conductors.  They instantiate the view controllers and inject them with the appropriate delegates.  In this way, a view controller never need to `import` or use anything other than a couple protocols.  The view controller is both configurable and "siloed".  It has a very clear realm of responsibility and the application logic (via the logic controller) can be switched out (and Unit Tested).  Often the Routers will hold a reference to a simple dependency injector they can use to inject the appropriate class objects.  Logic Controller often need access to several class type (often Model Managers) to do it's job and the Router will pass into the Logic Controller the injector so that it has access to the class objects it needs.  Usually, Logic Controllers are dynamically instantiated and injected into each view controller before it's loaded, either by the Router itself or by the dependency injector.  The router will then inject itself as the view controller's router so it can respond to routing requests by the view controller.

### Routers
There is at least one "Master Router".  The master router is initialized and retained by the app delegate and the key window is passed to it.  The Master router's first responsibility is to set a root view controller, sometimes as a simple placeholder (loading view), an Intro view, login screen or a home "page".  The point here, is the router can dynamically choose which to load, dependent upon whatever necessary conditions are present in the app.  You never need to "go through" another view controller in order to reach a desired state.

Routers are generally paired with a storyboard, or something equivalent, where they can instantiate the appropriate view controller.  It doesn't have to be a storyboard (though they're convenient).  The Router can instantiate the View Controller and inject a layout engine, but since storyboards are so good at this we're going to assume you're using a storyboard for now.

If the app is small, a Single Router & Storyboard may be sufficient.  But for larger apps it makes sense to split the storyboard into multiple storyboards and create a hierarchy of Routers to handle them.  Each "sub" router has it's own Storyboard and also has it's own `router`, injected by the instantiating router.  This allows the sub-router to notify it's parent when it is done.  It also allows the sub-router to forward requests it's not capable of handling.  When a parent router loads a sub-router, it will normally inject itself as the router's `router` (delegate) and provide some sort of "context" the sub-router can work in.  Very often, this context will be a Navigation Controller, but it could also be a Tab Controller or some other container.  

The parent-child router hierarchy works a lot like a responder chain.  Where a view controller (or some other object) may request some kind of routing behavior.  If the router can't handle the request, it should pass the request to it's own router/delegate, and so on.  This allows you to consolidate certain behavior into upper level Routers instead of duplicating code across view controllers.  As an example, instead of having each view controller display an Alert (for whatever case), forward that request to it's router.  It can then forward it up to the Master Router, who will then either handle the request or instantiate a sub-router devoted to displaying Alerts.  In this way, you can customize all alerts in a single place, maintaining the consistency across the app.  If you want to change how an alert is displayed, you need only change the code in one place.[^1]

### View Controllers
View Controllers are probably the most abused object type in the history of programming.  They seem to be the central repository of anything that not anything else.  I've seen view controllers balloon to 3k, 4k and even 5k lines of code.  This should not be.

View Controllers should control views and respond to user interaction.  That's it.  They operate as a layer between the user's interaction and the views themselves. As such:

 - No View Controller will be aware of any other view controller in the app.  
 - Even in the case of container view controllers, they do not know what kind of view controllers they're containing.  You should be able to switch out the view controllers without breaking your container.
 - View Controllers never instantiate another view controller.  Of course, if they're not aware of other view controllers then this is given.
 - View Controllers never initiate a segue. 
 - Never use `prepareForSeque`.  
 - View Controllers never store or cache data.

I always add at least one, but usually two different delegate properties to my View Controllers:

 1. A weakly held `router`, which the view controller can use to route interactions.  So, for instance, if the user hits the `back` button, the view controller will alert the `router` of this action.
 2. A strongly held `dataSource`, which the view controller will use to retrieve the data it needs.  The dataSource is almost always some kind of Logic Controller.

Both of these properties are defined *solely* by one or more protocols, which are defined in a separate file(s) and both properties must be injected, meaning the View Controller cannot instantiate either value.  This has the effect of making the view controllers configurable.  Because the `dataSource` and `router` are defined by protocols, any object can be used that satisfies the protocols, allowing you to swap out datasources or routers easily.  

Along with the dataSource protocol, I will also normally define transport objects I use to pass information to the view controller.  It's tempting to simply pass model objects to the view controller, but I warn against that.  I've often been able to reuse View Controllers to display different model types simply by using a different Logic Controller.  By tying your View Controller to a specific model class, you loose this ability.

### Views
Views are pretty obvious.  In iOS they almost alway derive from `UIView`, or `NSView` for Mac.  But they are still ripe for abuse.  The most common mistake I see is tying down a view to a Model object.  I've often see views that will take a Model object and configure itself according the data in that object.  This is bad design.  By doing this, you've essentially "locked" that view to the specific model.  If you wish to display any other data in the same way, you're forced to create a new view even instead of reusing the one you have.  Moreover, the *task* of filling view's with data is the responsibility of the *View Controller*.  This is what the view controller is *supposed* to do.  By having the view do this, you break even the basic MVC architecture.

However, I often see developers feel "forced" to use this pattern for table cells because they need to access some sort of "data" from within the cell, usually in response to a cell being selected or something similar.  If you 'must' store data in a view, do so by adding a generic, untyped (`id`) `data` property.  But really, consider your design paradigm.  Most of the time, it's better to request the data from the data source using the index path.

Generally, if you're passing some kind of Model object to a view you should probably re-consider what you're dong.  I do make one exception though: Asset Containers.  I often use a Asset Container instead of the asset themselves.  The container is generally paired with an Asset Manager, and *lazily* instantiates the asset in question by requesting it from the Asset Manager through a weak reference.  So, for example, if I have a view that cycles through a set of images, I will pass it a set of ImageContainers.  It will then request the images *as it needs them* from the container.  In this way, you can avoid loading up a bunch of images you don't need and the Asset Manager can maintain the asset cache and, more importantly *unique* the asset so it's not duplicated in memory. 

### Wireframes
In this case, we'll consider Apple's Storyboards and nib files a wireframe.  Wireframes can and often should be defined separately from a view controller so that often a view controller can have multiple wireframes.  Wireframes often utilize some kind of layout engine.  With Storyboards, this is "Auto-layout".  It doesn't have to be though.  You can create your own wireframe system and/or layout engine, but Apple's is good enough that I don't really need to explain a custom implementation [^2]

In the case of Storyboards, the Wireframe is doubling as a kind of rudimentary Dependency Injector by retrieving a predefined wireframe, injecting it into a View Controller and then returning the resulting View Controller.  It's insufficient, though, in that you cannot define any other injections which you will need to do for proper architecture.

**A word about Segues**: I avoid Segues as much as possible... in fact, I never use them except for visual annotation in a Storyboard.  While they are good idea, I think they are poorly implemented by Apple.  The primary problem is that a Segue *forces* a View Controller to become a dependency injector and creates static paths between view controllers.  It forces the developer to use bad architecture.  The remedy is easy: If we were allowed to set a `delegate` for a Segue, we could do the dependency injection in whatever object we choose (like, say, a dependency injector).  But that's not possible right now, and until it is I highly recommend you avoid Segues.  

### Logic Controllers as a Data Source
The basic definition of a logic controller is a class that encapsulates a piece of business logic.  In this case, we're using them as a datasource for our view controllers.  They will often sit "between" the view controller and one or more "Model Managers"[^3] and/or other logic controllers.  As a data source, they're responsible for responding to requests for data from a view controller and possibly for notifying the view controller (normally through a callback) of changes to the data.  They will often require injection of several class types (defined by protocols).  They not only provide an abstraction of the business logic from the view controllers, but they provide a convenient place to Unit Test the app's logic.  They will often maintain their own separate cache of objects and are responsible for converting the Model objects into transport objects that can be used by the view controller.

### Dependency Injectors
A dependency injector is essentially an object that is responsible for "providing" the required class objects another class needs to function.  There's a lot of information on dependency injection on the internet and why it should be used, so I'm not going to rehash that topic.  Suffice it to say that there is a wide range of complexity involved with creating a dependency injector.  It can be something as simple as a class container that's passed around and holds references to other class objects to be used as needed.  More functional dependency injectors will contain logic to inject the dependencies themselves where needed.  More advanced dependency injection systems are possible and may even be integrated into the OS/Framework (I believe Android/Java has one).  There are even whole 3rd-party Frameworks available for DI.  

I personally believe that a Framework is unnecessary.  Dependency injectors are easy to implement.  They'll often contain some kind of 'loading' method for loading the basic classes to be injected and then they'll have method to request certain object types along with optional parameters.  They instantiate the object in question along with the appropriate dependencies and return it.  This helps isolate "object creation" from "object usage".  And so long as you're using a protocol, you can swap out injectors for special use cases (Unit Testing, for example).  

## Conclusion
Using this architecture (or something like it) really helps make your apps adaptable, your codebase modular, and your components testable.  Moreover, it gives you a framework to build and maintain complex apps.
  
 In other posts, I will go over the other architectural patterns I use in developing apps.  I'll explain Model Manager, Persistence Layers, interacting with Server/Transport layers, etc.  




[^0]: Yes, it should always be capitalized. 
[^1]: Not many people will do this when `UIAlertView` is so readily available.  However, it still remains a valid and even desirable pattern.  By utilizing the Router responder chain, you don't *have* to rely on a `UIAlertView`.  And when `UIAlertView` is deprecated (ahem...iOS 8) you have only one place to implement the change.  More importantly, if you don't like the way `UIAlertView` presents itself, you can instantiate create your own... all in one place.
[^2]: Maybe I will in another post.
[^3]: I'll go over Model Managers in another post, but essentially they're responsible for providing model objects to the rest of the app, persisting the objects and keeping them in sync with any other persistence layers (a server for example).