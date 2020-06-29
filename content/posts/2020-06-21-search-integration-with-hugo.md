---
title: "Search Integration with Hugo Static Site"
date: 2020-06-20T15:29:00-05:00
type: post
author: James
categories:
  - Blogging
tags:
  - blogging
  - Hugo
  - Wordpress
  - lunr.js
  - lunr
comments: true
---

While [migrating from a Wordpress site to a static site in Hugo]({{< relref "2020-06-07-migrating-blog-from-wordpress-to-hugo.md" >}}), I needed to implement some kind of search to replace the search available in the dynamic Wordpress site. There were several articles written about how to do this using the  [lunr.js](https://lunrjs.com/) search framework but I couldn't get some of them to work very well so decided to write this post in case anyone else is having similar trouble.

The main idea is to generate a searchable index based on the current state of the site and then that index is used by lunr.js at runtime in the browser to execute the search.

#### Generating the Search Index

This step is performed after generating the site in Hugo. A script file is run from Node and the console output is redirected to a **search-index.json** file in the static folder via this command:

{{< highlight "text" >}}
  node ./build-lunrjs-index.js > static/gen/search-index.json
{{< / highlight >}}

The script file [build-lunrjs-index.js](https://github.com/turnkey-commerce/culbertsonexchange-blog/blob/master/build-lunrjs-index.js) has a dependency on [parser-front-matter](https://www.npmjs.com/package/parser-front-matter) in addition to lunr as spelled out in the [package.json](https://github.com/turnkey-commerce/culbertsonexchange-blog/blob/master/package.json) file.

The generation has to be run each time content is changed or added so that the search-index.json can be uploaded for use in the search.

#### Implementing the Search Form

This part was the more confusing aspect as I had issues getting the search form to work and even where to put the search form. I finally figured out the search form could be put in a directory called **layouts/_default** and the content and processing goes into the [search.html file](https://github.com/turnkey-commerce/culbertsonexchange-blog/blob/master/layouts/_default/search.html) in that directory. The layout concept allows it to be independent of the themes while integrating well with the content.

Then a search content page was added as [index.md](https://raw.githubusercontent.com/turnkey-commerce/culbertsonexchange-blog/master/content/search/index.md) in a directory called **content/search**:

{{< highlight "text" >}}
  ---
  layout: search
  title: Search
  permalink: /search/
  noToc: true
  menu: main
  ---
{{< / highlight >}}

The **layout: search** line ties it to the search.html layout and the **menu: main** line ensures that there will be a search reference in the main menu.

To improve the UI of the search form some tweaking was done to the [site.css](https://github.com/turnkey-commerce/culbertsonexchange-blog/blob/master/static/css/site.css) in addition to tweaking of the [search.html](https://github.com/turnkey-commerce/culbertsonexchange-blog/blob/master/layouts/_default/search.html) to produce a more pleasing result:

{{< figure src="/images/search.png" alt="Search UI" title="Search UI">}}

The complete site can be found at [this repository](https://github.com/turnkey-commerce/culbertsonexchange-blog).



