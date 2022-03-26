---
layout: post
title: Routing in SwiftUI
categories: [ios, swift]
---

I probably should have waited for the next iteration, but I was starting a new side-project and, well, there was that checkbox to use SwiftUI staring at me, taunting me, luring me in with the sweet promise of unending UI layout bliss.  So I scrunched up my face, clicked the box, waited a minute for the trumpets to die down, and then created the project.

This is not a thoughts post on SwiftUI.  It's too early for that.  Besides, there's plenty of posts out there on the interwebby thing if you want an opinion.  Plus, I want to wait until version "2" comes out for, you know, when Apple has had a chance to take their beta and turn it into a real boy.

Instead of creating a "thoughts on SwiftUI" post, I'm going to wrap my current thoughts on SwiftUI into what I hope is a somewhat comprehensive — or maybe just an acceptable — discussion on one piece of architecture: Routing. That's safe, right? It's not like SwiftUI isn't going to suddenly change their whole paradigm this year.

Okay, enough with the disclaimers; let's talk about Routing.

<!--more-->

If you're familiar with architectures, Routing is the R in VIPER and sometimes called a "Coordinator" in other patterns.  It's basically abstracting screen navigation and lifting it out of the Views [^1] into a component designed to handle it. All screen navigation is done in a Router, thus removing from any View the need to handle such things.

A Router essentially de-couples a View from its navigation and presentation, freeing it to focus _purely_ on the view itself.  It is, in my opinion, the natural conclusion of SRP.  A view should only be concerned about the view and nothing more.

> But why?

While I consider Routing an absolutely essential part any project, I've found that many developers, even really good developers, don't always agree with me on this. It's not that Routers are controversial; it's more like most developers just don't think it's a big deal... and, you know what? I _get_ it. 

The problem that Routers solve isn't like the whole "Massive View Controller" issue; it's not staring us in the face in the form of thousands of lines of code.  Navigating from one screen to the next is _easy_, and it's usually just one or two lines of code.  It's not satisfying to set up an entire abstraction only to replace those two lines of code with another two lines of code. Why make a lot of extra work for yourself?

In end end, I think it's one of those things where you need to get burned at least once before you realize just how important Routing can be. For me, realization came in the form of a simple user story: When the user taps on a URL on our website, the app should open, and then it should navigate from wherever the user is to the screen dictated by the URL. 

You wanna see developers panic?  Give them that requirement for an app with dozens of screens. And trust me, the client won't understand why their “simple” request will take months of work. 

Unless, of course, you've already got a Router, in which case you don't need to refactor dozens of View Controllers, but only need a few days to parse the URL and direct the Router where to go. 

All this to say, one of the first things I did in my new SwiftUI project was to ask: how can I route views?

The answer is [spoilers]: not easily. Or, at least, not easy to figure out (for me at least).

SwiftUI is a declarative, state-driven framework. What this means is we describe the UI as list of types and function calls, and then use state to update it; but the actual layout and display of the views is done by the framework. [^2] But there's more:

> Navigation is no longer event driven. Navigation is now state-based.

Huh? Pray-tell, what does that even mean?

Well, first, it means that you _don't_ navigate to a new view by telling the navigation controller to push a view controller onto its stack.  Instead, you bind a view to state, and then update the state if you want something displayed.  Everything is done by state.  The UI, layout, navigation, etc are all _derived_ from that state.  If the state changes, updates (including animations) are updated accordingly. 

If this feels like a round-a-bout way of doing things, it is. But it also removes a ton of code for the developer to write. You don't _need_ to worry about _how_ things are done. Just update the state; it's that easy, I swear; please sign on the dotted line.

The problem in SwiftUI, is the state is inextricably bound to the view, and Routers need to handle routing separate from the view... which is now literally part of the view's state.  Ugh.

```swift
var body: some View {
    VStack {
        ...
        List(items) { item in 
            NavigationLink(DetailView(item)) {
                ListItemCell(item: item)
            }
        }
    }
}
```

What's going on here?

Navigation is no longer a call, but literally part of the view structure.  When the user taps on whatever view it's wrapping (in this case: `ListItemCell()`), it will update the internal state so that the appropriate view will be rendered onto the stack.  There is no way for you to directly say: "please navigate here".  There's no `self.navigationController?.pushViewController(vc, animated: true)`. Those calls don't exist; it's an itchy feeling, I know. [^5]

In UIKit, I'd normally have a View Controller emit events (usually an enum) that the router could subscribe to via Combine or some Rx framework.  The Router would then route to the appropriate screen based on those events. Easy peasy. Here, though, SwiftUI has tightly bound the view not only to the "Detail Screen", but also to the type of navigation it is embedded in.  This is a big problem.  What if you need to put that screen in something that's not a navigation stack (a tab view, for instance)? [^3]

The answer I came up with is... tentative.  I'm unsure if this is what I'll use going forward, but I think it's a good first try. Since views now demand state for navigation, I use the router to inject whatever is appropriate into the view:

```swift
protocol Router {
  associatedtype Route
  func viewFor<T: View>(route: Route, content: () -> T) -> AnyView
}
```

The router is defined by a protocol, which the View will require in the constructor.  The `viewFor(route: _, content: _)` function should take a 'route', wrap the content in whatever navigation is appropriate for the route given, and return it all in an AnyView. [^4]

The view, then, can define the route and router it needs using generics:

```swift
enum ListRoute {
    case detailView(data: Item)
}

struct ListView<ListRouter: Router>: View where ListRouter.Route == ListRoute> {
    let router: ListRouter

    var body: some View {
        List(items) { item in 
            router.viewFor(route: .detailView(data: item)) {
                ListItemCell(item: item)
            }
        }
    }
}
```

Note: The `ListRoute` enum must be defined outside the ListView.  If you scope the enum to the ListView instead (something I like doing), you essentially create a type that "cannot" be created in swift: The Router needs to conform to the ListView router type (`ListRouter`), which requires the enum definition.  But you can only access the enum definition by referencing its scope, and since that scope is a generic, the generic must be fully typed (ex: `ListView<Router>.Route`), a type that depends on the very router you're trying to define. If you try, you'll get the error: `Type alias ListRouter refers to self`.

The last thing I need to do is create a router but there's a problem: normally, I'd create a `MainRouter` class and then use it for all my routing needs [^6].  I can _still_ do that, but I cannot pass that main router into the view.  The type system will only allow a single generic type to be passed in:

```swift
struct ListRouter: Router {
  typealias Route = ListRoute
  var usingStackNav = false
  
  func viewFor<T>(route: ListRoute, content: () -> T) -> AnyView where T : View {
    switch route {
    case .detailView(let item):
      if usingStackNav {
        return AnyView(NavigationLink(destination: DetailView(item)) {
          content()
        })
      } else {
        return AnyView(Button(action: { /* Update state, maybe set tab view to item */ }) {
          content()
        })
      }
    }
  }
}
```

The generics constrain each concrete router to a single, known type. Thus, I cannot use the same class/struct for both a ListRouter and another type of Router. This forced me to break up my routing logic, which means I'll probably end up with a main router that shares state with and manages a bunch of smaller struct routers. At first, I didn't like this. It's forcing me to break into pieces something I normally keep whole, and for small projects that kind of decomposition is overkill. But routing logic can get very complex, especially with larger apps with a lot of dependencies and many screens. In those cases, I'd have to break up my logic _anyway_, and while it's annoying to be forced into this kind of decomposition, I'm tentatively hopeful it will end up being a non-issue.

You may notice I haven't talked about how to handle the routing itself. The answer why is simple: I haven't gotten there yet. My biggest first step was to see if I could lift out the routing from the views and that is exactly what I have done. How best to manage the routing itself will require experimentation and likely another blog post.

There are probably a better way to handle this, but for now what I have is very composable and it decouples views from both each other and from the navigation. I suspect there's gonna be a lot of trial and error as we adapt to this "new" paradigm and learn what works best.


[^1]: or View Controllers, but for now let's just assume "View" means both View and View Controller.  SwiftUI has gotten rid of the idea of View Controllers anyway, no matter what going on under the hood.
[^2]: This paradigm will be instantly familiar to anyone who's use React Native; syntax aside, the paradigms are the same.  But whereas React Native has had years to work out navigation mechanisms, SwiftUI isn't even a year old. Many navigation frameworks (aka React Native Navigation or React Native Flux Router) hide some of this to provide a more familiar call-based, event-driven like interface. Peek beneath the scenes, though, and you'll find they're just updating state.
[^3]: And here you argue: But how often does that happen? I mean, when are you navigating to a different tab for a detail view? Fair enough, probably not often. But let's not confuse the example with the point. There's been lots of times I've needed to do this: Settings, Login, and the Detail Screen itself may need to be shown in an overlay/modal, a tab view, a _different_ stack than the one you're in. This happens a lot in larger apps, where one part of the app needs to display some context (a message, for example) normally displayed in another part.
[^4]: I do not like using AnyView this way, but I spent quite a bit of time trying to figure out how to pass back something more specific and kept butting heads against Swift's type system.  I may try again later, but this works for now.
[^5]: Okay, I lied; they actually do exist, for now at least. SwiftUI is using UIKit behind the scenes. Theoretically, you could "pierce the veil" and make explicit calls. You _could_, but I don't recommend it. There's no guarantee this will continue to be the case and doing so breaks the whole SwiftUI paradigm.  If you're going to use it, then please, _use_ it. 
[^6]: In larger apps, I'll create a hierarchy of Routers to manage different modules.