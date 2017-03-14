---
layout:     post
title:      DSP2017 project - first week
date:       2017-03-14 21:15:00
summary:    Atom is cool kids text editor now. And I don't really like it. 
categories: i3wm-config-generator 
tags:       dsp2017
---

## Hello there, fellow reader. 

I just ended the first week of work on my DSP2017 program. 

Sadly I didn't accomplish much this week. I only created a basic directory structure 
for my project. Program architecture will be centered around functionalities and 
each directory will contain View, Model, Msg and Update for certain functionality
or data type. It's the best choice since it will allow me to easily add and modify 
functionalities without all that hassle that separating by role creates.

I also set up webpack to compile elm, sass and made the core for first functionality 
from my road map. Good news is I solved most of the problems I predicted. All whats
left is to create and hook up correct actions and style program up. Here's my plan 
for next week:

0. Create basic readme with links and summary
1. Fixing creating new tile container in correct tile container
2. Add tiles with pure black background 
3. Add static top bar
4. Make it look like i3wm

## Biggest issues I faced 

### Recursive model 

![Recursion](https://i.imgur.com/dAlqqKl.png){: .img--center }

A recursive model itself is a problem when you operate on type aliases in Elm since 
type alias is just shortcuted for something else. In my case it's unit (like object 
in JS) - and you can't endlessly write nested objects. 
And simple endless recursion breaks it.

#### Doesn't work

```
type alias Model =
    { layout : Layout
    , tiles : List Model
    }
```

#### Works perfectly

```
type alias Model =
    { layout : Layout
    , tiles : ChildTiles
    }

type ChildTiles
    = ChildTiles (List Model)
```

### Working on recursive model created by type

Creating nested list like this created need for custom functions to work on that
list. I could not simply add items to it with simple operations. I needed helper
functions that would "extract" list I working with and append element passed as 
another argument. It was tricky to wrap my head around the idea that passing type 
as an argument can allow you to "extract" thing it contains. Example below. 

```
type ChildTiles
    = ChildTiles (List Model)

appendTile : ChildTiles -> Model -> ChildTiles
appendTile (ChildTiles list) element =
    ChildTiles (element :: list)
```

Simply trying to do `childTilesList :: list` causes type mismatch. 
