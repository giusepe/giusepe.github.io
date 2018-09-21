---
layout: post
title: Moving your blog from Ghost to Github and Jekyll
date: '2016-01-06 04:49:04'
---

The simplest thing to get started is to fork this blog, "startup" branch, and use it as a template

``` bash

    git clone https://github.com/juanfranblanco/juanfranblanco.github.io.git -b startup --single-branch

```

### Github setup

* Delete the .git folder to detach from the startup repo. 
* Create a new repository for the website, mine is an user/organisation website so it follows the naming format **USERNAME.github.io**. More info on [Github Pages](https://pages.github.com/)
* Clone your new repository 

``` bash

    git clone https://github.com/username/username.github.io

```

* Copy the "startup" files to your new repo.

### Ghost backup and transfer

* On Ghost admin interface, go to Labs and export your blog settings and data. 
* Install and run ruby gem **jekyll_ghost_importer** to generate your posts and drafts from your ghost backup. More info on the [jekyll ghost importer github page](https://github.com/eloyesp/jekyll_ghost_importer)

``` bash
    gem install jekyll_ghost_importer
    jekyll_ghost_importer GhostBackup.json

```

* Copy the content folder from your ghost site, to the root of your new repository. This contains all your images.

### Web site settings

* Edit _config.yml

``` yaml

    # Site settings
    title: Your blog title
    description: Your blog description
    baseurl: "" # the subpath of your site, e.g. /blog/
    url: "http://www.myblog.com" # the base hostname & protocol for your site
    twitter_username: twitterusername
    github_username:  githubusername

    # Build settings
    markdown: kramdown

    permalink: /blog/:year/:month/:day/:title/

    google_analytics: UA-0000000-1

    disqus_shortname: disqusshortname

```

* Also _data\footer.yml

### Finally CNAME

* Last but not least modify the CNAME file to match your blog address, you will need to add CNAME records (or modify your existing one) to point to **username.github.io**
