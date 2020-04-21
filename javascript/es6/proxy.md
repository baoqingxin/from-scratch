# Proxy
## 概念
Proxy用于修改某些操作的默认行为，等于在语言层面作出修改，所以属于一种“元编程”，即对编程语言进行编程。  
Proxy可以理解为代理某些操作，可以译为“代理器”
```js
var obj = new Proxy({}, {
  get: function (target, propKey, receiver) {
    console.log(`getting ${propKey}!`)
    return Reflect.get(target, propKey, receiver)
  },
  set: function (target, propKey, value, receiver) {
    console.log(`setting ${propKey}!`)
    return Reflect.set(target, propKey, value, receiver)
  }
})

obj.count = 1
// setting count!
++obj.count
// getting count!
// setting count!
// 2
```
上面代码说明，Proxy实际上重载(overload)了点运算符，即用自己的定义覆盖了语言的原始定义。  
ES6原生提供Proxy构造函数，用来生成Proxy实例。
```js  
var proxy = new Proxy(target, handler)
```
Proxy对象的所有用法，都是上面这种形式，不同的只是handler参数的写法。其中，new Proxy()表示生成一个Proxy实例，target参数表示索要拦截的目标对象，handler参数也是一个对象，用来定制拦截行为。  
下面是另一个拦截读取属性行为的例子。
```js
var proxy = new Proxy({}, {
  get: function (target, propKey) {
    return 35
  }
})
proxy.time // 35
proxy.name // 35
proxy.title // 35
```
上面代码中，作为构造函数，Proxy接受两个参数。第一个参数是所要代理的目标对象(上例是一个空对象)，即如果没有Proxy的介入，操作原来要访问的就是这个对象；第二个参数是一个配置对象，对于每一个被代理的操作，需要提供一个对应的处理函数，该函数将拦截对应的操作。  
注意，要使得Proxy起作用，必须对Proxy实例进行操作，而不是针对目标进行操作。  
如果handler没有设置任何拦截，那就等同于直接通向原对象。
```js
var target = {}
var handler = {}
var proxy = new Proxy(target, handler)
proxy.a = 'b'
target.a // b
```

一个技巧是将Proxy对象，设置到object.proxy属性，从而可以在object对象上调用。
```js
var object = { proxy: new Proxy(target, handler) }
```
Proxy实例也可以作为其他对象的原型对象.
```js
var proxy = new Proxy({}, {
  get: function (target, propKey) {
    return 35
  }
})

let obj = Object.create(proxy)
obj.time // 35
```
上面代码中，proxy对象是obj对象的原型，obj对象本身并没有time属性，所以根据原型链，会在proxy对象上读取该属性，导致被拦截。

同一个拦截器函数，可以设置拦截多个操作。
```js
var handler = {
  get: function (target, name) {
    if (name === 'prototype') {
      return Object.prototype
    }
    return 'Hello, ' + name
  }
  apply: function (target, thisBinding, args) {
    return args[0]
  },
  constructor: function (target, args) {
    return {value: args[1]}
  }
}

var fproxy = new Proxy(function (x, y) {
  return x + y
}, handler)
fproxy(1, 2)
new fproxy(1, 2)
fproxy.prototype === Object.prototype
fproxy.foo === 'Hello, foo'
```
对于可以设置、但没有设置拦截的操作，则直接落在目标对象上，按照原先的方式产生结果。
下面是Proxy支持的拦截操作一览，一共13种。  
- get(target, propKey, receiver): 拦截对象属性的读取，比如proxy.foo和proxy['foo']。
- set(target, propKey, value, receiver): 拦截对象属性的设置，比如proxy.f00 = v 或 proxy['foo'] = v，返回一个布尔值。 
- has(target, propKey): 拦截propKey in proxy的操作，返回一个布尔值。
- deleteProperty(target, propKey): 拦截delete proxy[propKey]的操作，返回一个布尔值。  
- ownKeys(target): 拦截Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属性。
- getOwnPropertyDescriptor(target, propKey): 拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。
- defineProperty(target, propKey, propDesc): 拦截Object.defineProperty(proxy, propKey, propDesc)、Object.defineProperties(proxy, propDescs),返回一个布尔值。  
- preventExtensions(target): 拦截Object.preventExtensions(proxy)，返回一个布尔值
- gerPrototypeOf(target): 拦截Object.getPropertypeOf(proxy)返回一个对象。
- isExtensible(target): 拦截Object.isExtensible(proxy)，返回一个布尔值.
- setPrototypeOf(target, proto): 拦截Object.setPrototypeOf(proxy, proto),返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。  
- apply(target, object, args): 拦截Proxy实例作为函数调用的操作，比如proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)。
- construct(target, args): 拦截Proxy实例作为构造函数调用的操作，比如new proxy(...args)

## Proxy实例的方法
### get()
```js
var person = {
  name: "张三"
}

var proxy = new Proxy(person, {
  get: function (target, propKey) {
    if (propKey in target) {
      return target[propKey]
    } else {
      throw new ReferenceError("Prop name \"" + propKey + "\" does not exist.")
    }
  }
})
proxy.name // "张三"
proxy.age // 抛出一个错误
```
get方法可继承。
```js
let proto = new Proxy({}, {
  get(target, propertyKey, receiver) {
    console.log('GET ' + propertyKey)
    return target[propertyKey]
  }
})
let obj = Object.create(proto)
obj.foo // "GET foo"
```
利用Proxy,可以将读取属性的操作（get），转变为执行某个函数，从而实现属性的链是操作。  
```js
var pipe = function (value) {
  var funcStack = []
  var oproxy = new Proxy({}, {
    get: function (pipeObject, fnName) {
      if (fnName === 'get') {
        return funcStack.reduce(function (val, fn) {
          return fn(val)
        }, value)
      }
      funcStack.push(window[fnName])
      return oproxy
    }
  })
}
var double = n => n * 2;
var pow    = n => n * n;
var reverseInt = n => n.toString().split("").reverse().join("") | 0;

pipe(3).double.pow.reverseInt.get; // 63
```
上面代码设置 Proxy 以后，达到了将函数名链式使用的效果。  
下面的例子则是利用get拦截，实现一个生成各种 DOM 节点的通用函数dom。
```js
const dom = new Proxy({}, {
  get (target, property) {
    return function (attrs = {}, ...children) {
      const el = document.createElement(property)
      for (let prop of Object.keys(attrs)) {
        el.setAttribute(prop, attrs[prop])
      }
      for (let child of children) {
        if (typeof child)
      }
    }
  }
})
```
## this问题
虽然Proxy可以代理针对目标对象的访问，但它不是目标对象的透明代理，即不做任何拦截的情况下，也无法保证与目标对象的行为一致。主要原因就是在Proxy代理的情况下，目标对象内部的this关键字会指向Proxy代理。
```js
const target = {
  m: function () {
    console.log(this === proxy)
  }
}

const handler = {}

const proxy = new Proxy(target, handler)
target.m() // false
proxy.m() // true
```
