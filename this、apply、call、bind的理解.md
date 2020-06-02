# this、apply、call、bind的理解

参考原作者文章：掘金【sunshine小小倩】https://juejin.im/post/59bfe84351882531b730bac2



## this 的指向

### 	重点理解：**this 永远指向最后调用它的那个对象**

​		只要理解了这句话，就能明白以下代码

例子1： 

```javascript
function hello(){
    console.log(this) // window
}
hello()
```

**注**： 调用hello()位于顶层作用域，即this将会指向最顶层的window； 等同于`window.hello()` ；

例子2： 

```javascript
var name = 'windowsName';
function fn() {
    var name = 'lisa'
    console.log(this.name) // windowsName
}
fn();
```

**注**： 调用`fn()`位于顶层作用域，即this将会指向最顶层的window； 等同于`window.fn()` ；所以打印的name是`winodow`的name 即 `windowsName`;

例子3： 

```javascript
var name = 'windowsName';
var a = {
    name = 'lisa';
    fn: function () {
        console.log(this.name) // lisa
    }
} 
a.fn();
```

**注**： 函数 fn 是对象 a 调用的，所以打印的值就是 a 中的 name 的值



只要记住  **this 永远指向最后调用它的那个对象** 就可以很好的理解this 的指向。是不是有一点清晰了呢~

现在提高点难度，看看是否真正理解**this 的指向**。

### 进阶：

在基础上做些小改动

例子1：

```javascript
var name = 'windowsName';
var a = {
    name = 'lisa';
    fn: function () {
        console.log(this.name) // lisa
    }
}
window.a.fn()
```

**注**：只要记住最后调用的函数 就很好理解为什么会打印`lisa`，因为最后调用的对象仍然是对象a

例子2：

```javascript
var name = 'windowsName';
var a = {
    // name = 'lisa';
    fn: function () {
        console.log(this.name) // undefined
    }
}
window.a.fn()
```

**注**：理解上个例子后就能很好理解这个，因为对象a 里面没有对name进行定义，所以 打印的结果 是 `undefined`

例子3：

```javascript
var name = "windowsName";
var a = {
    name: "Cherry",
    fn : function () {
        console.log(this.name);      // windowsName
     }
}

var f = a.fn;
f();
```

**注**：由于刚刚的 f 并没有调用，所以 `fn()` 最后仍然是被 window 调用的。所以 this 指向的也就是 window

#### 匿名函数的指向理解

> 在 JavaScript 中, 函数是对象。
>
> JavaScript 函数有它的属性和方法。
>
> 在 JavaScript 非严格模式(non-strict mode)下, 如果第一个参数的值是 null 或 undefined, 它将使用全局对象替代。

##### 重点理解： **匿名函数的 this 永远指向 window**

例子4：

```javascript
var name = "windowsName";
function fn() {
    var name = 'lisa';
    innerFn();
    function innerFn() {
        console.log(this.name) // windowsName
    }
}
fn();
```

**注**：读到现在了应该能够理解这是为什么了吧(oﾟ▽ﾟ)o。

### 毕业：

​	许多现在许多 js 框架的设计都是使用`构造函数`方式设计的。例如下面构造函数：

例子1：

```javascript
function Cat(name, type) {
	this.name = name;
	this.type = type;
	console.log(this)
}

var a = new Cat("奥利奥","英短")
```

运行结果如下图所示：

<img src="https://gitee.com/supcoolMa/store_pictures/raw/master/img/image-20200529154451618.png" align="left"/>

**注**：构造函数首先函数名称`首字母都会大写`，其次都会使用一个 `new` 来构造一个实例,new 的时候 构造函数就会运行一次，这个时候构造函数里的this就指向这个实例对象。



​	上述构造函数只是实现了一些属性，其实更多的时候构造函数内部应该根据传入的值来实现一些方法，从而体现良好的封装性。例如使得上述猫类增加一个方法:

例子2：

```javascript
function Cat(name, type) {
	this.name = name;
	this.type = type;
	this.say = function(){
    	console.log(`我是一只${this.type}的猫崽，我的名字叫做${this.name}`)
  	}
}

let a = new Cat("奥利奥","英短")
a.say() // 我是一只英短的猫崽，我的名字叫做奥利奥
```

**注**：先实例出来一个对象 `a `,再调用  `a `的 say 方法,say 方法内部可以获取到 name 和 type。

例子3：

```javascript
function Cat(name, type) {
	this.name = name;
	this.type = type;
	this.say = function(){
    	console.log(`我是一只${this.type}的猫崽，我的名字叫做${this.name}`)
  	}
}

let a = new Cat("奥利奥","英短")
let sayName = a.say
sayName() // 我是一只undefined的猫崽，我的名字叫做
```

**注**：变量`sayName`只是定义了a的say方法，并没有执行。而执行的`sayName`已经在全局window上，但此时全局并没有定义type和name,所以就成 undefined了。



## 改变this指向

##### 	改变 this 的指向我总结有以下几种方法：	

- **使用 ES6 的箭头函数**
- **在函数内部使用 `_this = this`**
- **使用 `apply`、`call`、`bind`**
- **new 实例化一个对象**



### 箭头函数

​	众所周知，ES6 的箭头函数是可以避免 ES5 中使用 this 的坑的。**箭头函数的 this 始终指向函数定义时的 this，而非执行时**。如果箭头函数被非箭头函数包含，则 this 绑定的是最近一层非箭头函数的 this，否则，this 为 undefined”。

#### 重点理解： 箭头函数的this指向本身

例子1：

```javascript
var name = 'windowsName'

var a = {
    name: 'lisa',
    fn1: function () {
        console.log(this.name)
    },
	fn2: function() {
        setTimeout( () => {
            this.fn1()
        }, 100)
    }
}
a.fn2()  //  'lisa'
```

例子 2：

```javascript
function Cat(name, type) {
	this.name = name;
	this.type = type;
	this.say = () => {
    	console.log(`我是一只${this.type}的猫崽，我的名字叫做${this.name}`)
  	}
}

let a = new Cat("奥利奥","英短")
let sayName = a.say
sayName() // 我是一只英短的猫崽，我的名字叫做奥利奥
```

例子3： 

```javascript
var x=11;
    //这是say2父类obj所在的作用域 
var obj={
	x:22,
	say1:function(){
	  console.log(this.x)
	},
    
    // 这里是say2存在的作用域 
	say2:()=>console.log(this.x)
}
obj.say1(); // 22
obj.say2(); // 11
```

**注**： `obj.say1` 的this指向是对象`obj`，对于`say2`来说,此时 say2这一行,看做this.say2,那么say2的父作用域是谁呢,他的父是`obj`**,父类所在的作用域**是window,也就是最外面的作用域啊,所以在这里输出是11



毕业季：

例子1:

```javascript
var a = 11
 
function test1(bb){
	this.a = bb;
	this.b=function(){
	   console.log(`b:${this.a}`);
	};
	this.b();
	let c = function(){console.log(`c:${this.a}`)}
	c();
	let d = () => console.log(`d:${this.a}`); 
	d();
}
var x=new test1(15);  // b:15 c: 11  d:15
```



如果明白这个例子，箭头函数就毕业啦。 接下来是 **call、apply、bind**了， 冲鸭



## 使用 apply、call、bind

可参考文档：https://www.runoob.com/w3cnote/js-call-apply-bind.html

使用 apply、call、bind 函数也是可以改变 this 的指向的，原理稍后再讲，我们先来看一下是怎么实现的：

### call 主动执行

 [w3scholl对call的解释](https://www.w3school.com.cn/js/js_function_call.asp)

例子：

```javascript
var a = {
    name: 'lisa',
    fn: fuction () {
      console.log(this.name) // lisa
	}
}
var b = a.fn;
b.call(a);

多个参数用法
var person = {
  fullName: function(type, country) {
    console.log(`我是一只来自${country}${type}的${this.cat}${this.sex}`) 
  }
}
var person1 = {
  sex:"弟弟",
  cat: "猫"
}
person.fullName.call(person1, "短毛", "英国"); // 我是一只来自英国短毛的猫弟弟
```

**注:**  通过在call方法，给第一个参数添加要把b添加到哪个环境中，简单来说，this就会指向那个对象。



### apply  主动执行

[w3scholl对apply的解释](https://www.w3school.com.cn/js/js_function_apply.asp)

例子:

```javascript
var a = {
    name: 'lisa',
    fn: fuction () {
      console.log(this.name) // lisa
	}
}
var b = a.fn;
b.apply(a);

同样apply也可以有多个参数，但是不同的是，第二个参数必须是一个数组
var person = {
  fullName: function(type, country) {
    console.log(`我是一只来自${country}${type}的${this.cat}${this.sex}`) 
  }
}
var person1 = {
  sex:"弟弟",
  cat: "猫"
}
person.fullName.apply(person1, ["短毛", "英国"]); // 我是一只来自英国短毛的猫弟弟
```



### bind  被动执行

> `bind()` 方法创建一个新的函数，在 `bind()` 被调用时，这个新函数的 `this` 被指定为 `bind()` 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。

[w3scholl对apply的解释](https://www.w3school.com.cn/js/js_function_apply.asp)

#### 重点理解：call、apply都是主动执行 而bind需要被调用

例子： 

```javascript
var a = {
    name: 'lisa',
    fn: fuction () {
      console.log(this.name) // lisa
	}
}
var b = a.fn;
var c = b.bind(a); // 指向调用的函数
c(); // 需要在多调用一次 不然不会执行

同样bind也可以有多个参数，并且参数可以执行的时候再次添加，但是要注意的是，参数是按照形参的顺序进行的。
var person = {
  fullName: function(type, country) {
    console.log(`我是一只来自${country}${type}的${this.cat}${this.sex}`) 
  }
}
var person1 = {
  sex:"弟弟",
  cat: "猫"
}
let a = person.fullName.bind(person1, "短毛", "英国"); 
a(); // 我是一只来自英国短毛的猫弟弟

也可以写成 等同上方
var person = {
  fullName: function(type, country) {
    console.log(`我是一只来自${country}${type}的${this.cat}${this.sex}`) 
  }
}
var person1 = {
  sex:"弟弟",
  cat: "猫"
}
let a = person.fullName.bind(person1,); 
a("短毛", "英国" );
```

#### ** react 方法调用 常用到bind

```react
组件方法的传递
 scrollLeft() {
    if (this.scrollTopRef.current.state.leftPosition > 0) {
      if (this.timer2) clearTimeout(this.timer2);
      this.timer2 = setTimeout(() => {
        this.scrollTopRef.current.scrollArea.scrollXTo(0);
      }, 10 * 1000);
    }
  }

  <Scroll update={this.scrollLeft.bind(this)} />
```

