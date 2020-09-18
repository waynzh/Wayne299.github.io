---
title: 'Sass'
date: 2020-8-28 8:20:21
categories: 前端
tags: css
---

# Intro to Sass
## Variables `$`

```scss
$font-stack:    Helvetica, sans-serif;
$primary-color: #333;

body {
  font: 100% $font-stack;
  color: $primary-color;
}
```

## Interpolation `#{}`
- Interpolation is useful for injecting values into strings.


## Nesting
<!--more-->
```scss
nav {
	ul {
		margin: 0;
		padding: 0;
		list-style: none;
	}

	li { display: inline-block; }

	a {
		padding: 6px 12px;
		text-decoration: none;
	}
}
```


## Partials
- Partial files start with a leading underscore. 
- When we're using `@use 'base'`, don't need to include the file extension.
- Only `Dart Sass` currently supports `@use`. Users of other implementations must use the `@import` rule instead.

```scss
// _base.scss
$font-stack:    Helvetica, sans-serif;
$primary-color: #333;

body {
  font: 100% $font-stack;
  color: $primary-color;
}
```

```scss
// styles.scss
@use 'base';  // `base` as a module

.inverse {
  background-color: base.$primary-color;  
  color: white;
}
```

## Mixins
- Make groups of CSS declarations that you want to reuse.
  1. To use the `@mixin` directive and give it a name
  2. Use the variable `$property`
  3. Use it starting with `@include` followed by the name of the mixin.

```scss
@mixin transform($property) {
  -webkit-transform: $property;
  -ms-transform: $property;
  transform: $property;
}

.box { @include transform(rotate(30deg)); }
```

## Extend
- Using `@extend` lets you share a set of CSS properties from one selector to another.

```scss
%message-shared {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}

.message {
  @extend %message-shared;
}

.success {
  @extend %message-shared;
  border-color: green;
}

.error {
  @extend %message-shared;
  border-color: red;
}
```

## Pseudo Classes
- use `&:hover` `&:after` 

```scss
a{
	color: black;
	text-decoration: none;
	
	&:hover {
		color: white;
	}
}

```

```scss
@mixin hoverEvent {
	&:hover {
		color: white;
		text-decoration: underline;
	}
}

a{
	color: black;
	text-decoration: none;
	
	@include hoverEvent;	
}

```

## Mathematical Operators
- Unitless numbers can be used with numbers of any unit. `100px + 50; // 150px`

### diffrent meanings of `-`
- between two numbers regardless of whitespace, which is parsed as subtraction.

```scss
$number: 2;
1 -$number 3; // -1 3
1 (-$number) 3; // 1 -2 3
```

### different meanings of `/`
- `/` will print the result as `divided` instead of `slash-separated` if one of these conditions is met:
1. The result is stored in a variable or returned by a function.
2. The operation is surrounded by parentheses, unless those parentheses are outside a list that contains the operation.
3. The result is used as part of another operation (other than /).

```scss
15px / 30px            // 15px/30px
$result: 15px / 30px;  // Situation1: 0.5

(15px/30px);           // Situation2: 0.5
(bold 15px/30px sans-serif); // bold 15px/30px sans-serif

15px/30px + 1;         // Situation3: 1.5
```

- Want to force `/` to be used as a `separator`, can write it as `#{<expression>} / #{<expression>}`.

```scss
#{10px + 5px} / 30px;   // 15px/30px
```

### Creating a Grid:

```scss
@mixin grid($cols, $mgn) {
	float: left;
	margin-right: $mgn;
	margin-bottom: $mgn;
	width: (100% - ($cols - 1) * $mgn)) / $cols;
	&:nth-child( #{ $cols }n ) {
		margin-right: 0;
	}
}

#imgDiv li {
	@include grid(4, 2%);
	img {
		wid	th: 100%;
	}
}
```

## Color Functions
- `lighten($color, $amount)`: hover funcitons.
- `complement($color)`: hover to change to complementary color.
- [sass-documentation](https://sass-lang.com/documentation/modules)

## `@if` statements

```scss
@mixin mQ($arg...) {
	@if length($arg) == 1 {
		@media screen and (max-width: nth($arg, 1)) {
			@content;
		}
	}
	@if length($arg) == 2 {
		@media screen and (max-width: nth($arg, 1)) and (min-width: nth($arg, 2)){
			@content;
		}
	}
}

```




