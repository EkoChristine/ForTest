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
call-site是否有情境物件(context object),又稱擁有物件(owning object),或包含物件(containing object)
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