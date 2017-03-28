---
layout:     post
title:      GitHub pages npm autobuild and autopublish tutorial
date:       2017-03-26 14:20:00 +0000
summary:    Very short tutorial that will teach you how to autopublish your node project
categories: tutorial 
tags:       dsp2017 npm 
---

## Hello there, fellow reader. 

I'll cover setting your npm based project to automatically build and publish to
your GitHub pages on whatever branch you want. It's actually really easy since
(surprise surprise) there is a plugin for it.

First you need to download gh-pages

``` 
$ npm install --save-dev gh-pages
```

Now you will be able to publish to GitHub pages directly calling gh-pages. 

```
$ node ./node_modules/gh-pages/bin/gh-pages -t -d YOUR_BUILD_DIR -b YOUR_GH_PAGES_BRANCH
```

-t option includes all dotfiles, -d specifies a directory with a built project, -b 
specifies GitHub pages branch you want to push to. You can view a newest help by 
calling gh-pages with `--help` param.

When you're done you need to prepare your GitHub pages branch and push it. Start 
out with no files to commit. 

```
$ git checkout -b gh-pages
$ git push origin gh-pages
```

Next step is hooking up your full gh-pages exec to some npm command for easy access.
In your package.json add the script that would run what you need under whatever command
you like. 


```
  "scripts": {
    "build": "webpack",
    "publish": "npm run build && gh-pages -t -d dist -b gh-pages"
  }
```

The last step is configuring GitHub to use your newly created branch to serve your page
to the world. Go into your project settings and navigate to GitHub pages to enable
it and chose branch you created earlier. 

![GitHub pages settings](https://i.imgur.com/A0pcvQR.png){: .img--center }