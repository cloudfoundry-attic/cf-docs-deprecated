---
title: Node.js
description: Creating UI elements
tags:
    - nodejs
    - express
    - tutorial
---

## Introduction

This chapter is a continuation of [Chapter 1](/frameworks/nodejs/nodejs-tutorial/step01-developing-nodejs-app.html).

## Prerequisites


+  Should have completed [chapter 1](/frameworks/nodejs/nodejs-tutorial/step01-developing-nodejs-app.html), that explains how to create a http and sockjs server
+  Should know how to develop JQuery widgets
+  Should have a basic knowledge of [Fabric.js](http://fabricjs.com/)
  
This Whiteboard application layout is divided into 2 basic parts

+  Top bar - for app title and user name
+  App part - for toolbar, canvas and chat window

Add below script in index.html 
```html
<!-- top bar -->
  <div class="top-bar">
      <p class="appname">Node.js Whiteboard</p>
      <p class="logintext">Logged in as <span id="username">username</span></p>
  </div>
<1-- app part -->
  <div class="whiteboard-container">
      <div id="shapesToolbar"></div>
      <div id="canvas-holder"></div>
      <div class="chat-div"></div>
  </div>

```
This application uses the following 3 widgets:

+ Shapes toolbar
+ Canvas
+ Chat Window

Download the folder called ['static'](/nodejs-code/whiteboard/static.zip). In it you will find the above three widgets in the ‘js/widgets’ folder, ‘css’ files for styling and ‘images’.

Now that you have all the required files, add them to your ‘html’ file.

### Adding ‘css’ files. 
Copy the below lines of script inside the head tag of index.html

```html
 <link rel="stylesheet" href="static/css/thirdparty/bootstrap/bootstrap.min.css"  />
 <link rel="stylesheet" type="text/css" href="static/css/styles.css" />
 <link rel="stylesheet" type="text/css" href="static/css/toolbar.css" />
 <link rel="stylesheet" type="text/css" href="static/css/chat.css" />
```

### Adding 'js' files - that inlcude JQuery, JQueryUI, sockJSClient, Fabric.js and Shapes.js

```html
    <script type="text/javascript" src="static/js/thirdparty/jquery/jquery-1.8.3.min.js"></script>
    <script type="text/javascript" src="static/js/thirdparty/jquery/jquery-ui-1.9.1.custom.min.js"></script>
    <script type="text/javascript" src="static/js/thirdparty/sockjs/sockjs.min.js"></script>
    <script type="text/javascript" src="static/js/thirdparty/fabric/fabric-all.js"></script>
    <script type="text/javascript" src="static/js/shapes.js"></script>
       
    <!-- components -->
    <script type="text/javascript" src="static/js/widgets/jquery.ui.toolbar.js"></script>
    <script type="text/javascript" src="static/js/widgets/jquery.ui.chatwindow.js"></script>
    <script type="text/javascript" src="static/js/widgets/jquery.ui.canvas.js"></script>
    <script>
         $(document).ready(function() {
	         whiteboardApp.init();
         })
    </script>
```

## Initializing widgets

To initialize the widgets, create a file called 'main.js' and save it under the folder 'static/js/' and add the below code.

```javascript
var whiteboardApp = {

 init:function () {
   this.initToolbar();
   this.initChatWindow();
   this.initCanvas();
 } 

 initToolbar: function () {
     var tb = $("#shapesToolbar").toolbar(
            {
                shapes:whiteboardApp.shapes, // shapes object with shape 'name' and 'iconname' ex: shapes = {  rectangle: {  name: 'rectangle', imagesPath:'/static/images/' } }
                dropTarget:$('.ui-canvas'),
                title:'Shapes',
                shapeSelected:this.onShapeSelect,  // callback
                dropTargetClicked:this.onClickDropTarget   //callback
            }
        );
    },
  
  initChatWindow: function() {
        whiteboardApp.chatWidget = $(".chat-div").chatwindow(
            {
                title: "Chat",
                textSubmitted:this.onTextSubmit
            } );
    },
    
   initCanvas:function() {
        whiteboardApp.canvasWidgetInstance = $("#canvas-holder").canvas(
            {
                title: "Canvas",
                fabric: fabric,
                shapeModified: this.onShapeModify,
                applyModify: this.onApplyModify,
                shapeDeleted: this.onShapeDelete
            } );

        whiteboardApp.canvas = whiteboardApp.canvasWidgetInstance.canvas("getCanvasInstance");
    }
   
}

```
At the start of the application, user will be asked for a name. The method below explains how to perform this action.

```javascript
 initShowPrompt: function () {
        window.showPrompt = function () {
            do {
                whiteboardApp.userName = prompt("Please enter your name( 4 to 15 chars)");
            }
            while (whiteboardApp.userName === null || whiteboardApp.userName.length < 4 || whiteboardApp.userName.length > 15);
            $('#username').text(whiteboardApp.userName);
        };
        window.showPrompt();
    }

```

If you look at init method of each widget, there are a few callback functions which will be invoked at certain events. This is explained in the next chapter.

Since explaining how to create a widget is not within the scope of this document, you can learn about it [here](http://net.tutsplus.com/tutorials/javascript-ajax/coding-your-first-jquery-ui-plugin/).

## Check Point

Open your browser and type http://localhost:4000. This time instead of a blank app with a welcome message, you will see widgets rendered on your screen after you have given your name when prompted for it.

![app with UI](/images/screenshots/nodejs-whiteboard/whiteboard-01.png)

<p><a class="button-plain"  style="padding: 3px 15px;" href="/frameworks/nodejs/nodejs-tutorial/step01-developing-nodejs-app.html">Prev</a>  <a class="button-plain"  style="padding: 3px 15px; float: right;" href="/frameworks/nodejs/nodejs-tutorial/step03-integrating-shapes-canvas.html">Next</a></p>

