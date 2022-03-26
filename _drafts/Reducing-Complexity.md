---
layout: post
title: Reducing Complexity
categories: [architecture, react-native]
---

I recently had a chance to refactor quite a bit of React Native code for client. The impetus for this came from errors produced by an old, out of date package (react-navigation). The new version did away with support for RN's old component-based model in favor of the newer functional model plus hooks. It was a fantastic opportunity to rewrite several dozen components [^1].

This is _not_ a post about that.  Maybe I will, later — functional components are an interesting shift — but for this post I'm focusing on one very specific hook [^2]:

```ts
const [state, dispatch] = useReducer(reducer, initialState)
```

The idea of a reducer isn't new, but it's generally been a concept limited to one part of Redux. Redux is a beast I have no intention of discussing; there's plenty of articles out there about it. The reducer concept, though, is actually fairly straight forward.

But before we dive into that, I'd like to talk about something I call "incidental complexity".

<!--more-->

## Incidental Complexity

Definition: Complexity that arises not from the problem itself, but from the tools used to solve the problem.

Let's take an example from native iOS. You have a complex view that needs to display an arrangement of tiles. For whatever reason, you can't use a Collection or Table view. You've gotta do it yourself [^3]. Calculating something like this requires a bit of intermediate state which, while necessary, should not be updated outside of the calculation itself. 




[^1]: I swear I got them all. I swear this every time I find an old component I cannot for the life of me understand how I missed.
[^2]: Okay, so a little bit about hooks. Functional components are, quite literally, just functions that return a layout. That's it. Because of this, every time the app needs to render something new, the function is called all over again, causing everything in the function to be recreated. This means there are no instance variables, and thus you can't hold state. To remedy this, RN created hooks. Basically, instead of you holding state in instance variable, RN does it for you. At it's core, a hook provides: "state" and a function to update that state. That's it. To update the state you call the function. On every "render" (or function call), it guarantees the "state" you get is up to date and consistent. 
[^3]: This isn't a great example, but it's the one in my head right now.
