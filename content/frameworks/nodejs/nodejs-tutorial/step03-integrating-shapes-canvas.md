---
title: Node.js
description: Integrating Shapes Toolbar and Canvas
tags:
    - nodejs
    - express
    - tutorial
---

## Introduction

This chapter is a continuation of [Chapter 2](/frameworks/nodejs/nodejs-tutorial/step02-creating-ui.html).

## Prerequisites

+ Should have completed [Chapter 2](/frameworks/nodejs/nodejs-tutorial/step02-creating-ui.html), which explains developing and integrating UI elements.
  
## Integrating Shapes Toolbar
    
In the previous exercise you have learned to initialize widgets with some parameters. Let's have a look at each of them.

####Shapes toolbar widget
```javascript
//main.js
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
```
There are two methods called `onShapeSelect` and `onClickDropTarget`. These are invoked when `shapeSelected` event and `dropTargetClicked` events occur in the toolbar widget.

Below is the code to implement these methods:
```javascript
//main.js
onShapeSelect:function(event) {
        whiteboardApp.shapeToDraw = event.shapeSelected;
        whiteboardApp.shapeSelected = true;
 },

 onClickDropTarget:function(event) {
     if (whiteboardApp.shapeSelected) {
         var scrollLeft = $('.ui-canvas').scrollLeft(),
             mouseX = event.pageX - $('.ui-canvas').offset().left + scrollLeft, // offset X
             mouseY = event.pageY - $('.ui-canvas').offset().top; // offset Y
         whiteboardApp.notifyNewShapeEvent({
             x: mouseX,
             y: mouseY
         });
         whiteboardApp.shapeSelected = false;
     }
 }

```

Add above two methods in 'main.js' file.

When user selects a shape from the toolbar, you  update two parameters in the WhiteboardApp (this will be your main namespace for the app). When a user clicks on the canvas, take those x,y coordinates to draw shapes at that location, using notifyShapeEvent method.

Next open main.js file and add the below two methods in it.


```javascript
//main.js
notifyNewShapeEvent: function (posObj) {
        var uniqId = util.getUniqId(),
            _data = {};
        whiteboardApp.sockJSClient.sockJS.send(JSON.stringify({
            action: 'new_shape',
            positionObj: posObj,
            shape: whiteboardApp.shapeToDraw,
            args: [
                {
                  uid: uniqId
                }
            ]
        }));
        _data.positionObj = posObj;
        _data.args = [{uid : uniqId }];
        _data.shape = whiteboardApp.shapeToDraw;
        whiteboardApp.createNewShape(_data);
    }
 
 createNewShape: function (data) {
        var args = [],
            argsObj = whiteboardApp.shapes[data.shape].defaultValues;

        argsObj.left = data.positionObj.x;
        argsObj.top = data.positionObj.y;
        argsObj.uid = data.args[0].uid;
        args.push(argsObj);
        whiteboardApp.shapes[data.shape].toolAction.apply(this, args);
 }
```
 
`notifyNewShapeEvent` notifies the server about the new shape creation, so that all other client's canvas is updated with this new shape. This method also calls `createNewShape` method to draw shape on this client canvas.


Now you need a method that will receive a data broadcasted from server. Now you create a file called 'sockJSClient.js' and save it in a folder 'static\js\'

```javascript
//sockJSClient.js
whiteboardApp.sockJSClient = {
    sockjs_url: '/echo',
    init: function () {
        this.sockJS = new SockJS(this.sockjs_url);
        this.sockJS.onopen = this.onSocketOpen;
        this.sockJS.onmessage = this.onSocketMessage;
        this.sockJS.onclose = this.onSocketClose;
    },

    onSocketOpen: function (conn) {
        $('#spinner').hide();
        whiteboardApp.sockJSClient.sockJS.send(JSON.stringify({
            action: 'text',
            message: 'Joined',
            userName: whiteboardApp.userName
        }));
    },

    onSocketMessage: function (e) {
        var data = JSON.parse(e.data);
        switch (data.action) {
            case 'new_shape':
                whiteboardApp.createNewShape(data);
            break;
            case 'modified':
                whiteboardApp.canvasWidgetInstance.canvas('modifyObject', data);
            break;
            case 'deleted':
                whiteboardApp.canvasWidgetInstance.canvas('deleteObject', data);
            break;
        }
    },

    onSocketClose: function (conn) {
        whiteboardApp.sockJSClient.sockJS.send(JSON.stringify({
            action: 'text',
            message: 'Left',
            userName: whiteboardApp.userName
        }));
    }
};

``` 

You can send a JSON object to the server using `sockJS.send` method as shown above.

Load this file by adding it to `index.html` as shown below.

```javascript
 //index.html
 <script type="text/javascript" src="static/js/sockJSClient.js"></script>

``` 
 
## Check Point

Open your browser and type http://localhost:4000. Now you can add shapes to the canvas as shown below:

![app with UI](/images/screenshots/nodejs-whiteboard/whiteboard-02.png)


To check the collaboration feature, open another instance of the app in a different tab/window. You will notice that whenever one user adds a shape to the canvas, the same shape is added in all other users’ canvases. However when you modify a shape, it does not get updated on their canvases. To do this, you need to implement the required methods.
Add below methods in `main.js`

```javascript
//main.js
onShapeModify:function(event, data) {
     whiteboardApp.sockJSClient.sockJS.send(whiteboardApp.getModifiedShapeJSON(data, "modified"));
 },

 onShapeDelete:function(event, data) {
     whiteboardApp.sockJSClient.sockJS.send(whiteboardApp.getModifiedShapeJSON(data, "deleted"));
 },

 getModifiedShapeJSON: function (shape, _action) {
     var _obj = JSON.stringify({
         action: _action,
         name: shape.name,
         args: [{
             uid: shape.uid,
             object: shape
         }]
     });
     return _obj;
 },
 
 onApplyModify: function(event, data) {
     whiteboardApp.shapes[data.name].modifyAction.apply(this, data.args);
 }

```
The above methods are triggered by canvas widget events. 

##Note

'init()' method in `main.js` should have following lines of code. Order of calling these methods should be same as below.

```javascript
// main.js 
init: function () { 
   this.initChatWindow();
   this.initCanvas();
   this.initToolbar();
   this.sockJSClient.init();
}
```

## Check Point

Open two instances of the app and check if the collaboration feature works.

<p><a class="button-plain"  style="padding: 3px 15px;" href="/frameworks/nodejs/nodejs-tutorial/step02-creating-ui.html">Prev</a>  <a class="button-plain"  style="padding: 3px 15px; float: right;" href="/frameworks/nodejs/nodejs-tutorial/step04-integrating-chat.html">Next</a></p>

