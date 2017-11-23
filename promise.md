### ES6-导图-14章. Promise对象

###概念

<iframe src="http://www.xmind.net/embed/2vfn" width="700px" height="540px"></iframe>
 


####  异步编程的解决方案
#####  三个状态
    a. padding 进行中
    b. fulfilled 已成功
    c. rejected 已失败
#####  两个特点
    1. 对象的状态不受外界影响
    2. 一旦状态改变就不会再变，任何时候都可以得到这个结果
#####  三个缺点
    无法取消
    如果不设回调内部的错误无法获取
    处于padding状态无法得知进度
####2. 基本用法
  实例
  
```
const promise = new Promise(function(resolve, reject) {  // ... some code  if (/* 异步操作成功 */){    resolve(value);  } else {    reject(error);  }});
    resolve函数
        将promise从padding状态变为resolve状态
    reject函数
        将promise从padding状态变为reject状态
    回调函数then
        写法
let promise = new Promise(function(resolve, reject) {  console.log('Promise');  resolve();});promise.then(function() {  console.log('resolved.');});console.log('Hi!');// Promise// Hi!// resolved

```

第一个参数表示成功的回调
第二个参数表示失败的回调
特点
  同步执行

### 3. Promise.prototype.then()
  为Promise 实例添加状态改变时的回调函数。
  
```
getJSON("/posts.json").then(function(json)
 {  return json.post;}).then(function(post) {  // ...});

```

### 4. Promise.prototype.catch

####  异常捕获

    p.then((val) => console.log('fulfilled:', val))  .catch((err) => console.log('rejected', err));// 等同于p.then((val) => console.log('fulfilled:', val))  .then(null, (err) => console.log("rejected:", err));
    
####  同步式写法

    // 写法一const promise = new Promise(function(resolve, reject) {  try {    throw new Error('test');  } catch(e) {    reject(e);  }});promise.catch(function(error) {  console.log(error);});// 写法二const promise = new Promise(function(resolve, reject) {  reject(new Error('test'));});promise.catch(function(error) {  console.log(error);});
    
####  冒泡性质

    逐级传递，向后传递直到被捕获
    
####  推荐写法

    用catch捕获异常，而不在reject函数里面
    // badpromise  .then(function(data) {    // success  }, function(err) {    // error  });// goodpromise  .then(function(data) { //cb    // success  })  .catch(function(err) {    // error  });
    
    
####  吃掉错误

    const someAsyncThing = function() {  return new Promise(function(resolve, reject) {    // 下面一行会报错，因为x没有声明    resolve(x + 2);  });};someAsyncThing().then(function() {  console.log('everything is great');});setTimeout(() => { console.log(123) }, 2000);// Uncaught (in promise) ReferenceError: x is not defined// 123
    Promise 内部的错误不会影响到 Promise 外部的代码
### 5. Promise.all()
####  用于包装多个promise实例
    const p = Promise.all([p1, p2, p3]);
    const databasePromise = connectDatabase();const booksPromise = databasePromise  .then(findAllBooks);const userPromise = databasePromise  .then(getCurrentUser);Promise.all([  booksPromise,  userPromise]).then(([books, user]) => pickTopRecommentations(books, user));
####  断电操作（串联）
    1. 三个同时fulfilled，p才fullfilled
    2. 只要一个reject，则preject
### 6. Promise.race()
####  将多个promise对象包装成一个promise对象
  并联操作
    只要有一个对象状态改变就改变
  例子
    const databasePromise = connectDatabase();const booksPromise = databasePromise  .then(findAllBooks);const userPromise = databasePromise  .then(getCurrentUser);Promise.all([  booksPromise,  userPromise]).then(([books, user]) => pickTopRecommentations(books, user));
  包含对象如果不是promise对象则自动转为promise对象
  
### 7. Promise.resolve()

####  将现有对象转为promise对象方法
  代码
    Promise.resolve('foo')// 等价于new Promise(resolve => resolve('foo'))
    
####  参数情况

    a. 参数是一个promise实例直接返回对象
    b. 参数是一个thenable对象，立即执行then
        let thenable = {  then: function(resolve, reject) {    resolve(42);  }};let p1 = Promise.resolve(thenable);p1.then(function(value) {  console.log(value);  // 42});
    c. 参数不是具有then方法的对象，或根本就不是对象则转换为promise对象并且状态为resolve
    d. 不带任何参数，则返回为resolve
    
    
### 8. Promise.reject()

####  返回一个新的对象，状态为reject，并且把里面的内容当做reject的#### 理由
####  实例
    const p = Promise.reject('出错了');// 等同于const p = new Promise((resolve, reject) => reject('出错了'))p.then(null, function (s) {  console.log(s)});// 出错了
    
#### 参数是thenable会返回该对象

    const thenable = {  then(resolve, reject) {    reject('出错了');  }};Promise.reject(thenable).catch(e => {  console.log(e === thenable)})// true
    
### 9. 两个方法
#### done()

捕捉任何错误并且全局抛出
    
```
        Promise.prototype.done = function (onFulfilled, onRejected) {  this.then(onFulfilled, onRejected)    .catch(function (reason) {      // 抛出一个全局错误      setTimeout(() => { throw reason }, 0);    });};

```

#### finally()
    不管最后结果如何都会执行
        server.listen(0)  .then(function () {    // run test  })  .finally(server.stop);
        
### 10. 应用

#### 加载图片

    const preloadImage = function (path) {  return new Promise(function (resolve, reject) {    const image = new Image();    image.onload  = resolve;    image.onerror = reject;    image.src = path;  });};
    
#### 与generator函数结合
    遇到异步则返回一个promise对象
    
### 11. Promise.try()

####  将同步的函数同步，异步的函数异步执行的保证

```
    const f = () => console.log('now');Promise.try(f);console.log('next');// now// next
    
```

####  更好的管理异常

```
    Promise.try(database.users.get({id: userId}))  .then(...)  .catch(...)
```

 
![持续更新公众号](img/extremefruit.jpg)
![](img/extremefruit.jpg =100x100)