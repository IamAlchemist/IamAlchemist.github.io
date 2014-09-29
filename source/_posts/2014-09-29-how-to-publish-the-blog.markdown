---
layout: post
title: "How to publish the blog"
date: 2014-09-29 21:37:35 +0800
comments: true
categories: 
---

    rake new_post["title"]
    rake generate
    rake watch
    rake preview
    rake deploy

    git add .
    git commit -m "new blog"
    git push origin source

    cd _deploy
    git push -u gitcafe master:gitcafe-pages
