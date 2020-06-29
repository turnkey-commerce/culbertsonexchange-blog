---
title: "Migrating a Blog From Wordpress to Hugo"
date: 2020-06-07T21:10:48-05:00
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

I recently decided to migrate this blog from Wordpress to a site using the [Hugo static site generator](https://gohugo.io/). The main goal was to simplify the hosting and learn more about the technology of static site generation. I chose Hugo mainly because it is fast and simple to use and install.

### Process

The process to migrate the blog was somewhat involved because I wanted to make some improvements to the layout and hosting in addition to getting it into a new host. I also wanted to make it compatible with the old permalinks which turned out to be a bit of a challenge.

The main steps were:

#### 1. Export from Wordpress to Hugo markdown files

For this I installed a Wordpress plugin called [hugo-to-wordpress-exporter](https://github.com/SchumacherFM/wordpress-to-hugo-exporter). This creates a zip file and there's a [good guide here](https://ma.ttias.be/step-by-step-guide-migrating-wordpress-to-hugo/) on how to use it. Overall it did a good job on exporting the content, but I did miss the fact in the README that you can configure it to also export the comments.  I wish I had done that as I had to add them back manually. As expected there was also some tweaking that had to be done to the content, especially for link/image references and code snippet highlighting.

#### 2. Choosing a Theme

I found a theme I like by browsing the ones on the Hugo site and came to appreciate the [Mainroad](https://github.com/Vimux/Mainroad/) theme. The layout is nice and simple and works great with mobile.

#### 3. Allow Redirects From Old Wordpress Permalinks

This is necessary so that search engine and other blog's links will still work and was a challenging aspect because the old wordpress permalinks were of this form:
{{< highlight "text" >}}
  http://www.culbertsonexchange.com/wp/?p=120
{{< / highlight >}}

So the natural inclination is to put the same permalink directly in the Front Matter at the top of the markdown file, e.g.

{{< highlight "text" >}}
  aliases:
    - /wp/?p=120
{{< / highlight >}}

However that didn't work as there were problems with the parsing of the "?" character when Hugo creates the alias as they have to get written as folders to contain the redirect HTML file. A workaround way to do that is to create a convention like the following:

{{< highlight "text" >}}
  aliases:
    - /permalink/p120
{{< / highlight >}}

This can then be handled by an HTML rewrite in the web server to change it to the old format when redirecting. For example, with the Caddy webserver a configuration such as this can be used to make the rewrite to the Hugo alias file that does the final redirect:

{{< highlight "text" >}}
  rewrite {
      if {path} has wp
      to /permalink/p{?p}
  }
{{< / highlight >}}

Basically it says if the incoming path has wp as part of the path then rewrite it to direct to the alias permalink with the value of the p? query parameter.

One other related detail involved moving any of the image files and old posts off the old "wp" paths so they don't get caught in the rewrite rule.

#### 4. Enabling Search

This aspect was more challenging than the permalinks as there is no automated way to do that with static file generators. I ended up using an approach where an index is prebuilt into a JSON datafile and then that index is searched using client-side Javascript. For this I used the [lunr.js](https://lunrjs.com/) framework. However, many of the examples I found didn't work well and [wrote another post giving more details]({{< relref "2020-06-21-search-integration-with-hugo.md" >}}) on how I got this to work.

#### 5. Hosting

Using the [Caddy webserver](https://caddyserver.com/) makes it easy to host on any affordable VPS with the added bonus of automatically installing [LetsEncrypt](https://letsencrypt.org/) certificates for HTTPS. It can't get much easier than that and removes any excuses for not having secure HTTPS connections.
