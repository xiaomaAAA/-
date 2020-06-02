# nslCloud 云货架

### 一、 开篇

####      1、脚手架

#####       （1）  脚手架

​	使用的是 公司老大的，已经搭好了基础环境语法。包括打包，运行，语法转义，webpack，定义了些 共用的方法， 如滚动条，传送门、 弹幕、 以及 数据流工具（mobx) 等命令。

##### 	（2） electron

​	因为要应用在云货架上，使用的是 electron 打包，环境由老大搭建，我只负责将其 打包生成 exe 文件 =。=

####      2、目录结构

![image-20200511150059339](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513154507.png)

##### 			1、api： 用于存放接口

###### (1) Ajax :   用于接口请求 实际上是 **<font color= 'red'>axios</font>**  （直接封装使用）

```react
import axios from 'axios';

/**
 \* Ajax.query({url, params, method = 'get'})
 *
 */

export default class Ajax {
    
 static query({url, params = {}, method = 'get', header = {cancel: false}}) {
     
  const api = eval('CFG.api.' + url);
     
  if (!params.token) {
   params = {
    ...params,
    token: CFG.token,
   };
  }

  if (method === 'get') {
   params = {params};
  }

  // 添加终止ajax
  const {cancel} = header;
  if (cancel) {
   const CancelToken = axios.CancelToken;
   this.source = CancelToken.source();
  }

  return new Promise((resolve, reject) => {
   axios[method](api, params, header).then(res => {
    const {data, code, msg} = res.data;
    if (!code && data) {
     resolve(data);
    }
    reject({code, msg, data: null});
   }).catch(err => {
    reject(err);
   });
  });

 }

 static cancel() {
  if (this.source) {
   this.source.cancel('取消请求');
  }
 }
}
```

######      (2) 请求的完整写法

​		大致写法：

```react
import Ajax from 'common/utils/ajax';

/**
 \* 获取商品列表
 \* @returns {Promise<any>}
 */
export const getGoodsList = () => new Promise((resolve) => {
 let url = 'goodsList';
 Ajax.query({url}).then(res => {
  resolve(res);
 });
    
/**
 * 结算购物车
 *
 * @param item_ids
 * @returns {Promise<any>}
*/
export function pay(ids, type) {
  // todo 成功时返回地址，失败本地拼接一个地址
  return new Promise((resolve, reject) => {
    Ajax.query({
      url: 'pay',
      params: {
        ids,
        token: CFG.other_token,
        type: type,
      },
    }).then(res => {
      resolve(res.result);
    }).catch(err => {
      reject(err);
    });
  });
}
    
});
```

##### 2、 assets 存放静态资源

如图片，视频，字体,等

##### 3、common 公共方法

###### （1） 定死的数据

###### 可以将一些  定死的数据 写在 这个里面，比如 某个组件要引用多个轮播图，可以定义在这里，然后直接map循环。

如：

 ![image-20200511154103096](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513154508.png)

或者

![image-20200511154203436](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513154509.png)

###### （2）定义 utils 方法

如 时间转义、防抖、节流 均可定义在这

##### 4、components 共用组件

如 传送门、滚动条

##### 5、layout 页面布局

##### 6、store 定义的仓库 存放数据

##### 7、src 下的 index 

写法： 

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

### 二、 页面篇

#### 1、第一个页面

![image-20200511155745808](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513154510.png)

#### 2、活动页面 多种相似

![image-20200511155816803](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513154511.png)

#### 3、分类

![image-20200511155848456](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513154512.png)

#### 4、详情

![image-20200511155908792](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513154513.png)

#### 5、购物车

![image-20200511155940873](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513154514.png)

### 三、开发

由于是 SPA 项目，所以使用的数据流  是 **<font color='red'>mobx</font>**

页面的展示和返回 均 使用不同变量控制，

#### 0、html 

定义接口，定义 是否有网络， token 是否存在 【每个店铺的token 不同，无token 则会第一屏 显示 错误】

```html
<!DOCTYPE html>
<html lang="cn">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="description" content="">
  <meta name="keywords" content="">
  <meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=0">
 <title>nsl云货架</title>
  <script src="./swiper.min.js"></script>
 <link rel="stylesheet" href="./swiper.min.css">
  <style>
  #modal {
   display: none;
   background: rgba(0, 0, 0, .6);
   position: absolute;
   top: 0;
   left: 0;
   right: 0;
   bottom: 0;
   z-index: 999;
   justify-content: center;
   align-items: center;
   font-size: 40px;
   color: #ffffff;
  }
 </style>
</head>
<body>
<div id="root"></div>
<div id="modal"></div>
<script>

 const IP = 'http://yoofun.ewszjk.m.jaeapp.com';

 window.CFG = {
  __publicPath__: '',
  token: '02dc241dda1913c71612916940b5dc30',
  other_token: '02dc241dda1913c71612916940b5dc30', //总店
  online: true,

  // api接口
  api: {
   goodsList: sourceIp + '/front/theland/goods',
  }
 };

 window.addEventListener('online', function() {
  CFG.online = true;
 });

 window.addEventListener('offline', function() {
  CFG.online = false;
 });
</script>
</body>
</html>
```



#### 1、第一个页面 

##### （1）轮播：

​	使用的是**<font color='red'>swiper</font>**插件，版本 ：**<font color='red'>"swiper": "^4.5.0"</font>**

【

​	一开始  electron 打包后 在云货架上无法 手动轮播，才发现是 版本问题，最初版本： electron": "^5.0.4",

然后使用 "electron": "^4.0.0", 即完美解决

】

 写法如下： 

```react
// 声明周期执行时调用
componentDidMount() {
    this.init();
}

init() {
  const mySwiper = new Swiper('.swiper-container', {
   observer: true, // 修改swiper自己或子元素时，自动初始化swiper
   observeParents: true, // 修改swiper的父元素时，自动初始化swiper
   direction: 'horizontal',
   loop: true,
   initialSlide: 0,
   speed: 600,
   autoplay: {
    delay: 2500,
    disableOnInteraction: false, // 解决滑动后不能轮播的问题
   },
   pagination: {
    el: '.swiper-pagination'
   },
  });
 }

 renderHeader() {
    const {store} = this.props;
    const {headers} = store.page;
    return (<div className="swiper-container ">
      <div className="swiper-wrapper">
        {
          headers.map((item, index) => {
            const sliderProps = {
              key: `swiper${index}`,
              className: 'swiper-slide swiper-no-swiping',
              style: {
                width: 1080,
                height: 490
              },
            };
            return (
              <div {...sliderProps}>
                <img style={{width: '100%'}} src={item.url} onClick={() => this.sliderClick(item)}/>
              </div>
            );
          })
        }
      </div>
    </div>);
  }
```



##### （2）token

检测 token 是否存在，

```react
 readToken() {
  return new Promise((resolve) => {
   window.CFG.fs.readFile('C:\\NSL/token.txt', 'utf-8', function(error, data) {
    if (error) {
     resolve(false);
    }
    resolve(data);
   })
  });
 }

async componentDidMount() {
    this.init();
    const {store} = this.props;
    const {shopCart} = store;

    const isLocal = Utils.isLocal();
    if (!isLocal) {
      const token = await this.readToken();
      if (!token) {
        this.setState({
          error: true,
        });
        return;
      }
      window.CFG.other_token = token;
    }
}
```

##### （3）品牌分类

​	是个横向滑动的品牌分布

​	使用的是 [reactScrollbar](https://github.com/souhe/reactScrollbar) 插件，直接梭哈。

##### （4）悬浮图标

​	实时运动，但不能超出屏幕，可使用拖动

```react
// 定义的float 页面
import React from 'react';
import css from './index.less';
import {inject, observer} from 'mobx-react';

export default class Float extends React.Component {

 constructor(props) {
  super(props);
  this.state = {
   top: 1400,
   left: 800,
  };
  this.type = 'left';
  this.isStart = false;
  this.timer = null;
 }

 componentDidMount() {
  this.setState({
   top: this.props.data.extraInfo.top,
   left: this.props.data.extraInfo.left
  });

  this.timer = setInterval(() => {
   if (this.isStart) return;
   if (this.state.left <= 0) this.type = 'right';
   if (this.state.left >= 800) this.type = 'left';
   this.setState({
    left: this.type === 'left' ? this.state.left - 0.5 : this.state.left + 0.5
   });
  }, 20);
 }

 componentWillUnmount() {
  clearInterval(this.timer);
 }

 iconStart(e) {
  this.isStart = true;
  const {top, left} = this.state;
  this.firstX = e.touches[0].clientX - left;
  this.firstY = e.touches[0].clientY - top;
 }

 iconMove(e) {
  this.setState({
   top: e.touches[0].clientY - this.firstY,
   left: e.touches[0].clientX - this.firstX,
  });
 }

 render() {
  const {data} = this.props;
  if (!data) return null;
  const {top, left} = this.state;
  const iconProps = {
   style: {
    top,
    left,
   },
   onTouchEnd: () => {
    this.isStart = false;
   },
   onTouchStart: this.iconStart.bind(this),
   onTouchMove: this.iconMove.bind(this),
   onClick: () => {
    this.props.onClick();
   },
   className: css.icon,
   src: data.url,
  };
  return <img {...iconProps} alt=""/>;
 }
}

```

```react
// 使用  
renderFloat() {
    const {page: {float}, showFloat} = this.props.store;
    if (!float || !showFloat) return null;
    return float.map((item, index) => {
      return (<Float data={item} key={item.action + index} onClick={() => {
        this.sliderClick(item);
      }}/>);
    });
  }
```

#### 2、 第二个页面  分类

###### （1）轮播： 3D 轮播

​	使用的是 [wj-slider3d](https://github.com/jiege1/wj-slider3d) 插件，直接梭哈。

###### （2）  滚动菜单

![image-20200511165237956](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513154515.png)

搬用老大定义的组件 [srcoll](https://github.com/xiaomaAAA/scroll) 

######  	（3）商品的滚动

![image-20200511175515419](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513154516.png)

```react
import React from 'react';
import PropTypes from 'prop-types';
import ScrollArea from 'components/scrollArea'; 
//  [reactScrollbar](https://github.com/souhe/reactScrollbar)
import requestAnimationFrame, {cancelAnimationFrame} from 'common/utils/requestAnimationFrame';

export default class Scroller extends React.Component {

  static propTypes = {
    width: PropTypes.number,
    height: PropTypes.number,
    autoScroll: PropTypes.bool,
    autoScrollAgainTime: PropTypes.number,
    speed: PropTypes.number,
    direction: PropTypes.oneOf(['vertical', 'horizontal']),
  };

  static defaultProps = {
    width: 1080,
    height: 1920,
    autoScroll: false,
    autoScrollAgainTime: 10,
    speed: 1,
    direction: 'vertical',
  };

  constructor(props) {
    super(props);
    this.state = {};
    this.isBack = false;
    this.scrollAreaRef = React.createRef();
  }

  componentDidMount() {
    if (this.props.autoScroll) {
      this.autoScroll();
    }
  }

  componentWillUnmount() {
    if (this.autoScrollTimer) {
      cancelAnimationFrame(this.autoScrollTimer);
    }
  }

  autoScroll() {
    if (this.autoScrollTimer) {
      cancelAnimationFrame(this.autoScrollTimer);
    }
    const {speed} = this.props;
    const scrollAreaInstance = this.scrollAreaRef.current;
    this.autoScrollTimer = requestAnimationFrame(() => {
      const isVertical = this.props.direction === 'vertical';
      const {topPosition, leftPosition} = this.scrollAreaRef.current.state;
      const lastPosition = isVertical ? topPosition : leftPosition;
      const nextPosition = this.isBack ? lastPosition - speed : lastPosition + speed;

      this.scrollTo(nextPosition);
      if (nextPosition <= 0) {
        this.isBack = false;
      }
      const {content, wrapper} = scrollAreaInstance;
      const wrapperLength = isVertical ? wrapper.offsetHeight : wrapper.offsetWidth;
      const contentLength = isVertical ? content.offsetHeight : content.offsetWidth;
      if (wrapperLength + nextPosition >= contentLength) {
        this.isBack = true;
      }
      this.autoScroll();
    });
  }

  scrollTo(position) {
    const scrollAreaInstance = this.scrollAreaRef.current;
    if (this.props.direction === 'vertical') {
      scrollAreaInstance.scrollYTo(position);
    } else {
      scrollAreaInstance.scrollXTo(position);
    }
  }

  render() {
    const {children, height, width, autoScrollAgainTime, autoScroll} = this.props;
    const isVertical = this.props.direction === 'vertical';
    const props = {
      ref: this.scrollAreaRef,
      vertical: isVertical,
      horizontal: !isVertical,
      style: {width, height},
    };
    const contentProps = {
      onTouchStart: () => {
        if (this.autoScrollTimer) {
          cancelAnimationFrame(this.autoScrollTimer);
        }
      },
      onTouchEnd: () => {
        if (autoScroll) {
          this.awaitAutoScrollTimer = setTimeout(() => {
            clearTimeout(this.awaitAutoScrollTimer);
            this.awaitAutoScrollTimer = null;
            this.autoScroll();
          }, autoScrollAgainTime * 1000);
        }
      },
    };
    return (
      <ScrollArea {...props}>
        <div {...contentProps}>
          {children}
        </div>
      </ScrollArea>
    );
  }
}
```

```react
import Scroller from 'components/scroller'; // 即上诉

    return (<Scroller width={819} height={1100} className={css.list}>
      <div className={css.list}>
        {
          store.getGoodsList.map((item, index) => {
            return (
              <div key={index}>
                {item}
              </div>
            );
          })
        }
      </div>
    </Scroller>);
```

#### 3、商品详情

![image-20200511180013607](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513154517.png)

###### （1）随机搭配的三个产品

```javascript
// filterList 为商品数据
for (let i = 0; i < 3; i++) {
      let _num = Math.floor(Math.random() * filterList.length);
      let mm = filterList[_num];
      filterList.splice(_num, 1);
      newRecommend.push(mm);
}
```

###### （2）详情图

```react
renderDescription() {
    const {customDescription} = this.props.store.goodsDetail; // 从数据库中获取的图片链接  
    const descriptionImgs = customDescription.replace(/[，]|[;]|[；]/g, ',').split(',');
    // 切割成数组
    return (
      <WjScroll className={css.scroll}>
        {
          descriptionImgs.map((url, index) => <img key={`desc_${index}`} src={url} alt=""/>)
        }
      </WjScroll>
    );
  }
```

#### 四、electron 打包

##### 1、目录结构

![image-20200511181721641](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513154518.png)

###### （1）app 为 项目build 打包后，将html、css、即引用的外部插件 放进去

###### （2） icon 可以放 应用的 图标

###### （3）installer.nsh 可有可无 只是加快依赖的下载

###### （4）package.json 依赖  梭哈就行

##### 2、app 结构

![image-20200511182047152](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513154519.png)

###### （1） html

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="description" content="">
  <meta name="keywords" content="">
  <meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=0">
  <link rel="stylesheet" href="./swiper.min.css">
  <script src="./swiper.min.js"></script>
  <link href="./index.min.css" rel="stylesheet">
</head>
  <title>react-temp</title>
  <style>
    #modal {
      display: none;
      background: rgba(0, 0, 0, .6);
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      z-index: 999;
      justify-content: center;
      align-items: center;
      font-size: 40px;
      color: #ffffff;
    }
    #btn {
      display: none;
      background: rgba(0, 0, 0, .6);
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      z-index: 999;
      justify-content: center;
      align-items: center;
      font-size: 48px;
    }
    #now {
      margin-right: 100px;
      padding: 10px;
    }
    #next {
      padding: 10px;
    }
  </style>
<link href="./index.min.css" rel="stylesheet"></head>
<body>
  <div id="root"></div>
  <div id="modal"></div> 
  <div id="btn">
    <input type="button" value="立即更新" id="now"/>
    <input type="button" value="下次重启更新" id="next"/>
  </div> 
  <script>
    let update = 1;
    const IP = 'https://yoooofun.zjk.taeapp.com';
  const newIp = 'https://yoofunpay.zjk.taeapp.com';
  const sourceIp = 'https://yoooshow.ewszjk.m.jaeapp.com';
    const {machineIdSync} = require('node-machine-id');
    const {ipcRenderer} = require('electron');


  
    ipcRenderer.on('message', (event, { message, data }) => { 
      switch (message) {
        case 'isUpdateNow':
        if (confirm('检测到新版本，是否立即更新？')) ipcRenderer.send('updateNow');
          break;
        case 'sendToken': {
          CFG.token = data;
          break;
        }
        default:
          break;
      }
    });
    
    window.CFG = {
      __publicPath__: '',
      fs: require('fs'),
      token: '02dc241dda1913c71612916940b5dc30',
      machineId: machineIdSync(),
      other_token: '02dc241dda1913c71612916940b5dc30', //总部门店 本店
      online: true,
      // api接口
      api: {
        goodsList: sourceIp + '/front/theland/goods',
        feedBack: IP + 'test',
      }
    };

    /**
     * 生产模式下F
     * 注入 electron 变量
     * @type {*}
     */
    var ele = require('./render');

    window.addEventListener('online', function() {
      CFG.online = true;
    });
    window.addEventListener('offline', function() {
      CFG.online = false;
    });
  </script>
<script type="text/javascript" src="./index.min.js"></script>
</body>

```

###### （2） main 老大搭的 

```javascript
// only add update server if it's not being run from cli
// if (require.main !== module) {
//   require('update-electron-app')({
//     logger: require('electron-log')
//   })
// }

const path = require('path');
const url = require('url');
const fs = require('fs');
const {app, BrowserWindow, ipcMain} = require('electron');
const { autoUpdater } = require('electron-updater');

// app.commandLine.appendSwitch('ignore-certificate-errors');
const debug = false
const feedUrl = 'https://yoofunpay.zjk.taeapp.com/nsl/';
let webContents = null;
// if (process.mas) app.setName('Electron APIs')

let mainWindow = null;
let isOnline = false;

function initialize () {

  function createWindow () {
    const windowOptions = { 
      width: 1920,
      // minWidth: 680,
      height: 1080,
      frame: false,
      alwaysOnTop: true,
      fullscreen: true,
      title: app.getName(),
      webPreferences: {
        nodeIntegration: true,
          devTools: true
      }
    }
    // const gotTheLock = app.requestSingleInstanceLock()

    // if (!gotTheLock) {
    //   app.quit();
  
    // } 
    mainWindow = new BrowserWindow(windowOptions)
    mainWindow.maximize();
    const winUrl = url.format({
        pathname: path.join(__dirname, '/online-status.html'),
        protocol: 'file:',
        slashes: true
    });
   

   webContents = mainWindow.webContents;
    mainWindow.loadURL( winUrl );
    // Launch fullscreen with DevTools open, usage: npm run debug
    if (debug) {
      mainWindow.webContents.openDevTools()
      mainWindow.maximize()
      // require('devtron').install()
    }
    // webContents.on('did-finish-load', () => {
    //   webContents.setZoomFactor(1);
    //   webContents.setVisualZoomLevelLimits(1, 1);
    //   webContents.setLayoutZoomLevelLimits(0, 0);
    // });
    checkForUpdates();
    mainWindow.on('closed', () => {
      mainWindow = null
    })
  }

  app.on('ready', () => {

    createWindow()
  })

  app.on('window-all-closed', () => {
    if (process.platform !== 'darwin') {
      app.quit()
    }
  })

  app.on('activate', () => {
    if (mainWindow === null) {
      createWindow()
    }
  })
}

function readToken() {
  return new Promise((resolve, reject) => {
    fs.readFile("C:\\NSL/token.txt", "utf-8", function(error, data) {
      if (error) {
        reject("读取文件失败,内容是" + error.message);
      }
      resolve(data);
    });
  })
}

// Make this app a single instance app.
//
// The main window will be restored and focused instead of a second window
// opened when a person attempts to launch a second instance.
//
// Returns true if the current version of the app should quit instead of
// launching.
function makeSingleInstance () {
  if (process.mas) return false

  return app.makeSingleInstance(() => {
    if (mainWindow) {
      if (mainWindow.isMinimized()) mainWindow.restore()
      mainWindow.focus()
    }
  })
}
let sendUpdateMessage = (message, data) => {
  webContents.send('message', { message, data });
};
let checkForUpdates = () => {
  autoUpdater.setFeedURL(feedUrl);
  autoUpdater.on('error', function (message) {
    console.log('error');
      sendUpdateMessage('error', message)
  });
  autoUpdater.on('checking-for-update', function (message) {
    console.log('checking-for-update');
      sendUpdateMessage('checking-for-update', message)
  });
  autoUpdater.on('update-available', function (message) {
    console.log('update-available');
      sendUpdateMessage('update-available', message)
  });
  autoUpdater.on('update-not-available', function (message) {
    console.log('update-not-available');
      sendUpdateMessage('update-not-available', message)
  });

  // 更新下载进度事件
  autoUpdater.on('download-progress', function (progressObj) {
    sendUpdateMessage('downloadProgress', progressObj)
  })
  // autoUpdater.on('update-downloaded', function (event, releaseNotes, releaseName, releaseDate, updateUrl, quitAndUpdate) {
  //     ipcMain.on('updateNow', (e, arg) => {
  //         //some code here to handle event
  //         autoUpdater.quitAndInstall();
  //     })
      
  //     sendUpdateMessage('isUpdateNow');
  //     // autoUpdater.quitAndInstall();
  // });

  // 执行自动更新检查
  autoUpdater.checkForUpdates().then(d => {
    d.downloadPromise.then(res => {
      console.log('res', res);
      setTimeout(() => {

        sendUpdateMessage('isUpdateNow');
        ipcMain.on('updateNow', (e, arg) => {
          //some code here to handle event
            autoUpdater.quitAndInstall();
        })
      }, 2000)
    })
  });
};
 ipcMain.on('close-main-window', (event, arg) => {
    app.quit()
  })

//设备是否在线
ipcMain.on('online-status-changed', (event, status) => {
  if (!isOnline && status === 'online') {
    mainWindow.loadURL(
      url.format({
        pathname: path.join(__dirname, '/index.html'),
        protocol: 'file:',
        slashes: true
      })
    );

      // mainWindow.loadURL(path.join(url))
    isOnline = true;
  }
});

initialize();
app.on('window-all-closed', () => app.quit());

```

###### （3）render.js 

```javascript
const {ipcRenderer} = require('electron');


const exit = () => {
	ipcRenderer.send('close-main-window');
};
module.exports = {
  exit,
};
```

##### 3、packjson

```javascript
{
  "name": "test",
  "version": "2.1.11",
  "description": "",
  "main": "./app/main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dist": "electron-builder --win --x64",
    "start": "electron ."
  },
  "build": {
    "productName": "nsl",
    "appId": "com.xxx.app",
    "mac": {
      "target": [
        "dmg",
        "zip"
      ]
    },
    "win": {
      "icon": "icon/logo.ico",
      "artifactName": "latest.${ext}",
      "target": [
        "nsis",
        "zip"
      ]
    },
    "publish": [
      {
        "provider": "generic",
        "url": "https://yoofunpay.zjk.taeapp.com/nsl/"
      }
    ],
    "nsis": {
      "oneClick": false,
      "allowElevation": true,
      "createDesktopShortcut": true,
      "createStartMenuShortcut": true,
      "include": "installer.nsh"
    }
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "electron": "^5.0.4",
    "electron-builder": "^20.44.4",
    "node-machine-id": "^1.1.12"
  },
  "dependencies": {
    "electron-updater": "^4.2.0",
    "node-machine-id": "^1.1.12"
  }
}

```



#### 五、总结 

这是我第一个负责的项目，项目基于老大的脚手架开发，其实很多组件都定义好了，直接调用。

