---
layout: post
title:  "Interface Design"
categories: [swift]
---

One of the key pillars of good architecture revolves around _Interface Design_. The reason for this is deceptively simple: software is complex and we want to do everything we can to make it as simple and easy as possible.  Part of the entire point of OOP (Object Oriented Programming) is to abstract complicated code into objects and then hide that complexity behind a simplified interface. That way programmers don't need to know what's going on behind the scenes, and instead can deal with the abstraction. The interface _should_ be the only thing someone needs to know about that object to use it. The only time someone should even consider the implementation is when they're modifying it. This is an essential tool in battling the complexity of a codebase.

<!--more-->

Because the interface defines exactly how a particular object can and should be used, I consider it to include _both_ the coded interface _and_ the documentation for that interface. I normally exclude _external_ documentation from this definition simply because it's a PITA to constantly refer to an external source.  Most code environments will take inline code documentation and present it to the developer in some way (IntelliSense, autocomplete, etc), so it makes to me that this documentation can be part of the interface.  Personally, I include the documentation as part of the interface because no matter how expressive you are in your code, it's almost impossible to design an interface that makes complete sense without writing stuff like:

```swift
func aGetNetworkCallForClientsThatWantToOnlyRetrieveAUserObjectWithoutRetrievingTheUsersDetails(forId: String) -> User
```

_Sure, you probably don't need to add docs for that but please, please don't do this._

The rest of the article will focus on the actual function signatures and interface design itself, but I need to emphasize that documentation is absolutely essential and should always be included.

## Restrictive

An interface needs to be restrictive so that it's _hard_ to use it improperly.  Consider the following interface:

```swift
func updateButtonLabel(withId: String, toValue: AnyObject?)
```

The function signature is way too broad, and as such creates a lot of confusion.  Where do I get the button Id?  What are the acceptable types of values?  What happens if I pass in an empty string for an id?  What happens if I pass in `nil` for the value?  Does it clear the button? Does it crash?

Many of these questions could be answered in the documentation, but it still _allows_ the object to be abused... documentation doesn't prevent abuse and "that's their problem" attitude isn't really appropriate.  It's not "their" problem if they abuse your object, it's _your_ fault for designing a bad interface[^1].  Besides, the more a developer can rely on their tools to catch mistakes, the better their code will be.  Interfaces are an excellent way to accomplish this.  Compare the prior example to a more explicit interface:

```swift
enum ButtonId : String {
  case loginButton
  case submitButton
}
enum ButtonValue {
  case string(String)
  case number(Double)
  case clearValue
}
func updateButtonLabel(withId: ButtonId, toValue: ButtonValue)
```

This interface is much better and much harder to abuse.  Because you're requiring a `ButtonId` type for the id, it's impossible for the user of the interface to pass anything else.  More importantly, it's _discoverable_.  The user can very easily find out what buttons they _can_ update with this interface. Xcode will even provide auto-completion suggestions. Moreover, they know _what types_ of values they can use to update the button label.  It's practically impossible to abuse this interface.  Swift won't allow it.

Perhaps more importantly, this interface is much easier to understand if you've never encountered it before. This cannot be understated.  Code is written once but read many times.  Each time you're forced to look something up, read the actual implementation logic to understand what's going on, or accidentally use an interface wrong because it was poorly designed,  time and money is being wasted.  But even if it didn't, _it's damned frustrating_. I don't want to have to look through your implementation to figure out the available Ids. When an entire project is written this way, it represents hours and even days of lost time... and it's repeated every time a new developer comes in to the project or even when an existing developer comes back old code.

## Simple, but No Simpler than Simple

Put simply, the more complex something is the more _cognitive load_ it puts on the developer.  Let's take time aside and talk about our frail human condition.  Humans have energy.  Humans expend energy.  As humans expend more energy, they become tired.  As humans become tired, they make mistakes.  Humans must rest to get more energy. There's no getting around any of this. While you _can_ break the cycle (death), it's not really recommended.  Some people use stimulants to "get" more energy, but it's a temporary reprieve that's more akin to stealing from your future self, and doing that can damage a human in the long term[^2]. I know this sounds stupid obvious but thinking requires energy, a lot of it actually. It's remarkable how often people ignore this truth. You've got two options to solve this energy problem: expend less energy or rest more.

So moving on, complex interfaces require more energy to think about. This "cognitive load" will wear out a developer more quickly, causing them to make mistakes later on. While creating simpler interfaces requires more thought, and thus requires more energy early on, it pays for itself in the long run.  Because, remember, you write once but read many.  Creating simple, easy to use interfaces is a fantastic way to expend less energy later on.

Some interfaces will be complex because the object is naturally a complex object. But in most cases, complex interfaces can be simplified.  In many cases, making the interface more restrictive will also simplify it, so that's probably one of the first steps to simplifying a complex interface.  There's a lot of ways to simplify an interface and probably a lot of opinions on what constitutes a "simple" interface.  That's ok. There's always going to be some ambiguity here, but I think what matters is that you put in the effort _now_ in order to get the payoff later.  Still, let's take an example of a complex interface:

```swift
func cell(withId: String?, type: String?, title: String?, observer: AnyObject?, key: AnyObject?, target: AnyObject?, action: Selector?, config: AnyObject?) -> UITableViewCell
```

Yes, this is an actual interface[^3] I had to deal with in a project. There's a lot wrong here, but let's start with the fact this was the only way to create a new cell.  So I guess you could say it was "simple" in that regard. However, the signature is so obscure that it's almost impossible to figure out _how_ it should be used, what constitutes improper use, or what happens if I use it improperly.

What I eventually learned through hours of studying the implementation logic, was that there were several implicit "interfaces" layered over this one.  For example, if you wanted a button:

```swift
let buttonCell = cell(withId: kButtonId, type: kButtonType, title: "Do Something", observer: nil, key: nil, target: self, action: #selector(Myclass.doSomething), config: nil) -> UITableViewCell
```

So if you wanted a button, you had to use the function in a specific way. If you wanted a text fields, you needed to pass in observer/key values to keep it updated. Simplifying the interface here involves creating _more_:

```swift
func cell(withId: String?, type: String?, title: String?, observer: AnyObject?, key: AnyObject?, config: AnyObject?) -> UITableViewCell
func buttonCell(withId: String, title: String?, target: AnyObject, action: Selector)  -> UITableViewCell
```

That better, but we've still got a long way to go. Why is `withId` nullable? Turns out, it's because headers don't need an id, just a type:

```swift
func cell(withId: String, type: String?, title: String?, observer: AnyObject?, key: AnyObject?, config: AnyObject?) -> UITableViewCell
func buttonCell(withId: String, title: String?, target: AnyObject, action: Selector)  -> UITableViewCell
func header(withTitle: String) -> UITableViewCell
```

What about the observer/key thing?  That's tends to be used for KVO (and it is), but it requires both the observer and the key to be non-nil, otherwise it won't make sense. This can be fixed using a simple struct to pass in instead of separate parameters.

```swift
struct KVObserver {
  let observer: AnyObject
  let keyPath: String
}

func cell(withId: String, type: String?, title: String?, observer: KVObserver?, config: AnyObject?) -> UITableViewCell
func buttonCell(withId: String, title: String?, target: AnyObject, action: Selector)  -> UITableViewCell
func header(withTitle: String) -> UITableViewCell
```

Ok, so now what's `config`? Turns out, `config` was intended to be either an Array or Dict to pass into a Combo Box or Pick List. In fact, you may also notice that I'm no longer including `type` in the new functions, because that's explicitly define by the function itself.  Let's finish extracting them out:

```swift
struct KVObserver {
  let observer: AnyObject
  let keyPath: String
}

func comboboxCell(withId: String, title: String?, observer: KVObserver?, items: [String]) -> UITableViewCell
func pickListCell(withId: String, title: String?, observer: KVObserver?, items: [String:String]) -> UITableViewCell
func buttonCell(withId: String, title: String?, target: AnyObject, action: Selector)  -> UITableViewCell
func checkBoxCell(withId: String, title: String?, observer: KVObserver?) -> UITableViewCell
func textFieldCell(withId: String, title: String?, observer: KVObserver?) -> UITableViewCell
func header(withTitle: String) -> UITableViewCell
```

Now we've got a bunch of functions that are much easier to understand, but wouldn't it be nice to reduce those functions. In order to know what kind of cells you can create, you've gotta read through all the function signatures for the class.  Because really, the original idea wasn't all that bad: You want a single function to get a cell, but you need to configure it. What if we created a configuration object instead of creating multiple functions?

```swift
struct KVObserver {
  let observer: AnyObject
  let keyPath: String
}

enum CellConfig {
  case button(id: string, title: String?, 
  case textField(id: String, title: String?, observer: KVObserver?)
  case checkBox(id: String, title: String?, observer: KVObserver?)
  case pickList(id: String, title: String?, observer: KVObserver?, items: [String:String])
  case comboBox(id: String, title: String?, observer: KVObserver?, items: [String])
  case header(title: String)
}

func cell(configured: CellConfig) -> UITableViewCell
```

This is not only much easier to understand, but it's very auto-complete friendly.  Xcode will helpfully let you know all the kinds of cells you can create after typing period in `configured: .`.  Not only is this simple, but it's easier to use.  The developer doesn't need to dig around and figure out what configurations are possible.

## Explicit and Comprehensive

A good interface is explicit and comprehensive. It describes exactly what and object does and the object does _only_ what the interface describes.  Moreover, any external state dependency an object has must be provided by the interface. That last one's a little tricky, so I'll explain it a bit more below. There are certain corollaries to the interface object that each could (and maybe will) constitute separate blog posts:

- An object cannot reach into or modify global state. Doing so, would cause the object to create or respond to side effects that the interface cannot accurately describe or provide, respectively. For example, you can't create a variable like this: `let network = Network.shared()`, in order to retrieve data from a network.  It also means you cannot access global data like so: `var data = SomeSingleton.shared().getSomeData()`.  If you find you're "reaching" into some global space, then you're already breaking the interface contract.
- An object cannot modify _shared_ state or respond to changes in _shared_ state, especially in order to update another object.  This is pretty much the definition of a "side-effects".  It obfuscates the interface and makes it unreliable and incomplete as the object will be changing or responding to changes not explicitly initiated by the interface itself.  It also adds hidden coupling between objects, which can make a program very difficult to understand.
- An object can never use "notifications" outside of itself[^4]. This means an object doesn't "emit" a notification for others to use nor does  it respond to a notification from another object. Similar to shared state, responding to notifications create hidden interfaces between objects that be extremely difficult to discover.  I recognize this is probably a very controversial statement to may developers.  `NSNotificationCenter` seems to be a favored way to get disparate objects to "talk" to each other in iOS. But I know from experience that whenever I see this pattern of behavior, I know it's going to take me extra hours or even days to figure out what the code base _actually_ does completely aside from what it _says_ it does.
- And object function should do only what it says, and no more (no side-effect).

So, the idea behind an explicit and comprehensive interface is that any uninitiated developer should be able to read an interface and know everything an object does exactly.  They should never have to go into the implementation itself to understand an object's behavior... and that's the a big part of the point of having an interface in the first place... domesticated objects. They do what you want, when you want and never anything else.

Dependent state is _always_ passed in, so the creator of the object controls and knows exactly what the object is operating on. This is insanely important. Any object reaching into some global state can literally _do anything_ with it and there's nothing the creator of the object can do to stop or control it. Although this can be a point of confusion for some developers. This does **not** mean that dependent objects necessary to function must be passed in. For instance, a View that creates other views (ex: a Text Field) to operate doesn't need those passed in[^5]. So long as the object it's creating is done so within the limited context is was creating in, it won't be able to do anything outside of it's scope assuming those objects also adhere to these rules.

And here's the crux of the matter: An interface should define what the object's "scope" is.  In other words: what it can do, and the data it can do it with. As soon as an object reaches beyond the scope given to it, all bets are off.  At that point you must read all the implementation details to understand the project and good luck with that when the project reaches 100k+ lines of code.

This may seem extreme, especially if you come from an environment where shared update "magic" is commonplace. Often, it's _easy_ for a developer to create a notification, switch to another object to "listen" to that notification and viola! everything works together in harmony. Shared state, frequently presented as some kind of store data models, can allow you to _easily_ modify and update your data models while other objects are automatically responding to and updating themselves from your changes.

Unfortunately, the problem only arises as the project grows larger. Easy to code does not translate into "easy to maintain". The more hidden behavior and side effects a code base has, the more difficult, time consuming and expensive it will be to maintain. Each hidden interface becomes something that all developers must _discover and remember_ whenever they deal with that component. But we developers are human and our memory is horribly ill-equipped to deal with that level of minutia.  We most likely won't remember all the hidden dependencies when dealing with some one component out of hundreds (or even thousands). The changes we make may or may not break some hidden dependency.  This is how regressions and "holy shit" bugs happen. The developer forgot about some hidden dependency or side-effect, changed a line of code to update a feature, and sometime later someone else (it's always someone else) realizes the app is crashing, leaking data, or misbehaving in some way because of that thing the developer forgot to remember.

## Conclusion

There are habits of behavior for many developers ingrained from a culture of "move fast and break things", especially in the mobile space. It demonstrates a bifurcation in the industry as a whole. There's been a strong focus for much of the industry to just "get shit done" as quickly and as efficiently as possible. This has led to large proliferation of "magic" frameworks that help developers to do just that, with as little friction as possible. A developer that can create an "app" quickly and efficiently is lauded, praised and rewarded. But as the mobile industry has matured, more and more companies are seeking mature, large-scale projects. These projects require a completely different set of priorities and skill sets. Maintainability,flexibility and expandability become much more important than startup efficiency. Proper architecture, design and coding standards become paramount to keeping a project on task and performant. At this scale, the magic frameworks and move-quick mentality start working against the developer. There's too much to remember.  Architecture, design patterns, and good coding practices start becoming much, much more important than the frameworks used in the app. 

While many of the restrictions and policies I advocate here will seem extreme to many mobile developers, they're designed to allow developers to create much more comprehensible and scalable apps.  Unfortunately, they very well may completely exclude certain types of frameworks and habits that many developers have relied upon to do their job quickly. But I can attest personally that once they become habit, adhering to these policies does not slow you down. Instead they do give you the framework needed to provide truly scalable and maintainable projects.

[^1]: Wanna a great example of this?  Google for "2018 hawaii false missile alert" for a great example of how a poor interface can wreak havoc on an entire state.

[^2]: I still love my coffee though.  Not going to give it up.

[^3]: It was actually objective-c, but I'm showing it ported into Swift for consistency here.

[^4]: Well, within reason.  There are certain system notifications that you have to pay attention to.  For example, keyboard notifications on iOS are essential to designing a responsive UX, and those notification are the _only_ way to do this.  

[^5]: Of course, the objects it creates also shouldn't be reaching into the global state either.