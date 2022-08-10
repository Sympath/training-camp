## 要做什么：webpack 产物缓存

学习并实战 webpack 中通过缓存实现性能提升的方案

- 文件指纹：打包后输出的文件名后缀，存在三种类型【Hash、Chunkhash、Contenthash】
- 缓存：webpack 模块化的自带缓存、懒加载

## 怎么做到：文件指纹

### 项目目录结构

既然是实战项目，我们现在先来搭建下框架，先找创建一个空目录`5.webpack性能优化篇之体积优化`，我们需要创建如下结构的测试项目

```js
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

简单介绍下

- src：项目源代码
- webpack.conf.js：本次项目的 webpack 配置

补充：一个个建目录太麻烦，推荐个人开发的 vscode 插件 weiyi-tools，安装好后

1. 复制上面目录结构内容
2. 在侧边栏自己想要的空目录处右键
3. 找到【选中目录工具箱 - 根据 tree 结果生成目录】，点击

效果如下

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h3kldh54m8j217v0u0q56.jpg" alt="2022-06-24 14.40.09" style="zoom:25%;" />

src/index.js

```js
import img from "../images/1.jpg";
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";

class Search extends React.Component {
  render() {
    return (
      <div>
        Search Text 文件指纹
        <img src={img} />
      </div>
    );
  }
}

ReactDOM.render(<Search />, document.getElementById("root"));
```

src/index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

src/index.css

```css
@font-face {
  font-family: "SourceHanSerifSC-Heavy";
  src: url("./SourceHanSerifSC-Heavy.otf") format("truetype");
}
div {
  font-family: "SourceHanSerifSC-Heavy";
  display: flex;
}
img {
  width: 100px;
}
```

这里用到了 react，所以需要 babel 配置对 react 的支持，`.babelrc`的内容如下

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```

至此，我们的实战项目的框架也就搭建好啦！

### 文件指纹种类

- Hash：代表每次 webpack 编译中生成的 hash 值，所有使用这种方式的文件 hash 都相同
- Chunkhash：基于入口文件及其关联的 chunk 形成，某个文件的改动只会影响与它有关联的 chunk 的 hash 值，不会影响其他文件
- Contenthash：根据文件内容创建。当文件内容发生变化时，contenthash 发生变化

### 怎么生成文件指纹

##### 输出 js 的文件指纹

```js
{
  ...
   output: {
    path: path.resolve(__dirname, "./dist"),
    filename: "[name]_[hash:8].js",
  },
}
```

##### 输出图片、字体文件等静态资源的文件指纹

```js
module: {
  rules: [
    {
      test: /.(png|jpg|gif|jpeg)$/,
      use: [
        {
          loader: "file-loader",
          options: {
            name: "[name]_[hash:8].[ext]",
          },
        },
      ],
    },
    {
      test: /.(woff|woff2|eot|ttf|otf)$/,
      use: [
        {
          loader: "file-loader",
          options: {
            limit: 1024000,
            options: {
              name: "[name]_[hash:8][ext]",
            },
          },
        },
      ],
    },
  ];
}
```

##### CSS 文件的文件指纹

安装依赖

```
npm i mini-css-extract-plugin
```

在 webpack 中使用，webpack.config.js 内容如下

- 将 css 文件抽离成单独文件：这里需要用`MiniCssExtractPlugin.loader`替换`style-loader`
- 文件指纹：使用`MiniCssExtractPlugin.loader`插件

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
 ...
  module: {
    rules: [
       ...
       {
        test: /.css$/,
        use: [MiniCssExtractPlugin.loader, "css-loader"],
      },
      {
        test: /.less/,
        use: [MiniCssExtractPlugin.loader, "css-loader", "less-loader"],
      },
    ]
  },
  ...
  plugins: [
    new MiniCssExtractPlugin({
      filename: "[name]_[contenthash:8].css",
    }),
  ],
};

```

![image-20220612123440977](https://tva1.sinaimg.cn/large/e6c9d24ely1h35cr3v3r6j20ea06lmxe.jpg)

## 怎么做到：webpack 模块化的自带缓存

webpack 内部实现模块化依赖`require`函数，此函数默认会实现缓存，加载过的模块不会再次进行加载动作，如果需要删除缓存可以使用如下语句，其中的 key 是指模块名

```
delete require.cache[key]
```

## 怎么做到：懒加载

代码库进行分割生成不同的 chunks，用到对应的 chunks 时才进行加载

- 抽离相同代码到同一个共享块
- 脚本懒加载，入口页加速渲染

### 懒加载 JS 脚本方式

- CommonJS：require.ensure
- ES6：动态 import，目前没有原生支持，需要 babel 转换

### 实现动态 import

实现 text 组件开始不加载，点击按钮后才加载代码并显示

![2022-06-12 18.14.29](https://tva1.sinaimg.cn/large/e6c9d24ely1h4d73phk0dj21q90u0785.jpg)

##### 安装依赖

```
npm install @babel/plugin-syntax-dynamic-import --save-dev
```

##### 在 webpack 中使用

.babelrc 中进行配置

```js
{
    "plugins": [
        "@babel/plugin-syntax-dynamic-import"
    ]
}
```

##### 项目结构

```
├── src
│   └── search
│       ├── index.html
│       ├── index.js
│       └── text.js
```

index.js

```js
import React from "react";
import ReactDOM from "react-dom";

class Search extends React.Component {
  constructor() {
    super(...arguments);
    this.state = {
      Text: null,
    };
  }
  loadComponent() {
    import("./text.js").then((Text) => {
      console.log(Text);
      this.setState({
        Text: Text.default,
      });
    });
  }
  render() {
    let Text = this.state.Text;
    return (
      <div>
        Search Text
        {Text ? <Text /> : null}
        <button onClick={this.loadComponent.bind(this)}>异步加载button</button>
      </div>
    );
  }
}

ReactDOM.render(<Search />, document.getElementById("root"));
```

text.js

```js
import React from "react";

class Search extends React.Component {
  render() {
    return <div>动态import</div>;
  }
}

export default Search;
```

index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Document Title</title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

## 课后作业：用于感兴趣同学的拔高
