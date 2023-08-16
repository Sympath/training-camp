## 要做什么：webpack配置文件拆分及开发环境搭建

之前是打包动作的完成，但我们开发阶段其实是会频繁更新代码并查看对应效果的，这就意味着如果我们要和上面一样，就得不断经历【开发 - 打包 - 看效果】的流程，这显然很低效；

所以现实开发中，我们是存在开发环境和生产环境的，开发环境使用devServer进行热更新实现【改完自动更新】的效果，打包内容存在于缓存中，这也意味着我们会有两套配置文件。

所以本文将完成如下目标

- 配置文件拆分：支持不同环境使用不同的webpack配置
- 开发环境搭建：支持开发环境下的文件更改监听

## 怎么做到：尽可能做到复制粘贴就能有成果

### 配置文件拆分

##### 对应依赖

**备注**：外层已安装，无需安装，只是为了方便后续日常开发中复用

拆分配置文件需要使用此依赖

```
npm i webpack-merge@^5.8.0 -S
```



这里我们需要使用`webpack-merge`这个依赖包，抽离通用配置放在`webpack.base.js`中，然后webpack.dev.js和webpack.pro.js分别合并。所以得出【拆分成三个文件】的结论

- webpack.base.js：通用共用的配置
- webpack.dev.js：开发环境独有的配置
- webpack.pro.js：生产环境独有的配置

这里我们可以新建一个新的空目录`2.webpack配置拆分`，结构内容如下

```
.
|-- src
|   |-- helloworld.js
|   |-- index.html
|   |-- index.js
|-- webpack.base.js
|-- webpack.dev.js
|-- webpack.prod.js
```

src下内容不变，配置文件内容拆分如下

`webpack.base.js`

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
module.exports = {
  context: process.cwd(), // 上下文对象
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
    })
  ],
};
```

`webpack.prod.js`

```js
const { merge } = require("webpack-merge");
const base = require("./webpack.base.js");

module.exports = merge(base, {
  mode: "production",
});
```

`webpack.dev.js`

```js
const { merge } = require("webpack-merge");
const base = require("./webpack.base.js");

module.exports = merge(base, {
  mode: "development",
});
```

### 支持开发环境下的文件更改监听

##### 对应依赖

**备注**：外层已安装，无需安装，只是为了方便后续日常开发中复用

拆分配置文件需要使用此依赖

```
npm i webpack-dev-server@^3.3.1 -S
```

在webpack中对于文件更改的监听，存在三种方式

- watch：将watch属性设置为true，则会监听文件改变自动打包
- 热更新WDS（webpack-dev-server）：HotModuleReplacementPlugin
- 热更新WDM（webpack-dev-middleware）：将webpack输出产物传输给服务器，高度自定义定制需求

这里我们实现日常开发中最常用的热更新，即使用`webpack-dev-server`去启动服务

**热更新原理**：热更新本质是自己开启了express应用(HMR Server)，添加了对webpack编译的监听(webpack Compiler)，添加了和浏览器的websocket长连接(HMR Runtime)，当文件变化触发webpack进行编译并完成后，会通过sokcet消息告诉浏览器准备刷新。而为了减少刷新的代价，就是不用刷新网页，而是刷新某个模块，webpack-dev-server可以支持热更新，通过生成 文件的hash值来比对需要更新的模块，浏览器再进行热替换

- Webpack Compile：将JS编译成Bundle
- HMR Server：将热更新的文件输出给HMR Rumtime
- Bundle Server：提供文件在浏览器的访问
- HMR Rumtime：会被注入到浏览器，更新文件的变化
- bundle.js：构建输出的文件

热更新实现只需在配置中新增`devServer`配置即可，改后的`webpack.dev.js`配置如下

```js
const { merge } = require("webpack-merge");
const base = require("./webpack.base.js");

module.exports = merge(base, {
  mode: "development",
  devServer: {
    hot: true, // 本质上是开启了HotModuleReplacementPlugin，也可以通过--hot启动
    contentBase: "./dist",
  },
  devtool: "source-map", 
});

```

其中的devtool是对应的sourcemap配置项，更多信息可见往期文章

- 【万字长文：关于sourcemap，这篇文章就够了】：https://juejin.cn/post/6969748500938489892

然后当我们运行如下命令，就会发现即改即用的效果实现啦

```
npx webpack-dev-server
```

## 课后作业：用于感兴趣同学的拔高

- 【谈谈你对webpack中热更新的理解】回答梳理