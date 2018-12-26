---
layout: post
title:  "Run My Old Work In 2010"
date:   2018-12-27 01:21:30 +0800
categories: old
---

I usually organize my old work, here is one of my old work in 2010: some path finding algorithm.  

It was neither first time nor last time I dived into path finding related algorithm. At first, I used to think and implement on my own. Gradually, I found more and more related resources. 

I still have very profound impressions about the concepts such as map, array, traversal although I'm not using it recently.  

But the origin project was created in Adobe Flex, the origin swf file cannot run on most browsers today because Flash Player is hard to be seen. 

(The origin project is here: [Origin Repository](https://github.com/jchprj/flashtests/tree/master/chengdu/findpath))

So in order to run the swf, I have to change the Flex UI components into pure ActionScript UI components. Then use [Mozilla Shumway](http://mozilla.github.io/shumway/) JS player to play the swf. A showcase is embedded below.

(The newly modified project is here: [Newly Repository](https://github.com/jchprj/flashtests/tree/master/swf/findpath))

Fortunately, it's very easy to compile the new project. No need to install any Flash related IDE. Just VS code and a plugin [ActionScript & MXML in Visual Studio Code](https://as3mxml.com/). Followed the guide, I used Apache Flex SDK to compile the project and got swf file.

There is also new way to compile some Flex based AS project to JS release. Or just a way to create project using mxml and AS. Using [Apache Royale](https://royale.apache.org/). But it can not use pure Flash classes. This is another topic if I will use it. 


Below is the swf file run in Mozilla Shumway JS player. The performance is much weaker than Adobe Flash Player. But Mozilla Firefox is better than Google Chrome to run. Shumway is discontinued by Mozilla.

<iframe src="/old/shumway/iframe/viewer.html?swf=../../findpath/findpath.swf" width="800" height="650"></iframe>

Origin flash player version(need Adobe Flash Player installed on browser)
<iframe src="/old/findpath/pathtest.html" width="800" height="650"></iframe>

