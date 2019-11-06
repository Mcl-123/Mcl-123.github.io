---
title: import 与 export
date: 2018-09-04 21:14:03
categories: "JavaScript"
tags:
    - js
    - import
    - export
    - export default
---

# 前言

在新的项目组的代码中, 看到了 require, 再考虑到 es6 中的 import 和 export, 有点方. 所以, 想在这里系统的总结下, 也便于给小伙伴们学习下.

<!-- more -->

# import

import命令用于输入其他模块提供的功能.

## 语法

```javascript
import defaultExport from "module-name";
import * as name from "module-name";
import { export } from "module-name";
import { export as alias } from "module-name";
import { export1 , export2 } from "module-name";
import { export1 , export2 as alias2 , [...] } from "module-name";
import defaultExport, { export [ , [...] ] } from "module-name";
import defaultExport, * as name from "module-name";
import "module-name";
```

defaultExport: 将引用模块默认导出的名称。

module-name: 要导入的模块。这通常是包含模块的.js文件的相对或绝对路径名，不包括.js扩展名。某些打包工具可以允许或要求使用该扩展；检查你的运行环境。只允许单引号和双引号的字符串。

name: 引用时将用作一种命名空间的模块对象的名称。

export, exportN: 要导入的导出名称。

alias, aliasN: 将引用指定的导入的名称。

## 使用

### 导入整个模块的内容

这将myModule插入当前作用域，其中包含来自位于/modules/my-module.js文件中导出的所有模块:

```javascript
import * as myModule from '/modules/my-module.js';
```

在这里，访问导出意味着使用模块名称（在这种情况下为“myModule”）作为命名空间。例如，如果上面导入的模块包含一个doAllTheAmazingThings()，你可以这样调用：

```javascript
myModule.doAllTheAmazingThings();
```

### 导入单个导出

给定一个名为myExport的对象或值，它已经从模块my-module导出（因为整个模块被导出）或显式地导出（使用export语句），将myExport插入当前作用域:

```javascript
import {myExport} from '/modules/my-module.js';
```

### 导入多个导出

将foo和bar插入当前作用域:

```javascript
import {foo, bar} from '/modules/my-module.js';

// CommonJS模块
let { foo, bar } = require('fs');
```

注: CommonJS require 的实质是整体加载fs模块（即加载fs的所有方法），生成一个对象（_fs），然后再从这个对象上面读取 2 个方法。这种加载称为“运行时加载”，因为只有运行时才能得到这个对象，导致完全没办法在编译时做“静态优化”。

而 import 实质是从fs模块加载 2 个方法，其他方法不加载。这种加载称为“编译时加载”或者静态加载，即 ES6 可以在编译时就完成模块加载，效率要比 CommonJS 模块的加载方式高。当然，这也导致了没法引用 ES6 模块本身，因为它不是对象。

### 导入带有别名的导出

导入时可以重命名导出。例如，将shortName插入当前作用域:

```javascript
import {reallyReallyLongModuleExportName as shortName} from '/modules/my-module.js';
```

### 导入默认值

在 default export（无论是对象，函数，类等）有效时可用。然后可以使用import语句来导入这样的默认值:

```javascript
import myDefault from "my-module";
```

也可以同时将default语法与上述用法（命名空间导入或命名导入）一起使用。在这种情况下，default导入必须首先声明:

```javascript
import myDefault, * as myModule from "my-module";
// 或者
import myDefault, {foo, bar} from "my-module";
```

## 注意

### 只读接口

import命令输入的变量都是只读的，因为它的本质是输入接口。也就是说，不允许在加载模块的脚本里面，改写接口:

```javascript
import {a} from './xxx.js'

a = {}; // Syntax Error : 'a' is read-only;
```

上面代码中，脚本加载了变量a，对其重新赋值就会报错，因为a是一个只读的接口。但是，如果a是一个对象，改写a的属性是允许的: 

```javascript
import {a} from './xxx.js'

a.foo = 'hello'; // 合法操作
```

注: 上面代码中，a的属性可以成功改写，并且其他模块也可以读到改写后的值。不过，这种写法很难查错，建议凡是输入的变量，都当作完全只读，轻易不要改变它的属性。

### 模块文件的位置

后面的 from 指定模块文件的位置，可以是相对路径，也可以是绝对路径，.js后缀可以省略。如果只是模块名，不带有路径，那么必须有配置文件，告诉 JavaScript 引擎该模块的位置.

### 提升效果

import 命令具有提升效果，会提升到整个模块的头部，首先执行:

```javascript
foo();

import { foo } from 'my_module';
```

原因是: import 命令是编译阶段执行的，在代码运行之前。

### 不能使用表达式和变量

由于import是静态执行，所以不能使用表达式和变量，这些只有在运行时才能得到结果的语法结构:

```javascript
// 报错
import { 'f' + 'oo' } from 'my_module';

// 报错
let module = 'my_module';
import { foo } from module;

// 报错
if (x === 1) {
  import { foo } from 'module1';
} else {
  import { foo } from 'module2';
}
```

### Singleton 模式

import语句是 Singleton 模式:

```javascript
import { foo } from 'my_module';
import { bar } from 'my_module';

// 等同于
import { foo, bar } from 'my_module';
```

# export

export命令用于规定模块的对外接口.

## 语法

### 导出变量

```javascript
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;
```

等同于:

```javascript
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export {firstName, lastName, year};
```

注: 应该优先考虑使用这种写法。因为这样就可以在脚本尾部，一眼看清楚输出了哪些变量。

### 导出函数

export命令除了输出变量，还可以输出函数或类（class）:

```kotlin
export function multiply(x, y) {
  return x * y;
};
```

### 导出重命名的变量或函数

```javascript
function v1() { ... }
function v2() { ... }

export {
  v1 as streamV1,
  v2 as streamV2,
  v2 as streamLatestVersion
};
```

上面代码使用as关键字，重命名了函数v1和v2的对外接口。重命名后，v2可以用不同的名字输出两次。

## 导出注意

### 一一对应关系

export命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系:

```javascript
// 报错
export 1;

// 报错
var m = 1;
export m;
```

上面两种写法都会报错，因为没有提供对外的接口:

```javascript
// 写法一
export var m = 1;

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};
```

它们的实质是，在接口名与模块内部变量之间，建立了一一对应的关系。

### 处于模块顶级

export命令可以出现在模块的任何位置，只要处于模块顶层就可以。如果处于块级作用域内，就会报错:

```javascript
function foo() {
  export default 'bar' // SyntaxError
}
foo()
```

## export default

### 使用

使用import命令的时候，用户需要知道所要加载的变量名或函数名，否则无法加载。但是，用户肯定希望快速上手，未必愿意阅读文档，去了解模块有哪些属性和方法。
为了给用户提供方便，让他们不用阅读文档就能加载模块，就要用到export default命令，为模块指定默认输出:

```javascript
export default function () {
  console.log('foo');
}
```

其他模块加载该模块时，import命令可以为该匿名函数指定任意名字:

```javascript
import customName from './export-default';

customName(); // 'foo'
```

### 对比下 export 和 export default

export default命令的本质是将后面的值，赋给default变量.
export 是指定对外的接口

```javascript
// 第一组
export default function crc32() { // 输出
  // ...
}

import crc32 from 'crc32'; // 输入

// 第二组
export function crc32() { // 输出
  // ...
};

import {crc32} from 'crc32'; // 输入
```

```javascript
// 正确
export var a = 1;

// 正确
var a = 1;
export default a;

// 错误
export default var a = 1;

// 正确
export default 42;

// 报错
export 42;
```

# export 和 import 的复合使用

## 基本使用

如果在一个模块之中，先输入后输出同一个模块，import语句可以与export语句写在一起:

```javascript
export { foo, bar } from 'my_module';

// 可以简单理解为(但不同)
import { foo, bar } from 'my_module';
export { foo, bar };
```

注: export和import语句可以结合在一起，写成一行。但需要注意的是，写成一行以后，foo和bar实际上并没有被导入当前模块，只是相当于对外转发了这两个接口，导致当前模块不能直接使用foo和bar.

```javascript
// 接口改名
export { foo as myFoo } from 'my_module';

// 整体输出
export * from 'my_module';

// 默认接口
export { default } from 'foo';

// 具名接口改为默认接口
export { es6 as default } from './someModule';

// 等同于
import { es6 } from './someModule';
export default es6;

// 具名接口改为默认接口
export { es6 as default } from './someModule';

// 等同于
import { es6 } from './someModule';
export default es6;

// 默认接口也可以改名为具名接口
export { default as es6 } from './someModule';
```

## 跨模块常量

const声明的常量只在当前代码块有效。如果想设置跨模块的常量（即跨多个文件），或者说一个值要被多个模块共享，可以采用下面的写法:

```javascript
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
```

如果要使用的常量非常多，可以建一个专门的constants目录，将各种常量写在不同的文件里面，保存在该目录下:

```javascript
// constants/db.js
export const db = {
  url: 'http://my.couchdbserver.local:5984',
  admin_username: 'admin',
  admin_password: 'admin password'
};

// constants/user.js
export const users = ['root', 'admin', 'staff', 'ceo', 'chief', 'moderator'];
```

然后，将这些文件输出的常量，合并在index.js里面:

// constants/index.js
export {db} from './db';
export {users} from './users';

使用的时候，直接加载index.js就可以了:

```javascript
// script.js
import {db, users} from './constants/index';
```