1.注册和登录时，密码需要经过RSA加密传入后台
```js
import RSA from "../../utils/wx_rsa.js"
    bindPassInput(e) {
      this.setData({
        password: e.detail.value
      })
      //获取应用实例
      var input_rsa = this.data.password;
      var encrypt_rsa = new RSA.RSAKey();
      const publicKey = `
        -----BEGIN PUBLIC KEY-----
      这里填你的公钥
      -----END PUBLIC KEY-----`;
      encrypt_rsa = RSA.KEYUTIL.getKey(publicKey);
      var encStr = encrypt_rsa.encrypt(input_rsa)
      encStr = RSA.hex2b64(encStr);
      console.log("加密结果：" + encStr)
      this.data.rsaPassword = encStr
    },
```
2.存入token到storage,渲染时取出
```js
            wx.setStorage({
              key: "token",
              data: res.data.data
            })
            
    ------------------------------------------
//app.js
App({
  globalData: { 
    // userInfo: null 
    baseURL: "这里自己填上",
    token:'',
    phone:''
  },
  onLaunch: function () {
    let that = this;
    wx.getStorage({
      key: 'token',
      success(res) {
        console.log(res)
        that.globalData.token = res.data.token
        that.globalData.phone = res.data.userInfo.phone
      }
    })
    console.log(this.globalData)
  }
})
```
3.调用上一页(或别的)的函数、设置别的页的数据(传值)
```js
    var pages = getCurrentPages();
    var beforePage = pages[pages.length - 2];
    // 调用列表页的获取数据函数
    beforePage.getEntryData();
    wx.navigateBack({
      delta: 1
    })
    
    let pages = getCurrentPages();
    let prevPage = pages[pages.length - 2];
    prevPage.setData({
      skill: e.target.dataset.name,
      tagId: this.data.tagId
    })
    wx.navigateBack({
      delta: 1,
    })
```
4.页面间传值:通过自定义属性来传入值
```html
<view data-num="{{item2.id}}" data-name="{{item2.name}}" class="{{_num==item2.id?'active':''}}" bindtap="menuClick">{{item2.name}}</view>
```
```js
click: function () {
    var data = this.data.data
    wx.navigateTo({
      url: '../../pages/mem/mem?tagId=' + data.num +'&page=1&tag=' + data.name,
    })
    console.log(data)
  }
  
  ----------------------------------------
  
  onLoad: function (options) {
    const {
      tagId,
      page,
      tag
    } = options;
    // 更新数据
    this.setData({
      tagId,
      page,
      tag
    });
    console.log(this.data)
```
5.上传图片到OSS(阿里云) (阿里云的配置百度就好，跨域比较重要)
```js
//config.js
var fileHost = "";//你的阿里云地址最后面跟上一个/   在你当前小程序的后台的uploadFile 合法域名也要配上这个域名
var config = {
   //aliyun OSS config
   uploadImageUrl: `${fileHost}`, // 默认存在根目录，可根据需求改
   AccessKeySecret: '',        // AccessKeySecret 去你的阿里云上控制台上找
   OSSAccessKeyId: '',         // AccessKeyId 去你的阿里云上控制台上找
   timeout: 87600 //这个是上传文件时Policy的失效时间
};
module.exports = config
```
```js
//uploadFile.js
const env = require('config.js'); //配置文件，在这文件里配置你的OSS keyId和KeySecret,timeout:87600;

const base64 = require('base64.js');//Base64,hmac,sha1,crypto相关算法
require('hmac.js');
require('sha1.js');
const Crypto = require('crypto.js');

/*
 *上传文件到阿里云oss
 *@param - filePath :图片的本地资源路径
 *@param - dir:表示要传到哪个目录下
 *@param - successc:成功回调
 *@param - failc:失败回调
 */
const uploadFile = function (filePath, dir, successc, failc) {
  if (!filePath || filePath.length < 9) {
    wx.showModal({
      title: '图片错误',
      content: '请重试',
      showCancel: false,
    })
    return;
  }

  console.log('上传图片.....');
  //图片名字 可以自行定义，     这里是采用当前的时间戳 + 150内的随机数来给图片命名的
  const aliyunFileKey = dir + new Date().getTime() + Math.floor(Math.random() * 150) + '.png';

  const aliyunServerURL = env.uploadImageUrl;//OSS地址，需要https
  const accessid = env.OSSAccessKeyId;
  const policyBase64 = getPolicyBase64();
  const signature = getSignature(policyBase64);//获取签名
  // console.log(aliyunServerURL, accessid, policyBase64, signature)
  wx.uploadFile({
    url: aliyunServerURL,//开发者服务器 url
    filePath: filePath,//要上传文件资源的路径
    name: 'file',//必须填file
    formData: {
      'key': aliyunFileKey,
      'policy': policyBase64,
      'OSSAccessKeyId': accessid,
      'signature': signature,
      'success_action_status': '200',
    },
    success: function (res) {
      if (res.statusCode != 200) {
        failc(new Error('上传错误:' + JSON.stringify(res)))
        return;
      }
      successc(aliyunServerURL + aliyunFileKey);
    },
    fail: function (err) {
      err.wxaddinfo = aliyunServerURL;
      failc(err);
    },
  })
}

const getPolicyBase64 = function () {
  let date = new Date();
  date.setHours(date.getHours() + env.timeout);
  let srcT = date.toISOString();
  const policyText = {
    "expiration": srcT, //设置该Policy的失效时间，超过这个失效时间之后，就没有办法通过这个policy上传文件了 
    "conditions": [
      ["content-length-range", 0, 5 * 1024 * 1024] // 设置上传文件的大小限制,5mb
    ]
  };

  const policyBase64 = base64.encode(JSON.stringify(policyText));
  return policyBase64;
}

const getSignature = function (policyBase64) {
  const accesskey = env.AccessKeySecret;

  const bytes = Crypto.HMAC(Crypto.SHA1, policyBase64, accesskey, {
    asBytes: true
  });
  const signature = Crypto.util.bytesToBase64(bytes);

  return signature;
}

module.exports = uploadFile;
```
```js
//使用上传图片的页面
var uploadImage = require('../../utils/uploadFile.js')
var util = require('../../utils/util.js')

uploadPhoto:function(){
    let that = this
    wx.chooseImage({
      count: 1,
      sizeType: ['original', 'compressed'],
      sourceType: ['album', 'camera'],
      success(res) {
        console.log(res)
        // tempFilePath可以作为img标签的src属性显示图片
        const tempFilePaths = res.tempFilePaths
        
        var nowTime = util.formatTime(new Date());
        
        uploadImage(tempFilePaths[0],'e-repair/',
          function (result) {
            console.log("======上传成功图片地址为：", result);
            //做你具体的业务逻辑操作
            that.setData({
              imgPath1: result
            })
            // wx.hideLoading();
          }, function (result) {
            console.log(result);
            //做你具体的业务逻辑操作

            // wx.hideLoading()
          })
      }
    })
  }
```

6.分页问题，考虑到是小程序，不做分页器，用上拉下拉事件
```js
  data: {
    i:1
  },
  
  
  getRepair() {
    var that = this
    wx.request({
      url: app.globalData.baseURL + '/api/users/maintenance/worker/order?type=1&pageNum=' + this.data.i,
      header: {
        'Authorization': app.globalData.token
      },
      success: res => {
        console.log(res)
        // 数据设置到页面中
        this.setData({
          repair: res.data.data.rows,
          i:that.data.i + 1
        });
        console.log("获取维修成功")
      },
      fail: (res) => {
        console.log('失败' + res)
      }
    })
  }
  
  
  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    this.getRepair()
  },
  /**
   * 页面相关事件处理函数--监听用户下拉动作
   */
  onPullDownRefresh: function () {
    this.setData({
      i:1
    })
    this.getRepair()
  },

  /**
   * 页面上拉触底事件的处理函数
   */
  onReachBottom: function () {
    this.getRepair()
  },

```
7.勾选的排他解决
```html
<!--mem.wxml-->
<image class="right" src="{{setImg}}" data-name="{{item.name}}" data-worker="{{item.workerId}}" bindtap="chooseMem" hidden="{{data.worker == item.workerId}}"></image>
<image class="right" src="{{chooseImg}}" data-name="{{item.name}}" data-worker="{{item.workerId}}" bindtap="chooseMem" hidden="{{data.worker !== item.workerId}}"></image>
```
8.星级评分

  1)显示星级
  ```html
  <block wx:for="{{item.praiseNum}}">
    <image src='../../assets/selected_star@2x.png'></image>
  </block>
  <block wx:for="{{5 - item.praiseNum}}">
    <image src='../../assets/star@2x.png'></image>
  </block>
  ```
  2)评价时点星级
  ```html
    <block wx:for="{{one_2}}" wx:key="index">
      <image catchtap='in_xin' id='{{index+1}}' data-in='use_sc2' src='../../assets/selected_star@2x.png'></image>
    </block>
    <block wx:for="{{two_2}}" wx:key="index">
      <image catchtap='in_xin' id='{{index+1}}' data-in='use_sc' src='../../assets/star@2x.png'></image>
    </block>
  ```
  ```js
data: {
    one_2: 0, //提交时星级提交这个数据
    two_2: 5
  },
  in_xin: function (e) {
    var in_xin = e.currentTarget.dataset.in;
    var one_2;
    if (in_xin === 'use_sc2') {
      one_2 = Number(e.currentTarget.id);
    } else {
      one_2 = Number(e.currentTarget.id) + this.data.one_2;
    }
    this.setData({
      one_2: one_2,
      two_2: 5 - one_2
    })
  }
  ```
9.支付：支付主要是后台比较麻烦，前端拿到code传回后台拿到openid然后生成订单即可
```js
  //点击支付
  pay(e) {
    var that = this;
    this.setData({
      id: e.currentTarget.dataset.id
    })
    wx.login({
      success:res => {
        wx.request({
          url: app.globalData.baseURL + '/api/users/maintenance/order/payment/wxma',
          method: 'post',
          header: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': app.globalData.token
          },
          data: {
            orderId: that.data.id,
            code: res.code
          },
          success: res => {
            console.log(res)
            console.log(that.data)
            if(res.data.errorCode == "00"){
              var r = JSON.parse(res.data.data)
              console.log(r)
              wx.requestPayment(
                {
                  'timeStamp': r.timeStamp,
                  'nonceStr': r.nonceStr,
                  'package': r.packageValue,
                  'signType': r.signType,
                  'paySign': r.paySign,
                  'success': function (res) {
                    console.log(res)
                  },
                  'fail': function (res) {
                    console.log(res)
                    console.log('fail')
                  }
                })
            }
          },
          fail: (res) => {
            console.log('失败' + res)
          }
        })
      }
    })
  },
```
10.自定义弹出层。主要是写一个遮罩层，并防止滑动穿透。
```html
<!-- 遮罩层 -->
<view class="mask" catchtouchmove="preventTouchMove" wx:if="{{ showModal || orderModal || ruleModal || couponModal }}"></view>
<!-- 弹出层 -->
<view class="modalDlg" wx:if="{{showModal}}" catchtouchmove='touchMove'>
    <!--内容-->
</view>
```
```css
/* 遮罩层 */
.mask{
    width: 100%;
    height: 100%;
    position: fixed;
    top: 0;
    left: 0;
    background: #000;
    z-index: 9000;
    opacity: 0.5;
}
```
```js
  // 禁止屏幕滚动
  preventTouchMove: function () {
  },
  touchMove: function () {
  },
```
11.实现tab栏的点击切换和滑动切换
```html
  <view class="swiper-tab">
    <view class="swiper-tab-item {{currentTab==0?'actived':''}}" data-current="0" bindtap="clickTab">可用优惠券(1)</view>
    <view class="swiper-tab-item {{currentTab==1?'actived':''}}" data-current="1" bindtap="clickTab">不可用优惠券(0)</view>
  </view>
  <swiper style="height: 480rpx" current="{{currentTab}}" duration="300"  bindchange="swiperTab">
      <swiper-item>
        <view class="ticket">
          <image src="../../assets/youxiaoyouhuijuan@2x.png">
            <view class="ticket-left"><text class="count">5</text><text>.0折</text></view>
            <view class="ticket-right">
              <view class="ticket_type">通用优惠券</view>
              <view class="ticket_buttom">
                <view>有效期至2019-07-06</view>
                <view style="margin-left: 70rpx">使用规则 ▼</view>
              </view>
            </view>
            <image src="../../assets/check@2x.png" class="cho"></image>
          </image>          
        </view>
      </swiper-item>
      <swiper-item>
        <view class="none">暂无优惠券</view>
      </swiper-item>
  </swiper>
  <view class="notUse" bindtap="closeCoupon">不使用优惠券</view>
```
```js
data: {
    currentTab: 0,
  },
  //滑动切换
  swiperTab: function (e) {
    var that = this;
    that.setData({
      currentTab: e.detail.current
    });
  },
  //点击切换
  clickTab: function (e) {
    var that = this;
    if (this.data.currentTab === e.target.dataset.current) {
      return false;
    } else {
      that.setData({
        currentTab: e.target.dataset.current
      })
    }
  },
```
12.文本域的字数统计用输入事件时监控输入值的长度实现，字数限制用maxlength属性
```html
<view class="detail">
  <textarea placeholder="请描述返工原由…" placeholder-class="area" class="textarea" bindinput="bindDetailInput" maxlength="150">
    <view class="currentWordNumber">{{currentWordNumber|0}}/150</view>
  </textarea>
</view>
```
```js
bindDetailInput(e) {
    this.setData({
      detail: e.detail.value,
      currentWordNumber: parseInt(e.detail.value.length)
    })
  },
```
13.判断时间是否超过3天，若超过则隐藏按钮。因为值是wx:for渲染的，不好拿到给JS(必须绑定事件)，而不能在刚渲染时就隐藏。所以使用小程序的wxs语法在页面中写事件
```html
<button hidden="{{timestamp.getDay(item.serviceEndTime)}}">申请返修</button>
<wxs module="timestamp">
  var getDay = function (serviceEndTime) {
    var time = getDate().valueOf()
    var endTime = getDate(serviceEndTime).valueOf()
    var day = (time - endTime) / (1000 * 60 * 60 * 24)
    if (day >= 3) {
      return true
    } else {
      return false
    }
  }

module.exports.getDay = getDay;
</wxs>
```
14.节流函数
```js
//utils/util.js
function throttle(fn, gapTime) {
  if (gapTime == null || gapTime == undefined) {
    gapTime = 1500
  }

  let _lastTime = null

  // 返回新的函数
  return function () {
    let _nowTime = + new Date()
    if (_nowTime - _lastTime > gapTime || !_lastTime) {
      fn.apply(this, arguments)   //将this和参数传给原函数
      _lastTime = _nowTime
    }
  }
}

module.exports = {
  throttle: throttle,
}
```
```js
var util = require('../../utils/util.js')
-------------------------------------------------
    click: util.throttle(function () {

    }, 1000),
```