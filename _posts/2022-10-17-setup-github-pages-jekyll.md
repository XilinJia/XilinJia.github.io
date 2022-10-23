---
layout: post
title:  "Setting up Github Pages with Jekyll and Minima"
date:   2022-10-17 19:36:30 +0100
tags:   Jekyll Minima Github
---

So creating a Github Pages site turns out quite straightforward, though for a newbie, some descriptions may sound daunting.  [This official documentation](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll) on using Jekyll to create the site helps one to get a quick upstart.

Then test the site locally with [these steps](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll).

Choosing a theme to go with Jekyll largely determines how site files are structured and how or whether further features can be enabled.  There are many Jekyll themes to choose from, free or paid, open-source or proprietary.  I found the default theme: Minima quite satisfying.  The current released version and the version accepted for Github Pages is 2.5.1, however, the Minima default page is about a development version.  Version 2.5.1 is [here](https://github.com/jekyll/minima/tree/2.5-stable).

Format of each page is governed by a specific html file under _layouts.  Note, Upon site creation, some of the files under _includes and _layouts are not present.  The default ones from the theme are used.  When I wanted to customize certain files, I simply copied them from the original source of the theme and edit it.

[This](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/adding-content-to-your-github-pages-site-using-jekyll) describes how to add new posts and new pages.  List of posts appears on the home page.  If you add a new page, a pointer to the page will appear on the right-hand side of the header area.  Creating a new page is simply creating a new .md file in the root directory of the source.  The title defined in the file will appear as pointer and title of the new page.

So any new .md file under the root directory and below will result in a new page.  This is often time undesirable.  To remedy this, add something like "included: true" in the front matter (header area) of the file that you want to appear as a page, then, add filtering statement in header.html file under _includes directory (check my [source](https://github.com/XilinJia/XilinJia.github.io/tree/main/_includes) for example).

Adding in Google Analytics to the site gave me some hurdle, though it actually turned out very simple.  As Google has changed tracking id scheme, so original descriptions on [Minima page](https://github.com/jekyll/minima) is no longer applicable for the new "Measurement ID".  Someone has documented on a way to deal with it, [like here](https://cuda-chen.github.io/blogging/2022/04/30/switch-your-jekyll-blog-to-google-analytics-4-simplified.html).  But with that implemented for my site, I couldn't see Google Analytics detecting access to my site.  So I had to do quite some research.  It turned out I was accessing my site using the Brave Browser (my default browser) which disables tracking by default, and so Google Analytics showed no info on my access to the site.  When I switched to Chrome or Firefox, then the accesses are recorded.

So on my site, I have this post page, page "Digital" referencing to my github repositories, page "Nomad" listing places I've traveled to, and page "About" a brief description about myself.

## Cheers!  Santé!  Prost!