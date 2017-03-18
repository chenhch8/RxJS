[TOC]
## RxJS操作方法记录——operators: 用于transform 或者 query 序列的方法

### .debounce()

### Rx.Observable.create()
1. 用处：用于创建一个Observable
2. 参数：callback——callback that accepts an Observer as a parameter
3. `Observers` 和 `Observables`关系：
	* **Observers listen to Observables. Whenever an event happens in an Observable, it calls the realted method in all of its Observers.**
	* **Observables don't do anything until at least one Observer subscibes to them**
4. 例子：
```javascript
var observable = Rx.Observable.create(observer => {
	observer.onNext('Simon');
	observer.onNext('Jen');
	observer.onNext('Sergi');
	observer.onCompleted();
})
```

### Rx.Observer.create()
1. 用处：用于创建一个Observer
2. 参数：onNext, onCompleted, onError三个回调函数
参数用处说明：
	* **onNext**: 当Observables 发射一个新值时，会调用该函数
	* **onCompleted**: Signals that there is no more data available. After onCompleted is called, further calls to onNext will have no effect.
	* **onError**: 当Observable发生错误时调用。在该函数被调用后，以后的onNext将不再生效
3. 例子：
```javascript
var observer = Rx.Observer.create(
	function onNext(x) {...},
	function onError(err) {...},
	function onCompleted() {...}
)
```

### HTTP get 例子
```javascript
function get(url) {
	return Rx.Observable.create(observer => {
		var req = new XMLHttpRequest();
		req.open('GET', url);
		req.onload = _ => {
			if (req.status == 200) {
				observer.onNext(req.reponse);
				observer.onCompleted();
			} else {
				observer.onError(new Error(req.statusText));
			}
		};
		req.onerror = _ => {
			observer.onError(new Error('Unknown Error'));
		};
		req.send();
	});
}
var test = get('/api/contents.json');
test.subscribe( // Observables don't do anything until at least one Observer subscibes to them
	function onNext(x) { console.log(x); },
	function onError(err) { console.log(err); },
	function onCompleted() { console.log('Completed'); }
)

// 上述代码等价于

Rx.DOM.get('/api/contents.json').subscribe( // 用到了 RxJS DOM 库——用于创建与DOM关联的Observables
	function onNext(x) {},
	function onError(err) {},
	function Completed() {}
)
```

### .from()
1. 用处：将普通数据变换到Observable域。其中普通数据集有：
	* Array: 数组的每个成员对应序列的一个元素
	* String:　字符串的每个字符对应序列的一个元素
	* Set:　Set的每个成员对应序列的一个元素
	* Map: 类似Json对象，但按照插入次序进行遍历。转换后，每个键值对，对应序列的一个元素
2. 例子：
```javascript
Rx.Observable
	.from(['Adria', 'Jen', 'Sergi'])
	.subscribe(...)
```

### .fromEvent()
1. 用处：将event转化为Observable
2. 参数：第一个为产生事件的对象，第二个为事件名
3. 例子：
```javascript
var allMoves = Rx.Observable.fromEvent(document, 'mousemove')
allMoves.subscribe(e => {
	console.log(e.clientX, e.clientY);
})
```

### .fromCallback() / .fromNodeCallback()
1. 用处：将回调函数转化为Observable。
2. **将回调函数的整个结果作为序列的一个元素**
3. 返回值：一个Observable，该Observable只有一个元素，即为回调函数的返回结果

#### .fromNodeCallback()
1. 用处：将具有Nodejs特色的回调函数——即回调函数的第一个参数总是error，用于指明是否发送错误——转化为Observable。
2. 参数：两个参数，第一个参数是回调函数，第二个参数为可选参数，用于指定回调函数中的this的上下文
3. 例子：
```javascript
const Rx = require('rx');
const mysql = require('mysql');
const dbConfig = {...}
var conn = mysql.createConnection(dbConfig);
conn.connect();

query = Rx.Observable.fromNodeCallback(conn.query, conn);

query('select * from course')
  .flatMap(results => {
    return Rx.Observable.from(results[0])
  })
  .flatMap(item => {
    let id = item.course_id;
    let semester = id % 3 ? '秋季学期' : '春季学期';
    let school_year = id % 2 ? '2016-2017' : '2015-2016';
    console.log(id, semester, school_year);
    return query(`
      update course
      set school_year=?, semester=?
      where course_id = ?;
    `, [school_year, semester, id]);
  })
  .subscribe(
    res => {},
    err => {
        console.log('Error:', err);
        process.exit(0);
    },
    _ => {
        console.log('Completed');
        process.exit(0);
    }
  );
```

### .range()
1. 用处：用于生成一个有限序列
2. 参数：两个必需参数——第一个参数指定序列元素的起始值，第二个指定元素数量
3. 返回值：一个Observable
4. 例子：
```javascript
var source = Rx.Observable.range(2, 3) // 序列：2 3 4
```

### .merge()
1. 用处：把**两个**序列的元素并如到源序列中，返回的不是一个全新的序列，而是被修改后的源序列（concat()返回的是一个全新的序列）
2. 参数：一个Observables参数
3. 返回值：一个Observable
4. 例子：
```javascript 
ar source1 = Rx.Observable.of(1, 2, 3)
	.flatMap(function(d, i) { return Rx.Observable.of(d).delay(i * 1000); })
var source2 = Rx.Observable.of(4, 5, 6)
	.flatMap(function(d, i) { return Rx.Observable.of(d).delay(i * 1200); })
var target = source1.merge(source2) // 序列：1 4 2 5 3 6
```

### .mergeAll()
1. 用处：对于**序列的序列**，用该方法进行融合。无参数
2. 例子：
```javascript
var source = Rx.Observable.interval(1000).take(5).map(item => Rx.Observale.of(item)); // 序列： Observable(0) Observable(1) ...
var target = source.mergeAll(); // 序列：0 1 2 3 ...
```

### .concat()
1. 用处：将**多个**Observable拼接起来
2. 例子：
```javascript
let a = Rx.Observable.from([1, 2])
let b = Rx.Observable.from([3, 4])
let c = Rx.Observable.from([5, 6])
a.concat(b, c)
	.subscribe(result => console.log(result)) // [1,2,3,4,5,6]
```

### .interval()
1. 用处： 创建一个无限长度的周期性_可观测序列/Observable_。
2. 参数：该方法有一个参数，用于声明了以毫秒为单位的周期。序列的元素值从0开始，每个周期比前一个周期加1。注意，第一个值的生成不是立刻的，依旧需要等待一个_静默周期_，该周期即与指定的参数周期一致。
3. 返回值：一个Observable
4. 例子：
```javascript
Rx.Observable.interval(1000) // 输出：0 1 2 3 ...
```

### .**map()
1. 用处：对源序列的元素进行变换

#### .map()
1. 用处：对序列成员进行变换，并返回一个新的序列。
2. 参数：一个变化函数，函数的返回值就是目标序列中对应的元素
3. 返回值：一个Observable
4. 别名：select()
5. 例子：
```javascript
//item - 值
//index - 索引 
var tfx = function(item,index){
    return item * 2;
};
var source = Rx.Observable.of(1,2,3); //输出： 1 2 3
var target = source.map(tfx);   //输出： 2 4 6
```

#### .flatMap()
1. 用处：首先将一个序列的**各元素**映射为新的序列，然后将映射成的各元素序列**融合**形成一个新的序列
2. 参数：一个变换函数，函数的返回值为一个序列
3. 返回值：一个新序列 / Observable
4. 例子：
```javascript
var source = Rx.Observable.of(10,20,3) // 序列：10 20 30
var mf = function(item) {
	return Rx.Observable.range(item. 3);
}
var target = source.flatMap(mf); // 序列：10 11 12 20 21 22 30 31 32
```

#### .flatMapLatest()
1. 用处：与flatMap()的区别在于，仅将最新激活的元素序列中的元素，输出到目标序列中。注意，其实其内部有它自己定义的一小段时间，从该段时间中收集到的所有收到的值，选择最新的那个返回，就是其实现思路。
2. 参数：一个变换函数，函数的返回值为一个序列。该变换函数的参数，即为最新激活的元素序列中的元素
3. 返回值：一个新序列 / Observable
4. 例子：
```javascript
var source = Rx.Observable.range(0, 5) // 序列：0 1 2 3 4
var target = source.flatMapLatest(item => { console.log('输出：', item); return item}) // 输出：4——说明变换函数的参数，即为最新激活的元素序列中的元素（序列：4）
```

#### .flatMapObserver()
1. 用处：为源序列的每一个元素、错误通知、结束通知都创建一个序列。
2. 参数：有三个参数，分别对应onNext、onError、onComplete三个函数，每个函数都返回一个序列。
3. 例子：
```javascript
var source = Rx.Observable.of(1,2,3);
var nf = function(d,i){ return Rx.Observable.of(d).delay(i*1000);};
var ef = function(err){ return Rx.Observable.of("error");};
var cf = function(){ return Rx.Observable.of("completed");}
var target = source.flatMapObserver(nf,ef,cf); //序列：1 2 3 completed
```

#### .concatMap()
1. 用处： 将源序列的各元素映射为序列，然后按顺序拼接起来。与flatMap()效果类似
2. 参数：一个参数，为映射函数，其返回值是序列
3. 返回值：一个Observable
4. 例子：
```javascript
var source = Rx.Observable.of(1, 2, 3); // 序列：1 2 3
var target = source.concatMap(item => R.Observable.range(item, 3)) // 序列：1 2 3 2 3 4 3 4 5
```

#### .concatMapObserver()
1. 用处：将源序列的元素、错误通知、结束通知都转化为序列，并按照顺序拼接为目标序列，通过subscribe的value回调，即可获得拼接后的目标序列。
2. 参数：三个参数，均为映射函数，分别对应onNext()、onError()、onComplete()，返回值均为序列
3. 返回值：一个Observable
4. 例子：
```javascript
var source = Rx.Observable.of(1,2,3)
var nf = function(d,i) {return Rx.Observable.of(d).delay(i * 1000);}
var ef = function(err) {return Rx.Observable.of("err");}
var cf = function() {return Rx.Observable.of("completed");}
var target = source.concatMapObserver(nf,ef,cf) // 序列：1 2 3 completed
```

### filter()
1. 用法：用于筛选源序列中满足条件的元素，并返回新的序列。
2. 参数：两个参数。第一个参数是一个断言(Predicate)函数，只有当该函数返回true时，当前元素才会加入到目标序列中。第二个参数b是可选的，被用作断言函数的this对象。
3. 返回值：一个Observable
4. 别名：where()
5. 例子：
```javascript
var source = Rx.Observable.of(1,2,3,4,5) // 序列：1 2 3 4 5
var target = source.filter((d, i) => d < 4); // 序列：1 2 3
var source = Rx.Observable.of("who","hello","max");
var context = { pattern: /^h/ }
var target = source.filter((d,i) => d.match(this.pattern), context) // 序列：hello
```

### reduce()
1. 用处：将源序列中的所有元素规约为一个元素。
2. 参数：两个参数，第一个参数是一个规约函数，第二个参数是一个初始值。当序列结束时，reduce才调用noNext将结果传给后面。
3. 返回值：一个只包含一个元素的Observable
4. 例子：
```javascript
var source = Rx.Observable.of(1,2,3)
var rf = function(acc, item) {return acc　+ item;}
var acc_init = 0;
var target = source.reduce(rf, acc_init); // 序列：6
```

### .scan()
1. 与reduce()一样，但第一个参数是初始值，第二个是变化函数，与reduce()相反

### 序列的取消

#### 显示取消——.dispose()
1. 用处：每次订阅subscribe后会返回一个Disposable object，它代表该订阅。该对象有一个dispose()方法，能够用于停止接受来自Observable的通知
2. 显示取消
3. 例子：
```javascript
var source = Rx.Observable.interval(1000)
var sub1 = source.subscribe(
	i => console.log('sub1:', i)
)
setTimeout(_ => {
	sub1.dispose();
}， 1000);
```

#### 隐示取消
1. 像 `range()`和`take()`等方法，在序列结束后或者操作条件满足后，都会自动取消订阅。类似的还有`withLatestFrom()`和`flatMapLatest()`等等。

###异常的捕获
1. 有两种实现方法，一种是通过catch()，另一种是在订阅函数中注册onError()函数。注意，使用了catch()后，onError()就会失效。
2. catch(): 参数是一个Observable或者是一个函数，该函数的参数是error对象。catch()的返回值是另一个Observable。使用该方法的好处是，Observable的执行不会因error而被终止，它类似与try catch。
3. 例子：
```javascript
function getJson(arr) {
	return Rx.Observable.from(arr).map(function(str) {
		return JSON.parse(str);
	})
}
var caught = getJson(['{ "1": 1, "2": 2 }', '{ "1: 1, "2": 2}'])
	.catch(Rx.Observable.return(
		error: 'There was an error parsing JSON'
	))
caught.subscribe(
	function(json) {
		console.log('Parse Json: ', json);
	},
	// 由于前面catch()，故该'onError'无效
	function(e) {
		console.log('Error: ', e.message)
	} // 输出： Parsed_JSON: Object { error: "There was an error parsing JSON" }
)
```

### retry()
1. 用处：错误时重新订阅。当源序列发生错误时，将重新执行订阅。**注意，是重启，即从头再来**。
2. 参数：**可选参数**a用于指定订阅的次数限制，若错误次数超过此阈值，该方法将会抛出错误而结束。若无a，则持续地重复订阅。
3. 使用例子：
```javascript
Rx.DOM.get(url).retry(5)
	.subcribe(...)
```

### distinct()
1. 用处：抑制序列中的重复值，并返回一个新的序列
2. 参数：有两个可选参数，若源序列的元素不是基本类型，那么第一个参数需传入一个键选择函数；若需要改变默认的等价规则，那么第二个参数需传如一个比较函数，返回值为true时，则认为是相同的元素。
3. 返回值：一个Observable
4. 例子：
```javascript
var source = Rx.Observable.of(1,2,3,4,2,1); //序列： 1 2 3 4 2 1
var target = source.distinct(); //序列：1 2 3 4
	 
 var data = [
     {id:111,name:"Elton John"},
     {id:222,name:"Mary"},
     {id:333,name:"Linda"},
     {id:111,name:"Mr. John"}
 ];
var source = Rx.Observable.from(data);
var ksf = function(d){ return d.id; }; //使用id作为元素的键
var target = source.distinct(ksf); //序列： {id:111..} {id:222...} {id:333...}
	 
var source = Rx.Observable.from("aAwWDd"); //序列：a A w W D d
var kcf = function(d1,d2){ return d1.toLowerCase() === d2.toLowerCase(); };
var target = source.distinct(null,kcf);  //序列： a w D
```

### distinctUntilChanged()
1. 用处：去掉**连续**的相同的元素，相同的但不连续的元素则保留
2. 参数：有两个可选参数，用法与distinct()一样。
3. 返回值：一个Observable
4. 例子：
```javascript
var source = Rx.Observable.of(1,2,2,4,2,1); //序列： 1 2 2 4 2 1
var target = source.distinctUntilChanged(); //序列：1 2 4 2 1
```

### RxJS's Subject Class

#### Subject
1. 说明：A Subject is a type that implements both **Observable** and **Observer** types. As an Observer, it can subscribe to Observables, and as an Obervable it can produce values and have Observers subscribe to it. In some scenarios a single Subject can do the work of a combination of Observers and Observables.
2. 例子：
```javascript
  const Rx = require('rx');
  var subject = new Rx.Subject();
  var source = Rx.Observable.interval(300)
    .map(i => 'Interval message #' + i)
    .take(5);
  source.subscribe(subject);
  subject.subscribe(
    res => console.log('onNext:' + res),
    err => console.log('onError: ' + err),
    _ => console.log('onComplete')
  );
  subject.onNext('Our message #1');
  subject.onNext('Our message #2');
  setTimeout(_ => {
    subject.onCompleted();
    process.exit(0);
  }, 1000);
 // 输出：
 // onNext:Our message #1
 // onNext:Our message #2
 // onNext:Interval message #0
 // onNext:Interval message #1
 // onNext:Interval message #2
 // onComplete
```
3. 个人理解：从例子中，能看出subject就像一个中间桥梁。

#### AsyncSubject
1. **AsyncSubject** emits the last value of a sequence only if the sequence completes. This value is then cached forever, and any Observerthat subscribes after the value has been emitted will receive it right away.
2. 例子：
```javascript
  const Rx = require('rx');
  var subject = new Rx.AsyncSubject();
  var delayedRange = Rx.Observable.range(0, 5).delay(1000);

  delayedRange.subscribe(subject);

  subject.subscribe(
    res => console.log(res),
    err => console.log('Error:', err),
    _ => { console.log('Completed'); process.exit(0); }
  )
  // 输出：
  // 4
  // Completed
 //=================================//
 function getBooksInfo(url) {
    var subject;
    return Rx.Observable.create(observer => {
      if (!subject) {
        subject = new Rx.AsyncSubject();
        request(url).subscribe(subject);
      }
      subject.subscribe(observer);
    })
  }
  getBooksInfo(URL).subscribe(
    res => console.log(res),
    err => console.log('Error:', err),
    _ => { console.log('Completed'); }
  )
```

#### BehaviorSubject
1. 用处：当Observer订阅BehaviorSubject时，它接收最后一个发射值，然后接收所有后续值。BehaviorSubject要求我们提供一个起始值，以便所有观察者在订阅一个BehaviorSubject时总是收到一个值。
2. 经过测试发现，当没有订阅者订阅BehaviorSubject时，它就直接输出其初始值后直接Completed了，如下的[1]；但当有定订阅者订阅它时，它会先输出其初始值，然后处于等待状态，等待订阅者发射值给它，即其寿命就不是由自己决定了，而是由订阅者决定，如下面的[2]：
```javascript
// [1]
  const Rx = require('rx');
  var subject = new Rx.BehaviorSubject('waiting...');
  subject.subscribe(
    res => console.log(res),
    err => { console.log('Error:', err); },
    _ => { console.log('Completed'); process.exit(0); }
  )
  // 输出：
  // waiting Completed
  
// [2]
  var subject = new Rx.BehaviorSubject('waiting...');
  subject.subscribe(
    res => console.log(res),
    err => { console.log('Error:', err); },
    _ => { console.log('Completed'); process.exit(0); }
  )
  Rx.Observable.of(1, 2, 3, 4, 5)
    .delay(10000)
    .subscribe(subject);
  // 输出：
  // waiting...
  // 1
  // 2
  // 3
  // 4
  // 5
  // Completed
```

#### ReplaySubject()
1. A ReplaySubject **caches** its values and **re-emits** them to **any** Observer that subscribes late to it.
2. 无需为该类订阅onCompleted。
3. 参数：有两个可选参数。第一个参数指定缓存的元素数量，指定该值为a后，它就只会缓存最新的a个值；第二个值表示缓存最近b毫秒中的数据
4. 例子：
```javascript
// [1] 从例子中的输出结果能看出，ReplaySubject是有将历史值缓存起来的，而Subject没有
  console.log('subject test');
  var subject = new Rx.Subject();
  subject.onNext(1);
  subject.onNext(2);
  subject.subscribe(
    res => console.log(res)
  );
  subject.onNext(3);
  subject.onNext(4);

  console.log('replaySubject test');
  var replaySubject = new Rx.ReplaySubject();
  replaySubject.onNext(1);
  replaySubject.onNext(2);
  replaySubject.subscribe(
    res => console.log(res)
  )
  replaySubject.onNext(3);
  replaySubject.onNext(4);
  // 输出：
  // subject test
  // 3
  // 4
  // replaySubject test
  // 1
  // 2
  // 3
  // 4 
  
 // [2] 第二个值表示缓存最近b毫秒中的数据
    var replaySubject = new Rx.ReplaySubject(null, 200); // Buffer size of 200ms
  setTimeout(_ => replaySubject.onNext(1), 100);
  setTimeout(_ => replaySubject.onNext(2), 200);
  setTimeout(_ => replaySubject.onNext(3), 300);
  setTimeout(_ => {
    replaySubject.subscribe(
      n => console.log(n)
    );
    replaySubject.onNext(4);
  }, 350);
  // 输出：
  // 2
  // 3
  // 4 
```

### sample()
1. 用处：采样函数，返回在一个采样时间中的最新值，最新值之前的值全都舍弃
2. 参数：一个参数，用于指定采样的时间间隔
3. 返回值：Observable
4. 例子：
```javascript

```

### pluck()
1. 用处：若源序列的元素值是JSON对象，那么可使用该方法选择某个具体属性值生成新的序列
2. 参数：一个参数，类型为string，用于指定要选择属性的键名
3. 返回值：一个Observable
4. 例子：
```javascript
var source = Rx.Observable.of({name:"John",age:22},{name:"Linda",age:21});
var target = source.pluck("name"); //序列： John Linda
```

### Rx.DOM

#### Rx.DOM.jsonpRequest()
1. 用处：用于返回数据为json格式的请求，返回结果是一个元素为经过解析后的json对象的Observable
2. 参数：Object
3. 注意，Rx.DOM是一个第三方库，需安装**'rx-dom'**。详情见: https://github.com/Reactive-Extensions/RxJS-DOM
4. 例子：
```javascript
Rx.DOM.jsonpRequest({
	url: URL,
	jsonpCallback: '函数名'
})
...
```

#### Rx.DOM.ready()
1. 用于在所有DOM元素加载完后触发DOMContentLoaded事件，从而执行指定的逻辑代码
2. 例子：
```javascript
  function initialize() { ... }
  Rx.DOM.ready().subscribe(initialize);
```

#### Rx.helpers.identity
1. 用处：是一个函数，给定参数x，返回参数x
2. 例子：
```javascript
Rx.helpers.identity(true) // 返回true
```

#### Rx.DOM.mouseover()
1. 用处：元素的鼠标悬浮在元素上的事件
2. 参数：指定具体的元素
3. 返回值：一个Observable
4. 例子：
```javascript
Rx.DOM.mouseover(ele)
```
#### Rx.DOM.mouseout()

#### Rx.DOM.click()

#### Rx.DOM.fromWebSocket()
1. 用法：创建一个websocket的Observable。它是对WebSocket的进一步封装
2. 参数：string，用于指定连接的服务端
3. 返回值：一个Observable，其元素是一个socket
4. 例子：
```javascript
// create a socket Observable but not connect yet
var socket = Rx.DOM.fromWebSocket('ws://127.0.0.1:8080');
// start to connect to server and receive message from server
socket.subscribe(function(msg) {
   console.log('socket', JSON.parse(message.data));
})
// send data to server
socket.onNext(...)
```

### Hot and Cold Observables
1. “Hot” Observables emit values **regardless of** Observers being subscribed to them. On the other hand, “cold” Observables emit the **entire sequence of values from the start** to every Observer.
2. hot observables: 无论是否有观察者订阅它，都会emit values。观察者只能接受到从它开始订阅时开始的Observable emit的值，之前的值是无法得知的；
     cold observables: 只有观察者订阅它时，它才会emit values，且每个观察者获取的值都是**从起始值开始**的而不是开始订阅的那个时刻的值。
3. 对于cold observables而言，每个订阅它的观察者获得的是序列的**拷贝**，**无法共享值**；而若想要共享值，就需要hot Observable。
4. cold observable 之所以对每个观察者从起始值开始发送，其实是因为对每个观察者，**cold observable是从头再跑一遍的**！

#### publish
1. 用处：$cold Observable \rightarrow hot Observable$
2. 注意：A *published* Observable is actually a *ConnectableObservable*, which has an extra method called **connet** that we **call to start receiving values**. This allows us to subscribe to it before it starts running
3. 例子：
```javascript
// create a cold Observable
  var source = Rx.Observable.interval(1000);
  var publisher = source.publish();

  // even if we are subscribing, no values are pushed yet
  var observer1 = publisher.subscribe(x => console.log('Observer 1:', x));

  // publisher connects and starts publishing values
  publisher.connect();

  setTimeout(_ => {
    var observer2 = publisher.subscribe(x => console.log('Observer2:', x))
  }, 5000);
  // 输出：
  // Observer 1: 0
  // Observer 1: 1
  // Observer 1: 2
  // Observer 1: 3
  // Observer 1: 4
  // Observer2: 4
  // Observer 1: 5
  // Observer2: 5
```

#### share()
1. 用处与publish()一样，但区别是publish()需要connect()来启动，而share()不需要
2. 例子：
```javascript
  // create a cold Observable
  var source = Rx.Observable.interval(1000).share();
  // start
  var observer1 =source.subscribe(x => console.log('Observer 1:', x));
  setTimeout(_ => {
    var observer2 = source.subscribe(x => console.log('Observer2:', x))
  }, 5000);
  // 输出：
  // Observer 1: 0
  // Observer 1: 1
  // Observer 1: 2
  // Observer 1: 3
  // Observer 1: 4
  // Observer2: 4
  // Observer 1: 5
  // Observer2: 5
```

### 缓存

#### buffer()
1. buffer(a): 使用第二个序列来触发源序列中多个元素的打包。参数a指定了提供打包触发时机的第二个序列，每当这个序列生成一个元素，源序列自上次打包以来的所有元素被打包成一个数组，并作为新序列中的元素。
2. 参数：一个参数，指定第二个序列
3. 例子：
```javascript
var source = Rx.Observable.timer(0, 1000)
var boundaries = Rx.Observable.interval(2500)
var target = source.buffer(boundarier) // 序列：[0, 1, 2], [3, 4, 5], [6, 7], ...
```

#### bufferWithCount()
1. 用处： 将源序列中指定数量的元素打包为新序列的一个元素
2. 参数：有两个参数，其中第二个为可选参数。第一个参数声明一次打包需要的元素数量，第二个为可选参数，用于声明打包的步进数量，默认等于第一个参数。
3. 返回值：一个Observable
4. 例子：
```javascript
var source = Rx.Observable.timer(0, 1000); // 序列：0 1 2 3 4 ...
var target1 = Rx.bufferWithCount(3) // 序列：[0, 1, 2], [3, 4, 5], ...
var target2 = Rx.bufferWithCount(3, 1) // 序列：[0, 1, 2], [1, 2, 3], [2, 3, 4], ...
```

#### bufferWithTime()
1. 用处：在每个固定的时间间隔中对源序列进行打包
2. 参数：一个参数，用于指定时间间隔
3. 返回值：一个Observable
4. 例子：
```javascript
var source = Rx.Observable.timer(0, 1000)
var target = source.bufferWithTime(2500) // 序列：[0, 1, 2], [3, 4], ...
```

#### bufferWithTimeOrCount()
1. 用处：比bufferWithCount()增加了一个超时时长参数。若在参数规定的时长内没有手机到足够数量的元素，这个方法就不会继续等待，即发送complete信号。
2. 参数：两个参数，第一个指定超时时长，第二个参数指定每次打包要求的元素数量
3. 放回值：一个Observable
4. 例子：
```javascript
var source = Rx.Observable.timer(0, 1000);
var target = source.bufferWithTimeOrCount(5000, 3) // 序列：[0, 1, 2], [3, 4, 5]
```

#### window()
1. 用处：buffer()用法与效果一样，只是window()将每次收集到的元素构造成一个序列而不是数组。
2. 参数：一个参数，指定第二个序列
3. 返回值：一个Observable，其每个元素也是一个Observable
4. 例子：
```javascript
var source = Rx.Observable.timer(0, 1000);
var boundaries = Rx.Observable.timer(5000)
var target = source.window(boundaries) // 序列：Observable(0,1,2,3,4,5)
target.subserbe((s, i) => {
	s.subscribe((x, j) => console.log(x)) // 序列：0 1 2 3 4 5
})
```
#### windowWithCount()
1. 与bufferWithCount()用法一致，不同的只是该方法返回的Observable的每个元素也是一个Observable

#### windowWithTime()

#### windowWithTimeOrCount()

### pairwise()
1. 用处：将当前接受到的值与该值前面的一个值组成一个数组
2. 参数：无参数
3. 返回值：一个Observable
4. 例子：
```javascript
  Rx.Observable.interval(1000)
    .map(i => String.fromCharCode(i % 26 + 97))
    .pairwise()
    .subscribe(x => console.log(x));
  // 输出：
  // [ 'a', 'b' ]
  // [ 'b', 'c' ]
  // [ 'c', 'd' ]
  // ...
```

### groupBy()
1. 用处：按照指定的键，将源序列进行分组，每个分组构成一个新的序列
2. 参数：两个参数，第一个参数是一个键选择函数，该函数的参数是源序列的元素，其返回值将作为序列的元素；第二个参数为可选参数，是一个映射函数，对源序列的元素进行映射，即分组后保存的值为映射后的值而非原值。
3. 返回值：一个Observable，其每个元素也是一个Observable。
4. 例子：
```javascript
var codes = [
    { keyCode: 38}, // up
    { keyCode: 38}, // up
    { keyCode: 40}, // down
    { keyCode: 40}, // down
    { keyCode: 37}, // left
    { keyCode: 39}, // right
    { keyCode: 37}, // left
    { keyCode: 39}, // right
    { keyCode: 66}, // b
    { keyCode: 65}  // a
];

var source = Rx.Observable.from(codes)
    .groupBy(
        function (x) { return x.keyCode; },
        function (x) { return x.keyCode; });

var subscription = source.subscribe(
    function (obs) {
        // Print the count
        obs.count().subscribe(function (x) {
            console.log('Count: ' + x);
        });
    },
    function (err) {
        console.log('Error: ' + err);
    },
    function () {
        console.log('Completed');
    });

// => Count: 2
// => Count: 2
// => Count: 2
// => Count: 2
// => Count: 1
// => Count: 1
// => Completed
```

### Scheduler
1.通常，如果有什么时间开销很大的操作时，为了效率，我们通常采取的是异步策略，即下面的**Rx.Scheduler.default**。

#### Rx.Scheduler.currentThread
1. 用处：一种同步执行的策略，只有Observable发Completed信号后，才会开始执行Observable后面的其它代码
2. *from* uses **Rx.Scheduler.currentThread** internally(内部), which schedules work to run after any current work is finished. Once it starts, it processes all the notifications **synchronously**
3. 例子：
```javascript
var timeStart = Date.now();
Rx.Observable.from(arr)
	.subscribe(
		res => {},
		err => {},
		_ => {
			console.log('Total time:' + (Date.now() - timeStart) + 'ms')
		}
	)
console.log('Hi there!')
// 输出：
// 6ms
// Hi there!
```

#### Rx.Scheduler.default
1. 用处：一种异步执行的策略，不管Observable有没开始执行，它后面的都会开始执行
2. 例子：
```javascript
var timeStart = Date.now();
Rx.Observable.from(arr, null, null, Rx.Scheduler.default)
	.subscribe(
		res => {},
		err => {},
		_ => {
			console.log('Total time:' + (Date.now() - timeStart) + 'ms')
		}
	)
console.log('Hi there!')
// 输出：
// Hi there!
// 5423ms
```

#### Rx.Scheduler.immediate
1. 用处：一种立即执行的策略。一旦subscribe，无论此时是否有其它代码在执行，都会阻塞代码的执行，从而立即执行subscribe里面的代码
2. Rx.Observable.range()默认采用该策略

#### observeOn() and subscribeOn
1. **observeOn** and **subscribeOn** are instance operators that return a copy of the Observable instabce, but that use the Scheduler we pass as a parameter. Both of them accept **Rx.Scheduler.default** or **Rx.Scheduler.currentThread** as input.
	* **observeOn** takes a Scheduler and returns a new Observable that uses that Scheduler. It will make every *onNext* call run in the new Scheduler
	* **subscribeOn** forces the subscription and un-subscription work of an Observable to run on a particular Scheduler
2. 例子：
```javascript
var source = Rx.Observable.from(...);
source.groupBy(value => value % 2 === 0)
	.map(function(value) { // note: value is an Observable
		return value.observeOn(Rx.Scheduler.default)
	})
	.map(function(groupedOservable) {
		return expensiveOperation(groupedOservable)
	})
```