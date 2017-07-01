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

***
### 容易混淆之處
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
