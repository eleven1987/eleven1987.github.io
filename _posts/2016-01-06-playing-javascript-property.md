---
layout: post
title: "练习：Property of Javascript Object"
date: 2016-01-06 02:01:01 +0800
comments: true
categories: javascript practice
---

今天我想玩儿一下 Javascript Object 的属性。开始：

<!--more-->

## Create Property Through Assignment

如果我创建一个对象

```javascript
var cat = {};
```

我就可以这样给他添加一些属性。

```javascript
cat.name = 'Kitty';
cat.birthYear = 2016;
```

我们就可以访问这些属性了！

```javascript
console.log(cat.name); // => Kitty
console.log(cat.birthYear); // => 2016
```

还可以为他们的属性赋值。

```javascript
cat.name = 'Tom';
console.log(cat.name); // => Tom
```

这真是太简单了。但是有些时候我并不希望一些属性被更改。比如说生日。Reset，重新创建世界！

## Create Property via `defineProperty`

```javascript
var cat = {};
```

这回使用另外一种手段来添加属性。

```javascript
Object.defineProperty(cat, 'birthYear', {
	__proto__: null,
	configurable: false,
	value: 2016
});
```

我仍然能够访问这个属性的值，真是太美妙了。

```javascript
console.log(cat.birthYear); // => 2016
```

而我现在要改变这个值了！

```javascript
cat.birthYear = 2017;
console.log(cat.birthYear); // => 2016
```

还是 2016！成功了，这回没有人可以更改我的生日了……么？我要试一试。

```javascript
delete cat.birthYear;
console.log(cat.birthYear); // => 2016
```

我的天，竟然不能够 delete，太精彩了。那么可以重新定义吗？

```javascript
Object.defineProperty(cat, 'birthYear', {
	__proto__: null,
	configurable: true,
	value: 2017
});

// => Error: TypeError: can't redefine non-configurable property "birthYear"
```

完美防范。

## Make Equivalent

其实他们是一样的。只不过 property 的 descriptor 不一样而已。例如：

```javascript
var cat = { name = 'Kitty' };
var descriptor = Object.getOwnPropertyDescriptor(cat, 'name');
console.log(descriptor); 
```

输出

```json
{
	value: 'Kitty',
	writable: true,
	enumerable: true,
	configurable: true 
}
```

因此只要我这样进行定义：

```javascript
Object.defineProperty(cat, 'name', {
	configurable: true, 
	writable: true, 
	enumerable: true, 
	value: 2016 
});
```

结果就和 `var cat = { name = 'Kitty' }` 一样了。