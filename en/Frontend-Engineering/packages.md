## Methods for Publishing npm Packages

1. **Create a project on github, clone it locally**

2. **Add `LICENSE` file, specify the corresponding open source license, add `README`**

3. **`.gitignore` file**

4. **`npm init` project, generate `package.json` and `lock` files**

5. **Set up file directory**

   ``` 
   -------------- src     // Source code directory, for example coffee, typescript, es6+ code directory
   -------------- lib     // Transpiled code directory, for example babel transpiled es5 code directory
   -------------- docs    // Code-related design and usage documentation
   -------------- tests   // Related test directory
   ```

6. **Write code in src directory**

7. **Use babel to transform code**

   - babel configuration file `.babelrc`

   ```json
   {
     "presets":["es2015","stage-0"]
   }
   ```

   - Add npm commands

   ```json
   "scripts": {
     "build": "babel src -d lib",
   }
   ```

8. **Implement an npm package that can be installed globally**

   - Add `package.json` configuration

   ```json
   "bin": {
   "markdown-clear": "./lib/cli.js"
   }
   ```

   - Add to first line of `cli.js` file

   ```
   #!/usr/bin/env node
   ```

9. **Local testing**

   - Use npm to install local file as local package

   ```
   npm install path/to/markdown-clear
   ```

   - Use npm to install local file as global package

   ```
   npm install path/to/markdown-clear -g
   ```

10. **Publish NPM package**

    - If you haven't registered an npm account

    ```
    npm adduser USERNAME
    ```

    - If you haven't logged in

    ```
    npm login
    ```

    - After logging in, publish package, execute in project directory

    ```
    npm publish
    ```

Reference articles: https://imweb.io/topic/5985e4ed35d7d0a321c5eb82, https://juejin.cn/post/6844904030976606216#heading-15

## Methods for Developing Your Own Library

1. **Create a library folder, initialize project**

```
npm init -y
```

Includes a package.json file. Then simply modify it as shown below:

```json
{
  "name": "library",
  "version": "1.0.0",
  "description": "",
  "main": "./dist/library.js",
  "scripts": {
    "build": "webpack"
  },
  "keywords": [],
  "author": "rocky",
  "license": "MIT"
}
```

2. **Create a few simple files**

Create a new src folder in the root directory, create a math.js and string.js. The relevant file contents are as follows:

```js
// math.js
export function add(a,b){
    return a+b;
}

export function minus(a,b){
    return a-b;
}

// string.js
export function join(a,b){
    return a+" "+b;
}
```

Continue to create an index.js

```js
import * as math from "./math";
import * as string from "./string";

export default {math,string}
```

3. **webpack configuration file webpack.config.js, configured as follows:**

```js
const path = require("path");

module.exports={
    mode:"production",
    entry:"./src/index.js",
    output:{
        path:path.resolve(__dirname,"dist"),
        filename:"library.js",
        library:"library",// Add a library variable in global variables
        libraryTarget:"umd"
    }
}
```

After successful installation, execute the build command

```
npm run build
```

Afterwards, a dist folder will be generated in the root directory, containing a library.js.

4. **How to use**

If others want to use this packaged library.js, there might be several ways as follows:

```js
// es6 method
import library from "library"

// commonjs method
const library=require("library")

// AMD method
require(["library"],function(){})

// script tag import
<script src="library.js"></script>
```

Create an index.html in the dist folder, use script to import the previously packaged library.js. Open index.html in browser, enter library in the console, you will get the following result:![result](https://segmentfault.com/img/remote/1460000021318634)
A simple library has been packaged and generated.

Reference article: https://segmentfault.com/a/1190000021318631