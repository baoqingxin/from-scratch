# let和const命令
## let
let用来声明变量。但是声明的变量 **只在命令所在的代码块内有效**
- 不存在变量提升, 也就意味着当你在变量声明前使用该变量时会报错
```js
console.log(a) // undefined
var a = 1

console.log(b) // Uncaught ReferenceError: b is not defined
b = 2

console.log(c) // Uncaught ReferenceError: c is not defined
let c = 3
```
- 暂时性死区
- 不允许重复声明

## ES6的块级作用域
ES5只有全局作用域和函数作用域
let实际上为JS新增了块级作用域。
```js
function f1 () {
  let n = 5
  if (true) {
    let n = 10
  }
  console.log(n) // 5
}
f1()
```
## 块级作用域与函数声明
ES5规定，函数只能在顶层作用域和函数作用域中声明，不能再块级作用域声明。
```js
// 情况一
if (true) {
  function f () {}
}

// 情况二
try {
  function f () {}
} catch (e) {
  // ...
}
```
上面两种声明，根据ES5声明的规定是非法的。但是浏览器没有遵守这个规定，
为了兼容以前的旧代码，还是支持在块级作用域中声明函数，因此实际上两个代码都能运行，不会报错。  
ES6规定，块级作用域之中，函数生命的语句行为类似于let，在块级作用域外不可引用。  
```js
function f() { console.log('I am outside!') }

(function () {
  if (false) {
    function f () { console.log('inside') }
  }
  f()
}())
```
上面代码在es5中运行会得到inside, 但是在ES6条件下会报错。
// Uncaught TypeError: f is not a function
原来，如果改变了块级作用域内声明的函数的处理规则，显然会对老代码产生很大影响。为了减轻因此产生的不兼容问题，ES6 在附录 B里面规定，浏览器的实现可以不遵守上面的规定，有自己的行为方式。

- 允许在块级作用域内声明函数。
- 函数声明类似于var，即会提升到全局作用域或函数作用域的头部。
- 同时，函数声明还会提升到所在的块级作用域的头部。  
注意，上面三条规则只对 ES6 的浏览器实现有效，其他环境的实现不用遵守，还是将块级作用域的函数声明当作let处理。

根据这三条规则，浏览器的 ES6 环境中，块级作用域内声明的函数，行为类似于var声明的变量。上面的例子实际运行的代码如下。
```js
function f () { console.log('I am outside!') }
(function () {
  var f = undefined
  if (false) {
    function f () {console.log('I am inside!')}
  }
  f()
}())
// Uncaught TypeError: f is not a function
```
## const 
### 基本用法
声明一个只读的常量。一旦声明，常量的值就不能改变。  
const只在声明所在的块级作用域有效，常量不能提升，暂时性死区。
### 本质
const实际上保证的，并不是变量的值不得改变，而是变量指向的那个内存地址所保存的数据不得改动。对于简单类型的数据(数值，字符串、布尔值)，值就保存在变量指向的内存地址，保存的只是一个指向实际数据的指针，const
只能保证指针是固定的。
## 顶层对象的属性
顶层对象，在浏览器环境指的是window对象，在 Node 指的是global对象。ES5 之中，顶层对象的属性与全局变量是等价的。

ES2020在语言层面引入了`globalThis`作为顶层对象，指向全局环境下的this.

## 再来说说解构
如果变量名和属性名不一致，必须写成下面这样
```js
let { foo: baz } = { foo: 'aaa', bar: 'bbb' }
baz // 'aaa'

let obj = { first: 'hello', last: 'world' }
let { first: f, last: l } = obj
f // hello
l // world
```
这也实际上说明，对象的解构赋值是下面形式的简写
```js
let { foo: foo, bar: bar } = { foo: 'aaa', bar: 'bbb' }
```
也就是，对象的解构赋值的内部机制，是先找到同名属性，然后再赋值给对应的变量。真正赋值的是后者，而不是前者。  
```js
let { foo: baz } = { foo: 'aaa', bar: 'bbb' }
baz // 'aaa'
foo // error
```
### 数值和布尔值的解构赋值
如果等号右边是数值和布尔值，则会先转为对象
```js
let {toString: s} = 123
s === Number.prototype.toString

let { toString: s } = true
s === Boolean.prototype.toString
```
由于undefined和null无法转为对象，所以对它们进行解构赋值，都会报错。
```js
let { prop: x } = undefined
let { prop: y } = null
```
函数参数的解构也可以使用默认值.
```js
function move ({x = 0, y = 0} = {}) {
  return [x, y]
}
move({x: 3, y: 8}) // [3, 8]
move({x: 3}) // [3, 0]
move({}) // [0, 0]
move() // [0, 0]
// 下面这种和上面的不一样
function move2 ({x, y} = {x: 0, y: 0}) {
  return [x, y]
}
move2({ x: 3, y: 8 }) // [3, 8]
move2({ x: 3 }) // [3, undefined]
move2({}) // [undefined, undefined]
move2() // [0, 0]
```