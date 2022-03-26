---
layout: post
title:  "Architecture: The Wrong Discussion"
categories: [architecture]
---
<p>Over the past few years, I’ve seen an increasing interest in software architecture as an important, perhaps now even an integral, part of software development. While this may seem obvious now, it’s still surprising to me just how much architecture is considered as an afterthought by many developers. At least in the mobile space, it’s taken a long time for architecture to enter the foreground of the conversation for software development. But even as the discussion about software architecture grows, I’ve noticed that the conversation tends to center on individual architectures as a <em>solution</em> to specific problems inherent in software development. You can see detailed descriptions of <a href="https://www.objc.io/issues/13-architecture/viper/">VIPER</a>, <a href="https://www.objc.io/issues/13-architecture/mvvm/">MVVM</a> (and it’s variants), the ubiquitous <a href="https://developer.apple.com/library/content/documentation/General/Conceptual/DevPedia-CocoaCore/MVC.html">MVC</a> as well as a scattering of others for app development. While each of these solutions do address specific set of problems <sup><a id="ffn1" href="#fn1" class="footnote">1</a></sup>, what they do not do is teach developers how to create their own architectures to address problems specific to their own development. As such, many developers will try to use an existing architecture to solve the wrong problem, or else failing that, to forget about architecture altogether and hobble along as best as they can. </p>

<p>While I have been pleased to see the conversations on software architecture, I think to a certain extent they are the wrong discussion. We’ve been discussing the benefits/drawbacks of individual architectures someone else created. What we should be talking about are the principle of architecture that allow us to create the architectures we need.</p>

<!--more-->

<h4>Attributes of Architecture</h4>

<p>The main problem most developers have when using any specific architecture is they haven’t yet defined exactly what problems they’re trying to solve and the <em>attributes</em> of the architecture they want to use. Without defining these attributes, you get the attributes of whatever architecture you use, whether you want them or not <sup><a id="ffn2" href="#fn2" class="footnote">2</a></sup>. The problem is that so many developers simply accept whatever attributes happen to come out the code, or by adhering blindly to an architecture they haven’t thought through. Eventually, the downsides of their architecture (or lack thereof) overcome their ability to produce. Creating a good architecture requires forethought into the desired attributes of the system you’re trying to create.</p>

<blockquote>
<p>All architectures involve a compromise of attributes.</p>
</blockquote>

<p>Need an app that’s extremely fast (perhaps realtime?), you’ll end up sacrificing testability and maintainability. Conversely, the most maintainable apps may not be suitable for realtime data processing. Developing a MVP (minimal viable product) isn’t normally done with more advanced architectures, sacrificing things like maintainability and testability for development speed. All architectures involve some sort of compromise of attributes, and so the first task is to define what attributes you want and their priority. Only then can you make the appropriate decisions for your architecture. </p>

<h4>Balance and Conflict</h4>

<p>It is not often that the attributes you wish to gain from architecture are in harmony. More often than not, they will conflict with each other, requiring you to prioritize and balance the attributes. As developers, we rarely have the opportunity to devote as much time to a project as we would like. So the speed of development must be balanced against testability, maintainability and whatever other desirable attributes you want from your architecture. </p>

<p>This in itself should demonstrate just why it’s important that developers be able to modify their architecture to their needs (or the needs of their client). VIPER is a heavy architecture for many projects, but MVC is far often too light for anything but the smallest projects. At the other end of the spectrum, in my current job, VIPER is way too <em>light</em> of an architecture. We use something much more complicated because it’s a large, long term project that requires something much more complex and VIPER is simply to small to our needs. Our architecture is more a living thing that evolves and changes to the shifting requirements of the project. Attempting to use anything less would result in an inordinate amount of rework, chasing bugs, and stalled development. Attempting to stick with a static architecture, no matter how good that architecture is, will eventually result in the same. Being able to modify or create an architecture to your needs can not only be very advantageous, but also quite necessary. </p>

<h4>Moving Forward</h4>

<p>There’s a lot to the discussion about how to create, modify and evolve <sup><a id="ffn3" href="#fn3" class="footnote">3</a></sup> architectures. While I cannot go over all of them (there’s lots of books that can help you do this) what I hope to do is go over what I’ve learned and the principles that <em>currently</em> matter to me (as related to my current project) as an ongoing series. </p>

<p>I’ll be posting a series of posts explaining my rationale behind certain decisions, principles I try to adhere to, and general explanations of the architecture(s) I use. There will likely be considerations I don’t make, that some people will need to consider. For example, <em>concurrency</em> isn’t a major consideration in our project aside from making sure we don’t block the UI. As such, it doesn’t play a strong role in our architecture. But it is a large project, so <em>maintainability</em> is huge, and I will spend a lot of time discussing that. Maybe once I’ve exhausted the principles that I’m most concerned with I’ll move onto speculative considerations of other architectures, but my main goal isn’t to get familiarize you with all possible considerations. What I hope to do is inspire you to think about this. It’s the discipline of thinking through the issues that will truly serve you well.</p>

<p><em>Note: I have not been the best at keeping up my blog. This is in no small part due to having a now 6 month old, a major surgery <sup><a id="ffn4" href="#fn4" class="footnote">4</a></sup> and the general struggles related to working a full time job and having a family. That said, I hope to do much better this year. Perhaps not a new year’s resolution… let’s call it a new year’s goal.
</em></p>

<ol id="footnotes">
	<li id="fn1">Hello <strong>M</strong>assive <strong>V</strong>iew <strong>C</strong>ontrollers <a href="#ffn1">&#x21A9;&#xFE0E;</a></li>
	<li id="fn2">and choosing no architecture will only yield a very crappy architecture with attributes you definitely don’t want. <a href="#ffn2">&#x21A9;&#xFE0E;</a></li>
	<li id="fn3">Evolving architectures is often “simply” a matter of refactors… but that’s really quite simplistic. Doing this properly can be very difficult without creating regressions.  <a href="#ffn3">&#x21A9;&#xFE0E;</a></li>
	<li id="fn4">Nothing life threatening. It was a corrective surgery done to my foot, something I’ve needed to do for decades but finally got to a place where I couldn’t ignore it any longer. But anyone who’s has foot surgery will tell you it ain’t no joke. Recovery takes a lot of drugs and a lot of time. <a href="#ffn4">&#x21A9;&#xFE0E;</a></li>
</ol>
