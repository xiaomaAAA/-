### nsl后台

#### 1、项目结构 等 可以参考 mobx 手册 

####   （1） ajax请求 

​		使用的是 axios 

```react
import React from 'react';
import axios from 'axios';
import { Modal, Spin } from 'antd';

/**
 * Ajax
 *
 * Ajax.req({ url, params, method })
 */
export default function(options, _params, _method) {
  
  function loading() {
    return <Spin size="large" style={{width: '100%', margin: '40px 0'}}/>;
  }
  
  // 兼容一下入参写法
  loading();
  if (typeof options === 'string') {
    options = {
      url: options,
      params: _params || {},
      method: _method || 'get',
      ignoreError: false, // 忽略错误，不进行错误弹层提示
    };
  }
  let { url, params, method = 'get'} = options;
  url = eval(`CFG.api.${url}`);
  method = method.toLowerCase();
  // get 请求要用 params 包一层
  if (method === 'get') {
    params = {
      params
    };
  }
  if (method === 'delete') {
    params = {
      data: params
    };
  }

  // 发起请求
  return axios[method](url, params).then(resp => {
    return new Promise((resolve, reject) => {
      const respData = resp.data;
      const { code, data } = respData;
      if (code !== 0) {
        reject(respData);
      } else {
        loading(false);
        resolve(data, respData);
      }
    });
  }).catch((error) => {
    loading(false);
    let message = '';
    if (error.msg) {
      message = error.msg;
    } else message = '系统繁忙，请稍后再试。';

    if (error.response) {
      console.warn('Ajax error.response', error.response);
    } else if (error.request) {
      console.warn('Ajax error.request', error.request);
    } else if (error.message) {
      console.warn('Ajax error.message', error.message);
    }
    console.warn('Ajax error', error);

    if (error.notice ) {
      message = error.notice;
    }

    if (!options.ignoreError) {
      Modal.error({
        title: '抱歉',
        content: (
          <p>{message}</p>
        )
      });
    }
  });
}
```

 		使用方法如下： 

```react
import ajax from 'common/utils/ajax';
// ！！不写方法 默认是get 请求 
/**
 * 当前门店 导购发送出去的赠品 信息
 * @param params
 */
export const sendOutList = async({pageNo, pageSize, storeId}) => await ajax({
  url: 'store.sendOutList',
  params: {pageNo, pageSize, storeId},
});

export const channelDelete = async(params) => await ajax({
  url: 'store.channelDelete',
  params,
  method: 'post',
});
```







#### 2、项目难点

#### 	（1）处理时间

​			由于服务端传过来的时间是UTC格式 2020-01-14T07:56:32.000Z 要转化为  2017-03-31 08:02:06这种格式呈现在页面

  方法： 

```javascript
 time(date) {
    // utc 时间 转 当地时间
    let T = date.indexOf('T');
    let Z = date.indexOf('Z');
    let ymd = date.substr(0, T);
    let hms = date.substr(T + 1, Z - T - 1);
    let newDate = ymd + ' ' + hms; // 2017-03-31 08:02:06
    // 处理成为时间戳
    timestamp = new Date(Date.parse(newDate));
    timestamp = timestamp.getTime();
    timestamp = timestamp / 1000;

    // 增加8个小时，北京时间比utc时间多八个时区
    let timestamp = timestamp + 8 * 60 * 60;

    // 时间戳转为时间
    let localTime = new Date(parseInt(timestamp) * 1000).toLocaleString().replace(/年|月/g, '-').replace(/日/g, '');
    return localTime;
  }
```

​    

#### （2）导出表格 

​	将tabel 表单导出 excal表格展示 

​    <font color='red'>**难点**</font> ：

##### （一）、在于 怎么将 图片 转化为 url格式，

​	因为在后台界面要展示 图片，但在 表格中 展示为 url 格式。所以，我写两个表格，一个为展示后台，一个隐藏起来不展示用于导出

##### （二） 导出

   使用的是 antd 自带的一个方法 ,

​	 因为要控制点击事件等，所以在他的api上，修改了一下，大体不变。

```react
import ReactHTMLTableToExcel from 'components/exportExcelList';
// import ReactHTMLTableToExcel from 'react-html-table-to-excel';

  async getListAction() {
    const {shop} = this.props;
    if (!shop.sendList.length) return message.warning('暂无数据，无法下载');
    let list = [];
    if (!this.state.list.length) {
      list = await this.props.shop.sendOutAllList(shop.buyer, shop.storeName);
    }
    const table = this.tableRef.current.querySelector('table');
    table.setAttribute('id', 'table-to-xls');
    if (!this.state.list.length) {
      document.getElementById('test-table-xls-button').click();
    }
    this.setState({
      list: list
    });
  }

<span onClick={this.getListAction.bind(this)} id="target">
          <ReactHTMLTableToExcel
            id="test-table-xls-button"
            table="table-to-xls"
            filename={`${this.props.storeName}_导购中心`}
            sheet="导购中心"
            className={css.exc}
            print={this.props.print}
            shop={shop}
            downList={shop.exportList}
            buttonText={'导出数据'}/>
        </span>
```

#### （3） 下载安装包 

​	后端处理，用生成链接下载。直接调服务端的接口即可。

#### （4）上传模板

 前端解析，给服务端

```react
// excel.js
import React, { Component } from 'react';
import { Button, Icon, message } from 'antd';
import * as XLSX from 'xlsx';
import styles from './index.less';

class Excel extends Component {
  onImportExcel = file => {
    // 获取上传的文件对象
    const { files } = file.target;
    // 通过FileReader对象读取文件
    const fileReader = new FileReader();
    fileReader.onload = async event => {
      try {
        const { result } = event.target;
        // 以二进制流方式读取得到整份excel表格对象
        const workbook = XLSX.read(result, { type: 'binary' });
        // 存储获取到的数据
        let data = [];
        // 遍历每张工作表进行读取（这里默认只读取第一张表）
        for (const sheet in workbook.Sheets) {
          // esline-disable-next-line
          if (workbook.Sheets.hasOwnProperty(sheet)) {
            // 利用 sheet_to_json 方法将 excel 转成 json 数据
            data = data.concat(XLSX.utils.sheet_to_json(workbook.Sheets[sheet]));
            // break; // 如果只取第一张表，就取消注释这行
          }
        }
        // 最终获取到并且格式化后的 json 数据
        message.success('上传成功！');
        let batchGiftArray = [];
        data.map(item => {
          batchGiftArray.push(
            {
              giftName: item['赠品名称'],
              giftUrl: item['赠品图片地址'],
              giftQuantity: item['赠品数量'],
              giftCode: item['赠品编码'],
              giftBarcode: item['赠品条码'],
              giftPrice: item['赠品价格'],
              isNeedCheck: item['是否核销（是/否）'] === '是',
              top: item['是否置顶（是/否）'] === '是',
            }
          );
        });
        let end = await this.props.gifts.batchGifts(batchGiftArray);
        if (end) message.success('批量上传赠品成功！');
        else message.error('批量上传赠品失败');
      } catch (e) {
        // 这里可以抛出文件类型错误不正确的相关提示
        message.error('文件类型不正确！');
      }
    };
    // 以二进制方式打开文件
    fileReader.readAsBinaryString(files[0]);
  }
  render() {
    return (
      <div>
        <Button className={styles['upload-wrap']}>
          <Icon type="upload" />
          <input className={styles['file-uploader']} type="file" accept=".xlsx, .xls" onChange={this.onImportExcel} />
          <span className={styles['upload-text']}>批量上传赠品</span>
        </Button>
        <p className={styles['upload-tip']}>支持 .xlsx、.xls 格式的文件</p>
      </div >
    );
  }
}

export default Excel;
```



#### （5） 调换顺序

```react
import React from 'react';
import css from './index.less';

class List extends React.Component {

  constructor(props) {
    super(props);
    this.state = {
      openSelector: false,
      id: '',
      data: []
    };
  }

  componentDidMount() {
  }

  async componentWillReceiveProps(nextProps) {
    if (nextProps) {
      await this.setState({
        data: nextProps.data
      });
    }
  }

  dragStart(e) {
    if (!this.props.brand.exchange) return;
    this.dragged = e.currentTarget;
  }

  dragEnd(e) {
    if (!this.props.brand.exchange) return;
    this.dragged.style.display = 'flex';

    e.target.classList.remove('drag-up');
    this.over.classList.remove('drag-up');

    e.target.classList.remove('drag-down');
    this.over.classList.remove('drag-down');

    let from = Number(this.dragged.dataset.id);
    let to = Number(this.over.dataset.id);

    let data = this.props.brand.dragEnd(from, to);
    this.setState({data: data});
  }

  dragOver(e) {
    if (!this.props.brand.exchange) return;
    e.preventDefault();

    this.dragged.style.display = 'none';

    if (e.target.tagName !== 'LI') {
      return;
    }

    // 判断当前拖拽target 和 经过的target 的 newIndex

    const dgIndex = JSON.parse(this.dragged.dataset.item).newIndex;
    const taIndex = JSON.parse(e.target.dataset.item).newIndex;
    const animateName = dgIndex > taIndex ? 'drag-up' : 'drag-down';


    if (this.over && e.target.dataset.item !== this.over.dataset.item) {
      this.over.classList.remove('drag-up', 'drag-down');
    }

    if (!e.target.classList.contains(animateName)) {
      e.target.classList.add(animateName);
      this.over = e.target;
    }
  }

  render() {
    const {brand, data} = this.props;
    let listItems = data.map((item, i) => {
      return (
        <li
          data-id={i}
          key={i}
          className={css.box}
          draggable={true}
          onDragEnd={this.dragEnd.bind(this)}
          onDragStart={this.dragStart.bind(this)}
          data-item={JSON.stringify(item)}
          onClick={() => {
            this.setState({open: true});
          }}
        >
132
        </li>
      );
    });
    return (
      <div>
        <ul onDragOver={this.dragOver.bind(this)}>
          {listItems}
        </ul>
      </div>
    );
  }
}

export default List;

```



#### （6）上传图片

​		使用的是 antd 的upload 组件

​		设置宽高来控制用户的上传格式，因为有些图片如轮播等 尺寸是定死的。或者有些图片 有最小的宽或高值，以及控制上传的图片格式

```react
import React from 'react';
import {Upload, Icon, Modal, message} from 'antd';
import css from './index.less';

class UploadPic extends React.Component {

  picturesRef = React.createRef();

  constructor(props) {
    super(props);
    this.state = {
      fileList: [],
      isWrong: false
    };
  }

  componentDidMount() {
    const {url} = this.props;
    const fileList = [];
    if (url) {
      let img = new Image();
      img.src = url;
      let result = '';
      img.onload = () => {
        // 图片有限制尺寸 但没 定死
        if (this.props.height) {
          if (this.props.noMax) {
            result = img.width <= this.props.width && img.height <= this.props.height;
          } else {
            if (this.props.width) {
              result = img.width === this.props.width && img.height === this.props.height;
            } else {
              result = img.height === this.props.height;
            }
            if (!result) message.error('上传的图片不符合尺寸要求');
            this.setState({
              isWrong: !result
            });
            this.props.complete(!result);
          }
        }

      };
      fileList.push({
        uid: this.props.uid || ~~(Math.random() * 1000000),
        name: url.slice(url.lastIndexOf('/') + 1),
        status: 'done',
        url: url,
      });
    }
    this.setState({fileList});
  }

  componentWillReceiveProps(nextProps) {
    const fileList = [];
    if (this.props.uid && !nextProps.uid) {
      this.setState({fileList});
    }
  }

  handleChange = ({fileList}) => {
    this.setState({
      isWrong: false
    });
    const file = fileList[0];
    // 读取图片数据
    if (file && file.status === 'done') {
      if (this.props.width || this.props.height) {
        let img = new Image();
        img.src = file.response.data.url;
        let result = '';
        img.onload = () => {
          if (this.props.width) {
            result = img.width === this.props.width && img.height === this.props.height;
          } else result = img.height === this.props.height;
          if (!result) message.error('上传的图片不符合尺寸要求');
          this.setState({
            isWrong: !result
          });
          this.props.complete(!result);
        };
      }
      this.props.getPic(file.response.data.url);
    } else {
      this.setState({fileList: [...fileList]});
    }
  };

  onRemove = (object) => {
    if (this.props.onRemove) {
      this.props.onRemove(object);
    }
  };

  beforeUpload = (file) => {
    const isJpgOrPng = file.type === 'image/jpeg' || file.type === 'image/png';
    if (!isJpgOrPng) {
      message.error('请上传JPG或Png格式图片');
    }
    return isJpgOrPng;
  };

  renderUploadButton() {
    if (this.state.fileList.length === 0) {
      return (
        <div>
          <Icon type="plus"/>
          <div>添加图片</div>
        </div>
      );
    }
    return null;
  }

  render() {
    return (
      <div className={css.box}>
        <Upload
          action="https://qin-back.yooofun.com/api-file/file/nsl/upload"
          listType="picture-card"
          fileList={this.state.fileList}
          onChange={this.handleChange}
          beforeUpload={this.beforeUpload}
          onRemove={this.onRemove}
          method={'post'}
        >
          {this.renderUploadButton()}
        </Upload>
        {this.props.height && this.state.isWrong && <div>
          <div className={css.title}>您上传的图片不符合要求</div>
          <div style={{marginTop: 16}}>上传的图片尺寸要求：</div>
          <div className={css.rules}>{this.props.imgRules}</div>
        </div>}
      </div>
    );
  }
}

export default UploadPic;

```

​      使用如下:

```react
 const props = {
      url: url, // 图片链接
      uid: id, // 用以标明
      noMax: noMax, //有没有最大值
      width: width, // 
      height: height,
      imgRules,
      complete: val => {
        this.setState({isComplete: val});
      },
      getPic: (imgUrl) => {
        this.props.onChange('url', imgUrl);
      }
    };
    return (
      <Upload {...props}/>
    );
```

