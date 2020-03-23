# 了解call、apply、bind和new

## apply
apply：
> apply()方法调用一个函数，其具有一个指定的的this值，以及作为一个数组(或类似数组的对象)提供的参数

```js
Function.prototype.apply = function (context, arr) {
  var context = Object(context) || window
  context.fn = this

  var result
  if (!arr) {
    result = context.fn()
  }
  else {
    var args = []
    for (var i = 0, len = arr.length; i < len; i++) {
      args.push('arr[' + i + ']')
    }
    result = eval('context.fn(' + args + ')')
  }

  delete context.fn
  return result
}
```

## call的区别  
call和apply基本类似，只是传参有区别，call需要传入多个参数, bind传数组
```js
Function.prototype.call2 = function (context) {
  var context = context || window
  context.fn = this

  var args = []
  for(var i = 1, len = arguments.length; i < len; i++) {
    args.push('arguments[' + i + ']')
  }

  var result = eval('context.fn(' + args +')')

  delete context.fn
  return result
}

```
## new
> new 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象类型之一

```js
function objectFactory() {
  var obj = new Object(), Constructor = [].shift.call(arguments)
  obj.__proto__ = Constructor.prototype
  var ret = Constructor.apply(obj, arguments)
  return typeof ret === 'object' ? ret : objs
}
```
new的作用:
- 创建一个新对象
- 新对象的原型指向构造函数
- 改变this指向并执行方法
- 返回新对象

## bind
> bind()方法创建一个新的函数, 当被调用时，将其this关键字设置为提供的值，在调用新函数时，在任何提供之前提供一个给定的参数序列。

MDN的解释是：bind()方法会创建一个新函数，称为绑定函数，当调用这个绑定函数时，绑定函数会以创建它时传入 bind()方法的第一个参数作为 this，传入 bind() 方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数。
```js
Function.prototype.bind2 = function (context) {
  if (typeof this !== "function") {
    throw new Error("Function.prototype.bind - what is trying to be bound is not callable")
  }
  var self = this
  var args = Array.prototype.slice.call(arguments, 1)

  var fNOP = function () {}

  var fBound = function () {
    var bindArgs = Array.prototype.slice.call(arguments);
    return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs))
  }

  fNOP.prototype = this.prototype
  fBound.prototype = new fNOP()
  return fBound
}
```
## 总结
- apply 、 call 、bind 三者都是用来改变函数的this对象的指向的；
- apply 、 call 、bind 三者第一个参数都是this要指向的对象，也就是想指定的上下文；
- apply 、 call 、bind 三者都可以利用后续参数传参；
- bind是返回对应函数，便于稍后调用；apply、call则是立即调用。

