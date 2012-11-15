---
title: Node.js
description: Defining basic shape objects and adding event listeners for shape icons
tags:
    - nodejs
    - express
    - tutorial
---

Now let's define shape objects for basic shapes. Create a new file called `shapes.js` and write code for shape objects. 

Below is the code snippet for rectangle shape. Since we are using `fabric.js` library, shapes we define are fabric.js shape objects.

```javascript
var shapes = {
		rectangle: {
			toolAction: function(args) {
				var rect = new fabric.Rect({
				  top: args.top,
				  left: args.left,
				  width: args.width,
				  height: args.height,
				  fill: args.fillColor
				});
			rect.name = "rectangle";	
			rect.uid = args.uid;
			app.canvas.add(rect);		
			},
		
			defaultValues:{
				top: 200,
				left: 500,
				width: 150,
				height: 50,
				fillColor: '#ccc'
			},
			
			modifyAction: function (args) {
				var obj = util.getObjectById(args.uid);
				var recvdObj = args.object;
				updateProperties(obj, recvdObj);
			}			
		}
}
```

After defining objects for basic shapes, we need to invoke these shape functions to draw them on canvas. 
To do this, we need to first add click handlers for icons in the tool bar.

Below is the code snippet from `main.js'.

```javascript
iconClickHandler: function () {
        $('.image_holder').on('click', function () {
            if (app.currentIcon) {
                app.currentIcon.removeClass('icon_selected');
            }
            $('#freeow').show();
            app.currentIcon = $(this);
            app.currentIcon.addClass('icon_selected');
            $("#freeow").freeow("", "Now click on canvas to add a new shape", {
                classes: ["gray"],
                prepend: false
            });
            app.shapeToDraw = $(this).attr('id');
            app.iconSelected = true;
        })
    }

```
Once the user selects the icon, he is asked to click on the canvas to draw a shape at that co-ordinates.
<br>
Below is the mouseDown function, which will give us mouse pointer and invokes notifyNewShapeEvent method.

```javascript
mouseDown: function (event) {
        if (app.iconSelected) {
            var scrollLeft = $('.canvas-bg').scrollLeft();
            var mouseX = event.pageX - app.canvasOffset.left + scrollLeft; // offset X
            var mouseY = event.pageY - app.canvasOffset.top; // offset Y
            app.notifyNewShapeEvent({
                x: mouseX,
                y: mouseY
            });
            app.iconSelected = false;
            app.currentIcon.removeClass('icon_selected');
            app.currentIcon = null;

```
And notifyNewShapeEvent method sends this data to server to broadcast this event to all users.

```javascript

notifyNewShapeEvent: function (posObj) {
        var uniqId = util.uniqid();
        app.sockjs.send(JSON.stringify({
            action: 'new_shape',
            message: 'new',
            position: posObj,
            shape: app.shapeToDraw,
            uid: uniqId
        }));
        var _data = {};
        _data.position = posObj;
        _data.uid = uniqId;
        _data.shape = app.shapeToDraw;
        app.createNewShape(_data, true);
    }

```
We are assigning unique id to every new shape we create, to fetch the modified shape later from other users when one shape in one client is modified.

CreateNewShape method prepares all required arguments for that shape object and call toolAction method of that shape to draw a shape on canvas.

```javascript
createNewShape: function (data, include) {
        var args = [];
        var argsObj = shapes[data.shape].defaultValues;
        argsObj['left'] = data.position.x;
        argsObj['top'] = data.position.y;
        argsObj['uid'] = data.uid;
        args.push(argsObj)
        shapes[data.shape].toolAction.apply(this, args);
        $("#freeow").hide();
    },

```
###Check point
Open your browser and type http://localhost:4000.<br> 
Now you will be able to draw shapes on canvas.
![Whiteboard](/images/screenshots/nodejs-whiteboard/whiteboard-02.png)

To check the collaboration feature open another tab / window and start one more instance of the app. You will see that whenever user adds a new shape or edit existing shape, same thing is reflected on on other user canvas in real time.

![Whiteboard](/images/screenshots/nodejs-whiteboard/whiteboard-03.png)

<p><a class="button-plain" style="padding: 3px 15px;" href="/frameworks/nodejs/nodejs-tutorial/step06-collaboration-client-side.html">Prev</a>  <a class="button-plain" style="padding: 3px 15px; float: right;" href="/frameworks/nodejs/nodejs-tutorial/step08-deploying-whiteboardapp.html">Next</a></p>
