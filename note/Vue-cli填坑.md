## 创建项目时发现没有webpack的配置
```
npm install -g @vue/cli-init
vue init webpack my-project
```

## Vue打包CSS顺序改变，导致样式垮掉
项目开发环境下，运行应用程序样式正常，但是通过npm run build打包后，放到服务器上，再次访问，就会出现css错乱，自定义的css文件被其它组件的样式给覆盖了，比如引用了iview.css样式，就会被此文件覆盖。

解决方案：
```js
import Vue from 'vue'
import iView from 'iview'
import 'iview/dist/styles/iview.css'
import App from './App'
import router from './router'
```
在 src 目录下，找到main.js文件并打开，将import 'iview/dist/styles/iview.css';和import App from './App';顺序调换一下即可，引用css的样式放到最前面。

## 路径问题报错
* Refused to apply style from ‘’ because its MIME type (‘text/html’) is not a supported stylesheet MIME type, and strict MIME checking is enabled.

解决方案：
```js
// config/index.js

const path = require('path')

module.exports = {
  build: {
    assetsPublicPath: './', //修改这一行即可
  }
}

```

## 字体图标路径报错
上面那个处理后，发现CSS和JS都正常了，但是字体图标还没，需要处理

解决方案：
webpack utils.js 修改：（build目录下utils.js）
添加 publicPath: '../../'
```js
    if (options.extract) {
      return ExtractTextPlugin.extract({
        use: loaders,
        fallback: 'vue-style-loader',
        publicPath: '../../' // 加这一行
      })
    } else {
      return ['vue-style-loader'].concat(loaders)
    }
```