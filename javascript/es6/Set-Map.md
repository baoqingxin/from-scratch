# Set和Map
## set
set是新的数据结构。类似于数组，但是成员的值都是唯一的，没有重复的值。  
Set本身是一个构造函数，用来生成Set数据结构  
Set函数可以接受一个数组(或者具有iterable接口的其他数据结构)作为参数，用来初始化。
```js
const set = new Set([1, 2, 3, 4, 4])
[...set] // [1, 2, 3, 4]

const set = new Set(document.querySelectorAll('div'))
set.size
```
数组去重的方法
```js
[...new Set(array)]
```
字符串去重
```js
[...new Set('aabbacdedd')].join('')
```
向Set加入值的时候，不会发生类型转换，所以5和’5‘是两个不同的值。Set内部判断两个值是否不同，使用的算法叫做'Same-value-zero equality'，它类似于精确相等运算符(===),主要的区别是Set中NaN和NaN相等，而精确相等运算符认为不相等  
### set实例的属性和方法
Set结构的实例有以下属性。
- Set.prototype.constructor: 构造函数，默认就是Set函数
-Set.prototype.size:返回Set实例的成员总数  

Set的四个操作方法：
- set.prototype.add(value):添加某个值，返回Set结构本身
- Set.prototype.delete(value):删除某个值，返回一个布尔值，表示删除是否成功
- Set.prototype.has(value): 返回一个布尔值，表示该值是否为Set的成员。
- Set.prototype.clear(): 清除所有成员，没有返回值
Array.from方法可以将Set结构转为数组
```js
const items = new Set([1, 2, 3, 4, 5])
const array = Array.from(items)
```
数组去重的另一个方法
```js
function dedupe (array) {
  return Array.from(new Set(array))
}
dedupe([1, 1, 2, 3]) // [1, 2, 3]
```
### 遍历操作
Set结构的实例有四个遍历方法，可以用于遍历成员
- Set.prototype.keys(): 返回健名的遍历器
- Set.prototype.values(): 返回键值的遍历器
- Set.prototype.entries(): 返回键值对的遍历器
- Set.prototype.forEach(): 使用回调函数遍历每个成员
```js
let set = new Set([1, 4, 9])
set.forEach((value, key) => console.log(key + ' : ' + value))
// 1 : 1
// 4 : 4
// 9 : 9
```
### 遍历的应用
扩展运算符(...)内部使用for...of循环，所以也可以用于Set结构
数组的map和filter方法也可以间接用于Set，因此使用Set可以很容易地实现并集、交集和差集
```js
let a = new Set([1, 2, 3])
let b = new Set([4, 3, 2])
// 并集
let union = new Set([...a, ...b])
// 交集
let intersect = new Set([...a].filter(x => b.has(x)))
// 差集
let difference = new Set([...a].filter(x => !b.has(x)))
```
如果想在遍历操作中，同步改变原来的Set结构，目前没有直接的方法，但有两种变通的方法。一种是利用原Set结构映射出一个新的结构，然后赋值给原来的Set结构；另一种是利用Array.from方法
```js
let set = new Set([1, 2, 3])
set = new Set([...set].map(val => val * 2)) // 2, 4, 6

let set = new Set([1, 2, 3])
set = new Set(Array.from(set, val => val * 2)) // 2, 4, 6
```
上面代码提供了两种方法，直接在遍历操作中改变原来的Set结构
### WeakSet
WeakSet结构和Set类似，也是不重复的值的集和。但是它与Set有两个区别。
- WeakSet的成员只能是对象，而不能是其他类型的值。  
```js
const ws = new WeakSet()
ws.add(1) // Uncaught TypeError: Invalid value used in weak set
ws.add(Symbol()) // Uncaught TypeError: Invalid value used in weak set
```
- WeakSet中的对象都是弱引用，即垃圾回收机制不考虑WeakSet对该对象的引用，也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑对象还存在于WeakSet中。  
这是因为垃圾回收机制依赖引用计数，如果一个值的引用次数不为0，垃圾回收机制就不会释放这块内存。结束使用该值之后，有时会忘记取消引用，导致内存无法释放，进而可能会引发内存泄漏。WeakSet里面的引用，都不计入垃圾回收机制，所以就不存在这个问题。因此WeakSet适合临时存放一组对象，以及存放跟对象绑定的信息。只要这些对象在外部消失，它在WeakSet里面的引用就会自动消失。  
由于上面这个特点，WeakSet的成员不适合引用的，因为它会随时消失。另外，由于WeakSet内部有多少个成员，取决于垃圾回收机制有没有运行，运行前后很可能成员个数是不一样的，而垃圾回收机制何时运行时不可预测的，因此ES6规定WeakSet不可遍历。WeakMap也是如此。
语法
WeakSet是一个构造函数，可以使用new命令，创建WeakSet数据结构。
```js
const ws = new WeakSet()
```
作为构造函数，WeakSet可以接受一个数组或类似数组的对象作为参数。(实际上，任何具有Iterable接口的对象，都可以作为WeakSet的参数)该数组的所有成员，都会自动成为WeakSet实例对象的成员。  
```js
const a = [[1, 2], [3, 4]]
const ws = new WeakSet(a) // WeakSet {[1, 2], [3, 4]}
```
WeakSet结构有以下三个方法：
- WeakSet.prototype.add(value): 向WeakSet实例添加一个新成员
- WeakSet.prototype.delete(value): 清除WeakSet实例的指定成员
- WeakSet.prototype.has(value): 返回一个布尔值，表示某个值是否在WeakSet实例中。   
```js
const ws = new WeakSet()
const obj = {}
const foo = {}

ws.add(window)
ws.add(obj)

ws.has(window) // true
ws.has(foo) // false

ws.delete(window)
ws.has(window) // false
```
WeakSet没有size属性，没有办法遍历它的成员  
WeakSet不能遍历，是因为成员都是弱引用，随时可能消失，遍历机制无法保证成员的存在，很可能刚刚遍历结束成员就取不到了。WeakSet的一个用处，是储存DOM节点，而不用担心这些节点从文档移除时，会引发内存泄漏。  
```js
const foos = new WeakSet()
class Foo {
  constructor () {
    foos.add(this)
  }
  method () {
    if (!foos.has(this)) {
      throw new TypeError('Foo.prototype.method只能在Foo的实例上调用！')
    }
  }
}
```
上面代码保证了Foo的实例方法，只能在Foo的实例上调用。这里使用 WeakSet 的好处是，foos对实例的引用，不会被计入内存回收机制，所以删除实例的时候，不用考虑foos，也不会出现内存泄漏。
## Map
JS的对象，本质上是键值对的集和（Hash结构），但是传统上只能用字符串当做键。这给它的使用带来了很大的限制。
```js
const data = {}
const element = document.getElementById('myDiv')

data[element] = 'metadata'
data['[object HTMLDivElement]'] // ‘metadata’
```
上面代码原意是将一个 DOM 节点作为对象data的键，但是由于对象只接受字符串作为键名，所以element被自动转为字符串[object HTMLDivElement]。

为了解决这个问题。Map数据结构出现了。它类似于对象，也是键值对的集和，但是“键”的范围不限于字符串，各种类型的值(包括对象)都可以当做键。也就是说，Object 结构提供了“字符串—值”的对应，Map 结构提供了“值—值”的对应，是一种更完善的 Hash 结构实现。如果你需要“键值对”的数据结构，Map 比 Object 更合适。

```js
const m = new Map()
const o = { p: 'Hello world' }

m.set(o, 'content')
m.get(o) // 'content'

m.has(o) // true
m.delete(o) // true
m.has(o) // false
```
上面的例子展示了如何向 Map 添加成员。作为构造函数，Map 也可以接受一个数组作为参数。该数组的成员是一个个表示键值对的数组。
```js
const map = new Map([
  ['name', '张三'],
  ['title', 'Author']
])
map.size // 2
map.has('name') // true
map.get('name') // "张三"
map.has('title') // true
map.get('title') // "Author"
```
上面代码在新建Map实例时，就指定了两个键name和title  
Map构造函数接受数组作为参数，实际上执行的是下面的算法。
```js
const items = [
  ['name', '张三'],
  ['title', 'Author']
]

const map = new Map();

items.forEach(
  ([key, value]) => map.set(key, value)
)
```
事实上，不仅仅是数组，任何具有Iterator接口、且每个成员都是一个双元素的数组的数据结构都可以当做Map构造函数的参数。这就是说，Set和Map都可以用来生成新的Map。
```js
const set = new Set([
  ['foo', 1],
  ['bar', 3]
])
const m1 = new Map(set)
m1.get('foo') // 1

const m2 = new Map([['baz', 3]])
const m3 = new Map(m2)
m3.get('baz') // 3
```
在上面代码中，我们分别使用Set和Map对象，当做Map构造函数的参数，结果都新生成了新的Map对象。  

注意，只有对同一个对象的引用，Map结构才将其视为同一个键。这一点要非常小心。  
```js
const map = new Map()
map.set(['a'], 555)
map.get(['a']) // undefined
```
上面代码的set和get方法，表面上是针对同一个键，但实际上这是两个不同的数组实例，内存地址是不一样的，因此get方法无法读取该键，返回undefined  
同理，同样的值的两个实例，在Map结构中被视为两个键。  
```js
const map = new Map()
const k1 = ['a']
const k2 = ['a']
map.set(k1, 111).set(k2, 222)
map.get(k1) // 111
map.get(k2) // 222
```
上面代码中，变量k1和k2的值是一样的，但是他们在Map中被视为两个键。  
由上可知，Map的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。这就解决了同名属性碰撞的问题。  
如果Map的键是一个简单类型的值(数字，字符串，布尔值)，则只要两个值严格相等，Map将其视为一个键，比如0和-0就是一个键。另外undefined和null是两个不同的键。虽然NaN不严格相等于自身，但Map将其视为同一个键。  
```js
let map = new Map()

map.set(-0, 123)
map.get(+0) // 123

map.set(true, 1)
map.set('true', 2)
map.get(true) // 1

map.set(undefined, 3);
map.set(null, 4);
map.get(undefined) // 3

map.set(NaN, 123);
map.get(NaN) // 123
```
### 实例的属性和操作方法
- size属性：返回Map结构的成员总数
- Map.prototype.set(key, value)
- Map.prototype.get(key)
- Map.prototype.has(key)
- Map.prototype.delete(key)
- Map.prototype.clear()

### 遍历方法
- Map.prototype.keys()：返回键名的遍历器。
- Map.prototype.values()：返回键值的遍历器。
- Map.prototype.entries()：返回所有成员的遍历器。
- Map.prototype.forEach()：遍历 Map 的所有成员。  
需要注意的是，Map的遍历顺序就是插入顺序。  
Map结构转为数组结构，比较快速的方法是使用扩展运算符(...)  
forEach方法还可以接受第二个参数，用来绑定this
```js
const reporter = {
  report: function (key, value) {
    console.log("Key: %s, Value: %s", key, value)
  }
}
map.forEach(function(value, key, map) {
  this.report(key, value)
}, reporter)
```
上面代码中，forEach方法的回调函数的this，就指向reporter。
### 与其他数据结构的互相转换
- Map转为数组： (...)
- 数组转为Map： 将数组传入 Map 构造函数，就可以转为 Map。
- Map转为对象：  
如果所有Map的键都是字符串，他可以无损地转为对象。  
```js
function strMapToObj (strMap) {
  let obj = Object.create(null)
  for (let [k, v] of strMap) {
    obj[k] = v
  }
  return obj
}

const myMap = new Map()
  .set('yes', true)
  .set('no', false)
strMapToObj(myMap)
```
- 对象转为Map： Object.entries()
```js
let obj = {a: 1, b: 2}
let map = new Map(Object.entries(obj))
```
- Map转JSON
分为两种情况：一种情况是Map的健名都是字符串，这时可以选择转为对象JSON  
```js
function strMapToJson (strMap) {
  return JSON.stringify(strMapToObj(strMap))
}
let myMap = new Map().set('yes', true).set('no', false)
strMapToJson(myMap)
```
另一种情况是，Map的健名有非字符串，这时可以选择转为数组JSON。
```js
function mapToArrayJson(map) {
  return JSON.stringify([...map])
}
let myMap = new Map().set(true, 7).set({foo: 3}, ['abc'])
mapToArrayJson(myMap)
// '[[true, 7], [['foo': 3, ['abc']]]]'
```
- JSON转为Map
JSON转为Map，正常情况下，所有健名都是字符串。
```js
function jsonToStrMap (jsonStr) {
  return objToStrMap(JSON.parse(jsonStr))
}
jsonToStrMap('{"yes": true, "no": false}')
// Map {'yes' => true, 'no' => false}
```
但是，有一种特殊情况，整个JSON就是一个数组，且每个数组成员本身又是一个有两个成员的数组，。这时，它可以一一对应的转为Map。这往往是Map转为数组JSON的逆操作。  
```js
function jsonToMap (jsonStr) {
  return new Map(JSON.parse(jsonStr))
}

jsonToMap('[[true,7],[{"foo":3},["abc"]]]')
// Map {true => 7, Object {foo: 3} => ['abc']}
```
## WeakMap
WeakMap结构与Map结构类似，也是用于生成键值对的集合。  
```js
// WeakMap可以使用set方法添加成员
const wm1 = new WeakMap()
const key = {foo: 1}
wm1.set(key, 2)
wm1.get(key) // 2

// WeakMap也可以接受一个数组
// 作为构造函数的参数
const k1 = [1, 2, 3]
const k2 = [4, 5, 6]
const wm2 = new WeakMap([[k1, 'foo']], [[k2, 'bar']])
wm2.get(k2) // 'bar'
```
WeakMap与Map的区别有两点
- 首先，WeakMap只接受对象作为键名(null除外), 不接受其他类型的值作为键名  
```js
const map = new WeakMap()
map.set(1, 2) // TypeError: 1 is not an object!
map.set(Symbol(), 2) // TypeError: Invalid value used as weak map key
map.set(null, 2) // TypeError: Invalid value used as weak map key
```
- 其次，WeakMap的键名所指向的对象，不计入垃圾回收机制。  
WeakMap的设计目的在于，有时我们想在某个对象上面存放一些数据，但是这回形成对于这个对象的引用。
```js
const e1 = document.getElementById('foo')
const e2 = document.getElementById('bar')
const arr = [
  [e1, 'foo 元素']，
  [e2, 'bar 元素']
]
```
上面代码，e1和e2是两个对象，我们通过arr数组对这两个对象添加一些文字说明。这就形成了arr对e1和e2的引用。  
一旦不需要这两个对象，我们就必须手动删除这个引用，否则垃圾回收机制就不会释放e1和e2占用的内存。  
```js
// 不需要e1和e2的时候
// 必须手动删除引用
arr[0] = null
arr[1] = null
```
上面这样的写法显然很不方便。一旦忘了写，就会造成内存泄露。  
WeakMap 就是为了解决这个问题而诞生的，它的键名所引用的对象都是弱引用，即垃圾回收机制不将该引用考虑在内。因此，只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。也就是说，一旦不再需要，WeakMap 里面的键名对象和所对应的键值对会自动消失，不用手动删除引用。  

基本上，如果你要往对象上添加数据，又不想干扰垃圾回收机制，就可以使用WeakMap。一个典型的应用场景是，在网页的Dom元素上添加数据，就可以使用WeakMap结构。当该DOM元素被清除，其所对应的WeakMap记录就会自动被移除。  
```js
const wm = new WeakMap()
const element = document.getElementById('example')
wm.set(element, 'some information')
wm.get(element) // some information
```
总之，WeakMap的专用场合就是，它的键所对应的对象，可能会在将来消失。WeakMap结构有助于防止内存泄漏。  
**注意，WeakMap弱引用的只是键名，而不是键值。键值依然是正常引用。**
```js
const wm = new WeakMap()
let key = {}
let obj = {foo: 1}
wm.set(key, obj)
obj = null
wm.get(key)
// Object {foo: 1}
```
上面代码中，键值obj是正常引用。所以，即使在 WeakMap 外部消除了obj的引用，WeakMap 内部的引用依然存在。
### 用法
WeakMap 与 Map 在 API 上的区别主要是两个：
- 一是没有遍历操作（即没有keys()、values()和entries()方法），也没有size属性。
- 二是无法清空，即不支持clear方法。
WeakMap只有四个方法可用：get()、set()、has()、delete()。
### WeakMap的用途：
前文说，WeakMap的应用典型场合就是DOM节点作为键名。
```js
let myWeakMap = new WeakMap()

myWeakMap.set(
  document.getElementById('logo'),
  {timesClicked: 0}
)
document.getElementById('logo').addEventListener('click', function () {
  let logoData = myWeakMap.get(document.getElementById('logo'))
  logoData.timesClicked++
}, false)
```
WeakMap的另一个用处是部署私有属性。
```js
const _counter = new WeakMap()
const _action = new WeakMap()

class Countdown {
  constructor (counter, action) {
    _counter.set(this, counter)
    _action.set(this, action)
  }
  dec () {
    let counter = _counter.get(this)
    if (counter < 1) return
    counter--
    _counter.set(this, counter)
    if (counter === 0) {
      _action.get(this)()
    }
  }
}

const c = new Countdown(2, () => console.log('DONE'))
c.dec()
c.dec()
// Done
```
上面代码中，Countdown类的两个内部属性_counter和_action，是实例的弱引用，所以如果删除实例，它们也就随之消失，不会造成内存泄漏。
