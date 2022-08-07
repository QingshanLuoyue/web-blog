### mockjs 使用笔记
### 一、背景
1. 前端开发需要依赖后端接口
2. 后端接口输出慢、接口规范随时可能会变，而前端毫无感知
3. 前端需要自己 mock 假数据 json 文件
4. 假数据 json 数据内容是静态的，测试不同返回情况需要修改 json 文件
   ...

因此我们需要一种可以帮我们构造数据的工具，并解决以上的若干痛点
mock.js 是一个不错的工具

### 二、安装与使用
安装

```bash
yarn add mockjs -D
```

使用

```javascript
// 使用 Mock
import Mock from 'mockjs'
Mock.mock('http://test.com/getjson.json', 'get', {
    // 属性 list 的值是一个数组，其中含有 1 到 10 个元素
    'list|1-10': [{
        // 属性 id 是一个自增数，起始值为 1，每次增 1
        'id|+1': 1
    }]
})
```

### 三、常用构造指令
以下示例可以在
http://mockjs.com/examples.html
网页上打开控制台

使用`Mock.mock({示例代码})`查看生成的结果

1、时间戳：

`'name|1564577990837-2564577990837': 0`

2、id

`'name|1-123456789': 0` // 6174430

3、指定长度范围，随机中文字符

`name: '@cword(2, 6)'` // 价亲三身千然

`'name|2-6': '@cword(1)'`

4、指定长度范围，随机英文字符

`name: '@word(1, 10)'` // yyuj

`'name|1-10': '@word(1)'`

5、http url

`name: '@url(http)'` //  "http://djldfusj.in/ccnknb"

6、生成布尔值，false/true各一半几率

`'name|1': true`

7、包含小数数字

`'name|1-100.3': 1` //  63.895

8、随机选择数组中的元素

`'name|1': ['AMD', ''CMD, 'UMD']` // 随机出现数组中的元素

9、万能的正则

（1）时间戳

   ` name: /170\d{10}/` // 1705237332101

（2）小数数字

   `name: /\d{2}\.\d{3}/` // 78.635

（3）id

   `name: /\d{1,9}/` // 651866094

（4）英文字符串

   `name: /\W{1,3}/` // DvX

（5）中文字符串

   `name: /[\u4e00-\u9fa5]{1,9}/` // 风我齤刘杌菦炧荷岆

### 四、从零开始完成一个使用例子

#### #1、创建环境

1. `yarn global add @vue/cli` 
   1. 安装 vuecli3.x 环境
2. `vue create mock-demo` ---  创建项目 `mock-demo`
   1. 选择 default (babel, eslint)
   2. 选择 Use Yarn
3. 安装依赖
   1. 安装 axios 
      1. `yarn add axios`
   2. 安装 mockjs 
      1. `yarn add mockjs -D`
4. 运行项目 `yarn serve`
   1. 按照命令中显示的项目地址，用浏览器打开

#### #2、封装 axios

1. 在 src 下创建 utils 目录，在 utils 下创建 fetch.js 文件
```javascript
// fetch.js
import axios from 'axios'
export default class baseRequest {
    constructor(baseURL) {
        baseURL = baseURL || window.location.origin
        this.$http = axios.create({
            timeout: 30000,
            baseURL
        })
        this.$http.interceptors.request.use(async config => {
            return config
        })
        this.$http.interceptors.response.use(
            ({ data, config }) => {
                if (data.code === 0) {
                    return data
                } else {
                    return Promise.reject(data)
                }
            },
            e => {
                console.log(e, '报错了')
                return Promise.reject({
                    msg: '网络开小差了,请稍后重试'
                })
            }
        )
    }
    post(url, params = {}, config = {}) {
        return this.$http.post(url, params, config)
    }
    get(url, params = {}, config = {}) {
        return this.$http.get(url, {
            params,
            ...config
        })
    }
}

```

#### #3、封装 api 接口
1. 在 src 下创建 service 目录，在 service 下创建 bilibili-server.js 文件
```javascript
// bilibili-server.js
// 在本例子中，我们将获取 bilibili 的最近更新番剧接口作为例子
import fetch from '@/utils/fetch.js'
let axios = new fetch()
export const getLastestAnima = () => {
    return axios.get('/api/timeline_v2_global')
}
```

#### #4、调用获取番剧接口
1. 在 src 下的 App.vue 文件中，created 生命周期中调用
```javascript
// App.vue
import HelloWorld from './components/HelloWorld.vue'
import { getLastestAnima } from '@/service/bilibili-server.js'
export default {
    name: 'app',
    components: {
        HelloWorld
    },
    async created() {
        try {
            let data = await getLastestAnima()
            console.log('getLastestAnima:data:>>>', data)
        } catch (error) {
            console.log('getLastestAnima:error:>>>', error)
        }
    }
}
```

#### #5、设置代理
1. 因为跨域限制，所以需要本地设置代理，才能访问 bilibili 的接口
2. 在项目根目录下创建 vue.config.js 文件
```javascript
// vue.config.js
module.exports = {
    devServer: {
        disableHostCheck: true,
        open: true, // 是否打开页面
        host: '0.0.0.0',
        port: 80,
        https: false,
        hotOnly: true,
        proxy: {
            '/api': {
                target: 'https://bangumi.bilibili.com',
                changeOrigin: true
            } 
        }
    }
}
```
3. 重启项目，然后打开项目，在浏览器控制台可以看到已经将获取到的数据打印出来了

#### #6、加入 mockjs
1. 在 src 下 创建 mock 目录
2. 在 mock 下创建 index.js 文件，用来注册各个 mock 接口
```javascript
// /src/mock/index.js
// 使用 Mock
import Mock from 'mockjs'
import modules from './modules'

// 注册每个 mock api
for (let key in modules) {
    let module = modules[key]
    // console.log('module:>>>', module)
    Mock.mock(...module)
}
```
3. 在 mock 下创建 modules，用来分类放置各个微服务相对应的 mock 接口

目录构成
```javascript
modules
    bilibili-server // 微服务
        get-lastest-anima.js // mock 接口
    index.js // 用来导出每个微服务下的所有 mock 接口
```
```javascript
// /src/mock/modules/bilibili-server/get-lastest-anima.js.js

/* eslint-disable no-console */
import { host } from "../../utils/host"
import { formatMockData } from "../../utils/util"
import Mock from "mockjs"
const formatData = formatMockData("", "", "", {
    code: 0,
    'result|0-100': [
        {
            'area|1': ["日本", '美国', '国产'],
            'arealimit|0-923456789': 0,
            'attention|0-923456789': 0,
            'bangumi_id|0-923456789': 0,
            'bgmcount|1': [/\d{1,6}/, 'SP'],
            cover: '@url(http)',
            'danmaku_count|0-923456789': 0,
            'ep_id|-1-923456789': 0,
            'favorites|0-923456789': 0,
            'is_finish|1': [0, 1],
            'lastupdate|0-1966120600': 0,
            'lastupdate_at': '@datetime',
            'new|1': true,
            'play_count|0-923456789': 0,
            pub_time: '@datetime',
            'season_id|0-9123456789': 5978,
            'season_status|0-9123456789': 13,
            'spid|0-9123456789': 0,
            square_cover: '@url(http)',
            title: "@cword(1,20)",
            'viewRank|0-912345678': 0,
            'weekday|1': [0, 1, 2, 3, 4, 5, 6]
        }
    ],
    message: '@cword(1,10)'
})
let url = host + "/api/timeline_v2_global"
let method = "get"
export default [
    url,
    method,
    function(options) {
        console.log("options:>>>", options)
        return Mock.mock(formatData)
    },
    {
        url,
        method,
        formatData
    }
]

```

```javascript
// /src/mock/modules/index.js
import getLastestAnima from './bilibili-server/get-lastest-anima'
export default {
    getLastestAnima
}
```

4. 在 mock 目录下创建 utils，用来放置工具类
```javascript
utils
    host.js
    util.js
```
```javascript
// /src/mock/utils/host.js
let h = window.location.origin
export const host = h || ''
```

```javascript
// /src/mock/utils/util.js
export const formatMockData = (
    mockData = {},
    codeStatus = 0,
    msg = '@cword(1,10)',
    customData
) => {
    if (customData) return customData
    let initFormatObj = {
        code: codeStatus,
        result: mockData,
        msg: msg
    }
    return initFormatObj
}

// 组合数据
export const comp = function(value) {
    return [null, '', value]
}

```

5. 最后在项目的 main.js 中引入 mock 入口文件
```javascript
// /src/main.js
import Vue from 'vue'
import App from './App.vue'

Vue.config.productionTip = false

// 开发环境使用，生产环境注释
import '@/mock'

new Vue({
  render: h => h(App),
}).$mount('#app')

```

#### #7、反向校验后端 api 返回数据
1. 在 mock 目录下的 utils 目录下 创建 proxy-valid.js 文件
```javascript
// 反向校验后端 api 配置文件
/* eslint-disable no-console */
import Mock from 'mockjs'
import modules from '../modules'

export default function(url, method, data) {
    Object.keys(modules).forEach(key => {
        let ele = modules[key][3]
        if (ele && ele.url === url && ele.method === method) {
            let validResult = Mock.valid(ele.formatData, data)
            // 验证通过，则不用输出信息
            if (validResult && validResult.length === 0) {
                return
            }
            console.group(url.replace(/http:\/\//, ''))
            console.log('valid response data :>>> url: ', url)
            console.log('valid response data :>>> validMsg: ', validResult)
            console.groupEnd()
        }
    })
}
```
2. 修改 /src/utils/fetch.js
```javascript
/* eslint-disable no-console */
import axios from 'axios'
// 开发环境使用，打包前注意要注释
import proxyValid from '@/mock/utils/proxy-valid.js'
export default class baseRequest {
    constructor(baseURL) {
        baseURL = baseURL || window.location.origin
        this.$http = axios.create({
            timeout: 30000,
            baseURL
        })
        this.$http.interceptors.request.use(async config => {
            return config
        })
        this.$http.interceptors.response.use(
            ({ data, config }) => {
                // 开发环境需要，生产环境需要注释
                proxyValid(config.url, config.method, data)
                if (data.code === 0) {
                    return data
                } else {
                    return Promise.reject(data)
                }
            },
            e => {
                console.log(e, '报错了')
                return Promise.reject({
                    msg: '网络开小差了,请稍后重试'
                })
            }
        )
    }
    post(url, params = {}, config = {}) {
        return this.$http.post(url, params, config)
    }
    get(url, params = {}, config = {}) {
        return this.$http.get(url, {
            params,
            ...config
        })
    }
}
```
此时，一个完整的 数据 mock 和反向校验 demo 就完成了


### 五、根据请求的数据修改 mock 已经生成的数据
根据前端请求的参数，修改 mock 渲染好的数据，然后返回给前端
这样实现根据不同的参数返回：
    成功状态
    失败状态
    其他异常状态

```javascript
/* eslint-disable no-console */
import { host } from "../../utils/host"
import { formatMockData } from "../../utils/util"
import Mock from "mockjs"
const formatData = formatMockData("", "", "", {
    code: 0,
    'result|0-100': [
        {
            'area|1': ["日本", '美国', '国产'],
            'arealimit|0-923456789': 0,
            'attention|0-923456789': 0,
            'bangumi_id|0-923456789': 0,
            'bgmcount|1': [/\d{1,6}/, 'SP'],
            cover: '@url(http)',
            'danmaku_count|0-923456789': 0,
            'ep_id|-1-923456789': 0,
            'favorites|0-923456789': 0,
            'is_finish|1': [0, 1],
            'lastupdate|0-1966120600': 0,
            'lastupdate_at': '@datetime', // bilibili 这边返回的接口中，有时候这个属性会少，导致反向校验出错
            'new|1': true,
            'play_count|0-923456789': 0,
            pub_time: '@datetime',
            'season_id|0-9123456789': 5978,
            'season_status|0-9123456789': 13,
            'spid|0-9123456789': 0,
            square_cover: '@url(http)',
            title: "@cword(1,20)",
            'viewRank|0-912345678': 0,
            'weekday|1': [0, 1, 2, 3, 4, 5, 6]
        }
    ],
    message: '@cword(1,10)'
})
let url = host + "/api/timeline_v2_global"
let method = "get"
export default [
    url,
    method,
    function(options) {
        console.log("options:>>>", options)
        // options:
            // body: null
            // type: "GET"
            // url: "http://localhost:1385/api/timeline_v2_global"
        // 此处 body 就是前端请求的参数集合
        // 在此处可以根据 body 里面的参数，来修改 Mock.mock(formatData) 渲染后的数据
        // 比如 修改 code = 404 
        // 设置各种成功、异常状态
        return Mock.mock(formatData)
    },
    {
        url,
        method,
        formatData
    }
]
```

### 六、yApi
 1. 安装
 2. 使用
 3. 创建第一个api
 4. 调用
 5. 前后联调修改一致性