## 要做什么：完善代码工具并发布

实现功能

  1. 一键删除当前页面所有console 
  2. 一键删除当前页面所有注释并格式化代码

期待成功

在应用市场中看到自己的插件

## 怎么做到：尽可能做到复制粘贴就能有成果

### 一、一键删除所有console

思路：我们首先获取所有log位置，然后把该位置置成空

##### 获取所有console

```
function getAllLogStatements() {
	const editor = vscode.window.activeTextEditor;
	// 获取编辑器页面文本
	const document = editor.document;
	const documentText = document.getText();

	let logStatements = [];
	// 检测console的正则表达式
	const logRegex = /console.(log|debug|info|warn|error|assert|dir|dirxml|trace|group|groupEnd|time|timeEnd|profile|profileEnd|count)\((.*)\);?/g;
	let match;
	// 正则循环匹配页面文本
	while (match = logRegex.exec(documentText)) {
	// 每次匹配到当前的范围--Range
		let matchRange =
			new vscode.Range(
				document.positionAt(match.index),
				document.positionAt(match.index + match[0].length)
			);
		if (!matchRange.isEmpty)
		// 把Range放入数组
			logStatements.push(matchRange);
	}
	return logStatements;
}
```

##### 把获取到的console逐个删除

```
	const deleteAllLog = vscode.commands.registerCommand('firstExt.delLog', () => {
		const editor = vscode.window.activeTextEditor;
		if (!editor) { return; }

		let workspaceEdit = new vscode.WorkspaceEdit();
		const document = editor.document;

		const logStatements = getAllLogStatements();

		// 循环遍历每个匹配项的range，并删除
		logStatements.forEach((log) => {
			workspaceEdit.delete(document.uri, log);
		});
		// 完成后显示消息提醒
		vscode.workspace.applyEdit(workspaceEdit).then(() => {
			vscode.window.showInformationMessage(`${logStatements.length} console.log deleted`);
		});
	});
	context.subscriptions.push(deleteAllLog);
```

补充：vscode.WorkspaceEdit 工作区编辑方法，可操作多个文件，也可新增或删除 文本、文件、文件夹

###  二、一键删除所有注释并格式化代码

第一个思路还是正则替换法

```
	let removeComments = vscode.commands.registerCommand('firstExt.removeComments', function () {
		const editor = vscode.window.activeTextEditor;
		editor.edit(editBuilder => {
			let text = editor.document.getText();
			// 正则匹配替换掉注释文本
			text = text.replace(/((\/\*([\w\W]+?)\*\/)|(\/\/(.(?!"\)))+)|(^\s*(?=\r?$)\n))/gm, '').replace(/(^\s*(?=\r?$)\n)/gm, '').replace(/\\n\\n\?/gm, '');
			// 全量替换当前页面文本
			const end = new vscode.Position(editor.document.lineCount + 1, 0);
			editBuilder.replace(new vscode.Range(new vscode.Position(0, 0), end), text);
			// 执行格式化代码操作
			vscode.commands.executeCommand('editor.action.formatDocument');
		});
	});

	context.subscriptions.push(removeComments);
```

- 调用VSCode内部命令方法：vscode.commands.executeCommand 传入参数是内置api名称
  <img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h2ioh72obej20wg0u00wv.jpg" alt="Alt text" style="zoom:33%;" />
  VSCode暴露了大量的自身api可供选择，善用api完全可以打造一套VSCode工作流

### 三、 打包 & 发布

### 发布，先创建账号

Visual Studio Code 的**应用市场**基于微软自己的`Azure DevOps`；所以需要在两个平台都注册好账号。有账号后，根据 Azure DevOps 平台生成`PAT`（`Personal Access Token`，个人访问令牌）；根据**应用市场**创建发布者账号，有了这两者就能发布了。

整理如下

- 注册账号
- 根据 Azure DevOps 平台生成`PAT`（`Personal Access Token`，个人访问令牌）；
- 根据**应用市场**创建发布者账号
- 根据 vsce 在本地发布

### 根据 Azure DevOps 平台生成 token

##### 登录 Microsoft

首先访问[Microsoft 官网]( https://login.live.com/ )登录你的`Microsoft`账号，没有的先注册一个：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h231rnykd9j20qi0k0dgz.jpg)



中间会要求输入邮箱，没有也可以直接通过手机号获取一个

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h231qb2s6zj20v20rcmz5.jpg)



一路 next，直到创建成功。

##### 登录 Azure

然后访问：[Azure 官网]( https://aka.ms/SignupAzureDevOps) ，如果你从来没有使用过 Azure，那么会看到如下提示：

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h231wbevkqj20qa0ri775.jpg" alt="image-20220510092738875" style="zoom:33%;" />

点击继续，默认会创建一个以邮箱前缀为名的组织。(只需要填写验证码即可)

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h231xmb2fij20nw0xgtax.jpg" alt="image-20220510092853732" style="zoom:33%;" />

### 创建令牌

默认进入组织的主页后，点击右上角的`Security`：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h231zo0c56j21h00u0jum.jpg)

点击创建新的个人访问令牌，这里特别要注意

- `Organization`要选择`all accessible organizations``
- ``Scopes`要选择`Full access

否则后面发布会失败。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h231cx0iolj20hr0dywft.jpg)

创建令牌成功后你**需要本地记下来**，因为网站是不会帮你保存的。

### 根据应用市场创建发布者账号

访问[应用市场](https://marketplace.visualstudio.com/manage/createpublisher?managePageRedirect=true)，填完名称，滑到最后保存即可

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h2322ny359j219l0u0juj.jpg" alt="image-20220510093343787" style="zoom:33%;" />	



至此，我们就有了如下内容

- PAT：Personal Access Token，个人访问令牌
- publisher：发布者账号

就可以执行在本地发布的动作啦！

### 根据 vsce 在本地发布

进入自己的插件，如果没有，可以阅读【插件脚手架：如何创建一个插件】了解创建

##### 配置插件信息

在发布前，我们需要配置一些项目信息在 package.json 中，下面是必备信息

- publisher：发布者 id
- activationEvents：扩展的激活事件，建议先填指定文件类型激活（如 js 文件，`onLanguage:javascript`）配置项可见[官网文档](https://code.visualstudio.com/api/references/activation-events)
- repository：仓库地址

其他还有很多配置，我们慢慢了解，文末放置 package.json 配置总览。

安装命令包

##### 利用发布工具发布插件

[vsce](https://github.com/Microsoft/vsce)是一个用于将插件发布到[市场](https://code.visualstudio.com/docs/editor/extension-gallery)上的命令行工具。

请确认本机已经安装了[Node.js](https://nodejs.org/)，然后运行：

```
npm install -g vsce
```

然后你就可以在命令行里直接使用`vsce`了。下面是一个快速发布的示例

1. 登陆

```
vsce login [publisherName]
# 然后输入之前获取到的 token
```

2. 执行发布（这里的 patch 是指自动更新版本后发布，不带也可以，就得手动修改`package.json`文件了）

```
$ vsce publish patch
Publishing uuid@0.0.1...
Successfully published uuid@0.0.1!
```

完成后过几分钟就可以去 vscode 的扩展区看自己的插件啦！

### 查看结果

一图胜千言，静静体会成就感的美妙吧！



## 课后作业：用于感兴趣同学的拔高

恭喜，你已经拥有自己的插件啦！