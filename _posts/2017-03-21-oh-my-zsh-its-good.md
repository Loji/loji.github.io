---
layout:     post
title:      Oh my zsh! It's so good!
date:       2017-03-21 18:04:00
summary:    How your shell can be better 
tags:       dsp2017
---

## Hello there, fellow reader. 

When we talk about Linux we think about command line tools such as git, vim, grep,
rsync, less... and of course bash. Who wouldn't know bash? It's *THE* shell you
use in your terminal. Or is it? There are some cool alternatives you can use like
fish or zsh. 

Today I'll be talking about zsh (as you probably know after you've entered this 
blogpost) and some of its cool functionality that can make your life easier.

But first I'm going to explain what zsh is. Z shell is powerful shell meant to be
used interactively. It's also fully functional scripting language. Zsh is mostly
compatible with bash, so any eventual switch should be painless. 

### Cool features, what about them?

Right, I wanted to talk about cool things zsh can do for you. I'll start with my
favorite. 

#### Great autocomplete

Go where you want, press TAB twice and it displays everything that matches your 
prompt in form of nicely formated table with a clear distinction between directories
and files. And you can type TAB and arrows to move through the options!

```
$ vim /etc/nginx/
fastcgi.conf    koi-utf         nginx.conf      uwsgi_params  
fastcgi_params  koi-win         scgi_params     win-utf       
global/         mime.types      sites/                        
``` 

The other cool feature is another type of fancy autocomplete. It let you type just
a part of the path and it (in most cases) resolves to correct file after you TAB
it.  

You can type: 
```
$ vim /e/ng/n.co
```
And after tab it becomes:
```
$ vim /etc/nginx/nginx.conf 
```

#### Spelling correction

If you type fast you most likely make mistakes. It's very tiring to go over your
previous command to the beginning if you made some stupid mistake. In zsh you can
fix it instantaneously. 

```
$ vin /etc/ssh/ssh_config 
zsh: correct 'vin' to 'vim' [nyae]? 
```

#### Command history 

Zsh has some really cool command history. Not only it works in more than one terminal
instance simultaneously preventing you from losing parts of your work history like
in bash but also it allows to easily go back to command you're looking for. Did you
execute some sick grep search 100 commands ago? No worries, you don't have to go 
that back all on your own. You just type `gre` and go back to any previous commands
staring with "gre" with your arrow key. 

#### Oh my zsh!

And lastly I have to thank [oh my zsh](https://github.com/robbyrussell/oh-my-zsh)
for making switch not only painless and fun but also glorious. In the instant I 
have access to awesome git tools, autocomplete and branch info in my prompt but
also I have access to many, many themes that are easy to install and modify. 

I really recommend trying zsh out. You won't regret it. 
