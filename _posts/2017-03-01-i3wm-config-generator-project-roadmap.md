---
layout:     post
title:      i3wm config generator - project roadmap
date:       2017-03-01 19:15:00
summary:    Project summary for i3wm config generator - project for DSP2017 competition
categories: 
tags:       dsp2017
---

Hello there, fellow reader. 

I'm taking part in [DSP2017 (link in polish)](http://devstyle.pl/daj-sie-poznac/) 
competition and project I'm making for it is web configurator for i3wm written 
entirely in [elm language](http://elm-lang.org/). 

Why this project? It's simple. I'm fan of linux and customizability it gives to 
you. I can spend hours customizing and tweaking my environement. But changing 
config and doing reload is something I don't really like. It makes tweaking my 
WM clunky and slow. That's why I want to create something that will allow me to 
choose colors with live preview. I hope at least one person other than me will 
find this project useful, at least once.

Why elm? Initially I wanted to use React+Redux but I thought it would be too easy.
Elm seems to be really interesting language after few presentations that I attended
in Wroclaw recently. And it's first functional language I'll be using. It'll be fun. 

As for my project, here is roadmap I'll try to stick to: 

1. Panel for adding tiles next to each other
2. Color picker for borders, backgrounds and tile header text 
3. Saving colors as config 
4. Ability to split tile and make it possible to add more in that space
5. Ability to add image as tile content 
6. Changing "desktop" background
7. Importing colors from existing config values

Project source and preview is available on my [github](https://github.com/Loji/i3wm-config-generator).

If you don't know what i3wm is head to [i3wm website](https://i3wm.org/) and 
[/r/unixporn](https://www.reddit.com/r/unixporn/search?q=i3&sort=top&restrict_sr=on&t=all)