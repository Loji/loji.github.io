---
layout:     post
title:      DSP2017 project - second, third week
date:       2017-03-26 14:20:00 +0000
summary:    A little late for a party with this update post. Luckily I have progres.
categories: i3wm-config-generator-in-elm 
tags:       dsp2017
---

## Hello there, fellow reader. 

My last weeks very pretty rough and hence no major update was published since now. 
Luckily I have done much more than I planned last time. Here's the list of changes 
and something more about more interesting, specific problems below that. 

0. Creating tiles now works properly, as planned
1. Everything looks and behaves like i3 in most parts 
2. Tiles now have one of 10 placeholder background images
3. There is settings bar that will have all customization and config generation 
options in the future
4. Code is cleaner 
5. There's an npm task that automatically publishes built project to GitHub pages

Project is available for live preview of latest functionalities on GitHub pages 
here: [https://loji.github.io/i3wm-config-generator/](https://loji.github.io/i3wm-config-generator/)

Plans for next cycle is: 

0. Add basic color selections
1. Make color selection change color on page
2. Improve adding/removing tiles by imposing restrictions to adding/removing 

### Creating tile grid with flexbox

I've created all of my grid layouts using flexboxes since they are amazing and require
no additional work other than describing how grid behaves. Each layer of tiles has
`display: flex` property and all its children have `flex-grow: 1` so no matter how
many are inside they all have same width/height. As long as their content is basically 
same - `flex-grow` causes children to occupy remaining space with provided ratio.

Another part of this tile grid is that every tile that has children added changes
its display to tile container that behaves differently if child tiles are displayed
directly. It causes a parent to occupy more space in its row than in should. Workaround 
around this was simple. I created a wrapper for children that position itself 
absolutely in parent container - this prevents it from changing computed parent 
width/height. 

All can be done with this:

``` 
.tile {
    position: relative;
    flex-grow: 1
}

.tile-container {
    position: absolute;
    display: flex; 
    left: -1px;
    right: -1px;
    top: -1px; 
    bottom: -1px;
}
```

### Manageable, modular Elm architecture

I'm moving more towards fully modular Elm architecture that would allow me, in future,
write in it quality, testable code. I'll briefly cover idea without getting into 
details for now, this section is more of a brain dump and something that will hopefully
develop into a full, meaningful blogpost. 

Good Elm code can do all operations for certain module in its model exposing (exporting)
only functions that are end-points, like API. This would result in a much cleaner
code that uses all update functions to just return data transformed by just one 
function that is defined alongside model. At the end it would mean that any module 
that uses any external resource can be easily replaced or mocked for some test that
would test business logic (you don't really need to test anything else in Elm). 
