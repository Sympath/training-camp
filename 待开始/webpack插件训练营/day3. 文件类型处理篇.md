## 要做什么：常见特性处理的方案

本文将产出webpack对各个特性的处理，给出业界最常见方案，本文面向特性如下
- es高级特性降级：babel
- react
- css
- less
- 静态资源（图片、文件）

## 怎么做到：尽可能做到复制粘贴就能有成果

### 项目基础结构

创建`3.webpack文件类型处理`目录，在其下实现如下结构

```
.
|-- images
|   |-- 1.jpg
|-- src
|   |-- SourceHanSerifSC-Heavy.otf
|   |-- index.css
|   |-- index.html
|   |-- index.js
|-- webpack.config.js
```

### 项目内容实现

##### 图片images

随便放一张jpg的图片，命名为1.jpg即可

##### 字体文件

 SourceHanSerifSC-Heavy.otf：git仓库内自取（https://github.com/Sympath/training-camp-project/blob/main/3.webpack%E6%96%87%E4%BB%B6%E7%B1%BB%E5%9E%8B%E5%A4%84%E7%90%86/src/SourceHanSerifSC-Heavy.otf）

##### .babelrc配置文件

需要支持基础配置和react配置

```js
{
    "presets": [
        "@babel/preset-env",
        "@babel/preset-react"
    ]
}
```

##### src/index.css

定义字体

```css
@font-face {
    font-family: 'SourceHanSerifSC-Heavy';
    src: url('./SourceHanSerifSC-Heavy.otf') format('truetype');
}
div {
    font-family: 'SourceHanSerifSC-Heavy';
    display: flex;
}
img {
    width: 100px;
}
```

##### src/index.html

模板文件

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

##### src/index.js

渲染的入口文件

```js
import img from "../images/1.jpg";
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";

class Search extends React.Component {
    render() {
        return (
            <div>
                Search Text change WDR
                <img src={img} />
            </div>
        );
    }
}

ReactDOM.render(<Search />, document.getElementById("root"));

```

### 实现对es高级特性降级：babel

##### 对应依赖

**备注**：外层已安装，无需安装，只是为了方便后续日常开发中复用

拆分配置文件需要使用此依赖

- @babel/core：babel的核心处理模块
- babel-loader：babel和webpack的桥梁
- @babel/preset-env：是Babel6时代babel-preset-latest的增强版。该预设除了包含所有稳定的转码插件，还可以根据我们设定的目标环境进行针对性转码。

```
npm i @babel/core@^7.4.4 @babel/preset-env@^7.4.4 babel-loader@^8.0.5 -D
```

##### 增加ES6对应的babel配置

新增配置文件.babelrc

```js
{
  "preset": [
    "@babel/preset-env"
  ]
}
```

##### 在webpack中使用

webpack.config.js

```js
  module: {
    rules: [
      {
        test: /.js$/,
        use: "babel-loader",
      },
    ],
  },
```

### 配置react

##### 安装依赖

```
npm i @babe/preset-react@^7.0.0 -D
```

##### 增加ES6对应的babel配置

新增配置文件.babelrc

```js
{
    "presets": [
        "@babel/preset-env",
        "@babel/preset-react"
    ]
}
```

##### 在webpack中使用

webpack.config.js

```js
  module: {
    rules: [
      {
        test: /.jsx?$/,
        use: "babel-loader",
      },
    ],
  },
```

### 配置CSS处理

- css-loader：用于加载.css文件，并转换成commonjs对象
- style-loader：将样式通过style标签插入到head中

##### 安装依赖

```
npm i style-loader@^0.23.1 css-loader@^2.1.1
```

##### 增加ES6对应的babel配置

新增配置文件.babelrc

```js
{
    "presets": [
        "@babel/preset-env",
        "@babel/preset-react"
    ]
}
```

##### 在webpack中使用

webpack.config.js

```js
  module: {
    rules: [
      {
        test: /.css$/,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
```

### 配置LESS处理

- css-loader：用于加载.css文件，并转换成commonjs对象
- style-loader：将样式通过style标签插入到head中
- less-loader：用于加载.less文件，并转换成commonjs对象

##### 安装依赖

```
npm i style-loader@^0.23.1 css-loader@^2.1.1 less-loader@^5.0.0
```

##### 增加ES6对应的babel配置

新增配置文件.babelrc

```js
{
    "presets": [
        "@babel/preset-env",
        "@babel/preset-react"
    ]
}
```

##### 在webpack中使用

webpack.config.js

```js
  module: {
    rules: [
      {
        test: /.less$/,
        use: ["style-loader", "css-loader", "less-loader"],
      },
    ],
  },
```



### 静态资源（图片、文件）处理

- file-loader：用于加载静态资源，并转换成commonjs对象
- url-loader：file-loader的封装，支持将体积在指定范围的图片转为base64内嵌

##### 安装依赖

```
npm i file-loader@^3.0.1 url-loader@^1.1.2
```

##### 在webpack中使用

webpack.config.js

```js
  module: {
    rules: [
      // {
      //   test: /.jpg|jpeg|gif|png$/,
      //   use: "file-loader",
      // },
      {
        test: /.(png|jpg|gif|jpeg)$/,
        use: [
          {
            loader: "url-loader",
            options: {
              limit: 1024000,
            },
          },
        ],
      },
      {
        test: /.(woff|woff2|eot|ttf|otf)$/,
        use: "file-loader",
      },
    ],
  },
```

### 最终的webpack配置文件内容

webpack.config.js

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const CleanWebpackPlugin = require("clean-webpack-plugin");

module.exports = {
    context: process.cwd(), // 上下文对象
    mode: "development",
    entry: {
        main: "./src/index.js",
    },
    output: {
        path: path.resolve(__dirname, "./dist"),
        filename: "[name].js",
    },
    module: {
        rules: [
            {
                test: /.js$/,
                use: "babel-loader",
            },
            {
                test: /.css$/,
                use: ["style-loader", "css-loader"],
            },
            {
                test: /.less/,
                use: ["style-loader", "css-loader", "less-loader"],
            },
            // {
            //   test: /.jpg|jpeg|gif|png$/,
            //   use: "file-loader",
            // },
            {
                test: /.(png|jpg|gif|jpeg)$/,
                use: [
                    {
                        loader: "url-loader",
                        options: {
                            limit: 1024000,
                        },
                    },
                ],
            },
            {
                test: /.(woff|woff2|eot|ttf|otf)$/,
                use: "file-loader",
            },
        ],
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: "./src/index.html",
            filename: "index.html",
        }),
        new CleanWebpackPlugin(),
    ],
};

```



## 课后作业：用于感兴趣同学的拔高







