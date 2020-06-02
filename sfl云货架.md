# sfl云货架

### 一、项目开篇

#### 1、数据流： mobx

  可参考 mobx 使用手册

#### 2、页面跳转：路由跳转

页面结构： 

##### （1）页面存放

所有页面路径写在 layout 的 Index里

![image-20200513163451550](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513172735.png)

##### （2）路由配置

router 的 index

![image-20200513163347858](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513163350.png)

#### 3、页面过度效果

每个页面都有过度的 效果，防止太过生硬切换

```less
animation: fadeIn 1s;
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
```

#### 4、pureComponent 减少页面重渲染

##### (1)、Component和PureComponent有一个不同点

除了为你提供了一个具有浅比较的`shouldComponentUpdate`方法，`PureComponent`和`Component`基本上完全相同。当`props`或者`state`改变时，`PureComponent`将对`props`和`state`进行浅比较。另一方面，Component不会比较当前和下个状态的`props`和`state`。因此，每当`shouldComponentUpdate`被调用时，组件默认的会重新渲染。

### 二、 页面书写

#### 1、防止首页白屏

首页打开 先请求数据，为了防止白屏，使用加载动效

只需要在store的初始值，设置为false，当请求完数据就设置为true即可

```react
homeLoading ? <FullLoading /> : HomeRender;
```

```react
import React from 'react';
import { Spin } from 'antd';
import css from './index.less';

export default function FullLoading() {
  return (
    <div className={css.loadWrapper}><Spin size="large" /></div>
  );
}
```

#### 2、轮播即指示器样式

轮播使用 swiper  4.0版本，

页面如下 即 指示器： 

![](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513164542.png)

代码如下： 

```react
  RenderHeader = () => {
    const { store, store: { page } } = this.props;
    const { headers } = page;
    // 渲染视频
    if (headers.length === 1 && headers[0].endsWith('mp4')) {
      return <video id="video" src={headers[0].url} autoPlay={true} muted={true} ref={this.videoRef} />;
    }
    // 轮播swiper
    return (<div className={`${css.homeSwiper} swiper-container`} id="homeSwiper">
      <div className="swiper-wrapper">
        {
          headers.map((item, index) => {
            const sliderProps = {
              key: `swiper${index}`,
              className: `swiper-slide ${css.swiperItem}`,
              style: {
                width: 1080,
                height: 530
              },
            };
            return (
              <div {...sliderProps} onClick={() => this.sliderClick(item)}>
                <img className={css.swiperItemImg} src={item.url} />
              </div>
            );
          })
        }
      </div>
      {/* 分页器 */}
      <div className={`${css.homeSwiperPagination} swiper-pagination`} ref={this.swiperPagRef} />
    </div>);
  }
```

指示器样式： 

```less
.homeSwiper{
  color: #000;
  .homeSwiperPagination{
    // bottom:70px;
    z-index: 0;
    :global{
      .swiper-pagination-bullet{
        width: 40px;
        height: 4px;
        background: #D7D7D7;
        position: relative;
        margin: 0 5px;
        opacity: 1;
        border-radius: 0;
        i{
          background: #000;
          height: 4px;
          position: absolute;
          top: 0;
          left: 0;
          right: 40px;
          bottom:0;
          animation-fill-mode:forwards;
          -webkit-animation-fill-mode:forwards;
          &.end{
            right:0;
          }
          &.isFull{
            animation: toRigt  8.9s linear;
            // animation-fill-mode:forwards;
            -webkit-animation: toRigt  8.9s linear;
            // -webkit-animation-fill-mode:forwards;
          }
        }
        &.swiper-pagination-bullet-active{
          i{
            animation: toRigt  8.8s linear;
            animation-iteration-count:1;
            animation-fill-mode:forwards;
            -webkit-animation: toRigt  8.8s linear;
            -webkit-animation-iteration-count:1;
            -webkit-animation-fill-mode:forwards;
          }
        }
      }
    }
  }
}
```

#### 3、折叠展示品牌 

![image-20200513164707909](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513172736.png)

展示全部：

![image-20200513164757262](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513172737.png)

有个过度效果,

代码如下： 

```react
import React, { PureComponent } from 'react';
import { inject, observer } from 'mobx-react';
import { withRouter } from 'react-router-dom';
import css from './index.less';

@withRouter // 为了拿到 history、location、match
@inject('store')
@observer
export default class BrandWall extends PureComponent {
  constructor(props) {
    super(props);
    this.state = {
      showAllBrand: false // 是否展开全部
    };
  }
  // 品牌墙展开与收起
  toggleAllBrandPanel = () => {
    this.setState(prevState => ({ showAllBrand: !prevState.showAllBrand }));
  }
  // 品牌点击去往品牌详情
  toBrandDetail = brandNum => {
    // 数据埋点 为后台提供 用户的点击数据
    this.props.store.feedbackData({ from: 'home', to: 'brand', msg: 'show_brand_detail', action: 'click_brand_name', bindId: brandNum });
    this.props.store.showBrandDetailNum(brandNum);
    this.props.history.push('brandDetail');
  }
  render() {
    const { store: { page } } = this.props;
    const { detail: brandList } = page.brand;
    const { showAllBrand } = this.state;
    const fiveBrandList = !showAllBrand && brandList ? brandList.slice(0, 5) : brandList;
    if (!brandList) return null;
    const brandHeight = Math.ceil(fiveBrandList.length / 5) * 86 + 60;
    return (
      <div className={showAllBrand ? `${css.noPadding} ${css.brandContent}` : css.brandContent}>
        <div className={css.brandContainer} style={{ height: showAllBrand ? `${brandHeight}px` : '' }}>
          <div className={css.brandBox}>
            {
              fiveBrandList.map((item, index) => {
                return (
                  <div key={`${item.alpha}_${index}`} className={css.brandItem} onClick={() => this.toBrandDetail(item.alpha)}>
                    <img src={item.url} className={css.brandImg} />
                  </div>
                );
              })
            }
          </div>
        </div>
        <div className={css.allBrandIcon} onClick={this.toggleAllBrandPanel}>
          {showAllBrand ? '收起' : '全部'}
          <span className={showAllBrand ? css.up : ''} />
        </div>
      </div>
    );
  }
}

```

css:

```less
@white: #ffffff;
// 品牌列表
.brandContent{
    padding: 0 70px;
    position: absolute;
    z-index: 10;
    top:0;
    transition: padding 0.5s;
    &.noPadding{
        padding: 0;
        .brandContainer{
        padding:30px 126px 30px 70px;
        height: 300px;
        box-shadow:0px 0px 20px 0px rgba(0,0,0,0.2);
        .brandBox{
            .brandItem{
            margin-bottom: 20px;
            }
        }
        }
    }
    .allBrandIcon{
        font-size: 14px;
        color: #333333;
        position: absolute;
        right: 90px;
        top:40px;
        font-weight: 600;
        span{
            display: block;
            width: 24px;
            height: 12px;
            background: url('assets/images/home/icon_down.png') 0 0 no-repeat;
            background-size: contain;
            margin:6px auto 0;
            transform: rotate(0);
            // transition: transform, 0.5s;
            &.up{
                transform: rotate(-180deg)
            } 
        }
    }
    .brandContainer{
        position: relative;
        padding: 30px 56px 30px 0;
        background-color: @white;
        height: 126px;
        box-shadow: -5px 0px 5px -4px rgba(0,0,0,.2), 5px -2px 5px -4px rgba(0,0,0,.2);
        transition: padding .5s ease, height .5s ease;
        overflow: hidden;
    }
    }
    // 品牌
    .brandBox{
    display: flex;
    flex-wrap: wrap;
    .brandItem{
        display: flex;
        align-items: center;
        .brandImg{
            object-fit: cover;
            width: 160px;
            height: 66px;
            // transform: scale(0.75);
        }
        &::after{
        content: '';
        display: block;
        height: 60px;
        background-color: #eee;
        width: 1px;
        margin: 0 10px;
        }
        &:nth-child(5n+5){
        &::after{
            display:none;
        }
        }
    }
}
```

#### 4、菜单切换

下面有条横向跟着切换菜单，不是瞬移的。

![image-20200513170507754](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513172738.png)

代码如下： 

```react
import React, { Component } from 'react';
import { inject, observer } from 'mobx-react';
import { Menu } from 'antd';
const { Item } = Menu;
import css from './index.less';

@inject('store')
@observer
export default class ClassifyList extends Component {
  constructor(props) {
    super(props);
    this.activeLineRef = React.createRef();
    this.menuListRef = React.createRef();
  }
  // 点击菜单动态设置跟随线位置
  setActiveLinePosition = (groupNum, e) => {
    const { flowFromData = 'home' } = this.props;
    // 数据回流
    this.props.store.feedbackData({ from: flowFromData, to: 'group', msg: 'change_group', action: 'click_home_group', bindId: groupNum });
    this.props.setTabIndex(groupNum);
    this.activeLineRef.current.style.left = e.item.node.offsetLeft + 'px'; // 计算位置 改变位置
  }
  render() {
    const { store, store: { page } } = this.props;
    const { group: { detail: MenuList } } = page;
    return (
      <div className={css.menuBox}>
        <div className={css.scorllBox}>
          <Menu mode="horizontal" className={css.menuContainer} ref={this.menuListRef}>
            {
              MenuList.map(item => {
                return (
                  <Item className={css.homeMenuItem} data-item={item.alpha} key={`${item.alpha}_${item.index}`} onClick={e => this.setActiveLinePosition(item.alpha, e)}>
                    <img src={item.url} />
                  </Item>
                );
              })
            }
          </Menu>
          <span className={css.activeLine} ref={this.activeLineRef} />
        </div>
      </div>
    );
  }
}
```

css:

```less
.menuBox{
    position: relative;
    margin-bottom: 50px;
    text-align: left;
    width: 1010px;
    overflow: auto;
    padding-left: 20px;
    &::-webkit-scrollbar {/*滚动条整体样式*/
      width: 0;     /*高宽分别对应横竖滚动条的尺寸*/
      height: 0;
    }
    .scorllBox{
      display: inline-block;
      position: relative;
    }
    .menuContainer{
      border-bottom: 5px solid #eee;
      padding-bottom: 8px;
    }
    .activeLine{
      transition: left 0.4s;
      display: inline-block;
      width: 225px;
      height: 5px;
      background-color: #333;
      position: absolute;
      bottom: 0;
      left: 0;
    }
  }
  .homeMenuItem{
    padding: 0;
    border-bottom: none;
    text-align: center;
    height: 60px;
    line-height: 60px;
  }
  :global{
    .ant-menu-horizontal>.ant-menu-item-active, .ant-menu-horizontal>.ant-menu-item, .ant-menu-horizontal>.ant-menu-item-open, .ant-menu-horizontal>.ant-menu-item-selected, .ant-menu-horizontal>.ant-menu-item:hover, .ant-menu-horizontal>.ant-menu-submenu-active, .ant-menu-horizontal>.ant-menu-submenu-open, .ant-menu-horizontal>.ant-menu-submenu-selected, .ant-menu-horizontal>.ant-menu-submenu:hover{
        border-bottom: none;
    }
  }
```

#### 5、购买的实时二维码

![image-20200513171515595](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513172739.png)

用Portal传送门 显示   倒计时 刷新二维码

代码如下： 

```react
import React, { PureComponent } from 'react';
import QRCode from 'qrcode.react';
import { Modal } from 'antd';
import { inject, observer } from 'mobx-react';
import css from './index.less';

@inject('store')
@observer
export default class DiaLog extends PureComponent {
  state = {
    countDown: 120 // 120s倒计时
  }
  static defaultProps = {
    qUrl: '',
    chooseGoodsId: '',
    isCoupon: false
  };
  UNSAFE_componentWillReceiveProps = nextProps => {
    if (nextProps.chooseGoodsId || nextProps.qUrl) {
      this.setCountTime();
    }
    if ((!nextProps.chooseGoodsId && !nextProps.qUrl) || !nextProps.show) {
      if (this.setCountDownTime) {
        this.clearCountTime();
      }
    }
  }
  // 设置120s倒计时定时器
  setCountTime = () => {
    this.clearCountTime();
    this.setCountDownTime = setInterval(() => {
      this.setState(prevState => ({ countDown: prevState.countDown - 1 }), () => {
        if (this.state.countDown === 0) { // 120s结束
          this.props.closeDialog();
          this.clearCountTime();
        }
      });
    }, 1000);
  }
  // clear
  clearCountTime = () => {
    if (this.setCountDownTime) {
      clearInterval(this.setCountDownTime);
      this.setCountDownTime = null;
      this.setState(() => ({ countDown: 120 }));
    }
  }
  componentWillUnmount() {
    this.clearCountTime();
  }
  render() {
    const { store, chooseGoodsId, closeDialog, show, qUrl = '', isCoupon } = this.props;
    const { countDown } = this.state;
    const qrcodeProps = {
      size: 305,
      value: qUrl ? qUrl : `https://detail.m.liangxinyao.com/item.htm?id=${chooseGoodsId}&tmgRetail=sfl-${store.storeId}-2`
    };
    if (chooseGoodsId || qUrl) {
      return (
        <Modal
          className={css.payQcodeBox}
          title=""
          closable={false}
          footer={null}
          visible={show}
          width={825}
          style={{ top: 306 }}
        >
          <div className={css.blurBg} />
          <div className={css.payDiaLogBox}>
            {isCoupon && <div className={css.getCoupon}>扫码领取优惠券</div>}
            <span className={css.closeIcon} onClick={closeDialog} />
            <span className={css.countDown}>{countDown}</span>
            {/* 二维码弹窗box */}
            <div className={css.qcodeBox}>
              <QRCode {...qrcodeProps} />
            </div>
          </div>
          
        </Modal>
      );
    }
    return null;
  }
}
```

#### 6、品牌滑动展示

![image-20200513172050174](https://gitee.com/supcoolMa/store_pictures/raw/master/img/20200513172740.png)

使用的是 BetterScroll   即 '@better-scroll/core'插件

参考：1、 [官方案例展示](https://ustbhuangyi.github.io/better-scroll/#/examples/vertical-scroll)

 			2、 [掘金文章--- BetterScroll](https://juejin.im/post/59b777015188257e764c716f)

代码如下：

```react
import React, { Component } from 'react';
import BScroll from '@better-scroll/core';
import { inject, observer } from 'mobx-react';
import css from './index.less';

@inject('store')
@observer
export default class BrandsList extends Component {
  constructor(props) {
    super(props);
    this.brandListRef = React.createRef(); // 最外层容器
    this.brandsScrollRef = React.createRef(); // scroll
    this.brandsContentRef = React.createRef(); // 品牌

    this.touch = {};
    this.scrollY = 0;// y轴的滚动值，默认-0
    this.state = {
      currentIndex: 0, // 默认的字母索引位置-0
    };
  }
  componentDidMount() {
    this.setState({ currentIndex: 0 });
    this.initScroll();
    this.calculateHeight(); // 初始化总高度
    if (localStorage.getItem('brandScrollTop')) {
      const moveY = localStorage.getItem('brandScrollTop') ? Number(localStorage.getItem('brandScrollTop')) : 0;
      this.scroll.scrollTo(0, moveY);
    }

  }
  // 初始化scroll
  initScroll = () => {
    this.scroll = new BScroll(this.brandsScrollRef.current, {
      scrollY: true,
      click: true,
      probeType: 3
    });

    // 监听滚动偏移
    this.scroll.on('scroll', (pos) => {
      this.scrollY = pos.y;
      localStorage.setItem('brandScrollTop', Number(pos.y));
      // 偏移动态设置字母索引值
      this.setCurrentIndex(this.scrollY); // 当前滚动y轴的偏移量
    });
  }
  // 记录容器总高度值-包含子元素撑起的总dom高度
  calculateHeight = () => {
    this.listHeight = [];
    const list = this.brandsContentRef.current.children;
    let height = 0;
    this.listHeight.push(height);
    for (let i = 0; i < list.length; i++) {
      let item = list[i];
      height += item.clientHeight;
      this.listHeight.push(height);
    }
  }
  // 跳转到content dom处
  scrollToElement = index => {
    // 处理边界情况
    // 因为 index 通过滑动距离计算出来的
    // 所以向上滑超过索引框框的时候就会 < 0，向上就会超过最大值
    if (index < 0) {
      return;
    } else if (index > this.listHeight.length - 2) {
      index = this.listHeight.length - 2;
    }
    // listHeight 是正的， 所以加个 -
    this.scrollY = -this.listHeight[index];
    this.scroll.scrollToElement(this.brandsContentRef.current.children[index]);

  }
  // 字母索引触摸点击跳转
  onShortcutStart = (e) => {

    let index = e.target.getAttribute('data-index');
    // 使用 better-scroll 的 scrollToElement 方法实现跳转
    this.scrollToElement(index);
    // 记录一下点击时候的 Y坐标 和 index
    let firstTouch = e.touches[0].pageY;
    this.touch.y1 = firstTouch;
    this.touch.anchorIndex = index;
  }
  // 字母索引的触摸偏移跳转
  onShortcutMove = e => {
    // 再记录一下移动时候的 Y坐标，然后计算出移动了几个索引
    let touchMove = e.touches[0].pageY;
    this.touch.y2 = touchMove;
    // 这里的 24 是索引元素的高度
    let delta = Math.floor((this.touch.y2 - this.touch.y1) / 24);
    // 计算最后的位置
    let index = this.touch.anchorIndex * 1 + delta;
    this.scrollToElement(index);
  }
  // 动态设置字母索引- newVal -y轴的滚动高度
  setCurrentIndex = newVal => {
    let currentIndex = 0;
    if (newVal > 0) {
      this.setState({ currentIndex });
      return;
    }

    // 计算 currentIndex 的值
    for (let i = 0; i < this.listHeight.length - 1; i++) {
      let height1 = this.listHeight[i];
      let height2 = this.listHeight[i + 1];
      if (-newVal >= height1 && -newVal < height2) {
        currentIndex = i;
        this.setState({ currentIndex });
        return;
      }
    }

    // 当超 -newVal > 最后一个高度的时候
    this.setState({ currentIndex: this.listHeight.length - 2 });

  }
  // 组件返回退出
  goBack = () => {
    // 销毁滚动
    this.scroll.stop();
    this.scroll.destroy();
    this.scroll = null;
    this.props.history.goBack();
    localStorage.removeItem('brandScrollTop');
  }
  // 过滤首字母
  shortcutList = () => {
    if (this.props.store.brandIndexList) {
      return this.props.store.brandIndexList.map((group) => {
        return group.title.substr(0, 1);
      });
    }
    return [];
  }
  // 点击进入品牌详情
  toBrandDetail = brandNum => {
    this.props.store.showBrandDetailNum(brandNum);
    this.props.store.feedbackData({ from: 'brand_list', to: 'brand', msg: 'show_brand_detail', action: 'click_brand_name', bindId: brandNum });
  }
  render() {
    const { currentIndex } = this.state; // 当前字母索引位置
    const { store: { brandIndexList } } = this.props;
    return (
      <div>
        <div className={css.wrapper} ref={this.brandListRef}>
          <div className={css.brandsWrapper} ref={this.brandsScrollRef}>
            {/* 品牌列表 */}
            <div className={css.brandsListContent} ref={this.brandsContentRef}>
              {
                brandIndexList.map((item, index) => {
                  return (
                    <div className={css.brandsItemBox} key={`${item.title}_${index}`}>
                      <p className={css.brandsItemTitle}>{item.title}</p>
                      <ul>
                        {
                          item.items.map(item => {
                            return (
                              <li key={`${item.alpha}_${item.index}`} onClick={() => {
                                this.toBrandDetail(item.alpha);
                                this.props.history.push('brandDetail');
                              }}>
                                <img src={item.url} className={css.brandsImg} />
                                {/* {item.english} */}
                              </li>
                            );
                          })
                        }
                      </ul>
                    </div>
                  );
                })
              }
            </div>
          </div>
          {/* 右侧字母索引 */}
          <div className={css.shortCutBox}>
            <ul>
              {
                this.shortcutList().map((item, index) => {
                  return (
                    <li key={`${item}_${index}`} onTouchMove={this.onShortcutMove} onTouchStart={this.onShortcutStart} data-index={index} className={currentIndex === index ? `${css.shortCutItem} ${css.current}` : css.shortCutItem}>{item}</li>
                  );
                })
              }
            </ul>
          </div>
          {/* topTitle */}
          <div className={css.topTitle}>
            所有品牌
            <div className={css.goBackIcon} onClick={this.goBack} />
          </div>
        </div>
      </div>
    );
  }
}
```

css: 

```less
@white:#ffffff;
.wrapper{
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: @white;
  overflow: hidden;
  animation: fadeIn 1s;
  .topTitle{
    position: absolute;
    right: 0;
    left: 0;
    top:0;
    height: 144px;
    line-height: 144px;
    font-size: 36px;
    color: #333;
    background-color: #fbfbfb;
    text-align: center;
    .goBackIcon{
      position: absolute;
      top:42px;
      background: url('assets/images/goodsDetail/goback.png') 0 0 no-repeat;
      width: 60px;
      height: 60px;
      left: 30px;
      background-size: contain;
    }
  }
  .brandsWrapper{
    position: absolute;
    top: 144px;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: @white;
    overflow: hidden;
    .brandsListContent{
      .brandsItemBox{
        position: relative;
        // padding-right: 152px;
        padding: 40px 152px 40px 0;
        display: flex;
        align-items: center;
        &::after{
          left: 0;
          position: absolute;
          right: 152px;
          bottom: 0;
          content: '';
          display: block;
          height: 4px;
          background-color: #eee;
        }
        .brandsItemTitle{
          font-size: 36px;
          color: #000;
          margin-bottom: 0;
          padding-left: 70px;
        }
        ul{
          padding-bottom: 0;
          padding-left: 144px;
          margin: 0;
          display: flex;
          flex-flow: wrap;
        } li{
          // outline: 1px solid red;
          list-style: none;
          margin-bottom: 10px;
          display: flex;
          align-items: center;
          font-size: 28px;
          color: #333;
          padding-left: 0;
          width: 194px;
          height: 144px;
          margin-right: 10px;
          img{
            width: 194px;
            height: 144px;
            // margin-right: 100px;
            object-fit: cover
          }
        }
      }
    }
  }
  // 字母索引
  .shortCutBox{
    position: absolute;
    top:50%;
    z-index: 3;
    right: 83px;
    width: 21px;
    text-align: center;
    transform: translateY(-50%);
    .shortCutItem{
      list-style: none;
      font-size: 16px;
      color: #666;
      width: 16px;
      margin-bottom: 21px;
      &.current{
        background-color: #000;
        color: #fff;
        font-size: 24px;
        width: 35px;
        height: 35px;
        border-radius: 50%;
        line-height: 35px;
        transform: translateX(-25%);
      }
    }
  }
}
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
```

