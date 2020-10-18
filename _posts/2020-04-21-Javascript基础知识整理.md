---
layout:     post
title:      Javascript基础知识整理
subtitle:   文档持续更新中...
date:       2020-04-21
author:     Faymi
header-img: img/post-bg-js-version.jpg
catalog: true
tags:
    - Javascript
    - 面试
---

# 基础知识集锦



## JS基础数据类型、复杂类型分别是什么以及其区别？null是对象吗？

基础数据类型：boolean、string、number、null、undefined、symbol。

复杂数据类型：object

区别：

1. 内存的分配不同：
   - 基础数据类型存储在栈内存中，存储的是值。
   - 复杂数据类型存储在堆内存中，栈中存储的变量，是指向堆中的引用地址。
2. 访问机制不同：
   - 基本数据类型是按值访问。
   - 复杂数据类型按引用访问，JS不允许直接访问保存在堆内存中的对象，在访问一个对象时，首先得到的是这个对象在栈内存中的地址，然后再按照这个地址去获得这个对象中的值。
3. 复制变量时不同：
   - 基本数据类型：a=b;是将b中保存的原始值的副本赋值给新变量a，a和b完全独立，互不影响。
   - 复杂数据类型：a=b;将b保存的对象内存的引用地址赋值给了新变量a;a和b指向了同一个堆内存地址，其中一个值发生了改变，另一个也会改变。
4. 参数传递的不同(实参/形参)：函数传参都是按值传递(栈中的存储的内容)：基本数据类型，拷贝的是值；复杂数据类型，拷贝的是引用地址。

null不是对象：

虽然typeof null 的结果为object，但是这个只是 JavaScript 语言本身的一个 bug。js对象底层是以二进制表示，若前三位为零的二进制，则判定为对象。而null在底层中的表示为都是零，即其前三位也是零，所以会被判定为对象。



## 如何让 (a == 1 && a == 2 && a == 3) 的值为true？

利用隐式转换规则。

`==` 操作符在左右数据类型不一致时，会先进行隐式转换。

`a == 1 && a == 2 && a == 3` 的值意味着其不可能是基本数据类型。因为如果 a 是 null 或者是 undefined bool类型，都不可能返回true。

因此可以推测 a 是复杂数据类型，JS 中复杂数据类型只有 `object。Object 转换为原始类型会调用什么方法？

`Symbol.toPrimitive` 是一个内置的 Symbol 值，它是作为对象的函数值属性存在的，当一个对象转换为对应的原始值时，会调用此函数。

```javascript
let a = {
    [Symbol.toPrimitive]: (function(hint) {
            let i = 1;
            //闭包的特性之一：i 不会被回收
            return function() {
                return i++;
            }
    })()
}
console.log(a == 1 && a == 2 && a == 3); //true
// or
let a = {
    valueOf: (function() {
        let i = 1;
        //闭包的特性之一：i 不会被回收
        return function() {
            return i++;
        }
    })()
}
console.log(a == 1 && a == 2 && a == 3); //true
// or
let a = {
    reg: /\d/g,
    valueOf () {
        return this.reg.exec(123)[0]
    }
}
console.log(a == 1 && a == 2 && a == 3); //true
// or 数据劫持
let i = 1;
Object.defineProperty(window, 'a', {
    get: function() {
        return i++;
    }
});
console.log(a == 1 && a == 2 && a == 3); //true
// or Proxy
let a = new Proxy({}, {
    i: 1,
    get: function () {
        return () => this.i++;
    }
});
console.log(a == 1 && a == 2 && a == 3); // true
// or 
let a = [1, 2, 3];
a.join = a.shift;
console.log(a == 1 && a == 2 && a == 3); //true
```



## 防抖(debounce)函数的作用是什么？有哪些应用场景，请实现一个防抖函数

防抖的作用是控制函数在一定时间内的执行次数。防抖意味着N秒内函数只会被执行一次，如果N秒内再次被触发，则重新计算延迟时间。

场景：

1. 搜索框输入查询，如果用户一直在输入中，没有必要不停地调用去请求服务端接口，等用户停止输入的时候，再调用，设置一个合适的时间间隔，有效减轻服务端压力。
2. 表单验证
3. 按钮提交事件。
4. 浏览器窗口缩放，resize事件等。

实现：

```javascript
function debounce(func, wait, immediate = true) {
    let timer;
    // 延迟执行函数
    const later = (context, args) => setTimeout(() => {
        timer = null;// 倒计时结束
        if (!immediate) {
            func.apply(context, args);
            //执行回调
            context = args = null;
        }
    }, wait);
    let debounced = function (...params) {
        let context = this;
        let args = params;
        if (!timer) {
            timer = later(context, args);
            if (immediate) {
                //立即执行
                func.apply(context, args);
            }
        } else {
            clearTimeout(timer);
            //函数在每个等待时延的结束被调用
            timer = later(context, args);
        }
    }
    debounced.cancel = function () {
        clearTimeout(timer);
        timer = null;
    };
    return debounced;
};
```

 [*实例参考*](https://jinlong.github.io/2016/04/24/Debouncing-and-Throttling-Explained-Through-Examples/)



## 说一说你对JS执行上下文栈和作用域链的理解？

#### [JS执行上下文](https://tc39.github.io/ecma262/?nsukey=rQHqMrFpKq6JJN%2F%2FOeubPCslaSTSRyuc%2FXCznnIDze1SGzwva5SZtzixJ13p2gAlxua95Xa7fraZXwj5tyLRDK33%2BpNhyfKR%2FxyzhWNyB%2FqaIlsDGyQBckNoHQGPveOB24M%2BcK%2FgF8Tg1ehUGLWiCvumxdgcQwZOWj2BGfD3n%2FY%3D#sec-execution-contexts)

执行上下文就是当前JS代码被解析和执行时所在环境的抽象概念， JavaScript 中运行任何的代码都是在执行上下文中运行。

> 执行上下文类型分为：

- 全局执行上下文
- 函数执行上下文
- eval函数执行上下文(不被推荐)

执行上下文创建过程中，需要做以下几件事:

1. 创建[变量对象](https://github.com/mqyqingfeng/Blog/issues/5)：首先初始化函数的参数arguments，提升函数声明和变量声明。
2. 创建作用域链（Scope Chain）：在执行期上下文的创建阶段，作用域链是在变量对象之后创建的。
3. 确定this的值，即 ResolveThisBinding。

#### 作用域

**作用域**负责收集和维护由所有声明的标识符（变量）组成的一系列查询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限。

作用域有两种工作模型：词法作用域和动态作用域，JS采用的是**词法作用域**工作模型，词法作用域意味着作用域是由书写代码时变量和函数声明的位置决定的。

> 作用域分为：

- 全局作用域
- 函数作用域
- 块级作用域

#### JS执行上下文栈(后面简称执行栈)

执行栈，也叫做调用栈，具有 **LIFO** (后进先出) 结构，用于存储在代码执行期间创建的所有执行上下文。

> 规则如下：

- 首次运行JavaScript代码的时候,会创建一个全局执行的上下文并Push到当前的执行栈中，每当发生函数调用，引擎都会为该函数创建一个新的函数执行上下文并Push当前执行栈的栈顶。
- 当栈顶的函数运行完成后，其对应的函数执行上下文将会从执行栈中Pop出，上下文的控制权将移动到当前执行栈的下一个执行上下文。

代码说明：

```javascript
function fun3() {
    console.log('fun3')
}

function fun2() {
    fun3();
}

function fun1() {
    fun2();
}

fun1();
```

伪代码:

```javascript
//全局执行上下文首先入栈
ECStack.push(globalContext);

//执行fun1();
ECStack.push(<fun1> functionContext);

//fun1中又调用了fun2;
ECStack.push(<fun2> functionContext);

//fun2中又调用了fun3;
ECStack.push(<fun3> functionContext);

//fun3执行完毕
ECStack.pop();

//fun2执行完毕
ECStack.pop();

//fun1执行完毕
ECStack.pop();

//javascript继续顺序执行下面的代码，但ECStack底部始终有一个 全局上下文（globalContext）;
```

#### 作用域链

作用域链就是从当前作用域开始一层一层向上寻找某个变量，直到找到全局作用域还是没找到，就宣布放弃。这种一层一层的关系，就是作用域链。



## 什么是BFC？BFC的布局规则是什么？如何创建BFC？

BFC 是 Block Formatting Context 的缩写，即块格式化上下文。

> 浮动、绝对定位的元素、非块级盒子的块容器（如inline-blocks、table-cells 和 table-captions），以及`overflow`的值不为`visible`（该值已传播到视区时除外）为其内容建立新的块格式上下文。

#### [BFC布局规则](https://www.w3.org/TR/2011/REC-CSS2-20110607/visuren.html#block-formatting)

- BFC内，盒子依次垂直排列。
- BFC内，两个盒子的垂直距离由 `margin` 属性决定。属于同一个BFC的两个相邻Box的margin会发生重叠【符合合并原则的margin合并后是使用大的margin】
- BFC内，每个盒子的左外边缘接触内部盒子的左边缘（对于从右到左的格式，右边缘接触）。即使在存在浮动的情况下也是如此。除非创建新的BFC。
- BFC的区域不会与float box重叠。
- BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。
- 计算BFC的高度时，浮动元素也参与计算。

#### 如何创建BFC

- 根元素(`<html>`)
- 浮动元素（元素的 [`float`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/float) 不是 `none`）
- [`overflow`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/overflow) 值不为 `visible` 的块元素
- 绝对定位元素（元素的 [`position`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position) 为 `absolute` 或 `fixed`）
- [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 值为 `flow-root` 的元素，无副作用创建BFC。兼容处理：`.wrapper{display: flow-root;} @supports not (display:flow-root) { .wrapper::after { content: ''; display: table; clear:both; } }`
- 行内块元素（元素的 [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为 `inline-block`）
- 表格单元格（元素的 [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display)为 `table-cell`，HTML表格单元格默认为该值）
- 表格标题（元素的 [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为 `table-caption`，HTML表格标题默认为该值）
- 匿名表格单元格元素（元素的 [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display)为 `table`、 `table-row`、 `table-row-group`、 `table-header-group`、 `table-footer-group`（分别是HTML table、row、tbody、thead、tfoot的默认属性）或 `inline-table`）
- [`contain`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/contain) 值为 `layout`、`content`或 paint 的元素
- 弹性元素（[`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display)为 `flex` 或 `inline-flex`元素的直接子元素）
- 网格元素（[`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display)为 `grid` 或 `inline-grid` 元素的直接子元素）
- 多列容器（元素的 [`column-count`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/column-count) 或 [`column-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/column-width) 不为 `auto`包括 ` column-count` 为 `1`）
- `column-span` 为 `all` 的元素始终会创建一个新的BFC，即使该元素没有包裹在一个多列容器中（[标准变更](https://github.com/w3c/csswg-drafts/commit/a8634b96900279916bd6c505fda88dda71d8ec51)，[Chrome bug](https://bugs.chromium.org/p/chromium/issues/detail?id=709362)）。

#### BFC的应用

1. 防止 margin 重叠

   根据BFC规则，同一个BFC内的两个两个相邻Box的 `margin` 会发生重叠，因此我们可以在div外面再嵌套一层容器，并且触发该容器生成一个 BFC，即产生两个 BFC，自然也就不会再发生 `margin` 重叠

2. 清除内部浮动（浮动元素的父级设置BFC规则）

3. 自适应多栏布局

   根据规则，BFC的区域不会与float box重叠。因此，可以触发生成一个新的BFC。

   ```javascript
   <style>
       body{
           width: 500px;
       }
       .a{
           height: 150px;
           width: 100px;
           background: pink;
           float: left;
       }
       .b{
           height: 200px;
           background: blue;
       }
   </style>
   <body>
       <div class="a"></div>
       <div class="b"></div>
   </body>   
   ```

   

   ![img](https://user-gold-cdn.xitu.io/2019/6/2/16b1775cc7bbe073)

   

   > 根据规则，BFC的区域不会与float box重叠。因此，可以触发生成一个新的BFC，如下：

   ```javascript
   <style>
   .b{
       height: 200px;
       overflow: hidden; /*触发生成BFC*/
       background: blue;
   }
   </style>
   ```

   

   ![img](https://user-gold-cdn.xitu.io/2019/6/2/16b1775cc7f30da0)

   

## let、const、var 的区别有哪些？

| 声明方式 | 变量提升 | 暂时性死区 | 重复声明 | 块作用域有效 | 初始值 | 重新赋值 |
| -------- | -------- | ---------- | -------- | ------------ | ------ | -------- |
| var      | 会       | 不存在     | 允许     | 不是         | 非必须 | 允许     |
| let      | 不会     | 存在       | 不允许   | 是           | 非必须 | 允许     |
| const    | 不会     | 存在       | 不允许   | 是           |        |          |

1. let/const 定义的变量不会出现变量提升，而 var 定义的变量会提升。

2. 相同作用域中，let 和 const 不允许重复声明，var 允许重复声明。

3. const 声明变量时必须设置初始值.

4. const 声明一个只读的常量，这个常量不可改变。

5. let/const 声明的变量仅在块级作用域中有效。而 var 声明的变量在块级作用域外仍能访问到。

   ```javascript
   {
       let a = 10;
       const b = 20;
       var c = 30;
   }
   console.log(a); //ReferenceError
   console.log(b); //ReferenceError
   console.log(c); //30
   ```

6. 顶层作用域中 var 声明的变量挂在window上(浏览器环境)。

7. let/const有暂时性死区的问题，即let/const 声明的变量，在定义之前都是不可用的。如果使用会抛出错误。

   ```javascript
   var a = 10;
   if (true) {
     a = 20; // ReferenceError
     let a;
   }
   ```

   

## 深拷贝和浅拷贝的区别是什么？如何实现一个深拷贝？

#### 深拷贝

> 深拷贝复制变量值，对于非基本类型的变量，则递归至基本类型变量后，再复制。 深拷贝后的对象与原来的对象是完全隔离的，互不影响，对一个对象的修改并不会影响另一个对象。

#### 浅拷贝

> 浅拷贝是会将对象的每个属性进行依次复制，但是当对象的属性值是引用类型时，实质复制的是其引用，当引用指向的值改变时也会跟着变化。

```javascript
function deepClone(obj, hash = new WeakMap()) { //递归拷贝
    if(obj instanceof RegExp) return new RegExp(obj);
    if(obj instanceof Date) return new Date(obj);
    if(obj === null || typeof obj !== 'object') {
        //如果不是复杂数据类型，直接返回
        return obj;
    }
    // 循环引用返回
    if (hash.has(obj)) {
        return hash.get(obj);
    }
    /**
     * 如果obj是数组，那么 obj.constructor 是 [Function: Array]
     * 如果obj是对象，那么 obj.constructor 是 [Function: Object]
     */
    let t = new obj.constructor();
    hash.set(obj, t);
    for(let key in obj) {
        //如果 obj[key] 是复杂数据类型，递归
        if(obj.hasOwnProperty(key)){//是否是自身的属性
            if(obj[key] && typeof obj[key] === 'object') {
                t[key] = deepClone(obj[key], hash);
            } else{
                t[key] = obj[key];
            }
            
        }
    }
    return t;
}
```



## package.json 中的 peerDependencies作用？

`peerDependencies`的目的是提示宿主环境去安装满足插件peerDependencies所指定依赖的包，然后在插件import或者require所依赖的包的时候，永远都是引用宿主环境统一安装的npm包，最终解决插件与所依赖包不一致的问题。

指定当前组件的依赖以其版本。如果组件使用者在项目中安装了其他版本的同一依赖，会提示报错。

[package.json 中的 peerDependencies](https://javascript.ruanyifeng.com/nodejs/packagejson.html#toc3)

[探讨npm依赖管理之peerDependencies](https://www.cnblogs.com/wonyun/p/9692476.html)



## React的虚拟DOM优势？

它减少了同一时间内的页面多处内容修改所触发的浏览器reflow和repaint的次数，可能把多个不同的DOM操作集中减少到了几次甚至一次，优化了触发浏览器reflow和repaint的次数



## compose实现

```javascript
const add = num => num  + 10
const multiply = num => num * 2
const foo = compose(multiply, add)
foo(5) => 30
```

```javascript
// 摘自 https://github.com/reactjs/redux/blob/master/src/compose.js
export default function compose(...funcs) {  
	if (funcs.length === 0) {
    return arg => arg  
  }
	if (funcs.length === 1) {
    return funcs[0]  
  }  
	return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

#### koa compose的实现

前置条件：中间件注册

```js
use(fn) {
  this.middleware.push(fn);
  return this;
}
```

实现：

```js
function compose (middleware) {
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  /**
   * @param {Object} context
   * @return {Promise}
   * @api public
   */

  return function (context, next) {
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        // 利用Promise包装中间件
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1))); 
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```



## webpack中loaders作用？plugins和loaders区别？是否写过webpack插件

![image-20200507114606676](https://tva1.sinaimg.cn/large/007S8ZIlgy1gejqor7p1sj317x0u0u0x.jpg)



## 箭头函数与普通函数区别？能不能作为构造函数

**箭头函数表达式**的语法比[函数表达式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/function)更简洁，并且没有自己的`this`，`arguments`，`super`或`new.target`，没有`prototype`属性。箭头函数表达式更适用于那些本来需要匿名函数的地方，并且它**不能用作构造函数**。

普通函数：

- 如果是该函数是一个构造函数，this指针指向一个新的对象
- 在严格模式下的函数调用下，this指向undefined
- 如果是该函数是一个对象的方法，则它的this指针指向这个对象

箭头函数：

不会创建自己的`this,它只会从自己的作用域链的上一层继承this`。通过 call 或 apply 调用时，由于箭头函数没有自己的this指针，通过 `call()` *或* `apply()` 方法调用一个函数时，只能传递参数，他们的第一个参数会被忽略。

 `yield` 关键字通常不能在箭头函数中使用（除非是嵌套在允许使用的函数内）。因此，箭头函数不能用作函数生成器。

```javascript
var adder = {
  base : 1,
    
  add : function(a) {
    var f = v => v + this.base;
    return f(a);
  },

  addThruCall: function(a) {
    var f = v => v + this.base;
    var b = {
      base : 2
    };
            
    return f.call(b, a);
  }
};

console.log(adder.add(1));         // 输出 2
console.log(adder.addThruCall(1)); // 仍然输出 2
```

 

## EventLoop 相关，有哪些宏任务和微任务？特点？对 requestAnimationFrame 的理解



## this的原理

例子：

```javascript
var obj = {
  foo: function () { console.log(this.bar) },
  bar: 1
};

var foo = obj.foo;
var bar = 2;

obj.foo() // 1
foo() // 2
```

**`this`指的是函数运行时所在的环境。**对于`obj.foo()`来说，`foo`运行在`obj`环境，所以`this`指向`obj`；对于`foo()`来说，`foo`运行在全局环境，所以`this`指向全局环境。所以，两者的运行结果不一样。

变量`obj`是一个地址（reference）。后面如果要读取`obj.foo`，引擎先从`obj`拿到内存地址，然后再从该地址读出原始的对象，返回它的`foo`属性。

原始的对象以字典结构保存，每一个属性名都对应一个属性描述对象。如下：

![img](https://www.wangbase.com/blogimg/asset/201806/bg2018061802.png)

```javascript
{
  foo: {
    [[value]]: 5
    [[writable]]: true
    [[enumerable]]: true
    [[configurable]]: true
  }
}
```

注意，`foo`属性的值保存在属性描述对象的`value`属性里面。

**函数**时，引擎会将函数单独保存在内存中，然后再将函数的地址赋值给`foo`属性的`value`属性。

![img](https://www.wangbase.com/blogimg/asset/201806/bg2018061803.png)

```javascript
{
  foo: {
    [[value]]: 函数的地址
    ...
  }
}
```

由于函数是一个单独的值，所以它可以在不同的环境（上下文）执行。JavaScript 允许在函数体内部，引用当前环境的其他变量。由于函数可以在不同的运行环境执行，所以需要有一种机制，能够在函数体内部获得当前的运行环境（context）。所以，`this`就出现了，它的设计目的就是在函数体内部，指代函数当前的运行环境。

[引用](http://www.ruanyifeng.com/blog/2018/06/javascript-this.html)



## 什么是内存泄漏和内存溢出？

- 内存溢出 out of memory：是指程序在为某段执行指令（程序）分配内存的时候，发现内存不足，抛出错误；比如申请了一个integer,但给它存了long才能存下的数，那就是内存溢出。
- 内存泄露 memory leak：不再用到的内存，没有及时释放。

垃圾回收机制：引用计数法，即语言引擎有一张"引用表"，保存了内存里面所有的资源（通常是各种值）的引用次数。如果一个值的引用次数是`0`，就表示这个值不再用到了，因此可以将这块内存释放。

```javascript
// arr 引用次数为1
let arr = [1, 2, 3, 4];
console.log('hello world');
// 置为null解除引用
arr = null;
```

内存泄漏解决：

- 手动清除引用；

- 用新的数据结构：[WeakSet](http://es6.ruanyifeng.com/#docs/set-map#WeakSet) (成员只能是对象，而不能是其他类型的值) 和 [WeakMap](http://es6.ruanyifeng.com/#docs/set-map#WeakMap) (只接受对象作为键名)。（它们对于值的引用都是不计入垃圾回收机制的）



## Reflect

静态方法：

- Reflect.apply(target, thisArg, args)

- Reflect.construct(target, args)

- Reflect.get(target, name, receiver)

  `Reflect.get`方法查找并返回`target`对象的`name`属性，如果没有该属性，则返回`undefined`。如果`name`属性部署了读取函数（getter），则**读取函数的`this`绑定`receiver`。**

- Reflect.set(target, name, value, receiver)

- Reflect.defineProperty(target, name, desc)

- Reflect.deleteProperty(target, name)

- Reflect.has(target, name)

- Reflect.ownKeys(target)

- Reflect.isExtensible(target)

- Reflect.preventExtensions(target)

- Reflect.getOwnPropertyDescriptor(target, name)

- Reflect.getPrototypeOf(target)

- Reflect.setPrototypeOf(target, prototype)

[引用](https://es6.ruanyifeng.com/#docs/reflect)



## Worker知识

作用就是为 JavaScript 创造多线程环境，允许主线程创建 Worker 线程，将一些任务分配给后者运行。在主线程运行的同时，Worker 线程在后台运行，两者互不干扰。等到 Worker 线程完成计算任务，再把结果返回给主线程。这样的好处是，一些计算密集型或高延迟的任务，被 Worker 线程负担了，主线程（通常负责 UI 交互）就会很流畅，不会被阻塞或拖慢。

1. **同源限制**

   分配给 Worker 线程运行的脚本文件，必须与主线程的脚本文件同源。

2. **DOM 限制**

   Worker 线程所在的全局对象，与主线程不一样，无法读取主线程所在网页的 DOM 对象，也无法使用`document`、`window`、`parent`这些对象。但是，Worker 线程可以`navigator`对象和`location`对象。

3. **通信联系**

   Worker 线程和主线程不在同一个上下文环境，它们不能直接通信，必须通过消息完成。

4. **脚本限制**

   Worker 线程不能执行`alert()`方法和`confirm()`方法，但可以使用 XMLHttpRequest 对象发出 AJAX 请求。

5. **文件限制**

   Worker 线程无法读取本地文件，即不能打开本机的文件系统（`file://`），它所加载的脚本，必须来自网络。

**构造函数**

- [`Worker()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker/Worker)

  创建一个专用Web worker，它只执行URL指定的脚本。Worker不指定URL时，而由使用[Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)创建。

**属性**

*继承*父对象*[`EventTarget`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget) 的属性，以及实现对象 [`AbstractWorker`](https://developer.mozilla.org/zh-CN/docs/Web/API/AbstractWorker)的属性。*

**事件句柄**

- [`AbstractWorker.onerror`](https://developer.mozilla.org/zh-CN/docs/Web/API/AbstractWorker/onerror)

  当[`ErrorEvent`](https://developer.mozilla.org/zh-CN/docs/Web/API/ErrorEvent) 类型的事件冒泡到 worker 时，事件监听函数 [`EventListener`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventListener) 被调用. 它继承于 [`AbstractWorker`](https://developer.mozilla.org/zh-CN/docs/Web/API/AbstractWorker).

- [`Worker.onmessage`](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker/onmessage)

  当[`MessageEvent`](https://developer.mozilla.org/zh-CN/docs/Web/API/MessageEvent)类型的事件冒泡到 worker 时，事件监听函数 [`EventListener`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventListener) 被调用. 例如，一个消息通过 [`DedicatedWorkerGlobalScope.postMessage`](https://developer.mozilla.org/zh-CN/docs/Web/API/DedicatedWorkerGlobalScope/postMessage)，从执行者发送到父页面对象，消息保存在事件对象的 [`data`](https://developer.mozilla.org/zh-CN/docs/Web/API/MessageEvent/data) 属性中.

- [`Worker.onmessageerror`](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker/onmessageerror)

  当`messageerror` 类型的事件发生时，对应的[`EventHandler`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventHandler) 代码被调用。

**方法**

*继承*父对象*[`EventTarget`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget) 的方法，以及实现对象 [`AbstractWorker`](https://developer.mozilla.org/zh-CN/docs/Web/API/AbstractWorker)的方法。*

- [`Worker.postMessage()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker/postMessage)

  发送一条消息到最近的外层对象，消息可由任何 JavaScript 对象组成。

- [`Worker.terminate()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker/terminate)

  立即终止 worker。该方法不会给 worker 留下任何完成操作的机会；就是简单的立即停止。Service Woker 不支持这个方法。

通过构造函数 [`Worker()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker/Worker) 创建了一个 [`Worker`](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker) 对象。

```js
var myWorker = new Worker('worker.js'); 
var first = document.querySelector('#number1');
var second = document.querySelector('#number2');

first.onchange = function() {
  myWorker.postMessage([first.value,second.value]);
  console.log('Message posted to worker');
}
```



## Webpack 工作原理

Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

1. 初始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
2. 开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译；
3. 确定入口：根据配置中的 entry 找出所有的入口文件；
4. 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
5. 完成模块编译：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；
6. 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
7. 输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。

在以上过程中，Webpack 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。



## CDN加速

#### 什么是CDN

CDN 又叫内容分发网络，通过把资源部署到世界各地，用户在访问时按照就近原则从离用户最近的服务器获取资源，从而加速资源的获取速度。 CDN 其实是通过优化物理链路层传输过程中的网速有限、丢包等问题来提升网速的。

#### 接入CDN

当把资源都放到同一个 CDN 服务上去时，网页是能正常使用的。 但需要注意的是由于 CDN 服务一般都会给资源开启很长时间的缓存，例如用户从 CDN 上获取到了 `index.html` 这个文件后， 即使之后的发布操作把 `index.html` 文件给重新覆盖了，但是用户在很长一段时间内还是运行的之前的版本，这会新的导致发布不能立即生效。

要避免以上问题，业界比较成熟的做法是这样的：

- 针对 HTML 文件：不开启缓存，把 HTML 放到自己的服务器上，而不是 CDN 服务上，同时关闭自己服务器上的缓存。自己的服务器只提供 HTML 文件和数据接口。
- 针对静态的 JavaScript、CSS、图片等文件：开启 CDN 和缓存，上传到 CDN 服务上去，同时给每个文件名带上由文件内容算出的 Hash 值， 例如上面的 `app_a6976b6d.css` 文件。 带上 Hash 值的原因是文件名会随着文件内容而变化，只要文件发生变化其对应的 URL 就会变化，它就会被重新下载，无论缓存时间有多长。

除此之外，浏览器有一个规则是同一时刻针对同一个域名的资源并行请求是有限制的话（具体数字大概4个左右，不同浏览器可能不同）， 你会发现上面的做法有个很大的问题。由于所有静态资源都放到了同一个 CDN 服务的域名下，也就是上面的 `cdn.com`。 如果网页的资源很多，例如有很多图片，就会导致资源的加载被阻塞，因为同时只能加载几个，必须等其它资源加载完才能继续加载。 要解决这个问题，可以把这些静态资源分散到不同的 CDN 服务上去， 例如把 JavaScript 文件放到 `js.cdn.com` 域名下、把 CSS 文件放到 `css.cdn.com` 域名下、图片文件放到 `img.cdn.com` 域名下。

>  使用了多个域名后又会带来一个新问题：增加域名解析时间。是否采用多域名分散资源需要根据自己的需求去衡量得失。 当然你可以通过在 HTML HEAD 标签中 加入 `<link rel="dns-prefetch" href="//js.cdn.com">` 去预解析域名，以降低域名解析带来的延迟。

#### Webpack接入CDN

Webpack构建需要实现以下几点：

- 静态资源的导入 URL 需要变成指向 CDN 服务的**绝对路径**的 URL 而不是相对于 HTML 文件的 URL。
- 静态资源的文件名称需要带上有**文件内容**算出来的 Hash 值，以防止被缓存。
- 不同类型的资源放到**不同域名的 CDN** 服务上去，以防止资源的并行加载被阻塞。

```js
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const {WebPlugin} = require('web-webpack-plugin');

module.exports = {
  // 省略 entry 配置...
  output: {
    // 给输出的 JavaScript 文件名称加上 Hash 值
    filename: '[name]_[chunkhash:8].js',
    path: path.resolve(__dirname, './dist'),
    // 指定存放 JavaScript 文件的 CDN 目录 URL
    publicPath: '//js.cdn.com/id/',
  },
  module: {
    rules: [
      {
        // 增加对 CSS 文件的支持
        test: /\.css$/,
        // 提取出 Chunk 中的 CSS 代码到单独的文件中
        use: ExtractTextPlugin.extract({
          // 压缩 CSS 代码
          use: ['css-loader?minimize'],
          // 指定存放 CSS 中导入的资源（例如图片）的 CDN 目录 URL
          publicPath: '//img.cdn.com/id/'
        }),
      },
      {
        // 增加对 PNG 文件的支持
        test: /\.png$/,
        // 给输出的 PNG 文件名称加上 Hash 值
        use: ['file-loader?name=[name]_[hash:8].[ext]'],
      },
      // 省略其它 Loader 配置...
    ]
  },
  plugins: [
    // 使用 WebPlugin 自动生成 HTML
    new WebPlugin({
      // HTML 模版文件所在的文件路径
      template: './template.html',
      // 输出的 HTML 的文件名称
      filename: 'index.html',
      // 指定存放 CSS 文件的 CDN 目录 URL
      stylePublicPath: '//css.cdn.com/id/',
    }),
    new ExtractTextPlugin({
      // 给输出的 CSS 文件名称加上 Hash 值
      filename: `[name]_[contenthash:8].css`,
    }),
    // 省略代码压缩插件配置...
  ],
};
```

以上代码中最核心的部分是通过 `publicPath` 参数设置存放静态资源的 CDN 目录 URL， 为了让不同类型的资源输出到不同的 CDN，需要分别在：

- `output.publicPath` 中设置 JavaScript 的地址。
- `css-loader.publicPath` 中设置被 CSS 导入的资源的的地址。
- `WebPlugin.stylePublicPath` 中设置 CSS 文件的地址。

[引用](https://webpack.wuhaolin.cn/)



## 递增 (++)

递增运算符为其操作数增加1，返回一个数值。

- 如果使用后置（postfix），即运算符位于操作数的后面（如 x++），那么将会在递增前返回数值。
- 如果使用前置（prefix），即运算符位于操作数的前面（如 ++x），那么将会在递增后返回数值。

```js
// 后置 
var x = 3;
y = x++; 
// y = 3, x = 4

// 前置
var a = 2;
b = ++a; 
// a = 3, b = 3
```



## 算法的时间与空间复杂度

- 时间维度：是指执行当前算法所消耗的时间，我们通常用「时间复杂度」来描述。
- 空间维度：是指执行当前算法需要占用多少内存空间，我们通常用「空间复杂度」来描述。

时间复杂度的公式是： T(n) = O( f(n) )，其中f(n) 表示每行代码执行次数之和，而 O 表示正比例关系，这个公式的全称是：**算法的渐进时间复杂度**。



## [window.requestAnimationFrame](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame)

**`window.requestAnimationFrame()`** 告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行。

调函数会在浏览器下一次重绘之前执行

>  **注意：若你想在浏览器下次重绘之前继续更新下一帧动画，那么回调函数自身必须再次调用`window.requestAnimationFrame()`**

当你准备更新动画时你应该调用此方法。这将使浏览器在下一次重绘之前调用你传入给该方法的动画函数(即你的回调函数)。回调函数执行次数通常是每秒60次，但在大多数遵循W3C建议的浏览器中，回调函数执行次数通常与浏览器屏幕**刷新次数**相匹配。

#### 语法

```
window.requestAnimationFrame(callback);
```

#### 参数

- `callback`

  下一次重绘之前更新动画帧所调用的函数(即上面所说的回调函数)。该回调函数会被传入[`DOMHighResTimeStamp`](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMHighResTimeStamp)参数，该参数与[`performance.now()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Performance/now)的返回值相同，它表示`requestAnimationFrame()` 开始去执行回调函数的时刻。

#### 返回值

一个 `long` 整数，请求 ID ，是回调列表中唯一的标识。是个非零值，没别的意义。你可以传这个值给 [`window.cancelAnimationFrame()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/cancelAnimationFrame) 以取消回调函数。

```js
var start = null;
var element = document.getElementById('SomeElementYouWantToAnimate');
element.style.position = 'absolute';

function step(timestamp) {
  if (!start) start = timestamp;
  var progress = timestamp - start;
  element.style.left = Math.min(progress / 10, 200) + 'px';
  if (progress < 2000) {
    window.requestAnimationFrame(step);
  }
}

window.requestAnimationFrame(step);
```



## HTTP状态码备忘单

#### 什么是 HTTP 状态码

Fernando Doglio在他的书中 - 使用NodeJS的**REST API开发**将状态代码定义为：

> 一个数字，总结了与之相关的响应。

当客户端向服务器发出请求时，服务器提供HTTP（超文本传输协议）响应状态代码，这使我们能够了解网站后端发生的情况，确定需要修复的错误。

#### HTTP 状态码

##### 响应信息

###### 1xx：临时回应，表示客户端请继续 

1xx 的状态会被浏览器 HTTP 库直接处理掉，不会让上层应用知晓

##### 成功

###### 200 - OK：成功传输 

请求成功

###### 201 - Created：创建 

资源已创建，服务器已确认。它对POST或PUT请求的响应很有用。此外，新资源可以作为响应正文的一部分返回。

###### 204 - No content：没有内容 

该操作请求成功，但没有返回任何内容。对于不需要响应主体的操作很有用，例如 DELETE 操作。

###### 205 - Reset Content：重置内容 

表示请求成功，但响应报文不含实体的主体部分，但是与 204 响应不同在于要求请求方重置内容。

##### 重定向

3xx: 当服务器通知客户端请求的目标有变化，希望客户端进一步处理，将使用这些。

为什么需要重定向？

1. 网站调整（如改变网页目录结构）。
2. 网页被迁移到一个新地址。
3. 网页扩展名改变。（如.php => .html）用户收藏夹或者搜索引擎数据库的就地址会返回404，导致流量损失；
4. 多域名跳转到主站点。

###### 301 - moved permanently：永久移动 

用于通知浏览器所请求的文件已被移动，并且它应该从服务器提供的位置（响应的Location首部）请求文件，并记住该新位置以供将来参考。这只能用于HTTP GET和HEAD请求。

此资源已移至另一个位置，并返回该位置。当URL随着时间的推移而变化时（尤其是由于版本，迁移或其他一些破坏性更改），此标头特别有用，保留旧标头并将重定向返回到新位置允许旧客户端更新其引用自己的时间。

###### 302 - found（找到）：临时重定向 

相似301; 但它是临时重定向。它将客户端从旧资源引导到新资源，但它不会告诉搜索引擎更新页面的索引（保存的还是旧地址的索引）。告诉客户端浏览另一个URL。

###### 304- Not Modified：客户端缓存没有更新 

产生的前提：客户端本地已经有缓存的版本，并且在 Request 中告诉了服务端，当服务端通过时间或 tag，发现没有更新的时候，就会返回一个不含 body 的 304 状态码

###### 307 - temporary redirect：暂时移动

临时重定向，和302含义类似，但是期望客户端保持请求方法不变向新的地址发出请求

##### 客户端错误

4xx: 定义客户端错误，这是服务器认为Web浏览器出错的地方。

###### 400 - bad request：不良请求 

发出的请求有问题（例如，可能缺少一些必需的参数）。对400响应的良好补充可能是开发人员可用于修复请求的错误消息

###### 401 - unauthorized：未经授权 

当拥有请求的用户无法访问所请求的资源时，对身份验证特别有用。服务器响应www-authenticate，客户端请求首部Authenticate。

###### 403 - forbidden：禁止 - 资源不可访问，但与401不同，身份验证不会影响响应。

通常在请求的文件有效但文件无法提供时发出，这通常是由于服务器端权限问题导致Web服务器不允许将文件提供给客户端。如IP被列入黑名单；同一IP地址发送请求过多，被服务器屏蔽；DNS解析错误等。

**401 与 403 的区别：**

- **401** 我去找个人，门卫说不认识我不让我进
- **403** 我去找个人，门卫说认识我，但是我不能进，因为我不配

###### 404 - Not Found：请求的资源不存在

这可能是最常见且经常出现的错误。当Web浏览器请求服务器上不存在的文件时，会发生此问题。

###### 405 - 方法不允许

不允许在资源上使用HTTP动词**（例如POST，GET，PUT等）** - 例如，在只读资源上执行PUT。

##### 服务端错误

5xx: 定义服务器端错误。尽管客户端提供了有效请求，但这些都是服务器部分发生的错误。

###### 500 - internal sever error：内部服务器错误

这是一个不幸的模糊通用错误代码。只要认为服务器遇到与任何更具体的错误代码不匹配的错误，就会发出它。

###### 501 - Not Implemented：未实现

服务器要么不识别请求方法，要么不支持请求。

###### 502 - Bad Gateway：错误网关

对用户访问请求的响应超时造成的。或当Web浏览器联系充当另一个服务器的代理的Web服务器并且从另一个服务器获得无效响应时，会发生这种情况。

###### 503 - service unavailable：服务不可用

这通常在服务器暂时性错误（暂时处于超负载或正在停机维护）的情况下遇到，此时服务器无法处理请求，可以一会再试



## [继承与原型链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)

每个实例对象（ object ）都有一个私有属性（称之为 __proto__ ）指向它的构造函数的原型对象（**prototype** ）。该原型对象也有一个自己的原型对象( __proto__ ) ，层层向上直到一个对象的原型对象为 `null`。根据定义，`null` 没有原型，并作为这个**原型链**中的最后一个环节。



## HTML rel属性的作用

#### rel是什么

rel属性通常用来描述链接之间的关系，也就是说目的地址(href) 跟源（站点）之间的关系。rel属性通常出现在a,link标签中。rel是relationship的英文缩写。

#### rel的属性值有哪些

`alternate` -- 文档的替代版本（比如打印页、翻译或镜像）

`appendix` -- 定义文档的附加信息

`bookmark` -- 书签

`chapter` -- 当前文档的章节

`contents`--文档的目录

`copyright` -- 当前文档的版权

`glossary` -- 词汇

`help` -- 链接帮助信息\帮助文档

`index` -- 当前文档的索引

`next` -- 记录文档的下一页.(浏览器可以提前加载此页)

`nofollow` -- 不被用于计算PageRank

`prev` -- 记录文档的上一页.(定义浏览器的后退键)

`section` -- 作为文档的一部分

`start` -- 通知搜索引擎,文档的开始

`stylesheet` -- 定义一个外部加载的样式表

`subsection` -- 作为文档的一小部分

`me`--告诉搜索引擎，这是自己的内容

`home`--告诉搜索引擎，这是返回首页

`license`--描述版权

`friend`--这是朋友的

`tag`--标签

`same`--相同的链接

#### 什么是nofollow?

> nofollow是一个HTML标签的属性值，随着搜索引擎优化(SEO)的兴起，它渐渐被大家所了解，这个标签的意思是告诉搜索引擎不要此网页上的链接或不要追踪此特定链接。如果A网页上有一个链接指向B网页，但A网页给这个链接加上了rel=”nofollow” 标注，则搜索引擎不把A网页计算入B网页的反向链接。搜索引擎看到这个标签就可能减少或完全取消链接的投票权重。



## 前端安全问题

#### iframe

**a、如何让自己的网站不被其他网站的 iframe 引用？**

```js
// 检测当前网站是否被第三方iframe引用
// 若相等证明没有被第三方引用，若不等证明被第三方引用。当发现被引用时强制跳转百度。
if(top.location != self.location){
    top.location.href = 'http://www.baidu.com'
}
```

**b、如何禁用，被使用的 iframe 对当前网站某些操作？**

> **sandbox**是html5的新属性，主要是提高iframe安全系数。iframe因安全问题而臭名昭著，这主要是因为iframe常被用于嵌入到第三方中，然后执行某些恶意操作。
> 现在有一场景：我的网站需要 iframe 引用某网站，但是不想被该网站操作DOM、不想加载某些js（广告、弹框等）、当前窗口被强行跳转链接等，我们可以设置 sandbox 属性。如使用多项用空格分隔。

- allow-same-origin：允许被视为同源，即可操作父级DOM或cookie等
- allow-top-navigation：允许当前iframe的引用网页通过url跳转链接或加载
- allow-forms：允许表单提交
- allow-scripts：允许执行脚本文件
- allow-popups：允许浏览器打开新窗口进

**c、设置`X-Frame-Options` HTTP响应头**

用来给浏览器 指示允许一个页面 可否在 [`frame`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/frame), [`iframe`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/iframe), [`embed`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/embed) 或者 [`object`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/object) 中展现的标记。站点可以通过确保网站没有被嵌入到别人的站点里面，从而避免 [clickjacking](https://zh.wikipedia.org/wiki/clickjacking) 攻击。

`deny`

表示该页面不允许在 frame 中展示，即便是在相同域名的页面中嵌套也不允许。

`sameorigin`

表示该页面可以在相同域名页面的 frame 中展示。

`allow-from *uri*`

表示该页面可以在指定来源的 frame 中展示。

配置nginx:

```nginx
add_header X-Frame-Options sameorigin always;
```

####  XSS

- **什么是 XSS**

Cross-Site Scripting（跨站脚本攻击）简称 XSS，是一种代码注入攻击。攻击者通过在目标网站上注入恶意脚本，使之在用户的浏览器上运行。利用这些恶意脚本，攻击者可获取用户的敏感信息如 Cookie、SessionID 等，进而危害数据安全。

XSS 的本质是：恶意代码未经过滤，与网站正常的代码混在一起；浏览器无法分辨哪些脚本是可信的，导致恶意脚本被执行。

- **XSS的分类**

  - **存储型XSS**

    存储型 XSS 的攻击步骤：

    1. 攻击者将恶意代码提交到目标网站的数据库中。
    2. 用户打开目标网站时，网站服务端将恶意代码从数据库取出，拼接在 HTML 中返回给浏览器。
    3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
    4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

    这种攻击常见于带有用户保存数据的网站功能，如论坛发帖、商品评论、用户私信等。

  - **反射型XSS**

    反射型 XSS 的攻击步骤：

    1. 攻击者构造出特殊的 URL，其中包含恶意代码。
    2. 用户打开带有恶意代码的 URL 时，网站服务端将恶意代码从 URL 中取出，拼接在 HTML 中返回给浏览器。
    3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
    4. 恶意代码窃;取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

    反射型 XSS 跟存储型 XSS 的区别是：存储型 XSS 的恶意代码存在数据库里，反射型 XSS 的恶意代码存在 URL 里。

    反射型 XSS 漏洞常见于通过 URL 传递参数的功能，如网站搜索、跳转等。

    由于需要用户主动打开恶意的 URL 才能生效，攻击者往往会结合多种手段诱导用户点击。

    POST 的内容也可以触发反射型 XSS，只不过其触发条件比较苛刻（需要构造表单提交页面，并引导用户点击），所以非常少见。

  - **DOM型XSS**

    DOM 型 XSS 的攻击步骤：

    1. 攻击者构造出特殊的 URL，其中包含恶意代码。
    2. 用户打开带有恶意代码的 URL。
    3. 用户浏览器接收到响应后解析执行，前端 JavaScript 取出 URL 中的恶意代码并执行。
    4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

    DOM 型 XSS 跟前两种 XSS 的区别：DOM 型 XSS 攻击中，取出和执行恶意代码由浏览器端完成，属于前端 JavaScript 自身的安全漏洞，而其他两种 XSS 都属于服务端的安全漏洞。

- **XXS攻击的预防**

  - **预防存储型和反射型 XSS 攻击**

    - 改成纯前端渲染，把代码和数据分隔开：
      1. 浏览器先加载一个静态 HTML，此 HTML 中不包含任何跟业务相关的数据。
      2. 然后浏览器执行 HTML 中的 JavaScript。
      3. JavaScript 通过 Ajax 加载业务数据，调用 DOM API 更新到页面上。
    - 对 HTML 做充分转义。

  - **预防DOM型XSS攻击**

    - 在使用 `.innerHTML`、`.outerHTML`、`document.write()` 时要特别小心，不要把不可信的数据作为 HTML 插到页面上，而应尽量使用 `.textContent`、`.setAttribute()` 等。
    - 如果用 Vue/React 技术栈，并且不使用 `v-html`/`dangerouslySetInnerHTML` 功能，就在前端 render 阶段避免 `innerHTML`、`outerHTML` 的 XSS 隐患。
    - DOM 中的内联事件监听器，如 `location`、`onclick`、`onerror`、`onload`、`onmouseover` 等，标签的 `href` 属性，JavaScript 的 `eval()`、`setTimeout()`、`setInterval()` 等，都能把字符串作为代码运行。如果不可信的数据拼接到字符串中传递给这些 API，很容易产生安全隐患，请务必避免。

  - **输入内容长度控制**

  - [CSP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy)：HTTP 响应头**`Content-Security-Policy`**允许站点管理者控制用户代理能够为指定的页面加载哪些资源。

    CSP 允许在一个资源中指定多个策略, 包括通过 `Content-Security-Policy` 头, 以及 [`Content-Security-Policy-Report-Only`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only) 头，和 [`meta`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/meta) 组件。

    示例: 禁用不安全的内联/动态执行, 只允许通过 https加载这些资源 (images, fonts, scripts, etc.)

    ```nginx
    // header
    Content-Security-Policy: default-src https:
    
    // meta tag
    <meta http-equiv="Content-Security-Policy" content="default-src https:">
    ```

  - **其他**

    - Cookie设置**Secure**（Cookie只应通过被HTTPS协议加密过的请求发送给服务端）、**HttpOnly**（通过JavaScript的 [`Document.cookie`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/cookie) API无法访问带有 `HttpOnly` 标记的Cookie，它们只应该发送给服务端）: 禁止 JavaScript 读取某些敏感 Cookie，攻击者完成 XSS 注入后也无法窃取此 Cookie。

      ```html
      Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly
      ```

    - 验证码：防止脚本冒充用户提交危险操作。

#### [CSRF](https://juejin.im/post/5df5bcea6fb9a016091def69#heading-74)

- **什么是CSRF攻击？**

CSRF(Cross-site request forgery), 即跨站请求伪造，通过伪装成受信任用户的请求来利用受信任的网站。指的是黑客诱导用户点击链接，打开黑客的网站，然后黑客利用用户**目前的登录状态**发起跨站请求。

- **与XSS的区别：**

  - 一般攻击发起点不在目标网站,而是被引导到第三方网站再发起攻击,这样目标网站就无法防止
  - 攻击者不能获取到用户Cookies,包括子域名,而是利用Cookies的特性冒充用户身份进行攻击
  - 通常是跨域攻击,因为攻击者更容易掌握第三方网站而不是只能利用目标网站自身漏洞
  - 攻击方式包括图片,URL,CORS,表单,甚至直接嵌入第三方论坛,文章等等,难以追踪

- **攻击方式：**

  - 自动 GET 请求
  - 自动 POST 请求
  - 诱导点击发送 GET 请求。

- 防范措施

  - 利用cookie的SameSite属性。（Cookie允许服务器要求某个cookie在跨站请求时不会被发送。问题：SameSite不支持子域名）

    `SameSite`可以设置为三个值，`Strict`、`Lax`和`None`。

    **a.** 在`Strict`模式下，浏览器完全禁止第三方请求携带Cookie。比如请求`sanyuan.com`网站只能在`sanyuan.com`域名当中请求才能携带 Cookie，在其他网站请求都不能。

    **b.** 在`Lax`模式，就宽松一点了，但是只能在 `get 方法提交表单`况或者`a 标签发送 get 请求`的情况下可以携带 Cookie，其他情况均不能。

    **c.** 在`None`模式下，也就是默认模式，请求会自动携带上 Cookie。

  - 验证来源站点。

    用到请求头中的两个字段: **Origin**和**Referer**。

    其中，**Origin**只包含域名信息，而**Referer**包含了`具体`的 URL 路径，包含了当前请求页面的来源页面的地址，即表示当前页面是通过此来源页面里的链接进入的。

    当然，这两者都是可以伪造的，通过 Ajax 中自定义请求头即可，安全性略差。

  - 验证码、Token验证。

    在前后端交互中携带一个有效验证“令牌”来防范CSRF攻击,大概流程:

    1. 当用户首次登录成功之后, 服务端会生成一个唯一性和随机性的 token 值保存在服务器的Session或者其他缓存系统中，再将这个token值返回给浏览器；
    2. 浏览器拿到 token 值之后本地保存；
    3. 当浏览器再次发送网络请求的时候,就会将这个 token 值附带到参数中(或者通过Header头)发送给服务端；
    4. 服务端接收到浏览器的请求之后,会取出token值与保存在服务器的Session的token值做对比验证其正确性和有效期。



## 数字证书认证机构的业务流程

![image-20200426233554915](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge7ldwvz54j30y40pykjl.jpg)



## HTTP1.0、HTTP1.1 和 HTTP2.0 的区别

**HTTP的基本优化**

影响一个 HTTP 网络请求的因素主要有两个：**带宽和延迟。**

- **带宽：**如果说我们还停留在拨号上网的阶段，带宽可能会成为一个比较严重影响请求的问题，但是现在网络基础建设已经使得带宽得到极大的提升，我们不再会担心由带宽而影响网速，那么就只剩下延迟了。

- **延迟：**

- - 浏览器阻塞（HOL blocking）：浏览器会因为一些原因阻塞请求。浏览器对于同一个域名，同时只能有 4 个连接（这个根据浏览器内核不同可能会有所差异），超过浏览器最大连接数限制，后续请求就会被阻塞。
  - DNS 查询（DNS Lookup）：浏览器需要知道目标服务器的 IP 才能建立连接。将域名解析为 IP 的这个系统就是 DNS。这个通常可以利用DNS缓存结果来达到减少这个时间的目的。
  - 建立连接（Initial connection）：HTTP 是基于 TCP 协议的，浏览器最快也要在第三次握手时才能捎带 HTTP 请求报文，达到真正的建立连接，但是这些连接无法复用会导致每次请求都经历三次握手和慢启动。三次握手在高延迟的场景下影响较明显，慢启动则对文件类大请求影响较大。

**HTTP1.0和HTTP1.1的一些区别**

HTTP1.0最早在网页中使用是在1996年，那个时候只是使用一些较为简单的网页上和网络请求上，而HTTP1.1则在1999年才开始广泛应用于现在的各大浏览器网络请求中，同时HTTP1.1也是当前使用最为广泛的HTTP协议。 主要区别主要体现在：

1. **缓存处理**，在HTTP1.0中主要使用header里的If-Modified-Since,Expires来做为缓存判断的标准，HTTP1.1则引入了更多的缓存控制策略例如Entity tag，If-Unmodified-Since, If-Match, If-None-Match等更多可供选择的缓存头来控制缓存策略。
2. **带宽优化及网络连接的使用**，HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。
3. **错误通知的管理**，在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。
4. **Host头处理**，在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。
5. **长连接**，HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启Connection： keep-alive，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。

**HTTP2.0和HTTP1.X相比的新特性**

- **二进制分帧**（Binary Format），在应用层跟传送层之间增加了一个二进制分帧层，改进传输性能，实现低延迟和高吞吐量。HTTP1.x的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。
- **多路复用**（MultiPlexing），即一个tcp/ip连接可以请求多个资源。即连接共享，即每一个request都是是用作连接共享机制的。一个request对应一个id，这样一个连接上可以有多个request，每个连接的request可以随机的混杂在一起，接收方可以根据request的 id将request再归属到各自不同的服务端请求里面。
- **header压缩**，如上文中所言，对前面提到过HTTP1.x的header带有大量信息，而且每次都要重复发送，HTTP2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。
- **服务端推送**（server push），服务端可以对客户端的一个请求发出多个响应，可以主动通知客户端。例如我的网页有一个sytle.css的请求，在客户端收到sytle.css数据的同时，服务端会将sytle.js的文件推送给客户端，当客户端再次尝试获取sytle.js时就可以直接从缓存中获取到，不用再发请求了。
- **请求优先级**（如果流被赋予了优先级，它就会基于这个优先级来处理，由服务器决定需要多少资源来处理该请求。）

**HTTP2.0的多路复用和HTTP1.X中的长连接复用有什么区别？**

- HTTP/1.* 一次请求-响应，建立一个连接，用完关闭；每一个请求都要建立一个连接；
- HTTP/1.1 Pipeling解决方式为，若干个请求排队串行化单线程处理，后面的请求等待前面请求的返回才能获得执行机会，一旦有某请求超时等，后续请求只能被阻塞，毫无办法，也就是人们常说的线头阻塞；
- HTTP/2多个请求可同时在一个连接上并行执行。某个请求任务耗时严重，不会影响到其它连接的正常执行；



## HTTP缓存

#### 强缓存与弱缓存

缓存可以简单的划分成两种类型：`强缓存`（`200 from cache`）与`协商缓存`（`304`）

区别简述如下：

- 强缓存（`200 from cache`）时，浏览器如果判断本地缓存未过期，就直接使用，无需发起http请求
- 协商缓存（`304`）时，浏览器会向服务端发起http请求，然后服务端告诉浏览器文件未改变，让浏览器使用本地缓存

对于协商缓存，使用`Ctrl + F5`强制刷新可以使得缓存无效

但是对于强缓存，在未过期时，必须更新资源路径才能发起新的请求（更改了路径相当于是另一个资源了，这也是前端工程化中常用到的技巧）

#### 缓存头部简述

上述提到了强缓存和协商缓存，那它们是怎么区分的呢？

答案是通过不同的http头部控制

先看下这几个头部：

```
If-None-Match/E-tag、If-Modified-Since/Last-Modified、Cache-Control/Max-Age、Pragma/Expires
```

这些就是缓存中常用到的头部，这里不展开。仅列举下大致使用。

属于强缓存控制的：

```
（http1.1）Cache-Control/Max-Age
（http1.0）Pragma/Expires
```

注意：**`Max-Age`不是一个头部，它是`Cache-Control`头部的值**

属于协商缓存控制的：

```
（http1.1）If-None-Match/E-tag
（http1.0）If-Modified-Since/Last-Modified
```

可以看到，上述有提到`http1.1`和`http1.0`，这些不同的头部是属于不同http时期的

再提一点，其实HTML页面中也有一个meta标签可以控制缓存方案-`Pragma`

```html
<META HTTP-EQUIV="Pragma" CONTENT="no-cache">
```

不过，这种方案还是比较少用到，因为支持情况不佳，譬如缓存代理服务器肯定不支持，所以不推荐

#### 头部的区别

首先明确，http的发展是从http1.0到http1.1

而在http1.1中，出了一些新内容，弥补了http1.0的不足。

**http1.0中的缓存控制：**

- `Pragma`：严格来说，它不属于专门的缓存控制头部，但是它设置`no-cache`时可以让本地强缓存失效（属于编译控制，来实现特定的指令，主要是因为兼容http1.0，所以以前又被大量应用）
- `Expires`：服务端配置的，属于强缓存，用来控制在规定的时间之前，浏览器不会发出请求，而是直接使用本地缓存，注意，Expires一般对应服务器端时间，如`Expires：Fri, 30 Oct 1998 14:19:41`
- `If-Modified-Since/Last-Modified`：这两个是成对出现的，属于协商缓存的内容，其中浏览器的头部是`If-Modified-Since`，而服务端的是`Last-Modified`，它的作用是，在发起请求时，如果`If-Modified-Since`和`Last-Modified`匹配，那么代表服务器资源并未改变，因此服务端不会返回资源实体，而是只返回头部，通知浏览器可以使用本地缓存。`Last-Modified`，顾名思义，指的是文件最后的修改时间，而且只能精确到`1s`以内

**http1.1中的缓存控制：**

- `Cache-Control`：缓存控制头部，有no-cache、max-age等多种取值
- `Max-Age`：服务端配置的，用来控制强缓存，在规定的时间之内，浏览器无需发出请求，直接使用本地缓存，注意，Max-Age是Cache-Control头部的值，不是独立的头部，譬如`Cache-Control: max-age=3600`，而且它值得是绝对时间，由浏览器自己计算
- `If-None-Match/E-tag`：这两个是成对出现的，属于协商缓存的内容，其中浏览器的头部是`If-None-Match`，而服务端的是`E-tag`，同样，发出请求后，如果`If-None-Match`和`E-tag`匹配，则代表内容未变，通知浏览器使用本地缓存，和Last-Modified不同，E-tag更精确，它是类似于指纹一样的东西，基于`FileEtag INode Mtime Size`生成，也就是说，只要文件变，指纹就会变，而且没有1s精确度的限制。

**Max-Age相比Expires？**

`Expires`使用的是服务器端的时间

但是有时候会有这样一种情况-客户端时间和服务端不同步

那这样，可能就会出问题了，造成了浏览器本地的缓存无用或者一直无法过期

所以一般http1.1后不推荐使用`Expires`

而`Max-Age`使用的是客户端本地时间的计算，因此不会有这个问题

因此推荐使用`Max-Age`。

注意，如果同时启用了`Cache-Control`与`Expires`，`Cache-Control`优先级高。

**E-tag相比Last-Modified？**

`Last-Modified`：

- 表明服务端的文件最后何时改变的
- 它有一个缺陷就是只能精确到1s，
- 然后还有一个问题就是有的服务端的文件会周期性的改变，导致缓存失效

而`E-tag`：

- 是一种指纹机制，代表文件相关指纹
- 只有文件变才会变，也只要文件变就会变，
- 也没有精确时间的限制，只要文件一遍，立马E-tag就不一样了

如果同时带有`E-tag`和`Last-Modified`，服务端会优先检查`E-tag`

各大缓存头部的整体关系如下图

![16217a2d2b4b73d0](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge9vo2jqvzj30om0ezq37.jpg)



## [如何解决ajax跨域](https://juejin.im/post/5a3770e6518825127e7456a4)

一般ajax跨域解决就是通过JSONP解决或者CORS解决,如以下:(注意，现在已经几乎不会再使用JSONP了，所以JSONP了解下即可)

#### JSONP方式解决跨域问题

jsonp解决跨域问题是一个比较古老的方案(实际中不推荐使用),这里做简单介绍(实际项目中如果要使用JSONP,一般会使用JQ等对JSONP进行了封装的类库来进行ajax请求)

##### 实现原理

JSONP之所以能够用来解决跨域方案,主要是因为 `script` 脚本拥有跨域能力,而JSONP正是利用这一点来实现。具体原理如图

![16069ea85abd92fe](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge9w9511qkj30p10jhmxw.jpg)

##### 实现流程

JSONP的实现步骤大致如下(参考了来源中的文章)

- 客户端网页网页通过添加一个`script`元素，向服务器请求JSON数据，这种做法不受同源政策限制

  ```javascript
  function addScriptTag(src) {
    var script = document.createElement('script');
    script.setAttribute("type","text/javascript");
    script.src = src;
    document.body.appendChild(script);
  }
  
  window.onload = function () {
    addScriptTag('http://example.com/ip?callback=foo');
  }
  
  function foo(data) {
    console.log('response data: ' + JSON.stringify(data));
  };
  ```

  请求时,接口地址是作为构建出的脚本标签的src的,这样,当脚本标签构建出来时,最终的src是接口返回的内容

- 服务端对应的接口在返回参数外面添加函数包裹层

  ```
  foo({
    "test": "testData"
  });
  ```

- 由于`script`>元素请求的脚本，直接作为代码运行。这时，只要浏览器定义了foo函数，该函数就会立即调用。作为参数的JSON数据被视为JavaScript对象，而不是字符串，因此避免了使用JSON.parse的步骤。

注意,一般的JSONP接口和普通接口返回数据是有区别的,所以接口如果要做JSONO兼容,需要进行判断是否有对应callback关键字参数,如果有则是JSONP请求,返回JSONP数据,否则返回普通数据

**使用注意**

基于JSONP的实现原理,所以JSONP只能是“GET”请求,不能进行较为复杂的POST和其它请求,所以遇到那种情况,就得参考下面的CORS解决跨域了(所以如今它也基本被淘汰了)

#### [CORS](http://www.ruanyifeng.com/blog/2016/04/cors.html)解决跨域问题

CORS的原理上文中已经介绍了，这里主要介绍的是，实际项目中，后端应该如何配置以解决问题(因为大量项目实践都是由后端进行解决的)，这里整理了一些常见的后端解决方案:

##### Node.js后台配置(express框架)

Node.js的后台也相对来说比较简单就可以进行配置。只需用express如下配置:

```javascript
app.all('*', function(req, res, next) {
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "X-Requested-With");
    res.header("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS");
    res.header("X-Powered-By", ' 3.2.1')
        //这段仅仅为了方便返回json而已
    res.header("Content-Type", "application/json;charset=utf-8");
    if(req.method == 'OPTIONS') {
        //让options请求快速返回
        res.sendStatus(200); 
    } else { 
        next(); 
    }
});
```

##### JAVA后台配置

JAVA后台配置只需要遵循如下步骤即可:

- 第一步:获取依赖jar包

  下载 [cors-filter-1.7.jar](https://dailc.github.io/staticResource/blog/basicKnowledge/ajax/resource/cors-filter-2.4.jar), [java-property-utils-1.9.jar](https://dailc.github.io/staticResource/blog/basicKnowledge/ajax/resource/java-property-utils-1.9.1.jar) 这两个库文件放到lib目录下。(放到对应项目的webcontent/WEB-INF/lib/下)

- 第二步:如果项目用了Maven构建的,请添加如下依赖到pom.xml中:(非maven请忽视)

```xml
<dependency>
    <groupId>com.thetransactioncompany</groupId>
    <artifactId>cors-filter</artifactId>
    <version>[ version ]</version>
</dependency>
```

其中版本应该是最新的稳定版本,CORS过滤器

- 第三步:添加CORS配置到项目的Web.xml中( App/WEB-INF/web.xml)

```xml
<!-- 跨域配置-->    
<filter>
  <!-- The CORS filter with parameters -->
  <filter-name>CORS</filter-name>
  <filter-class>com.thetransactioncompany.cors.CORSFilter</filter-class>

  <!-- Note: All parameters are options, if omitted the CORS 
             Filter will fall back to the respective default values.
          -->
  <init-param>
    <param-name>cors.allowGenericHttpRequests</param-name>
    <param-value>true</param-value>
  </init-param>

  <init-param>
    <param-name>cors.allowOrigin</param-name>
    <param-value>*</param-value>
  </init-param>

  <init-param>
    <param-name>cors.allowSubdomains</param-name>
    <param-value>false</param-value>
  </init-param>

  <init-param>
    <param-name>cors.supportedMethods</param-name>
    <param-value>GET, HEAD, POST, OPTIONS</param-value>
  </init-param>

  <init-param>
    <param-name>cors.supportedHeaders</param-name>
    <param-value>Accept, Origin, X-Requested-With, Content-Type, Last-Modified</param-value>
  </init-param>

  <init-param>
    <param-name>cors.exposedHeaders</param-name>
    <!--这里可以添加一些自己的暴露Headers   -->
    <param-value>X-Test-1, X-Test-2</param-value>
  </init-param>

  <init-param>
    <param-name>cors.supportsCredentials</param-name>
    <param-value>true</param-value>
  </init-param>

  <init-param>
    <param-name>cors.maxAge</param-name>
    <param-value>3600</param-value>
  </init-param>

</filter>

<filter-mapping>
  <!-- CORS Filter mapping -->
  <filter-name>CORS</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

请注意,以上配置文件请放到web.xml的前面,作为第一个filter存在(可以有多个filter的)

- 第四步:可能的安全模块配置错误(注意，某些框架中-譬如公司私人框架，有安全模块的，有时候这些安全模块配置会影响跨域配置，这时候可以先尝试关闭它们)

#### 代理请求方式解决接口跨域问题

注意，由于接口代理是有代价的，所以这个仅是开发过程中进行的。

与前面的方法不同，前面CORS是后端解决，而这个主要是前端对接口进行代理，也就是:

- 前端ajax请求的是本地接口
- 本地接口接收到请求后向实际的接口请求数据，然后再将信息返回给前端
- 一般用node.js、nginx即可代理



## JS 继承

#### 组合方式实现继承

运用`Object.create(proto)`与`call`实现:

> **`Object.create(proto[, propertiesObject])`**方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__。

```js
// Shape - 父类(superclass)
function Shape() {
  this.x = 0;
  this.y = 0;
}

// 父类的方法
Shape.prototype.move = function(x, y) {
  this.x += x;
  this.y += y;
  console.info('Shape moved.');
};

// Rectangle - 子类(subclass)
function Rectangle() {
  Shape.call(this); // call super constructor.
}

// 子类续承父类
Rectangle.prototype = Object.create(Shape.prototype);
Rectangle.prototype.constructor = Rectangle;

var rect = new Rectangle();

console.log('Is rect an instance of Rectangle?',
rect instanceof Rectangle); // true
console.log('Is rect an instance of Shape?',
rect instanceof Shape); // true
rect.move(1, 1); // Outputs, 'Shape moved.'
```



## 异步加载

#### 异步加载的方式

1. 动态脚本加载
2. defer
3. async

#### 异步加载的区别

1. defer 是在 HTML 解析完之后才会执行，如果是多个，按照加载的顺序依次执行。`defer`脚本会在`DOMContentLoaded`和`load`事件之前执行。
2. async 是在脚本加载完之后立即执行，如果是多个，执行顺序和加载顺序无关。`async`会在`load`事件之前执行，但并不能确保与`DOMContentLoaded`的执行先后顺序。



## 为什么用transform写动画不用position top left？

主要还是从浏览器渲染的性能方面考虑。

1. **浏览器渲染过程**

我们知道，浏览器中有JS引擎和渲染引擎，对于HTML页面的渲染就靠渲染引擎来完成。

浏览器渲染主要包括Parse Html、Recalculate Style、Layout、Paint、Image Decode、Image Resize和Composite Layers等。相对应的中文表述就是：html解析、查找并计算样式、排布、绘制、图片解码、图片大小设置、合并图层并输出页面到屏幕。浏览器最终渲染出来的页面，跟Photoshop有点类似，是由多个图层合并而来。

2. **transform的原理**

transform是通过创建一个RenderLayers合成层，拥有独立的GraphicsLayers。每一个GraphicsLayers都有一个Graphics Context，其对应的RenderLayers会paint进Graphics Context中。合成器（Compositor）最终会负责将由Graphics Context输出的位图合并成最终屏幕展示的图案。

满足如下条件的RenderLayers，会被认为是一个独立的合成层：

- 有3D或者perspective transform的CSS属性的层
- video元素的层
- canvas元素的层
- flash
- 对opacity和transform应用了CSS动画的层
- 使用了CSS滤镜（filters）的层
- 有合成层后代的层
- 同合成层重叠，且在该合成层上面（z-index）渲染的层

如果RenderLayer是一个合成层，那么它有属于它自己的单独的GraphicsLayer，否则它和它的最近的拥有GraphicsLayer的父layer共用一个GraphicsLayer。

由此可见，transform发生在Composite Layer这一步，它所引起的paint也只是发生在**单独的GraphicsLayer**中，并**不会引起整个页面的回流重绘**。

3. **GPU**

我们经常会听到GPU会加速渲染，那GPU在这里又扮演什么角色呢？

前面说到，合成器会负责将层合成绘制为最终的屏幕画面。在硬件加速体系结构，合成由GPU负责。在chrome浏览器多进程模型中，有一个专门的进程来负责传递Render进程的命令，即GPU进程。Render进程和GPU进程是通过共享内存传递的。

Render进程可以快速的将命令发给命令缓冲区，并且返回到CPU密集的render活动中，留给GPU进程去处理这些命令。我们可以充分利用多内核机器上的GPU进程和CPU进程。这也是为什么GPU会加速渲染，使transform渲染速度更快的又一原因。

4. **margin top/left**

**`position`**属性用于指定一个元素在文档中的定位方式。[`top`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/top)，[`right`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/right)，[`bottom`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/bottom) 和 [`left`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/left) 属性则决定了该元素的最终位置。简言之，可以改变元素的位移。在浏览器页面渲染的时候，`top/left`可以控制元素的位置，也就是说，改变`top/left`，就会改变render tree的结构，必定会**引起页面layout回流和repaint重绘**。

因此，从浏览器性能考虑，transform会比`top/left`更省时间。

5. **transform的局限性**

transform实际上也是用到了GPU加速，也就是说占用了内存。由此可见创建GraphicsLayer，虽然节省了layout，paint阶段，但Layer创建的越多，占用内存就会越大，而过多的渲染开销会超过性能的改善。

因此，当且仅当需要的时候，才会为元素创建渲染层。



## [CSS实现宽高等比例自适应矩形](https://juejin.im/post/5b0784566fb9a07abd0e14ae#heading-0)

#### 概述

今天遇到一个很有趣的问题：**「如何实现一个宽度自适应，高度为宽度的一半的矩形」**。

经过搜索引擎的筛选和自己的反复试验,发现使用`padding-bottom`是最完美的解决方案。

#### 解决方案

首先我们要明白，`padding-top/bottom`和`margin-top/bottom`都是**相对于父元素的宽度**来计算的，我们可以利用这一属性来实现我们的需求。

代码如下：

```css
<div class="scale"></div>
.scale {
  width: 100%;
  height: 0;
  padding-bottom: 50%;
}
```

这其中的关键点就是`height: 0;`和`padding-bottom: 50%;`。

我们将元素的高度由`padding`撑开，由于`padding`是根据父元素宽度计算的，所以高度也就变成了相对父元素宽度，同时要将`height`设置为 0，这是为了将元素高度完全交给`padding`负责。

最后`padding-bottom`的值设为`width`的值一半，就可以实现高度是宽度的一半且自适应啦。

#### 改进

光是这样写还是不够的，因为元素的`height`为 0，导致该元素里面再有子元素的时候，就无法正常设置高度。所以我们需要用到`position: absolute;`。代码如下：

```css
<div class="scale">
    <div class="item">
        这里是所有子元素的容器
    </div>
</div>
.scale {
  width: 100%;
  padding-bottom: 56.25%;
  height: 0;
  position: relative; //
}

.item {
  width: 100%;
  height: 100%;
  background-color: aquamarine;
  position: absolute; //
}
```

#### 继续改进

解决了子元素的问题，那么我们再来看看元素本身。由于我们一开始的需求是宽高比 `2:1`,这种比较好实现，但是后来需求又想要 `16:9` 的宽高比，而且宽度不是 100%，那这样计算 `padding-bottom`的时候就很麻烦了。如何解决呢？

这时候我们需要在外层再套一个父元素，将宽度的控制交给这个父元素来做。

代码如下：

```html
<body>
    <div class="box">
        <div class="scale">
            <div class="item">
                item
            </div>
        </div>
    </div>
</body>
/* box 用来控制宽度 */
.box {
  width: 80%;
}
/* scale 用来实现宽高等比例 */
.scale {
  width: 100%;
  padding-bottom: 56.25%;
  height: 0;
  position: relative;
}
/* item 用来放置全部的子元素 */
.item {
  width: 100%;
  height: 100%;
  background-color: aquamarine;
  position: absolute;
}
```

如此，就可以完美解决。