## JSBridge 原理与封装

移动应用功能越来越丰富，部分功能需要以 H5 方式来实现<br>
这就需要 H5 与 Native 进行交互，我们使用 JSBridge 来进行二者通信

JSBridge 定义 Native 和 H5 的通信方式
>Native 通过协定的桥对象调用 H5<br>
H5 通过协定的桥对象调用 Native

<br>

### 一、交互方式
#### **1、伪协议 url scheme（适用于所有设备）**

伪协议 与 url 类似，H5 触发伪协议，系统按优先级判断：
1. 是否是系统应用，是则打开系统应用
2. 是否有 app 注册当前伪协议，有则打开该 app
3. native 触发 url 事件，捕获该伪协议，解析伪协议，调用方法

**具体过程：**<br>
1. H5 调用 Native<br>
触发伪协议，例如：<br>
window.location.href= 'xxx://event=a&data=b&successName=c&failName=d'

2. Native 调用 H5<br>
H5 在 window 注册方法供 native 调用(同 API 交互方式中 Native 调用 H5 一样)

<br>

#### **2、通过 API 交互**
**具体过程：**<br>
1. H5 调用 Native<br>
    1.1、 H5 调用 Android： native 通过 addJavascriptInterface 注册，之后供 H5 调用<br>
    1.2、 H5 调用 iOS：native 通过 javascriptCore（需 iOS7 以上） 注册，之后供 H5 调用

2. Native 调用 H5<br>
    2.1、 Android 调用 H5：H5 在 window 上注册方法，native 通过 loadUrl 调用 H5，4.4及以上版本可以通过 evaluateJavascript 调用<br>
    2.2、 iOS 调用 H5：H5 在 window 上注册方法，native 通过 stringByEvaluatingJavascriptFromString 调用 H5

<br>

#### **3、区别**
区别在于 `H5 如何调用 Native` 以及 `Native 响应 H5 当前调用指定方法的方式` 不一样

**3.1 H5 如何调用 Native：**<br>
伪协议： <br>
```javascript
window.location.href = 'xxx://event=a&data=b&successName=c&failName=d'
```
API: 
```javascript
window.JSActionBridge.handlerAction(
    event,
    data,
    successName,
    failName
)
window.webkit.messageHandlers.JSActionBridge.postMessage({
    method: 'handlerAction',
    data: {
        actionEvent: event,
        paramsJsonValue: data,
        successCallback: successName,
        errorCallback: failName
    }
})
```

<br>

**3.2 Native 响应 H5 就不列出了**

<br>

**3.3 API 交互方式的好处**：<br>
- 在 H5 中写起来更简单，不用创建 URL 的方式<br>
- H5 传递参数更方便，使用拦截 URL 的方式，参数需要拼接到 url 后面，如果含有特殊字符，则需要转义，否则解析参数会出错，如 & = ?
- 例如
window.location.href = 'xxx://event=a&data=http://www.baidu.com/xxx.html?p=2&successName=c&failName=d'<br>
这里解析参数得到的不是想要的值，需要对 `http://www.baidu.com/xxx.html?p=2` 进行转义

<br>

### 二、方案

目前公司内部使用 API 方式交互（Android 需要 4.2 以上，iOS 需要 7 以上）

### **1、约定名称**
```javascript
原生通过 JSActionBridge 注册 JSBridge 对象
H5 通过 JSActionBridge 获取 JSBridge 对象
```

### **2、原生 JSBridge 对象注册**
Android： 
```javascript
webView.addJavascriptInterface(new JSActionBridge(), "JSActionBridge")
```

iOS：
```javascript
// 通过 javascriptCore 注册
self.context = [[JSContext alloc] int];
self.context[@"JSActionBridge"] = self.bridgeobj;
```

### **3、调用**
**3.1 Native 方法定义**
```javascript
Android：
/*
* @params {String} actionEvent: 需要调用的原生事件名、方法名
* @params {String|Object|Array} paramsJsonValue： 传给原生的参数，最好约定为 json 格式
* @params {String} successCallBack：调用原生方法成功后，需要执行的的回调函数名，该函数定义在 window 上
* @params {String} errorCallback：调用原生方法失败后，需要执行的的回调函数名，该函数定义在 window 上
*/
void handlerAction(String actionEvent, String paramsJsonValue, String successCallBack, String errorCallback)
```

**3.2 H5 调用 Native**
```javascript
JSActionBridge.handlerAction(actionEvent, paramsJsonValue, successCallBack, errorCallback)
```

**3.3 Native 调用 H5**

Android 调用 H5
```javascript
String callbackUrl = "javascript:" + callback + "(\"" + jsonData + "\")";
webView.loadUrl(callbackUrl);
```
iOS 调用 H5
```javascript
[webView stringByEvaluatingJavaScriptFromString:[NSString stringWithFormat: @"callBack(%@)", jsonData]];
```

<br>

### 三、数据约定

>原生方法定义
```javascript
void handlerAction(String actionEvent, String paramsJsonValue, String successCallBack, String errorCallback)
```
**1、成功或失败回调函数都返回 json 数据**<br>
数据格式:
```javascript
{
    "code":0, 
    "desc":"success", 
    "data": nativeReturnData
}
```

2、**约定 code === 0 时表示调用成功，否则失败**<br>

3、**actionEvent 归类**<br>
- 跳转原生页面<br>
    首页、登录注册页等<br>
- 获取原生数据<br>
    获取用户信息、获取设备信息等<br>
- 调用原生功能<br>
    跳转新页面、唤醒密码输入框、查看网络状态、设置标题等<br>
- 其他

<br>

### 四、JSBridge 封装

1、协定：
- Android 传送字符串
- iOS 传送 json

<br>

2、successCallBack/errorCallback 回调数据格式：
```javascript
{
    "code":0, 
    "desc":"success", 
    "data": nativeData
}
```

<br>

3、JSBridge 封装
```javascript
import { isIOS, isAndroid, isApp } from './utils.js'
const JSBridge = {
    isApp,

    // APP离线环境
    isOfflineApp: isApp && location.protocol === 'file:',

    // H5 调用 App
    /**
     * 调用 App 方法
     * @param event：事件名称
     * @param data：json 数据
     * @param successCallBack: 成功回调
     * @param failCallBack: 失败回调
     * @returns {Promise<object>}
     */
    callApp (event, data = {}) {
        return new Promise((resolve, reject) => {
            // 不在 App 内调用，提示不在 App 内
            if (!this.isApp) {
                resolve('not in app')
                return
            }

            // 构造唯一 key
            const callbackKey =
                Date.now() + '' + Math.floor(Math.random() * 100000)
            const successName = `s${callbackKey}`
            const failName = `f${callbackKey}`
            // 使用构造好的 `成功\失败的 key` 在 window 上注册一个函数，使得 App 可以调用
            this.registerFn(successName, function(data) {
                if (data.code === -1) {
                    // 失败则 reject
                    reject(data)
                } else {
                    // 成功则 resolve
                    resolve(data.data)
                }
            })
            this.registerFn(failName, function(err) {
                reject(err)
            })

            // ******开始调用 App 提供的方法******
            // 安卓
            if (isAndroid) {
                // 安卓需要将数据字符串化
                data = JSON.stringify(data)
                window.JSActionBridge.handlerAction(
                    event,
                    data,
                    successName,
                    failName
                )
            }

            // iOS
            if (isIOS) {
                window.webkit.messageHandlers.JSActionBridge.postMessage({
                    method: 'handlerAction',
                    data: {
                        actionEvent: event,
                        paramsJsonValue: data,
                        successCallback: successName,
                        errorCallback: failName
                    }
                })
            }
        })
    },

    /**
     * 注册 H5 供 app 调用的方法
     * @param fnName
     * @param fn
     */
    registerFn (fnName, fn) {
        if (typeof fnName !== 'string') {
            throw TypeError('Illegal fnName: Not an string')
        }
        if (typeof fn !== 'function') {
            throw TypeError('ol. fn: Not an function')
        }

        window[fnName] = function (data) {
            if (isIOS) {
                fn(data)
            }
            if (isAndroid) {
                // 安卓环境需要做转换
                data = data || '{}'
                fn(JSON.parse(data))
            }
        }
    },

    /**
     * 注销 H5 供 app 调用的方法
     * @param fnName
     */
    unregisterFn (fnName) {
        if (typeof fnName !== 'string') {
            throw TypeError('Illegal fnName: Not an string')
        }

        delete window[fnName]
    },

    /**
     * 跳转至app模块
     * @param url
     * @param isWaitingResult
     * @returns {*|Promise<Object>}
     */
    gotoNativeModule (url, isWaitingResult = false) {
        if (this.isApp) {
            this.callApp('goto_native_module', {
                url,
                isWaitingResult
            })
        }
    },

    /**
     * 在APP内新打开
     * @param url
     * @param titleBarVisible
     * @param title
     */
    gotoNewWebview (url, titleBarVisible = true, title = '') {
        if (this.isApp) {
            this.gotoNativeModule(
                `xxx://webview?url=${encodeURIComponent(
                    url
                )}&titleBarVisible=${titleBarVisible}&title=${title}`
            )
        } else {
            window.location.href = url
        }
    },

    /**
     * 监控页面活动状态
     * @param activated
     * @param deactivated
     */
    watchPageActivity (activated, deactivated) {
        const successCallBackName =
            'command_watch_activity_status_success_callback'
        const failCallBackName = 'command_watch_activity_status_fail_callback'
        const successCallBack = window[successCallBackName]
        const failCallBack = window[successCallBackName]

        this.registerFn(successCallBackName, function (data) {
            if (
                Object.prototype.toString.apply(successCallBack) ===
                '[object Function]'
            ) {
                successCallBack(data)
            }

            try {
                if (typeof data === 'string') {
                    data = JSON.parse(data)
                }

                if (data.data.status === 'visible') {
                    activated(data)
                } else {
                    deactivated(data)
                }
            } catch (e) {
                console.log(successCallBackName, e)
            }
        })

        this.registerFn(failCallBackName, function (data) {
            if (
                Object.prototype.toString.apply(failCallBack) ===
                '[object Function]'
            ) { 
                failCallBack(data) 
            }
        })

        this.callAppNoPromise(
            'command_watch_activity_status',
            {},
            successCallBackName,
            failCallBackName
        )
    },
    /**
     * 调用app方法 注册全局事件
     * @param event：事件名称
     * @param data：json数据
     * @param successCallBack: 主动设置成功回调
     * @param failCallBack: 主动设置失败回调
     * @returns {Promise<object>}
     */
    callAppNoPromise (event, data = {}, successCallBackName, failCallBackName) {
        if (!this.isApp) {
            return
        }
        // 安卓
        if (isAndroid) {
            data = JSON.stringify(data)
            window.JSActionBridge.handlerAction(
                event,
                data,
                successCallBackName,
                failCallBackName
            )
        }
        // ios
        if (isIOS) {
            window.webkit.messageHandlers.JSActionBridge.postMessage({
                method: 'handlerAction',
                data: {
                    actionEvent: event,
                    paramsJsonValue: data,
                    successCallback: successCallBackName,
                    errorCallback: failCallBackName
                }
            })
        }
    },

    /**
     * 获取app用户信息，防止多次获取
     * @param needUpdate 是否需要再次更新
     * @returns {*|Promise<Object>}
     */
    _appUser: {
        _loaded: false, // 是否从APP加载过数据，该字段不是从app中获取
        userName: '',
        phoneNum: '',
        userId: '',
        userToken: '', // 用户token，没有则代表未登录
    },
    getAppUser (needUpdate) {
        return new Promise(resolve => {
            if (!this.isApp) {
                resolve(this._appUser)
            }
            if (needUpdate || !this._appUser._loaded) {
                this.callApp('get_user_info').then(res => {
                    this._appUser = res
                    this._appUser._loaded = true
                    resolve(this._appUser)
                })
            } else {
                resolve(this._appUser)
            }
        })
    }
}
export default JSBridge
```