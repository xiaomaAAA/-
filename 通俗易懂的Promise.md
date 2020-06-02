# 通俗易懂的Promise

## 简介说明：该文章是整理各位大佬的文章，后续会更新手写promise。

## 整合大佬的文章及自己的理解，排名不分先后，感恩各位大佬。![img](https://gitee.com/supcoolMa/store_pictures/raw/master/img/0031ACAC.jpg)

大佬名单：

​	**掘金：** 【  **蔓蔓雒轩、**   **lucefer、**  **石墨文档、** **卡梅隆、**   **OBKoro1、**  **艾特老干部、**】

​	**简书**： 【   **李悦之、** 】



### 1、为啥子需要 Promise的出现

#### （1）回调地狱

> 在 Promise 出现以前，我们处理一个异步网络请求。一旦需求变化了，我们需要根据第一个网络请求的结果，再去执行第二个网络请求。一个两个其实还好，但需求是永无止境的，就会形成以下代码，【 **回调地狱**】

```javascript
请求1(function(请求结果1){
    请求2(function(请求结果2){
        请求3(function(请求结果3){
            请求4(function(请求结果4){
                请求5(function(请求结果5){
                    请求6(function(请求结果3){
                        ...
                    })
                })
            })
        })
    })
})
```

> 回调地狱带来的负面作用有以下几点：
>
> - 代码臃肿。
> - 可读性差。
> - 耦合度过高，可维护性差。
> - 代码复用性差。
> - 容易滋生 bug。
> - 只能在回调里处理异常。



### 2、promise的使用

常规使用promise:

```javascript
new Promise(请求1)
    .then(请求2(请求结果1))
    .then(请求3(请求结果2))
    .then(请求4(请求结果3))
    .then(请求5(请求结果4))
    .catch(处理异常(异常信息))
```

### 3、promise API

##### 	（1）	`resolve, reject`

> 类方法，且与 resolve 唯一的不同是，返回的 promise 对象的状态为 rejected。
>
> ```javascript
>    let p = new Promise((resolve, reject) => {
>         //做一些异步操作
>       setTimeout(function(){
>             var num = Math.ceil(Math.random()*10); //生成1-10的随机数
>             if(num<=5){
>                 resolve(num); // 成功
>             }
>             else{
>                 reject('数字太大了'); // 失败
>             }
>       }, 2000);
>     });
> ```
>
> 

##### 	（2）`Promise.prototype.then`

> 实例方法，为 Promise 注册回调函数，函数形式：`fn(vlaue){}`，value 是上一个任务的返回结果，then 中的函数一定要 return 一个结果或者一个新的 Promise 对象，才可以让之后的then 回调接收。

结合上面的例子：

```javascript
    let p = new Promise((resolve, reject) => {
        //做一些异步操作
      setTimeout(function(){
            var num = Math.ceil(Math.random()*10); //生成1-10的随机数
            if(num<=5){
                resolve(num);
            }
            else{
                reject('数字太大了');
            }
      }, 2000);
    });
    p.then((data) => {
            console.log('resolved',data); // 5
        },(err) => {
            console.log('rejected',err); // 数字太大了
        }
    ); 
```

##### 	（3）`Promise.prototype.catch`

> 实例方法，捕获异常，函数形式：`fn(err){}`, err 是 catch 注册 之前的回调抛出的异常信息。

```javascript
p.then((data) => {
    console.log('resolved',data);
}).catch((err) => {
    console.log('rejected',err); // 失败的打印结果
});
```

##### 	（4）`Promise.race`

> 类方法，多个 Promise 任务同时执行，返回最先执行结束的 Promise 任务的结果，不管这个 Promise 结果是成功还是失败 。

**race的用法：谁跑的快，以谁为准执行回调**

race的使用场景：比如我们可以用race给某个异步请求设置超时时间，并且在超时后执行相应的操作，代码如下：

```javascript
let p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('success')
  },1000)
})

let p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject('failed')
  }, 500)
})

Promise.race([p1, p2]).then((result) => {
  console.log(result)
}).catch((error) => {
  console.log(error)  // 打开的是 'failed' 因为p2的执行会快一点
})
```

##### 	（5）`Promise.all`

> 类方法，多个 Promise 任务同时执行。
> 如果全部成功执行，则以数组的方式返回所有 Promise 任务的执行结果。 如果有一个 Promise 任务 rejected，则只返回 rejected 任务的结果。

**all的使用**：**Promise.all可以将多个Promise实例包装成一个新的Promise实例。同时，成功和失败的返回值是不同的，成功的时候返回的是一个结果数组，而失败的时候则返回最先被reject失败状态的值。**

```javascript
let wake = (time) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(`${time / 1000}秒后醒来`)
    }, time)
  })
}

let p1 = wake(3000)
let p2 = wake(2000)

Promise.all([p1, p2]).then((result) => {
  console.log(result)       // [ '3秒后醒来', '2秒后醒来' ]
}).catch((error) => {
  console.log(error)
})
```

**注**：	需要特别注意的是，`Promise.all`获得的成功结果的数组里面的数据顺序和`Promise.all`接收到的数组顺序是一致的，即p1的结果在前，即便p1的结果获取的比p2要晚。

​	这带来了一个绝大的好处：在前端开发请求数据的过程中，偶尔会遇到发送多个请求并根据请求顺序获取和使用数据的场景，使用`Promise.all`毫无疑问可以解决这个问题。

##### 	（6）`Promise.finally()`

> finally方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。该方法是 ES2018 引入标准的

`finally`方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。它与`done`方法的最大区别，它接受一个普通的回调函数作为参数，该函数不管怎样都必须执行。

以上面的例子补充：

```javascript
Promise.all([p1, p2]).then((result) => {
  console.log(result)       // [ '3秒后醒来', '2秒后醒来' ]
}).catch((error) => {
  console.log(error)
}).finally(function(){
    console.log('最后执行结束啦');
});
```



### 4、promise 题目

promise的题目多半都是设计到**事件循环**。

##### 例1： 

```javascript
console.log('start');
setTimeout(()=>{
    console.log('A');
},1000);
console.log('end');
//start
//end
//A

console.log('start');
setTimeout(()=>{
    console.log('A');
},0);
console.log('end');
//start
//end
//A
```

##### 例2：结合promise

```javascript
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('resolved.');
});

console.log('Hi!');
// Promise
// Hi!
// resolved
```

**注：**上面代码中，Promise 新建后**立即执行**，所以首先输出的是`Promise`。然后，`then`方法指定的回调函数，将在当前脚本**所有同步任务执行完才会执行**，所以`resolved`最后输出。

##### 例3： 

```javascript
console.log(1);
setTimeout(function(){
  console.log(2);
}, 0);
Promise.resolve().then(function(){
  console.log(3);
}).then(function(){
  console.log(4);
});

// 1
// 3
// 4
// 2
```

**注：** **Promise**的函数代码的异步任务会**优先**于`setTimeout`的延时为0的任务先执行。

##### 例子4:

```javascript
   // 执行顺序问题，考察频率挺高的，先自己想答案**
    setTimeout(function () {
        console.log(1);
    });
    new Promise(function(resolve,reject){
        console.log(2)
        resolve(3)
    }).then(function(val){
        console.log(val);
    })
    console.log(4);

// 2
// 4
// 3
// 1
```

**注：** 

1. 先执行`script`同步代码

   > ```javascript
   >  先执行new Promise中的console.log(2),then后面的不执行属于微任务
   >  然后执行console.log(4)
   > ```

2. 执行完`script`宏任务后，执行微任务，console.log(3)，没有其他微任务了。

3. 执行另一个宏任务，定时器，console.log(1)。

根据本文的内容，可以很轻松，且有理有据的猜出写出正确答案：2,4,3,1.

##### 例子5： Promise 构造函数是同步执行的，`promise.then` 中的函数是异步执行的。

```javascript
const promise = new Promise((resolve, reject) => {
  console.log(1)
  resolve()
  console.log(2)
})
promise.then(() => {
  console.log(3)
})
console.log(4)

// 1
// 2
// 4
// 3
```

##### 例子6：**promise 状态一旦改变则不能再变**

```javascript
const promise = new Promise((resolve, reject) => {
  resolve('success1')
  reject('error')
  resolve('success2')
})

promise
  .then((res) => {
    console.log('then: ', res)
  })
  .catch((err) => {
    console.log('catch: ', err)
  })

// then: success1
```

**注：** 构造函数中的 resolve 或 reject 只有第一次执行有效，多次调用没有任何作用，



##### 例子7：Promise的立即执行性

```javascript
var p = new Promise(function(resolve, reject){
  console.log("create a promise");
  resolve("success");
});

console.log("after new Promise");

p.then(function(value){
  console.log(value);
});

// create a promise
// after new Promise
// success
```



### 5、手写promise 【还没完成 =.=]