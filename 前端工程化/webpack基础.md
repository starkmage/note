## webpack 是什么

webpack 本质上是一个打包工具，它会根据代码的内容解析模块依赖，帮助我们把多个模块的代码打包。

webpack 会把我们项目中使用到的多个代码模块（可以是不同文件类型），打包构建成项目运行仅需要的几个静态文件。

## 核心概念

* entry: 入口
* output: 输出
* loader: 模块转换器，用于把模块原内容按照需求转换成新内容
* 插件(plugins): 扩展插件，在webpack构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要做的事情

## 初始化项目

npm init 命令

* 生成 package.json 文件和 package-lock.json 文件

项目本地安装 webpack

* npm install webpack@版本号 --save-dev
* package.json 文件中 scripts 设置 build

## 几个区分

### 两种环境

即 `process.env.NODE_ENV` 有 `'production'` 和 `'development'` 两种环境，生产环境和开发环境，一般我们用 `npm run build` 打包时是生产环境，而 `npm run dev` 的时候是开发环境。

我们使用 webpack 时传递的 mode 参数， `webpack 4.x` 中，在我们的应用代码运行时，直接可以通过 `process.env.NODE_ENV` 这个变量获取的，这样方便我们在运行时判断当前执行的构建环境。

对于 `3.x` 版本应该如何实现。需要用到 `DefinePlugin` 插件，它可以帮助我们在构建时给运行时定义变量，那么我们只要在前面 `webpack 3.x` 版本区分[构建环境的例子的基础上](http://interview.poetries.top/docs/webpack.html#%E4%B8%83%E3%80%81%E5%BC%80%E5%8F%91%E5%92%8C%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83%E7%9A%84%E6%9E%84%E5%BB%BA%E9%85%8D%E7%BD%AE%E5%B7%AE%E5%BC%82)，再使用 `DefinePlugin` 添加环境变量即可影响到运行时的代码。

``` js
module.exports = {
  plugins: [
    new webpack.DefinePlugin({
      // webpack 3.x 的 process.env.NODE_ENV 是通过手动在命令行中指定 NODE_ENV=... 的方式来传递的
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
    }),
  ]
}
```

#### 环境差异

- 生产环境可能需要分离 `CSS`成单独的文件，以便多个页面共享同一个 `CSS` 文件
- 生产环境需要压缩 `HTML/CSS/JS` 代码
- 生产环境需要压缩图片
- 开发环境需要生成 `sourcemap` 文件
- 开发环境需要打印 `debug` 信息
- 开发环境需要 `live reload`或者 `hot reload` 的功能

> `webpack 4.x` 的 `mode` 已经提供了上述差异配置的大部分功能，`mode` 为 `production` 时默认使用 `JS` 代码压缩，而`mode` 为 `development` 时默认启用 `hot reload`，等等。这样让我们的配置更为简洁，我们只需要针对特别使用的 `loader` 和 `plugin` 做区分配置就可以了。

### 区分 npm --save 和 --save-dev的区别

1. 使用场景不同
   * 使用 --save 安装的包是项目发布之后还需要依赖的包 ， 如axios, vue等包，等项目上线以后还需使用。
   * 使用 --save-dev 安装的包则是开发时依赖的包，等项目上线则不会使。如项目中使用的压缩css、js 的模块等在项目上线后则不会使用。

2. npm 安装时信息写入位置不同

   npm install 在安装依赖包的时候 ， 使用 --save-dev 和 --save都会将信息写入package.json 中，但是

   * --save 安装的会将信息写入 dependencies 中
   * --save-dev 安装的会将信息写入 devDependencies 中

### 区分 package.json 和 package.lock.json

#### package.json 版本控制

1. 指定版本：比如 "axios": "3.10.7"或"axios": "=3.10.7"，表示安装3.10.7的版本
2. ~：比如 "axios": "~3.10.7"，表示安装3.10.x的最新版本（不低于3.10.7），但是不安装3.11.x，也就是说安装时不改变大版本号和次要版本号
3. ^：比如 "axios": "^3.10.7"，表示安装3.10.7及以上的版本，但是不安装4.0.0，也就是说安装时不改变大版本号

#### package-lock.json

`package-lock.json`的作用就是：

* 锁住各个模块之间的依赖和版本号，保证多人协作开发时保证每个人的依赖版本是一致的
* 保存各个模块的下载地址

在 npm 5. x 以后，当项目中不存在`package-lock.json`文件时，使用`npm install`时会自动生成这个文件。

当`package-lock.json`存在时，则会安装文件中指定版本的依赖，并且安装速度会比不存在此文件时快很多。因为`package-lock.json`中已经指定依赖的版本，下载地址和整个`node_modules`的文件结构等信息。

#### 更新依赖

在npm 5.x之前，我们可以直接更改`package.json`中的版本号，再`npm install`就可以直接更新了，并且也不会自动生成 lock 文件

自npm 5.0版本发布以来，npm i的规则发生了三次变化。

1. npm 5.0.x 版本，不管package.json怎么变，npm i 时都会根据lock文件下载，所以这种情况，想要更新一个包的话，只能 `npm i xxx@版本号 --save` 手动安装，这样 lock 文件中的版本号才会改变

2. 5.1.0版本后 npm install 会无视lock文件去下载最新的npm 

3. 5.4.2版本后

   * 如果改了package.json，且package.json和lock文件不同，那么执行`npm i`时npm会根据package中的版本号以及语义含义去下载最新的包，并更新至lock

   * 如果两者是同一状态，那么执行`npm i `都会根据lock下载，不会理会package实际包的版本是否有新

## 入口/出口配置

### 入口配置

入口的字段为: `entry`，常规一个入口的配置很简单：

```js
//webpack.config.js
module.exports = {
    entry: './src/index.js' //webpack的默认配置
}
```

`entry` 的值可以是一个字符串，一个数组或是一个对象。

为数组时，表示有“多个主入口”，想要多个依赖文件一起注入时，会这样配置，例如:

```js
entry: [
    './src/polyfills.js',
    './src/index.js'
]
```

### 出口配置

配置 `output` 选项可以控制 `webpack` 如何输出编译文件。

```js
const path = require('path');
module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'), //必须是绝对路径
        filename: 'bundle.js',
        publicPath: '/' //通常是CDN地址
    }
}
```

## 将 ES6 转为 ES5

### 基础配置

- babel-loader
- babel-core
- babel-preset-env
- babel-preset-stage-2

babel 的工作原理就是将 ES6 的代码解析生成`ES6的AST`，然后将 ES6 的 AST 转换为 `ES5 的AST`,最后才将 ES5 的 AST 转化为具体的 ES5 代码。

在 `webpack.config.js`，配置如下:

``` js
//webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: /\.jsx?$/,
                use: ['babel-loader'],
                exclude: /node_modules/ //排除 node_modules 目录
            }
        ]
    }
}
```

### babel 和 webpack 的关系

`Babel` 是编译工具，把高版本语法编译成低版本语法，或者将文件按照自定义规则转换成js语法。

`webpack` 是打包工具，定义入口文件，将所有模块引入整理后，通过loader和plugin处理后，打包输出。

`webpack` 通过 `babel-loader` 使用 `Babel` 。

### 常见匹配规则

* `test: ... ` 字段是匹配规则，针对符合规则的文件进行处理

- `include: ...  匹配特定路径
- `exclude: ... `排除特定路径

### use的写法

`use` 字段有几种写法

- 可以是一个字符串，例如上面的 `use: 'babel-loader'`
- `use` 字段可以是一个数组，例如处理CSS文件是，`use: ['style-loader', 'css-loader']`
- `use` 数组的每一项既可以是字符串也可以是一个对象，当我们需要在`webpack` 的配置文件中对 `loader` 进行配置，就需要将其编写为一个对象，并且在此对象的 `options` 字段中进行配置

### .babelrc文件

这是问了兼容 ES2017、ES2018等等

``` js
{
  'presets': ['@babel/preset-env']
}
```

## 关联html文件

`webpack` 默认是从作为入口的 `.js` 文件进行构建的，但通常一个前端项目都是从一个页面（即 HTML）出发的，最简单的方法是，创建一个 HTML 文件，使用 `script` 标签直接引用构建好的 JS 文件。

但是这样子不方便，因为如果我们的文件名或者路径会变化，例如使用 `[hash]` 来进行命名，那么最好是将 `HTML` 引用路径和我们的构建结果关联起来。

解决方法就是将 html 文件一起打包——使用 `html-webpack-plugin` 插件。

``` js
module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin({
      filename: 'index.html', // 配置输出文件名和路径
      template: 'assets/index.html', // 配置文件模板
    }),
  ],
}
```

此时执行打包，可以看到 `dist` 目录下新增了 `index.html` 文件，并且其中自动插入了 `<script>` 脚本，引入的是我们打包之后的 js 文件。

## CSS

- `css-loader` 负责解析 `CSS` 代码，主要是为了处理 `CSS` 中的依赖，例如 `@import` 和 `url()` 等引用外部文件的声明；
- `style-loader` 负责动态创建 `style` 标签，将 `css` 插入到 `head` 中

``` js
module.exports = {
  module: {
    rules: [
      // ...
      {
        test: /\.css/,
        include: [
          path.resolve(__dirname, 'src'),
        ],
        use: [
          'style-loader',
          'css-loader',
        ],
      },
    ],
  }
}
```

**注意：**

loader的执行顺序是从右向左执行的，也就是后面的loader先执行，上面loader的执行顺序为：

css-loader--->style-loader。

经由上述两个 `loader` 的处理后，CSS 代码会转变为 JS，和 `index.js`一起打包了。如果需要单独把 CSS 文件分离出来，我们需要使用 `extract-text-webpack-plugin` 插件，可参考[这里](http://interview.poetries.top/docs/webpack.html#_2-2-%E6%9E%84%E5%BB%BA-css)

### CSS 预处理

如果用了 sass ，可以安装 sass-loader，然后这样配置

``` js
module.exports = {
  module: {
    rules: [
      // ...
      {
        test: /\.css/,
        include: [
          path.resolve(__dirname, 'src'),
        ],
        use: [
          'style-loader',
          'css-loader',
          'sass-loader'
        ],
      },
    ],
  }
}
```

## 图片/文件处理

在前端项目的样式中总会使用到图片，虽然我们已经提到 `css-loader` 会解析样式中用 `url()` 引用的文件路径，但是图片对应的 `jpg/png/gif` 等文件格式，`webpack` 处理不了。

我们可以使用 `url-loader` 或者 `file-loader` 来处理本地的资源文件。`url-loader` 和 `file-loader` 的功能类似，但是 `url-loader` 可以指定在文件大小小于指定的限制时，返回 `DataURL`，因此，个优先选择使用 `url-loader`。

安装 `url-loader` 的时候，控制台会提示你，还需要安装下 `file-loader`，这个没啥说的，安装就好。

``` js
//webpack.config.js
module.exports = {
    modules: {
        rules: [
           {
              test: /\.(png|jpg|gif)$/,
              use: [
                {
                  loader: 'url-loader',
                  options: {
                    //当使用的图片，小于limit，会将图片编译成base64的字符串形式
                    //大于limit，会使用file-loader加载
                    limit: 8196,
                    name: 'img/[name].[hash:8].[ext]'
                  }
                }
              ]
            }
        ]
    }
}
```

## 优化

### 配置externals

我们可以将一些JS文件存储在 `CDN` 上(减少 `Webpack`打包出来的 `js` 体积)，在 `index.html` 中通过 `<script>` 标签引入，如:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div id="root">root</div>
    <script src="http://libs.baidu.com/jquery/2.0.0/jquery.min.js"></script>
</body>
</html>
复制代码
```

我们希望在使用时，仍然可以通过 `import` 的方式去引用(如 `import $ from 'jquery'`)，并且希望 `webpack` 不会对其进行打包，此时就可以配置 `externals`

```js
//webpack.config.js
module.exports = {
    externals: {
      	// 左侧jquery是我们自己引入时候要用的，右侧是开发依赖库的主人定义的不能修改
        'jquery': '$', 
    }
}
```

其实，我发现，在 vue-cli3 中，不需要再去`import $ from 'jquery'`，直接使用 $ 就可以

### 抽离公共代码

抽离公共代码是对于多页应用来说的，如果多个页面引入了一些公共模块，那么可以把这些公共的模块抽离出来，单独打包。公共代码只需要下载一次就缓存起来了，避免了重复下载。

即使是单页应用，同样可以使用这个配置，例如，打包出来的 bundle.js 体积过大，我们可以将一些依赖打包成动态链接库，然后将剩下的第三方依赖拆出来。这样可以有效减小 bundle.js 的体积大小。

``` js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: "all", // 所有的 chunks 代码公共的部分分离出来成为一个单独的文件
    },
  }
}
```

### 压缩代码

在 webpack 4 以上，production 环境下会自动启用压缩代码，压缩的是 CSS 和 JS 文件，在webpack 4 之前，压缩 JS 代码用的是 `uglifyjs-webpack-plugin` 插件

``` js
module.exports = {
  optimization: {
    minimize: true
  }
};
```

minimize 和 minimizer 的区别：optimization.minimizer的数组列表中主要配置第三方的压缩插件，比如**TerserPlugin**。

**Webpack4 不再支持UglifyJSPlugin，需要改为TerserPlugin**，因为 UglifyJSPlugin 不支持ES6了，所以用TerserPlugin

[webpack4 升级的主要改变](https://www.taijicoder.com/2020/04/09/from-webpack2-to-webpack4/)

### 图片压缩

除了 `url-loader` 外，对于比较大的图片可以使用 `image-webpack-loader`来压缩图片文件

``` js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        use: {
          loader: 'image-webpack-loader',// 压缩图片
          options: {
          	bypassOnDebug: true
      		}
        }
    	}
    ]
  }
}
```

### 开启gzip压缩

可以用 webapck 将文件压缩成 .gz 格式，使用 CompressionWebpackPlugin 插件

``` js
const CompressionWebpackPlugin = require('compression-webpack-plugin');

webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      algorithm: 'gzip',
      test: new RegExp('\\.(js|css)$'),
      // 只处理大于xx字节 的文件，默认：0
      threshold: 10240,
      // 示例：一个1024b大小的文件，压缩后大小为768b，minRatio : 0.75
      minRatio: 0.8 // 默认: 0.8
      // 是否删除源文件，默认: false
      deleteOriginalAssets: false
    })
)
```

#### 双端gzip的区别

同样的，在后端，比如 nginx 中也可以开启 gzip 压缩，它们的区别是什么呢？

* `Webpack`压缩会在构建运行期间一次压缩文件，然后将这些压缩版本保存到磁盘。

* `nginx`在请求时压缩文件时，某些包可能内置了缓存,因此性能损失只发生一次(或不经常)，但通常不同之处在于，这将在响应 HTTP请求时发生。

* 对于实时压缩,让上游代理(例如 Nginx)处理 gzip 和缓存通常更高效，因为它们是专门为此而构建的,并且不会遭受服务端程序运行时的开销(许多都是用C语言编写的) 

* 使用 `Webpack` 压缩的好处是， `Nginx`每次请求服务端都要压缩很久才回返回信息回来，不仅服务器开销会增大很多，请求方也会等的不耐烦。我们在 `Webpack`打包时就直接生成高压缩等级的文件，作为静态资源放在服务器上，这时将 Nginx作为二重保障就会高效很多。

* 具体是在请求时实时压缩，或在构建时去生成压缩文件，就要看项目业务情况。

https://juejin.im/post/6844903825585897485#heading-2

https://zhuanlan.zhihu.com/p/37429159

https://www.cnblogs.com/qiuzhimutou/p/7592875.html

### 多线程打包

webpack打包过程中也是单线程的，特别是在执行loader的时候，长时间编译的任务很多，这样就会导致等待的情况。

**HappyPack**可以将loader的同步执行转换为并行的，这样就能充分利用系统资源来加快打包效率了。

## Vue-cli3的 webpack 配置

vue-cli升级到3.x版本之后，构建出来的项目比较简练，但是没有了webpack的显式配置，如果想要脚手架适用自己的项目，就需要配置`vue.config.js`文件。

官网给出了两个[配置webpack的选项](https://cli.vuejs.org/zh/guide/webpack.html)：

- 简单配置：

  ```
  configureWebpack
  ```

   选项

  - 比较简单，跟以前配置webpack一样
  - 通过 `webapck-merge` 的方式来合并(所以无法修改已经配置的选项)

- 链式操作：

  ```
  chainWebpack
  ```

   选项

  - 需要一定的学习成本，学习 `webapck-chain` 的操作，点击查看[官网](https://github.com/neutrinojs/webpack-chain#getting-started)
  - 直接操作webapck，可以修改已经配置的选项

## Vue 配置多页面

vue-cli3 提供了开箱即用的多页面配置，配置`vue.config.js` 中的 `pages` [选项](https://cli.vuejs.org/zh/config/#pages)即可，因为每增加一个页面就需要增加个选项，不可能自己每次都添加，考虑到借助工具来自动处理这些事情，所以单独配置了 `MultiPage.config.js` 文件，借助 `node` 来自动处理：

```js
// MultiPage.config.js
// 多页面配置
const fs = require('fs');
const path = require('path');
const fileNames = fs.readdirSync(path.resolve(__dirname, './src/pages'));
const MutiPageConfig = {};

fileNames.forEach((pageName) => {
  MutiPageConfig[pageName] = {
    // page 的入口
    entry: `src/pages/${pageName}/index.js`,
    // 模板来源
    template: `src/pages/${pageName}/index.html`,
    // 在 dist 的输出
    filename: `views/${pageName}/index.html`,
    // 当使用 title 选项时，template 中的 title 标签需要是 <title><%= htmlWebpackPlugin.options.title %></title>
    // title: '',
    // 在这个页面中包含的块，默认情况下会包含提取出来的通用 chunk 和 vendor chunk，如果自己有配置 splitChunks 选项，可以在此添加
    chunks: ['chunk-vendors', 'chunk-common', pageName]
  }
});

module.exports = MutiPageConfig;
```

然后在 `vue.config.js` 中引入即可：

```js
// 引入多页面配置文件
const MutiPageConfig = require('./MultiPage.config');
// multiple-pages 多页面模式下构建应用 
pages: MutiPageConfig,
```