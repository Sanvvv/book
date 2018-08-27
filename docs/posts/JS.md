# 数组
## 方法
### sort

V8 引擎的 sort 方法，在 10 以内是插入排序，10 以上（注释里写的是 22）是快速排序（由于快排不稳定，所以不太适合直接使用 sort 来排之前已经排序好的数据）

### 转换为数组

```js
Array.from(string)
[].concat(...rest)
```

## 类数组

拥有 length 属性，其它属性（索引）为非负整数（但是不具有数组所具有的方法，需要使用 Function.call 间接调用）

包括：arguments、document.getElementsByTagName()

### 判断

```js
function isArrayLike(o) {
	if (o &&                              // o is not null, undefined, etc.
		typeof o === 'object' &&            // o is an object
		isFinite(o.length) &&               // o.length is a finite number
		o.length >= 0 &&                    // o.length is non-negative
		o.length===Math.floor(o.length) &&  // o.length is an integer
		o.length < 4294967296)              // o.length < 2^32
		return true;                        // Then o is array-like
	else
		return false;                       // Otherwise it is not
}
```

### 转化为数组

```js
// 由于 IE9 以前的版本使用 COM 来实现 DOM ，所以不支持这种方法
Array.prototype.slice.call(arrayLike)
Array.prototype.splice.call(arrayLike, 0)
Array.prototype.concat.apply([], arrayLike)
// es6 方法，从一个类数组或可迭代对象中创建一个新的数组实例
Array.from(arrayLike)
```

# Map & Set
## Map
### 描述

保存键值对

以插入顺序迭代其元素，for...of 循环每次迭代返回一个 [key, value] 数组

使用 "SameValueZero" 比较键是否相等（包括 NaN ，正负 0）

### 与对象的区别

- 对象的键只能是字符串或者 Symbols ，但是 Map 的键可以是任意值（包括对象和函数）
- Map 可以通过 size 属性直接获取键值对个数
- Map 可迭代，对象需要先获取它的键数组然后再迭代
- 性能优势

## Set
### 描述

允许存储任何类型的唯一值

## WeakMap
### 描述

保存键值对，其中的键是弱引用的，必须为对象

### 与 Map 的区别

- 只能存放对象
- 不可枚举，没有 forEach，没有 size （因为弱引用的关系，无法得到确定的结果）

### 用法

与 Dom 关联，并在不需要 Dom 的时候能够立即被销毁防止内存泄漏

在构造函数里实现私有变量（保存 this 为键，通过 this 获取值），同时不用担心内存泄漏的问题

## WeakSet
### 描述

允许将弱保持对象存储在一个集合中，并且每个对象只能出现一次

# 函数

js 中的函数是一等公民

## 原型
### prototype

只有函数有 prototype 属性，该属性指向一个对象，对象具有一个指向函数本身的 constructor 属性

### 原型链

试图访问对象的属性时，如果在对象上找不到就会沿着 `__proto__` 找（找到原型的 prototype），直到找到或者为 null（倒数第二层是 Object.prototype，之后就是 null）

> 注意和作用域链区分

### 无原型对象

```js
Object.create(null)
```

## 继承

[https://github.com/mqyqingfeng/Blog/issues/16]

### 原型链继承

```js
Child.prototype = new Parent() // 1
Child.prototype = Object.create(Parent.prototype) // 2
Child.prototype.__proto__ = Parent.prototype // 3
Object.setPrototypeof(Child.prototype, Parent.prototype) // 4
```

问题：

1. 引用类型的属性被所有实例共享

2. 在创建 Child 实例时不能向 Parent 传参

### 借用构造函数

```js
function Parent () {
  this.names = ['kevin', 'daisy']
}

function Child () {
  Parent.call(this)
}
```

优点：

1. 避免了引用类型的属性被所有实例共享（在上面的例子中 names 是不共享的）

2. 可以在 Child 中向 Parent 传参

缺点：

1. 无法继承 Parent.prototype 中的属性，方法都需要在构造函数中定义，每次创建实例都会创建一遍方法

### 组合继承

混合上述两种方式，属性在构造函数中定义，方法在 prototype 中定义，并在最后让 Child.prototype 指向 Parent.prototype

```js
function Parent (name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}

Parent.prototype.getName = function () {
  console.log(this.name)
}

function Child (name, age) {
  Parent.call(this, name)
  this.age = age
}

Child.prototype = new Parent()
Child.prototype.constructor = Child
```

缺点：

1. 在 `Child.prototype = new Parent()` 中调用了一次 Parent ，在 `Parent.call(this, name)` 中又调用了一次，所以 Child.prototype 和 实例中都会有 color 属性（解决方法请看寄生组合式继承）

### 寄生式继承

创建一个仅用于封装继承过程的函数，该函数在内部以某种形式来做增强对象，最后返回对象

```js
function createObj (o) {
  var clone = Object.create(o)
  clone.sayName = function () {
    console.log('hi')
  }
  return clone
}
```

缺点：

1. 跟借用构造函数模式一样，每次创建对象都会创建一遍方法

### 寄生组合式继承

通过让 Child.prototype 间接访问到 Parent.prototype 的方式解决问题

```js
function Parent (name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}

Parent.prototype.getName = function () {
  console.log(this.name)
}

function Child (name, age) {
  Parent.call(this, name)
  this.age = age
}

// 这里不一样
var F = function () {}
F.prototype = Parent.prototype
Child.prototype = new F()
```

## 闭包

- 闭包是javascript支持头等函数的一种方式，它是一个能够引用其内部作用域变量(在本作用域第一次声明的变量)的表达式，这个表达式可以赋值给某个变量，可以作为参数传递给函数，也可以作为一个函数返回值返回
- MDN：闭包是指那些能够访问自由变量的函数（自由变量是指在函数中使用的，但既不是函数参数也不是函数的局部变量的变量）
  - 由于函数访问全局变量也相当于在访问自由变量，所以技术角度上可以认为所有函数都是闭包（权威指南就这么说）
  - 实践角度：即使创建它的上下文已经销毁，它仍然存在；在代码中引用了自由变量
- 闭包是函数开始执行的时候被分配的一个栈帧，在函数执行结束返回后仍不会被释放(就好像一个栈帧被分配在堆里而不是栈里！)
- 即使父级函数的执行上下文已经在调用栈中被弹出了，闭包函数还是可以访问到父级函数的变量对象，是因为内部函数的作用域链中包含父级函数的变量对象
- 应用：柯里化、模拟私有变量或者方法

```js
var currying = function(fun) {
	//格式化arguments
	var args = Array.prototype.slice.call(arguments, 1);
	return function() {
    	//收集所有的参数在同一个数组中，进行计算
    	var _args = args.concat(Array.prototype.slice.call(arguments));
    	return fun.apply(null, _args);
	};
}
```

# 判断类型
## 数组

# 事件

 事件是在编程时系统内发生的动作或者发生的事情 —— 系统会在事件出现的时候触发某种信号并且提供一个自动加载某种动作的机制（比如运行代码）

## 模型

当一个事件发生在具有父元素的元素上时，现代浏览器运行两个不同的阶段 - 捕获阶段和冒泡阶段

默认情况下，所有事件处理程序都在冒泡阶段进行注册

### 捕获

浏览器检查元素的最外层祖先 `<html>` ，是否在捕获阶段中注册了一个事件处理程序，如果是，则运行它，之后再移动到下一个元素，直至到达实际触发事件的元素

### 冒泡

浏览器检查实际触发事件的元素是否在冒泡阶段中注册了事件处理程序，如果是，则运行它，之后再向直接祖先元素移动，直至到达 `<html>`

### 阻止默认行为

```js
e.preventDefault()
```

### 阻止冒泡

```js
e.stopPropagation()
```

### return false

同时阻止默认行为和冒泡，只在 HTML 事件属性 和 DOM0 级事件处理方法中有效

### 委托

如果想要在大量子元素中单击任何一个都可以运行一段代码，可以将事件监听器设置在其父节点上

# Ajax
## 状态值

0: (未初始化)还没有调用open()方法

1: (载入)已经调用open()方法，正在派发请求，`send()`方法还未被调用

2: (载入完成)send()已经调用，响应头和响应状态已经返回

3: (交互)响应体下载中; `responseText`中已经获取了部分数据

4: (完成)响应内容已经解析完成，用户可以调用

# JS 运行原理

引擎：调用栈（Call Stack）、Memory Heap

Web 环境：运行上下文（Runtime）、事件循环（Event Loop）

引擎负责整个代码的编译以及运行，编译器则负责词法分析、语法分析、代码生成等工作而作用域则负责维护所有的标识符（变量）

## Runtime

Web APIs：DOM、XHR、setTimeout() & Node APIs

## 执行上下文

# 模块
## 规范
### AMD（Asyncchronous Module Definition）

是 RequireJS 在推广过程中对模块定义的规范化的产出

异步加载（适合浏览器环境），通过回调，在加载完模块后执行依赖它的语句  

### CMD（Common Module Definition）

是 SeaJS 在推广过程中对模块定义的规范化产出

### CommonJS

- 每个模块内部，module 变量代表当前模块，所有模块都是 Module 的实例
- module 的 exports 属性是对外的接口，加载某个模块，其实是加载该模块的 module.exports 属性
- require：该方法读取一个文件并执行，最后返回文件内部的 exports 对象，require 的值是值拷贝，影响不到模块内部
- 模块可以多次加载，但是只会在第一次加载时运行一次，之后结果就缓存了（即使在其他模块中再次 require 也只会返回第一次加载的缓存）
- 循环加载：如果发生模块的循环加载，即A加载B，B又加载A，则B将加载A的不完整版本（只输出已经执行的部分）

### ES2015 Module

- 模块中的代码自动运行在严格模式下，其中的顶级变量不会添加到全局作用域中
- import 会在编译期间提升到模块头部
- 由于引擎需要静态决定 import 和 export 的内容，两者只能在顶级作用域下使用
- import 语句只创建了只读变量（是引用的），但是 import 而来的函数可以修改与其同一个模块中定义的变量
- 关于在浏览器中引入：使用 script 标签（type="module"，总是 defer）的 src 属性或者在 script 内联代码中 import
- 循环加载：遇到模块加载命令 import 时（如果不是循环加载的情况就正常执行需要加载的模块），不会去执行模块，而是只生成一个引用，等到真的需要用到时，再到模块里面去取值（因为是动态引用，没有缓存值）

```js
/* 模块语法 */
export let name = 'neko'

export function sum (a, b) {
  return a + b
}

function multiply (a, b) {
  return a * b
}
export { multiply }

/* 重命名，import 同样可以 */
export { sum as add }

/* 输出默认值，只能给模块设定一个默认值，不必须命名 */
export default function (a, b) {
  return a + b
}
export default sum
export { sum as default }

/* 类似于 const ，不可再次定义同名变量 */
import { identifier1, identifier2 } from './example.js'

/* 将整个模块引入，所有的输出可以以变量的形式访问 */
import * as example from './example.js'

/* 多次 import 同样的模块，该模块也只会执行一次，第一次 import 之后，其实例化的模块会驻留在内存中并随时可由另一个 import 语句引用 */

/* 引入默认值，需要注意的是没有使用花括号 */
import sum from './example.js'

/* 同时引入 */
import sum, { color } from "./example.js"
import { default as sum, color} from "./example.js"

/* 再输出 */
export { sum } from "./example.js"

/* 全局引入 */
/* 一些模块可能并不输出任何内容，相反，他们只是修改全局作用域内的对象（他们是可以访问到全局作用域的） */
Array.prototype.pushAll = function (items) {
  return this.push(...items)
}
/* 然后引入即可 */
import "./example.js"
```

## 问题
### AMD 和 CMD 的区别

- 对于依赖的模块，AMD 是提前执行（不过 RequireJS 在 2.0 之后也改成可以延迟执行），CMD 是延迟执行（lazy loading?）
- AMD 直接加载完依赖再执行回调，CMD 是在执行需要依赖的代码前再加载这个依赖
- AMD 的 API 默认是**一个当多个用**，CMD 的 API 严格区分，推崇**职责单一**。比如 AMD 里，require 分全局 require 和局部 require，都叫 require。CMD 里，没有全局 require，而是根据模块系统的完备性，提供 seajs.use 来实现模块系统的加载启动。CMD 里，每个 API 都**简单纯粹**

### CommonJS 与 ES2015 Modules 的区别

- [https://www.zhihu.com/question/56820346]
- [http://es6.ruanyifeng.com/#docs/module-loader]
- [https://github.com/olifer655/commonJS/issues/6]
- CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用
- CommonJS 模块是运行时加载，ES6 模块是编译时（静态编译）输出接口（ES6 入门）

### 为什么在 webpack 中 import 和 require 都可以使用? 

CommonJS 由 webpack 默认支持，而 import 由 babel 支持

### 循环依赖

```js
// a.js
console.log('a starting');
exports.done = false;
const b = require('./b');
console.log('in a, b.done =', b.done);
exports.done = true;
console.log('a done');

// b.js
console.log('b starting');
exports.done = false;
const a = require('./a');
console.log('in b, a.done =', a.done);
exports.done = true;
console.log('b done');

// node a.js
// 执行结果：
// a starting
// b starting
// in b, a.done = false
// b done
// in a, b.done = true
// a done

// a.js
console.log('a starting')
import {foo} from './b';
console.log('in b, foo:', foo);
export const bar = 2;
console.log('a done');

// b.js
console.log('b starting');
import {bar} from './a';
export const foo = 'foo';
console.log('in a, bar:', bar);
setTimeout(() => {
  console.log('in a, setTimeout bar:', bar);
})
console.log('b done');

// babel-node a.js
// 执行结果：
// b starting
// in a, bar: undefined
// b done
// a starting
// in b, foo: foo
// a done
// in a, setTimeout bar: 2
```

# 高级
## 严格模式

  - 必须要事先声明变量
  - 直接使用函数调用（不是作为方法）的时候，this 是 undefined （而不是 window）
  - 禁止给函数提供多个相同的参数
  - 去除了 with 语句
  - 会使引起静默失败的赋值语句抛出错误
  - 禁止八进制数字语法（在 es6 中需要在数字前面加上 0o）
  - 禁止设置 primitive 值的属性
  - eval 不再为上层范围引入新变量（eval 不会在当前作用域中声明变量（虽然 eval 能访问到当前作用域），仅为运行的代码创建变量）
    - eval 具有某种意义上的动态作用域
    - 将 eval 赋值给其他变量，调用的时候是在全局的作用域中执行（而不是当前）
  - 禁止删除变量声明 （var x; delete x;）
  - 在非严格模式中 arguments 和函数参数完全绑定在一起，会一起修改，严格模式下不会（符合对指针的理解）
  - 不再支持 arguments.callee（指向当前正在进行的函数）和 func.caller
  - this 不再会被强制转换为对象（包装对象），并且未指定的 this 会变成 undefined
  - 禁止了不在脚本或者函数层面的函数声明（在条件、循环语句中声明）

## 类型化数组（Typed Arrays）

是一种类似数组的对象，用来处理来自网络协议、二进制文件格式和原始图形缓冲区等源的二进制数据，还可用于管理具有已知字节布局的内存中二进制数据（MSDN）

由于是直接操作二进制数据，效率比常规的数组更高，适合于有大量二进制数据操作需求的场合

为了达到最大的灵活性和效率，类型数组将实现拆分为缓冲（ArrayBuffer）和视图（类型数组视图或DataView）两部分

### ArrayBuffer

是一种数据类型，用来表示一个通用的、固定长度的二进制数据缓冲区（没有机制获取数据，仅提供缓冲作用），需要使用视图来访问其中的内容

### 类型数组视图（TypedArray?）

具有自描述性的名字和所有常用的数值类型像 Int8，Uint32，Float64 等等

是类数组，需要某些特定数组操作的时候可以使用 Array.from 转换为数组

| 数据类型 | 字节长度 | 含义 | 对应的C语言类型 |
| ------- | ---- | ---------------- | -------------- |
| Int8    | 1    | 8位带符号整数          | signed char    |
| Uint8   | 1    | 8位不带符号整数         | unsigned char  |
| Uint8C  | 1    | 8位不带符号整数（自动过滤溢出） | unsigned char  |
| Int16   | 2    | 16位带符号整数         | short          |
| Uint16  | 2    | 16位不带符号整数        | unsigned short |
| Int32   | 4    | 32位带符号整数         | int            |
| Uint32  | 4    | 32位不带符号的整数       | unsigned int   |
| Float32 | 4    | 32位浮点数           | float          |
| Float64 | 8    | 64位浮点数           | double         |

### DataView

是一种底层接口，它提供有可以操作缓冲区中任意数据的读写接口，适合与操作不同类型数据的场景（不像 TypedArray 局限于某一种类型的数据）

### 应用

- FileReader API(readAsArrayBuffer方法)
- XMlHttpRequest的send方法(支持类型数组作为参数)
- Canvas

说明：类型数组并不是支持所有的原生数组的API(比如push和pop就不可用，因为ArrayBuffer给定了字节数，TypedArray视图自然无法调用)

## Web Worker

[http://www.alloyteam.com/2015/11/deep-in-web-worker/]

[https://www.html5rocks.com/zh/tutorials/workers/basics/]

Web Workers是一种机制，通过它可以使一个脚本操作在与Web应用程序的主执行线程分离的后台线程中运行。这样做的优点是可以在单独的线程中执行繁琐的处理，让主（通常是UI）线程运行而不被阻塞/减慢

Web Worker 规范中定义了两类工作线程，分别是专用线程Dedicated Worker和共享线程 Shared Worker，其中，Dedicated Worker只能为一个页面所使用，而Shared Worker则可以被多个页面所共享

## 垃圾回收

语言内嵌的 “垃圾回收器” 跟踪内存的分配和使用，以便分配的内存不再使用时自动释放它，但也只能由有限制地解决一般问题

### 引用计数

如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收

对象对原型的引用是隐式的、对属性的引用是显式的，所以遇到两个对象之间循环引用的情况就无法回收，这样就容易发生内存泄漏

### 标记清除

根据对象是否可以获得来决定是否不再需要（现代浏览器采用）

垃圾回收器在运行的时候会给存储在内存中的所有变量都加上标记。然后，它会去掉环境中的变量以及被环境中的变量引用的变量的标记（闭包）。而在此之后再被加上标记的变量将被视为准备删除的变量，原因是环境中的变量已经无法访问到这些变量了。最后，垃圾回收器完成内存清除工作，销毁那些带标记的值并回收它们所占用的内存空间

垃圾回收器定期从根（全局对象）开始，找所有从根开始引用的对象，然后找这些对象引用的对象
  
### 内存泄漏

- 无意的全局变量（意外的 this 指向 window）
- 未 clearInterval 或者 clearTimeout
- IE6-8 在 DOM & BOM 等 COM 对象循环绑定时会发生泄漏，需要切断循环引用，将对对象的应用设置为 null
- IE7-8 会发生 XMLHttpRequest 泄漏，需要手动设置其实例为 null
- 定时器（严格上说不能算是泄露，是被闭包持有了，是正常的表现），对于闭包中无用的变量可以使用 delete 操作符进行释放
- removeChild 删除节点只是在 DOM 树中删除