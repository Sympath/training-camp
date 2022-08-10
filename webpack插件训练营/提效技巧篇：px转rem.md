## 前言

### 背景

浏览器分辨率的兼容困难

![image-20220608145438578](https://tva1.sinaimg.cn/large/e6c9d24ely1h30ubihirjj22ej0u0q5x.jpg)

### 实现思路方案

- 媒体查询：@media，缺点是需要写多套适配代码
- rem：动态更改html根字体大小，响应页面
  - flexible：实现动态更改html根字体大小
  - px2rem-loader：实现将px单位转为rem



## 具体实现

##### 安装依赖

```
npm i lib-flexible -S
npm i px2rem-loader -D
```

##### 在webpack中使用

webpack.config.js实现接入px2rem-loader，将px单位转为rem

```js
module.exports = {
 module: {
   rules: [
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
   ]
 }
};
```



##### 内联html实现动态更改HtmlFontSize

在index.html模板中添加如下内容

```html
<script>
${require('raw-loader!babel-loader!../node_modules/lib-flexible').default}
</script>
```

### 

