## 要做什么：webpack打包的基础使用

es6模块化应用，生成可直接在浏览器环境使用的js内容

### 项目搭建

我们先实现下基础的项目环境，这样就能避免因为依赖装错，版本不对等问题耽误各位同学时间了，我们需要创建如下结构的测试项目

```
.
|-- package.json
```

补充：一个个建目录太麻烦，推荐个人开发的vscode插件weiyi-tools，安装好后

1. 复制上面目录结构内容
2. 在侧边栏自己想要的空目录处右键
3. 找到【选中目录工具箱 - 根据tree结果生成目录】，点击

效果如下

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h45fa7fyjkj217v0u0q56.jpg" alt="2022-06-24 14.40.09" style="zoom:25%;" />

考虑到版本等原因，本处直接贴上项目的package.json，这样子目录就可以复用最外层的依赖了，避免每个实战都需要安装依赖

```json
{
    "name": "webpack-study",
    "version": "1.0.0",
    "description": "- entry\r - chunk",
    "main": "webpack.config.js",
    "devDependencies": {
        "@babel/core": "^7.4.4",
        "@babel/plugin-syntax-dynamic-import": "^7.2.0",
        "@babel/preset-env": "^7.4.4",
        "@babel/preset-react": "^7.0.0",
        "autoprefixer": "^9.5.1",
        "babel-eslint": "^10.0.1",
        "babel-loader": "^8.0.5",
        "clean-webpack-plugin": "^2.0.2",
        "css-loader": "^2.1.1",
        "cssnano": "^4.1.10",
        "eslint": "^5.16.0",
        "eslint-config-airbnb": "^17.1.0",
        "eslint-config-airbnb-base": "^13.1.0",
        "eslint-loader": "^2.1.2",
        "eslint-plugin-import": "^2.17.3",
        "eslint-plugin-jsx-a11y": "^6.2.1",
        "eslint-plugin-react": "^7.13.0",
        "express": "^4.17.1",
        "file-loader": "^3.0.1",
        "friendly-errors-webpack-plugin": "^1.7.0",
        "glob": "^7.1.4",
        "html-inline-css-webpack-plugin": "^1.2.1",
        "html-loader": "^0.5.5",
        "html-webpack-externals-plugin": "^3.8.0",
        "html-webpack-inline-source-plugin": "0.0.10",
        "html-webpack-plugin": "^3.2.0",
        "less": "^3.9.0",
        "less-loader": "^5.0.0",
        "mini-css-extract-plugin": "^0.6.0",
        "node-notifier": "^5.4.0",
        "optimize-css-assets-webpack-plugin": "^5.0.1",
        "postcss-loader": "^3.0.0",
        "postcss-preset-env": "^6.6.0",
        "px2rem-loader": "^0.1.9",
        "raw-loader": "^0.5.1",
        "react": "^16.8.6",
        "react-dom": "^16.8.6",
        "rimraf": "^2.6.3",
        "style-loader": "^0.23.1",
        "terser-webpack-plugin": "^1.3.0",
        "uglifyjs-webpack-plugin": "^2.1.2",
        "url-loader": "^1.1.2",
        "webpack": "^4.31.0",
        "webpack-cli": "^3.3.2",
        "webpack-dev-server": "^3.3.1",
        "cache-loader": "^4.0.1",
        "happypack": "^5.0.1",
        "hard-source-webpack-plugin": "^0.13.1",
        "image-webpack-loader": "^5.0.0",
        "purgecss-webpack-plugin": "^1.5.0",
        "speed-measure-webpack-plugin": "^1.3.1",
        "thread-loader": "^2.1.2",
        "webpack-bundle-analyzer": "^3.3.2"
    },
    "dependencies": {
        "babel-polyfill": "^6.26.0",
        "jszip": "^3.10.0",
        "large-number": "^1.0.1",
        "lib-flexible": "^0.3.2",
        "webpack-merge": "^5.8.0",
        "wzyan-large-number": "^1.0.2"
    },
    "scripts": {
        "dev": "webpack-dev-server --open --config webpack.dev.js",
        "build": "webpack --config webpack.prod.js",
        "watch": "webpack --watch",
        "build:ssr": "webpack --config 15.webpack实现react-ssr打包/webpack.ssr.js && nodemon 15.webpack实现react-ssr打包/server/index.js",
        "build:stats": "webpack --config 21.webpack性能分析/webpack.config.js --json > stats.json",
        "build:speed-stats": "webpack --config 21.webpack性能分析/webpack.speed.js",
        "build:bundle-stats": "webpack --config 21.webpack性能分析/webpack.bundle.js"
    },
    "keywords": [],
    "author": "",
    "license": "ISC"
}
```

然后在最外层安装依赖即可

```
npm i
```

**注意**：这种直接给出package.json的行为是为避免更新未做到向上兼容情况，从而锁定版本下载；日常开发中很难满足，所以在后续文章中会告知实现特性对应所需要的依赖，方便后续日常开发中复用，无需安装



## 怎么做到：尽可能做到复制粘贴就能有成果

### 对应依赖

备注：外层已安装，无需安装，只是为了方便后续日常开发中复用

4版本后，webpack将cli分离了，所以需要两个都按照

```
npm i webpack@4.44.1 webpack-cli@3.3.12 -S
```

查看版本确定是否安装成功

```
./node_module/.bin/webpack -v
```

为了方便看效果和避免每次都要手动清理打包产物，我们用两个插件：html-webpack-plugin 和 clean-webpack-plugin；先安装下依赖

```js
npm i html-webpack-plugin@4.4.1 clean-webpack-plugin@3.0.0
```



### 项目基础结构

创建`1.webpack初识`目录，在其下实现如下结构

```
.
|-- src
|   |-- helloworld.js
|   |-- index.html
|   |-- index.js
|-- webpack.config.js
```

### webpack打包的基础使用需求

1. 将index.js设为入口文件
2. 在helloworld.js中实现一个简单的【打印hello world】的功能，然后在index.js目录内导入
3. 打包后js文件使用html模板文件引入
3. 启动静态资源服务器，在浏览器中访问

### 实现一：将index.js设为入口文件

在`webpack.config.js`中存在entry字段，用于指定入口文件

`webpack.config.js`

```js
entry: {
    main: "./src/index.js",
  },
```

### 实现二：业务实现

在helloworld.js中实现一个简单的【打印hello world】的功能，然后在index.js目录内导入，使用es6模块化语法即可

`src/index.js`

```js
import { helloworld } from "./helloworld";

document.write(helloworld());
```

`src/helloworld.js`

```js
export function helloworld() {
    return 'hellow webpack'
}
```

### 实现三：打包后js文件使用html模板文件引入

`src/index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body></body>
</html>
```

这里可以手动引入，在上面的html模板中写script标签，但更常见的写法是使用webpack的插件`html-webpack-plugin`;

在`webpack.config.js`中存在plugins字段，用于指定要用到的插件（插件其实就是一个存在apply方法的类）

```js
const HtmlWebpackPlugin = require("html-webpack-plugin");
module.exports = {
  ....
  plugins: [
    new HtmlWebpackPlugin({
      template: "./src/index.html",
      filename: "index.html",
    })
  ],
};

```

##### 最终的webpack配置文件

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
  context: process.cwd(), // 上下文对象
  mode: "production",
  entry: {
    main: "./src/index.js",
  },

  output: {
    path: path.resolve(__dirname, "./dist"),
    filename: "[name].js",
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: "./src/index.html",
      filename: "index.html",
    }),
    new CleanWebpackPlugin()
  ],
};
```

这时，我们切换进`1.webpack初识`目录，然后运行打包命令

```
cd 1.webpack初识 && npx webpack
```

就会存在打包产物dist目录了。

### 实现四：启动静态资源服务器，在浏览器中访问

这个点建议使用全局命令包`live-server`进行启动，如果之前有安装【weiyi-tools】vscode插件，则更为简单，在dist目录右键即可

![2022-06-08 10.50.01](https://tva1.sinaimg.cn/large/e6c9d24ely1h467uvnnzjg21et0lftlj.gif)

（面向问题：我们使用 live-server 启动一个静态服务，但每次都要经历【开终端 - cd 到指定目录 - 执行 live-server】，终端还不能干掉）

至此，我们就完成了一个webpack打包最基础的使用啦！

## 课后作业：用于感兴趣同学的拔高
