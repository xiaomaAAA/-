# `javascript`查缺补漏整合及骚操作

## 简介说明：该文章是整理各位大佬的文章，不定时更新。

## 整合大佬的文章及自己平时遇到的问题，排名不分先后，感恩各位大佬。![img](https://gitee.com/supcoolMa/store_pictures/raw/master/img/0031ACAC.jpg)

大佬名单：**掘金：** 【  **前端森林、**  **陈大鱼头、**   】



### 1、flat() 、 flatMap()   

多维数组转一维

```javascript
   var arr1 = [1, 2, [3, 4, [5, [6, [7]]]]];
        // flat() 接受一个参数  
        // 如果不写默认为1 , 只能将 二维 转为 一维数组
        // Infinity 可直接将 多维 转为 一维
        let a1 = arr1.flat(Infinity);
        // return a1 ;  [1, 2, 3, 4, 5, 6, 7]

        // flatMap()  flatMap()只能展开一层数组 // 若多层，可以先flat() 
        // 相当于 [[2, 4], [3, 6], [4, 8]].flat()
        let a2 = [2, 3, 4].flatMap((x) => [x, x * 2])
        let a3 = [2, 3, 4].flatMap((x) => [ x * 2] )
        // return a2,  [2, 4, 3, 6, 4, 8]
        // return a2,  [4, 6, 8]
```



### 2、reducer()

#### 1、数组求和，求乘积

```javascript
var  arr = [1, 2, 3, 4];
var sum = arr.reduce((x,y)=>x+y)
var mul = arr.reduce((x,y)=>x*y)
console.log( sum ); //求和，10
console.log( mul ); //求乘积，24
```

#### 2、每个元素出现的次数

```javascript
let names = ['Alice', 'Bob', 'Tiff', 'Bruce', 'Alice'];

let nameNum = names.reduce((pre,cur)=>{
  if(cur in pre){
    pre[cur]++
  }else{
    pre[cur] = 1 
  }
  return pre
},{})
console.log(nameNum); //{Alice: 2, Bob: 1, Tiff: 1, Bruce: 1}
```



#### 3、将二维数组转化为一维

```javascript
let arr = [[0, 1], [2, 3], [4, 5]]
let newArr = arr.reduce((pre,cur)=>{
    return pre.concat(cur)
},[])
console.log(newArr); // [0, 1, 2, 3, 4, 5]
```



#### 4、将多维数组转化为一维

```javascript
let arr = [[0, 1], [2, 3], [4,[5,6,7]]]
const newArr = function(arr){
   return arr.reduce((pre,cur)=>pre.concat(Array.isArray(cur)?newArr(cur):cur),[])
}
console.log(newArr(arr)); //[0, 1, 2, 3, 4, 5, 6, 7]
```



#### 5、对象里的属性求和

```javascript
var result = [
    {
        subject: 'math',
        score: 10
    },
    {
        subject: 'chinese',
        score: 20
    },
    {
        subject: 'english',
        score: 30
    }
];

var sum = result.reduce(function(prev, cur) {
    return cur.score + prev;
}, 0);
console.log(sum) //60
```

### 3、指数运算符    【幂运算】

ES2016新增 （**）

```javascript
 2 ** 2  // 4
 2 ** 3  // 8
// 指数运算符可以与等号结合，形成一个新的赋值运算符（**=）
  let a = 1.5;
  a  **= 2;
// 等同于 a = a * a;

let b = 4;
 b **= 3;
// 等同于 b = b * b * b
```

### 4、Math.trunc() 去小数，保整数

将数字的小数部分去掉，只保留整数部分

`Math.trunc()` 的执行逻辑很简单，仅仅是**删除**掉数字的小数部分和小数点，不管参数是正数还是负数。

传入该方法的参数会被隐式转换成数字类型。

```javascript
Math.trunc(4.1)  // 4
Math.trunc(-4.1)  // -4
Math.trunc(-0.1123123)  // -0
                        对于'非数值'，Math.trunc内部使用Number方法将其先转为数值
Math.trunc('123.456')  // 123
                       对于'空值'和无法截取'整数'的值，返回NaN
Math.trunc(Nan)  // NaN
Math.trunc('foo')  // NaN
Math.trunc()  // NaN
```

### 5、Math.sign() 判断 数字 正负或0

此函数共有5种返回值, 分别是 **1, -1, 0, -0, NaN.** 代表的各是**正数, 负数, 正零, 负零, NaN**。

传入该函数的参数会被**隐式转换**成数字类型。

```javascript
Math.sign(-5) // -1
Math.sign(5)  // +1
Math.sign(0) //  0
Math.sign(-0) // -0
Math.sign(NaN) // NaN
Math.sign('9') // +1
Math.sign('foo') // Nan
```

### 6、 repeat() 字符串复制指定次数

```javascript
var str = "Runoob";
str.repeat(2); // RunoobRunoob
```

### 7、includes() 检查数组是否包含某个值

```javascript
 let arr = [1, 2, 3 ,4 ,5];
 arr.includes(5)   // true
```



### 8、find() 通过测试的数组的第一个元素的值

```javascript
 var ages = [3, 10, 18, 20];
	function checkAdult(age) {
    	  return age >= 18;
	} 
	
	// 18
```

### 9、 fill()  固定值填充数组 【数组的方法】

```javascript
// fill 方法使用给定值填充一个数组。
['a', 'b', 'c' ].fill(7)    //  [7, 7, 7]

new Array(3).fill(7)  //  [7, ,7, 7]

// 用于数组的初始化时非常方便。
// 数组中已有的元素会被全部抹去

// fill 方法还可以接受 第二个 和第三个参数
用于指定填充的起始位置和结束位置

['a', 'b', 'c'].fill(7, 1, 2)  // ['a', 7, 'c']
// fill()方法从1号位开始向原是数组填充7,到2号位之前结束。
```



### 10、Array.of() 和 Array()的区别

```javascript
// Array.of()  用于将一组值转换为数组
例： Array.of(3, 11, 6) // [3, 11, 6]
 Array.of(3)    //  [3]
 Array.of(3).length   //    1

// Array.of() 和 Array 构造函数之间的区别在于处理整数参数：
Array.of(3) 创建一个具有单个元素 3 的数组， //    [ 3 ]
Array(3) 创建一个长度为3的空数组  //    [,  ,   ,]
（注意：这是指一个有7个空位(empty)的数组，而不是由7个undefined组成的数组）。
'!!!'
Array()  //   [  ]    
Array(3) //   [, , ,]

Array.of 基本上可以代替Array()或者new Array(),
  并且不存在由于参数不同而导致的重载。行为非常统一

Array.of ()   //   []
Array.of(undefined)  //  [undefined]
Array.of(1)   //  [1]
Array.of(1, 2)  //  [1,  2]
Array.of 总是返回参数值组成的数组。
    如果没有参数，就放回一个空数组
```



### 11、 Array.from 达到 .map 的效果

![img](https://user-gold-cdn.xitu.io/2019/10/28/16e0fa883dfc8695?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 12、置空/清空 数组

有时候我们需要清空数组，一个快捷的方法就是直接让数组的 `length` 属性为 `0`，就可以清空数组了。

![img](https://user-gold-cdn.xitu.io/2019/10/28/16e0fa89a5b762dd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 13、将数组转换为对象

有时候，出于某种目的，需要将数组转化成对象，一个简单快速的方法是就使用展开运算符号(`...`):

![img](https://user-gold-cdn.xitu.io/2019/10/28/16e0fa8ac3dc7095?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 14、数组合并 【数据整合】

使用展开操作符，也可以将多个数组合并起来。

![img](https://user-gold-cdn.xitu.io/2019/10/28/16e0fa8d766b8ec9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 15、求两个数组的交集

求两个数组的交集在面试中也是有一定难度的正点，为了找到两个数组的交集，首先使用上面的方法确保所检查数组中的值不重复，接着使用`.filter` 方法和`.includes`方法。

如下所示：

![img](https://user-gold-cdn.xitu.io/2019/10/28/16e0fa8ec0988182?imageslim)



### 16、reverse() 数组反转

![img](https://user-gold-cdn.xitu.io/2019/10/28/16e0fa925550abc1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 17、lastIndexOf() 查找 最后出现的位置【下标】

```javascript
var str="I am from runoob，welcome to runoob site.";
var n=str.lastIndexOf("runoob");
// 28

var nums = [1, 5, 2, 6, 3, 5, 2, 3, 6, 5, 2, 7]
var lastIndex = nums.lastIndexOf(5); // 9
```

### 18、for…in和for…of的区别

**for in遍历的是数组的索引（即键名），而for of遍历的是数组元素值。**

```javascript
// for...of 用于遍历一个迭代器
let letters = ['a', 'b', 'c'];
for (let letter of letters) {
  console.log(letter); // 结果: a, b, c
}

// for...in 用来遍历对象中的属性：
let stus = ["Sam", "22", "男"];
 for (let index in stus) {
   console.log(index); // 1, ,2 , 3
   console.log(stus[index]); // 结果: Sam, 22, 男
  }

```



### 19、数组去重

#### 	（1） 利用ES6的set

```javascript
let arr = [1,2,3,3];
let resultarr = [...new Set(arr)]; 
console.log(resultarr); //[1,2,3]
```

#### 		（2）indexOf  判断 是否重复存在

```javascript
function uniq(arr) {
    let temp =  []; // 创建一个新的数组用来储存
    for(let i = 0; i < arr.length; i++ ) {
        if(temp.indexOf(arr[i]) == -1) {
            temp.push(arr[i])
        }
    }
    return temp
}
```

#### 	（3）indexOf 判断 下标 

​	**indexOf() 方法可返回某个指定的字符串值在字符串中首次出现的位置**

```javascript
function norepeat(arr){
	var newArr = [];
	for(var i = 0;i<arr.length;i++){
		if(newArr.indexOf(arr[i])==-1){
			newArr.push(arr[i]);
		}
	}
	return newArr;
}

```

#### （4）filter() 过滤

```javascript
let arr = [1,2,3,4,5,6,2,3,4,5,6,1,1,1,1]
arr.filter( (item,index) => arr.indexOf(item) === index)
```



### 20、性能追踪  **🐶**

​	如果你想测试一段`js`代码的执行耗时，那么你可以尝试下`performance`：

```javascript
let start = performance.now();
let sum = 0;
for (let i = 0; i < 100000; i++) {
  sum += 1;
}
let end = performance.now();
console.log(start);
console.log(end);
```

效果如下：

![1724ef0f67c65d72](https://gitee.com/supcoolMa/store_pictures/raw/master/img/1724ef0f67c65d72.gif)



### 21、取数组的最后一位 **🐶**

- **最常用**：

```javascript
let arr = [1, 2, 3, 4, 5]
arr[arr.length - 1] // 5
```

- **slice()** 

> ```javascript
> arrayObject.slice(start,end) // start 必带  end 可选
> ```

参数： **【  start】**必需带。规定从何处开始选取。如果是<font color='red'>**负数**</font>，那么它规定从数组尾部开始算起的位置。也就是说，**-1 指最后一个元素**，**-2 指倒数第二个元素**，以此类推。

```javascript
let arr = [1, 2, 3, 4, 5]
arr.slice(-1)[0] // 5
```



### 22、**美化你的`JSON `** **🐶**

日常开发中，我们会经常用到`JSON.stringify`，但大家可能并不大清楚他具体有哪些参数。

他有三个参数：

- `json`: 必须，可以是数组或`Object`
- `replacer`: 可选值，可以是数组，也可以是方法
- `space`: 用什么来进行分隔

而我们恰恰可以指定第三个参数`space`的值去美化我们的`JSON`：

![consoleJSon (https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200530083851.gif)](C:\Users\Administrator\Desktop\consoleJSon (1).gif)



### 23、拷贝数组  **🐶**

日常开发中，数组的拷贝是一个会经常遇到的场景。其实实现数组的拷贝有很多骚技巧。

##### （1）、`Array.slice`

```javascript
const arr = [1, 2, 3, 4, 5];
const copyArr = arr.slice();
```

##### （2）、展开操作符

```javascript
const arr = [1, 2, 3, 4, 5];
const copyArr = [...arr]
```

##### （3）、使用 `Array` 构造函数和展开操作符

```
const arr = [1, 2, 3, 4, 5];
const copyArr = new Array(...arr)
```

##### （4）、`Array.concat`

```javascript
const arr = [1, 2, 3, 4, 5];
const copyArr = arr.concat();
```



### 24、**避免多条件并列 🦀**

开发中有时会遇到多个条件，执行相同的语句，也就是多个`||`这种：

例：

```javascript
if (status === 'process' || status === 'wait' || status === 'fail') {
  doSomething()
}
```

这种写法语义性、可读性都不太好。可以通过 `switch case`  或 `includes` 这种进行改造。

##### （1）`switch case`

```javascript
switch(status) {
  case 'process':
  case 'wait':
  case 'fail':
    doSomething()
}
```

##### （2）inclueds

```javascript
const enum = ['process', 'wait', 'fail']
if (enum.includes(status)) {
  doSomething()
}
```



### 25、Array对象的扩展

##### （1）Array.from()` 

可参考[使用Array.from()遍历数组](#11、 Array.from 达到 .map 的效果)

`Array.prototype.from`：转换具有`Iterator接口`的数据结构为真正数组，返回新数组

```javascript	
console.log(Array.from('foo')) // ["f", "o", "o"]
console.log(Array.from([1, 2, 3], x => x + x)) // [2, 4, 6]
```

##### （2）Array.of()

可参考[Array.of() 和 Array()的区别](#10、Array.of() 和 Array()的区别)

`Array.prototype.of()`：转换一组值为真正数组，返回新数组。

```javascript
Array.of(7)       // [7] 
Array.of(1, 2, 3) // [1, 2, 3]

Array(7)          // [empty, empty, empty, empty, empty, empty]
Array(1, 2, 3)    // [1, 2, 3]
```

##### （3）`Array.prototype.copyWithin() `：把指定位置的成员复制到其他位置，返回原数组`

```javascrpit
const array1 = ['a', 'b', 'c', 'd', 'e']

console.log(array1.copyWithin(0, 3, 4)) // ["d", "b", "c", "d", "e"]

console.log(array1.copyWithin(1, 3)) // ["d", "d", "e", "d", "e"]
```

##### （4）`Array.prototype.find()` ：返回第一个符合条件的成员`

可参考[find() 通过测试的数组的第一个元素的值](#8、find() 通过测试的数组的第一个元素的值)

```javascript
const array1 = [5, 12, 8, 130, 44]
const found = array1.find(element => element > 10)
console.log(found) // 12
```

##### （5）`Array.prototype.findIndex()`：返回第一个符合条件的成员索引值

```javascript
const array1 = [5, 12, 8, 130, 44]

const isLargeNumber = (element) => element > 13

console.log(array1.findIndex(isLargeNumber)) // 3
```

##### （6）`Array.prototype.fill()`：根据指定值填充整个数组，返回原数组

可参考[ fill()  固定值填充数组 【数组的方法】](#9、 fill()  固定值填充数组 [数组的方法])

```javascript
const array1 = [1, 2, 3, 4]

console.log(array1.fill(0, 2, 4)) // [1, 2, 0, 0]

console.log(array1.fill(5, 1)) // [1, 5, 5, 5]

console.log(array1.fill(6)) // [6, 6, 6, 6]
```

##### （7）Array.prototype.keys()`：返回以索引值为遍历器的对象

```javascript
const array1 = ['a', 'b', 'c']
const iterator = array1.keys()

for (const key of iterator) {
      console.log(key)
}

// 0
// 1
// 2
```

##### （8）`Array.prototype.values()`：返回以属性值为遍历器的对象

```javascript
const array1 = ['a', 'b', 'c']
const iterator = array1.values()

for (const key of iterator) {
      console.log(key)
}

// a
// b
// c
```

##### （9）`Array.prototype.entries()`：返回以索引值和属性值为遍历器的对象

```javascript
const array1 = ['a', 'b', 'c']
const iterator = array1.entries()

console.log(iterator.next().value) // [0, "a"]
console.log(iterator.next().value) // [1, "b"]
```

##### （10）数组空位`：ES6明确将数组空位转为`undefined`或者`empty

```javascript
Array.from(['a',,'b']) // [ "a", undefined, "b" ]
[...['a',,'b']] // [ "a", undefined, "b" ]
Array(3) //  [empty × 3]
[,'a'] // [empty, "a"]
```



### 26、隐藏电话号码 `padStart()` `padEnd()`

##### （1）`padStart()`

​	`padStart()` 方法用另一个字符串填充当前字符串(重复，如果需要的话)，以便产生的字符串达到给定的长度。填充从当前字符串的开始(左侧)应用的。

```javascript
const str1 = '5'
console.log(str1.padStart(2, '0')) // "05"

const fullNumber = '2034399002125581'
const last4Digits = fullNumber.slice(-4) // 保留后四位
const maskedNumber = last4Digits.padStart(fullNumber.length, '*') 
console.log(maskedNumber) // "************5581"

```

##### （2）`padEnd()`

​	`padEnd()` 方法会用一个字符串填充当前字符串（如果需要的话则重复填充），返回填充后达到指定长度的字符串。从当前字符串的末尾（右侧）开始填充。

```javascript
const str1 = 'Breaded Mushrooms' //  17个字符
console.log(str1.padEnd(25, '.')) // "Breaded Mushrooms........" // 填充到25个字符
const str2 = '200'
console.log(str2.padEnd(5)) // "200  " 长度为5
```

