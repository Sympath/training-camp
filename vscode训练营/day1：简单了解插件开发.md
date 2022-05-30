## 要做什么：简单了解插件开发 - 插件可以做什么？

前期调研同学们书写插件的需求
![Alt text](https://tva1.sinaimg.cn/large/e6c9d24ely1h2io9zgstjj20u00w4tdu.jpg)

**可以肯定地说，当前需求都可以实现。**

在实现自己的插件之前，我们先了解VSCode插件能够做什么？

##### 基于Electron框架的能力

- 读取本地各类文件包括excel等
- 可以发送接收跨域的请求
- 创建一个本地的node服务器
- 持久化存储本地数据

##### 界面上的可扩展定制区域

![Alt text](https://tva1.sinaimg.cn/large/e6c9d24ely1h2io9zbz1oj21720u044m.jpg)

- 编辑器主界面
- 编辑器图标栏
- 编辑器侧边栏
- 编辑器底部状态栏
- 首页的欢迎页面
- 设置页自己插件的设置

##### 插件在代码编辑中可提供的能力

- 基于正则等工具自动编辑页面中代码
- 编排使用VSCode内置命令（核心）
- 自定义跳转、自动补全、悬浮提示
- 增强VSCode内置的MD预览和Git工具
- 对其他后缀名文件的解析和编辑
- 对非js/ts语言进行支持
- 自定义一个新主题

##### 插件限制

插件不能访问VS Code UI的DOM节点（如果强行改动，VSCode会提示自身损坏）

### 怎么做到：插件开发准备工作

##### 初始化一个插件开发环境

确保已安装Node.js和Git，然后使用以下命令安装Yeoman和VS Code Extension Generator：

```
npm install -g yo generator-code
```

进入到合适的文件夹运行`yo code`

准备好进行开发的TypeScript或JavaScript项目（选择js项目）

填写项目信息并选择项目结构（类似cli脚手架）

##### 启动项目

用VSCode打开你的项目，然后，在编辑器中按F5，会打开一个运行此插件的新编辑器窗口。在新窗口中在命令面板（⇧⌘P）运行命令：Hello World，会发现底部多出了一个消息提示
![Alt text](https://tva1.sinaimg.cn/large/e6c9d24ely1h2ioa08oh0j20sg0lc0v5.jpg)

##### 目录文件说明

![Alt text](https://tva1.sinaimg.cn/large/e6c9d24ely1h2io9zrmjuj20er0ibwfg.jpg)
这里面最主要的是两个文件，package.json和extension.js

- extension.ts文件是我们写插件具体逻辑的地方，放在后面详讲
- package.json文件管理着整个项目，大量属性需要我们手动配置，以下说明👇

```
{
	// 插件项目名
    "name": "firstextension",
	// 显示在应用市场的插件名，支持中文
    "displayName": "FirstExtension插件教程",
	// 插件描述
    "description": "a tutorial for extension",
	// 版本号
    "version": "0.0.1",
	// 表示插件最低支持的vscode版本
    "engines": {
        "vscode": "^1.50.0"
    },
	// 插件应用市场分类
    "categories": [
        "Other"
    ],
	// 插件图标，至少128x128像素（可选配）
    // "icon": "images/icon.png",
	// 插件的被动激活事件，可配*来保持在后台持续运行
    "activationEvents": [
        "onCommand:firstExt.hello"
    ],
	// 插件的主入口文件
    "main": "./out/extension.js",
	// 贡献点，整个插件最重要最多的配置项
    "contributes": {
		// 命令
        "commands": [
            {
                "command": "firstExt.hello",
                "title": "Hello World"
            }
        ],
		// 快捷键绑定，分为mac和win操作系统；when是触发的先决条件，可不配；
        "keybindings": [
            {
                "command": "firstExt.hello",
                "key": "ctrl+w",
                "mac": "cmd+shift+w",
                "when": "editorTextFocus"
            }
        ],
		// ---- 下面配置在后面小说案例中说明
		// 菜单
        "menus": {
			// 编辑器鼠标右键菜单
            "editor/context": [],
			// 编辑器右上角图标，不配置图片就显示文字
            "editor/title": [],
			// 编辑器标题右键菜单
            "editor/title/context": [],
			// 资源管理器右键菜单
            "explorer/context": []
        },
		// 自定义新的activitybar图标，也就是最左侧 图标栏
        "viewsContainers": {},
		// 自定义左侧侧边栏内view树的实现
        "views": {},
        // 插件自定义设置项，由用户配置后传参进入代码逻辑里，暂不展示
		"configuration": {},
    }
}
```

##### 调试

当前开发插件调试起来非常方便，只要当前行左侧点击红点，运行时即走到断点。
![Alt text](https://tva1.sinaimg.cn/large/e6c9d24ely1h2io9z1qwaj21e40t67b8.jpg)

更改代码文件保存后，需要在调试窗口执行Command+R刷新。

另外，对于ts支持也是完全自动化的，不用额外输入命令。

建议在 launch.json 中的“args”中添加"--disable-extensions"，阻止其他插件加载，不然要调试的插件加载很慢。

## 课后作业

经过介绍，第一天是不是开始准备大展拳脚呢？

别着急，VSCode提供了海量的api供我们实现各种各样的功能，千里之台，始于垒土，我们先从更改官方示例开始吧！

**留的任务**

1.更改官方示例中注册命令名字为`myExt.hello` ，并把message类型换为warning，内容换为“你好，王志远“

2.尝试增加官方示例中的该command激活途径，使可通过快捷键激活

3.（稍难）官网示例提示内容换为“要不要去百度官网呢”，底部产生两个选项“去官网”、“不去”，点击去官网可转到百度官网

