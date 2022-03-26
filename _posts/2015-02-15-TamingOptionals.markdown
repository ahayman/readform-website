---
layout: post
title:  "Functionally Taming Optionals"
categories: [swift]
---
There are a lot of blog posts and probably even more opinions about Swift.  I will continue this short, but certainly worthy trend.  

One common opinion I keep hearing is about how the Optional system kind of sucks, and is perhaps better in theory than in practice.  I would like to take a <s>brief</s>, slightly inebriated moment to defend Swift's optional system and provide some tricks I have found that greatly help to tame the optional system and provide a path out of "if-let nested hell". [^0]

But first, let's talk about why Swift's Optional is truly great.  Many other peopld have spoken of this and Apple itself has loudly proclaimed the benefits.  The shell of the nut, though, is this: Optionals force you to deal with the possibility that a value doesn't exist.  If there is a variable (or parameter) and it's not an optional, you can be sure that variable exists.  If it is an optional, the only way to access that variable is to account for the possibility that the value doesn't exist.  After working with my own frameworks and creating apps for client in Swift I will say this: ***it works***.  Yes, it may be annoying to deal with at times, but after creating an app in Swift I found there were several types of bugs and crashes that *simply did not exist*.  All of my bugs were functional in nature[^2].  I never had a bug where I thought I was dealing with one Type but instead it was another.  Not once did my app crash because an object was nil.  Whatever difficulty the Optional system might entail, it is well worth the price of admission, IMHO.

<!--more-->

Unfortunately, Apple has also provided a way to completely undermine this system with a very simple operator: `!`.  My adive: never use this unless there is any other way.  Never bang an optional.[^1]  Think of it as a "system unstablizer", an efficient and excellent way to make your program unstable and crash.  Use the optional system.  Embrace it.  It is your destiny and it will lead you to greatness.

If only you could get around "if-let nested hell".  Anyone who has programmed more than two lines in Swift know what I'm talking about, and it's led many a developer to exclaim, "fuck it": `var x = y!` and be done with it.  The problem is that Swift is a different paradigm than objective-c is, but a lot of developers are treating it like "objective-c without the c" or "c++ without the c" or, well, fill in the blank: "x with/without y".  By doing so, they corner themselves into frustration.  Swift requires that you think differently and solve problems differently than in Objective-c or many other languages.   Yes, folks, functional programming is in vogue and we should learn to use it.

First, a caveat:  I've been writing this particular blog-post over than span of a week or so and since I started Swift 1.2 Beta has come out.  They address one very big aspect of "if-let nested hell" (or, if you wish: pyramid of doom) by allowing you to assign several `let` variables in a single statement:

	if let x = y, a = b, c = d {
	  //Do something spectacular here
	}

This is a *huge* help, as I'm sure most of you know that would be 3 nested if let statements in the current version of swift.  But even in the current version of Swift, *it doesn't have to be*.  It's astonishingly easy to use generics to create your own conditionaly multi-variable if let statement.  So, without further ado, meet the `comb` function [^3].

	func comb<A,B>(a:A?, b:B?) -> (A, B)? {
		if a1 = a {
			if b1 = b {
				return (a1, b1)
			}
		}
		return nil
	}

To be fair, the most inelegant aspect of this was that I had to copy and paste that code about 6 times to overload it with more generics: `func comb<A,B,C>(a:A?, b:B?, c:C?) -> (A,B,C)?` But once it was done it was done.  Yes, it would be nice to be able to set a variadic generic, but honestly I'm not hopeful that will happen or even certain that it should.

The results is that you can now flatten out your if-let pyramid of doom into:

	if (x, a, c) = comb(y, b, d) { }

[^4]  
This is great, and it's something I used constantly in Swift.  But, to be honest, I'll probably abandon it with Swift 1.2.  Swift also has the `where` operator.  While it's possible to replicate that particular paradigm into it's own functional statement, you're starting to get to a place where you're duplicating the language's features, which is probably not a good idea.  Still, I kind of want to prove it's possible, so let's write a kind of `where` infix operator:

    infix operator <-? { associativity left }
    func <-? <A,B>(v:(A,B)?, f:((A,B) -> Bool)) -> (A,B)? {
      if let value = v {
        if f(value) {
          return value
        }
      }
      return nil
    }

Infix operators are limited to certain characters which do not include the alphanumerics, so I've attempted to be as "expressive" with those characters as possible.  I'm also not going to explain what an Infix operator is, other than to say that it's like a mathmatical operator: it has a left hand side, a right hand side and does something with those two values.  Apple has plenty of documentation, so if you're unsure go look it up.  I'm not a dictionary.  The `where` operator we've now defined takes a tuple for it's left value and a closure for it's right value.  The closure itself takes the same tuple and returns a `Bool`.  Now in the implementation we unwrap the left value and pass it to the function.  If the function returns `true`, we return the value, otherwise, we return nothing.  Now we can take the `comb` function and use with our new Infix operator `<-?` to replicate Swifts `where` funtionality:

    var y:Int? = 20
    var b:Int? = 10

    if let (x, a) = comb(y, b) <-? { x, a in x == 20 } {
      println("x = \(x), y = \(a)")
    }

Compare this to:

    if let x = y, a = b where x == 20 {
        println("x = \(x), y = \(a)") //Swift 1.2 and greater only
    }
  
Our `<-?` isn't nearly as concise as Swift's `where` but it works.  Moreover, it's a closure and as such can actually be much more flexible than Swift's `where` clause.  We could stick a complex, multi-line calculation in the closure and it would work just as well.  I'm pretty sure you can't do that with `where`.  Still,  `where` *is* much more readable.

My point is this: The functional aspects of Swift [^6] are often the answer to the problems we're having in Swift.  Or let me put it another way: The reason we're having so many problems with Swift ***is because we're not using it properly***.  If we want to use Swift well, we *need* to learn functional programming, we've gotta understand the generic system and how it can be leveraged, and we need to embrace the "Optional".  If we don't, we're going to run-aground again and again by trying to fit the square habits we've developed in Objective-c [^5] to the round hole that is Swift.

One more example and I'm done.  Let's consider the problem of "piping" that often leads to nested if-let statement.  Essentially, you have a value that must successfully pass through a series of transformations in order for it to be usefull.  The best example of this is translating data returned from a server.  Let's say you need to pull a "date" value from JSON returned from a server.  Assuming you've already translated the JSON into a `dict` object. 

	if let dateString = dict["date"] as? String {
		if let date = self.dateFormatter.dateFromString(dateString) {
			// Do something with the date
		}
	}
	
This is a simple nest, not overly horrendous.  But what if you want to use the result in conjunction with other values you grap from the JSON?  All of a sudden, you're *forced* to place everything inside this `if let` statement.  What happens if you require two date?  You've just set the foundation for a pyramid of doom.  Instead, let's flatten out this nest:

    infix operator ?<? { associativity left }
    func ?<? <A,B>(a:A?, f:A -> B?) -> B? {
      if let a1 = a {
        return f(a1)
      }
      return nil
    }

[^7]  
So what does this do?  It takes an optional as a right hand value and a function as the left hand value.  The function takes a value and returns an optional.  The implementation of this operator simple unwraps the optional, and if a value exists then applies that value to the function and returns the result.  Simple, and yet is allows you to "pipe" optional information through a series of functions.

	if let date = dict["date"] as? String ?>? self.dateFormatter.dateFromString {
		// Do something with date.
	}
	
What's important here isn't that we've flattened the nest.  That's nice, mind you, but the real value comes from using this in conjunction with other operators:

	if let name = dict["name"] ?>? JONString,
			 date =  dict["date"] ?>? JSONString ?>? self.dateFormatter.dateFromString,
			 age = dict["age"] ?>? JSONInt
			 where NSDate.timerIntervalSinceReferenceDate - date.timeIntervalSinceReferenceDate > 60 * 5
		//Do something with the name, date and age if the date is within the last 5 minutes
	} else {
		// Post error?
	}

Note: I have a series of functions I use to translate JSON data (ex: JSONString, JSONInt, JSONDict, JSONArray).  I use these to sanitize the data I get from the server because, quite frankly, server devs are <s>capricious</s> under a lot of pressure and sometimes change the data format of the values they provide.  So I do everything I can to translate the values into something I can use.  

My point here is this: The above code would have resulted in an almost ludicrous amount of nested statements without leveraging the functional aspect of Swift.  Swift's optional system can be a real pain.  But Swift has also given us the tools to manage that system.  And here's the kicker:  It's still safer than objective-c.  I know exactly what's going on and exactly what kind of values I will get.  Nothing is assumed, and I can't count the times I've been caught with my pants down in objc because I assumed I was getting one value type but instead I got another.  Swift is better (IMHO), but it requires you to learn a different paradigm.  Learn that, and you'll be set.


[^0]: It should always be presented in qoutes, necessary to impart the proper gravity of the situation.
[^1]: I know, it's horrible but hell, *it's memorable*.  I really, really want this phrase to catch on.
[^2]: I've also had bugs with the Swift compiler (not to mention the extremely irritating SourceKit Editor crash).  But this is improving a lot.  I've been playing around with Swift 1.2 and it's convinced me that Apple really is serious about  Swift.  
[^3]: `comb`, short for "combine".  No, it's not very original but dammit I can't think of a better name and I felt like `combine` was just too long for something I would be using all over my codebase.
[^4]: If you're being lazy and want to make your code more unreadable, you can also assign the statement like so: `if let v = comb(a,b)` but then you've gotta reference your values as `v.0`, `v.1` etc.  
[^5]: or `c` or `C++` or `Java` or fill in the blank with any non-functional programming language.
[^6]: And they are *just* functional aspects.  Swift is ***not*** (please note emphasis) a functional programming language.  If you want a functional programming language, try Haskell.  If you want to develop an iOS app in a functional programming language... well, you're pretty much SOL.  Swift is an OOP language with functional aspects.  Personally, I think this is great.  I always buy into pragmatism before I'll by into an idealogy, and "Functional Programming" really sounds a lot like an ideology when people pitch it.  Swift, though, is trying what Scala does: rationally mix OOP with FP... except Swift is positioned to truly introduce FP into the mainstream through the incredible power of Apple marketing.  Rather than diluting both worlds, I think each side enriches the other.  Solving a problem never resolves itself down to a simple paradigm, and by providing a careful balance of FP and OOP, I think Swift can provide a much richer set of solutions than can OOP or FP alone.
[^7]: Please note that the function name `?>?` is purely of my making.  I've actually tried several different names in various different Swift apps and have yet to settle.  Swift is seriously in need of some naming conventions. I used to use `>>?` and I just might go back to that.

