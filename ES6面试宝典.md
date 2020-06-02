

# ES6面试宝典

## 简介说明：该宝典是整理各位大佬的面试题目，不定时更新。

## 整合大佬的面试题及自己对一些问题的答案，排名不分先后，感恩各位大佬

大佬名单：**掘金：** 【**wangly19**、**宅神king、**】



### 1、ES6中你常用的新特性有哪些? 你是怎么使用它的 ？

答：

1、 首先是新的块级变量定义方式, let & const， 在项目中我默认使用 `const` 来定义我的变量，只有涉及变量赋值时，我才会将其声明为 `let` 。

2、**模板字符串**： 其次是模板字符串，以前的时候都是采用 `+` 和 `toString` 来去做一些字符串的拼接，如 `URL` 。这样拼接后可读性非常的差。当ES6模板字符串出现后我就立马在项目中使用起来了。

3、**解构赋值**：

```javascript
let [a, b, c] = [1, 2, 3];
// a = 1
// b = 2
// c = 3

let [a, b, c, d, e] = 'hello';
// a = 'h'
// b = 'e'
// c = 'l'
// d = 'l'
// e = 'o'

let {a, b, ...rest} = {a: 10, b: 20, c: 30, d: 40};
// a = 10
// b = 20
// rest = {c: 30, d: 40}
```

4、**箭头函数**： 

箭头函数最直观的三个特点: 

- 不需要 function 关键字来创建函数
- 省略 return 关键字
- 继承当前上下文的 this 关键字

```javascript
// ES5
var add = function (a, b) {
    return a + b;
};
// 使用箭头函数
var add = (a, b) => a + b;

// ES5
[1,2,3].map((function(x){
    return x + 1;
}).bind(this));
    
// 使用箭头函数
[1,2,3].map(x => x + 1);
```

5、Promise和Async异步处理他解决了以前异步的回调地狱，让JavaScript的异步有了更好的体验。通常在Vue中数据请求都是通过Promise的形式来去做数据的异步获取的。

```javascript
var wait1000 =  new Promise((resolve, reject)=> {
  setTimeout(resolve, 1000);   
}).then(()=> {
  console.log('Yay!');
});
```

6、函数的参数默认值：

```javascript
// ES6之前，当未传入参数时，text = 'default'；
function printText(text) {
    text = text || 'default';
    console.log(text);
}

// ES6；
function printText(text = 'default') {
    console.log(text);
}

printText('hello'); // hello
printText();// default
```



### 2、let var const 区别

答： var声明的变量会挂到window上，而let和const不会

var声明的变量存在变量提升，而let和const不会

let和const声明形成块作用域，只能在块作用域里访问，不能跨块访问，也不能跨函数访问，不可重复声明

同一作用域下let和const不能声明同名变量，而var可以

暂时性死区，let和const声明的变量不能在声明前被使用



### 3、ES6之let（理解闭包）

例子1：

```javascript
var a=[];
        for(var i=0;i<10;i++){
            a[i]=function(){
                console.log(i);
            };
        }
        a[6](); //10
```

例子2：

```javascript
var a=[];
    for(let i=0;i<10;i++){
        a[i]=function(){
            console.log(i);
        };
    }
    a[6]();    //6
```

 

解答：

**例子1**：

```javascript
var a  = [];
var i=0;//由于var来声明变量i，所以for循环代码块不具备块级作用域，因此i认为是全局变量，直接放在全局变量中。

a[0] = function(){
    console.log(i) // 
}// 由于不具备块级作用域，所以该函数定义就是全局作用域。

var i=1;//第二次循环，这时var i=1;覆盖了前面的var i=0；即现在i为1;
a[1]=function(){
    console.log(i);//解释同a[0]函数。
}

var i=2;// 第三次循环，这时 i=2，在全局作用域中，所以覆盖了前面的i=1;
a[2]=function(){
    console.log(i);
}

......第四次循环 此时i=3  这个以及下面的i不断的覆盖前面的i，因为都在全局作用域中
......第五次循环 此时i=4
......第六次循环 此时i=5
......第七次循环 此时i=6
......第八次循环 此时i=7
......第九次循环 此时i=8   


var i=9;
a[9]=function(){
    console.log(i);
}


var i=10;// 这时i为10，因为不满足循环条件，所以停止循环。
紧接着在全局环境中继续向下执行。
a[6](); //这时调用a[6]函数，所以这时随即进入a[6]函数的执行环境
//即a[6]=function(){console.log(i)};执行函数中的代码 console.log(i); 因为在函数执行环境中不存在变量i; 此时就会沿着作用域链向上寻找，即找到a[10] 返回10
// 因为变量i 此时10 覆盖了之前的所有
// return 10
```

**例子2：**

**闭包的特性**：闭包内使用的父级函数的变量或参数会永久**<font color='red'>保存</font>**

```javascript
var a=[];//创建一个数组a;

// for循环块是块级作用域，形成了闭包，因为 内部函数使用外部函数的变量
let i=0; //注意：因为使用let使得for循环为块级作用域，此次let i=0在这个块级作用域中，而不是在全局环境中。
    a[0]=function(){
        console.log(i);
     }; //注意：由于循环时，let声明i,所以整个块是块级作用域，那么a[0]这个函数就成了一个闭包。

 let i=1; //注意：因为let i=1; 和 上面的let i=0;出在不同的作用域中，所以两者不会相互影响。
     a[1]=function(){
         console.log(i);
     }; //同样，这个a[i]也是一个闭包

......进入第三次循环，此时其中let i=2;
......进入第四次循环，此时其中let i=3;
......进入第五次循环，此时其中let i=4;
......进入第六次循环，此时其中let i=5;
......进入第七次循环，此时其中let i=6;
......进入第八次循环，此时其中let i=7;
......进入第九次循环，此时其中let i=8;

    let i=9;
    a[i]=function(){
        console.log(i);
    };//同样，这个a[i]也是一个闭包

  let i=10;//不符合条件，不再向下执行。于是这个代码块中不存在闭包，let i=10;在这次循环结束之后难逃厄运，随即被销毁。

a[6]();//调用a[6]()函数，这时执行环境随即进入下面这个代码块中的执行环境：funcion(){console.log(i)};
     let i=6;
     a[6]=function(){
          console.log(i);
     }; //同样，这个a[i]也是一个闭包
```

