## VS Code 插件开发
<br>

[VS Code 插件开发文档](https://code.visualstudio.com/api/get-started/your-first-extension)

[VS Code Github Samples](https://code.visualstudio.com/api/extension-guides/overview)

[VS Code 插件开发清单 - package.json 说明](https://code.visualstudio.com/api/references/extension-manifest)

[Contribution 配置说明](https://code.visualstudio.com/api/references/contribution-points)：插件配置，比如命令，菜单，视图等等

[Activation  配置说明](https://code.visualstudio.com/api/references/activation-events)：激活时机配置

[插件发布说明](https://code.visualstudio.com/api/working-with-extensions/publishing-extension)



<br>

### 一、环境安装

#### 1、安装 `yo` 和 `generator-code`

```javascript
 yarn global add yo generator-code
```

#### 2、执行 `yo code` 生成插件脚手架

```javascript
yo code
```

![](./md-img/VSCode-extension/01-init-config.png)

>配置项说明:<br>
选择插件类型<br>* ? What type of extension do you want to create? New Extension (TypeScript)<br>
设置插件名字<br>* ? What's the name of your extension? HelloWorld<br>
插件标识，可选择默认<br>* ? What's the identifier of your extension? helloworld<br>
插件描述，可选择默认<br>* ? What's the description of your extension? LEAVE BLANK<br>
是否初始化一个 git 仓库地址，可选择默认<br>* ? Initialize a git repository? Yes<br>
选择包管理器，可选择默认<br>* ? Which package manager to use? npm

<br>

#### 3、生成项目并自动安装依赖后，使用 VS Code 打开项目
```javascript
code ./my-vscode-plugin
```

**按 F5 启动项目，将会进行编译，并启动一个新的 vscode 插件开发主机窗口<br>
当前插件会直接作用在这个新的窗口，窗口上方会显示 `[扩展开发宿主]`**

Windows 平台下，新的窗口中，同时按下 ctrl + shift + p 调出命令控制面板

输入 hello 找到 命令 `my-vscode-plugin.helloWorld`

![](./md-img/VSCode-extension/02-command-search.png)

执行输出:

![](./md-img/VSCode-extension/03-command-excuted.png)

<br>

### 二、上手修改
<br>

对插件进行修改
1. 修改提示语
2. 修改注册命令名

<br>

#### **修改前：**

`activate` 函数

![](./md-img/VSCode-extension/04-activate.png)

`package.json`

![](./md-img/VSCode-extension/05-package.png)


#### **修改后：**

`activate` 函数

![](./md-img/VSCode-extension/06-activate-modify.png)

`package.json`

![](./md-img/VSCode-extension/07-package-modify.png)


package.json 说明：

```javascript
// package.json
{
    // https://code.visualstudio.com/api/references/activation-events
    // 激活时机配置，"onCommand:first-plugin.helloWorld" 表示在执行 first-plugin.helloWorld 时激活
    "activationEvents": [
        "onCommand:first-plugin.helloWorld"
    ],
    
    // 程序入口
    "main": "./extension.js",

    // https://code.visualstudio.com/api/references/contribution-points
    // 插件配置，比如命令，菜单，视图等等
    // 这里指定了一个 'first-plugin.helloWorld' 命令
    // 该命令可以在 ctrl + shift + p 面板来调用
    // 但是命令必须在 activate 或者 插件生命周期中使用 vscode.commands.registerCommand 注册过才能使用，不可无中生有
	"contributes": {
		"commands": [{
            "command": "first-plugin.helloWorld",
            // ctrl + shift + p 调用命令面板后，输入关键字匹配命令，使用的是 title 值来匹配
            "title": "Hello First Plugin"
		}]
	}
}
```

修改后重新载入窗口

ctrl + shift + p 调出命令控制面板

输入 Hello First Plugin 找到 命令

![](./md-img/VSCode-extension/08-command-search-modify.png)

输出:

![](./md-img/VSCode-extension/09-command-excuted-modify.png)

<br>

### 三、打包发布

<br>

发布说明：
```javascript
Visual Studio Code leverages Azure DevOps for its Marketplace services. This means that authentication, hosting, and management of extensions are provided through Azure DevOps.

vsce can only publish extensions using Personal Access Tokens. You need to create at least one in order to publish an extension.


Visual Studio 代码利用 Azure DevOps 作为插件市场服务。
这意味着扩展的身份验证、托管和管理，通过 Azure DevOps 提供

vsce 只能使用 `Personal Access Tokens` 发布扩展访问。
您需要创建至少一个用以发布一个扩展
```

1、所以先要去 [Azure DevOps](https://azure.microsoft.com/services/devops/) 注册账号

2、获取 `Personal Access Tokens` https://dev.azure.com/
>备注：`Personal Access Tokens` 会过期，失效后，需要重新创建

`Personal Access Tokens` 创建流程
![](./md-img/VSCode-extension/token1.png)
![](./md-img/VSCode-extension/token2.png)
![](./md-img/VSCode-extension/token3.png)
![](./md-img/VSCode-extension/token4.png)

创建好后，复制保存下来

3、 全局安装 vsce

VS Code 插件使用 vsce 来进行管理，如打包、发布等
 
```javascript
yarn global add vsce
```

4、创建发布者
```javascript
vsce create-publisher (publisher name)
// 执行后，会让你输入 Personal Access Tokens
// 将上面创建好的 Personal Access Tokens 输入
```

5、登录创建好的发布者
```javascript
vsce login (publisher name)
// 执行后，会让你输入 Personal Access Tokens
```

6、打包插件
```javascript
vsce package
```

7、发布插件
```javascript
vsce  publish 
// 发布完成后，大概几分钟后可以在插件市场搜索到
```

<br>

### 四、QA

Windows 下，gitbash 针对 vsce 的一些命令操作无法适配，建议使用自带的 cmd 进行操作

`Personal Access Tokens` 会过期，如果提示没有权限访问，则表示失效，需要重新创建