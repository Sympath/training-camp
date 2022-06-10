## 要做什么：利用vscode配置文件 + prettier + eslint实现保存自动格式化

![2022-05-30 15.48.11](https://tva1.sinaimg.cn/large/e6c9d24ely1h2qhawa71eg20se09pmxq.gif)

## 怎么做到：尽可能做到复制粘贴就能有成果

### 在现有项目中使用

利用vscode配置文件 + prettier + eslint实现保存自动格式化：https://juejin.cn/post/7103462793500131336/



### 通过插件实现命令写入

根据文章，我们很容易就能做到实现保存自动格式化，但还是要做【新建配置文件 - 复制粘贴】的动作，既然研制插件了，我们就尝试用一个命令实现它吧！

同样是三步走

- 配置命令
- 实现注册
- 实现逻辑

##### package.json中配置命令

```js
		"commands": [
			{
        "command": "edit-article.formatOnSave",
        "title": "formatOnSave"
      }
		],
```

在这个例子中，我们使用onCommand来激活插件，这样该命令可以被快捷键等执行，从而激活插件逻辑

```
	"activationEvents": [
		"onCommand:firstExt.formatOnSave"
	],
```

##### 实现注册

下面我们在extension.js中的activate中注册命令；命令的实现我们另起一个文件`formatOnSave.js`

```js
let formatOnSave = require('./formatOnSave');
function activate(context: vscode.ExtensionContext) {  
// 注册命令名和对应的回调函数
    const insertLog = vscode.commands.registerCommand('edit-article.formatOnSave', formatOnSave);
    context.subscriptions.push(insertLog);
}
```

在同层目录中书写逻辑，逻辑很简单

- 获取当前工作区目录
- 判断配置文件是否存在
  - 如果不存在，写入指定内容
  - 如果存在，提示自动生成失败

```js
const vscode = require("vscode");
const fs = require("fs");
const path = require("path");

/**
 * 类型判断函数 传递一个要判断的类型 会返回一个函数 传要判断的值 返回是否属于之前的类型
 * @param {*} type 是否是此类型
 * @returns
 */
function typeCheck(type) {
  let types = [
    "Array",
    "Object",
    "Number",
    "String",
    "Undefined",
    "Boolean",
    "Function",
    "Map",
    "Null",
  ];
  let map = {};
  types.forEach((type) => {
    map[type] = function (target) {
      return Object.prototype.toString.call(target) === `[object ${type}]`;
    };
  });
  return map[type];
}
/**
 * 遍历对象 直接获取key value  （不会遍历原型链  forin会）
 * @param {*} obj 被遍历对象
 * @param {*} cb 回调
 */
function eachObj(obj, cb) {
  if (typeCheck("Map")(obj)) {
    for (let [key, value] of obj) {
      cb(key, value);
    }
  } else if (typeCheck("Object")(obj)) {
    for (const [key, value] of Object.entries(obj)) {
      cb(key, value);
    }
  } else {
    console.log("请传递对象类型");
  }
}
/** 判断文件是否存在
 *
 * @param {*} filePath
 * @returns
 */
const fileIsExist = async (filePath) => {
  return await fs.promises
    .access(filePath)
    .then(() => true)
    .catch((_) => false);
};
/** 写入文件
 *
 * @param {*} path
 * @param {*} buffer
 * @returns
 */
const writeFileRecursive = function (path, buffer) {
  return new Promise((res, rej) => {
    let lastPath = path.substring(0, path.lastIndexOf("/"));
    fs.mkdir(lastPath, { recursive: true }, (err) => {
      if (err) return rej(err);
      fs.writeFile(path, buffer, function (err) {
        if (err) return rej(err);
        return res(null);
      });
    });
  });
};
module.exports = function () {
  let Workspace = vscode.workspace;
  const folders = Workspace.workspaceFolders;
  let fileMap = {
    "settings.json": `
{
  	// 支持.vue文件的格式化
    "[vue]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
     // 匹配.tsx文件
    "[javascriptreact]": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
     // 匹配.tsx文件
    "[typescriptreact]": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
     // 匹配.ts文件
    "[typescript]": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "eslint.alwaysShowStatus": true,
    "eslint.format.enable": true,
    "eslint.packageManager": "yarn",
    "eslint.run": "onSave",
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true
    },
    "vetur.validation.template": false,
    "editor.formatOnPaste": true,
    "editor.formatOnType": true,
    "editor.formatOnSave": true,
  }
`,
  };

  folders.forEach((folder) => {
    // 获取当前工作区目录
    let vscodeFilePath = `${folder.uri.fsPath}/.vscode`;
    eachObj(fileMap, async (fileName, content) => {
      let filePath = path.join(vscodeFilePath, fileName);
      let isExist = await fileIsExist(filePath);
      // 判断配置文件是否存在，如果不存在，写入指定内容；如果存在，提示自动生成失败
      if (isExist) {
        vscode.window.showErrorMessage(`${fileName}已存在，自动生成失败`);
      } else {
        writeFileRecursive(filePath, content).then(
          () => {
            vscode.window.showInformationMessage(`${fileName}自动生成成功`);
          },
          (err) => {
            vscode.window.showErrorMessage(`${fileName}自动生成失败，${err}`);
          }
        );
      }
    });
  });
};

```

## 课后作业：用于感兴趣同学的拔高

- 通过文章，我们会了解到还有`tasks.json`，可以做到指定任务，如下配置可以做到【使用`command + t`执行`npm run dev`】，请扩展本文的实现，做到同时自动生成此文件

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "runNode",
            "type": "npm",
            "script": "dev",
            "group": {
                "kind": "test",
                "isDefault": true
            }
        }
    ]
}
```

