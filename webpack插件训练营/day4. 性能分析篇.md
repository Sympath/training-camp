## 前言

webpack是一款打包工具，功能集成各款插件，插件性能有好有坏，这也意味着一个webpack的配置有好有坏，性能分析就显得尤为重要了。

- 基础分析：输出收集stats.json
- 速度分析：speed-measure-webpack-plugin
- 体积分析：webpack-bundle-analyzer

我们来依次学习下如何实践。

## 实战项目搭建

纸上得来终觉浅，我们还是走实战路线，一步步实现我们想要的效果。但不同于直接发仓库链接，本文的实战项目要求被分享的同学能够知道项目为什么这么实现，文件内容目标是什么。

### 项目目录结构

既然是实战项目，我们现在先来搭建下框架，先找一个空目录，我们需要创建如下结构的测试项目

```js
.
|-- src
|   |-- index
|   |   |-- index.html
|   |   |-- index.js
|-- webpack.conf.js
|-- package.json
```

简单介绍下

- src：项目源代码
- webpack.conf.js：本次项目的webpack配置

补充：一个个建目录太麻烦，推荐个人开发的vscode插件weiyi-tools，安装好后

1. 复制上面目录结构内容
2. 在侧边栏自己想要的空目录处右键
3. 找到【选中目录工具箱 - 根据tree结果生成目录】，点击

效果如下

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h3jbv2job8g217v0u0q56.gif" alt="2022-06-24 14.40.09" style="zoom:25%;" />

### 项目内容实现

src/index.js

```js
import React from 'react';
import ReactDOM from 'react-dom';

class Index extends React.Component {
  render() {
    return <div>HELLO </div>;
  }
}

ReactDOM.render(<Index />, document.getElementById('root'));
```

src/index.html

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

webpack.conf.js

```js
const path = require("path");
const glob = require("glob");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
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
  mode: "production",
  entry,
  output: {
    path: path.resolve(__dirname, "./dist"),
    filename: "[name]_[chunkhash:8].js",
  },
  module: {
    rules: [
      {
        test: /.js$/,
        use: "babel-loader",
      },
      {
        test: /.css$/,
        use: [
          MiniCssExtractPlugin.loader,
          "css-loader",
          {
            loader: "px2rem-loader",
            options: {
              remUnit: 75, // rem相对px转换的单位 1rem = 75px
              remPrecision: 8, // 小数点后位数
            },
          },
          {
            loader: "postcss-loader",
            options: {
              plugins: () => [
                require("autoprefixer")({
                  overrideBrowserslist: ["last 2 version", ">1%", "ios 7"],
                }),
              ],
            },
          },
        ],
      },
      {
        test: /.less/,
        use: [
          MiniCssExtractPlugin.loader,
          "css-loader",
          {
            loader: "px2rem-loader",
            options: {
              remUnit: 75, // rem相对px转换的单位 1rem = 75px
              remPrecision: 8, // 小数点后位数
            },
          },
          {
            loader: "postcss-loader",
            options: {
              plugins: () => [
                require("autoprefixer")({
                  overrideBrowserslist: ["last 2 version", ">1%", "ios 7"],
                }),
              ],
            },
          },
          "less-loader",
        ],
      },
      // {
      //   test: /.jpg|jpeg|gif|png$/,
      //   use: "file-loader",
      // },
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
    ],
  },
  stats: "errors-only",
  plugins: [
    ...htmlWebpackPlugins,
    new CleanWebpackPlugin(),
    new MiniCssExtractPlugin({
      filename: "[name]_[contenthash:8].css",
    })
  ],
};

```

至此，我们的实战项目的框架也就搭建好啦！

## 基础分析：输出收集stats.json

**期待目标：**将webpack的打包结果输出的一个文件`stats.json`中，以供分析

**实现思路：**这是webpack自身提供的能力，我们只需要添加一个打包命令就好。

找到package.json文件，在script中添加

```json
“script”: {
   "build:stats": "webpack --json > stats.json"
}
```

**实现效果**

执行`npm run build:stats`，会发现控制台输出的结果变少了很多，并且会在项目根目录创建一个`stats.json`，里面就是打包输出。



## 速度分析：speed-measure-webpack-plugin

**期待目标：**输出更详细的webpack处理信息 -- 将每个插件的处理时间输出

**实现思路：** 借助webpack插件speed-measure-webpack-plugin

##### 安装依赖

```
npm i speed-measure-webpack-plugin
```

##### 在webpack中使用

webpack.conf.js中进行配置

```js
const SpeedMeasureWebpackPlugin = require("speed-measure-webpack-plugin");
const smp = new SpeedMeasureWebpackPlugin();

module.export = smp.wrap({
  ...
})
```

##### 配置命令

找到package.json文件，在script中添加

```json
“script”: {
   "build:speed-stats": "webpack"
}
```

**实现效果**

执行`npm run build:speed-stats`，会发现控制台输出的结果如下

![image-20220624221617945](https://tva1.sinaimg.cn/large/e6c9d24ely1h4d7a5npdhj20yw0p0go9.jpg)

## 体积分析：webpack-bundle-analyzer

**期待目标：**输出更详细的webpack处理信息 -- 将每个插件的所占体积以图表的形式输出

**实现思路：** 借助webpack插件

##### 安装依赖

```
npm i webpack-bundle-analyzer
```

##### 在webpack中使用

webpack.conf.js中进行配置

```js
const BundleAnalyzerPlugin = require("webpack-bundle-analyzer").BundleAnalyzerPlugin;

module.export = {
  plugins: [
    new BundleAnalyzerPlugin()
  ]
}
```

##### 配置命令

找到package.json文件，在script中添加

```json
“script”: {
   "build:bundle-stats": "webpack"
}
```

**实现效果**

执行`npm run build:bundle-stats`，会发现控制台输出的结果如下（浏览器会自动唤起8888窗口）

![image-20220625110406465](https://tva1.sinaimg.cn/large/e6c9d24ely1h4d7a6j28hj21gz0u0djn.jpg)

