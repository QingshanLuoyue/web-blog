## CSS格式化---属性排序

### 一、背景
与同事合作开发一个项目，后面修改 CSS 时，发现属性顺序跟我写的不一样

我从事开发前端时，导师是有给我大概指定了一定的书写规范

现在开发时，看到的 CSS 属性排序不一样，看起来有点难受（代码洁癖），修改起来也多少有点不方便

都说 JavaScript 代码要求规范，那么我们仔细说说为啥 CSS 也要有：

`注：`
> 这里参考了知乎的一篇文章，他说的理由我表示赞同，这里直接拷贝放这里
> 
> [CSS 属性排序千千万，我只爱那一种](https://zhuanlan.zhihu.com/p/32905439)


```JavaScript
1、
作为写程序的人，除了希望代码的输出具有确定性;
即每写一行代码我知道我在写什么，我能预知到结果，还希望从代码风格上保持一致;
这样在写类似功能时，写出来的东西是有某种确定性的，代码会有种统一的风格在里面，同时阅读起来会很轻松

2、
当这些看似杂乱无序的属性在书写时遵从一定规律后，改起来会很方便;
比如我习惯将 width，height 这种基本的常用的属性写在元素样式的最前面，我在改的时候就直接到最前面去找

3、
会有一种，哲学上的美;
恩，就是看起来舒心的那种;
没有风格的编码是新手，想到哪写到哪，而具有一定风格的代码或许是老司机，虽然所坚持的风格不一定普适
```
<br>

### 二、规范 -- 依据 CSS 盒模型定制

流行的CSS顺序有多种书写方式

根据多年编码习惯，我比较偏向依据 `盒模型` 来定制 CSS 属性编写顺序

并且结合多种书写方式，在此基础上，做了一些调整，整理成自己喜欢的书写风格

大致分为以下几类

```javascript
1、 content overflow position z-index display float  ... 
表示定位/布局的属性（content比较特殊，作为伪元素不可少的，经常放置于第一位）

2、 width height margin padding border ... 
表示盒子模型的属性

3、 background ... 
表示背景的属性

4、 color font line-height text-* vertical-align ... 
字体相关的属性

5、 list-style ... 
除 CSS3 外的其他属性

6、 border-radius transform ... 
CSS3 属性
```

常用 CSS 属性排序
```css
/* 每一类的顺序并不严格，可以随意，自己开心就好 */
/* 下面这个是比较通用的样式 */
.el {
    /* 表示定位/布局的属性 */
    content: ;
    overflow: ;
    display: ;
    visibility: ;
    float: ;
    clear: ;

    position: ;
    top: ;
    right: ;
    bottom: ;
    left: ;
    z-index: ;

    /* 表示盒子模型的属性 */
    width: ;
    min-width: ;
    max-width: ;
    height: ;
    min-height: ;
    max-height: ;
    margin: ;
    margin-top: ;
    margin-right: ;
    margin-bottom: ;
    margin-left: ;
    padding: ;
    padding-top: ;
    padding-right: ;
    padding-bottom: ;
    padding-left: ;
    border: ;
    border-top: ;
    border-right: ;
    border-bottom: ;
    border-left: ;
    border-width: ;
    border-top-width: ;
    border-right-width: ;
    border-bottom-width: ;
    border-left-width: ;
    border-style: ;
    border-top-style: ;
    border-right-style: ;
    border-bottom-style: ;
    border-left-style: ;
    border-color: ;
    border-top-color: ;
    border-right-color: ;
    border-bottom-color: ;
    border-left-color: ;

    /* 表示背景的属性 */
    background: ;
    background-color: ;
    background-image: ;
    background-repeat: ;
    background-position: ;

    /* 字体相关的属性 */
    color: ;
    font: ;
    font-family: ;
    font-size: ;
    font-weight: ;
    line-height: ;
    text-align: ;
    text-indent: ;
    text-transform: ;
    text-decoration: ;
    letter-spacing: ;
    word-spacing: ;
    white-space: ;
    vertical-align: ;

    /* 除 CSS3 外的其他属性 */
    outline: ;
    list-style: ;
    table-layout: ;
    caption-side: ;
    border-collapse: ;
    border-spacing: ;
    empty-cells: ;
    opacity: ;
    cursor: ;
    quotes: ;

    /* CSS3 属性 */
    border-radius: ;
    transform: ;
}
```

<br>

### 三、使用工具来约束 CSS 规范

1、csscomb.js --- CSS 编码风格格式化工具

2、Github： [csscomb.js](https://github.com/csscomb/csscomb.js)

3、[csscomb.json / yandex.json / zen.json](https://github.com/csscomb/csscomb.js/tree/dev/config)

预配置三种编码风格，可以挑选自己想要的直接copy代码

4、配置文件：
创建 csscomb.json 配置文件，放于项目根目录

将上面的风格代码 copy 之后，放到 csscomb.json 即可

5、配合开发工具使用

我这里是使用 VS Code，到插件库里搜索下载 `CSScomb` 安装

该插件支持
```javascript
.csscomb.json or csscomb.json
.csscomb.js or csscomb.js
```
等配置文件格式

使用方式：
>F1, Ctrl+Shift+P on Windows
>
>Cmd+Shift+P on macOS

6、贴一份自己使用的配置文件

配置代码太长，独立一篇随笔，这里放个链接

[CSScomb.js --- 自定义 CSS 编写风格配置文件](https://www.cnblogs.com/linjunfu/p/11558007.html)