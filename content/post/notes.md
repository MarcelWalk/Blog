+++
author = "Marcel Walk"
title = "Notes (Browser Extension)"
date = "2021-09-09"
description = "The lessons learned and journey developing it"
tags = [
    "browser",
    "extension",
    "javascript"
]
+++

Issues, lessons learned and my journey of developing a extension for chromium 
based browsers. And getting it approved by Google and published to the extension
store.
<!--more-->

# Intro
As a developer I do a lot of searching on the web to solve bugs and browse 
documentations, but its kinda annoying to have so many tabs open and finding the
correct one at a later point. So I figured it would be great if you were able to
just select text and save it for later and look at other solutions for the issue
you are having, and implement them later.

This is exactly what my new "Notes" extension is trying to solve, it enables you
to select text on a website and just right-click and "Save as note".
When you are done browsing you can visit the web-page integrated into the
extension to review the notes, you can even share them with others by clicking a
button and the extension will upload it to [paste.rs](https://paste.rs/).


# Development Process

### Extension Basics
So lets get to the good stuff, the development of the extension. The basics are
pretty straight forward, all a extension basically needs is a `worker.js` and
a `manifest.json`. You could name the `.js` file however you want but the 
`manifest.json` has to be called that way.

The `manifest.json` contains the most important informations about your 
extension such as:
- Name
- Description
- Version
- Name of the `js file`

For a basic extension it would look like this:
```json
{
  "name": "Notes",
  "description": "Save maked text easily as note for later use",
  "version": "0.4.1",
  "manifest_version": 3,
  "background": {
    "service_worker": "./worker.js",
    "type": "module"
  }
}
```

The `.js` file contains the logic for your extension, and will automatically be
launched when the extension is installed and will always run in the background.


### Development
I started out with a basic setup, just the `manifest.json` and my `worker.js`,
and tried to load that into chrome to check if the basics are working.

Then I added the basic functions for the context menu entry which are just those
lines:
```js
// Creating the menu entry
chrome.contextMenus.create({
    title: "Save as note",  // The text that is shown in the entry
    id: "SAVE_NOTE",        // The id for the entry, has to be unique
    contexts: ["selection"] // The context in which the entry will appear
});

// Adding save_note function to the clicked event
chrome.contextMenus.onClicked.addListener(save_note)
```

This will require us to add the `contextMenus` permission to the `manifest.json`.
So with this part in our `worker.js` a new entry is shown in the right-click
context menu when we select some text on a web site. If we wouldn't specify the
`context` as `selection` it would default to `page` so the entry would be shown
when whe right-click the page without selecting anything.[^1]

Additionally I added the `save_note` function which accepts the `OnClickData`
and `tabs.Tab` parameter[^2], and gets the selected text and the url from the
`OnClickData` and combines them in a `JSON object` which is then stored via the
`chrome.storage`[^3] API which has 2 variants `sync` and `local`.

The `sync` variant will sync the data to all chrome browsers the user is logged
in (given that the user has sync enabled), `local` will just store the data 
locally. I have decided to use the `sync` variant since it could be practical to
sync the data to other sessions, between the browser at home and at work for 
example.

Debugging up until here was pretty tiresome since I had to go to the chrome
extension tab and had to reload my extension every time I needed to test
something. Chrome itself gave me very limited feedback if there is something
severely broken in my `worker.js`, for example why it couldn't be loaded.

Less severe issues could be debugged easily though clicking on the service 
worker link below the extension id on the extensions page, which will open
a dev tool window which shows you the console output and other information you
are used to from the dev tools.

So with the basic functions out of the way I needed a way to show the notes,
and I saw that other extensions have a cool internal web page that can be opened
to edit settings for example. Sadly I couldn't find any information on 
StackOverflow or the internet in general, so I headed to the best place I could
think of, which was the Scrimba[^4] community discord[^5].

There I got some help from [Thacchi#6035](https://naseki.com/) and it turns out 
those web pages are just HTML files in the project structure, and can be easily
accessed by visiting:
- `chrome-extension://{extension_id}/{path_to_file}`

With that knowledge I was ready to do some work again, so I added the
`index.html` and the complementing `index.js` and used the 
`chrome.storage`[^3] API again to retrieve the previously stored notes data
and just displayed the text on the page. And was in front of issue again, 
I wanted it to look decently and since designing is not one of my best skills,
I decided to use a framework. 

So the question was, what frontend framework should I use? 
I looked at several, of course the more famous ones were Bootstrap, Vue and React.
The thing is those are pretty heavy and have a lot of features I wouldn't use.

I thought about it for a bit and came to the conclusion that I don't need one of
those heavy frameworks that come with JS code and everything, I needed something
simpler so I continued my search and decided I'll use Bulma[^6]. 
Since its a CSS only framework and has everything I need, so its just the right
tool for the job.

So I took the notification[^7] element since it closely resembled a sticky-note and
integrated it into my code so I added a function to generate the HTML for me and
add the necessary CSS classes and adds my data to it.

With my MVP done, I then gave it [Alexander Alemayhu](https://www.twitch.tv/alexanderalemayhu)
a streamer and youtuber for a initial review to collect some unbiased user
feedback. Since you can't trust your own opinions in terms of usability since
you wrote it, so things may seem obvious to you which for other people may be not.
One of the other regulars in the stream [Mads Bram Cordes](https://github.com/Mobilpadde),
reviewed it too and really liked it which gave me the motivation to continue to
work on it. Both gave me awesome feedback and ideas for additional features.
Mads later helped to resolve some bugs and issues I were facing with `async` and
`promises` and contributed to the project.

I then implemented a few of the feedback I got for example the sharing feature
and changing the previously used notification element to the card[^8] component
since it allows you to add buttons to the bottom of it which was a good feature
for the sharing.


### Release Process
I then decided thats a good point to start thinking about releasing it,
so I started searching for a way to release it to the Google chrome extension
market. After a bit of searching I found the page to register for a developer
account which is done via the [developer console](https://chrome.google.com/webstore/devconsole),
registering for a developer account costs 5$ which is very affordable compared
to other developer accounts.

The release process itself is pretty simple, compress all your extension files
into a zip archive hit "New item" in the developer console and upload it.
But thats just the start, you'll have to add some information:
- Support email address
- Title
- Summary
- Description
- Category
- Language
- Sore Icon
- Screenshots

You then have to describe the **single purpose** of your extension and have to
**justify the permissions** you added to the `manifest.json`.
Additionally you have to select what user data you want to collect (if any),
and confirm that you don't do any of the following:
- Sell or transfer user data to third parties, apart from the approved use cases
- Use or transfer user data for purposes that are unrelated to my item's single purpose
- Use or transfer user data to determine creditworthiness or for lending purposes

Thats pretty much it in terms of information to enter.
One of the reasons why my extension was rejected twice was that the 
`chrome.tabs.create` API call I used actually doesn't require the tabs permission.
> You can use most chrome.tabs methods and events without declaring any 
permissions in the extension's manifest file. However, if you require access to 
the url, pendingUrl, title, or favIconUrl properties of tabs.Tab, you must 
declare the "tabs" permission in the manifest[^9]

So **ALWAYS** check if the permissions you requests are really needed, it may
seem obvious to you that you need the permissions, like for me in this case.
But you may be wrong and your extension gets rejected!

[^1]: chrome.contextMenus.create [Documentation](https://developer.chrome.com/docs/extensions/reference/contextMenus/#method-create)
[^2]: chrome.contextMenus.onClick [Documentation](https://developer.chrome.com/docs/extensions/reference/contextMenus/#event-onClicked)
[^3]: chrome.storage [Documentation](https://developer.chrome.com/docs/extensions/reference/storage/)
[^4]: [Scrimba](https://scrimba.com/) is platform for learning web technologies,
mainly targeted to frontend developers and offers several free and paid online courses
[^5]: [Discord](https://discord.com/) A social network / messenger which is 
widely used by gamers and online communities to connect with each other
[^6]: [Bulma](https://bulma.io/) is a free, open source framework that 
provides ready-to-use frontend components that you can easily combine to build
responsive web interfaces.
[^7]: Bulma [notification element](https://bulma.io/documentation/elements/notification/)
[^8]: Bulma [card component](https://bulma.io/documentation/components/card/)
[^9]: [Section of the documentation](https://developer.chrome.com/docs/extensions/reference/tabs/#manifest)