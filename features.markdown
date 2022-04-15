---
layout: page
title: Features
permalink: /features
---

## [Progress Sync](#syncing)

I'm using platform native api's for progress syncing, so your progress will sync across iPad and iPhone if you're signed into the same Apple account on both devices. For those who care, I'm using Apple's CloudKit api in conjunction with CoreData.

I haven't settled on a syncing solution for Android yet. Maybe (probably) Firebase.

Series downloaded on one device will become available on other devices.

## [Customizable Reading Experience](#reading)

I've added as many options to control the reading experience as I could think of, allowing you to customize the text to fit your preferred style.

### Font & Text Size Selection

![Font Selection](/assets/images/ReadformiPhoneFontSelection.png)

Adjust text size from a tiny 10pt font to a massive 30pt font. Includes a preview so you can see exactly what you're text will look like.

Readform has a variety of fonts to read with:

- System Default (San Francisco for iOS, Roboto for Android)
- Arial
- Baskerville
- Courier
- Georgia
- Helvetica Neue
- Palatino
- Times New Roman
- Verdana

### Paragraph Styles

![Paragraph Selection](/assets/images/ReadformiPhoneParagraphSelection.png)

Not everyone like reading blocks of text, so Readform provides three paragraph styles to choose from:

- Block: cause some people like it
- Standard: Indented paragraphs with an extra line between each paragraph.
- Compact: Indented paragraphs with no spacing between each paragraph.

### Themes

Readform has a variety of color schemes for both light and dark styles. I'll probably add even more later.

![Themes](/assets/images/ReadformiPhoneThemeSelection.png)

### Orientation Lock

Lock your device in portrait or landscape. It's a small thing, but really handy when reading in bed.

### See only what you want

![Reading Option](/assets/images/ReadformiPhoneReadingOptions.png)

Tap on the center of the screen to show/hide the "Status bar" and, optionally, the Navigation bar. You can hide everything but the text itself, or show as much info as you want:

- Chapter Title (Yay! No more, "wait, what chapter is this?")
- Previous/Next Chapter buttons (these buttons always appear at the beginning/end of a chapter)
- Percent of Chapter Read
- Current Page of Total Page
- Turn on/off Lore Highlights

### Integrated Dictionary

![Dictionary Slice](/assets/images/ReadformDictSlice.png)

Tap and Hold on a word to see it's definition using your device's Dictionary.

## [Links](#links)

![Links](/assets/images/ReadformiPhoneLinks.png)

I want to make it as easy as possible to support your favorite authors. So I provide easy to access links that allow you to visit:

- The Provider's Website
- The Home Page of the Series
- The Home Page of the Current Chapter
- The Author's Page
- Any "Support" options for the author: Patreon, Paypal, etc.

## [Series Search](#search)

![Provider Search](/assets/images/ReadformProviderSearchSlice.png)

I integrate with Providers to provide the best possible search experience I can. I've full search along with as many of the "lists" I could find (Popular, Trending, Rated, etc).

Tapping on a Series inside will bring up a full card Preview of, including description and metadata tags (ex: Rating, Views, etc). In case that's not enough, a quick tap will open the full result in the web browser.

## [Searchable Table of Contents](#toc)

![Table of Contents](/assets/images/ReadformTOCSlice.png)

For each series, I download and parse the Table of Contents for the series and make it easily accessible.

Also includes Cover Art and access to all the Lore/Glossary, if the series has one.

## [Author Notes](#authornotes)

Many authors provide notes before and/or after a chapter. Whenever possible, Readform will parse those notes into a separate section and provide easy access to them from anywhere within the chapter. Simply tap on the notes button for that chapter.

## [Per Chapter and Full Series Download](#download)

There's nothing so annoying as finding yourself without internet and unable to view the next chapter of your favorite series. So Readform allows you to download chapters in advance, and even the entire series so you can read on without waiting for the plane to land first!

## [Integrated Lore system](#lore)

![Highlight Lore](/assets/images/ReadformProviderLoreHighlight.png)

_This only works for Series that have some kind of parse-able Lore, Glossary, Cast List, etc._

Ever come across a character that feels familiar, but you just can't remember who the hell they are? You might be tempted to go searching through the Glossary, or even Google search, but if you're like me, you probably just shrug, go meh, and keep reading, hoping you'll figure it out eventually.

Well, no longer. Readform integrates a Series' Lore into the Reading Experience, highlighting names, places, words or whatever is in the glossary. Tap an hold and see all the details without leaving the page you're reading.

![Lore Display](/assets/images/ReadformProviderLoreSlice.png)

## [Background Refresh](#background)

Readform will periodically check for new content in the background. If new chapters are found, it will display a Notification letting you know.
