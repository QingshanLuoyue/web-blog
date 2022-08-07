为了让人容易理解，且不踩坑，官方文档会写的很详细

我们在熟练使用之后，需要提炼属于自己的知识点和关注点，总结以便后续快速查阅

官网文档地址[https://cli.vuejs.org/zh/guide/](https://cli.vuejs.org/zh/guide/)

## 一、介绍

注意最新版是 @vue/cli 不是 vue-cli

- @vue/cli
  - 项目脚手架
  - 包含运行时依赖 `@vue/cli-service`，可升级，同步升级其他如 `webpack` 等工具
  - 包含丰富的官方插件集合，如 `@vue/cli-plugin-babel`、`@vue/cli-plugin-eslint` 等
  - 支持图形化创建、管理项目

- `@vue/cli-service` 作为 `devDependencies` 局部安装在项目中，提供 `vue-cli-service` 命令

<br>

## 二、安装

### Node.js 版本最好在 10 以上
<br>

### 若存在旧版 `vue-cli` 需要提前卸载
```bash
npm uninstall vue-cli -g
# OR
yarn global remove vue-cli
```
<br>

### 安装
```bash
npm install -g @vue/cli
#  OR
yarn global add @vue/cli
```

**安装完成之后，可以全局使用 `vue` 命令**<br>
安装各种插件和扩展后， `vue` 可以使用的命令有：
```javascript
vue serve // 启动开发原型项目
vue build // 构建项目
vue config // 审查或修改全局的 CLI 配置
vue ui // 启动图形化界面创建或者管理项目
vue create // 创建项目
vue inspect // 查看当前项目 webpack 配置
```
<br>

### 升级
```bash
npm update -g @vue/cli
# 或者
yarn global upgrade --latest @vue/cli
```

<br>

## 三、基础知识

### 原型开发（单纯以一个 js / vue 文件启动）
1、需要安装全局扩展
```bash
npm install -g @vue/cli-service-global
```

2、开发一个页面
```bash
vue serve 
    -o, --open  打开浏览器
    -c, --copy  将本地 URL 复制到剪切板
    -h, --help  输出用法信息
    [entry], mycomponent.vue/app.vue/index.js/或其他 .js / .vue 类型入口文件
```

例子:
```bash
vue serve mycomponent.vue
```

3、构建一个生产包
```bash
vue build 
    -t, --target <target> 构建目标 (app | lib | wc | wc-async, 默认值：app)
    -n, --name <name> 库的名字或 Web Components 组件的名字 (默认值：入口文件名)
    -d, --dest <dir> 输出目录 (默认值：dist)
    -h, --help 输出用法信息
    [entry], mycomponent.vue/app.vue/index.js/或其他 .js / .vue 类型入口文件
```

例子:
```bash
vue build -t app mycomponent.vue
```
<br>

### 创建项目

1、使用 cli 创建: vue create 创建项目

根据命令行交互，选择配置<br>
默认的 preset 包含了基本的 Babel + ESLint
```bash
vue create myapp
```

2、使用图形界面创建

vue ui 启动一个图形界面来创建和管理项目
```bash
vue ui
```
<br>

### CLI 服务

1、启动项目服务

初始项目中，存在命令
```javascript
// package.json
{
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build"
  }
}
```

2、启动项目命令
```bash
npm run serve
# OR
yarn serve
```

3、命令详解

**vue-cli-service serve**<br>
>基于 webpack-dev-server 启动一个开发服务器<br>
开箱即用的模块热重载
```bash
vue-cli-service serve
    --open    在服务器启动时打开浏览器
    --copy    在服务器启动时将 URL 复制到剪切版
    --mode    指定环境模式 (默认值：development)
    --host    指定 host (默认值：0.0.0.0)
    --port    指定 port (默认值：8080)
    --https   使用 https (默认值：false)
```

**vue-cli-service build**
>构建项目
```bash
vue-cli-service build
    --mode        指定环境模式 (默认值：production)
    --dest        指定输出目录 (默认值：dist)
    --modern      面向现代浏览器带自动回退地构建应用
    --target      app | lib | wc | wc-async (默认值：app)
    --name        库或 Web Components 模式下的名字 (默认值：package.json 中的 "name" 字段或入口文件名)
    --no-clean    在构建项目之前不清除目标目录
    --report      生成 report.html 以帮助分析包内容
    --report-json 生成 report.json 以帮助分析包内容
    --watch       监听文件变化
```

**vue-cli-service inspect**
>查看项目的 webpack config
```bash
vue-cli-service inspect
    --mode    指定环境模式 (默认值：development)
```

4、缓存和并行

**cache-loader**<br>
- 缓存编译结果，下次编译复用<br>
- 默认为 Vue/Babel/TypeScript 编译开启缓存。文件缓存在 node_modules/.cache<br>
- 遇到编译问题，可以删除缓存试试
  
**thread-loader**<br>
- 多线程转译代码，加快转译速度<br>
- 会在多核 CPU 上为 Babel/TypeScript 转译开启

5、Git Hook

@vue/cli-service 会安装 yorkie<br>
在 package.json 中配置 git hook
```javascript
// 在 pre-commit 阶段，执行 lint-staged
// lint-staged 时候若提交有 js、vue文件，执行
// "vue-cli-service lint",
// "git add"
// 进行校验
{
  "gitHooks": {
    "pre-commit": "lint-staged"
  },
   "lint-staged": {
    "*.{js,vue}": [
      "vue-cli-service lint",
      "git add"
    ]
  }
}
```
<br>

## 四、开发

### 兼容性

1、browserlist

根据 package.json 中的 browserlist 字段或者 .browserlistrc 文件，指定需要支持的浏览器范围<br>
该值会被 @babel/preset-env 和 Autoprefixer 来确定需要转译的
javascript 特性和需要添加的浏览器前缀

2、polyfill

useBuiltIns：<br>
- usage
  - 根据 browserlist 目标，检测源代码中需要转化的语言特性，自动加载对应的 polyfill
- entry
  - 在入口文件添加 import 'core-js/stable'; import 'regenerator-runtime/runtime';
  - 会根据 browserslist 目标导入所有 polyfill
- false
  - 构建一个库或者 Web Component
  - 确保你的库或是组件不包含不必要的 polyfills。打包 polyfills 应当是最终使用你的库的应用的责任

```javascript
// babel.config.js
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "entry"
      }
    ]
  ]
}
```

3、现代模式

产生两个应用的版本：一个现代版的包，面向支持 ES modules 的现代浏览器，另一个旧版的包，面向不支持的旧浏览器
```bash
vue-cli-service build --modern
```

现代版的包会通过 `<script type="module">` 在被支持的浏览器中加载；<br>
它们还会使用 `<link rel="modulepreload">` 进行预加载：<br>
- `<script type="module">` 需要配合始终开启的 CORS 进行加载
- 服务器必须返回诸如 Access-Control-Allow-Origin: * 的有效的 CORS 头
- 旧版的包会通过 `<script nomodule>` 加载，并会被支持 ES modules 的浏览器忽略

<br>

### HTML 与静态资源

1、HTML
  - html-webpack-plugin
    - 使用 public/index.html 作为页面模板
    - 生成的资源链接会被自动注入该页面
    - 自动注入资源提示（预加载preload/空闲时获取prefetch）

2、插值
  - html-webpack-plugin 本身暴露的变量
  - 客户端的环境变量
    - .env\.env.local 等等


3、preload （`<link rel="preload">`）
  - 页面加载的过程中，我们希望在浏览器开始主体渲染之前尽早 preload
  - 默认情况下 Vue CLI 会为所有初始化渲染需要的文件自动生成 preload 提示
  - preload 标记由 @vue/preload-webpack-plugin 注入，可通过修改 chainWebpack 删除
    - config.plugin('preload')


4、prefetch （`<link rel="prefetch">`）
  - 在页面加载完成后，利用空闲时间提前获取用户未来可能会访问的内容
  - 默认情况下，Vue CLI 会为所有作为 async chunk 生成的 JavaScript 文件 (通过动态 import() 按需 code splitting 的产物) 自动生成 prefetch 提示。
  - prefetch 标记由 @vue/preload-webpack-plugin 注入，可通过修改 chainWebpack 删除
    - config.plugin('prefetch') 

5、构建多页面应用
  -  配置 vue.config.js 中的 pages 选项可以构建一个多页面的应用
  -  构建好的应用将会在不同的入口之间高效共享通用的 chunk 以获得最佳的加载性能

6、资源路径处理规则
  - 以 . 开头
    - 当做相对路径处理，根据当前项目的目录结构解析
  - 以 ~ 开头
    - 其后的任何内容都会作为一个模块请求被解析。这意味着你甚至可以引用 Node 模块中的资源
    - 例如：`<img src="~some-npm-package/foo.png">`
  - 以 @ 开头
    - Vue CLI 默认设置一个指向 `<projectRoot>/src` 的别名 @
    - 解析时候将 @ 替换
  - 以 / 开头
    - 按照绝对路径解析

- public
  - 任何放置在 public 文件夹的静态资源都会被简单的复制，而不经过 webpack。你需要通过绝对路径来引用它们
<br>

### CSS 相关

1、基础
- 天生支持 PostCSS、CSS Modules、Sass、Less、Stylus 预处理器
- 所有编译后的 css 通过 css-loader 处理 url() 中的资源引用
- 想要引用 npm 依赖或者使用 webpack 别名，需要在前面加载 ~ 作为前缀，避免歧义 

2、PostCSS 配置
  - postcss-loader
    - .postcssrc
    - postcss-config.js
    - vue.config.js 中的 css.loaderOptions.postcss

3、向预处理器 Loader 传递选项
```javascript
// vue.config.js
module.exports = {
  css: {
    loaderOptions: {
      // 给 sass-loader 传递选项
      sass: {
        // @/ 是 src/ 的别名
        // 所以这里假设你有 `src/variables.sass` 这个文件
        // 注意：在 sass-loader v8 中，这个选项名是 "prependData"
        additionalData: `@import "~@/variables.sass"`
      },
      // 默认情况下 `sass` 选项会同时对 `sass` 和 `scss` 语法同时生效
      // 因为 `scss` 语法在内部也是由 sass-loader 处理的
      // 但是在配置 `prependData` 选项的时候
      // `scss` 语法会要求语句结尾必须有分号，`sass` 则要求必须没有分号
      // 在这种情况下，我们可以使用 `scss` 选项，对 `scss` 语法进行单独配置
      scss: {
        additionalData: `@import "~@/variables.scss";`
      },
      // 给 less-loader 传递 Less.js 相关选项
      less:{
        // http://lesscss.org/usage/#less-options-strict-units `Global Variables`
        // `primary` is global variables fields name
        globalVars: {
          primary: '#fff'
        }
      }
    }
  }
}
```
<br>

### webpack

1、简单配置
  - 该对象将会被 webpack-merge 合并入最终的 webpack 配置
```javascript
// vue.config.js
module.exports = {
  configureWebpack: {
    plugins: [
      new MyAwesomeWebpackPlugin()
    ]
  }
}
```

**注意：有些 webpack 选项是基于 vue.config.js 中的值设置的，所以不能直接修改**
  - 修改 vue.config.js 中的 outputDir，而不是 output.path
  - 修改 vue.config.js 中的 publicPath ，而不是 output.publicPath
  - **这样做是因为 vue.config.js 中的值会被用在配置里的多个地方，以确保所有的部分都能正常工作在一起**

<br>

基于环境有条件的配置或者直接修改配置
```javascript
// vue.config.js
module.exports = {
  configureWebpack: config => {
    // config: 已经解析好的配置
    // 可以直接修改 config 配置，或者返回一个将会被合并的对象
    if (process.env.NODE_ENV === 'production') {
      // 为生产环境修改配置...
    } else {
      // 为开发环境修改配置...
    }
  }
}
```

2、链式操作 (高级)

>**通过 webpack-chain 维护<br>
这个库提供了一个 webpack 原始配置的上层抽象，使其可以定义具名的 loader 规则和具名插件，并有机会在后期进入这些规则并对它们的选项进行修改<br>
可以使用 vue inspect 查看 webpack 配置**

```javascript
// vue.config.js
module.exports = {
  chainWebpack: config => {
    config.module
      .rule('vue')
      .use('vue-loader')
        .tap(options => {
          // 修改它的选项...
          return options
        })
  }
}
```

- 实例操作

添加loader
```javascript
// vue.config.js
module.exports = {
  chainWebpack: config => {
    // GraphQL Loader
    config.module
      .rule('graphql')
      .test(/\.graphql$/)
      .use('graphql-tag/loader')
        .loader('graphql-tag/loader')
        .end()
      // 你还可以再添加一个 loader
      .use('other-loader')
        .loader('other-loader')
        .end()
  }
}
```

替换loader
```javascript
// vue.config.js
module.exports = {
  chainWebpack: config => {
    const svgRule = config.module.rule('svg')

    // 清除已有的所有 loader。
    // 如果你不这样做，接下来的 loader 会附加在该规则现有的 loader 之后。
    svgRule.uses.clear()

    // 添加要替换的 loader
    svgRule
      .use('vue-svg-loader')
        .loader('vue-svg-loader')
  }
}
```
修改插件
```javascript
// vue.config.js
module.exports = {
  chainWebpack: config => {
    config
      .plugin('html')
      .tap(args => {
        // 1、返回新的配置参数
        return [/* 传递给 html-webpack-plugin's 构造函数的新参数 */]
        
        // 2、修改现有参数
        // args[0].template = '/Users/username/proj/app/templates/index.html'
        // return args
      })
  }
}
```

- 审查 webpack 配置

将配置导出到 output.js 进行查阅
```bash
vue inspect > output.js
```

指定一个路径来审查配置的一小部分
```bash
# 只审查第一条规则
vue inspect module.rules.0
```

指向一个规则或插件的名字
```bash
vue inspect --rule vue
vue inspect --plugin html
```

列出所有规则和插件的名字
```bash
vue inspect --rule
vue inspect --plugin
```
<br>

### 模式和环境变量

1、模式---根据 mode 参数不一样， Vuc CLI 会解析出不同的 webpack 配置
- vue-cli-service build --mode development
  - 该配置启用热更新，不会对资源进行 hash 也不会打出 vendor bundles
- vue-cli-service build --mode production
  - 压缩、混淆、提取公共代码等等
- vue-cli-service build --mode test
  - 不会处理图片以及一些对单元测试非必需的其他资源

2、环境变量
- 项目根目录中放置下列文件来指定环境变量：
```bash
.env                # 在所有的环境中被载入
.env.local          # 在所有的环境中被载入，但会被 git 忽略
.env.[mode]         # 只在指定的模式中被载入
.env.[mode].local   # 只在指定的模式中被载入，但会被 git 忽略
```

3、环境文件
- 格式：键=值
- 只有 `NODE_ENV，BASE_URL` 和以 `VUE_APP_` 开头的变量将通过 `webpack.DefinePlugin` 静态地嵌入到客户端侧的代码中
```bash
NODE_ENV=production
VUE_APP_TITLE=My App
FOO=foo
BAR=bar
CONCAT=$FOO$BAR # CONCAT=foobar

```
4、环境文件加载优先级
  - 为一个特定模式准备的环境文件 (例如 .env.production) 将会比一般的环境文件 (例如 .env) 拥有更高的优先级
  - Vue CLI 启动时已经存在的环境变量拥有最高优先级，并不会被 .env 文件覆写
  - .env 环境文件是通过运行 vue-cli-service 命令载入的，因此环境文件发生变化，需要重启服务

<br>

### 构建目标

1、应用模式（默认模式）
```bash
vue-cli-service build 
```

2、库模式
```bash
# Vue 是外置的
vue-cli-service build --target lib

# Vue 是内联的
vue-cli-service build --target lib --inline-vue

# 指定名字
vue-cli-service build --target lib --name myLib [entry]
```

3、Vue vs. JS/TS 入口文件
- 使用一个 .vue 文件作为入口时，库会直接暴露这个 Vue 组件本身
- 使用一个 .js 或 .ts 文件作为入口时，它可能会包含具名导出，所以库会暴露为一个模块
  - UMD：window.yourLib.default
  - CommonJS：const myLib = require('mylib').default 
- 若没有任何具名导出并希望直接暴露默认导出
```javascript
// vue.config.js
module.exports = {
    configureWebpack: {
        output: {
            libraryExport: 'default'
        }
    }
}
```

4、Web Components 组件(不支持 IE11)
  - Vue 以外置形式处理
    
```bash
vue-cli-service build --target wc --name my-element [entry]

# 注册多个 Web Components 组件的包
vue-cli-service build --target wc --name foo 'src/components/*.vue'

# 异步 Web Components 组件
vue-cli-service build --target wc-async --name foo 'src/components/*.vue'
```
