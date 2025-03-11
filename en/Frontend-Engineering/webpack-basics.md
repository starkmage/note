## What is webpack

Webpack is essentially a bundling tool that analyzes module dependencies based on code content and helps us bundle code from multiple modules.

Webpack takes multiple code modules (which can be different file types) used in our project and bundles them into a few static files that are required for the project to run.

## Core Concepts

* entry: Entry point
* output: Output
* loader: Module transformer, used to convert module content into new content according to requirements
* plugins: Extension plugins, inject extension logic at specific times in the webpack build process to change build results or do what you want

## Project Initialization

npm init command

* Generates package.json file and package-lock.json file

Local installation of webpack in project

* npm install webpack@version --save-dev
* Set build in scripts field of package.json file

## Several Distinctions

### Two Environments

That is, `process.env.NODE_ENV` has two environments, `production` and `development`. Production environment and development environment. Generally, we use production environment when packaging with `npm run build`, and development environment when using `npm run dev`.

When we use webpack and pass the mode parameter, in `webpack 4.x`, we can directly get this variable through `process.env.NODE_ENV` when our application code is running, which makes it convenient for us to judge the current build environment at runtime.

For `3.x` version, how to implement it? We need to use the `DefinePlugin` plugin, which can help us define variables at runtime during build. Then we just need to use `DefinePlugin` to add environment variables based on the example of distinguishing [build environments in webpack 3.x](http://interview.poetries.top/docs/webpack.html#%E4%B8%83%E3%80%81%E5%BC%80%E5%8F%91%E5%92%8C%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83%E7%9A%84%E6%9E%84%E5%BB%BA%E9%85%8D%E7%BD%AE%E5%B7%AE%E5%BC%82) to affect the runtime code.

```js
module.exports = {
  plugins: [
    new webpack.DefinePlugin({
      // In webpack 3.x, process.env.NODE_ENV is passed by manually specifying NODE_ENV=... in the command line
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
    }),
  ]
}
```

#### Environment Differences

- Production environment may need to separate CSS into separate files so multiple pages can share the same CSS file
- Production environment needs to minify HTML/CSS/JS code
- Production environment needs to compress images
- Development environment needs to generate sourcemap files
- Development environment needs to print debug information
- Development environment needs live reload or hot reload functionality

> `webpack 4.x`'s `mode` has already provided most of these differential configuration functions. When `mode` is `production`, JS code compression is used by default, and when `mode` is `development`, hot reload is enabled by default, etc. This makes our configuration more concise, we only need to make differential configurations for specially used `loaders` and `plugins`.

### Difference between npm --save and --save-dev

1. Different usage scenarios
   * Packages installed with --save are packages that are still needed after project release, such as axios, vue, etc., which are still needed after the project goes online.
   * Packages installed with --save-dev are packages that are dependent during development and will not be used when the project goes online. Such as modules used to compress css and js in the project will not be used after the project goes online.

2. Different locations where npm installation information is written

   When npm install installs dependency packages, both --save-dev and --save will write information to package.json, but

   * --save installation will write information to dependencies
   * --save-dev installation will write information to devDependencies

### Difference between package.json and package.lock.json

#### package.json Version Control

1. Specify version: For example, "axios": "3.10.7" or "axios": "=3.10.7", means install version 3.10.7
2. ~: For example, "axios": "~3.10.7", means install the latest version of 3.10.x (not lower than 3.10.7), but not install 3.11.x, that is, do not change the major version number and minor version number during installation
3. ^: For example, "axios": "^3.10.7", means install version 3.10.7 and above, but not install 4.0.0, that is, do not change the major version number during installation

#### package-lock.json

The purpose of `package-lock.json` is:

* Lock the dependencies and version numbers between modules to ensure that everyone's dependency versions are consistent in multi-person collaborative development
* Save the download address of each module

After npm 5.x, when there is no `package-lock.json` file in the project, using `npm install` will automatically generate this file.

When `package-lock.json` exists, it will install the version of dependencies specified in the file, and the installation speed will be much faster than when this file does not exist. Because `package-lock.json` has already specified the version of dependencies, download address and the entire file structure of `node_modules`, etc.

#### Update Dependencies

Before npm 5.x, we could directly change the version number in `package.json`, then `npm install` would directly update, and would not automatically generate lock files

Since npm 5.0 was released, the rules of npm i have changed three times.

1. In npm 5.0.x version, no matter how package.json changes, npm i will download according to the lock file, so in this case, if you want to update a package, you can only manually install `npm i xxx@version number --save`, so that the version number in the lock file will change

2. After version 5.1.0, npm install will ignore the lock file and download the latest npm

3. After version 5.4.2

   * If package.json is changed and package.json is different from the lock file, then when executing `npm i`, npm will download the latest package according to the version number and semantic meaning in package, and update to lock
   
   * If both are in the same state, then executing `npm i` will download according to lock, and will not care whether the actual package version in package has new ones

## Build Process

Webpack's running process is a serial process, which will execute the following processes in sequence from start to finish:

1. **Initialize parameters**: Read and merge parameters from configuration files and Shell statements to get the final parameters
2. **Start compilation**: Initialize the Compiler object with the parameters obtained in the previous step, load all configured plugins, execute the object's run method to start compilation
3. **Determine entry**: Find all entry files according to entry in the configuration
4. **Compile modules**: Starting from the entry file, call all configured Loaders to translate the module, then find the modules that the module depends on, and recursively execute this step until all files that the entry depends on have been processed by this step
5. **Complete module compilation**: After all modules have been translated by Loaders in step 4, get the final content of each module after translation and their dependencies
6. **Output resources**: According to the dependency relationship between entry and modules, assemble into Chunks containing multiple modules, then convert each Chunk into a separate file and add it to the output list, this step is the last chance to modify the output content
7. **Complete output**: After determining the output content, determine the output path and file name according to the configuration, write the file content to the file system

During the above process, Webpack will broadcast specific events at specific times, and plugins will execute specific logic after listening to interested events, and plugins can call the API provided by Webpack to change Webpack's running results.

## Webpack's Bundling Principle

1. Initialize parameters based on configuration file and shell command

2. Initialize compile object based on parameters and load configured plugins

3. Identify entry files

4. Identify module dependencies layer by layer

5. Perform module compilation

6. Output resources

## Difference between loader and plugin

### Specific meaning of loader

Loader is a file loader that can load resource files and perform some processing on these files, such as compilation, compression, etc., and finally package them together into the specified file

1. Multiple loaders can be used to process one file, the execution order of loaders is opposite to the order in the configuration, that is, the last loader executes first, and the first loader executes last
2. The first executing loader receives the source file content as a parameter, other loaders receive the return value of the previously executed loader as a parameter, and the last executing loader will return the JavaScript source code of this module

### Specific meaning of plugin

Many events will be broadcast during webpack's lifecycle, and plugins can listen to these events and change the output results through the API provided by webpack at the appropriate time.

### Differences

For loader, it is a converter that compiles file A to form file B. It operates on files, such as converting A.scss to A.css, which is a pure file conversion process

Plugin is an extender that enriches webpack itself, targeting the entire webpack packaging process after the loader ends. It does not directly operate on files, but works based on an event mechanism, monitoring certain nodes in the webpack packaging process and performing extensive tasks, such as packaging optimization

## Principle of Loader

https://zhuanlan.zhihu.com/p/104205895

### Principle

Webpack can only directly process javascript format code. Any non-js files must be pre-processed and converted to js code before they can participate in packaging. The role of loader is to give webpack the ability to **load and parse non-javascript**. **It can also handle different format files**, such as converting scss to css, Typescript to js.

Loader (loader) is such a code converter. It is called and executed by webpack's `loader runner`, receives the original resource data as a parameter (when multiple loaders are used together, the result of the previous loader will be passed to the next loader), and finally outputs javascript code (and optional source map) to webpack for further compilation.

**The essence of loader is a node module, this module exports a function, and this function may also have a pitch method.**

### Execution Issues

#### Classification

- pre: Pre loader
- normal: Normal loader
- inline: Inline loader
- post: Post loader

Loader for single file packaging is called inline loader; for loaders in rules, webpack also defines a property enforce, which can take values pre (for pre loader), post (for post loader), if there is no value then it is (normal loader).

#### Execution Priority

- The execution priority of 4 types of loaders is: `pre > normal > inline > post`.
- The execution order of loaders with the same priority is: `from right to left, from bottom to top`.

#### Effect of Prefix

Inline loader can skip other types of loaders by adding different prefixes.

- `!` Skip normal loader.
- `-!` Skip pre and normal loader.
- `!!` Skip pre, normal and post loader.

#### Loader with Pitch

`pitch` is a method on the loader, and its role is to block the loader chain.

```js
// loaders/simple-loader-with-pitch.js
module.exports = function (source) {  
    console.log('normal excution');   
    return source;
}

// loader's pitch method, not required
module.exports.pitch = function() {
    console.log('pitching');
}
```
## How to Customize Webpack Plugins

- JavaScript named function
- Define an apply method on the plugin function's prototype
- Define a hook bound to webpack itself
- Process specific data inside webpack
- Call the callback provided by webpack after completing the function

A typical Webpack plugin code looks like:

```js
// Plugin code
class MyWebpackPlugin {
  constructor(options) {
  }
  
  apply(compiler) {
    // Insert hook function during emit phase
    compiler.hooks.emit.tap('MyWebpackPlugin', (compilation) => {});
  }
}

module.exports = MyWebpackPlugin;
```

Next, this plugin needs to be imported in webpack.config.js.

```js
module.exports = {
  plugins:[
    // Pass plugin instance
    new MyWebpackPlugin({
      param:'paramValue'
    }),
  ]
};
```

**When Webpack starts, it instantiates the plugin object. After initializing the compiler object, it calls the plugin instance's apply method, passing in the compiler object. The plugin instance registers interested hooks in the apply method, and Webpack calls the corresponding hooks during different build phases.**

## Entry/Output Configuration

### Entry Configuration

The entry field is `entry`. A basic entry configuration:

```js
//webpack.config.js
module.exports = {
    entry: './src/index.js' // webpack's default configuration
}
```

`entry` can be a string, array or object.

When using array format, it means "multiple main entries", typically used when needing to inject multiple dependency files together:

```js
entry: [
    './src/polyfills.js',
    './src/index.js'
]
```

### Output Configuration

Configure `output` options to control how webpack outputs compiled files:

```js
const path = require('path');
module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'), // must be absolute path
        filename: 'bundle.js',
        publicPath: '/' // typically CDN address
    }
}
```

## Converting ES6 to ES5

### Basic Configuration

- babel-loader
- babel-core
- babel-preset-env
- babel-preset-stage-2

**The working principle of babel is to parse ES6 code to generate `ES6 AST`, then convert ES6 AST to `ES5 AST`, and finally transform ES5 AST into specific ES5 code.**

In `webpack.config.js`, configure as follows:

```js
//webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: /\.jsx?$/,
                use: ['babel-loader'],
                exclude: /node_modules/ // exclude node_modules directory
            }
        ]
    }
}
```

### Relationship between babel and webpack

`Babel` is a compilation tool that compiles high-version syntax into low-version syntax, or converts files into js syntax according to custom rules.

`webpack` is a bundling tool that defines entry files, organizes all imported modules, processes them through loaders and plugins, and then bundles them for output.

`webpack` uses `Babel` through `babel-loader`.

### Common Matching Rules

* `test: ... ` field is the matching rule, processing files that match the rule

- `include`: ... match specific paths
- `exclude: ... ` exclude specific paths

### Ways to Write use

The `use` field has several writing methods

- Can be a string, like `use: 'babel-loader'` above
- `use` field can be an array, like when processing CSS files, `use: ['style-loader', 'css-loader']`
- Each item in the `use` array can be either a string or an object. When we need to configure `loader` in webpack's configuration file, we need to write it as an object and configure it in the `options` field of this object

### .babelrc File

This is for compatibility with ES2017, ES2018, etc.

```js
{
  'presets': ['@babel/preset-env']
}
```

## Linking HTML Files

`webpack` by default builds from `.js` files as entry points, but typically a frontend project starts from a page (i.e., HTML). The simplest method is to create an HTML file and directly reference the built JS file using a `script` tag.

However, this is not convenient because if our filename or path changes, for example using `[hash]` for naming, it's better to link the `HTML` reference path with our build results.

The solution is to bundle the html file together—using the `html-webpack-plugin` plugin.

```js
module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin({
      filename: 'index.html', // configure output filename and path
      template: 'assets/index.html', // configure file template
    }),
  ],
}
```

When executing the bundle now, you can see that an `index.html` file has been added to the `dist` directory, and a `<script>` tag has been automatically inserted, referencing our bundled js file.

## CSS

- `css-loader` is responsible for parsing `CSS` code, mainly to handle dependencies in `CSS`, such as `@import` and `url()` declarations that reference external files;
- `style-loader` is responsible for dynamically creating `style` tags and inserting `css` into the `head`

```js
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

**Note:**

Loaders execute from right to left, meaning the later loader executes first. The execution order of the above loaders is:

css-loader--->style-loader.

After processing by these two `loaders`, CSS code will be transformed into JS and bundled together with `index.js`. If you need to separate CSS files, we need to use the `extract-text-webpack-plugin` plugin, refer to [here](http://interview.poetries.top/docs/webpack.html#_2-2-%E6%9E%84%E5%BB%BA-css)

### CSS Preprocessing

If using sass, you can install sass-loader and configure like this:

```js
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
## Image/File Processing

In frontend projects' styles, we often use images. Although we mentioned that `css-loader` will parse file paths referenced by `url()` in styles, webpack cannot handle image file formats like `jpg/png/gif`.

We can use `url-loader` or `file-loader` to handle local resource files. The functionality of `url-loader` and `file-loader` is similar, but `url-loader` can return a `DataURL` when the file size is smaller than a specified limit, therefore, `url-loader` is the preferred choice.

When installing `url-loader`, the console will prompt you that you also need to install `file-loader`, just install it as suggested.

```js
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
                    //When the image used is smaller than limit, it will be compiled into base64 string format
                    //When larger than limit, it will use file-loader to load
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

## Hot Update Principle

`HMR` stands for `Hot Module Replacement`. Before this concept, we used `Hot Reloading`, which notifies the browser to refresh the page when code changes, to avoid frequent manual browser refreshes. HMR can be understood as an enhanced version of `Hot Reloading`, but instead of refreshing the entire page, it partially replaces module code and makes it effective, showing the effects of code changes. Therefore, `HMR` both avoids frequent manual page refreshes and reduces waiting time during page refreshes, greatly improving frontend page development efficiency.

**Simply put: in webpack, everything is a module with a unique identifier. When file content changes, it notifies the browser through established socket, and then the webpack scaffold code on the page side will reload this module file.**

1. When one or more files are modified;
2. The file system receives changes and notifies webpack;
3. Webpack recompiles and rebuilds one or more modules, and notifies the HMR server to update;
4. HMR Server uses webSocket to notify HMR runtime that updates are needed, HMR runtime requests updates via HTTP jsonp;
5. HMR runtime replaces modules being updated, if it determines these modules cannot be updated, it triggers a full page refresh.

The hot-module-replacement-plugin package provides hot update capability to webpack-dev-server, they are used together, and writing them as two separate packages is for functional decoupling.

1) webpack-dev-server (WDS) provides bundle server capability, meaning the generated bundle.js file can be accessed via localhost://xxx, and WDS also provides livereload (automatic browser refresh).

2) hot-module-replacement-plugin's role is to provide HMR runtime and inject the runtime into bundle.js code. Once files on disk are modified, the HMR server will send modified js module information to HMR runtime, then HMR runtime updates the page code locally. Therefore, this method can update without refreshing the browser.

Reference article:

[Question 70: Introduce webpack hot update principle, how to update pages without refreshing browser](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/118)

## webpack alias configuration

```js
const path = require('path');
const resolve = dir => path.resolve(__dirname, dir);

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: resolve('dist')
    },
    resolve: {
        // Set alias
        alias: {
            '@': resolve('src')// After this configuration @ can point to src directory
        }
    }
};
```

Reference article: https://www.cnblogs.com/Jimc/p/10282969.html

## Optimization

### Configure externals

**Prevent** certain `import`ed packages from being **bundled** into the bundle, instead getting these *external dependencies* from the runtime environment.

We can store some JS files on `CDN` (reducing the `js` size that `Webpack` bundles), and include them via `<script>` tags in `index.html`, like:

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
```

We hope that when using it, we can still reference it through `import` (like `import $ from 'jquery'`), and hope that `webpack` won't bundle it, in this case we can configure `externals`

```js
//webpack.config.js
module.exports = {
    externals: {
        // Left side jquery is what we use when importing; right side is the global variable exposed in node-modules file defined by the dependency library owner, cannot be modified
        'jquery': '$', 
    }
}
```

Actually, I found that in vue-cli3, you don't need to `import $ from 'jquery'` anymore, you can use $ directly

### Extract Common Code

Extracting common code is for multi-page applications. If multiple pages import some common modules, these common modules can be extracted and bundled separately. Common code only needs to be downloaded once and cached, avoiding repeated downloads.

Even for single-page applications, you can use this configuration. For example, if the bundled bundle.js is too large, we can package some dependencies into dynamic link libraries, and then split out the remaining third-party dependencies. This can effectively reduce the size of bundle.js.

```js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: "all", // Separate the common parts of all chunks code into a separate file
    },
  }
}
```

### Code Compression

In webpack 4 and above, code compression is automatically enabled in production environment, compressing CSS and JS files. Before webpack 4, the `uglifyjs-webpack-plugin` plugin was used to compress JS code

```js
module.exports = {
  optimization: {
    minimize: true
  }
};
```

minimize tells webpack to use [TerserPlugin](https://webpack.docschina.org/plugins/terser-webpack-plugin/) or other plugins defined in [`optimization.minimizer`](https://webpack.docschina.org/configuration/optimization/#optimizationminimizer) to compress bundles.

Difference between minimize and minimizer: optimization.minimizer's array list mainly configures third-party compression plugins, like **TerserPlugin**.

**Webpack4 no longer supports UglifyJSPlugin, need to change to TerserPlugin** because UglifyJSPlugin no longer supports ES6, so use TerserPlugin

```js
optimization: {
  minimizer: [
    new TerserPlugin({
      // Must be set to true if using source-maps in production
      sourceMap: true, 
      terserOptions: {
        compress: {
          drop_console: true,
        },
      },
    }),
  ],
}
```

[Major changes in webpack4 upgrade](https://www.taijicoder.com/2020/04/09/from-webpack2-to-webpack4/)

### Image Compression

Besides `url-loader`, for larger images, you can use `image-webpack-loader` to compress image files

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        use: {
          loader: 'image-webpack-loader',// Compress images
          options: {
            bypassOnDebug: true
          }
        }
      }
    ]
  }
}
```

### Enable gzip compression

You can use webpack to compress files into .gz format using the CompressionWebpackPlugin plugin

```js
const CompressionWebpackPlugin = require('compression-webpack-plugin');

webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      algorithm: 'gzip',
      test: new RegExp('\\.(js|css)$'),
      // Only process files larger than xx bytes, default: 0
      threshold: 10240,
      // Only compress files better than minRatio, default: 0.8
      minRatio: 0.8
    })
);
```
### Multi-threaded Bundling

Webpack bundling process is also single-threaded, especially when executing loaders, there are many long-time compilation tasks, which will lead to waiting situations.

**HappyPack** can convert loader's synchronous execution to parallel, thus making full use of system resources to speed up bundling efficiency.

### Tree Shaking

Tree shaking is a term commonly used to describe the removal of dead-code (unused code) in JavaScript context.

Sometimes, the code or referenced modules contain some unused code blocks, which are also bundled into the final file during bundling, increasing the size. In such cases, we can use tree shaking technology to safely delete unused parts in the file.

How to use:

**Import Method Requirements:**

- Use ES2015 module syntax (i.e., import and export)

- Write imports in a way that supports tree-shaking

  - When writing code that supports tree-shaking, the import method is very important. You should avoid importing an entire library into a single JavaScript object. When you do this, you're telling Webpack that you need the entire library, and Webpack won't shake it. Taking the popular library Lodash as an example, importing the entire library at once is a big mistake, but importing individual modules is much better.

    ```js
    // Import everything (doesn't support tree-shaking)
    import _ from 'lodash';
    // Named import (supports tree-shaking)
    import { debounce } from 'lodash';
    // Direct import of specific module (supports tree-shaking)
    import debounce from 'lodash/lib/debounce';
    ```

**Webpack Configuration:**

* Must be in production mode. Webpack only performs tree-shaking when compressing code, which only happens in production mode.

* Must set the optimization option "usedExports" to true. This means Webpack will identify code it thinks is unused and mark it in the initial bundling step.

- Introduce a minifier that can delete dead code (e.g., TerserPlugin)

```js
const config = {
 mode: 'production',
 optimization: {
  usedExports: true,
  minimizer: [
   new TerserPlugin({...})
  ]
 }
};
```

**Handling Side Effects:**

Just because Webpack can't see code being used doesn't mean it can safely tree-shake it. Some module imports have important effects on the application just by being imported. A good example is global stylesheets or JavaScript files that set global configurations.

Webpack considers such files to have "side effects". Files with side effects should not be tree-shaken, as this would break the entire application. Webpack's designers clearly understood the risk of bundling code without knowing which files have side effects, **so by default they treat all code as having side effects. This protects you from deleting necessary files, but it means Webpack's default behavior actually doesn't perform tree-shaking.**

In the **project's package.json file**, adding a "sideEffects" property solves this problem.

For tree-shaking, each project must set the `sideEffects` property to either `false` or an array of file paths.

```js
// All files have side effects, none can be tree-shaken
{
 "sideEffects": true
}
// No files have side effects, all can be tree-shaken
{
 "sideEffects": false
}
// Only these files have side effects, all other files can be tree-shaken, but these will be preserved
{
 "sideEffects": [
  "./src/file1.js",
  "./src/file2.js"
 ]
}
```

https://www.jianshu.com/p/34b8f4062d2a

## Vue-cli3's webpack Configuration

After vue-cli upgraded to version 3.x, the built project is more concise, but there's no explicit webpack configuration. If you want the scaffold to suit your project, you need to configure the `vue.config.js` file.

The official website provides two [options for configuring webpack](https://cli.vuejs.org/zh/guide/webpack.html):

- Simple configuration: `configureWebpack` option
  - Relatively simple, similar to previous webpack configuration
  - Merges through `webpack-merge` (so cannot modify already configured options)

- Chain operation: `chainWebpack` option
  - Requires some learning cost, learning `webpack-chain` operations, click to view [official website](https://github.com/neutrinojs/webpack-chain#getting-started)
  - Directly operates webpack, can modify already configured options

## Vue Multi-page Configuration

vue-cli3 provides out-of-the-box multi-page configuration, just configure the `pages` [option](https://cli.vuejs.org/zh/config/#pages) in `vue.config.js`. Since you need to add an option for each additional page, it's impossible to add them manually each time. Considering using tools to automatically handle these things, a separate `MultiPage.config.js` file is configured, using `node` to handle automatically:

```js
// MultiPage.config.js
// Multi-page configuration
const fs = require('fs');
const path = require('path');
const fileNames = fs.readdirSync(path.resolve(__dirname, './src/pages'));
const MutiPageConfig = {};

fileNames.forEach((pageName) => {
  MutiPageConfig[pageName] = {
    // page entry
    entry: `src/pages/${pageName}/index.js`,
    // template source
    template: `src/pages/${pageName}/index.html`,
    // output in dist
    filename: `views/${pageName}/index.html`,
    // when using title option, title tag in template needs to be <title><%= htmlWebpackPlugin.options.title %></title>
    // title: '',
    // chunks included in this page, by default will include extracted common chunk and vendor chunk
    chunks: ['chunk-vendors', 'chunk-common', pageName]
  }
});

module.exports = MutiPageConfig;
```

Then just import it in `vue.config.js`:

```js
// Import multi-page configuration file
const MutiPageConfig = require('./MultiPage.config');
// multiple-pages mode for building application
pages: MutiPageConfig,
```

## 3 Methods of Code Splitting in webpack

* Configure multiple entry files in entry
* Extract common code with splitchunks
* Dynamic loading (refer to vue router lazy loading)

https://imweb.io/topic/5b66dd601402769b60847149

https://juejin.cn/post/6844903848977367048

## About sourcemap

Source map is an information file that stores location information. That is, each position in the transformed code corresponds to a position in the pre-transformed code.

With it, when an error occurs, the debugging tool will directly display the original code instead of the transformed code. This undoubtedly brings great convenience to developers.
https://juejin.cn/post/6844903971648372743