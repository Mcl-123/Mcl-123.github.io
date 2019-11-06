---
title: exports 与 module.exports
date: 2018-09-04 22:34:35
categories: "JavaScript"
tags:
    - js
    - exports
    - node
---

# 前言

前面有总结 es6 中的 import 和 export, 顺带着一起总结下 node 中的 exports 和 module.exports.

<!-- more -->

# exports 和 module.exports

在一个node执行一个文件时，会给这个文件内生成一个 exports和module对象，
而module又有一个exports属性。他们之间的关系如下图，都指向一块{}内存区域.

```javascript
exports = module.exports = {};
```

![exports与module_exports](/images/exports与module_exports.png)

```javascript
//utils.js
let a = 100;

console.log(module.exports); //能打印出结果为：{}
console.log(exports); //能打印出结果为：{}

exports.a = 200; //这里辛苦劳作帮 module.exports 的内容给改成 {a : 200}

exports = '指向其他内存区'; //这里把exports的指向指走

//test.js

var a = require('/utils');
console.log(a) // 打印为 {a : 200} 
```

```javascript
test.js

var a = {name: 1};  // 类比 module.exports
var b = a;          // 类比 exports

console.log(a);
console.log(b);

b.name = 2;         // 将 a 的值改了
console.log(a);
console.log(b);

var b = {name: 3};  // b 重新赋值了, 但是 a 不变
console.log(a);
console.log(b);

运行 test.js 结果为：

{ name: 1 }
{ name: 1 }
{ name: 2 }
{ name: 2 }
{ name: 2 }
{ name: 3 }
```

总结:

- module.exports 初始值为一个空对象 {}
- exports 是指向的 module.exports 的引用
- require() 返回的是 module.exports 而不是 exports

所以，为了避免糊涂，尽量都用 module.exports 导出，然后用require导入.