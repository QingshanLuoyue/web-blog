## JavaScript 网页异常捕获

### 一、异常大概分类

<br>

**一般我们想要捕获的异常大概分类：**

### **1、语法错误**

开发阶段通过 IDE 提示和 eslint 等工具处理

注意：<br>
**a、onerror 事件代码块与 `语法错误代码块` 不在一起**<br>
**b、或者同在一个代码块，但是 `语法错误代码块` 异步执行**<br>
都可以用 onerror 捕获语法错误
```javascript
setTimeout(() => {
   eval('function()') 
}, 1000);
// Uncaught SyntaxError: Function statements require a function name
```

<br>

### **2、引用错误，类型错误，uri 错误，范围错误等等**

非 try catch 包裹情况下<br>
可以使用 onerror 捕获同步错误、异步错误
```javascript
console.log(a)
// Uncaught ReferenceError: a is not defined

Array.test() // 调用了 Array 上不存在的 test，值为 undefined，作为函数执行，则会抛出类型错误
// Uncaught TypeError: Array.test is not a function

new Array(12221312312)
// Uncaught RangeError: Invalid array length

decodeURI('%')
// Uncaught URIError: URI malformed
```

<br>

### **3、try{} catch{}**

若 try 代码块报错，只能在 catch 中捕获<br>
但是 try 代码块中若有异步错误代码，catch 无法捕获，会被 onerror 捕获
```javascript
try {
    setTimeout(() => {
        console.log('a', a) // 可以被 onerror 捕获
    }, 1000)   
} catch(e) {
    console.log('e', e)
}
// Uncaught ReferenceError: a is not defined
```

<br>

### **4、Promise 抛出错误**

**a、没有设置 catch 捕获**
```javascript
let p = new Promise((resolve, reject) => {
    reject(1)
}) // 这里没有做 catch 处理
// Uncaught (in promise) 1
```

**b、在 catch 中报错没有捕获**
```javascript
;(async function xx() {
    try {
        throw 1
    } catch(e) {
        console.log('a', a)  // 这里出错可以使用 unhandledrejection 来捕获
    }
})()
// Uncaught ReferenceError: a is not defined
```

上面两种情况可以监听 unhandledrejection 捕获错误
```javascript
window.addEventListener('unhandledrejection', function(e) {
    // e.preventDefault(); // 阻止异常向上抛出
    console.log('捕获到异常 unhandledrejection ：', e)
})
```

<br>

### **5、静态资源加载失败**
**a、在资源上添加 onerror 事件**
```javascript
// html
// <img src="" alt="" id="imgID">

// js
let imgID = document.getElementById('imgID')
imgID.onerror  = function(e) {
    console.log('img load error :>> ', e);
}
// 注意：onerror 需要定义之后，再设置图片路径，才能捕获到加载失败
imgID.src = 'http://xxx.png'

```

**b、静态资源网络请求失败事件不会冒泡，需要在捕获阶段捕获**<br>
chrome、FF 中可以通过冒泡方式监听 error 事件捕获资源加载失败
```javascript
// 注意：此处会与上面的 onerror 事件一起触发，将导致日志重复上报
// 可以只使用 addEventListener 捕获模式统一监听, 就不需要注册 window.onerror 了
window.addEventListener(
    'error',
    error => {
        console.log('addEventListener 捕获到异常：', error)
    },
    true
)
```

<br>

### **6、网页崩溃**

**a、网页加载后，埋入一个标志，表示正在加载**
```javascript
// 初次进来，将埋入一个标志,值为 pending ，正常退出后，会设置为 true
// 若网页崩溃，第二次回来页面后，读取当前标志，如果值存在且为 true 则表示正常退出
// 如果不是 true ，则表示上次可能是崩溃了，需要上报之前定时更新的时间值
if(localStorage.getItem('good_exit') &&
    localStorage.getItem('good_exit') !== 'true') {
    // 日志上报，将崩溃之前的时间一起上报 localStorage.getItem('time_before_crash')
}
window.addEventListener('load', function () {
    localStorage.setItem('good_exit', 'pending');
    // 定时更新崩溃之前的网页时间
    setInterval(function () {
        localStorage.setItem('time_before_crash', new Date().toString());
    }, 10000);
})

window.addEventListener('beforeunload', function () {
    // 网页正常退出后，将埋入标志，设置成 true，表示正常退出
    localStorage.setItem('good_exit', 'true');
})
```

上面是在 `第二次` 进入页面才知道网页崩溃，那么有什么方法可以在网页崩溃之后就可以上报呢

**b、可以使用 Service Worker 来进行监控**<br>
其生命周期与页面无关（关联页面未关闭时，它也可以退出，没有关联页面时，它也可以启动）<br>
流程：
```javascript
1、注册 service worker js
2、
每间隔 10 秒，就向 worker 发送消息
消息中包含：
{
    type 字段：active 表示正常活跃，exit 表示正常退出
    time 字段：表示当前时间
}

在页面要退出后，发送 type: exit，表示正常退出
3、
worker 内部注册有消息接收事件，接收页面发送过来的消息
接收到 type 为 active，表示正常活跃，更新内部 time 的值
接收到 type 为 exit，表示正常退出，更新内部 time 值为 0
worker 内部维护一个状态对象，包含时间，值为页面发送过来
每隔 15 秒检查一次，若时间与上一次没有改变，则说明页面可能崩溃(注意区分 time 为 0 的情况)
```
<br>

### **7、Script Error**
跨域脚本的错误信息，因为处于保护信息的原因，只会展示 Script Error<br>
通过以下方式解决：
```javascript
1、外链脚本增加 crossorigin 属性：<script crossorigin src="http://other-domain.js"></script> 
2、同时脚本的 Access-Control-Allow-Origin: 设置为 * 或者 当前域名
```

<br>

### **8、iframe Error**

```javascript
<iframe src="./test.html"></iframe>
<script>
window.frames[0].onerror = function(msg, source, lineno, colno, error) {
    console.log('frames onerror :>> ', msg,source,lineno,colno,error)
}
</script>
```

<br>

### **9、vue 自身 try catch 处理了错误，导致我们无法捕获**

使用 Vue 提供的 errorHandler 方法捕获
```javascript
Vue.config.errorHandler = function(err, vm, info) {
    let {
        message, // 异常信息
        name, // 异常名称
        script, // 异常脚本url
        line, // 异常行号
        column, // 异常列号
        stack // 异常堆栈信息
    } = err
    // vm 为抛出异常的 Vue 实例
    // info 为 Vue 特定的错误信息，比如错误所在的生命周期钩子
    console.log('vue err，vm，info :>> ', err, vm, info)
}
```

<br>

### 二、捕获实践

所以总结下来：
```javascript
// 1、try catch 中的 catch 错误捕获
可以在 webpack 打包时候，使用 AST 方式解析并在 catch 中插入日志上报代码

// 2、error 事件
window.addEventListener(
    'error',
    error => {
        let data = {}
        // 此处与上面的 onerror 会重复事件
        let { colno, lineno, message, filename, error, stack } = error
        //不一定所有浏览器都支持 colno 参数
        let col = colno || (window.event && window.event.errorCharacter) || 0
 
        data.url = url
        data.line = line
        data.col = col

        if (!!stack){
            //如果浏览器有堆栈信息
            //直接使用
            data.msg = stack.toString()
        }else if (!!arguments.callee){
            //尝试通过callee拿堆栈信息
            let ext = []
            let f = arguments.callee.caller, c = 3
            //这里只拿三层堆栈信息
            while (f && (--c>0)) {
               ext.push(f.toString())
               if (f  === f.caller) {
                    break //如果有环
               }
               f = f.caller
            }
            ext = ext.join(",")
            data.msg = ext
        }
        console.log('3、addEventListener error 捕获阶段>>>异常：', data)
    },
    true
)

// 3、unhandledrejection
window.addEventListener('unhandledrejection', function(e) {
    // e.preventDefault(); // 阻止异常向上抛出
    console.log('4、promise 异常 unhandledrejection ：', e)
})

// 4、errorHandler
Vue.config.errorHandler = function(err, vm, info) {
    // vm 为抛出异常的 Vue 实例
    // info 为 Vue 特定的错误信息，比如错误所在的生命周期钩子
    let {
        message, // 异常信息
        name, // 异常名称
        script, // 异常脚本url
        line, // 异常行号
        column, // 异常列号
        stack // 异常堆栈信息
    } = err
    console.log('2、vue errorHandler :>> ', err, vm, info)
}
```