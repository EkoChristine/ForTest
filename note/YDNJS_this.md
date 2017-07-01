## this or That?

### this?
使用this
```javascript
function identify(){
  return this.name.toUpperCase();
}

var me = {
  name: "Kyle"
};

identify.call(me);  //KYLE
```
明確傳入一個情境物件(context object)
```javascript
function identify(context){
  return context.name.toUpperCase();
}

var me = {
  name: "Kyle"
};

identify(me);  //KYLE
```
> 使用模式越複雜，越能看出以明確參數(explicit parameter)的方式四處傳遞情境(context)，比傳遞this情境還要來的雜亂

### 容易混淆之處
#### 誤解1. 認為this參考到函式本身(the function itself)
```javascript
function foo(num) {
  console.log( "foo:" + num );

  this.count++; //指的不是foo.count
}

foo.count = 0;  
//過度解讀
//新增一個特性count給函式物件foo

var i; 

for(i=0; i<10; i++){
  if(i > 5){
    foo(i);
  }
}

console.log(foo.count);  //0
  
```

hack技巧
```javascript
function foo(num) {
  console.log( "foo:" + num );
  
  data.count++;
  foo.count++;
}

var data = {
  count: 0
};
var foo.count = 0;

var i;

for (i=0; i<10; i++) {
  if(i > 5) {
    foo(i);
  }
}

console.log( data.count); //4
console.log( foo.count); //4
```

> 看起來解決問題，其實只是忽略真正的問題，不去理解this的意義及運作方式
> 退回舒適圈的熟悉機制: 語彙範疇(lexical scope)


> 語彙範疇(lexical scope)代表該範疇是由編寫時期(author-time)對於函式  
> 在何處宣告的決策所定義。
> 編譯的語彙分析(lexing)階段基本上能夠知道所有的識別字(identifiers)是在哪裡宣告的，
> 因此可以預測執行過程中它們如何被查找。
  
  
擁抱this
```javascript
function foo(num){
  console.log( "foo:" + num);
  
  this.count++;
  //'this'現在是'foo'，基於'foo'被呼叫的方式
}

foo.count = 0;

var i;

for (i=0; i<10; i++){
  if(i > 5) {
    foo.call(foo, i); //確保'this'指向函式物件('foo')本身
  }
}

console.log(foo.count); //4
```

#### 誤解2 this以某種方式參考了函式的範疇(function's scope)
```javascript
function foo() {
  var a = 2;
  this.bar();  
  //誤1. 經由this.bar()來參考bar()函式  ※調用(invoke) bar()最自然的方式為省略this. 
  //誤2. 使用this在foo()與bar()的語彙範疇(lexical reference)之間建立一座橋梁，讓bar()取用foo()內的變數a
}

function bar() {
  console.log( this.a );
}

foo(); //undefined
```

### this是什麼?
1. 執行時期(被調用時)的繫結(runtime binding)
2. this的binding與function在何處宣告無關，取決於"函式被呼叫的方式"
3. this參考的是什麼，取決於該function被呼叫的呼叫地點(call-site)