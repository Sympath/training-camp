## 要做什么：基于glob实现自动化路由的MPA

在webpack中实现MPA（多页应用），一般都是多entry + HTMLWebpackPlugin，本部分我们将用一个小实战熟悉一下基础实现，然后分享基于glob实现配置化自动引入。

- webpack中实现MPA基础实现
- 基于glob实现配置化自动引入

## 怎么做到：尽可能做到复制粘贴就能有成果

### 项目基础结构

创建`4.webpack性能优化篇之MPA`目录，在其下实现如下结构

```
.
|-- src
|   |-- index
|   |   |-- index.html
|   |   |-- index.js
|   |-- search
|       |-- index.html
|       |-- index.js
|-- .babelrc
|-- webpack.config.js
```

### 项目内容实现

index/index.html 和 search/search.html内容都如下

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

index/index.js

```js
import React from "react";
import ReactDOM from "react-dom";
class Search extends React.Component {
  render() {
    return <div> index </div>;
  }
}
ReactDOM.render(<Search />, document.getElementById("root"));
```

search/index.js

```js
import React from "react";
import ReactDOM from "react-dom";
class Search extends React.Component {
  render() {
    return <div> search </div>;
  }
}
ReactDOM.render(<Search />, document.getElementById("root"));
```

这里用到了react，所以需要babel配置对react的支持，`.babelrc`的内容如下

```
{
    "presets": [
        "@babel/preset-env",
        "@babel/preset-react"
    ]
}
```

执行打包动作，对打包产物启动一个静态服务，最后效果应该为如下情况

![2022-07-19 10.47.19](https://tva1.sinaimg.cn/large/e6c9d24ely1h4c1lx5u15g21gz0s7ne0.gif)

### webpack中实现MPA基础实现

我们知道webpack中有多entry的概念，其实核心步骤就是

1. 配置多个`entry`
2. 配置多个`html-webpack-plugin`

```js
const HtmlWebpackPlugin = require("html-webpack-plugin");
const path = require("path");

module.exports = {
  mode: "development",
  entry: {
    index: path.join(__dirname, "./src/index/index.js"),
    search: path.join(__dirname, "./src/search/index.js"),
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
    ],
  },
  plugins: [new HtmlWebpackPlugin({
    		template: path.join(__dirname, `src/index/index.html`),
        filename: `index.html`,
        chunks: ["vendors", 'index'], // 指定使用哪些chunks
  }),
        new HtmlWebpackPlugin({
    		template: path.join(__dirname, `src/search/index.html`),
        filename: `search.html`,
        chunks: ["vendors", 'search'], // 指定使用哪些chunks
  })],
};
```

可以发现，如果新增一个页面，就得修改配置文件。

### 基于glob实现配置化自动引入

上面的内容，我们可以发现目标文件是有很大共性的，所以接下来我们和上面的一致，但基于glob读取项目目录文件的能力，做到动态路由，新增路由文件时不再需要新增配置了。

- 规定所有【目录下有index.js】文件都是路由文件
- 采用glob读取指定格式的文件，获取对应的文件名
- 根据文件名生成对应的entry和HtmlWebpackPlugin

这时候我们就可以基于【glob】这个包实现自动化

##### 安装依赖

```
npm i glob@^7.1.4 -D
```

##### 在webpack中实现

webpack.config.js

```js
const path = require("path");
const glob = require("glob");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const CleanWebpackPlugin = require("clean-webpack-plugin");
const setMPA = () => {
  const entry = {};
  const htmlWebpackPlugins = [];

  const entryFiles = glob.sync(path.join(__dirname, "./src/*/index.js"));
  Object.keys(entryFiles).map((index) => {
    const entryFile = entryFiles[index];
    const match = entryFile.match(/src\/(.*)\/index\.js/);
    const pageName = match && match[1];
    entry[pageName] = entryFile;
    htmlWebpackPlugins.push(
      new HtmlWebpackPlugin({
        template: path.join(__dirname, `src/${pageName}/index.html`),
        filename: `${pageName}.html`,
        chunks: ["vendors", pageName], // 指定使用哪些chunks
        inject: true, //自动将所有chunks注入到模板文件中来 默认是true
        minify: {
          html5: true,
          collapseWhitespace: true,
          preserveLineBreaks: false,
          minifyCSS: true,
          minifyJS: false,
          removeComments: false,
        },
      })
    );
  });
  return {
    entry,
    htmlWebpackPlugins,
  };
};
const { entry, htmlWebpackPlugins } = setMPA();
module.exports = {
  context: process.cwd(), // 上下文对象
  mode: "development",
  entry,
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
    ],
  },
  plugins: [new CleanWebpackPlugin()].concat(htmlWebpackPlugins),
};

```

至此，我们也就完成了【基于glob实现配置化自动引入】啦！

## 课后作业：用于感兴趣同学的拔高