# JS面试宝典

## 简介说明：该宝典是整理各位大佬的面试题目，不定时更新。

## 整合大佬的面试题及自己对一些问题的答案，排名不分先后，感恩各位大佬。![img](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200530100627.jpg)

**掘金：** 【  **wangly19、**   **麦乐、**     **宅神king、**      **sunshine小小倩、**   **木易杨说、**    】



### 1、JavaScript的数据数据类型有哪些?

答： JavaScript钟分为基础类型和引用类型，分别是：

- **基本**类型：number，string，boolean，null，undefined
- **引用**类型: object，function，array

#### （1）堆和栈的区别

答：

- 其实深拷贝和浅拷贝的主要区别就是其在内存中的存储类型不同。

- 堆和栈都是内存中划分出来用来存储的区域。

  > 栈（stack）为自动分配的内存空间，它由系统自动释放；而堆（heap）则是动态分配的内存，大小不定也不会自动释放。

#### （2）基本类型

##### 	（2.1）基本数据类型存放在栈中

​		存放在栈内存中的简单数据段，数据大小确定，内存空间大小可以分配，是直接按值存放的，所以可以直接访问。

```javascript
var str = "abc";
console.log(str[1]="f");    // f
console.log(str);           // abc
```

​	**重点：** <font color='red'>**基本数据类型值不可变** </font>

##### 	（2.2）基本类型的比较是值的比较 [== 和 === 的区别 可参考 **16**<a href="#16、三个等号和两个等号的区别（“===”和“==”）">点击跳转</a>]

​		基本类型的比较是值的比较，只要它们的值相等就认为他们是相等的。

例如：

```javascript
var a = 1;
var b = 1;
console.log(a === b);//true
```

​	比较的时候最好使用严格等，因为 `==` 是会进行类型转换的

​	例如：

```javascript
var a = 1;
var b = true;
console.log(a == b);//true
```

#### （3）引用类型

##### 		(3.1)引用类型存放在堆中

​		引用类型（`object`）是存放在堆内存中的，变量实际上是一个存放在栈内存的指针，这个指针指向堆内存中的地址。每个空间大小不一样，要根据情况开进行特定的分配

例：

```javascript
var person1 = {name:'jozo'};
var person2 = {name:'xiaom'};
var person3 = {name:'xiaoq'};
```

##### 		（3.2）引用类型值可变

​		引用类型是可以直接改变其值的，例如：

```javascript
var a = [1,2,3];
a[1] = 5;
console.log(a[1]); // 5
```

##### 		（3.3）引用类型的比较是引用的比较

​		所以每次我们对 js 中的引用类型进行操作的时候，都是操作其对象的引用（保存在栈内存中的指针）。

​		所以比较两个引用类型，是看其的引用是否指向同一个对象。

例如：

```javascript
let a = [1,2,3];
let b = [1,2,3];
console.log('111', a == b, a === b); // false false

let c = {a: '1'};
let d = {a: '1'};
console.log('222', c == d, c === d); // false false
```



### 2、下面foo函数输出值是什么

```javascript
// try:  语句测试代码块的错误，一般把可能会出错的代码放到这里
// catch： 只有try里面的代码块发生错误时，才会执行这里的代码，参数err记录着try里面代码的错误信息
// finally: 无论有无异常里面代码都会执行

function foo(i) {
  try {
    if (i === 1) {
       // 会走到这一步
      return i
    }
  } catch (error) {
    return 0
    
  } finally {
     // 无论如何 最终都走这一步
    ++i
    return i
  }
}
console.log(foo(1)) // 2
```

### 3、给定一个数组，请你按照升序 & 降序的规则去排列它

```javascript
// 原始数组：[1, 4, 10, 2, 5, 18, 14]
sort( (a, b) => a - b) // 降序 // [1, 2, 4, 5, 10, 14, 18]
sort( (a, b) => b - a) // 倒叙  // [18, 14, 10, 5 ,4 , 2, 1]
```



### 4、作用域的理解  下面一段代码执行的结果是什么

```javascript
var name = 'zangsan';
(function () {
  console.log(name)
  name = 'Lisi'
  console.log(window.name)
  var name = 'wangwu'
  console.log(name)
})();
// underfined > zangsan // wangwu
```



### 5、 for 循环和 forEach循环的区别在于？

答： 

for 循环为原生语法糖，无上下文，而forEach则是Array上的方法。for 循环可以通过break， continue 实现中途退出循环，而forEach则无法打断循环。

### 6、执行下面的异步函数大约需要消耗多长时间?

```javascript
function speed() {
  return new Promise(resolve => {
    setTimeout(resolve, 3000)
  })
}

// @async function
async function foo() {
  let mySpeed = speed()
  await mySpeed
  await mySpeed
  await mySpeed
  await speed()
  await speed()
  await speed()
}

foo()
//答：约等于12S，但是时间超出12S。
```



### 7、请你实现一个简单的防抖函数?

```javascript
function debounce(func, time = 500) {
    let timer = null;
    return (...args) => {
        timer !== null && cleraTimerout(timer)
        timer =  setTimeout( () => {
          func.apply(this, args)  
        }, time)
    }
}
// @test function
function myText() {
  console.log('text......')
}

// @output
shake(myText, 3000)
```



### 8、localStorage、sessionStorage、cookie之间的区别?

答： 它们都是用来做数据持久化的存储方式

**cookie**: 存放在客户端，随着浏览器关闭，cookie将会丢失。存储大小为4K

**sessionStorage**: 只在本次会话有效，当浏览器关闭或者页面关闭后将会失效，存储大小为 > 4M

**localStorage**：存储在浏览器本地，除非手动清除或意外因素，一般不会丢失。存储大小为4M



### 9、实现一个函数，判断输入是不是回文字符串

```javascript
// 回文字符串  AACDBB 即 level  AABB 即 noon 
fucntion run(txt) {
    if (typeof txt !== 'string') return false;
    return txt.split('').reverse().join(',') === txt
}
```



### 10、实现效果，点击容器内的图标，图标边框变成border 1px solid red，点击空白处重置。

```javascript
const box = document.getElementById('box');
// 判断是否包含图标
function hasIcon(target) {
    return target.className.includes('icon')
}

box.onClick = fuction(e) {
    e.stopPropagation();
    const target = e.target;
    if( hasIcon(target) ) {
        target.style.border = '1px solid red'
    }
}

const doc = document;
doc.onClick = function(e) {
	const children = box.children;   
    for (let i; i < children.length; i++) {
        if( hasIcon(children[i]) ) {
            children[i].style.border = 'none'
        }
    }
}


```



### 11、事件流

DOM事件流分为三个阶段，一个是捕获节点，一个是处于目标节点阶段，一个是冒泡阶段。

```javascript
btn.addEventListener(eventType, function () {
}, false);
// 第一个参数为事件名
// 第二个为事件处理程序
// 第三个为布尔值，true为事件捕获阶段调用事件处理程序，false为事件冒泡阶段调用事件处理程序
```

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>事件冒泡</title>
</head>
<body>
    <div>
        <p id="parEle">我是父元素    <span id="sonEle">我是子元素</span></p>
    </div>
</body>
</html>
<script type="text/javascript">
var sonEle = document.getElementById('sonEle');
var parEle = document.getElementById('parEle');

parEle.addEventListener('click', function () {
    alert('父级 冒泡');
}, false);
parEle.addEventListener('click', function () {
    alert('父级 捕获');
}, true);

sonEle.addEventListener('click', function () {
    alert('子级冒泡');
}, false);
sonEle.addEventListener('click', function () {
    alert('子级捕获');
}, true);

</script>

// 当点击“我是子元素” 时，弹出的顺序是：“父级捕获”--》“子级冒泡”--》“子集捕获”--》“父集冒泡”；
// 当点击子元素时，父级的执行顺序是先捕获，后冒泡的。
```

- **preventDefault  取消事件默认行为,比如链接的跳转**

- **stopPropagation 取消事件冒泡**



### 12、什么是闭包

语义： **<font color='red'>闭包就是能够读取其他函数内部变量的函数</font>**



自我理解：**<font color='red'>定义在一个函数内部的函数，外部使用内部函数变量的函数</font>** 



**最通俗的理解**：

1. 函数a<font color='red'>**嵌套**</font>函数b
2. 函数b**使用**<font color='red'>父级函数a的变量或参数</font>



**闭包的特性**：闭包内使用的父级函数的变量或参数会永久**<font color='red'>保存</font>**



例： 

```javascript
function f1(){

　　　　var n=999;

　　　　nAdd=function(){n+=1}

　　　　function f2(){
　　　　　　alert(n);
　　　　}

　　　　return f2;

　　}

　　var result=f1();

　　result(); // 999

　　nAdd();

　　result(); // 1000

//原因就在于f1是f2的父函数，而f2被赋给了一个全局变量，这导致f2始终在内存中，而f2的存在依赖于f1，因此f1也始终在内存中，不会在调用结束后，被垃圾回收机制（garbage collection）回收。

//这段代码中另一个值得注意的地方，就是"nAdd=function(){n+=1}"这一行，首先在nAdd前面没有使用var关键字，因此nAdd是一个全局变量，而不是局部变量。其次，nAdd的值是一个匿名函数（anonymous function），而这个匿名函数本身也是一个闭包，所以nAdd相当于是一个setter，可以在函数外部对函数内部的局部变量进行操作。
```



### 13、如何判断两个数组是否相等

```javascript
   function ArrayIsEqual(arr1,arr2){//判断2个数组是否相等
    if(arr1===arr2){//如果2个数组对应的指针相同，那么肯定相等，同时也对比一下类型
        return true;
    }else{
        if(arr1.length!=arr2.length){
            return false;
        }else{//长度相同
            for(let i in arr1){//循环遍历对比每个位置的元素
                if(arr1[i]!=arr2[i]){//只要出现一次不相等，那么2个数组就不相等
                    return false;
                }
            }//for循环完成，没有出现不相等的情况，那么2个数组相等
            return true;
        }
    }
   }

   var arrone = [
        {op: "A", name: "暂时将委托人的营业执照副本代为保管", is_answer: 0, sureid: true},
        {op: "B", name: "签订“专有经纪权合同", is_answer: 1, sureid: true},
        {op: "C", name: "到工商行政管理机构进行合同公证", is_answer: 1, sureid: true}
   ]
   var arrtwo = [
        {op: "B", name: "签订“专有经纪权合同", is_answer: 1, sureid: true},
        {op: "C", name: "到工商行政管理机构进行合同公证", is_answer: 1, sureid: true},
        {op: "D", name: "预收佣金", is_answer: 1}
   ]
   
   
   console.log(ArrayIsEqual(arrone, arrtwo)) // false
```





### 14、0.1+0.2在三等情况下是否等于0.3

```js
   console.log( 0.1 + 0.2 === 0.3); // false
//其实这都是因为浮点数运算的精度问题。

// 简单来说，因为计算机只认识二进制，在进行运算时，需要将其他进制的数值转换成二进制，然后再进行计算。
```



### 15、undefined和null区别

答：

##### 相同点:

- 在 if判断语句中,值都默认为 false

##### 差异:

- null转为数字类型值为0,而undefined转为数字类型为 NaN(Not a Number)
- undefined是代表调用一个值而该值却没有赋值,这时候默认则为undefined
- null是一个很特殊的对象,最为常见的一个用法就是作为参数传入(说明该参数不是对象)
- 设置为null的变量或者对象会被内存收集器回收



​	undefined 表示一个变量自然的、最原始的状态值，而 null 则表示一个变量被人为的设置为空对象，而不是原始状态。

#### undefined:

##### 	**1、声明一个变量，但是没有赋值**

```javascript
var foo;

console.log(foo); // undefined

// 访问 foo，返回了 undefined，表示这个变量自从声明了以后，就从来没有使用过，也没有定义过任何有2效的值。
```

##### 	**2、访问对象上不存在的属性或者未定义的变量**

```javascript
console.log(Object.foo); // undefined

console.log(typeof demo); // undefined

// 访问 Object 对象上的 foo 属性，返回 undefined ， 表示Object 上不存在或者没有定义名为 foo 的属性；对未声明的变量执行typeof操作符返回了undefined值。

```

##### 	**3、函数定义了形参，但没有传递实参**

```javascript
// 函数定义了形参a

function fn(a) {

    console.log(a); // undefined

}

fn(); //未传递实参

函数 fn 定义了形参 a，但 fn 被调用时没有传递参数，因此，fn 运行时的参数 a 就是一个原始的、未被赋值的变量
```





### 16、三个等号和两个等号的区别（“===”和“==”）

答： 

1、===：称为等同符，当两边值的类型相同时，直接比较值，若类型不相同，直接返回false；

2、==：称为等值符，当等号两边的类型相同时，直接比较值是否相等，若不相同，则先转化为类型相同的值，再进行比较；

类型转换规则：

​	(1）如果等号两边是boolean、string、number三者中任意两者进行比较时，优先转换为数字进行比较。

  	(2）如果等号两边出现了null或undefined,null和undefined除了和自己相等，就彼此相等

注意：NaN==NaN  //返回false，NaN和所有值包括自己都不相等。

```javascript
let a = ['1'];
let b = 1;
console.log('111', a == b, a === b); // true false

let c = undefined;
let d = null;
console.log('222', c == d, c === d); // true false

let f = '1';
let g = 1;
console.log('333', f == g, f === g); // true false

let w = 1;
let e = true;
console.log('444', w == e, w === e);  // true false
```



### 17、合并两个数组

答： 

```javascript
let a = [1,2 ]
let b = [3,4]
// concat   
a = a.concat(b)
console.log(a); //  [1, 2, 3, 4]

// 扩展运算法
a = [...a, ...b] 
console.log(a); //  [1, 2, 3, 4]

// Array.prototype.push.apply(a, b)
a.push.apply(a, b);
console.log(a); //  [1, 2, 3, 4]
```



### 18、合并对象

答：

```javascript
let a = {a: 1}
let b = {b: 2}

// 扩展运算法
a = {...a, ...b}
console.log(a); // {a:1, b:2}

// Object.assign
a = Object.assign(a, b);
console.log(a); // {a:1, b:2}

```

### 19、求字符串数组的最长公共前缀

可参考[力扣的题](https://leetcode-cn.com/problems/longest-common-prefix)

> 编写一个函数来查找字符串数组中的最长公共前缀。
>
> 如果不存在公共前缀，返回空字符串 ""。
>
> 示例 1:
>
> 输入: ["flower","flow","flight"]
> 输出: "fl"
> 示例 2:
>
> 输入: ["dog","racecar","car"]
> 输出: ""
> 解释: 输入不存在公共前缀。

```javascript
var longestCommonPrefix = function(strs) {
    
    if(!strs.length) return "";
    let s = strs[0];
    let idx = 1;
    while(idx < strs.length){
        //判断s是否存在strs[idx]里面 并且 是以下标为0开始的
        //例如 s = "abc" ；strs[idx] = "aaaabc" 此时得到的值为3  不符合跳过的
        //或者 s = "abc" ；strs[idx] = "aaaacc" 此时得到的值为-1 不符合跳过的
        //当 s = "a" ；strs[idx] = "aaaabc" 此时得到的值为0 才跳过
        while(strs[idx].indexOf(s)){ 
            s = s.substr(0, s.length - 1);
            console.log('s', s);            
            if(!s.length) { return "没有公共" } //当最长公共前缀长度为0时
        }
        idx++;
    }
    return s;
}

let a = ["flower","flow","flight"];
let b = ["dog","racecar","car"]


console.log( longestCommonPrefix(a) ); // fl

console.log( longestCommonPrefix(b) ); // 没有公共
```



### 20、在JavaScript中，++在前和++在后有什么区别

答：

> ```
> 如果自加语句独立成为一个单独的语句，那么前后自加是完全相同的。
> 比如单独的一行
> a++;和++a;是一样的。
> 再比如，常见的for循环:
> for(let i = 0; i < 100; i ++)
> for(let i = 0; i < 100; ++i)
> 这里用到的++i和i++是完全相同的，没有区别。
> ```
>
>  
>
> ```
> 当运算变量本身值会在自加语句中，同时执行其它操作，二者就有区别了。
> 比如
> let i =0;
> while (i++<10); //11
> i会先和10比较大小，然后执行自加。这样当i=10时，退出循环，再执行一次自加，退出后i值为11。
> 而如果写成
> while``(++i<10); //10
> 是先执行自加，然后再与10比较。这样在i=9时，先自加，得到i=10,然后比较就会退出循环了。这种情况下，退出后i值为10。
> ```
>
>  
>
> ```
> //不出现赋值时，执行自增自减运算；出现赋值时，先赋值，后运算；
> let c;
> let d = 10;
> c = d++;
> console.log('c', c,d ); //10 11
> let e;
> let f = 10;
> e = ++f;
> console.log('e', e, f); //11 11
> ```



### 21、实现给数字添加千分位符的方法

- 正则表达式：`"12345678".replace(/(\d)(?=(?:\d{3})+$)/g,'$1,')`

- 数字分析， 取到整数部分， %1000 操作， 然后加逗号 拼接字符串 

  (12345678).toLocaleString("en-US") => "12,345,678"

- ```javascript
  字符串分析， 循环，然后3位加','
  function toThousands(number: number) {
    const result = [];
    let counter = 0;
    const numStr = (number || 0).toString().split('');
  
    for (let i = numStr.length - 1; i >= 0; i--) {
      counter++;
      result.unshift(numStr[i]);
      if (!(counter % 3) && i != 0) {
        result.unshift(',');
      }
    }
    return result.join('');
  }
  
  toThousands(123123131)
  ```

  

  

