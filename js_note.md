[TOC]
#javascript 学习随笔
## 原型与原型属性
1. \__proto__：原型；prototype：原型属性
2. a.\__proto__ == a.constructor.prototype;

    a.\__proto__ == A.prototype (var a = new A())。
 函数A的原型属性(prototype property )是一个对象，当这个函数被用作构造函数来创建实例时，该函数的原型属性将被作为原型赋值给所有对象实例(注:即 所有实例的原型引用的是函数的原型属性)。只有函数才有该属性——prototype！
```javascript
/创建一个函数b
var b = function(){ var one; }
//使用b创建一个对象实例c
var c = new b();
//查看b 和c的构造函数
b.constructor;  // function Function() { [native code]}
b.constructor==Function.constructor; //true
c.constructor; //实例c的构造函数 即 b function(){ var one; }
c.constructor==b //true
//b是一个函数，查看b的原型如下
b.constructor.prototype // function (){}
b.__proto__  //function (){}

//b是一个函数，由于javascript没有在构造函数constructor和函数function之间做区分，所以函数像constructor一样，
//有一个原型属性，这和函数的原型(b.__proto__ 或者b.construtor.prototype)是不一样的
b.prototype //[object Object]   函数b的原型属性

b.prototype==b.constructor.prototype //fasle
b.prototype==b.__proto__  //false
b.__proto__==b.constructor.prototype //true

//c是一个由b创建的对象实例，查看c的原型如下
c.constructor.prototype //[object Object] 这是对象的原型
c.__proto__ //[object Object] 这是对象的原型

c.constructor.prototype==b.constructor.prototype;  //false  c的原型和b的原型比较
c.constructor.prototype==b.prototype;  //true c的原型和b的原型属性比较

//为函数b的原型属性添加一个属性max
b.prototype.max = 3
//实例c也有了一个属性max
c.max  //3
```
3. 原型是一个对象，每个对象都有原型(注意：当试图获取一个主数据类型的原型时，它会被先强制转化成了一个对象)
4. 一个对象的真正原型是被对象内部的**Prototype**属性(property)所持有。ECMA引入了标准对象原型访问器Object.getPrototype(object)，到目前为止只有Firefox和chrome实现了此访问器。除了IE，其他的浏览器支持非标准的访问器\__proto__，如果这两者都不起作用的，我们需要从对象的构造函数中找到的它原型属性。
5. 如果更改了构造函数的原型，是否意味着已经存在的该构造函数的实例将获得构造函数的最新版本？
不一定。如果修改的是原型属性，那么这样的改变将会发生。因为在a实际被创建之后，a.\__proto__是一个对A.prototype 的一个**引用**(如果在后面进行“A.prototype=新对象”，那么“a.\__proto__==旧对象“，因为依旧是原来的引用，即依旧指向原先的内存地址，不会指向新对象所在的内存地址)。
```javascript
var A = function(name) {
	this.name = name;
}
var a = new A(‘alpha’);
a.name; //’alpha’

A.__proto__.max = 19880716;  // a.max => undefined
A.prototype.max = 19880716; // a.max => 19880716
```
6. 如果a的原型属于A的原型链，表达式 a instance of A 值为true
7. 原型的继承机制是发生在内部且是隐式的，即无法直接看到继承的属性和方法，但却能调用的到。
## 对象
1. 定义：在javascript中，一个对象就是任何无序键值对的集合，如果它不是一个主数据类型(undefined，null，boolean，number，or string)，那它就是一个对象。
2. **localStorage**是HTML5中提供的一个以键值对形式保存信息的一个全局变量。
```javascript
// 保存信息
localStorage.setItem('key', value);
// 获取信息
localStorage.getItem('key');
```
# typescript学习随笔
1. (Array.)map函数：在map中执行完所有的元素后才继续后面的
![](/home/flyman/Pictures/map.png) 
