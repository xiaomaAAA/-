# React宝典

## 简介说明：该宝典是整理各位大佬的面试题目，不定时更新。

## 整合大佬的面试题及自己对一些问题的答案，排名不分先后，感恩各位大佬

大佬名单：**掘金：** 【**peen**、**麦乐**丶】

### 1、高阶组件(HOC) , render props 以及 hook 的对比和用处

#### （1）、学习 HOC 我们只需要记住以下 2 点定义:

1. 创建一个函数, 该函数接收一个组件作为输入*除了组件, 还可以传递其他的参数*
2. 基于该组件返回了一个不同的组件

```react
function withSubscription(WrappedComponent, selectData) {
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.state = {
        data: selectData(DataSource, props)
      };
    }
    // 一些通用的逻辑处理
    render() {
      // ... 并使用新数据渲染被包装的组件!
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}

```

#### （2）HOC 的优点: 不会影响内层组件的状态, 降低了耦合度

#### （3）HOC 的缺点：固定的 props 可能会被覆盖.



### 2、虚拟 DOM 是什么?

答：

- 在 React 中, React 会先将代码转换成一个 JS 对象, 然后再将这个 JS 对象转换成真正的 DOM. 这个 JS 对象就是所谓的虚拟 DOM。
- 它可以让我们无须关注 DOM 操作, 只需要开心地编写数据,状态即可。



### 3、JSX做表达式判断时候，需要强转为boolean类型

```react
render() {
  const b = 0;
  return <div>
    {
      !!b && <div>这是一段文本</div>
    }
  </div>
}
// 如果不使用 !!b 进行强转数据类型，会在页面里面输出 0。
```



### 4、react div随鼠标移动



```react
import React from 'react'

export default class extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            translateX: 0,
            translateY: 0,
        };
        this.moving = false;
        this.lastX = null;
        this.lastY = null;
        window.onmouseup = e => this.onMouseUp(e);
        window.onmousemove = e => this.onMouseMove(e);
    }

    onMouseDown(e) {
        e.stopPropagation();
        this.moving = true;
    }

    onMouseUp() {
        this.moving = false;
        this.lastX = null;
        this.lastY = null;
    }

    onMouseMove(e) {
        this.moving && this.onMove(e);
    }

    onMove(e) {
        if(this.lastX && this.lastY) {
            let dx = e.clientX - this.lastX;
            let dy = e.clientY - this.lastY;
            this.setState({ translateX: this.state.translateX + dx, translateY: this.state.translateY + dy })
        }
        this.lastX = e.clientX;
        this.lastY = e.clientY;
    }

    render() {
        return (
            <div
                onMouseDown={e => this.onMouseDown(e)}
                style={{transform: `translateX(${this.state.translateX}px)translateY(${this.state.translateY}px)`}}
            >
                <div style={{ width: 100, height: 100, backgroundColor: 'blue' }} />
            </div>
        )
    }
};
```

**注：**

1. onMouseDown事件由div提供，onMouseMove和onMouseUp事件由window提供的原因是防止鼠标移动过快造成事件丢失
2. this.moving为记录移动开始标志
3. stopPropagation作用为终止事件在传播过程的捕获、目标处理或起泡阶段进一步传播
4. div的实际移动由css3属性transform: { translateX, translateY }提供
5. 高阶组件封装的github项目链接[react-drag-hoc](https://github.com/yjy5264/react-drag-hoc)

转载：<https://segmentfault.com/a/1190000013036635>



### 5、防抖与节流的区别

防抖是**<font color='red'>延迟执行</font>**，而节流是<font color='red'>**间隔执行**</font>。

函数节流即每隔一段时间就执行一次，实现原理为设置一个定时器，约定xx毫秒后执行事件，如果时间到了，那么执行函数并重置定时器。

和防抖的区别在于，**<font color='red'>防抖</font>**每次触发事件都重置定时器，而<font color='red'>**节流**</font>在定时器到时间后再清空定时器

### 6、定义一个防抖函数

防抖是**<font color='red'>延迟执行</font>**，**<font color='red'>防抖</font>**每次触发事件都重置定时器

使用场景：

- 每次 resize/scroll 触发统计事件
- 文本输入的验证（连续输入文字后发送 AJAX 请求进行验证，验证一次就好）

```react
/**
   * 防抖函数
   * @param handler
   * @param interval
   * @returns {Function}
   */
  static debounce(handler, interval = 500) {
    let timer;
    return (...args) => {
      const currentTime = Date.now();
      if (timer && currentTime - timer < interval) return clearTimeout(timeout);
      timeout = setTimeout(() => {
            handler(...args);
        }, interval);
    };
  }

// 调用例子： 
  @action lessen = Utils.debounce(() => {
    if (this.zoom < .1) return;
    this.update({
      zoom: this.zoom - .1,
    });
  });
```



### 7、定义一个节流函数

使用两个时间戳prev旧时间戳和now新时间戳，每次触发事件都判断二者的时间差，如果到达规定时间，执行函数并重置旧时间戳

使用场景： 

- DOM 元素的拖拽功能实现（mousemove）
- 射击游戏的 mousedown/keydown 事件（单位时间只能发射一颗子弹）
- 计算鼠标移动的距离（mousemove）
- Canvas 模拟画板功能（mousemove）
- 搜索联想（keyup）

监听滚动事件判断是否到页面底部自动加载更多：给 scroll 加了 debounce 后，只有用户停止滚动后，才会判断是否到了页面底部；如果是 throttle 的话，只要页面滚动就会间隔一段时间判断一次



### 8、ref的使用

#### （1）组件直接使用  或者  class组件 使用 ref

```react
constructor(props) {
super(props)
this.inpRef = React.createRef();
}

取值：  this.inpRef.current.value

<input type='text' ref = {this.inpRef}>
```

**class组件** 

```react
export default class Child extends React.Component {
    constructor(props) {
        super(props)
    }
    render(){
        return <div ref={this.props.myRef}>子组件</div>
    }
}
Class Parent extends Component {
	 constructor(props){
        super(props);
        this.myChild = React.createRef();
    }
     componentDidMount(){
        console.log(this.myChild.current);//获取的是Child组件
    }
    render() {
        return <Child ref={this.myChild} />
    }
}

export default Son;
```



#### （2）hook  方法 useRef()

```react
function CityInp() {
	const cityInpRef = useRef(null)
    return (
        <>
            <input type='text' ref={cityInpRef } />
            <button onclick=(() => {
                let val = cityInpRef.current.value
            })  />
        </>
    );

}
```



#### （3）function组件  转发 React.forward

```react
const AgeRef  = React.forwardRef(({ className, ...props }, ref) => {
  return <AgeRef ref = {this.ageInlutRef} />;
});

function AgeInput(props, ref) {
    return <input type='text' ref= {ref}/>;
}
```

```react
import React,{Component} from 'react';
import ReactDOM,{render} from 'react-dom';
const Child = React.forwardRef((props,ref)=>{
  React.useImperativeHandle(ref, () => ({
    clickBtn,
  })
  function clickBtn() {
    alert('自定义的方法')
  }
// props.txt 拿到的是 父组件的文本 =》 爸爸在此
// 绑定ref 
    return (
        <div ref={ref}>{props.txt}</div>
    )
})

class Parent extends Component{
    constructor(props){
        super(props);
        this.myChildRef = React.createRef();
    }
    componentDidMount(){
        console.log(this.myChildRef.current);//获取的是Child组件中的div元素
    }
    render(){
        return (
            <div>
                  <div onClick={() => {
                        myChildRef.current.clickBtn();
                      }}>使用子组件的自定义方法</div>
                  <Child ref={this.myChildRef} txt="爸爸在此"/>
            </div>
		)
    }
    
}
```



### 9、 hooks 自定义  防抖函数

```react
 static debounce(func, time = 200) {
    return (...args) => {
      if (timeout) clearTimeout(timeout);
      timeout = setTimeout(() => func(args), time);
    };
  }
```

**封装成一个组件使用**

```react
import { useEffect, useRef } from  'react'  

const  useDebounce = (fn, ms = 30, deps = []) => {

let  timeout = useRef()

useEffect(() => {

if (timeout.current) clearTimeout(timeout.current)

timeout.current = setTimeout(() => {

fn()

}, ms)

}, deps)

  

const  cancel = () => {

clearTimeout(timeout.current)

timeout = null

}

return [cancel]

}

export  default  useDebounce


// 使用方法
import { useDebounce } from  'hooks' //引入

const [cancel] = useDebounce(() => {

setB(a)

}, 2000, [a])
```



### 10、useMemo 

> ​	把“创建”函数和依赖项数组作为参数传⼊ useMemo ，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进⾏⾼开销的计算。

```react
import React, { useState, useMemo } from "react";
export default function UseMemoPage(props) {
     const [count, setCount] = useState(0);
     const expensive = useMemo(() => {
        console.log("compute");
        let sum = 0;
        for (let i = 0; i < count; i++) {
            sum += i;
        }
        return sum;
        //只有count变化，这⾥才重新执⾏
     }, [count]);
     const [value, setValue] = useState("");
     return (
        <div>
            <h3>UseMemoPage</h3>
            <p>expensive:{expensive}</p>
            <p>{count}</p>
            <button onClick={() => setCount(count + 1)}>add</button>
            <input value={value} onChange={event => setValue(event.target.value)} />
        </div>
     );
}
```



### 11、useCallBack

> 把内联回调函数及依赖项数组作为参数传⼊ `useCallback `，它将返回该回调函数的 `memoized `版本， 该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使⽤引⽤相等性去避 免⾮必要渲染（例如 `shouldComponentUpdate `）的⼦组件时，它将⾮常有⽤。

```react
import React, { useState, useCallback, PureComponent } from "react";
export default function UseCallbackPage(props) {
     const [count, setCount] = useState(0);
     const addClick = useCallback(() => {
     	let sum = 0;
     	for (let i = 0; i < count; i++) {
     		sum += i;
     	}
     	return sum;
     }, [count]);
     const [value, setValue] = useState("");
     return (
     	<div>
     		<h3>UseCallbackPage</h3>
     		<p>{count}</p>
     		<button onClick={() => setCount(count + 1)}>add</button>
     		<input value={value} onChange={event => setValue(event.target.value)} />
     		<Child addClick={addClick} />
     	</div>
     );
}
class Child extends PureComponent {
     render() {
     	console.log("child render");
     	const { addClick } = this.props;
         return (
            <div>
                <h3>Child</h3>
                <button onClick={() => console.log(addClick())}>add</button>
            </div>
         );
    }
}
```





### 12、 React  中 Key 值的作用

**答：**  <font color='red'>**标识唯一性，   较少性能开销**</font>

！！！ **Key的值必须保证其唯一和稳定性**

​	react 为了提升渲染性能，在内部维持了一个虚拟dom，当渲染结构有所变化时，会在虚拟dom中先用diff算法先进行一次对比，再根据虚拟dom的变化，渲染到真实的Dom结构

​	如果两个元素的key是相同的，且满足第一点元素类型相同，若元素属性有变化，则React 只更新组件对应的属性，较少性能开销

​	key值不同， 则在render中会将两个新旧的虚拟dom节点进行对比，

​	如果两个元素的key不同，那么渲染过程中 即认为是两个不同的元素，

​	则会在React 源码中 此时key将作为下标，来移除掉旧的dom，append新的dom

​	当没有key值时，则会 直接移除原先的子元素，并append新的，所以提供下标会较少性能开销

​	如果数组有增删时，不建议 使用  index 来做key值

​	因为此时的key没有意义，新增或者删除 都会 使 原先的index 发生改变。

​	以`Math.random() `作为 key，会重新销毁后重新创建，影响性能 

**例：** 个人在  使用antd的上传图片的组件中，为了消除缓存，则使用。







### 