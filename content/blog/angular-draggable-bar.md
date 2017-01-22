+++
date = "2014-06-24T09:53:50+08:00"
title = "Draggable bar directive in AngularJS"
categories = ["angularjs", "jquery"]
slug = ""
showpagemeta = true
tags = []
comments = true
draft = false
showcomments = true

+++

Over the past few days, I read a ton of AngularJS tutorials on directives. This topic is indeed hard to grok but I finally hacked together a draggable bar directive for the 10kft clone.

You can check it out [here](http://jsfiddle.net/uohzxela/9m3Tg/) on JsFiddle. Feel free to play around with its attributes.

I think AngularJS’s opinionated architecture and 2-way data binding is awesome for CRUD apps. But when it comes to extensive manipulation of DOM, JQuery is still king.

This is not to say that AngularJS is totally useless when it comes to creating interactive widgets; it is useful in an architectural way.

Without AngularJS, there will be no directives to shorten and reuse the code.

Notice that I only used two terse lines to invoke the draggable bars:

```html
<body>
    <div draggable percent="19%" height="30px" width="100%" bar-color="LightCoral"></div>
    </br>
    <div draggable percent="40%" height="60px" width="60%" bar-color="PaleTurquoise"></div>
</body>
```

Whereas if you go the non-Angular route, you have to duplicate the messy code in the directive template twice:

```html
<div id ='container' ng-style="{'width':width, 'border':'3px solid black', 'overflow':'hidden'}">
    <div id='sidebar' ng-style="{'width':percent,'height':height, 'background-color':barColor, 'overflow-y':'hidden'}">
          <span class='position' ng-style="{'overflow':'hidden'}"> </span>
            <div class='dragbar' ng-style="{'width':'3px','height':'100%', 'background-color':'black', 'float':'right', 'cursor':'col-resize'}"></div>
    </div>
</div>
```

In short, AngularJS directives provide a way to package code into reusable HTMl widgets, but you still have to use JQuery if your DOM manipulation is extensive.