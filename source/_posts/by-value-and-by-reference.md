---
title: Javascript's by value and by reference
date: 2017-01-23 00:26:39
tags: 
- javascript
- i-dont-know-JS
---

這篇筆記是從 [Javascript30](https://javascript30.com/) 的 __Day14 - JavaScript References VS Copying__ 整理下來的。

> 記得在stackoverflow上面也有人提出其實JS沒有分所謂的`by value`和`by reference`，
> 而是以變數的值是否`immutable`的差別來區分，
> 不過我覺得這樣子的分法對我來說比較好理解。

<!-- more -->


## 1. By Value (Copying)

一般來說只要是 strings, numbers 和 booleans，都可以說是 `by value`。

```javascript
let a = "string";
let b = a;
console.log(a, b); // "string", "string"
b = "another string";
console.log(a, b); // "string", "another string"
```

## 2. By Reference

如果是 `array` 或者 `object`，則會以 `by reference` 的方式傳遞。

```javascript
let person1 = {
	name: "Trina",
	age: 100,
	gender: "female"
};
let person2 = person1;

console.log(person1, person2);
//Object {name: "Trina", age: 100, gender: "female"}, Object {name: "Trina", age: 100, gender: "female"}

person2.name = "Sherry";
console.log(person2.name); //"Sherry"
console.log(person1.name); //"Sherry" --> person1 has been changed too!
```

Array 也是一樣的道理：

```javascript
let players = ["Trina", "Sherry", "Pisuke", "Kuma"];
let team = players;
console.log(players, team);
// ["Trina", "Sherry", "Pisuke", "Kuma"],
// ["Trina", "Sherry", "Pisuke", "Kuma"]

team[3] = "Usagi";
console.log(players);
// ["Trina", "Sherry", "Pisuke", "Usagi"] --> players has been changed too!
	```
要解決這個問題，就必須把`Object`或`Array`直接Copy一份才行。

- Copying an array

	```javascript
	//以下幾種方式皆可行
	const teamCopy1 = players.slice();
	const teamCopy2 = [].concat(players);
	const teamCopy3 = [...players]; //es6
	const teamCopy4 = Array.from(players);
	```
- Copying an object

	```javascript
	const personCopy = Object.assign({}, person, {
		newProperty: "some additional property for personCopy"
	});
	```
	要注意的是以上的方法是 `Shallow copy`，如果 Object 本身是`二維`以上的話，使用上面的方式還是會有 `By reference`的情況發生。

	```javascript
	let me = {
		name: "Trina",
		age: 24,
		social: {
			twitter: "@tri613",
			github: "tri613"
		}
	};

	let me2 = Object.assign({}, me);
	me2.social.twitter = "@nomoney";

	console.log(me.social);
	//{twitter: "@nomoney", github: "tri613"} --> Changed!
	```
	這種情況需要靠`Deep clone`來解決，最簡單 (但效率表現沒那麼好) 的方式 是直接使用JSON格式encode再decode的方式解決。

	```javascript
	const meCopy = JSON.parse(JSON.stringify(me));
	```
	其他的方式可以參考stackoverflow上面的[這篇](http://stackoverflow.com/questions/122102/what-is-the-most-efficient-way-to-deep-clone-an-object-in-javascript)。