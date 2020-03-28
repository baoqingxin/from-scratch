# this
## Reference
什么是reference呢？  
它就是用来解释诸如delete,typeof,以及赋值操作行为的.
> 这里的Reference是一个Specification Type，也就是只存在于规范里的抽象类型。他们是为了更好地描述语言的底层行为逻辑才存在的，但并不存在于实际的js代码中  
## Reference组成
`Reference`类型有三个属性的对象：
- base
- propertyName(referenced name)
- strict reference
base value就是属性所在的对象或者是EnvironmentRecord,它的值只能是undefined, an object, a boolean, a string, a number, or an environment record中的一种。  
referenced name就是属性的名称。  
strict，它表示是否以严格模式来解析reference  
```js
'use strict'
var foo = 1
// 对应的reference就是
const fooReference = {
  base: EnvironmentRecord,
  propertyName: 'foo',
  strict: true
}
// 再举个例子
var foo = {
  bar: function () {
    return this
  }
}
foo.bar() // foo

// 它对应的reference就是
var barReference = {
  base: foo,
  propertyName: 'bar',
  strict: false
}
```
规范中还提供了获取Reference组成部分的方法，比如GetBase和IsPropertyReference  
GetBase：返回reference的base value
>GetBase(v). Returns the base value component of the reference v

isPropertyReference
>IsPropertyRerference(v).returns true if either the base value is an object or HasPrimitiveBase(v) is true; otherwise returns false.
简单理解：如果base value是一个对象，就返回true

## GetValue
该方法是用来从Reference中获取对应值的方法
```js
var foo = 1
var fooReference = {
  base: EnvironmentRecord,
  name: 'foo',
  strict: false
}
GetValue(fooReference) 、、 1
```
GetValue返回对象属性真正的值，但是要注意
调用GetValue返回的将是具体的值，而不再是一个Reference
## 如何确定this的值
1. 计算MemberExpression的结果赋值给ref
2. 判断ref是不是一个Reference类型
- 如果ref是Reference,并且IsPrototype(ref)是true，那么this的值为GetBase(ref)
- 如果ref不是Reference, 那么this的值为undefined

## 具体分析
1. 计算MemberExpression的结果赋值给ref
MemberExpression:
- PrimaryExpression // 原始表达式
- FunctionExpression // 函数定义表达式
- MemberExpress[Expression] // 属性表达式
- MemberExpress.IdentifierName // 属性访问表达式
- new MemberExpression Arguments // 对象创建表达式
```js
function foo () {
  console.log(this)
}
foo() // MemberExpression是foo
function foo () {
  return function () {
    console.log(this)
  }
}
foo()() // MemberExpression是foo()

var foo = {
  bar: function () {
    return this
  }
}
foo.bar() // 是foo.bar
```
简单理解MemberExpression其实是()左边的部分
2. 判断ref是不是一个reference类型
关键在于如何处理各种MemberExpression，返回的结果是不是一个Reference类型
```js
var value = 1
var foo = {
  value: 2,
  bar: function () {
    return this.value
  }
}

foo.bar() // 2
(foo.bar)() // 2
(foo.bar = foo.bar)() // 1
(false || foo.bar)() // 1
(foo.bar, foo.bar)() // 1
```
## 补充
```js
function foo () {
  console.log(this)
}
foo()
```
MemberExpression是foo,解析标识符,会返回一个reference类型的值
```js
var fooRerence = {
  base: EnvironmentRecord,
  name: 'foo',
  strict: false
}
```
因为base value是EnvironmentRecord，不是object类型 所以isPropertyReference(ref)的结果是false，所以会调用ImplicitThisValue(ref) 该函数始终返回undefined
## 另附this指向的优先级(由低到高)
1. 普通情况下就是全局，浏览器里就是window,在use strict情况下就是undefined
2. 属性访问
3. call, apply
4. bind
5. new
6. () => {}

参考： 
- [JavaScript深入之从ECMAScript规范解读this](https://github.com/mqyqingfeng/Blog/issues/7)
- [JavaScript this 的六道坎](https://blog.crimx.com/2016/05/12/understanding-this/)
<!-- Reference的值在下面两种情况下存在:
- 处理标识符(identifier)
- 或者使用属性访问器  
标识符是变量名、函数名、函数参数名和全局对象的非限定属性名
```js
var foo = 10
function bar () {}
```
```js
var fooReference = {
  base: global,
  propertyName: 'foo'
}
var barReference = {
  base: global,
  propertyName: 'bar'
}
``` -->

<!-- 为了从Reference中获取真实值，有一个GetValue方法，可以用下面的伪代码来描述:
```js
function GetValue (value) {
  if (Type(value) != Reference) {
    return value
  }
  var base = GetBase(value)
  if (base === null) {
    throw new ReferenceError
  }
  return base.[[Get]]((GetPropertyName(value)))
}
```
[[Get]]方法返回对象的真实值
```
GetValue(fooReference) // 10
GetValue(barReference) // function object 'bar'
```
访问属性有两种方式：
- foo.bar()
- foo['bar']()
```JS
var fooBarReference = {
  base: foo,
  propertyName: 'bar'
}
GetValue(fooBarReference) // function object 'bar'
```
在函数上下文的值如何确定Reference呢
- 在函数上下文中，this的值由函数调用方提供，并有调动表达式的当前形式确定
- 如果在调用括号(...)的左侧，有一个应用类型的值，则此值设置为该引用类型值的基础对象
- 在所有其他情况下(及与Reference不同的任何其他类型)，此值始终为null。但是由于此值在null
中没有任何意义，因此它将隐式转换为全局对象

```js
function foo () {
  return this
}
foo() // global
```
我们看到在调用括号的左侧有一个Reference(因为foo是一个标识符)
```js
var fooReference = {
  base: global,
  propertyName: 'foo'
}
```
与属性访问相似
```js
var foo = {
  bar: function () {
    return this
  }
}
foo.bar() // foo
```
```js
var fooBarReference = {
  base: foo,
  propertyName: 'bar'
}
```
```js
var test = foo.bar
test() // global

var testReferene = {
  base: global,
  propertyName: 'test'
}
// 在es5严格模式下，此值不强制到全局对象，而是设置undefined
```
现在我们可以准确的说明，为什么用不同形式的调用表达式激活的同一函数也具有不用的this，答案是Reference不同的中间值.
```js
function foo () {
  console.log(this)
}
foo() // global

var fooReference = {
  base: global,
  propertyName: 'foo'
}

console.log(foo === foo.prototype.constructor) // true
foo.prototype.constructor() // foo.prototype, because
var fooPrototypeConstructorReference = {
  base: foo.prototype,
  propertyName: 'constructor'
}
```
函数调用且非Reference类型
如果等号左边不是reference类型，则this变成null, 指向全局对象
```js
(function () {
  console.log(this) // null => global
})()
```
```js
var foo = {
  bar: function () {
    console.log(this)
  }
}
foo.bar() // reference, => foo
(foo.bar)() // reference => foo

(foo.bar = foo.bar)() // global
(false || foo.bar)() // global
(foo.bar, foo.bar)() // global
```
```js
function A () {
  console.log(this)
  this.x = 10
}
var a = new A()
console.log(a.x) // 10
``` -->
