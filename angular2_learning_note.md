[TOC]
1. Input 用于从父模板中接受值作为本模板的输入（Notice that we changed the name property to have an annotation of @Input . We talk a lot more about
Input s (and Output s) in the next chapter, but for now, just know that this syntax allows us to pass
in a value from the parent template.）.Example:
```js
import { Input } from '@angular/core';
export class ... {
	@Input() name: String;
	......
}
```
2. To pass values to a component we use the bracket [] syntax in our template. Expamle:
```js
<ul>
	<app-user-item *ngFor="let name of names" [name]="name"></app-user-item>
</ul>
```
3. Notice that in the input tags we used the # (hash) to tell Angular to assign those tags to a local
variable. By adding the #title and #link to the appropriate < input /> elements, we can pass them
as variables into the addArticle() function on the button! Example:
```js
<form class="ui large form segment">
	<h3 class="ui header">Add a Link</h3>
	<div class="field">
		<label for="title">Title:</label>
		<input name="title" #newtitle> <!-- changed -->
	</div>
	<div class="field">
		<label for="link">Link:</label>
		<input name="link" #newlink> <!-- changed -->
	</div>
	<!-- added this button -->
	<button (click)="addArticle(newtitle, newlink)" class="ui positive right floated button">
		Submit link
	</button>
</form>
```
4. In javascript, by default, propagates the click event to all the parent components. Because the click event is propagated to parents, our browser is trying to follow the empty link, which tells the browser to reload.

##内嵌指令 
1. ngIf
```html
<div *ngIf="false"></div> <!-- never display -->
<div *ngIf="a > b"></div> <!-- displayed if a is more than b -->
<div *ngIf="str == 'yes'"></div> <!-- displayed if str holds the string "yes" -->
<div *ngIf="myFunc()"></div> <!-- displayed if myFunc returns a true value -->
```
2. ngSwitch
```html
<div [ngSwitch]="myVar">
	<div *ngSwitchCase="'A'">Var is A</div>
	<div *ngSwitchCase="'B'">Var is B</div>
	<div *ngSwitchDefault>Var is somethin</div>
</div>
```
3. ngClass: 动态设置css类
```html
<style>
	.bordered {
		border: 1px dashed black;
		background-color: #eee;
	}
</style>
<div [ngClass]="{bordered: false}">This is never bordered</div>
<div [ngClass]="{bordered: true}">This is always bordered</div>
<div [ngClass]="['blue', 'round']">This will always have a blue background and round corners</div>
```
4. *ngFor: let i=index;来获取下标
5. ngNonBindable: 用于告诉angularjs编译器，此处无需进行编译、绑定处理，即此处不属于angular的内容。
```html
<span ngNonBindable>
	This is what {{ content }} rendered
</span>
```

##Form in Anuglar 2
1. FormControl 和 FormGroup。
	* A FormControl reporsents a single input field - it is the smallest unit of an Angular form.
	* FormGroup: consists of multiple FormControls
```js
// [1] FormControl
// create a new FormControl with the value "Nate"
let nameControl = new FormControl("Nate"); // 在html中等价于 <inpuy type="text" [FormControl]="nameControl">，其中value为输入的值
nameControl.value; // -> Nate
nameControl.errors; // -> StringMap<string, any> of errors
nameControl.dirty; // -> false, means no change for the value
nameControl.valid; // -> true

// [2] FormGroup
let personInfo = new FormGroup({
	firstName: new FormControl("Nate");
	lastName: new FormControl("Murray");
	zip: new FormControl("90210")
});
personInfo.value; // -> { firstName: "Nate", lastName: "Murray", zip: "90210" }
personInfo.errors; // StringMap<string, any> of errors
personInfo.dirty; // -> false
personInfo.valid; // -> true
```
2. 两种使用forms的方式：
	* FormsModule
		* ngModel
		* ngForm
	* ReactiveFormsModule
		* formControl
		* ngFormGroup

    在根模块中的NgModule中的imports中注入其一后，就能在app中任一一个模块中使用它。
```html
<!-- [1] FormsModule -->
<div class="ui raised segment">
  <h2 class="ui header">Demo Form: Sku</h2>
  <form #f="ngForm"
    (ngSubmit)="onSubmit(f.value)"
    class="ui form"> <!-- #f为ngForm的别名，通过它能获得对应的NgForm对象，从而获得输入值 -->
      <div class="field">
        <label for="skuInput">Sku</label>
        <input type="text" id="skuInput" name="sku" placeholder="SKU" ngModel> <!-- 以name的值作为键。若ngModule="xxx"，则默认input的value为xxx，否则为空  -->
      </div>
      <button type="submit" class="ui button">Submit</button>
    </form>
</div>
```
3. formbuilder是在类中创建formGroup和formControl，然后再将之绑定到html中
formodule是在html中创建formGroup和formControl，然后再将之绑定到相应的类中
4. Validators.compose 会将数组中的每个函数都执行一遍，注意每个函数的参数和返回值都需一致
```js
import { Validators } from "@angular/forms"; 

Validators.compose([
	function1,
	function2,
	......
])

function1(argument1, .......): A {
	......
}

function2(argument1, ......): A {
	......
}

// argument1, ......, A 都是一样的
```
5. formControl**.valueChanges** / formGroup**.valueChanges**: 用于监听input / form输入值的监听；
formControl.valueChanges**.subscribe(() => {})** / formGroup.valueChanges**.subscribe(() => {})**: 为监听创建回调函数。
```js
    this.myForm = fb.group({
        'sku': ['dddd', Validators.compose([ // Validators.compose 会将数组中的每个函数都执行一遍，注意每个函数的参数和返回值都需一致
          Validators.required,
          this.skuValidators
        ])]
    });
    this.sku = this.myForm.controls['sku']; // 如此就能在该模块中的任意一处地方引用到该formcontrol
    this.sku.valueChanges.subscribe(
      (value: string) => {
        console.log('sku changed to:', value);
      }
    );
    this.myForm.valueChanges.subscribe(
      (form: any) => {
          console.log('form changed to:', form);
      }
    );
```
6. Angularjs2中为input进行双向数据绑定的方法如下：
```
...
<input [(ngModel)]="×××" />
...

class abc {
	...
	×××: string = '';
	...
}
```

##HTTP
1. 在js中，异步的实现方法有三个：
	* Callbacks
	* Promises( .then() )
	* Observables( .subscribe() )
2. 在angular2中，使用Observables来实现异步：@angular/http
```typescript
import { Http, Response, RequestOptions, Headers } from '@angular/http';
```
3. 一个最简单的get请求是http.request(URL)，其中，http.request返回的是一个Observables对象，而且当返回该对象时，会令stream发射一个Response对象，即为请求结果。完整的过程如下：
```typescript
http.request(URL).subscribe((res: Response) => {}) // 最简单的get请求
```
4. javascrpt的简单语法记录
```javascript
let i = a && b; // a、b均不为空时，选b，即后者；有任意一个为空时，为空
let j = a || b; // a、b均不为空时，选a，即前者；有任意一个为空时，选不为空的那个
```
5. 

##RxJS
1. 对事件进行监听：将事件转变成一个可观察流(observable stream)，例如：
```typescript
// convert the 'keyup' event into an observable stream
Observable.fromEvent(this.el.nativeElement, 'keyup')
```
2. subscribe()**依次**接受三个回调：onSuccess、onError、onCompletion，其中后两个可选

##Routing
1. Routes、RouterModule：不同页面之间的跳转
```typescript
import {
	RouterModule,
	Routes
} from '@angular/router';

const routes: Routes = [
	{ path: '', redirectTo: 'home', pathMatch: 'full' },
	{ path: 'home', component: HomeComponent },
	......
]

@NgModule({
	......
	imports: [
		......
		RouterModule.forRoot(routes) // 为 应用 能使用路由而提供必须的 依赖逻辑，将路由注入到 应用 中
		......
	]
	......
})

......

// 使用
<a [routerLink]="['home']">...</a>
...
```
2. routerLink。
router-outlet: is used to indicate where the in our template the route contents should be rendered（由于在Ngmodule中引用了RouterModule，所有就能在任一地方使用该指令）
```html
<ul>
	<li><a [routerLink]="['home']"></a></li>
	<li><a [routerLink]="['about']"></a></li>
	<li><a [routerLink]="['contact']"></a></li>
</ul>
<router-outlet></router-outlet>  <!-- 跳转的结果在该指令中显示 -->
```
3. `<base href="/">`：指定所有路由的跟挂载点，后面的路径就变成以 “/#” 开头的了。倘若为`<base href="/app">`，则变成以 "/app/#" 开头的了。注意，`<base href="/">`和如下等价：
```typescript
@NgModule({
	......
	providers: [
		......
		{provide: APP_BASE_HREF, useValue: '/' },
		......
	]
	......
})
```
4. router strategy: 缺省下采用的路由策略为PathLocalStrategy，而我们在**单页面应用**的客户端的路由，应该采用HashLocationStrategy。前者对应的url是“/×××”，后者对应的url是"#/×××"。
```typescript
......
import { LocationStrategy, HashLocationStrategy } from '@angular/common';
......

@NgModule({
	......
	providers: [
		......
		{ provide: LocationStrategy, useClass: HashLocationStrategy }
		......
	]
	......
})
```
5. 路由参数
```typescript
// 例如：对于 { path: 'article/:id', component: ArticleComponent }
// 则在ArticleComponent的构造函数中来获取路由参数
import { ActivateRoute } from '@angular/router';
......
export class ArticleComponent {
	id: string;
	
	constrcutor(provate route: ActivedRoute) {
		route.params.subscribe(params => this.id = params['id']);
	}
}
```

### Route Guards: 客户端的路由保护
```typescript
// [1] *.guard.ts，实现引导类，继承CanActivate接口，实现函数canActivate的定义
import { Injectable } from '@angular/core';
import { CanActivate } from '@angular/router';
......
@Injectable
export class * implements CanActivate {
	canActivate(): boolean {
		...
		return ...;
	}
}
// [2] app.module.ts: 将guard类作为服务导入到providers中
providers: [
	''
]
// [3] 在路由中使用它——若canActivate执行结果为true，则将对应的ProtectedComponent模块正常渲染出来，否则不渲染
{ path: 'protected', component: ProtectedComponent, canActivate: [LoggedInGuard]}
```

### nested routes 嵌套路由
1. 详情看router_learning/base中的products
```tpyescript
import { routes_products } from './ts/components/products/products.router';
{ path: 'products', component: ProductsComponent, children: routes_products }  // 嵌套路由
```

##Dependency Injection 依赖注入（代码dependency_injectory/complex）
1. 定义：当组件/服务A需要组件/服务B运行时，就说B是A的一个依赖
2. 依赖注入由三个部分组成：
	* Provider：{provide: ... , useClass/useValue: ...}，用于将token和依赖绑定在一起，其中provide就是token，useClass/useValue就是依赖
	* Injector：负责维护由Provider组成的集合，并且在创建组件时，解决其依赖问题并且将依赖注入到该组件中
	* Dependency：就是被注入的依赖![](/home/flyman/Desktop/Angular/angular2/note/screenshot/DI_3.png) 
3. 目的：降低组件/服务间的耦合度
	* 没有依赖注入时：![](/home/flyman/Desktop/Angular/angular2/note/screenshot/DI_1.png) 
	* 有依赖注入时：![](/home/flyman/Desktop/Angular/angular2/note/screenshot/DI_2.png) 
4. **单例模式**注入：
	* 形式：`{ provide: ... , useClass / useValue: ... }`
	* 实现机制：injector将会自动创建一个单例，此处记为A，并且在注入的时候直接将A返回。（当A被第一次注入时，其实它是还未被实例化的，所有在第一注入时，DI系统将会自动调用A的构造函数，实例化A。此后当再注入A时，就直接返回第一次已经实例化的A。注意，这里要求A的构造函数不能有参数，倘若有参数，则需要工厂模式而非单例模式）
```typescript
/*
 * 下面两种注入实现方式均为“单例模式”（所谓的“单例模式”，是指其实例仅会有一个，每次注入的其实是同一个）
 */
 
/*  一、 自己手动创建自己的injector，无需在NgModule中的providers中注入  */
// [1]
export class MyService {
	......
}
// [2]
import { ReflectiveInjector } from '@angular/core';
import { MyService } from '......'
......
class app {
	myService: MyService;
	constructor() {
		// 在构造中创建一组依赖，此后在该component中就能使用这些依赖，而且是以“单例模式”获得的。
		// 个人认为，这其实是下面angualr自带的injector的背后的实现方法，即将providers作为参数放入
		// 到resolveAndCreate中
		let injector: any = ReflectiveInjector.resolveAndCreate([MyService]);
		this.myService = injector.get(MyService);
		console.log('Same instance?', this.myService === injector.get(myService)); // true，单例模式
	}
}
/* 二、使用angular自带的injector */
// [1] 加入到NgModule的providers中
@NgModule({
	......
	providers: [
		......
		{provide: MyService, useClass: MyService}, // 等价于直接写MyService
		......
	]
	......
})
// [2] 使用，以下两种方法均是等价的
impor { MyService } from '......';
......
class * {
	......
	constructor(private myService: MyService) {}
	......
}
// 或者
class * {
	private myService: MyService;
	constructor(@Inject(MyService) myService) {
		this.myService = myService;
	}
}
```
5. **工厂模式**注入
	* 形式：`{ provide: ... , useFactory: function}`，其中，function的返回值是任意一个对象，例如`() => { ...; return new MyComponent(); }`
	* 自带的factory只会执行一次：在程序启动时
```typescript
/* 使用例子 */
// [1] 声明
providers: [
  ApiService,
  ViewPortService,
  { provide: 'ApiServiceAlias', useExisting: ApiService }, // 单例模式
  {
    provide: 'SizeService', // 工厂模式
    useFactory: (viewport: any) => {
      return viewport.determineService();
    },
    deps: [ViewPortService]
  }
]
// [2] 使用
constructor(private apiService: ApiService,
            @Inject('ApiServiceAlias') private aliasService: ApiService,
            @Inject('SizeService') private sizeService: any) {}
```
6. 注意，依赖注入，可以做到“多选一”。利用这个特点，我们就能实现“动态加载（自己想的词，不知是否合适贴切）”的效果了。以下为例：
```typescript
{
	provide: API_URL,
	useValue: isProduction ? 'http://www.baidu.com' : 'http://www.google.com' 
}
```

##NgModule
1. 一种在Angular框架中管理依赖的方式：**what tags are compiled** and **what dependencies should be injected**
2. 目的：
	* 可让其在“module”层次上避免自定义的tag冲突　Page265
3. 
```typescript
@NgModule({
	imports: [],　　　　// 用于声明该模块依赖于哪些其它模块
	exports: [],       // 用于向其它模块公布本模块的组件，即声明本模块的哪些组件可作为公有组件给外面用，未公布出去的模块只能供本模块使用，即为私有组件
	declarations: [],　// 用于声明该模块依赖于哪些组件
	providers: [],　　　// 依赖注入
	bootstrap: [] 　　　// 用于指定哪个Componet作为程序的入口（根）组件
})
```
##Data Architecture in Angualar 2
1. Angular2并未强制规定要使用哪些数据架构，这由开发者自由选择。
下面为一些常见的客户端方面的数据架构：
> MVW / Two-way data binding: 双向数据绑定
 Flux: 单向数据绑定
Observables: 观察者模式——Observables give us streams of data. We subscribe to (订阅) the streams and then perform operations to react to changes. (RxJs)

###Data Architecture with Observables

####Part 1: Services
1. Reactive Programming: 反应性编程，即使用Observables来管理我们的数据架构。
> Reactive Programmng is a way to work with asynchronous streams of data. Observables are the main data structure we use to implement Reactive Programming.

2. 学习笔记
	* RxJS将数据分为两种：
		* 普通数据——如数组、字符串、JSON对象之类的
		* 可观察对象——如使用Rx.Observable.create()方法获得的对象
		* from(): 将普通数据变换到Observable域
		   to(): 将Observable还原为普通数据
		   ![](/home/flyman/Desktop/Angular/angular2/note/screenshot/D_3.png) 
	* 扩展方法 / 操作符(Operator)：一个操作符通常返回的 是另一个新的Observable对象
```javascript
[1] from():将普通数据变换到Observable域。其中普通数据集有：
		Array: 数组的每个成员对应序列的一个元素
		String:　字符串的每个字符对应序列的一个元素
		Set:　Set的每个成员对应序列的一个元素
		Map: 类似Json对象，但按照插入次序进行遍历。转换后，每个键值对，对应序列的一个元素
		注意，Observable每次只会emit一个元素，当所有元素emit完后，emit一个completed信号
	pairs(): 将传统Json对象转化为可观测序列。转换后，原先的每个键值对，对应序列的一个元素
	to(): 将Observable还原为普通数据
	of(): 不在同一个数据集中的多个来源的数据，能用of()方法直接构造_可观测序列/Observable_。该方法可传入任意数量的参数，每个参数对应序列中的一个元素
例如：var a = Rx.Observable.from([1,2,3])
	 var b = a.toArray();
	 
	 var score　= {a: 80, b: 90, c: [1,2,3], d: {m: 1, n: 2}}
	 Rx.Observable.pairs(score) // 序列: ['a', 80], ['b', 90], ['c', [1,2,3], [d,{m: 1, n: 2}]]
	 
	 Rx.Obervable.of("a", "b", "c") // 序列：a b c　等价与 Rx.Observable.from([a, b, c])
	 
[2] concat(): 将多个可观察对象拼接起来
例如：let a = Rx.Observable.from([1, 2])
	 let b = Rx.Observable.from([3, 4])
	 let c = Rx.Observable.from([5, 6])

	 a.concat(b, c)
 	 	.subscribe(result => console.log(result)) // [1,2,3,4,5,6]
 	 	
[3] map(): 将Observable对象生成的每个数据进行映射处理
例如：// a: 1 2 3
	a.map(function(d) => return d*10) // 10 20 30
	
[4] empty(): 创建一个没有袁术的空序列，并立刻结束。该方法无需参数
例如：let source = Rx.Observable.empty()　等价与 Rx.Observable.create(o => o.onCompleted());
	never(): 创建一个没有元素的空序列，并且永远不结束。该方法无需参数
例如：let source = Rx.Observable.never() 等价于 Rx.Observable.create(o => {});
	throw(): 创建一个没有元素的空序列，并立即抛出错误。；
例如：var source = Rx.Observable.throw("Error throw")　等价与 Rx.Observable.create(o => o.onError("Error throw"));

[5] range(a, b):　创建一个有限长度的_整数序列/Observable_ 。该方法有两个参数第一个为序列的起始值，第二个为序列的元素数量。
例如：Rx.Observable.range(30, 4) // 30, 31, 32, 33

[6] interval(a): 创建一个无限长度的周期性_可观测序列/Observable_。该方法有一个参数，用于声明了以毫秒为单位的周期。序列的元素值从0开始，每个周期比前一个周期加1。注意，第一个值的生成不是立刻的，依旧需要等待一个_静默周期_，该周期即与指定的参数周期一致
例如：Rx.Observable.interval(1000) // 输出：0 1 2 3 ...

	timer(a, [b]): 可以指定一个额外的参数a来调节第一个值生成之前的_静默时长_。第二个值b为可选值，若有，则会生成一个除第一个值外的无限长度的周期为b的序列；若无，则仅仅在规定的静默时长后输出一个值，然后结束序列。
例如：Rx.Observable.timer(100, 1000) // 输出：0 1 2 3 ...
	 Rx.Observable.timer(500) // 输出：0
	 
[7] just(a): 将任何数据转化为一个单值输出(即序列的长度为1)的_可观测序列/Observable_。该方法传入一个任一类型的参数，且有一个别名return()
例如：Rx.Observable.just([1,2,3])　or Rx.Observable.return([1,2,3])　 // 输出序列：[1,2,3]
	 Rx.Observable.just({x:1, y: 2}) or Rx.Observable.return({x:1, y: 2})  // 输出序列：{x:1, y:2}
	 
[8] repeat(a,b): 创建一个重复值序列。第一个参数指定需要重复的值，第二个参数指定重复的次数
例如：Rx.Observable.repeat("a", 5) // 序列：a a a a a

[9] fromEvent(a, b[, callback]): 将事件流转化为_可观测序列/Observable_，每个事件对应序列中中的一个元素。该方法必需的参数有两个，第一个参数a用于指定一个事件源对象，第二个参数b指定需要监听的事件名称，第三个为可选参数，它是一个映射函数，负责将传入的事件转换为其它值作为序列的元素。
例如：var el = document.getElementById("btn"); //DOM对象作为事件源
	 var mf = function(event){
    	　return [event.offsetX,event.offsetY];
	 };
	 Rx.Observable.fromEvent(el,"click",mf); //序列: [??,??] [??,??] ...
	 等价于
	 Rx.Observable.fromEvent(el, "click")
	 	.map(event => {
	 		return  [event.offsetX,event.offsetY];
	 	})
RxJS支持的事件源：DOM对象、jQuery对象、Zepto对象、Angular对象、 Ember.js对象、EventEmitter对象。
	fromEventPattern(a,b): 另一种将事件流转化为序列的方法。区别在于， 你需要使用两个函数分别封装监听事件和解除监听事件的逻辑，然后，将这两个函数 作为fromEventPattern()的输入参数a、b。其优点是可以一次将多个对象的多种事件转化为 一个事件序列。
例如：var input = document.getElementById("input");
　　　var target1 = Rx.Observable.fromEvent(input, 'click');
　　　var target2 = Rx.Observable.fromEventPattern(
        function(handler) {  
        	input.addEventListener('click', handler); 
        	input.addEventListener('mouseover', handler); 
        	input.addEventListener('mouseout', handler); 
        },
        function(handler) {  
        	input.removeEventListener('click', handler);
        	input.removeEventListener('mouseover', handler); 
        	input.removeEventListener('mouseout', handler); 
        }
　　　);
　　　[target1,target2].forEach(function(s,i){
        s.subscribe(
            function (data) { timelines[i].next(data.type); },　// 即为上面的handler
            function (err) { timelines[i].error(err); }, // 即为上面的handler
            function () { timelines[i].completed(); } // 即为上面的handler
        );
　　　});

[10] toArray(): 将序列还原为数组对象。需要指出的是，toArray()方法返回的还是一个可观测序列/Observable， 因此需要订阅后获得还原的结果
例如：var source = Rx.Observable.of(1,2,3,4);    //序列：1 2 3 4 
	 var target = source.toArray(); //序列：[1,2,3,4]
	 target.subscribe(function(d){
        console.log(d);        //d： [1,2,3,4]
     })
     
    toMap(a[, b]): 将序列还原为一个Map对象。返回的还是一个可观测序列/Observable， 因此需要订阅后获得还原的结果。第一个参数a用于声明键选择函数，第二个为可选参数，用于声明值选择函数。
例如：var data = [
    	{id:123,name:"John"},
   		{id:456,name:"Kate"}
	 ];
	 var source = Rx.Observable.from(data); //序列： {...} {...}
	 var ksf = function(d){
    	return d.id;  //使用id作为键
	 }
	 var target = source.toMap(ksf);       //序列：Map{123=>{...},456=>{...}}
	 
	 var vsf = function(d){
    	return d.name; //使用name作为值	
	 };
	 var target = source.toMap(ksf,vsf);   //序列： Map{123=>John,456=>Kate}
	 
    toSet(): 将一个序列还原为Set对象。该方法返回的还是一个可观测序列/Observable， 因此需要订阅后获得还原的结果
例如：var source = Rx.Observable.of(1,2,3,4)    //序列：1 2 3 4 
	 var target = source.toSet();   //序列: Set{1,2,3,4}
	 target.subscribe(function(d){
        console.log(d);        //d： Set{1,2,3,4}
     })

[11] delay(a): 将序列推迟制定的时间。参数用于声明延迟的具体时间，以ms为单位；参数也可以是一个Date，声明序列开始的具体时间
例如：var source = Rx.Observable.timer(0,1000); //序列： 0 1 2 ...
	 var target = source.delay(1000); //序列延迟1秒：0 1 2 ...
	
	 var source = Rx.Observable.timer(0,1000); //序列： 0 1 2 ...
	 var t = new Date(Date.now + 1000*60); //1小时之后
	 var target = source.delay(t); //序列延迟1小时：0 1 2 ...
	 
	delalySubscription(a): 通过观测者的订阅行为而实现类似的延迟
例如：var source = Rx.Observable.timer(0,1000); //序列： 0 1 2 ...
	 var target = source.delaySubscription(1000); //序列延迟1秒：0 1 2 ...

[12] timeout(a[, b]): 每当源程序生成一个元素时，timeout()就开始计时，若超出一定时间，源程序还未生成下一个元素，timeout()就抛出错误。该方法有两个参数，第一个a是以ms为单位的超时长度，第二个参数b为可选，用于声明抛出的错误。
例如：var source = Rx.Observable.timer(3000); //序列： 0 
	 var target = source.timeout(1000,new Error("timeout!")); //序列：<ERROR>
	 target.subscribe(
          function (x) { timelines[i].next(x); },
          function (err) { timelines[i].error(err); },
          function () { timelines[i].completed(); }
     );

[13] timeStamp(): 为序列中的每个元素添加Linux时间戳。其返回的新序列中，每个元素具有如下形式：{value: ... , timestamp: ...}
例如：var source = Rx.Observable.timer(0, 5000);
	 var target = source.timestamp()
	 target.subscribe();

[14] doWhile(a, b): 在满足条件的情况下循环执行一个已有序列。第一个参数是一个断言函数，函数的返回值若为true，就继续执行一次源序列，否则就结束序列；第二个参数是一个源序列。该方法总是先复制一次源程序，即使条件不满足。
例如：var source = Rx.Observable.of(1,2,3) // 序列：1 2 3
	 var loops = 2
	 var target = source.doWhile(_　=> {return loops-- > 0;}, source) // 序列：1 2 3 1 2 3
	
	while(): 先检查条件是否满足，再执行源序列
例如：var source = Rx.Observable.of(1,2,3); //序列： 1 2 3
	 var loops = 2;
	 var target = Rx.Observable.while(
     	function(d){  return loops-- > 0; },
    	source
	 ); //序列： 1 2 3 1 2 3

[15] map(a): 对序列成员进行变换，并返回一个新的序列。参数是一个变化函数，函数的返回值就是目标序列中对应的元素。其别名是select()，两者行为一致，可互换。
例如：//item - 值
	 //index - 索引 
	 var tfx = function(item,index){
    	return item * 2;
	 };
	 var source = Rx.Observable.of(1,2,3); //输出： 1 2 3
	 var target = source.map(tfx);   //输出： 2 4 6

[16] pluck(a): 若源序列的元素值是JSON对象，那么可使用该方法选择某个具体属性值生成新的序列。参数a是string，即为属性值的键。
例如：var source = Rx.Observable.of({name:"John",age:22},{name:"Linda",age:21});
	 var target = source.pluck("name"); //序列： John Linda
	 
[17] flatMap(a): 首先将一个序列的各元素映射为新的序列，然后将映射成的各元素序列融合形成一个新的序列。该方法的参数是一个映射函数，其返回值应当是一个序列。
例如：var source = Rx.Observable.of(10,20,3) // 序列：10 20 30
	 var mf = function(item) {
	 	return Rx.Observable.range(item. 3);
	 }
	 var target = source.flatMap(mf); // 序列：10 11 12 20 21 22 30 31 32
	 
	flatMapLatest(a):　与flatMap()的区别在于，仅将最新激活的元素序列中的元素，输出到目标序列中。该方法的参数是一个映射函数，其返回值是一个序列。该方法的别名是selectSwitch()。注意，其实其内部有它自己定义的一小段时间，从该段时间中收集到的所有收到的值，选择最新的那个返回，就是其实现思路。
例如：var source = Rx.Observable.range(0, 5) // 序列：0 1 2 3 4
	 var target = source.flatMapLatest(item => Rx.Observable.interval(500).map(i => item)) // 序列：4
	
	flatMapObserver(a,b,c): 为源序列的每一个元素、错误通知、结束通知都创建一个序列。有三个参数，分别对应onNext、onError、onComplete三个函数，每个函数都返回一个序列。
例如：var source = Rx.Observable.of(1,2,3);
	 var nf = function(d,i){ return Rx.Observable.of(d).delay(i*1000);};
	 var ef = function(err){ return Rx.Observable.of("error");};
	 var cf = function(){ return Rx.Observable.of("completed");}
	 var target = source.flatMapObserver(nf,ef,cf); //序列：1 2 3 completed
	 
	 
	 concatMap(a): 将源序列的各元素映射为序列，然后按顺序拼接起来。该方法有一个参数，为映射函数，其返回值是序列。
例如：var source = Rx.Observable.of(1, 2, 3); // 序列：1 2 3
	 var target = source.concatMap(item => R.Observable.range(item, 3)) // 序列：1 2 3 2 3 4 3 4 5
	 
	 concatMapObserver(a,b,c): 将源序列的元素、错误通知、结束通知都转化为序列，并安装顺序拼接为目标序列，通过subscribe的value回调，即可获得拼接后的目标序列。三个参数，均为映射函数，分别对应onNext()、onError()、onComplete()，返回值均为序列
例如：var source = Rx.Observable.of(1,2,3)
	 var nf = function(d,i) {return Rx.Observable.of(d).delay(i * 1000);}
	 var ef = function(err) {return Rx.Observable.of("err");}
	 var cf = function() {return Rx.Observable.of("completed");}
	 var target = source.concatMapObserver(nf,ef,cf) // 序列：1 2 3 completed
	 
[18] reduce(a, b): 将一组数据规约为一个数据。参数a是一个规约函数，该函数有两个参数；参数b是一个初始值。
例如：var source = Rx.Observable.of(1,2,3)
	 var rf = function(acc, item) {return acc　+ item;}
	 var acc_init = 0;
	 var target = source.reduce(rf, acc_init); // 序列：6

[19] average(): 计算所有元素的均值并返回。参数是一个可选映射函数参数，用法如下：
例如：var source = Rx.Observable.of(10, 20, 30) // 序列：10 20 30
	 var target1 = source.average(); // 序列：30
	 var target2 = source.average(item => item * 10); // 序列：300

[20] sum(): 计算序列所有元素的总和。无参数。返回值是序列，序列subscribe获得结果
例如：var source = Rx.Observable.of(10,20,30); // 序列：10 20 30
	 var target = source.sum(); // 序列：60
	 
	 count(): 计算序列所有元素的总数
例如：var source = Rx.Observable.of(10,20,30); // 序列：10 20 30
	 var target = source.count() // 序列：3

[21] max():　获得序列上所有元素中的最大值。该方法有一个可选的参数，为重载的比较函数。用法如下：
例如：var source = Rx.Observable.of(10,20,19) // 序列：10 20 19
	 var target = source.max(); // 序列：20
	 
	 var data = [
	 	{name: "zhangsan", score:80},
	 	{name:"lisi",score:90},
	 	{name:"wangwu",score:79}
	 ]
	 var source = Rx.Observable.from(data);
	 var cf = function(d1,d2) { return　d1.score > d2.score; }
	 var target = source.max(cf); // 序列：{ name: "list", score: 90 }
	
	maxBy(a): 通过指定一个键选择函数来实现大小的比较。用法如下：
例如：var data = [
	 	{name: "zhangsan", score:80},
	 	{name:"lisi",score:90},
	 	{name:"wangwu",score:79}
	 ]
	 var source = Rx.Observable.from(data);
	 var ksf = function(d1) { return　d.score; } // 键选择函数
	 var target = source.max(ksf); // 序列：{ name: "list", score: 90 }

	min() / minBy()

[22] filter(a[,b]): 用于筛选源序列中满足条件的元素，并返回新的序列。参数a是一个断言(Predicate)函数，只有当该函数返回true时，当前元素才会加入到目标序列中。第二个参数b是可选的，被用作断言函数的this对象。filter()的别名是where()
例如：var source = Rx.Observable.of(1,2,3,4,5) // 序列：1 2 3 4 5
	 var target = source.filter((d, i) => d < 4); // 序列：1 2 3
	 
	 var source = Rx.Observable.of("who","hello","max");
	 var context = { pattern: /^h/ }
	 var target = source.filter((d,i) => d.match(this.pattern), context) // 序列：hello

[23] skip(a): 抑制序列头部指定数量的元素输出，参数a声明需要抑制输出的元素数量。
例如：var source = Rx.Observable.of(1,2,3,4,5) // 序列：1 2 3 4 5
	 var target = source.skip(2) // 序列：3 4 5
	 
	skipLast(a): 抑制尾部指定数量的元素输出。
例如：var source = Rx.Observable.of(1,2,3,4,5) // 序列：1 2 3 4 5
	 var target = source.skipLast(2) // 序列：1 2 3

	skipUntilWithTime(a): 抑制序列头部指定时间内的元素输出。参数a是抑制时间，单位为ms
	skipLastWithTime(a): 抑制序列尾部指定时间内的元素输出。参数a是抑制时间，单位为ms
例如：var source1 = Rx.Observable.timer(0, 1000) // 序列：0 1 2 3 ...
	 var target1 = source1.skipUntilWithTime(2200) // 序列：3 4 5 ...
	 var source2 = Rx.Observable.timer(0, 1000).take(5) // 序列：0 1 2 3 4
	 var target2 = source2.skipLastWithTime(2200) // 序列：0 1 2
	 
	skipWhile(a): 指定一个条件，RxJS将跳过序列头部所有满足条件的元素。参数a是一个断言函数，若其返回值为true，新的序列将忽略源序列的当前元素。注意，一旦新的序列开始复制源序列，对于后续的元素将不再使用断言函数进行过滤。
例如：var source = Rx.Observable.interval(1000) // 序列：0 1 2 3 ...
	 var pf1 = function(item) {return item < 5} // 忽略小于5的元素
	 var target1 = source.skipWhile(pf1) // 序列：5 6 4 7 ...
	 var pf2 = function(item) {return item % 2 === 0} // 忽略偶数
	 var target2 = source.skipWhile(pf2) // 序列：1 2 3 4 5 ... =>　说明一旦新的序列开始复制源序列，对于后续的元素将不再使用断言函数进行过滤

[24] take(a): 截取序列头部指定数量的元素，并返回新的序列。参数a为指定数量
	takeLast(a): 与take()相反
	takeUntilWithTime(a): 截取序列头部指定时长内的元素，并返回新的序列。参数a为时长
	takeLastWithTime(a):　与takeUntilWithTime()相反。参数a为时长
	takeWhile(a): 定一个条件，RxJS将截取序列头部所有满足 条件的元素。参数是一个断言函数，如果其返回值为true，新的序列将截取源序列的当前元素。一旦断言条件不满足，takeWhile()将立即结束
例如：var source = Rx.Observable.timer(0,1000); //序列：0 1 2 3 ...
	 var target = source.takeUntilWithTime(2300); //序列：0 1 2
	 var source = Rx.Observable.timer(0,1000).take(5); //序列：0 1 2 3 4
	 var target = source.takeLastWithTime(2100); //序列：3 4
	 
	 var source = Rx.Observable.interval(1000); //序列： 0 1 2 3 ...
	 var pf = function(item){
    	return item < 5;      //截取小于5的元素
	 };
	 var target = source.takeWhile(pf); //序列：0 1 2 3 4
	 
	 var source = Rx.Observable.interval(1000); //序列： 0 1 2 3 ...
	 var pf = function(item){
    	return item % 2 === 0;      //截取偶数
	 };
	 var target = source.takeWhile(pf); //序列：0 => 一旦断言条件不满足，takeWhile()将立即结束

[25] distinct([a,b]): 抑制序列中的重复值，并返回一个新的序列。有两个可选参数，若源序列的元素不是基本类型，那么参数a需传入一个键选择函数；若需要改变默认的等价规则，那么参数b需传如一个比较函数，返回值为true时，则认为是相同的元素。
例如：var source = Rx.Observable.of(1,2,3,4,2,1); //序列： 1 2 3 4 2 1
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
	 
	distinctUntilChanged([a,b]): 去掉_连续_的相同的元素。有两个可选参数，用法与distinct()一样。
比如：var source = Rx.Observable.of(1,2,2,4,2,1); //序列： 1 2 2 4 2 1
	 var target = source.distinctUntilChanged(); //序列：1 2 4 2 1

[26] elementAt(a[, b]): 获取序列中指定下标的某个元素。参数a是下标，参数b是可选的，用于当指定的下标中没有元素时，b就作为一个缺省值。
比如：var source = Rx.Observable.of(10, 20, 30); // 序列：10 20 30
	 var target1 = source.elementAt(1); // 序号：20
	 var target2 = source.elementAt(8, 10); // 序号：10

[27] first([a]): 从序列中查找第一个满足指定条件a的元素，并返回新的序列，若无a则直接返回第一个元素。参数a是一个断言函数，若返回值为true，就表示当前元素满足条件；若没有满足条件的序列，则抛出错误；
	 last([a]): 从序列找查找最后一个满足条件a的元素，并返回新的序列，若无a则直接返回最后一个元素。用法与first()一样；
	 find(a): 条件a是必需的，除了a是必需的，其余的与first()一致；
	 findIndex(a): 返回第一个匹配元素的序号。参数a是一个断言函数，若返回true，表明匹配。
例如：var source = Rx.Observable.of(17,57,28,31,52); //序列： 17 57 28 31 52
	 var pf = function(d,i){ return d % 2 === 0; }; //如果是偶数就返回true
	 var target1 = source.first(pf); //序列： 28
	 var target2 = source.last(pf); // 序列：52
	  var target3 = source.find(pf); //序列： 28
	   var target = source.findIndex(pf); //序列： 2

[28] ignoreElements(): 用于忽略源序列的所有元素，仅使用其结束通知。虽然返回的是一个空序列，但其时间周期是具有利用价值的。
例如：var source = Rx.Observable.timer(0,1000).take(5); //序列：4秒长度
	 var target = source.ignoreElements(); //空序列：4秒长度

[29] buffer(a): 使用第二个序列来触发源序列中多个元素的打包。参数a指定了提供打包触发时机的第二个序列，每当这个序列生成一个元素，源序列自上次打包以来的所有元素被打包成一个数组，并作为新序列中的元素。
例如：var source = Rx.Observable.timer(0, 1000)
	 var boundaries = Rx.Observable.interval(2500)
	 var target = source.buffer(boundarier) // 序列：[0, 1, 2], [3, 4, 5], [6, 7], ...
	
	bufferWithCount(a[, b]): 将源序列中指定数量的元素打包为新序列的一个元素。参数a声明一次打包需要的元素数量，可选参数b声明打包的步进数量，默认下该值等于参数a。
例如：var source = Rx.Observable.timer(0, 1000); // 序列：0 1 2 3 4 ...
	 var target1 = Rx.bufferWithCount(3) // 序列：[0, 1, 2], [3, 4, 5], ...
	 var target2 = Rx.bufferWithCount(3, 1) // 序列：[0, 1, 2], [1, 2, 3], [2, 3, 4], ...
	 
	bufferWithTimeOrCount(a, b): 比bufferWithCount()增加了一个超时时长参数。若在参数规定的时长内没有手机到足够数量的元素，这个方法就不会继续等待，即发送complete信号。参数a是指定超时时长，以ms为单位；参数b是指定每次打包要求的元素数量。
例如：var source = Rx.Observable.timer(0, 1000);
	 var target = source.bufferWithTimeOrCount(5000, 3) // 序列：[0, 1, 2], [3, 4, 5]
	 
	bufferWithTime(a): 在每个固定的时间间隔中对源序列进行打包。参数a为时间间隔
例如：var source = Rx.Observable.timer(0, 1000)
	 var target = source.bufferWithTime(2500) // 序列：[0, 1, 2], [3, 4], ...

[30] window(): 与buffer()用法与效果一样，只是window()将每次收集到的元素构造成一个序列而不是数组。
例如：var source = Rx.Observable.timer(0, 1000);
	 var boundaries = Rx.Observable.timer(5000)
	 var target = source.window(boundaries) // 序列：Observable(0,1,2,3,4,5)
	 target.subserbe((s, i) => {
	 	s.subscribe((x, j) => console.log(x)) // 序列：0 1 2 3 4 5
	 })
	
	windowWithCount(a[, a]): 参数a指定每个窗口需要采集的元素数量，可选参数b指定步长，默认等于a
例如：var source = Rx.Observable.timer(0, 1000)
	 var target = source.windowWithCount(3) // 序列：Observable(0, 1, 2) Observable(3, 4, 5) ...
	 var target = source.windowWithCount(3, 1) // 序列：Observable(0, 1, 2) Observable(1, 2, 3) ....
	 
	windowWithTimeOrCount(a, b): 与windowWithCount()效果类似，只是多了一个指定开窗最大时长。参数a用于指定开窗最大时长，参数b指定每个窗口需要收集的元素数量。
例如：var source = Rx.Observable.timer(0,1000); //序列：0 1 2 3 ...
	 var target = source.windowWithTimeOrCount(5000,3); //序列：Observable{0,1,2} Observable{1,2,3}...
	
	windowWithTime(a): 指定的周期对源序列进行开窗
例如：var source = Rx.Observable.timer(0, 1000) 
	 var target = source.windowWithTime(2500) // 序列：Observable(0, 1, 2) Observable(3, 4) ...

[31] groupBy(a[, b]): 按照指定的键，将源序列进行分组，每个分组构成一个新的序列。参数a是一个键选择函数，该函数的参数是源序列的元素，其返回值将作为序列的元素。可选参数b是一个映射函数，对源序列的元素进行映射，即分组后保存的值为映射后的值而非原值。
例如：    var members = [
    		{n:"Jane",g:"F"},
        	{n:"Tom",g:"M"},
        	{n:"Annie",g:"F"},
        	{n:"John",g:"M"}
    	 ];

    	 var source = Rx.Observable.fromArray(members)
    		.flatMap(function(d,i){
        		return Rx.Observable.of(d).delay(i*3000);
        	});
   	 	 var target = source.groupBy(
        	function (x) { return x.g; },
        	function (x) { return x.n; }
    	 );
    	 target.subscribe(obs => {
    	 	// obs.key == 'F' / 'M'
    	 	obs.subscrible(...) // 即需要再订阅一次才能获取到值
    	 })

[32] groupByUntil(a, b, c): 使用第二个序列作为触发信号，来实现对源序列的分段分组。与groupBy()一样，参数a是一个键选择函数，可选参数b是一个值映射函数，参数c是一个触发序列，每当该触发序列产生新的元素，之前的分组就固定下来，随后出现的源序列元素将被添加到新的分组中。
例如：var source = Rx.Observable.timer(0,1000); //序列：0 1 2 3 ...
	 var target = source.groupByUntil(
    	function(item){ return item %2 === 0 ? "EVEN" : "ODD"; },
    	null,
    	function(item){ return Rx.Observable.interval(10000); }
	 ); //序列： Observable{0,2,4,6,8} Observable{1,3,5,7,9} Observable{10,12,14,16,18} ....

[33] concat(): 将多个序列拼接为一个大的序列。参数是数量不定的若干序列，或是一个序列数组。
例如：var source = Rx.Observable.of(1, 2, 3)
	 var target = source.concat(
	 	Rx.Observable.of(4, 5, 6),
	 	Rx.Observable.of(7, 8, 9)
	 ); // 序列：1 2 3 4 5 6 7 8 9
	 
	concatAll(): 若源序列的元素也是序列——序列的序列，那么可以使用concatAll()将各元素序列按顺序拼接起来。无参数。
例如：var source = Rx.Observable.of(10, 20, 30)
		.map(item => Rx.Observable.range(item, 3) )
	 var target = source.concatAll(); // 序列： 10 11 12 20 21 22 30 31 32
	 
	combineLatest(): 将多个序列的最后一个元素（即一段时间内的最近元素），使用组合函数构成目标序列的一个新元素。可以传入多个序列作为参数，或者使用一个序列数组作为参数，组合函数作为最后一个参数，负责将各个序列的最近值组合在一起。需要指出的是，任何一个序列输出一个元素时，都会在目标序列中产生一个新的序列。
例如：var source1 = Rx.Observable.of(1, 2, 3)
		.flatMap(function(d, i) { return Rx.Observable.of(d).delay(i * 1000) }) // 序列：1 2 3
	 var source2 = Rx.Observable.of(4, 5, 6)
	 	.flatMap(function(d, i) { return Rx.Observable.of(d).delay(i * 1000) }) // 序列：4 5 6
	 var cf = funciton(d1, d2) { return d1 + '-' + d2; }
	 var target = source.combineLatest(source2, cf); // 序列：1-4 2-4 2-5 3-5 3-6
	
	withLatestFrom(): 仅在源序列输出元素时，触发生成目标序列中的新元素。参数是一个可变数量的序列，最后一个参数是一个组合函数。注意，当源序列输出一个元素时，此时其它序列只要有一个没有输出值，则不产生新元素
例如：var source1 = Rx.Observable.of(1, 2, 3)
		.flatMap(function(d, i) { return Rx.Observable.of(d).delay.(i * 1000); }) // 序列：1 2 3
	 var source2 = Rx.Observable.of(4, 5, 6)
	 	.flatMap(function(d, i) { return Rx.Observable.of(d).delay(i * 1500); }) // 序列：4 5 6
	 var cf = function(d1, d2) { return d1 + '-' + d2; }
	 // When source1 emits a value, combine it with the latest emission from source2.
	 var target = source1.withLatestFrom(source2) // 序列：2-4 3-5

[33] merge(): 把多个序列的元素并如到源序列中，返回的不是一个全新的序列，而是被修改后的源序列（concat()返回的是一个全新的序列）。该方法的参数是可变数量的序列。
例如：var source1 = Rx.Observable.of(1, 2, 3)
		.flatMap(function(d, i) { return Rx.Observable.of(d).delay(i * 1000); })
	 var source2 = Rx.Observable.of(4, 5, 6)
	 	.flatMap(function(d, i) { return Rx.Observable.of(d).delay(i * 1200); })
	 var target = source1.merge(source2) // 序列：1 4 2 5 3 6
	 
	mergeAll(): 对于序列的序列，用该方法进行融合。无参数
例如：var source = Rx.Observable.interval(1000).take(5).map(item => Rx.Observale.of(item)); // 序列： Observable(0) Observable(1) ...
	 var target = source.mergeAll(); // 序列：0 1 2 3 ...

[34] startWith(): 可以在源序列之前添加额外的元素。接受可变数量的参数，每个参数按照其顺序构成源序列的前缀。
例如：var source = Rx.Observable.of(1, 2, 3) // 序列：1 2 3
	 var target = source.starWith(7, 8, 9) // 序列：7 8 9 1 2 3

[35] switch(): 序列切换。对于_序列的序列_，使用该方法在目标序列中包含最近有输出的序列的元素。这看起来像是在不同元素序列中进行切换。无需参数，每个源序列的元素都是一个序列。

[36] zip(): 按序组合。将多个序列的元素按照顺序进行组合，构成目标序列的元素。可变数量的参数作为输入序列，最后一个参数是一个组合函数，其返回值将作为目标序列的元素。
例如：var source1 = Rx.Observable.of(1,2,3); //序列： 1 2 3
	 var source2 = Rx.Observable.of(4,5,6); //序列：4 5 6
	 var cf = function(d1,d2){ return d1 + '-' + d2;};
	 var target = Rx.Observable.zip(source1,source2,cf); //序列： 1-4 2-5 3-6

[37] forkJoin(): 尾值组合。将多个序列的最后一个元素组合为一个数组后，作为目标序列的唯一元素。接受可变数量的序列作为参数。
例如：var source1 = Rx.Observable.of(1, 2, 3)
	 var source2 = Rx.Observable.of(4, 5, 6)
	 var target = Rx.Observable.forkJoin(source1, source2) // 序列：[3, 6]

[38] catch(): 捕获源序列发生的错误，并返回一个新的序列。参数是一个可观察序列，用于在源序列发生错误时执行。若源序列不发生错误，catch()返回的新序列和源序列完全一致；若发生错误，则从发生错误的该时刻其的后续的源序列中的元素将有替代序列替代。
例如：var source = Rx.Observable.create(function(o) {
		o.onNext(1);
		o.onNext(2);
		o.onError(new Error('fake error'));
		o.onNext(4)
	 }); // 序列： 1 2 <ERROR> 4
	 var target = source.catch(Rx.Observable.from("abc")) // 序列：1 2 a b c

[39] onErrorResumeNext(): 用来为源序列追加后续序列，并返回一个新的序列。参数是一个可观测序列，当源序列结束后，将执行该序列。当源序列错误发生时，其行为与catch()无差别 ，但若错误没发生，则依然执行其参数指定的序列。
例如：var source = Rx.Observable.create(function(o){
    	o.onNext(1);
    	o.onError(new Error("fake bug"));
	 });//序列：1 <ERROR>
 	 var target = source.onErrorResumeNext(Rx.Observable.from("abc"));//序列： 1 a b c
 	 
 	 var source = Rx.Observable.of(1,2,3); //序列： 1 2 3
	 var target1 = source.catch(Rx.Observable.from("abc")); //序列： 1 2 3 
	 var target2 = source.onErrorResumeNext(Rx.Observable.from("abc")); //序列：1 2 3 a b c

[40] retry([a]): 错误时重新订阅。当源序列发生错误时，将重新执行订阅。可选参数a用于指定订阅的次数限制，若错误次数超过此阈值，该方法将会抛出错误而结束。若无a，则持续地重复订阅。
例如：var source = Rx.Observable.create(function(o){
    	o.onNext(1);
    	o.onNext(2);
 		o.onError(new Error("permanent bug"));
	 }); //序列：1 2 <ERROR>
	 var target = source.retry(3); //序列：1 2 1 2 <ERROR>
```
