# mobx-react  使用手册

## 一、以 SPA 为例 单页面开发 不涉及路由 

### 一、 一般在入口文件就挂在 了 Provider 【 可理解为  数据容器  】

​		

```react
import React from 'react';
import ReactDom from 'react-dom';
import {Provider} from 'mobx-react';
import Layout from './layout';
import './index.less';
import 'common/utils/backDesktop';
import store from './store';

ReactDom.render(
  <Provider store={store}>  
    <Layout/>
  </Provider>,
  document.getElementById('root')
);

```



### 二、 store 可分为  多个Store 【即每个模块下有一个自己的store】 和 一个全局的store 

### 	(一) 、   **<font color=#FF000 >共用一个 store</font>**

	SPA 开发 可以 共用一个store 

目录结构如下： ![image-20200508182211319](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513155147.png)



base  可 认为是 基础写法， 用于 写一些方法 

如 观察变量的 更新， 判断是否是一个 可观察变量



#### 1、 base:  用于基础语法，用于 自定义 更新 方法 等

![image-20200508182420969](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513155148.png)



【 isObservableProp 】 返回的是个boolean 是否是一个 可观察属性 

使用 自定义的 <font color=#FF000 >update</font > 函数方法 来  <font color=#FF000 >更新 </font >状态



#### 2、 shopCart ： 是 一个 自定义 的文件夹  可 自定义名字 无规范  

```
	可以用来区别去 其他 变量专用 如加购等
```

######  1、主要语法：

###### 	（1）@observable  

​		官方解释:   装饰器可以用于ES7环境或者TypeScript中，以使类属性转变为可观察的。 @observable 可用于实例字段和属性getters。 这为你的普通对象转变为可观察对象提供了细粒度的方法		

​	   自行 理解为： 这是个可以观察的属性  可以 使用  <font color=#FF000 >上方所诉的自定义 update 方法 来更新 其值 </font >

###### 	（2）@observer

​		官方解释:   `observer` 函数/装饰器可以用来将 React 组件转变成响应式组件。 它用 `mobx.autorun` 包装了组件的 render 函数以确保任何组件渲染中使用的数据变化时都可以强制刷新组件。 `observer` 是由单独的 `mobx-react` 包提供的。

​      自行理解为： `@observer` 是`mobx-react`提供的，通过使用`@observer`，将react组件转换成一个监听者，这样在被监听的应用状态变量(Observable)有更新时，react组件就会重新渲染。

​	类组件写法： 直接挂在到组件上面

```react
@observer
export default class Layout extends React.Component {}
```

或者 挂在屁股

```react
export default observer(carouselRadio);
```

   

   函数式组件写法： 

```react
const carouselRadio = ({ mod, attr }) => {
 return 123;
};

export default observer(carouselRadio);
```

 或者 

```react
const ImageCard = observer(({ mod, onImgClick, elementRef }) => {
 return 123;
});

export default ImageCard;
```

######   	（3）@inject

   可理解为： 是为了向当前被装饰的组件 注入 store 这个props。当然 store 这个 prop 其实是由 Provider 提供的

   通常写法 ： 在最外层 先定义个Provider 如上所示 ， 然后在组件中 则直接挂在即可 @inject('store')，就可以直接调用store中的方法了。 

  类组件写法： 

```react
@inject('store')
@observer
export default class DetailModal extends React.Component {}
```

函数式组件写法： 

```react
const Simulator = ({ store }) => {
 const { simulator, pageMod } = store;
 const { Render } = Utils.getCacheModuleByType(pageMod.type).components;
 return 123;
};

export default inject('store')(observer(Simulator));
```

###### （4）action 

​     官方解释：  任何应用都有动作。动作是任何用来修改状态的东西。 使用MobX你可以在代码中显式地标记出动作所在的位置。 动作可以有助于更好的组织代码

  自我理解 ：  声明 这是个方法，需要手动调用 

写法：

```react
 @action setFloatVisible(flag) {
  this.showFloat = flag;
 }
```

可以写异步，用来请求数据发送数据等

```react
 @action
 async queryGoodsList() {
  // 请求数据
  try {
   const {goods} = await getGoodsList(); // 定义的接口方法 
   const {items} = await getWeGoodsList();
   let goodsObject = {};
   goods.forEach(item => {
    item.isSellOut = !((item.approveStatus === 'onsale' || item.approveStatus === 'null') && item.num && parseInt(item.num) !== 0);
    goodsObject[item.goodsTaobaoId] = item;
   });
   // 拿到服务端返回的数据后  更新变量  
   this.update({
    goodsList: goods,
    youzanGoodsList: items,
    goodsObject: goodsObject
   });
  } catch (e) {
  }
 }
```



###### （5）computed 

  官方解释： 如果你想响应式的产生一个可以被其它 observer 使用的**值**，请使用 `@computed`

  自我理解： 一个 可被调用  执行的方法，自动 计算 并 返回值 【  由于是 get 所以不能带参数 】

例子： 

  定义在 store 的  一个获取总数的方法  

```react
 @computed get selectTotalNum() {
  let total = 0;
  this.selected.forEach(item => {
   total += this.goodsContainer[item];
  });
  return total;
 }
```

外部组件调用： ` {this.props.store.shopCart.totalNum}`

 

###### （6）自定义函数

  需要带参数的方法 可以自定义

例子： 

```react
getGoodsDetail(goodsId) {
  return this.parent.goodsList.find(item => item.goodsTaobaoId === goodsId);
 }
```



###### （7）reaction

 用法: `reaction(() => data, (data, reaction) => { sideEffect }, options?)`。

 官方解释：它接收两个函数参数，第一个(*数据*函数)是用来追踪并返回数据作为第二个函数(*效果* 函数)的输入。

 自我理解： 第一个参数为监听的观察值，第二个为反应函数 （即这个值改变就触发这个函数）

 例子： 

```react
 constructor() {
    super();
    reaction(
      () => this.isPut,
      () => {
        this.queryChannelList();
      }
    );
  }
```

如要将监听多个值，则用数组 【】

```react
  constructor() {
    super();
    reaction(
      () => [this.pageNo, this.pageSize, this.keyword],
      () => {
        this.queryList();
      }
    );
  }
```



###### （8）runInAction

自我理解： 随着执行的行为 【 一般配合 结合 请求 一起 用 】

例子：  将接口返回的数据  赋值到观察属性上 即 channelList loading

```react
 @action
 async queryChannelList() {
  this.loading = true;
  try {
   const data = await getChannelList();
   runInAction('getChannelList', () => {
     this.loading = false;
     this.channelList = data.list;
   });
  } catch (e) {
   console.log(e);
  }
 }
```



###### 2、整体写法如下： 

```react
import {observable, action, computed, reaction, toJS， runInAction} from 'mobx';
import Base from './models/base';


class Store extends Base {
  @observable loading = true;

  constructor(props) {
    super(props);
    reaction(
      () => this.showGoodsId,
      () => {
        this.update({
          recommendGoods: [],
        });
      }
    );
  }

  /**
   * 当前展示的商品信息
   * @return {*}
   */
  @computed get goodsDetail() {
    return this.goodsObject[this.showGoodsId];
  }

  @action setShowActivity(activityId) {
    this.feedbackData({event: 'show_activity', msg: activityId});
  }


  @action
  async queryChannelList() {
    this.loading = true;
    try {
      const data = await getChannelList();
      runInAction('getChannelList', () => {
        this.loading = false;
        this.channelList = data.list;
      });
    } catch (e) {
      console.log(e);
    }
  }

}

export default new Store();
```



#### 3、 store 的 index 

 如上所示 直接存放变量

```react
import {observable, action, computed, reaction, toJS} from 'mobx';
import Base from './models/base';

class Store extends Base {

 @observable status = true;

 constructor(props) {
  super(props);
 }

 /**
  \* 获取分类页轮播
  */

 @computed get classifySwiper() {
  if (!this.selectedRowClassify.showSwiper)
   return this.page[this.selectedRowClassify.type].swiper;
  return this.selectedRowClassify.showSwiper;
 }
 
}

export default new Store();
```



## 二、 多页面开发 如后台等

​		要考虑到路由，可能是多级路由（一个路由下仍有其他子路由）。

​	所以要分模块开发。每个模块都有自身的**<font color='red'>model</font>**, 来处理该模块的逻辑、数据交互等。模块的切换后，要将上个模块的store初始化。

​	入口文件（src）下的 Inde 也仍需要挂载 store，以后台开发为例：

```react
import 'babel-polyfill';
import React from 'react';
import ReactDom from 'react-dom';
import Router from './router';
import { Provider } from 'mobx-react';
import store from 'store';
import 'moment/locale/zh-cn';
import './index.less';
import zhCN from 'antd/lib/locale-provider/zh_CN';
import { ConfigProvider } from 'antd';

ReactDom.render(
 <ConfigProvider locale={zhCN}>
  <Provider store={store}> // 所挂载的store 为一个全局的store
   <Router /> // 路由（页面的跳转）
  </Provider>
 </ConfigProvider>,
 document.getElementById('root')
);
```



​	全局store目录结构如下： 

![image-20200509165553933](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513155149.png)

####    1、base 为 基础语法  用来更新 状态  

```react
import {action, isObservableProp, extendObservable} from 'mobx';

export default class Base {

 /**

  \* 扩展 observable 属性

  \* @param observableProps

  */

 @action extend(observableProps) {
  if (observableProps) {
   extendObservable(this, observableProps);
  }
 }


 @action update(updateKeys) {
  Object.keys(updateKeys).forEach(key => {

   if (isObservableProp(this, key)) {

    this[key] = updateKeys[key];

   } else { // 不允许更新非监听状态的键值

    console.warn(`${key} is not an observable key!`);

   }

  });
 }
 
}
```



#### 2、 app 也是 存放 最基础的 状态  如 整体的颜色等...

```react
import { observable } from 'mobx';
import Base from '../base';

export default class App extends Base {
 @observable sideCollapsed = false;
 @observable sideTheme = 'dark'; // 'dark' : 'light'
 @observable list = [];
}
```



#### 3、store 的 Index 用于最开始根目录下的Provider 挂载

```react
import { observable, configure } from 'mobx';
import Base from './model/base';
import App from './model/app';

/**
 \* 使用严格模式

 \* 只能在action里面更新 observable属性

 \* https://github.com/mobxjs/mobx/blob/gh-pages/docs/refguide/api.md#configure
 */
configure({
 enforceActions: 'always',
});

/**
 \* 全局唯一的store
 */
class store extends Base {
 @observable app = new App();
}

export default new store();
```



#### 4、路由的配置  

使用的是 react-router-dom 

```react
import React from 'react';
import {HashRouter as Router, Route, Switch} from 'react-router-dom';
import routeData from 'common/const/route';
import Page404 from 'pages/404';
import Layout from 'components/layout';

function buildRoute(routes, parentPath = '') {
 return routes.map(route => {
  if (route.children && route.children.length) {
   return buildRoute(route.children, `${parentPath}${route.path}`);
  }

  if (route.path && route.component) {
   const props = {
    key: `route_${route.key}`,
    path: `${parentPath}${route.path}`,
    component: route.component,
    exact: true,
   };

   return (
    <Route {...props}/>
   );
     
  }

  return null;
 });
}

export default class Routers extends React.Component {
 componentDidMount() {}
 render() {
  return (
   <div style={{height: '100%'}}>
    <Router>
     <Layout>  {/* 页面的的 骨架 */}  
      <Switch>
       {buildRoute(routeData)}  {/* 页面的路由 */}
       <Route component={Page404}/> {/* 404页面 */} 
      </Switch>
     </Layout>
    </Router>
   </div>
  );
 }
}
```





5、 页面的骨架 重点在 **<font color='red'>side</font>** (侧边栏 即对应的路由页面)

layOut 文件夹目录 // 即后台框架样式

![image-20200511103907677](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513155150.png)

```react
import React from 'react';
import css from './index.less';
import { Layout as AntLayout } from 'antd';
import Header from './components/header';
import Footer from './components/footer';
import Side from './components/side';

const { Content, Footer: AntFooter, Header: AntHeader } = AntLayout;

export default class Layout extends React.Component {
 static propTypes = {};
 static defaultProps = {};

 constructor(props) {
  super(props);
  this.state = {};
 }

 componentDidMount() {}

 render() {
  return (
   <AntLayout className={css.layout}>
    <Side />  {/* ！！！重点 侧边栏 */}
    <AntLayout className={css.container}>
     <AntHeader className={css.header}>
      <Header />
     </AntHeader>
     <Content className={css.content}>  {/* 侧边栏 对应的内容部分 */}
      {this.props.children}
     </Content>
    </AntLayout>
   </AntLayout>
  );
 }

}
```



#### 5、 Side 侧边栏的路由渲染

```react
import React from 'react';
import { Layout as AntLayout, Menu as AntMenu, Icon, Switch, Radio } from 'antd';
import css from './index.less';
import classNames from 'classnames';
import {inject, observer} from 'mobx-react';
import routeData from 'common/const/route'; // 侧边栏的所有路由地址，用以跳转
import {Link, withRouter} from 'react-router-dom';

const { Sider} = AntLayout;
const {SubMenu, Item: MenuItem} = AntMenu;

//  @withRouter  这个方法是把一个非路由管控的组件，变成路由管控的组件,可以使用this.props获取到 history、location、match
// 使用方法在export default 组件时用withRouter(APP)
// 受路由管控组件例：<Route path={'/login'} component={Login}/>

@withRouter
@inject('store') // 挂载 store 
@observer // 将这个组件设置为可观察，数据有更新的话 组件内也会随着更新

export default class Side extends React.Component {
 static propTypes = {};
 static defaultProps = {};

 constructor(props) {
  super(props);
  this.state = {};
 }

 // 匹配路由，去找到对应文件夹路径 
 findRoute(pathname) {
  const paths = pathname.split('/');
  let route = null;
  paths.shift();
  paths.forEach(path => {
   if (!route) {
    route = routeData.find(item => item.path === `/${path}`);
   } else if (route) {
    route = route.children.find(item => item.path === `/${path}`);
   }
  });
  return route;
 }

 get selectedKey() {
  const route = this.findRoute(this.props.location.pathname);
  if (!route) {
   return [];
  }
  return [route.key];
 }

 // 渲染 侧边栏的同时 渲染对应的文件夹路径
 renderNavItems() {
  const renderMenu = (item, parentPath = '') => {
   if (item.noSideShow) {
    return null;
   }
   // 渲染 多级 路由
   if (item.children && item.children.length) {
    const props = {
     key: item.key,
     title: (
      <span>
       {!!item.icon && <Icon type={item.icon} />}
       <span>{item.label}</span>
      </span>
     ),
    };
    return (
     <SubMenu {...props}>
      {item.children.map(child => renderMenu(child, `${parentPath}${item.path}`))}
     </SubMenu>
    );
   }
   // 不展示在左侧的路由则不渲染

   if (!item.noSideShow) {
    if (item.path) {
     return (
      <MenuItem key={item.key}>
       <Link to={`${parentPath}${item.path}`}>
        {!!item.icon && <Icon type={item.icon} />}
        <span>{item.label}</span>
       </Link>
      </MenuItem>
     );
    }
    return (
     <MenuItem key={item.key} >
      {!!item.icon && <Icon type={item.icon} />}
      {item.label}
     </MenuItem>
    );
   }
   return null;
  };
  return routeData.map(item => renderMenu(item));
 }



 // 渲染 路由
 renderMenu() {
  const {app} = this.props.store;
  const props = {
   selectedKeys: this.selectedKey,
   mode: 'inline',
   theme: app.sideTheme,
  };

  return (
   <AntMenu {...props}>
    {this.renderNavItems()}
   </AntMenu>
  );
 }

 // 控制后台的 侧边栏 样式 暗黑样式 和 亮色样式 可有可无

 renderDarkSwitch() {
  const {app, app: {sideTheme, sideCollapsed}} = this.props.store;
  const renderRadio = () => (
   <Radio.Group
    value={sideTheme}
    buttonStyle="solid"
    size="small"
    onChange={(e) => {
     app.update({
      sideTheme: e.target.value,
     });
    }}>
    <Radio.Button value="light">亮色</Radio.Button>
    <Radio.Button value="dark">暗色</Radio.Button>
   </Radio.Group>
  );

  const renderSwitch = () => (
   <Switch
    checkedChildren="暗色"
    unCheckedChildren="亮色"
    checked={sideTheme === 'dark'}
    onChange={(checked) => {
     app.update({
      sideTheme: checked ? 'dark' : 'light',
     });
    }} />
  );

  return (
   <div className={css.themeBox}>
    {
     sideCollapsed ? renderSwitch() : renderRadio()
    }
   </div>
  );
 }

 render() {
  const { app: {sideCollapsed, sideTheme} } = this.props.store; 
  const props = {
   className: classNames(css.side, {[css.light]: sideTheme === 'light'}),
   collapsible: true,
   trigger: null,
   collapsed: sideCollapsed,
  };

  return (
   <Sider {...props}>
        <div className={css.sideBox}>
          {/* 设置 logo */}
          <div className={css.logoBox}>
            <img src={'https://img.alicdn.com/imgextra/i1/4074958541/O1CN01XiAfcz2CxpNVjpVo7_!!4074958541.png'} alt=""/>
     </div>
     {this.renderMenu()}
    </div>
   </Sider>
  );
 }
}
```



#### 6、填写路由，动态挂载各个模块的model

```react
import React from 'react';
import Loadable from 'react-loadable'; // 一个动态导入加载组件的高阶组件 代码分割
import {Spin} from 'antd'; // 旋转 loading
import store from 'store';

// 加载样式
const loading = (err) => {
 if (err.error) {
  console.error(err.error);
 }
 return (
  <Spin size="large" style={{width: '100%', margin: '40px 0'}}/>
 );
};

let lastModel = null; // 记录使用的路由model

// 匹配model
const lazyComponent = (path, model) => {
 let loader = {
  component: () => import(`pages/${path}`)
 };

 // 动态加载model
 if (model) {
  if (typeof model !== 'string') model = path;
  loader[model] = () => import(`pages/${path}/model`);
 }

 return Loadable.Map({
  loader,
  loading,
  render(loaded, props) {
   // 将model动态挂载到 store 上
   if (loaded[model]) {
    const Model = loaded[model].default;
    // 对比 路由 
    // 如果不同 即跳转到其他模块了， 则 将上一个model (卸载) 初始化 保证 数据即页面 的 正确
    if (lastModel !== model) {
     store[model] = new Model();
     lastModel = model;
    }
   }
   let C = loaded.component.default;
   return <C {...props}/>;
  }
 });
};

const routeData = [
 {
  key: 'store',
  label: '门店列表',
  // noSideShow: true,
  // path: '/store',
  path: '/', // 根路由   即页面就会挂载的第一个路由  自动挂载
  icon: 'desktop',
  desc: '门店列表',
  component: lazyComponent('shop', true),
  isAuth: true,
 },
 {
  key: 'classify',
  label: '商品分类',
  path: '/classify',
  icon: 'gold',
  desc: '商品分类',
  children: [
   {
    key: 'people',
    label: '人群分类配置',
    path: '/people',
    desc: '人群分类配置',
    component: lazyComponent('classify/people', 'people'),
    isAuth: true,
   },
   {
    key: 'brand',
    label: '品牌分类配置',
    path: '/brand',
    desc: '品牌分类配置',
    component: lazyComponent('classify/brand', 'brand'),
    isAuth: true,
   },
  ],
 },
];
export default routeData;
```





#### 7、 动态挂载 模块下的model

```react
import React from 'react';
import {inject} from 'mobx-react';

/**
 \* model 支持懒加载
 \* 建议不要跨页面使用model
 \* 推荐使用该方法，不建议 inject('store')
 *
 \* @param modelName store里面的字段
 \* @param outputName 输出字段
 \* @returns {function(*): ((function(*): *) & IWrappedComponent<function(*): *>)}
 */

export default function injectModel(modelName, outputName) {
 if (!outputName) {
  outputName = modelName;
 }
 return function(Com) {

  return inject('store')((props) => {
   let { store, ...comProps } = props;
   if (store[modelName]) {
    comProps[outputName] = store[modelName];
   } else {
    comProps.store = store;
   }
   return <Com {...comProps}/>;
  });
 };
}
```





#### 8、 模块下的 model 的使用 

```react
import React from 'react';
import {observer, inject} from 'mobx-react';
import injectModel from 'common/utils/injectModel';

@injectModel('shop') // 使用上诉方法挂载Model
@observer
class Shop extends React.Component {
 constructor(props) {
  super(props);
 }
 componentDidMount() {
   this.props.shop.queryChannelList(); // 使用 模块 的方法
 }

 render() {
  const {shop} = this.props; // 这就拿到了 该模块 的model
  return <div>123</div>
 }
}
export default Shop;
```





#### 9、 模块下 model 的写法 

<font color="red">确保名字与在Route路由中需挂载的名字统一</font>

```react
import Base from 'store/model/base';
import {observable, computed, toJS, action, runInAction, reaction} from 'mobx';
import {getStoreList} from 'api/store'; // api 方法

export default class Shop extends Base {

 @observable showDownLoadModal = false;

 constructor() {
  super();
  reaction(
   () => [this.keyword, this.pageNo, this.pageSize],
   () => this.queryList();
  );
 }

 @action
 async checkGift(giftProvided, expressId, storeId) {
  this.loading = true;
  let result = false;
  try {
   this.loading = false;
   const data = await checkGift({giftProvided: giftProvided, expressId: expressId});
   if (data.success) {
    result = true;
    this.queryList();
    this.getCheckList(storeId);
    this.queryGiftsList();
   }
  } catch (e) {
   this.loading = false;
  }
  return result;
 }
}
```

