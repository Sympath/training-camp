

### 支持能力

-

### manifest.json 中的处理

```json
{
  ...
  "permissions": [
    "cookies",
    "*://*.域名.com", // 如果想管理所有域名下的cookies 可以配置为"<all_urls>"
  ]
}
```

### 对应处理对象

```js
chrome.
```

##### 其他数据结构

```js
// 书签对象 bookmark
{
  id: String，标签id，chrome管理
  title: String，书签标题
  parentId: String?，父节点id
  index: Number?, 书签在父节点中的位置，从0开始
  url: String?，书签对应超链接
  dateAdded: String?，从1970.1.1至创建时间经过的毫秒数
  dateGroupModified: String?，从1970.1.1至修改时间经过的毫秒数
  children: Array?，子书签数组
}
```

##### 存在方法

- 

##### 相关钩子



##### 方法名：方法作用

```js

```

如

```js

```



