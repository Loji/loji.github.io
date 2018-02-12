---
layout:     post
title:      How I solved big data problem with no webworkers
date:       2018-02-12 19:10:00 +0000
summary:    Event loop and recursive iteration over large set of data. How to handle maximum call stack size exceeded.
tags:       programming dsp dsp2017 javascript
---

## Hello there, fellow reader.

Long time no see. Well, I was very busy since my last post here. I created and killed few projects already. And also get a new job at Collibra. But this isn't a post about this... or is it?

At my new job, I was given a task to optimize complicated algorithm that traces connections between nodes in the very unusual way. To put it simply - it wasn't regular node traversing like in tree view. The logic made it more like pathfinding, with own logic and rules depending on sibling/parent/children nodes and their types. You click on one node and every node that can be connected to your node gets highlighted. 

![Path hightlight on node click]({{ "/images/diagram-click.png" | absolute_url }}){: .img--center }

So, you should have a basic idea what was the environment and probably you already know what was the problem here - scale. Creating diagrams with thousands of nodes and hundrends of complicated connections was not fast enough, we weren't happy with performance.

There were basically 3 solutions: rewrite it to webworkers, rewrite it to be a single loop (so the stack trace won't ever grow) or use async functions. I used async because it allowed me to implement it with little effort and avoid webworker overhead for smaller data packages.

### Let's get to the meat.

The algorithm behind the data is complex and proprietary, let's just assume it's on the beginning of `queryNode(from, to)` function. It's launched inside a loop over all nodes we want to connect.

```
for(let i = 0; i < nodes.length - i; i++) {
  for(let j = i; j < nodes.length; j++) {
    this.queryNode(nodes[i], nodes[j]);
  }
}
```

Looks like it will be a big loop, doesn't it? Now let's look into `queryNode` method.

```
function queryNode(source, dest, path = [], conditions = null) {
  // complex fancy logic here

  if (source.id === dest.id) {
    mergePathToTraced(path);
  } else {
    // checkConnections launches queryNode
    // whenever checkConnections logic applies.
    source.children.forEach(child => checkConnections(
      child,
      dest,
      [ ...path, child ],
      conditions,
    ));
  }
}

function checkConnections(source, dest, path, conditions) {
  const newConditions = ...;
  queryNode(source, dest, path, newConditions);
}
```

This method is very problematic since it either finishes or it launches itself with as many additional nodes it finds as current node children. To make it even worse the algorithm sometimes made pathfinder go back over already checked nodes that finished with a dead end with a new set of conditions. 

Sometimes diagram that looks very simple could be complicated and consist of many more checks than how many nodes it has.

So, what was my first step after optimizing it as much as possible? First I'd make sure to cache every check with same conditions so it would never launch the same check again. After I was sure time has come for some testing. All I got is my favorite `Maximum call stack size exceeded` error. No luck with any further simple optimization.

### How do I async?

In JavaScript, you can do anything asynchronously if you'd put it into `setTimeout` function. The JS engine takes the function and puts it on event loop query after 1ms (it's minimal time for `setTimeout`, even 0 as a second arg evaluates to at least 4ms. I'll tell you why later). `setTimeout` put's callback function into macrotask query. More on that later in the text.

So, we have the basics. To really get into async I changed how the application works. There is now an array containing references to functions that launch node check with proper arguments and infinite, "recursive" `setTimeout` loop that takes those functions and launches them in their own `setTimeout`.

```
const nodesQuery = [];

function launchCheck() {
  for(let i = 0; i < nodes.length - i; i++) {
    for(let j = i; j < nodes.length; j++) {
      this.queryNode(nodes[i], nodes[j]);
    }
  }

  queryNodes();
}

function queryNode(source, dest, path = [], conditions = null) {
  // complex fancy logic here

  if (source.id === dest.id) {
    mergePathToTraced(path);
  } else {
    // checkConnections launches queryNode
    // whenever checkConnections logic applies.
    source.children.forEach(child => {
      nodesQuery.push(checkConnections(
        child,
        dest,
        [ ...path, child ],
        conditions,
      ));
    });
  }
}

function checkConnections(source, dest, path, conditions) {
  const newConditions = ...;
  queryNode(source, dest, path, newConditions);
}

function queryNodes() {
  const MAX_QUERIED = 1200;
  const length = nodesQuery.length;
  const nodesToQuery = getNodes(MAX_QUERIED);
  if (nodesToQuery.length > 0) {
    nodesToQuery.forEach(queryCallback => setTimeout(queryCallback));
    setTimeout(queryNodes);
  } else {
    onFinish();
  }
}

function getNodes(amount) {
  return nodesQuery.splice(0, amount)
}
```

Last bit looks confusing, doesn't it? Let me explain how it works.

First, it takes 1200 elements or as many as possible mutating `nodesQuery` array. It later takes elements that were removed from the query and passes it to `setTimeout`. After this, it pushes itself to the event loop. All those functions will be executed synchronously one after another and in between each of those functions browser will be able to handle animation and user input. But we don't really need to animate frame or handle button click after each of those functions. Better performance would be much more useful, right?

And if `nodesQuery` was empty when we executed this function we know we're done.

### What is faster than setTimeout

As I said earlier `setTimeout(callback, timeout, ...additionalArgs)` that takes at least 1 argument has the second one with a time that guarantees minimum time after which callback gets executed. It evaluates to 4ms if no argument is provided or if it's anything less than 4ms. It's a part of HTML5 specification.

To get faster we'll have to use something that doesn't depend on the time at all. Something like `process.nextTick` from Node or `setImmediate` from Internet Explorer (Yes, IE has something that nobody else has! Too bad it's this way because this API has really shitty and confusing name so nobody wants to implement it...). Unluckily for us, we can't use any of those. There is one other way though.

###  Microtasks and macrotasks

A promise is asynchronous, isn't it? It is, in fact it uses microtasks. The main difference from macrotask is that event loop executes all of them at one sitting. If any microtask creates another microtask, it will get executed right away as well. Macrotasks, on the other hand, will get executed in way that in between browser will be able to render new frames and handle user input. Microtasks will be executed first, before any macrotask.

![How event loop handles events]({{ "/images/eventloop.png" | absolute_url }}){: .img--center }

It's a bit hack-ish but `Promise.resolve().then(callback)` does the job really well. It creates a new Promise that is resolved and registers our function as it's success callback to the microtasks query - this way it will be executed after all other microtask. Too many callbacks called like this would behave like hanged tab - no user event would pass and be handled.

The biggest advantage is that it won't cause a stack overflow.

```
function happyNoError() {
  Promise.resolve().then(happyNoError);
}

function sadStackOverflow() {
  sadStackOverflow();
}
```

Even if it's slower than simple recursion in some cases you might find it useful. You can *avoid maximum call stack size exceeded* with it.

So, after the changes, you'll get basically this.

```
const nodesQuery = [];

function launchCheck() {
  for(let i = 0; i < nodes.length - i; i++) {
    for(let j = i; j < nodes.length; j++) {
      this.queryNode(nodes[i], nodes[j]);
    }
  }

  queryNodes();
}

function queryNode(source, dest, path = [], conditions = null) {
  // complex fancy logic here

  if (source.id === dest.id) {
    mergePathToTraced(path);
  } else {
    // checkConnections launches queryNode
    // whenever checkConnections logic applies.
    source.children.forEach(child => {
      nodesQuery.push(checkConnections(
        child,
        dest,
        [ ...path, child ],
        conditions,
      ));
    });
  }
}

function checkConnections(source, dest, path, conditions) {
  const newConditions = ...;
  queryNode(source, dest, path, newConditions);
}

function queryNodes() {
  const MAX_QUERIED = 1200;
  const length = nodesQuery.length;
  const nodesToQuery = getNodes(MAX_QUERIED);
  if (nodesToQuery.length > 0) {
    nodesToQuery.forEach(queryCallback => callAsync(queryCallback));
    setTimeout(queryNodes);
  } else {
    onFinish();
  }
}

function getNodes(amount) {
  return nodesQuery.splice(0, amount)
}

function callAsync(callback) {
  return Promise.resolve(null).then(callback);
}
```

`queryNodes` is still called by `setTimeout` because we don't want to entirely block the browser. We want it to display nice loading overlay and animate it. It will happen in spare time from `setTimeout(queryNodes)` 4ms default wait time, after 1200 `queryNode` executes.

### Summary

I found that for this case calling inner function with microtask query meant better memory management. My case had very complicated logic and used a lot of memory, doing it this way allowed me to use less memory than without it. Benchmarked, tested and polished as much as you can in few days.