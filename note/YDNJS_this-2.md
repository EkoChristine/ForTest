## this的呼叫地點(call-site)

### 呼叫地點
考慮呼叫堆疊(call-stack, 執行到目前為止已被呼叫的函示所成的堆疊)

```javascript
function baz() {
  // 呼叫堆疊(call-stack): `baz`
  // 呼叫地點(call-site): `global scope` 
  
  console.log( "baz" );
  bar(); //`bar`的呼叫地點(call-site)
}

function bar() {
  //呼叫堆疊(call-stack): `bar`-> `bar`
  //呼叫地點(call-site): baz`
  console.log( "bar" );
  foo(); //`foo` 的呼叫地點
}

function foo() {
  //呼叫堆疊(call-stack): `baz` -> `bar` -> `foo`
  //呼叫地點(call-site): `bar`
  console.log( "foo" );
}

baz(); // `baz`的呼叫地點
```

### 呼叫地點(call-site)如何決定this在function執行的過程中會指向的東西
#### 1. 預設繫結(default binding)
```javascript
function foo() {
  console.log(this.a); //被解析為global的a
  //呼叫地點(call-site): global
  //this指向全域物件(global object)
}

var a = 2;

foo(); //2
```

※ 若在strict mode中, this會改被設成undefined
```javascript
function foo() {
  "use strict";
  
  console.log( this.a);
}

var a = 2;
foo(); //TypeError: `this` is `undefined`
```

※ 雖然this繫結規則整體而言取決於call-site，
全域物件只有在foo()的內容不是在strict mode中執行才會成為合格的default binding
foo()的呼叫地點(call-site)是否處於strict mode狀態不重要
```javascript
function foo() {
  console.log( this.a);
}

var a = 2;

(function() {
  "use strict";
  foo(); //2
})();
```

#### 2. 隱含繫結(implicit binding)
call-site是否有情境物件(context object),又稱擁有物件(owning object),或包含物件(containing object)。
call-site使用obj情境來參考該函式，obj物件在函式被呼叫時"擁有(owns)"或"包含(contains)"那個函式參考
```javascript
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2,
  foo: foo
}

obj.foo(); //2
```
在一個物件特性參考的串鏈(object property reference chain)中，只有最頂層或最終層(top/last level)才對呼叫地點(call-site)有意義
```javascript
function foo() {
  console.log( this.a);
}

var obj2 = {
  a: 42,
  foo: foo
};

var obj1 = {
  a:2,
  obj2: obj2
};

obj1.obj2.foo(); //42
```

**失去隱含的繫結**
例1.
通常代表會退回到備用的預設繫結(default binding)，也就是全域物件(global)。
或是strict mode的undefined。
```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo: foo
}

var bar = obj.foo  //函式參考(reference)或別名(alias)
var a = "oops, global";

bar(); //"oops, global"
```
呼叫地點(call-site) = bar()
套用default binding

例2. 傳遞一個callback function
```javascript
function foo(){
  console.log( this.a);
}

function dooFoo(fn){
  //`fn`只是對`foo`的另一個參考
  fn(); //call-site
}

var obj = {
  a: 2,
  foo: foo
};

var a = "oops, global";
dooFoo(obj.foo); //"oops, global"

```
如果callback是js 內建的呢?
```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2, 
  foo: foo
};

var a = "oops, global";

setTimeout(obj.foo, 100); //"oops, global"
```

> 函式callbacks會失去它們的this binding
> 某些受歡迎的js程式庫的event handles會迫使你們callback把this指向觸發該事件的DOM element。
> 雖然有時候會有用處，但其他時候根本是令人發怒的做法


#### 3. 明確的繫結(explicit binding)
透過使用call()和apply()，直接指出想要的this是什麼，
```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2
};

foo.call(obj); //2
```

> 如果傳入一個簡單的基本型別值(string/boolean/number)作為this的binding，
> 那個基本型別值會被包裹在它的物件形式(object form,分別是 new String(), new Boolean()或 new Number())中，
> 這通常被稱為封裝(boxing)

但光靠明確的繫結(explict binding)無法解決剛剛callback的問題(失去預期的this binding)

**硬繫結(hard binding)**
```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2
};

var bar = function(){
  foo.call(obj);  //手動呼叫foo.call(obj)，強制以obj作為this的繫結，永遠以obj手動調用foo
};

bar(); //2
setTimeout(bar, 100); //2

//硬繫結(hard-bound)的`bar`不再會讓它的`this`被覆寫
bar.call( window); //2
```