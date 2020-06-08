---
title: "Migrating Blog From Wordpress to Hugo"
date: 2020-06-07T21:10:48-05:00
type: post
author: James
draft: true
categories:
  - Blogging
tags:
  - blogging
  - Hugo
  - Wordpress
---

I recently decided to migrate the blog from Wordpress to a site using the Hugo static site generator. The main goal was to simplify the hosting and learn more about the technology of static site generation. I chose Hugo mainly because it is fast and simple to use and install.

### Process

The process to migrate the blog was somewhat involved because I wanted to make some improvements to the layout and hosting in addition to getting it into the new system. I also wanted to make it compatible with the old permalinks which was a bit of a challenge.

The main steps were:

#### 1. Export from Wordpress to Hugo markdown files

For this I installed a Wordpress plugin called [hugo-to-wordpress-exporter](https://github.com/SchumacherFM/wordpress-to-hugo-exporter). This creates a zip file and there's a [good guide here](https://ma.ttias.be/step-by-step-guide-migrating-wordpress-to-hugo/) on how to use it. Overall it did a good job on exporting the content, but I did miss the fact in the README that you can configure it to also export the comments.  I wish I had done that as I had to add them back manually. As expected there was also some tweaking that had to be done to the content, especially for link/image references and code snippet highlighting.

#### 2. Choose a Theme

I found a theme I like by browsing the ones on the Hugo site and came to like the [Mainroad](https://github.com/Vimux/Mainroad/) theme.

#### 3. Redirect to Old Wordpress Permalinks

This was a challenging aspect as the old wordpress permalinks were of this form:
{{< highlight "Html" >}}
  http://www.culbertsonexchange.com/wp/?p=120
{{< / highlight >}}

So the natural inclination is to put the same permalink directly in the Front Matter at the top of the markdown file, e.g.

{{< highlight "text" >}}
  aliases:
    - /wp/?p=120
{{< / highlight >}}

However that didn't work as there were problems with the parsing of the "?" character when creating the alias as they have to get written as folders to contain the redirect HTML file. A workaround way to do that is to create a convention like the following:

{{< highlight "text" >}}
  aliases:
    - /permalink/p120
{{< / highlight >}}

This can then be handled by an HTML rewrite in the web server. For example, with the Caddy webserver a configuration such as this can be used to make the rewrite to the Hugo alias file that does the final redirect:

{{< highlight "text" >}}
  rewrite {
      if {path} has wp
      to /permalink/p{?p}
  }
{{< / highlight >}}




