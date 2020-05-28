---
title: Better F1 Help for Visual Studio
author: James
type: post
date: 2009-02-02T01:29:35+00:00
url: /?p=50
categories:
  - Software Development
tags:
  - IDE
  - Visual Studio

---
The F1 Help key can be frustrating because it is very slow to invoke on Visual Studio. I figured out a way to make it faster by having it bring up a browser rather than the MS help. This technique will also work with the currently highlighted word.

Use the following steps to make this happen:

  1. Use the Tools > External Tools menu.
  2. Add a new tool as shown in the top image below.
  3. Use the following text for the Arguments: http://www.google.com/search?q=site:microsoft.com%20$(CurText)
  4. Use the Tools > Options menu for and select Environment > Keyboard.
  5. Type in &#8220;External&#8221; in &#8220;Show Commands Containing&#8221;
  6. Select &#8220;Tools.ExternalCommandN&#8221; (Where N is its order in the External Tools list).
  7. Fill out the rest of it as shown in the bottom image below. If the F1 command is already taken by the existing Help then disable it in the Help.F1Help command.

{{< figure src="/wp/wp-content/uploads/2009/02/external_tools.png" alt="" >}}

{{< figure src="/wp/wp-content/uploads/2009/02/keymapping.png" alt="" >}}

Now your help should come up very fast.