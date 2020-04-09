# Symbol
## 概述
ES5的对象属性名都是字符串，这容易造成属性名的冲突。比如，你使用了一个他人提供的对象，但又想为这个对象添加新的方法(mixin模式)，新方法的名字就有可能与现有方法产生冲突。如果有一种机制，保证每个属性的名字都是独一无二的就好了，这样就从根本上防止属性名的冲突。这就是ES6引入Symbol的原因。  
ES6引入了一种新的原始数据类型Symbol，表示独一无二的值。  
Symbol值通过Symbol函数生成。这就是说，对象的属性名现在可以有两种类型。一种是原来就有的字符串，另一种就是新增的Symbol类型。凡是属性名属于Symbol类型，就是独一无二的，可以保证不会与其他属性名产生冲突。  
Symbol函数可以接受一个字符串作为参数，表示对Symbol实例的描述，主要是为了在控制台显现，或者转为字符串时，比较容易区分。
```js
let s1 = Symbol('foo')
let s2 = Symbol('bar')

s1 // Symbol(foo)
s2 // Symbol(bar)
s1.toString() // 'Symbol(foo)'
s2.toString() // 'Symbol(bar)'
```
上面代码中，s1和s2是两个Symbol值。如果不加参数，它们在控制台的输出都是Symbol()，不利于区分。有了参数以后，就等于为它们加上了描述，输出的时候就能够分清，到底是哪一个值。  
如果Symbol的参数是一个对象，就会调用该对象的toString方法，将其转为字符串，然后才生成一个Symbol值  
```js
const obj = {
  toString () {
    return 'abc'
  }
}
const sym = Symbol(obj)
sym // Symbol(abc)
```
注意，Symbol函数的参数只是表示对当前Symbol值的描述，因此相同参数的Symbol函数的返回值是不相等的。  
Symbol值可以显示转为字符串  
```js
let sym = Symbol('My symbol')
String(sym) // 'Symbol(My symbol)'
sym.toString() // 'Symbol(My symbol)'
```
Symbol值可以转为布尔值，但是不能转为数值  
```js
let sym = Symbol()
Boolean(sym) // true
!sym // false
```
## Symbol.prototype.description
创建Symbol的时候，可以添加一个描述
```js
const sym = Symbol('foo')
```
```js
const sym = Symbol('foo')
String(sym) // 'Symbol(foo)'
sym.toString() // 'Symbol(foo)'
```
上面的用法不太方便，ES2019提供了一个实例属性，直接返回Symbol的描述  
```js
const sym = Symbol('foo')
sym.description // 'foo'
```
## 作为属性名Symbol
由于每一个Symbol值都是不相等的，这意味着Symbol值可以作为标识符，用于对象的属性名，就能保证不会出现同名的属性。这对一个对象由多个模块构成的情况非常有用，能防止某一个键被不小心改写或覆盖  
```js
let mySymbol = Symbol()

let a = {}
a[mySymbol] = 'Hello!'

let a = {
  [mySymbol]: 'Hello!'
}

let a = {}
Object.defineProperty(a, mySymbol, { value: 'Hello!' })

// 以上写法都得到相同结果
a[mySymbol] // ’Hello!‘
```
Symbol类型还可以用于定义一组常量，保证这组常量的值都是不相等的  
```js
const log = {}
log.levels = {
  DEBUG: Symbol('debug'),
  INFO: Symbol('info'),
  WARN: Symbol('warn')
}
console.log(log.levels.DEBUG, 'debug message')
console.log(log.levels.INFO, 'info message')
```
注意：Symbol值作为属性名时，该属性还是公开属性，不是私有属性  
## 属性名遍历
Symbol作为属性名，遍历对象的时候，该属性不会出现杂for...in,for...of循环中，也不会被Object.keys(),Object.getOwnPropertyNames(),JSON.stringify()返回。  
但是，他也不是私有属性，有一个Object.getOwnPropertySymbols()方法，可以获取指定对象的所有Symbol属性名。
下面是另一个例子，Object.getOwnPropertySymbols()方法与for...in循环、Object.getOwnPropertyNames方法进行对比的例子。
```js
const obj = {}
const foo = Symbol('foo')
obj[foo] = 'bar'
for (let i in obj) {
  console.log(i) // 无输出
}

Object.getOwnPropertyNames(obj) // []
Object.getOwnPropertySymbols(obj) // [Symbol(foo)]
```
Reflect.ownKeys()方法可以返回所有类型的键名，包括常规健名和Symbol键名。  
## Symbol.for()，Symbol.keyFor()
有时我们希望重新使用同一个Symbol值，Symbol.for()方法可以做到这一点。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的Symbol值。如果有，就返回这个Symbol值，否则就新建一个以该字符串为名称的Symbol值，并将其注册到全局。
```js
let s1 = Symbol.for('foo')
let s2 = Symbol.for('foo')
s1 === s2 // true
```
Symbol.keyFor()方法返回一个已登记的Symbol类型值的key
```js
let s1 = Symbol.for('foo')
Symbol.keyFor(s1) // foo
let s2 = Symbol('foo')
Symbol.keyFor(s2) // undefined
```
注意，Symbol.for()为Symbol值登记的名字，是全局环境的，不管有没有在全局环境运行。  
```js
function foo () {
  return Symbol.for('bar')
}
const x = foo()
const y = Symbol.for('bar')
console.log(x === y) // true
```
Symbol.for()的这个全局登记特性，可以用在不同的iframe或service worker中取到同一个值。
## Singleton模式
Singleton模式指的是调用一个类，任何时候返回的都是同一个实例。  