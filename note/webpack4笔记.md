# webpack4笔记

## Loaders

* 用法：**test**指定匹配规则，**use**指定使用的loader名称

  ```js
  const path = require('path');
  module.exports = {
      output: {
      	filename: 'bundle.js'
      },
      module: {
          rules: [
          	{ test: /\.txt$/, use: 'raw-loader' }
          ]
      }
  };
  ```

### 解析CSS

* css-loader 用于加载 .css ⽂文件，并且转换成 commonjs 对象
* style-loader 将样式通过<style>标签插入到 head 中

#### CSS3属性加前缀

IE：Trident(-ms)  火狐：Geko(-moz)  谷歌：Webkit(-webkit)  欧朋：Presto(-o)

```css
.box {
    -moz-border-radius: 10px;
    -webkit-border-radius: 10px;
    -o-border-radius: 10px;
    border-radius: 10px;
}
```

使用**PostCSS** 插件 **autoprefixer** 自动补齐 CSS3 前缀

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.less$/,
                use: [
                    'style-loader',
                    'css-loader',
                    'less-loader',
                    + {
                        + loader: 'postcss-loader',
                        + options: {
                            + plugins: () => [
                                + require('autoprefixer')({
                                + browsers: ["last 2 version", "> 1%", "iOS 7"]
                                + })
                            + ]
                        + }
                    + }
                ]
            }
        ]
    }
};
```

#### 移动端CSS px自动转换rem

使用**px2rem-loader**和**lib-flexible**

入口文件main.js要引入lib-flexible

```
npm i lib-flexible -S
npm i px2rem-loader -D
```

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.less$/,
                use: [
                    'style-loader',
                    'css-loader',
                    'less-loader',
                    + {
                        + loader: "px2rem-loader",
                        + options: {
                            + remUnit: 75,
                            + remPrecision: 8
                        + }
                    + }
                ]
            }
        ]
    }
};
```



### 解析图片、字体

1. file-loader

2. url-loader:可以设置较小资源⾃自动 base64

   ```js
   module: {
       rules: [
       + {
           + test: /\.(png|svg|jpg|gif)$/,
           + use: [{
               + loader: 'url-loader’,
               + options: {
               + 	limit: 10240
               + }
           + }]
       + }
       ]
   }
   ```

## Plugins

* 插件用于 bundle ⽂文件的优化，资源管理和环境变量注⼊

* 作用于整个构建过程

* 用法：

  ```js
  const path = require('path');
  module.exports = {
      output: {
      	filename: 'bundle.js'
      },
      plugins: [
      	new HtmlWebpackPlugin({
              template:'./src/index.html'
          })
      ]
  };
  ```
### 代码压缩

#### js文件的压缩

内置了**uglifyjs-webpack-plugin**

#### CSS文件的压缩

* **optimize-css-assets-webpack-plugin**
* 同时使用**cssnano**

```js
module.exports = {
    entry: {
        app: './src/app.js',
        search: './src/search.js'
    },
    output: {
        filename: '[name][chunkhash:8].js',
        path: __dirname + '/dist'
    },
    plugins: [
    + new OptimizeCSSAssetsPlugin({
    + 	assetNameRegExp: /\.css$/g,
    + 	cssProcessor: require('cssnano’)
    + })
    ]
};
```

#### html文件的压缩

修改**html-webpack-plugin**，设置压缩参数

```js
module.exports = {
    entry: {
        app: './src/app.js',
        search: './src/search.js'
    },
    output: {
        filename: '[name][chunkhash:8].js',
        path: __dirname + '/dist'
    },
    plugins: [
    + new HtmlWebpackPlugin({
    + 	template: path.join(__dirname, 'src/search.html’),
    + 	filename: 'search.html’,
    + 	chunks: ['search’],
    + 	inject: true,
    + 	minify: {
    + 		html5: true,
    + 		collapseWhitespace: true,
    + 		preserveLineBreaks: false,
    + 		minifyCSS: true,
    + 		minifyJS: true,
    + 		removeComments: false
    + 	}
    + })
    ]
};
```

### 自动清理构建目录

避免构建前每次都要手动删除dist，使用**clean-webpack-plugin**，默认会删除output指定的输出目录

```js
module.exports = {
    entry: {
        app: './src/app.js',
        search: './src/search.js'
    },
    output: {
        filename: '[name][chunkhash:8].js',
        path: __dirname + '/dist'
    },
    plugins: [
    + 	new CleanWebpackPlugin()
    ]
};
```

### 提取页面公共资源

#### 使用html-webpack-externals-plugin

* 将react、react-dom等基础包通过cdn引入，不打入bundle中

* 使用**html-webpack-externals-plugin**

  ```js
  const HtmlWebpackExternalsPlugin = require('html-webpack-externals-plugin')
  
  plugins: [
      new HtmlWebpackExternalsPlugin({
          externals:[
              {
                  module:'react',
                  entry:'//链接',
                  global:'React'
              }, {
                  module:'react-dom',
                  entry:'//链接',
                  global:'ReactDom'
              }
          ]
      })
  ]
  ```

#### 使用SplitChunksPlugin

Webpack4 内置的，替代CommonsChunkPlugin插件

* 公共脚本分离：

  ```js
  module.exports = {
      optimization: {
          splitChunks: {
          chunks: 'async',
          minSize: 30000,
          maxSize: 0,
          minChunks: 1,
          maxAsyncRequests: 5,
          maxInitialRequests: 3,
          automaticNameDelimiter: '~',
          name: true,
          cacheGroups: {
                  vendors: {
                      test: /[\\/]node_modules[\\/]/,
                      priority: -10
                  }
              }
          }
      }
  };
  ```

  chunks参数说明：

  * async 异步引入的库进行分离（默认）
  * initial 同步引入的库进行分离
  * all 所有引入的库进行分离（推荐）

* 分离基础包

  test：匹配出需要分离的的包

  ```js
  module.exports = {
      optimization: {
          splitChunks: {
              cacheGroups: {
                  commons: {
                      test: /(react|react-dom)/,
                      name: 'vendors',
                      chunks: 'all'
                  }
              }
          }
      }
  };
  ```

* 分离页面公共文件

  ```js
  module.exports = {
      optimization: {
          splitChunks: {
              minSize: 0,
              cacheGroups: {
                  commons: {
                          name: 'commons',
                          chunks: 'all',
                          minChunks: 2
                      }
                  }
              }
          }
      }
  };
  ```

  minChunks:设置最小引用次数为两次

  minuSize：分离的包体积的大小

### scope hoisting 

原理：将所有模块的代码按照引用顺序放在一个函数作用域里，然后适当的重命名一些变量以防止变量名冲突

对比: 通过 scope hoisting 可以减少函数声明代码和内存开销

使用：webpack mode 为 production 默认开启，必须是 ES6 语法，CJS 不支持

```js
module.exports = {
    entry: {
        app: './src/app.js',
        search: './src/search.js'
    },
    output: {
        filename: '[name][chunkhash:8].js',
        path: __dirname + '/dist'
    },
    plugins: [
    + 	new webpack.optimize.ModuleConcatenationPlugin()
    ]
};
```





## 文件监听

文件监听是在发现源码发⽣生变化时，自动重新构建出新的输出⽂文件。

**缺陷**：每次需要⼿手动刷新浏览器

> 开启监听模式：

* 启动 webpack 命令时，带上 --watch 参数
* 在配置 webpack.config.js 中设置 watch: true

> 文件监听的原理分析:

 轮询判断文件的最后编辑时间是否变化
 某个文件发生了了变化，并不会立刻告诉监听者，而是先缓存起来，等 aggregateTimeout

 ```js
 module.export = {
     //默认 false，也就是不不开启
     watch: true,
     //只有开启监听模式时，watchOptions才有意义
     wathcOptions: {
     //默认为空，不监听的文件或者文件夹，支持正则匹配
     ignored: /node_modules/,
     //监听到变化发生后会等300ms再去执行，默认300ms
     aggregateTimeout: 300,
     //判断文件是否发生变化是通过不停询问系统指定文件有没有变化实现的，默认每秒问1000次
     poll: 1000
     }
 }
 ```

## 文件指纹

指打包后输出的文件名的后缀

* **Hash**：只要项目文件有修改，hash值就会更改
* **Chunkhash**：和webpack的chunk有关，不同的entry会生成不同的Chunkhash值
* **Contenthash**:根据文件内容定义hash，内容不变则contenthash不变

用法示例：`filename: '[name][chunkhash:8].js'`

```js
options: {
 name: 'img/[name][hash:8].[ext] '
}
```



| 占位符名称    | 含义                          |
| ------------- | ----------------------------- |
| [ext]         | 资源后缀名                    |
| [name]        | 文件名称                      |
| [path]        | 文件的相对路径                |
| [folder]      | 文件所在的文件夹              |
| [contenthash] | 文件的内容hash，默认是MD5生成 |
| [hash]        | 文件内容的Hash，默认是MD5生成 |
| [emoji]       | 一个随机的指代文件内容的emoji |

## 资源内联

1. 代码层面意义：
   * 页面框架的初始化脚本
   * 上报相关打点
   * css内联避免页面闪动
2. 请求层面：减少HTTP请求
   * 小图片或者字体内联（url-loader）

### HTML和JS内联

* **raw-loader**

  ```js
  <script>${require(' raw-loader!babel-loader!. /meta.html')}</script>
  <script>${require('raw-loader!babel-loader!../node_modules/lib-flexible')}</script>
  ```

### CSS内联

1. 借助**style-loader**

   ```js
   module.exports = {
       module: {
           rules: [
               {
                   test: /\.scss$/,
                   use: [
                       {
                       loader: 'style-loader',
                       options: {
                           insertAt: 'top', // 样式插入到<head>
                           singleton: true, //将所有的style标签合并成一个
                           }
                       },
                       "css-loader",
                       "sass-loader"
                   ],
               },
           ]
       },
   };
   ```

2. **html-inline-css-webpack-plugin**

## 多页面应用(MPA)

每一次页面跳转的时候，后台服务器都会给返回一个新的 html 文档，这种类型的网站也就是多页网站，也叫做多页应用。

### 打包基本思路

每个页面对应一个 entry，一个 html-webpack-plugin

```js
module.exports = {
    entry: {
        index: './src/index.js',
        search: './src/search.js ‘
    }
};
```

缺点：每次新增或删除页面需要改 webpack 配置

### 通用方案

动态获取 entry 和设置 html-webpack-plugin 数量

利用**glob.sync**

```js
module.exports = {
    entry: {
        index: './src/index/index.js',
        search: './src/search/index.js ‘
    }
};
```

转化为：

```js
entry: glob.sync(path.join(__dirname, './src/*/index.js')),
```

## source map

作用：定位到源代码

[科普文](http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)

### source map 关键字

* eval：使用eval包裹模块代码
* source map：产生.map文件
* cheap：不包含列信息
* inline：将.map作为DataURL嵌入，不单独生成.map文件
* module：包含loader的sourcemap

### source map 类型

| devtool                        | 首次构建 | 二次构建 | 是否适合生产环境 | 可以定位的代码                       |
| ------------------------------ | -------- | -------- | ---------------- | ------------------------------------ |
| (none)                         | +++      | +++      | yes              | 最终输出的代码                       |
| eval                           | +++      | +++      | no               | webpack生成的代码（一个个的模块）    |
| cheap-eval-source-map          | +        | ++       | no               | 经过loader转换后的代码（只能看到行） |
| cheap-module-eval-source-map   | o        | ++       | no               | 源代码（只能看到行）                 |
| eval-source-map                | --       | +        | no               | 源代码                               |
| cheap-source-map               | +        | o        | yes              | 经过loader转换后的代码（只能看到行） |
| cheap-module-source-map        | o        | -        | yes              | 源代码（只能看到行）                 |
| inline-cheap-source-map        | +        | o        | no               | 经过loader转换后的代码（只能看到行） |
| inline-cheap-module-source-map | o        | -        | no               | 源代码（只能看到行）                 |
| source-map                     | --       | --       | yes              | 源代码                               |
| inline-source-map              | --       | --       | no               | 源代码                               |
| hidden-source-map              | --       | --       | yes              | 源代码                               |



## tree shaking(摇树优化)

概念：1 个模块可能有多个方法，只要其中的某个方法使用到了了，则整个文件都会被打到bundle 里面去，tree shaking 就是只把用到的方法打入 bundle ，没⽤到的方法会在uglify 阶段被擦除掉。

使用：webpack 默认支持，在 .babelrc 里设置 modules: false 即可

* production mode的情况下默认开启

要求：必须是 ES6 的语法，CJS 的方式不支持

## 代码分割

意义：对于大的 Web 应用来讲，将所有的代码都放在一个文件中显然是不够有效的，特别是当你的某些代码块是在某些特殊的时候才会被使用到。webpack 有一个功能就是将你的代码库分割成chunks（语块），当代码运行到需要它们的时候再进行加载。

适用场景：

* 抽离相同代码到一个共享块

* 脚本懒加载，使得初始下载的代码更小

  * CommonJS：require.ensure

  * ES6：动态 import（目前还没有原生支持，需要 babel 转换）

    * 安装babel插件

      `npm install @babel/plugin-syntax-dynamic-import --save-dev`

    * 配置babel的配置文件：.babelrc

      ```json
      {
          "plugins": ["@babel/plugin-syntax-dynamic-import"],
          ...
      }
      ```

      

## ESLint

* 不重复造轮子，基于 eslint:recommend 配置并改进
* 能够帮助发现代码错误的规则，全部开启
* 帮助保持团队的代码风格统一，而不是限制开发体验

| 规则名称                    | 错误级别 | 说明                                                         |
| --------------------------- | -------- | ------------------------------------------------------------ |
| for-direction               | error    | for循环的方向要求必须正确                                    |
| getter-return               | error    | getter必须有返回值，并且禁止返回值为undefined，比如return;   |
| no-await-in-loop            | off      | 允许在循环里面使用await                                      |
| no-console                  | off      | 允许在代码里面使用console                                    |
| no-prototype-builtins       | warn     | 直接调用对象原型链上的方法                                   |
| valid-jsdoc                 | off      | 函数注释一定要遵守jsdoc规则                                  |
| no-template-curly-in-string | warn     | 在字符串里面出现{和}进行警告                                 |
| accessor-pairs              | warn     | getter和setter没有成对出现时给出警告                         |
| array-callback-return       | error    | 对于数据相关操作函数比如reduce，map，filter等，callback必须有return |
| block-scoped-var            | error    | 把var关键字看成块级作用域，防止变量提升导致的bug             |
| class-methods-use-this      | error    | 要求在class里面合理使用this，如果某个方法没有使用this，则应该申明为静态方法 |
| complexity                  | off      | 关闭代码复杂度控制                                           |
| default-case                | error    | switch case语句里面一定需要default分支                       |

## 打包库和组件

* 暴露库

  ```js
  module.exports = {
      mode: "production",
      entry: {
          "large-number": "./src/index.js",
          "large-number.min": "./src/index.js"
      },
      output: {
          filename: "[name].js",
          library: "largeNumber",
          libraryExport: "default",
          libraryTarget: "umd"
      }
  };
  ```

  library:指定库的全局变量

  libraryTarget:支持库引入的方式

* 指定对.min压缩

  ```js
  module.exports = {
      mode: "none",
      entry: {
          "large-number": "./src/index.js",
          "large-number.min": "./src/index.js"
      },
      output: {
          filename: "[name].js",
          library: "largeNumber",
          libraryTarget: "umd"
      },
      optimization: {
          minimize: true,
          minimizer: [
              new TerserPlugin({
              	include: /\.min\.js$/,
              }),
          ],
      }
  };
  ```

  通过 include 设置只压缩 min.js 结尾的文件

* 设置入口文件

  package.json 的 main 字段为 index.js

  ```js
  if (process.env.NODE_ENV === "production") {
  	module.exports = require("./dist/large-number.min.js");
  } else {
  	module.exports = require("./dist/large-number.js");
  }
  ```

  