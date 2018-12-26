---
layout: post
title:  "Keyboard Navigation In PixiJS"
date:   2018-12-23 03:15:30 +0800
categories: js
---

I found an intersting feature request for PixiJS here:   
[How to do mouse and keyboard navigation?](https://github.com/pixijs/pixi.js/issues/4519)

Here is PixiJS: [https://github.com/pixijs/pixi.js](https://github.com/pixijs/pixi.js)  
A fast lightweight 2D library

From the description of the feature request, PixiJS has an AccessibilityManager, here is the source code of the AccessibilityManager: [https://github.com/pixijs/pixi.js/blob/dev/src/accessibility/AccessibilityManager.js](https://github.com/pixijs/pixi.js/blob/dev/src/accessibility/AccessibilityManager.js)

The AccessibilityManager implements focus switch by tab key in PixiJS displayobjects. After investigating the code, I understand the logic. The functionality is activated by pressing down tab key. Then update every frame. Duaring the updating, each displayobject which is focusable generate a html div element. The div elements are keep changing their size and position to match the displayobjects respectively. When press tab key, focus automatically switches among the html div elements.

This is a basic accessible functionality. The feature request introduced a more advanced approach which uses a Javascript library: [https://github.com/luke-chang/js-spatial-navigation](https://github.com/luke-chang/js-spatial-navigation)

This js-spatial-navigation implements focus navigation by arrow keys, 4 directions available. I also looked into the source implementation. When press an arrow key, the logic first group all matched objects into rects. The grouping algorithm called partition generate a relative position table:  
0 1 2  
3 4 5  
6 7 8  
The current focus target is in position 4. Every other object is placed to other number by calculating the distance of the current focus and each object. Only when the compared object intersects the current focus, they are on the same row or same column. Many objects maybe on same number position. A second partion is performed. This time, compare the center point of the current focus and all objects in position 4. This comparison generates another internal table with same algorithm.  
Next step is prioritize the targets when press arrow key. Each arrow key has different priority. For example, when press UP key, the priority array is: [internal group 0, 1, 2], [group 1], [group 0, 2]. First priority is always internal group, that means all objects in group 4. Then find object from group 1, if not found, find from group 0 or 2. The first group that has elements in the priority is selected as target. Last step, the algorithm to find the most proper object is a sorting all objects by several distance calculation functions. The distance calculation functions are also direction related.

I thought about implement the arrow key in Pixi AccessibilityManager. But after investigating both source, I tried a simple way, without modify any code of both source code, just use them. I created a jsfiddle here:
[http://jsfiddle.net/jchprj/epqwun84/](http://jsfiddle.net/jchprj/epqwun84/)

I also reference the fiddle test on the feature request page for further discussion.

Below is an embeded test page on my Github Pages.



<iframe name="nav" src="/test/nav/index.html" width="600" height="450" frameborder="0"></iframe>

Source code:

```
        var KEYMAPPING = {
            '37': 'left',
            '38': 'up',
            '39': 'right',
            '40': 'down'
        };
        window.addEventListener('load', function() {
            // Initialize
            SpatialNavigation.init();

            // Define navigable elements (anchors and elements with "focusable" class).
            SpatialNavigation.add({
                selector: 'button'
            });
        });
        window.addEventListener('keydown', function(e) {
            var direction = KEYMAPPING[e.keyCode];
            if (!direction) {
                return;
            }
            if (app.renderer.plugins.accessibility.isActive) {
                return;
            }
            app.renderer.plugins.accessibility.activate();
        });


        let type = "WebGL"
        if (!PIXI.utils.isWebGLSupported()) {
            type = "canvas"
        }

        PIXI.utils.sayHello(type)

        //Create a Pixi Application
        let app = new PIXI.Application({
            width: 500, // default: 800
            height: 360, // default: 600
            antialias: true, // default: false
            transparent: false, // default: false
            resolution: 1 // default: 1
        });
        app.renderer.backgroundColor = 0xAAAAAA;


        app.renderer.on('postrender', function() {
            SpatialNavigation.makeFocusable();
        }, this);

        //Add the canvas that Pixi automatically created for you to the HTML document
        document.body.appendChild(app.view);

        function setup() {
            for (var i = 0; i < 10; i++) {
                var xx = i % 6;
                var yy = Math.floor(i / 6);
                newRect(50 + xx * 60, 50 + yy * 60, 50, 50);
            }
            newRect(290, 110, 110, 50);
            newRect(50, 170, 50, 50);
            newRect(50, 230, 50, 50);
            newRect(110, 170, 110, 110);
            newRect(230, 170, 110, 50);
            newRect(230, 230, 50, 50);
            newRect(290, 230, 50, 50);
            newRect(350, 170, 50, 110);
        }

        function newRect(x, y, w, h) {
            var graphics = new PIXI.Graphics();
            graphics.position.set(x, y);
            graphics.beginFill(0xA0A0A0);
            graphics.lineStyle(5, 0xFFFFFF);
            graphics.drawRect(0, 0, w, h);
            // graphics.tabIndex = 2;
            graphics.accessible = true;
            graphics.interactive = true;
            app.stage.addChild(graphics);
        }

        setup();

        window.addEventListener('mousemove', function() {
            var x = window.event.clientX - 8;
            var y = window.event.clientY - 8;
            document.getElementById("info").innerHTML = x + ", " + y;
        });
```