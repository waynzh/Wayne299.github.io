---
title: 'UCI js-jquery 学习笔记'
date: 2020-07-09 19:20:43
category: FE
tags: [js基础, Javascript-Jquery-Jon-Duckett]
---

# Javascript & jQuery 学习笔记
Book: *Interactive Front-End Web Development* Jon Duckett 1st edition

## Functions, Methods & Objects
### Function Expressions
- Function Declaration

 ``` javascript
 function area(width, height) {
 	return width*height;
 }
 ```
<!-- more -->

- Function Expression

 ``` javascript
 var area = function(width, height) {  // anonymous function
 	return width*height;
 }
 ```
 
- Immediately Invoked Function Expression(IFFE)

 ``` javascript
 var area = (function() {  
 	var width = 3;
 	var height = 2;
 	return width * height;
 } () )   // (): call the function immediately   ): all as an expression
 ```

### Creat A Object
- Literal Notation

 ``` javascript
 var hotel = {
	 name: 'Wanda',
	 rooms: 40,
	 booked: 25,
	 checkAvailability: function() {
		 return this.rooms - this.booked;
	 }
 };
 ```

- Object Constructor Notation And Templates

 ``` javascript
 // Creat then Add Properties
 var hotel = new Object();
  
 /** Properties */
 hotel.name = 'Wanda'; 
 hotel.rooms = 40;
 hotel.booked = 25;
 /** Method */
 hotel.checkAvailability = function() {
		 return this.rooms - this.booked;
 };
 ```
 
 ``` javascript
 // Creat Object with Properties
 function Hotel(name, rooms, booked) {
	/** Properties */
	this.name = name; 
	this.rooms = rooms;
	this.booked = booked;
	/** Method */
	this.checkAvailability = function() {
		 return this.rooms - this.booked;
	};
 }
 
 var WandaHotel = new Hotel("Wanda", 40, 25);
 ```
 
### Built-in Objects
#### Browser object model
- window
  - document
  - history
  - location: URL of current page
  - navigator: info about browser
  - screen: device's display info
 
#### Document Object Model (DOM)

#### Global Javascript Objects
- String Number Boolean Date Math Regex ... 
- Date Object
  - Year: 4 digits
  - Day: 1-31
  - Month Hour Minutes Seconds Milliseconds: 0- ...


## Decision & Loop
### Short Circuit Value
- short-circuit (stop) as soon as logical operators have a reslut.

 ``` javascript
 /** left (done return the value) => right */
 var artist = "Wayne";
 var result = artist || "Unknown";  // result = "Wayne"
 
 /** left => right(done) */
 var artist = "";
 var result = artist || "Unknown";  // result = "Unknown"
 
 /** create objects or set values */
 var result = artist || {}          // result = {}
 ```
 
### Operator precedence
- the `&&` operator is executed before the `||` operator.

## Document Object Model (DOM)
### Accessing Element
#### Methods that return a single element node.
- `getElementById('id')`
- `querySelector('css')`
#### Methods that return more elements(as a nodelist => collection).
- `getElementsByClassName('class')` 
- `getElementsByTagName('tag')`  getElementBy... ==> live NodeList
- `querySelectorAll('css')`      ==> static NodeList

### Whitespace Nodes
- ul
  - /space: create extra text nodes
  - li
  - /space
  - li
  - ...
- ==> problems: iterate through nodes in the DOM tree/ nextSibling / firstChild
- ==> solutions: use JS library such as jQuery

### Access & Update Element Content
Element node: `<li>` `<em>`  / Text node: "word"

- **nodeValue**: get & update a TEXT NODE.
- **textContent**: ignore any makeup inside the element, return the text value.
- innerText: DON'T USE: never implement in Firefox, obeys CSS(hide elements) and performs bad(check elements visible or not). Use `textContent` instead.
- **innerHTML**: add or remove HTML content, but has SECURITY RISKS.
  (eg. To remove one `<li>` from `<ul>`, need to provide entire fragment minus that `<li>` element).
- DOM manipulation: 
  - `createElement()`: Create the element.
  - `createTextNode()`: Give a content.
  - `appendChild()`: Add text to the element and Add it to the DOM.
  - `removeChild()`: select elment -> parent of element -> `parentNode.removeChild(removeNode)`.
  - `insertBefore()`: `list.insertBefore(newItemFirst, list.firstChild)`

### Attribute Nodes
- `getAttribute()`
- `hasAttribute()`
- `setAttribute('class', 'className')`
- `removeAttribute()`
- `className`: get & change class attribute.
- `id`

## Events
### Event Flow
- Event Bubbling: `<a>`->`<li>`->`<ul>`->`<body>`
- Event Capturing: `Document`->`<html>`->`<body>`->`<ul>`
- Final parameter in the `addEventListener()` decides the Event Flow.
  - false: bubbling phase (default)
  - true: capturing phase (not supported in IE8)

### Event Object
- target(or srcElement): the target of the event.
- type: type of event that was fired.
- cancelable: whether you can canel the default behavior of the element.
- `preventDefault()`  `stopPropagation()`

### Different Types of Events
- W3C DOM Events
- HTML5 Events
  - beforeunload: when user leaves the unsaved changes.
  - hashchange: when the URL hash changes.
  - DOMContentLoaded: after HTML loaded. NOT contain any scripts generated HTML.
- BOM（Browser Object Model）Events

### data-* Attribute (* is lowcase)
custom attribute: `<li data-state="stop">` `<li data-animal-tpye="bird">`

## jQuery
### jQuery selector
- `:header`: selects all elements that are headers, like `<h1> ... <h6>`
- `:lt(3)`: selects elements at an index less than the specified index(3).
- ... P302

### Ready to work
``` javascript
/** as soon as the DOM has loaded */
$(document).ready(function() {
	// script goes here
});
```

``` javascript
/** commonly used */
$(function() {
	// script goes here
});
```
 
- `load` Event: when script relies on assets to have loaded.
- `.ready()` Method: when DOESNOT wait for assets to finish loading.

## AJAX 
- `<script>` tag: synchronous processing model.
- Ajax(Asynchronous Javascript And XML): asynchronous(non-blocking) processing model. Now used to refer to a group of technologies that offer asynchronous functionality in the browser.

### Requests & Responses
- The Request

 ``` javascript
 var xhr = new XMLHttpRequest();
 // 1. HTTP method 2. url 3. if asynchronous
 xhr.open('GET', 'data/test.json', true); 
 xhr.send('search=arduino');
 ```
 
- TheResponse
 ``` javascript
 xhr.onload = function() {
 	if (xhr.status === 200) {
 		// code to process the results from the server
 	}
 }
 ```
 
### Working with Json Data
- `JSON.stringify()`: coverts `javaScript objects` into `String` formatted using Json.
- `JSON.parse()`: coverts `Json data` into a `JavaScript objects` ready to use.
- status property： `200`: OK; `304`: Not modified; `404`: Page not found; `500`: Internal error on the server.
- Run the code locally: Not get a server status property.
 
### Working with data from other servers
- Proxy: to create a file on server that collects the data from remote server.
- JSON-P(JSON with Padding): adding a `<script>` elementto load JSON data.
- CORS(Cross-Origin Resource Sharing): adding extra infor to the HTTP headers.

### jQuery & AJAX
#### $.each()
- json: `.(json, function(key, value) {})`
- array: `.(arr, function(index, value) {})`

#### .find()
`$(data).find('#container')`

- As a selector. Because `$.ajax` can’t load one part of the page.

#### .load('url part-of-the-page')
- `load('url #container')`: the string has a SPACE between `url` and `fragment`.

#### $.ajax()
- More powerful and more complex than `.load()`.
- There are several shorthand methods.

## APIS


