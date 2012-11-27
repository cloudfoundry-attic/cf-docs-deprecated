---
title: Node.js
description: Integrating Chat Feature
tags:
    - nodejs
    - express
    - tutorial
---

## Introduction

This chapter is a continuation of [chapter 3](/frameworks/nodejs/nodejs-tutorial/step03-integrating-shapes-canvas.html). 

## Prerequisites

+ Should have completed [chapter 3](/frameworks/nodejs/nodejs-tutorial/step03-integrating-shapes-canvas), that showed you how to add shapes to canvas by selecting a shape from shapes toolbar.
    
    
## Integrating Chat Feature
    
In the previous exercise we have initialized chat widget with some parameters. Let's have a look at each of them.

Chat Window widget

```javascript
initChatWindow: function() {
  whiteboardApp.chatWidget = $(".chat-div").chatwindow(
      {
          title: "Chat",
          textSubmitted:this.onTextSubmit
      } );
  }
``` 
There are is a  method called `onTextSubmit`. This method is invoked when user submits a text message by using "chat widget's text input"  

Below is the implementation of that method

```javascript
onTextSubmit: function (event) {
 var target = $(event.currentTarget),
      data = {
          userName: whiteboardApp.userName,
          message: target.val(),
          action: 'text'
      };

  whiteboardApp.showTextMessage(data);

  whiteboardApp.sockJSClient.sockJS.send(JSON.stringify(data));

  target.val('').focus();

  return false;
 }
```
And above  and below methods in 'main.js' file.

```javascript
   showTextMessage: function (data) {
        var _userName = (data.userName === undefined) ? "user *"  : data.userName;
        var userNameString  = "<b>[ " + _userName + " ]:</b>"
        whiteboardApp.chatWidget.chatwindow('displayMessage' , userNameString, data.message);
    }
 ```  
Open `sockJSClient.js` and add below code.

In 'onSocketMessage' method:
```javascript
case 'text':
   whiteboardApp.showTextMessage(data);

```  

In 'onSocketOpen' method:

```javascript
   whiteboardApp.chatWidget.chatwindow('displayMessage' ," <b>[ Opened ]:</b>  ", whiteboardApp.sockJSClient.sockJS.protocol);
```
  
 And in 'socketClose' method:
 ```
 whiteboardApp.chatWidget.chatwindow('displayMessage' ," <b>[ closed ]</b>", "");
 ``` 
  
## Check Point

Below is the screenshot that shows two users using the app to chat each other and share drawings.

![app with UI](/images/screenshots/nodejs-whiteboard/whiteboard-03.png)
<p><a class="button-plain"  style="padding: 3px 15px;" href="/frameworks/nodejs/nodejs-tutorial/step03-integrating-shapes-canvas.html">Prev</a>  <a class="button-plain"  style="padding: 3px 15px; float: right;" href="/frameworks/nodejs/nodejs-tutorial/step05-deploying-whiteboardapp.html">Next</a></p>

