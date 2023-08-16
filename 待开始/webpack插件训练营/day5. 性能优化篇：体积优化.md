## 要做什么：webpack产物体积优化

- 文件压缩：包括【JS文件压缩、css文件压缩、html文件压缩、图片压缩】
- treeshaking：无用代码删除被称为treeshaking，包含js和css的处理
- scope hoisting：将所有模块的代码按照引用顺序放在一个函数作用域里，然后适量的重命名一些变量以避免变量冲突。
- 资源分离：抽离基础库，通过cdn引入，不打包到bundle中

## 怎么做到：文件压缩

### 项目目录结构

既然是实战项目，我们现在先来搭建下框架，先找创建一个空目录`5.webpack性能优化篇之体积优化`，我们需要创建如下结构的测试项目

```js
.
|-- images
|   |-- 1.jpg
|-- readme.md
|-- src
|   |-- index.css
|   |-- index.html
|   |-- index.js
|-- .babelrc
|-- webpack.config.js
```

简单介绍下

- src：项目源代码
- webpack.conf.js：本次项目的webpack配置

补充：一个个建目录太麻烦，推荐个人开发的vscode插件weiyi-tools，安装好后

1. 复制上面目录结构内容
2. 在侧边栏自己想要的空目录处右键
3. 找到【选中目录工具箱 - 根据tree结果生成目录】，点击

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
        Search Text change WDR
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
  <body><div id="root"></div></body>
</html>
```

src/index.css

```css
img {
    width: 100px;
}
```

这里用到了react，所以需要babel配置对react的支持，`.babelrc`的内容如下

```json
{
    "presets": [
        "@babel/preset-env",
        "@babel/preset-react"
    ]
}
```

至此，我们的实战项目的框架也就搭建好啦！

### 实现：JS文件压缩

webpack4中默认内置了uglifyjs-webpack-plugin进行js的压缩，但不支持es6语法，可以使用terser-webpack-plugin代替处理

##### 使用

```markdown
npm install terser-webpack-plugin@^1.3.0 --save-dev
```

##### webpack配置

```js
const TerserWebpackPlugin = require("s");

module.exports = {
 ...
  plugins: [
   // 上线压缩 去除console等信息
    new TerserWebpackPlugin({
      parallel: true,
      extractComments: true,
      terserOptions: {
        ecma: undefined,
        warnings: false,
        parse: {},
        compress: {
          drop_console: true,
          drop_debugger: false,
          pure_funcs: ["console.log"], // 移除console
        },
      },
    }),
  ],
};
```

##### 注意：

- 若正确配置之后，仍然不能正确去除console，请检查是否添加了下面配置项，导致代码被转成了字符串，注释无法剔除

```
devtool: 'cheap-module-eval-source-map'
```

- webpack中用到terser-webpack-plugin压缩插件需要对应版本
  1.  webpack4对应 terser-webpack-plugin@4.x版本（推荐4.2.3）
  2.  webpack5对应的使用terser-webpack-plugin@5.x版本

### 实现：css文件压缩

##### 安装依赖

```
npm i optimize-css-assets-webpack-plugin@^5.0.1 cssnano@^4.1.10 -D
```

##### 在webpack中使用

webpack.config.js

```js
const OptimizeCssAssetsPlugin = require("optimize-css-assets-webpack-plugin");

module.exports = {
 ...
  plugins: [
    new OptimizeCssAssetsPlugin({
      assetNameRegExp: /.css$/g,
      cssProcessor: require('cssnano')
    })
  ],
};
```

### 实现：HTML文件实现压缩

##### 安装依赖

```
npm i html-webpack-plugin@^3.2.0 -D
```

##### 在webpack中使用

webpack.config.js

```js
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
 ...
  plugins: [
     new HtmlWebpackPlugin({
      template: "./src/index.html",
      filename: "index.html",
      // chunks: [] 指定使用哪些chunks
      // inject: true 自动将所有chunks注入到模板文件中来 默认是true
      minify: {
        html5: true,
        collapseWhitespace: true,
        preserveLineBreaks: false,
        minifyCSS: true,
        minifyJS: false,
        removeComments: false,
      },
    }),
    ./
  ],
};
```

### 实现：图片压缩

##### 安装依赖

```
npm i image-webpack-loader@^5.0.0 -D
```

##### 在webpack中使用

webpack.config.js中进行配置

```js
{
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
          {
            loader: 'image-webpack-loader',
            options: {
              mozjpeg: {
                progressive: true,
              },
              // optipng.enabled: false will disable optipng
              optipng: {
                enabled: false,
              },
              pngquant: {
                quality: [0.65, 0.90],
                speed: 4
              },
              gifsicle: {
                interlaced: false,
              },
              // the webp option will enable WEBP
              webp: {
                quality: 75
              }
            }
          },
        ],
      },
    ]
  }
}
```

使用前

![image-20220625165541786](https://tva1.sinaimg.cn/large/e6c9d24ely1h3klcozocbj20we0ccjt1.jpg)

使用后

![image-20220625165511423](https://tva1.sinaimg.cn/large/e6c9d24ely1h3klc5rqsej20xk0tun26.jpg)

### 最终在webpack中实现

webpack.config.js

```js
const path = require("path");
const webpack = require("webpack");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const CleanWebpackPlugin = require("clean-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const OptimizeCssAssetsPlugin = require("optimize-css-assets-webpack-plugin");
const TerserWebpackPlugin = require("terser-webpack-plugin");

module.exports = {
  mode: "development",
  entry: {
    main: "./src/index.js",
  },
  output: {
    path: path.resolve(__dirname, "./dist"),
    filename: "[name]_[hash:8].js",
  },
  module: {
    rules: [
      {
        test: /.js$/,
        use: "babel-loader",
      },
      {
        test: /.css$/,
        use: [MiniCssExtractPlugin.loader, "css-loader"],
      },
      {
        test: /.less/,
        use: [MiniCssExtractPlugin.loader, "css-loader", "less-loader"],
      },
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

  plugins: [
    new HtmlWebpackPlugin({
      template: "./src/index.html",
      filename: "index.html",
      minify: {
        html5: true,
        collapseWhitespace: true,
        preserveLineBreaks: false,
        minifyCSS: true,
        minifyJS: false,
        removeComments: false,
      },
    }),
    new CleanWebpackPlugin(),
    new webpack.HotModuleReplacementPlugin(),
    new MiniCssExtractPlugin({
      filename: "[name]_[contenthash:8].css",
    }),
    new OptimizeCssAssetsPlugin({
      assetNameRegExp: /.css$/g,
      cssProcessor: require("cssnano"),
    }),
    // 上线压缩 去除console等信息
    new TerserWebpackPlugin({
      parallel: true,
      extractComments: true,
      terserOptions: {
        ecma: undefined,
        warnings: false,
        parse: {},
        compress: {
          drop_console: true,
          drop_debugger: false,
          pure_funcs: ["console.log"], // 移除console
        },
      },
    }),
  ],
  devServer: {
    hot: true,
    contentBase: "./dist/",
  },
};
```

## 怎么做到：treeshaking

1个模块可能有多个方法，只要其中一个方法使用到了，则整个文件都会被打包到bundle中；而使用tree shaking会在uglify阶段对没使用到的方法进行擦除。

即无用代码删除被称为treeshaking，分为两部分【js和css】

### 实现原理

实现原理在于`DCE（Elimination）`，即

- 代码不可达
- 代码执行结果不可达
- 代码只写不读

### JS的treeshaking

在webpack中的treeshaking，在webpack4+，对生产环境是默认执行的，将mode改为none就可以做到不执行treeshaking。

##### treeshaking特点

- webpack默认支持，在.babelrc中设置modules: false即可，production模式下默认开启
- 必须是ES6预发，CJS不支持，原因在于es6是静态导入，可以支持uglify阶段分析代码引用情况，而commonjs则是动态运行时。

##### ES6模块特点

- 只能作为模块顶层的语句出现
- import模块名只能是字符串常量
- import binding是immutable的

### css的treeshaking

- PurifyCss：遍历代码，识别已经用到的css class(对应github：)
- uncss：需要通过jsdom加载，所有样式通过PostCss解析，通过document.querySelector来识别在html文件里不存在的选择器

##### 安装依赖

```
npm i purgecss-webpack-plugin@^1.5.0 -D
```

##### 在webpack中使用

webpack.config.js中进行配置

```js
const PurgecssWebpackPlugin = require("purgecss-webpack-plugin");

{
  plugins: [
    new PurgecssWebpackPlugin({
      paths: glob.sync(`${path.join(__dirname, 'src')}/**/*`, { nodir: true })
    })
  ]
}
```

### 使用效果

打包出来的dist中会存在`search.css`文件，里面只会有`.used-css`，而`.unused-css`则被treeshaking移除了

## 怎么做到：scope hoisting

原理：将所有模块的代码按照引用顺序放在一个函数作用域里，然后适量的重命名一些变量以避免变量冲突。

优点：通过 scope hoisting 可以减少函数声明代码和内存开销

### 应用

webpack4+中是默认开启的（必须是ES6语法，CJS不支持），webpack3中需要做如下实现

```js
{
  plugins: [
    new webpack.optimize.ModuleConcatenationPlugin() 
  ]
}
```



## 怎么做到：资源分离

基础库分离：通过cdn引入，不打包到bundle中（如react、react-dom）；实现思路如下

- 将公共包不打包进bundle.js中
- 公共包打包成一个vendors.js并上传到cdn上（如果已经有cdn了，可以省略此步骤）
- 在html中引入cdn

我们以【将react、react-dom】为例，实现，在打包时将这两个包采用cdn引入，或抽离到vendor.js中去引入（之后上传cdn，这步不做演示）

### html-webpack-externals-plugin

采用cdn引入

##### 安装依赖

```
npm i html-webpack-externals-plugin -S
```

##### 在webpack中使用

webpack.prod.js

```js
const HtmlWebpackExternalsPlugin = require("html-webpack-externals-plugin");
{
  plugins: [
    new HtmlWebpackExternalsPlugin({
      externals: [
        {
          module: "react",
          entry: "https://11.url.cn/now/lib/16.2.0/react.min.js",
          global: "React",
        },
        {
          module: "react-dom",
          entry: "https://11.url.cn/now/lib/16.2.0/react-dom.min.js",
          global: "ReactDOM",
        },
      ],
    }),
  ]
}
```

使用前

![image-20220612140039068](https://tva1.sinaimg.cn/large/e6c9d24ely1h35f8jy846j20lq059jrv.jpg)

使用后

![image-20220612140054091](https://tva1.sinaimg.cn/large/e6c9d24ely1h35f8t3ayoj20ld079q3v.jpg)

### SplitChunks抽离到vendor.js中

这个优化内置在webpack中，不需要安装依赖

##### 基础介绍

##### chunks

- async：异步引入的库进行分离
- initial：同步引入的库进行分离
- all：所有引入的库进行分离

```js
{
  optimization: {
    splitChunks: {
      chunks: "async",
      minSize: 30000, // 抽离公共包最小的体积大小，单位kb
      maxSize: 0,// 抽离公共包最大的体积大小，单位kb
      minChunks: 1, // chunks最小被使用次数
      maxAsyncRequests: 5, // 浏览器同时请求的数量
      maxInitalRequests: 3,
      automaticNameDelimiter: "~",
      name: true,
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
        },
      },
    },
  },
}
```

如果想抽离react、react-dom为vendor的chunk包，可以做如下配置

```js
{
  optimization: {
    splitChunks: {
      minSize: 0, // 抽离公共包最小的体积大小，单位kb
      cacheGroups: {
        common: {
          name: 'vendors',
          chunks: 'all',
          test: /[\\/]react|react-dom[\\/]/,
        },
      },
    },
  },
}
```

##### 在webpack中使用

webpack.prod.js

- 先配置对应的优化配置项`optimization - splitChunks`
- 再在模板中引入，这点需要在`html-webpack-plugin` 的配置中实现

```js
{
  optimization: {
    splitChunks: {
      cacheGroups: {
        commons: {
          test: /(react|react-dom)/,
          name: "vendors",
          chunks: "all",
        },
      },
    },
  },
  plugins: [
     new HtmlWebpackPlugin({
        // ...
        chunks: ["vendors", pageName], // 指定使用哪些chunks
      })
  ] 
  
}
```

### 最终在webpack中实现

webpack.config.js

```js
const path = require("path");
const glob = require("glob");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const CleanWebpackPlugin = require("clean-webpack-plugin");
// const HtmlWebpackExternalsPlugin = require("html-webpack-externals-plugin");
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
  optimization: {
    splitChunks: {
      cacheGroups: {
        commons: {
          test: /(react|react-dom)/,
          name: "vendors",
          chunks: "all",
        },
      },
    },
  },
  module: {
    rules: [
      {
        test: /.js$/,
        use: "babel-loader",
      },
    ],
  },
  plugins: [
    new CleanWebpackPlugin(),
    // new HtmlWebpackExternalsPlugin({
    //   externals: [
    //     {
    //       module: "react",
    //       entry: "https://unpkg.com/react@16/umd/react.production.min.js",
    //       global: "React",
    //     },
    //     {
    //       module: "react-dom",
    //       entry:
    //         "https://unpkg.com/react-dom@16/umd/react-dom.production.min.js",
    //       global: "ReactDOM",
    //     },
    //   ],
    // }),
  ].concat(htmlWebpackPlugins),
};

```



## 课后作业：用于感兴趣同学的拔高