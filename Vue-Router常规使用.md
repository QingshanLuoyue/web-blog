## 背景
很多时候我们已经熟悉了框架的运用，但是有时候就是忘了怎么用<br>
所以这里想记下大部分的框架使用方法，方便使用的时候拷贝
<br>

注：大部分文案以及示例拷贝自 Vue-Router 官方文档
<br>

## 一、安装

### **npm 方式**
```bash
npm install vue-router --save
```
<br>

### **yarn 方式**
```bash
yarn add vue-router
```
<br>


## 二、功能

- 嵌套的路由/视图表

- 模块化的、基于组件的路由配置

- 路由参数、查询、通配符

- 基于 Vue.js 过渡系统的视图过渡效果

- 细粒度的导航控制

- 带有自动激活的 CSS class 的链接

- HTML5 历史模式或 hash 模式，在 IE9 中自动降级

- 自定义的滚动条行为

<br>

## 三、用法

### 使用注意点(非常！非常！非常！重要的点！！！！！！！)
**路由匹配有时候会匹配到多个路由**

**则匹配优先级为：先定义，先匹配**

**以 / 开头的嵌套路径会被当作根路径**

### **一、基本**

#### **1、引入以及安装**
```javascript
import Vue from 'vue'
import Router from 'vue-router'
Vue.use(Router)
```

<br>

#### **2、平常用法**

```javascript
// router.js
import Vue from 'vue'
import Router from 'vue-router'
Vue.use(Router)
export default new Router({
    routes: [
        {
            path: '/',
            name: 'dashboard',
            meta: { title: '首页', auth: true },
            component: () => import(`@/pages/project/views/dashborad/index.vue`)
        }
    ]
})

// ******************************************************

// main.js
import router from './router.js'
const app = new Vue({
    router
}).$mount('#app')
```

<br>

#### **3、嵌套路由**
```javascript
export default new Router({
    routes: [
        {
            path: '/',
            name: 'user',
            meta: { title: '用户', auth: true },
            component: () => import(`@/pages/project/views/user/index.vue`)
            children: [
                // 当路由没有匹配时候，可以设置一个 path 为 ''，来进行渲染
                {
                    path: '',
                    component: () => import(`@/pages/project/views/components/empty/index.vue`)
                },
                // this.$router.push('/user/zhangsan')
                {
                    path: 'zangsan',
                    name: 'zhangsan',
                    meta: { title: '用户信息', auth: true },
                    component: () => import(`@/pages/project/views/components/post/index.vue`)
                },
                // 以 / 开头的嵌套路径会被当作根路径
                // this.$router.push('/lisi')
                {
                    path: '/lisi',
                    name: 'lisi',
                    meta: { title: '用户信息', auth: true },
                    component: () => import(`@/pages/project/views/components/post/index.vue`)
                }
            ]
        }
    ]
})
```

<br>

#### **4、动态匹配**
```javascript
export default new Router({
    routes: [
        {
            // 匹配路径：
            // /User/zhangsan
            // /User/lisi
            path: '/User/:name',
            name: 'user',
            meta: { title: '用户信息页', auth: true },

            // 在 component 中，可以通过 `this.$route.params` 获取到动态参数 `:name` 的值

            // 如上面两个路径，取值方式为： 
            // /User/zhangsan
            // console.log(this.$route.params.name) // zhangsan

            // /User/lisi
            // console.log(this.$route.params.name) // lisi
            component: () => import(`@/pages/project/views/user/index.vue`)
        }
    ]
})
```

此种动态路由，在动态变化的时候，原来的组件实例会被复用，此时组件生命周期不会再被调用

可以使用以下两种方式实现监听

- 1、监听路由变化
```javascript
export default {
    watch: {
        '$route'(to, from) {
            // code
        }
    }
}
```

- 2、路由更新
```javascript
export default {
    // 在动态匹配路由场景下，复用的组件中才会生效
    beforeRouteUpdate(to, from, next) {
        // code
        // 注意，要记得使用 next()，否则程序无法执行下去
    }
}
```

<br>

#### **5、路由模式**
- 默认 hash 模式
```javascript
export default new Router({
    mode: 'hash', // 默认  http://test.com/#/user
    mode: 'history', // http://test.com/user
    routes: []
})
```

- history 模式，需要后端配合 

Nginx 相应配置
```bash
location / {
  try_files $uri $uri/ /index.html;
}
```

- 还需要配置 `匹配不到路由` 时候的情况
```javascript
export default new Router({
    routes: [
        {
            path: '*', // 匹配不到路由时，将会在这里匹配到，进而渲染
            component: NotFindComponent,
        }
    ]
})
```

<br>

#### **6、路由懒加载**
```javascript
export default new Router({
    routes: [
        {
            
            path: '/user/:id',
            component: import(/* webpackChunkName: "group-foo" */ './Foo.vue')
        }
    ]
})
```

特殊写法，使用 `命名 chunk` 将组件按组打包在一起
```javascript
import(/* webpackChunkName: "group-foo" */ './Foo.vue')
import(/* webpackChunkName: "group-foo" */ './Bar.vue')
import(/* webpackChunkName: "group-foo" */ './Baz.vue')
```
Foo、Bar、Baz 将被打包到 group-foo 文件中

<br>

#### **7、滚动行为**
```javascript
`注意: 这个功能只在支持 history.pushState 的浏览器中可用`
export default new Router({
    routes: [],
    scrollBehavior (to, from, savedPosition) {
        // return 期望滚动到哪个的位置
        return { x: Number, y: Number } // x 横向滚动 y 纵向滚动
        return savedPosition // 滚动到之前保存的位置
        return { 
            selector: string, 
            offset: { x: Number, y: Number } // offset 只在 2.6.0+ 支持
        }
        if (to.hash) {  // 滚动到锚点
            return {
                selector: to.hash
            }
        }
    }
})
```
`savedPosition` 当且仅当 popstate 导航 (通过浏览器的 前进/后退 按钮触发) 时才可用

<br>

#### **8、异步行为（2.8.0 新增）**
```javascript
export default new Router({
    routes: [],
    scrollBehavior (to, from, savedPosition) {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                resolve({ x: 0, y: 0 })
            }, 500)
        })
    }
})
```

<br>

### **二、进阶**

#### **1、匹配不到 404 路由**
```javascript
export default new Router({
    routes: [
        {
            // 会匹配所有路径
            // 注意要放在最后一条路由，保证前面都没有匹配到的时候，这里匹配到进而展示提示用户 404
            path: '*',
            name: 'NotFound',
            meta: { title: '404' },
            component: () => import(`@/pages/project/views/404/index.vue`)
        }
    ]
})
```

使用通配符后 `$route.params` 会自动添加一个 pathMatch 参数用来记录 URL 被匹配到的部分

```javascript
// 1、
{ path: '*' }
this.$router.push('/non-existing')
this.$route.params.pathMatch // /non-existing

// 2、
{ path: '/user/name-*' }
this.$router.push('/user/name-zhangsan')
this.$route.params.pathMatch // zhangsan

```

<br>

#### **2、重定向和别名**

- 2.1 重定向 (redirect)
```javascript
export default new Router({
    routes: [
        {
            
            path: '/b',
            redirect: '/c',
            redirect: {}, // 路由对象
            redirect: to => {
                // 接收目标路由对象 to 作为参数
                // return 字符串路径/一个路由对象
            }

        }
    ]
})
```


> redirect 与 导航守卫
```javascript

例子：

当前有三个路由 a、 b、 c

b 配置了重定向 c

当前路由为 a

1、从 a 跳转到 b，

2、因为 b 配置了重定向 c

3、所以最后跳转到 c

在这个过程中，a - b 的过程不会被导航守卫捕获

最后跳转到 c 的过程，会被导航守卫捕获
```

>备注官方说明：`注意导航守卫并没有应用在跳转路由上，而仅仅应用在其目标上`

```javascript
// a - b - c
router.beforeEach((to, from, next) => {
    console.log('to :', to) 
    // 打印 to：{ fullPath: b }, 
    // a - b - c 跳转过程，并不会打印跳转 b 过程
    // 没研究具体跳转逻辑，可能内部根本没跳转 b ，只做了判断，直接跳转 c
    next()
})
```

- 2.2 别名

>备注官方说明: `/a 的别名是 /b，意味着，当用户访问 /b 时，URL 会保持为 /b，但是路由匹配则为 /a，就像用户访问 /a 一样。`
```javascript
export default new Router({
    routes: [
        {
            
            path: '/a',
            alias: '/b'
        }
    ]
})
```

<br>

#### **3、路由组件传参**
- 目的是使组件与 $route 解耦

例如一个组件需要获取 `路由参数 $route.params.id`

那么就需要在组件的 template 模板中或者 script 中 `使用 $route 对象`

这样该组件无法在 `不使用路由或者不使用 $route` 的情况下复用

那么既然参数是从 $route.params 中取值，现在提供 `路由组件传参` 的方式代替他

- props 属性
  
路由对象增加一个 `props` 属性，将路由参数 `$route.params` 设置到需要复用组件的 `组件属性` 中

不用在组件中使用 `$route.params/$route.query`  ，最终实现组件复用


>备注官网说明：`请尽可能保持 props 函数为无状态的，因为它只会在路由发生变化时起作用。如果你需要状态来定义 props，请使用包装组件，这样 Vue 才可以对状态变化做出反应`
```javascript
export default new Router({
    routes: [
        {
            
            path: '/user/:id',
            component: User,
            // 布尔模式
            // 在组件中可以直接使用 this.id
            props: true // true: 将路由对象中 params 参数 id 设置到组件属性中 
            // 对象模式
            // { mode: 'static', id: 1 } 对象会被按原样设置为组件属性. 当 props 是静态的时候有用
            // 在组件中可以直接使用 this.mode \ this.id
            props: { mode: 'static', id: 1 } 
            // 函数模式
            // 在组件中可以直接使用 this.params \ this.query
            props: (route) => ({ 
                    params: route.params.id, 
                    query: route.query.name 
                }) 
            }
        }
    ]
})
```

<br>

#### **4、路由动效**
- 使用 transition 配合 router-view 组件实现动效

动态的或者静态的 动画/过度 name
```html
<transition :name="transitionName">
  <router-view></router-view>
</transition>
```

```css
/* 进入状态变化：

1、元素插入前： 
.transitionName-enter 生效，给定元素一个初始状态
.transitionName-enter-active 生效，设置过度状态的时间

2、元素插入后，动画/过度变化状态
.transitionName-enter 被删除
.transitionName-enter-active 生效，过度时间持续生效
.transitionName-enter-to 生效，设置过度的目标状态

3、动画/过度结束状态
.transitionName-enter-active 被删除
.transitionName-enter-to 被删除


备注：
.transitionName-enter，.transitionName-enter-active 一开始觉得两者作用有点重复，
但是 .transitionName-enter 的使命是给与元素一个初始状态，元素被插入之后，该类名就会被删除；
如果用 .transitionName-enter-active 来给定元素初始状态，
在动画/过度过程中，.transitionName-enter-active 没有被删除，会一直存在，持续生效；
这时 .transitionName-enter-to 生效，设置元素过度的目标状态；
那么要保证 .transitionName-enter-to 的作用优先级要比 .transitionName-enter-active 大才行。

为了明确各自的作用，建议还是使用 三种类名，来区分生效作用 */

/* 1、元素插入前状态 */
.transitionName-enter {
  opacity: 0;
}
/* 2、元素插入动画/过度状态 */
.transitionName-enter-active, {
  transition: opacity .5s;
}
/* 3、元素插入完成时状态 2.1.8+版本 */
.transitionName-enter-to, {
  opacity: 1;
}


/* 
离开状态变化

1、元素移除前： 
.transitionName-leave 生效，给定元素一个初始状态
.transitionName-leave-active 生效，设置过度状态的时间

2、元素移除前，动画/过度变化状态
.transitionName-leave 被删除
.transitionName-leave-active 生效，过度时间持续生效
.transitionName-leave-to 生效，设置过度的目标状态

3、动画/过度结束状态，元素被移除
.transitionName-leave-active 被删除
.transitionName-leave-to 被删除 */

/* 元素移除前状态 */
.transitionName-leave {
  opacity: 1;
}
/* 元素移除动画/过度状态 */
.transitionName-leave-active {
  transition: opacity .5s;
}
/* 元素移除完成时状态 2.1.8+版本 */
.transitionName-leave-to, {
  opacity: 0;
}
```

<br>

## 四、组件内常用操作

### 一、this.$router

- `this.$router.push/this.$router.replace`

`对应 window.history.pushState/window.history.replaceState`
  
```javascript
// 字符串
this.$router.push('/user')      // /user

// 对象
this.$router.push({             // /user
    path: '/user'
})

// 命名路由
this.$router.push({             // /user/123
    name: 'user',
    params: {
        userId: '123'
    }
})

// 带查询参数
this.$router.push({             // /user?userId=123
    path: '/user',
    query: {
        userId: '123'
    }
})

// 带查询参数,
// 若存在 path，则 params 会失效 
this.$router.push({             // /user
    path: '/user',
    params: { // 不起作用
        userId: '123'
    }
})
```
`replace` 与 `push` 的操作一致

`push` 会向浏览器 history `添加一条记录，replace` 不会添加，会直接替换当前记录

- 两个可选参数

`push` 与 `replace` 还支持其他两个可选参数
```javascript
this.$router.push(location, onComplete?, onAbort?)
this.$router.replace(location, onComplete?, onAbort?)
```
onComplete 在导航完成时执行（所有异步钩子被解析）

onAbort 在导航终止时执行（导航到相同路由、在导航完成之前导航到另外一个不同的路由）

- 跳转建议：
```javascript
this.$router.push({             // /user/123
    name: 'user',
    params: {
        userId: '123'
    }
})
```
建议使用 name 跳转，因为 name 不会变，但是路径可能会变

路径变了，若我们使用 path 跳转，那么我们的代码也要修改

但是如果我们使用 name 进行跳转，那么代码就无需改动

- go/back

`对应 window.history.go/window.history.back`

```javascript
// 前进
this.$router.go()
this.$router.go(1)
this.$router.go(-1)  === this.$router.back()

// 后退
this.$router.back()
this.$router.back(1)
this.$router.back(-1) === this.$router.go()

// history 记录不够用，则失败，不做任何操作
this.$router.go(100)
this.$router.back(100)
```

- 完整的导航解析流程

>导航被触发。

>在失活的组件里调用离开守卫。

>调用全局的 beforeEach 守卫。

>在重用的组件里调用 beforeRouteUpdate 守卫 (2.2+)。

>在路由配置里调用 beforeEnter。

>解析异步路由组件。

>在被激活的组件里调用 beforeRouteEnter。

>调用全局的 beforeResolve 守卫 (2.5+)。

>导航被确认。

>调用全局的 afterEach 钩子。

>触发 DOM 更新。

>用创建好的实例调用 beforeRouteEnter 守卫中传给 next 的回调函数。

<br>

### 二、this.$route

- this.$route.params
```javascript
this.$router.push({             // /user/123
    name: 'user',
    params: {
        userId: '123'
    }
})
```

- this.$route.query

以下例子若不刷新跳转，下一个路由接收到的 userId 是个数值类型： `123`

如果跳转过去后，刷新页面，那么 userId 将变成 字符串类型： `'123'`
```javascript
// 带查询参数
this.$router.push({             // /user?userId=123
    path: '/user',
    query: {
        userId: 123
    }
})
```

- 监听路由变化，多用于动态路由匹配，解决页面复用，导致生命周期不执行的问题

```javascript
export default {
    watch: {
        '$route'(to, from) {
            // code
        }
    }
}
```

<br>

### 三、beforeRouteEnter
使用 beforeRouteEnter 实现获取数据后，再渲染dom

```javascript
beforeRouteEnter(to, from, next) {
    // 在当前无法使用组件实例
    // 获取详情
    getDetail(to.query.id - 0)
        .then(res => {
            console.log('beforeRouteEnter>>>then :', res)
            next(vm => {
                // 这里可以使用组件实例 vm
                vm.setDetail(res, vm)
            })
        })
        .catch(e => {
            console.log('beforeRouteEnter>>>error :', e)
        })
},
```

<br>

### 四、beforeRouteUpdate
```javascript
export default {
    // 在动态匹配路由场景下，复用的组件中才会生效
    beforeRouteUpdate(to, from, next) {
        // code
        // 注意，要记得使用 next()，否则程序无法执行下去
    }
}
```

<br>

### 五、router-link 、router-view 

#### **1、router-link**
使用 router-link 导航

简单用法使用 `to` 属性来指定路径
```html
<router-link to="/dashboard"></router-link>
```

复杂用法使用 `to` 绑定一个 `配置对象`
```html
<router-link :to="{ path: '/dashboard', query: { id: 2}}"></router-link>
```

默认会被渲染为一个 `a` 标签，可以使用 `tag` 属性来指定渲染的标签类型
```html
<router-link :to="{ path: '/dashboard', query: { id: 2}}" tag="div"></router-link>
```

匹配到路由后，渲染后的标签将会添加类名 `router-link-active`

<br>

#### **2、router-view**

使用 router-view 来给配置的路由组件占位

匹配到路由后，会被替换掉，其实是一个占位标签

`路由对象`
```javascript
// router
export default new Router({
    routes: [
        {
            path: '/',
            // 注意区分 component 和  components

            // component: commonComponent // 匹配单一组件

            components: { // 匹配多组件
                default: defaultComponent, // router-view
                a: aComponent, // router-view name="a"
                b: bComponent, // router-view name="b"
            }
        }
    ]
})
```

`html 结构`

```html
<div id="app">
    <!-- 渲染 defaultComponent -->
    <router-view class="view one"></router-view>
    <!-- 渲染 aComponent -->
    <router-view class="view two" name="a"></router-view>
    <!-- 渲染 bComponent -->
    <router-view class="view three" name="b"></router-view>
</div>
```

同个组件，多个 router-view，可以通过指定`具名组件`，来渲染对应的 router-view

匹配多个组件，`路由对象` 中使用 `components` 对象来存储，而不是 `component`


<br>

没有 `name` 属性，默认渲染-->匹配到的`路由对象`中 `components` 对象中-->属性名为 `default` 的组件
```html
<!-- template -->
<router-view class="view one"></router-view>
```

`name` 属性为 `a`，渲染-->匹配到的路由对象中 `components` 对象中-->属性名为 `a` 的组件
```html
<router-view class="view two" name="a"></router-view>
```

`name` 属性为 `b`，渲染-->匹配到的路由对象中 `components` 对象中-->属性名为 `b` 的组件
```html
<router-view class="view three" name="b"></router-view>
```