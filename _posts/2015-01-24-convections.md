---
layout: post
title:  "In Need of Some Conventions"
categories: [swift]
---
There are three things I told myself I would not do when I started developing in Swift:

1. I would never bang an optional.  I think the optional system, though sometimes annoying, is a great idea.  Banging optionals is dangerous... as I'm sure my wife would agree.
1. I would never use an emoji for a variable.  Seriously, just because you can do it doesn't mean you should. Not even the poop emoji.  It's only funny once and then you have to copy and paste the shit all over the place and no one likes that.
1. I would avoid custom operators for the sake of anyone else that might look at my code.

Unfortunately, I've only managed the first two, even though Xcode persistently insists on banging all of my `@IBOutlets`.  And seriously, why?  Why do they do that?  For the life of me I can't think of a single reason aside from sheer laziness.  I've often used the same View Controller for several different Wireframes, many of which don't have the same views or configuration... thus, `!` is stupid and dangerous.  It always is and always will be in Swift... don't use it.  

Arg... I got sidetracked.  Damns bangs.  

<!--more-->

Of those things I sought to avoid, avoiding custom operators has been a complete failure.  The problem is this: custom operator *are awesome*, especially in a functional language like Swift.  But even more than being awesome, they're *necessary*.  Don't believe me?  Try parsing a deeply nested JSON structure without custom operators. To add insult to injury, try doing this *and* requiring that values x, y and z be present.  You'll be so neck deep in `if let` statements not even a backhoe will get you out. In my foray into Swift JSON parsing this function has been my best friend:

    infix operator >>? { associativity left}
    func >>? <U,T>(opt: T?, f:T -> U?) -> U? {
      if let x = opt {
        return f(x)
      }
      return nil
    }

What does it mean?  Simply this: If the optional on the left returns a value, apply it to the function provided on the right (since this is an infix operator).  Simply put, this has allow me to take deeply nested `if let` statements and place them on a single, coherent, conditional line/statement.  I could not live without it.  That, and in conjunction with something I call the "combine" function (`comb`) to combine multiple `if-let` statements into a single tuple if all the optionals return a value:

    func comb<A,B>(a:A?, b:B?) -> (A, B)? {
      if let x = a {
        if let y = b {
          return (x, y)
        }
      }
      return nil
    }

Unfortunately, I have to overload this function multiple times to accommodate varying number of parameters I want to combine (I'm up to seven at the moment).  It's painful, yes, but only painful once.  I suppose, in the grand scheme of things, I shouldn't complain... but I probably will in some other post.

Back the the point (again)... what the hell is the `>>?` operator?  It's something I made up.  The symbol is, for all intense and purposes, arbitrary even though I posit that it makes perfect sense.  Pass the variable to the operation on the right (`>>`) **if** the optional returns a value `?`.  And thus we get: `>>?`.   Yes? Or should it perhaps be: `?>>`.  Hmmm.... but I've already written code, so fuck it, we're sticking with `>>?`.

And yet this is a kind of operation I find essential in Swift.  The problem is, there is not yet a standarized convention for these kind of operators in Swift and at the same time a strong need to have these kind of functions.  My code is much easier to type because of this operator, but it's also much harder for someone else to understand.

Don't believe me? Do a search for "effecient Swift JSON" and at some point you'll come across this culmination of effeciency:

    static func decode(json: JSON) -> User? {
      return _JSONParse(json) >>> { d in
        User.create
          <^> d <|  "id"
          <*> d <|  "name"
          <*> d <|? "email"
      }
    }

    
Do you have any idea what that does? Of course not.  No cheating.  Don't go looking up the artice first or find the source code. Without which you haven't a clue what this does.  I count 5 seperate "arbitrary" symbols that could mean anything. Sure, you could try to decode `<|?`.  But seriously, no... no you can't.  And for that matter, why the hell does the `User.create` accept any of that shit in the first place.... oh, because it's a `curried` function.  That's why.  Didn't know that?  Oh well, emoti poo on you.  

Swift is in serious need of some conventions.  For some reason people don't seem to remember that C++ was a complete clusterfuck of unreadable code when it first came out due to the ability to create random and arbitrary operators combined with developer enthusiasm. (and no, I don't know this personally... only through reading article of what I posit are really smart people saying really smart things).  The point I'm making is this: C++ doesn't have optionals. Optionals are great, they really are. But they force the need for custom operators waaaay more than C++ ever did.  C++ developers created custom operators because they wanted to and it was fun (and often succient, even if not consistent).  But in Swift, you really *need* to create these custom operators in order to write semi-decent code that doesn't have 20 layers of `if let` statements.  I'm not kidding here.  Consider these 20 layers of `if let` statments in conjuction with the requisite `else <error code>` and just looking at the repition will give you tendinitis.

Swift either needs a clear set of conventions for creating custom operators or else it needs some of these problems, ie - nested `if let` statments, solved in the language.  Otherwise, we're going to end up with a clusterfuck of conflicting custom operators from variously smart and good intentioned developers attempting to write good code.

