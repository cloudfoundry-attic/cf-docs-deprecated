---
title: Node.js
description: Creating basic view
tags:
    - nodejs
    - express
    - tutorial
---

Now lets start writing html page for our Whiteboard app.

This page is divided into 4 areas - top bar, toolbar on the left, canvas in the middle and chat area on the right.

#### Part 1 - top bar

``` html
<div class="top-bar">
	<p class="appname">Node.js Whiteboard</p>
	<p class="logintext">Logged in as <span id="username">username</span></p>
</div>  
		
```

Now add a div which is a container for below 3 parts - toolbar, canvas and chat window.

```html
 <div class="whiteboard-container"></div>

```
Wrap below parts into above div.

#### part 2 - tool bar on the left

```html
<div class="left-bar">
	<div class="box-title">Tools</div>
	<div class="image_holder"><a href="#" rel="tooltip" title="Rectangle"><img class="image_style" id="rectangle" src="static/images/rectangle_g.png" /></img></a></div>
	<hr />
	<div class="image_holder"><a href="#" rel="tooltip" title="Circle"><img class="image_style" id="circle" src="static/images/circle_g.png" /></img></a></div>
	<hr />
	<div class="image_holder"><a href="#" rel="tooltip" title="Triangle"><img class="image_style" id="triangle" src="static/images/triangle_g.png" /></img></a></div>
	<hr />
	<div class="image_holder"><a href="#" rel="tooltip" title="Line"><img class="image_style" id="line" src="static/images/line_g.png" /></img></a></div>
	<hr />
	<div class="image_holder"><a href="#" rel="tooltip" title="Arc"><img class="image_style" id="arc" src="static/images/arc_g.png" /></img></a></div>
	<hr />
	<div class="image_holder"><a href="#" rel="tooltip" title="Arrow-1"><img class="image_style" id="arrow" src="static/images/arrow_g.png" /></img></a></div>
	<hr />
	<div class="image_holder"><a href="#" rel="tooltip" title="Arrow-2"><img class="image_style" id="arrow_bidir" src="static/images/arrow_bidir_g.png" /></img></a></div>
	<hr />
</div>
```            

#### part 3 - canvas in the middle

```html
<div class="canvas-div">
	<div class="canvas-bg">
		<canvas id="canvas_area" width="850" height="500">This text is displayed if your browser 
		   does not support HTML5 Canvas.
		</canvas>
	</div>
</div>
```

#### part 4 - chat window on the right

```html
<div id="chat-div">
	<div class="box-title">Chat</div>
	<div id="chat-text"></div>
	<form id="chat-form">
		<textarea ignoreesc="true" itacorner="6,7" dir="ltr" type="text" width="200" placeholder="Type here"></textarea>
	</form>
</div>
```

Add below lines inside `<header>` tag

```html
<link href="//netdna.bootstrapcdn.com/twitter-bootstrap/2.1.1/css/bootstrap-combined.min.css" rel="stylesheet" />
<link rel="stylesheet" type="text/css" href="static/css/freeow/freeow.css" />
<link rel="stylesheet" type="text/css" href="static/css/styles.css" />
```

Include `js` files in the `<body>` tag

```html
<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
      <!-- SockJS -->
      <script src="http://cdn.sockjs.org/sockjs-0.3.min.js"></script>
      <!-- Twitter Bootstrap -->
      <script src="//netdna.bootstrapcdn.com/twitter-bootstrap/2.1.1/js/bootstrap.min.js"></script>
      <!-- Fabric JS library -->
      <script src="static/js/lib/fabric-all.js"></script>
      <script src="static/js/lib/jquery.freeow.min.js"></script>
      <script src="static/js/shapes.js"></script>
      <script src="static/js/main.js"></script>
      <script type="text/javascript" src="static/js/utilities.js"></script>
```
<br>
<hr>

Here is the CSS file for this. 

Global rules 
```css
body {
color:#333;
}

a {
display:inline-block;
}

blockquote {
-moz-border-bottom-colors:none;
-moz-border-image:none;
-moz-border-left-colors:none;
-moz-border-right-colors:none;
-moz-border-top-colors:none;
background-color:#F9F9F9;
color:rgba(0,0,0,0.75);
font:italic 1.05em/1.55em "Skolar Italic","Times New Roman",serif;
border-color:#41B7DB rgba(0,0,0,0.1) rgba(0,0,0,0.1);
border-style:solid;
border-width:3px 1px 1px;
margin:0 20px;
padding:1em 2em;
}

hr {
border-top:1px solid @grayLighter;
border-style:solid 1px;
margin:0;
}

```
Title bar 

```css

.box-title {
	height: 20px;
	color: #fff;
	background-color: #0088CC;
	padding-left: 15px;
}

```
<br>
Top bar rules
```css
.top-bar {
	height: 50px;
	display: block;
	background-color: #FAFAFA;
	background-image: -moz-linear-gradient(center top , #FFFFFF, #F2F2F2);
	background-repeat: repeat-x;
	line-height:120%; 
	font-size: 18px;
	-moz-box-shadow: 10px 5px 5px #888;
	-webkit-box-shadow: 10px 5px 5px #888;
	box-shadow: 10px 5px 5px #888;
}
.top-bar .appname {
	float: left;
	padding: 13px 5px 5px 10px;
	color: #0088CC;
}

.top-bar .logintext {
	float: right;
	padding: 13px 10px 5px 5px;
	font-size: 14px; 
}
.top-bar .logintext #username {
	color: #0088CC;
}

```

<br>
Main container div 
``` css
.whiteboard-container {
	height: 500px;
	margin-bottom: 10px;
	margin-top: 7px;
}

```
<br>
Left-bar rules
```css
.left-bar {
	float: left;
	width: 70px;
	height: 100%;
	margin: 0px 10px 0px 1px;
	border: 1px solid #ccc;
	background-color:  #FAFAFA
}

```

<br>
Canvas div rules
```css
.canvas-div {
	width:850px;
	max-width: 850px;
	float: left;
	border: 1px solid #ccc;
	background-color: #FAFAFA
}

.canvas-bg {
	width: 100%;
	height: 100%;
	overflow: auto;
}
```
<br>
Finally chat window rules
```css
#chat-div {
	width: 200px;
	float: right;
	border: 1px solid #ccc;
	height: 100%;
	margin-right: 5px;
	background-color:  #FAFAFA
}
#chat-div textarea {
	width:93%;
	border: 1px solid #ccc;
	resize: none;
	height: 26px;
	overflow-y: auto; 
}
#chat-text {
	height: 89%;	
	padding: 0px 5px 0px 5px 
}

```
<hr>


## Check point
Now open your browser and type http://localhost:4000. You will see below output.


*Make sure that you have added required css and images in the corresponding folders.


![Welcome screen](/images/screenshots/nodejs-whiteboard/whiteboard-01.png)

<p><a class="button-plain" style="padding: 3px 15px;" href="/frameworks/nodejs/nodejs-tutorial/step03-developing-nodejs-app.html">Prev</a>  <a class="button-plain" style="padding: 3px 15px; float: right;" href="/frameworks/nodejs/nodejs-tutorial/step05-collaboration-server-side.html">Next</a></p>

