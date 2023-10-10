---
title: "Status Updater App"
date: 2023-10-08T09:00:00-05:00
type: post
author: James
categories:
  - Software Development
tags:
  - App
  - Limited Vision
  - Wails
  - Go
comments: true
---

My father-in-law has very limited vision due to
[Macular Degeneration](https://www.hopkinsmedicine.org/health/conditions-and-diseases/agerelated-macular-degeneration-amd/). As
the disease progressed he became unable to use email to contact us from
his assisted living facility. We needed to
provide a way for him to contact us to let us know he is OK in the morning
and afternoon or to call him if needed. For serious issues he has a button
on his wrist to alert the local staff.

## App Design

To help him to be able to stay in contact with status updates and call requests
I designed an application that could run on Windows and provide two very big
buttons that he would be able to see. When he would click on a button it would
send an email to us with the contents depending on which button he clicked.

The initial design looked like this:

![Initial Design](/images/status-updater-design.png)

When testing that design with my father-in-law, he was able to see the
button with the yellow background but unable to see the button with the
green background. So one design change was to make both of the buttons
with a yellow background to provide the most contrast.

The final design looks like this:

![Final Design](/images/status-updater-screen1.png)

And when the "I'm OK" button is clicked it becomes temporarily disabled
to prevent sending multiple emails, and once the email has been successfully
sent it will show a status message:

![After Successful Update](/images/status-updater-screen2.png)

The status message is shown with large text in order to be readable to the user.

## Technology Used for the App

The app was built with the [Wails](https://wails.io/) framework. It allows the
building of cross-platform UI applications using native UI elements for the
front-end and the use of [Go language](https://go.dev/) in the backend. It is
similar in concept to the [Electron](https://www.electronjs.org/) framework, but provides interoperability with native Go in the backend rather
than [Node.js](https://nodejs.org/en).

Wails was useful for this use case in that it allows for a familiar environment
for the front end using Typescript and CSS to build the UI. For developers
that are familiar with front-end Javascript and HTML frameworks it makes it
easy to iterate on the size and color of the buttons using CSS. The interoperability with GO and its built-in libraries allowed for easy implementation of the SMTP protocol for sending the messages when the button
is clicked.

Additionally Wails is multi-platform with current support for Windows, Linux,
and MacOS. In my use-case it was built as a Windows application since my
father-in-law has a Windows machine. In the future Wails will also support
Android and iOS which would allow for a tablet version of the app. A phone
app would also be possible although the buttons may not be big
enough for such issues with sight.

## Code Repository

The code for this app is available on Github at
[status-updater](https://github.com/turnkey-commerce/status-updater).

The installable executable for Windows can be found at
[status-updater Releases](https://github.com/turnkey-commerce/status-updater/releases).



























