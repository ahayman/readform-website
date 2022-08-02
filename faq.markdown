---
layout: page
title: Frequently Asked Questions
permalink: /faq
---

### How does Readform make money?

It doesn't, not really. I do provide a way to pay me via In-App Purchases, but that's just a tip jar. There are no subscriptions and the app itself is free and I intend to keep it that way.

If you're concerned I'll loose interest: Eventually, I'll start release my own web series and Readform will be a critical part of that.

### How do I add my Web Series to Readform

The quickest and easiest way is to simply post your series on one of the [Providers](/providers) that Readform already supports.

If you want to add your series directly to Readform, you can [contact me](mailto: aaron@flexile.co). Before you do, please keep in mind:

- It can take 1-3 days worth of work to add a series.
- I am happily married, a father of three, I work a full time job, and I'm writing my own web novel. Take a moment and let that sink in.
- I will always prioritize adding a Provider over any individual Web Series, for obvious reasons.
- If your series will work just as well on an already-supported [Provider](/providers), then I'll likely point to one of those [Providers](/providers).
- I will prioritize Web Series that have a parsable Lore/Glossary, especially if it's extensive and it's clear it facilitates the story. Since no Provider has a dedicated section/page for a Glossary (as far as I can tell, at least), this is something unique to Readform series.
- I may stop accepting new series if I get too backlogged.  I have no idea how popular Readform will be (if at all), but I am only one person. 

### How does Readform affect a site's Analytics?

For each Web Series added, Readform will periodically check the "table of contents", wherever that may be. It's possible this will affect analytics, depending on how they're setup, but somewhat unlikely. Direct request like Readform don't report itself as a browser, which most analytics require to count as a "hit". Also, most analytics rely on Javascript injection to do their thing, and Readform runs absolutely no Javascript.

### How does Readform affect a site's Load

As mentioned above, Readform periodically checks the "table of contents" page for new content. However, it does respect eTags (304 codes). So long as your server supports them (most do), this means instead of downloading the page, Readform will simply use cached data instead.

Also, Readform doesn't download all the other ~sh**~ stuff that browsers do. Most websites have a ton of extra stuff that's downloaded (analytics, scripts, async-loaded data, etc). Readform only downloads the page itself. This makes it much lighter on a website than a normal "user visit" would.

### How do I delete a Series

Tap and hold on the series on the Home Screen. This will bring up a preview card. The option to delete will be in there.

Once you delete a series off your device, it will show up in the "On Other Devices" section. This is data shared across all your devices, which allows you to easily download a series on multiple devices without having to search for them. To delete the series from this section, follow the same steps: Tap & Hold. Then delete from the Preview card.

Note: Deleting a Series from "On Other Devices" will **not** delete the series from other devices that have already downloaded it.

### How do I download all Chapters in a Series?

There's two ways to do this:

- On the "Home" screen, long-press on the Series. A Preview card will show up. In that screen there should be an option to download all chapters.
- Open the Series, then open the Table of Contents. Near the top right will be a download button (next to "Contents"). This will download any remaining chapters. You can also download individual chapters from from this screen.

### Embedded Tables are messed up

Yeah, they probably are. This is mostly a problem in the litRPG genre, where stat pages are a thing. The data is there, but it's not formatted into a table. I've not figured out how to do that (yet?). 

The problem is that I'm converting each chapter into text that can be broken up into pages, and that process seems to strip tables of their "tableness". For now, if you want to see the tables in all their original glory, I recommend opening the Chapter on the web (go to Chapter settings to and tap "View Chapter). This will open up the chapter in the web browser.

### Some Images don't show up

This seems to be a problem with some content management systems (ex: WordPress) that try to asynchronously load images after the fact using javascript. In some cases, I'm able to extract the proper URL information from the javascript. In other cases, I cannot.  

### Spoilers doesn't work

Some providers use javascript to hide sections under a "Spoiler" button. Readform doesn't support this and, depending on how it's setup, Readform will end up either showing the "spoiler text" or simply "Spoiler" in it's place.

There's not much I can do about that. I do-not/cannot/will-not add Javascript processing to Readform. Not only would it bog things down considerably, but it would break the pagination thing that makes Readform... well, Readform.

### My data isn't syncing

First, data will not sync across iOS and Android (whenever that's done.)

On iOS, I'm using Cloudkit (Apple's framework) for syncing data, and it requires that you are logged in to an Apple account on your device.  So:

- Check to make sure both devices are logged into the same Apple account.
- Be patient. Data can sometimes take a little bit to sync.  This is controlled by iOS/Apple, not me.
- Try restarting the app. That usually triggers the app to sync itself.