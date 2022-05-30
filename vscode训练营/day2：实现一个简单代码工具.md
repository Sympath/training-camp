## 要做什么：实现一个简单代码工具

完成一个简单插件，功能是 使用快捷键 或者鼠标右键选项，就能把选中的内容在下一行用console输出

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h2iodjcdcvg20wu0u0div.gif" alt="Alt text" style="zoom:33%;" />

## 怎么做到：尽可能做到复制粘贴就能有成果

### 一、在package.json中注册激活命令

##### 取一个名字

command的名字一般是[插件名称(自取).[具体名称]，这里我就叫做 `firstExt.insertLog`；这时我们在contributes里面的commands写入名字和title（title可设为中文，在配置后会展示在鼠标右键选项里）

```
		"commands": [
			{
				"command": "firstExt.hello",
				"title": "Hello World"
			},
			{
				"command": "firstExt.insertLog",
				"title": "Insert Log Statement"
			}
		],
```


##### 将命令注册到activationEvents（激活事件）

激活事件发生时，该扩展插件将被激活。下面是官方提供的激活时机，如果设置为*，那么插件就会一直处在激活状态在后台运行。

![Alt text](https://tva1.sinaimg.cn/large/e6c9d24ely1h2iodih42pj20a709rgly.jpg)

在这个例子中，我们使用onCommand来激活插件，这样该命令可以被快捷键等执行，从而激活插件逻辑

```
	"activationEvents": [
		"onCommand:firstExt.hello",
		"onCommand:firstExt.insertLog"
	],
```

##### 添加快捷键

我们在contributes里的keybindings写入自定义的快捷键

```
		"keybindings": [
			{
				"command": "firstExt.hello",
				"key": "ctrl+e",
				"mac": "cmd+shift+e",
			},
			{
				"command": "firstExt.insertLog",
				"key": "shift+ctrl+l",
				"mac": "shift+cmd+l",
				// 只有在编辑器中的文本具有焦点（光标闪烁）执行
				"when": "editorTextFocus"
			}
		],
```

##### 添加为鼠标右键选项

我们在contributes => menus => editor/context里面添加右键选项

```
		"menus": {
			"editor/context": [
				{
					"when": "editorTextFocus",
					"command": "firstExt.insertLog",
					// 右键选项中出现的顺序
					"group": "navigation"
				}
			]
		}
```

该编辑器鼠标右键菜单中有这些默认组group：

- navigation-navigation在所有情况下都排在第一位。
- 1_modification -接下来是该组，其中包含用于修改代码的命令。
- 9_cutcopypaste -使用基本编辑命令的是倒数第二个默认组。
- z_commands -最后一个默认组，其中包含用于打开命令面板的条目。

![Alt text](https://tva1.sinaimg.cn/large/e6c9d24ely1h2iodk29wmj20hc07jt9i.jpg)

（我们也可以自定义添加几个菜单组)

### 二、调用命令

Extension.js是插件的入口，一般包括两个函数 

- activate ：插件激活时也就是在注册的 Activation Event 发生的时候就会执行
- deactivate。插件关闭时执行的代码。

下面我们在extension.js中的activate中书写逻辑

```
export function activate(context: vscode.ExtensionContext) {  
// 注册命令名和对应的回调函数
    const insertLog = vscode.commands.registerCommand('firstExt.insertLog', () => {
	    // 拿到当前编辑页面的内容对象 editor
        const editor = vscode.window.activeTextEditor;
        // 拿到光标选中的文本并格式化
        const selection = editor.selection;
        const text = editor.document.getText(selection);
        // 在这里拼写console语句
		const logToInsert = `console.log('${text}: ',${text});\n`;
		// 执行插行方法
		text ? insertText(logToInsert) : insertText('console.log();');
    });
    // 最后，我们把注册的命令事件放入上下文的订阅中 
    context.subscriptions.push(insertLog);
}
```

registerCommand完整的 API 是：registerCommand(command: string, callback: (args: any[]) => any, thisArg?: any): Disposable
主要功能是给功能代码(callback)注册一个命令(command)，然后通过 subscriptions.push() 给插件订阅对应的 command 事件。

### 三、具体逻辑

#### 文件编辑方法

- editBuilder中的insert、delete和replace方法

我们在编辑页面时有3个方法：insert、delete和replace。顾名思义，不展开了。

#### 当前位置和区域

> vscode.Position 与 vscode.Range

vscode.Position是一个坐标，传入行数和列数（char）；

vscode.Range是由两个vscode.Position确定头尾后的一块区域。

```
const insertText = (val) => {
    const editor = vscode.window.activeTextEditor;
    const selection = editor.selection;
    // 获取光标当前行
    const lineOfSelectedVar = selection.active.line;
	// edit方法获取editBuilder实例，在后一行添加
    editor.edit((editBuilder) => {
		editBuilder.insert(new vscode.Position(lineOfSelectedVar + 1, 0),val)
	});
}
```



## 课后作业：用于感兴趣同学的拔高

1.对console使用css，增加console图形样式、颜色的个性化，发挥创造力

![Alt text](https://tva1.sinaimg.cn/large/e6c9d24ely1h2iodkptoyj20n402zjro.jpg)

2.根据选中文本输出不同的类型，如检测到err或者error，就用console.error输出，检测到数组中括号，用console.tabel输出

3.（稍难）添加console.time的应用场景，选中大段代码后，使用特定快捷键，在代码段前后增加console.time和console.timeEnd统计代码执行时间