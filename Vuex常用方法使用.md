## 背景
很多时候我们已经熟悉了框架的运用，但是有时候就是忘了怎么用<br>
所以这里想记下大部分的框架使用方法，方便使用的时候拷贝
<br>

## 一、安装

### **npm 方式**
```bash
npm install vuex --save
```
<br>

### **yarn 方式**
```bash
yarn add vuex
```
<br>

## 二、vuex 用法

### **1、引入以及安装**
```javascript
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)
const store = new Vuex.Store({
    state: {},
    getters: {},
    mutations: {},
    actions: {}
})
new Vue({
    // 把 store 提供给 store 选项，
    // Vue 最初实例化的时候将 store 的实例注入到所有的子组件，意味着子组件可以使用 this.$store 调用 store 实例
    store 
})
```
<br>

### **2、正常使用**
（1）、state<br>
（2）、getters<br>
（3）、mutations<br>
（4）、actions

```javascript
const store = new Vuex.Store({
    state: {
        count: 0,
        todos: [
            { id: 1, done: true },
            { id: 2, done: false }
        ]
    },
    // 是 state 的 computed
    getters: {
        doneTodos: (state, getters) => {
            return state.todos.filter(todo => todo.done)
        },
        doneTodoCount: (state, getters) => {
            return getters.doneTodos.length
        }
    },
    // mutation 必须是同步函数，以便追踪状态（编写规范如此，非强制）
    // 可以使用 state.obj = { ...state.obj, newProp: 1 } 进行值的替换（新对象替换旧对象，支持增加新属性）
    mutations: {      
        mutationChangeState__increment(state, payload) {
            // payload： 调用该方法时，传递进来的参数
            state.count++
        }
    },
    // mutation 里不允许的异步操作，可以放在 action 里面
    actions: {
        // 1、普通操作
        actionCallMutation__mutationChangeState__increment(context) {
            // context 是与 store 实例具有相同方法和属性的对象
            context.commit('mutationChangeState__increment')
        },
        // 解构 context 使用
        actionCallMutation__mutationChangeState__increment({ state, commit, dispatch, getters }) {
            commit('mutationChangeState__increment')
        },

        // 2、Promise 应用
        // 异步操作转同步 Promise 1
        actionCallMutation__mutationChangeState__increment({ state, commit, dispatch, getters }) {
            return new Promise((resolve, reject) => {
                // ...
                // resolve()
                // reject()
            })
        },
        // 异步操作转同步 Promise 2
        actionCallMutation__mutationChangeState__increment2({ state, commit, dispatch, getters }) {
            dispatch('actionCallMutation__mutationChangeState__increment').then(() => {
                // ...
            })
        },

        // 3、async/await 应用
        // 异步操作转同步 async 1
        async actionCallMutation__mutationChangeState__increment({ state, commit, dispatch, getters }) {
            await getData()
        },
        // 异步操作转同步 async 2
        async actionCallMutation__mutationChangeState__increment({ state, commit, dispatch, getters }) {
            await dispatch('actionCallMutation__mutationChangeState__increment')
            // dosomething...
        },
    }
})
export default store
```
<br>

### **3、存在子模块**

#### 3.1、modulesA
```javascript
// 注意！注意！注意！
// modulesA 中的 
// Astate，Agetters，Amutations，Aactions 应该为
// state，getters，mutations，actions
// 我这里是为了区分子模块和根模块才这么命名
const modulesA = {
    // 1、在 namespaced 为 false 的情况下：
    // 默认情况下，模块内部的 action、mutation、getter 
    // 分别和根模块的 action、mutation、getter 共享一个命名空间
    // 即 Vuex 实例的 _getters、_mutations、_actions
    // store: {
    //     _getters: {},
    //     _mutations: {},
    //     _actions: {}
    // }
    
    // （1）、如果子模块的 getters 定义了和根模块相同的 getters，将会报错提示定义了相同的变量名
    // store: {
    //     _getters: {
    //         getter1: 0,
    //         getter2: {},
    //     }
    // }

    // （2）、如果子模块的 mutation 定义了和根模块相同的的 mutation，Vue 将该同名 mutation 转换成以该 mutation 名称为变量名的数组，并存储这两个 mutation 函数
    // 先执行根模块的 mutation ，然后执行子模块的 mutation
    // store: {
    //     _mutation: {
    //         mutationRepeatName: [function(){}, function(){}],
    //         otherMutation: function(){}
    //     }
    // }


    // 2、在 namespaced 为 true 的情况下，不存在覆盖根模块的情况
    // （1）、如果子模块的 getters 定义了和根模块相同的 getters，不会报错，会加入模块名作为变量名前缀
    // store: {
    //     _getters: {
    //         a/getter1: 0,
    //         getter1: {},
    //     }
    // }

    // （2）、如果子模块的 mutation 定义了和根模块相同的的 mutation，会加入模块名作为 mutation 名前缀
    // store: {
    //     _mutation: {
    //         a/mutationRepeatName: function(){},
    //         mutationRepeatName: function(){}
    //     }
    // }
    
    // 3、启用了命名空间的 getter 和 action 会收到局部化的 getter，dispatch 和 commit
    // 所以 namespaced 的改变，不影响模块代码
    namespaced: false,

    // 子模块的 Astate 直接附加到根模块的 state
    // 在根模块的 state 中， 以子模块名为 key，作为 state 的一个属性存在
    // state: { 
    //     a: { // 这里即子模块的 Astate

    //     } 
    // }
    Astate: {
        count: 0,
        todos: [
            { id: 1, done: true },
            { id: 2, done: false }
        ]
    },
    Agetters: {
        AdoneTodos: (Astate, Agetters, rootState, rootGetters) => {
            // rootState 为根模块的 state
            // rootGetters 为根模块的 rootGetters
            return Astate.todos.filter(todo => todo.done)
        },
    },
    Amuattions: {
        mutationChangeState__increment(Astate, payload) {
            // payload： 调用该方法时，传递进来的参数
            Astate.count++
        }
    },
    Aactions: {
        actionCallMutation__mutationChangeState__increment({ Astate, Acommit, Adispatch, Agetters, rootState, rootGetters }) {
            // rootState 为根模块的 state
            Acommit('mutationChangeState__increment')

            // 调用时，传递第三个参数{ root: true }， 可以访问根模块的 actions: someMutation
            Acommit('someMutation', null, { root: true })
        },
        someRootAction: {
            root: true, // 设置 root 属性为 true，可以将 someRootAction 注册在根模块
            handler (namespacedContext, payload) { ... } // -> 'someRootAction'
        }
    }
}
```
<br>

#### 3.2、使用子模块
```javascript
const store = new Vuex.Store({
    modules: {
        a: modulesA
    },
    state: {},
    // 是 state 的 computed
    getters: {},
    // mutation 必须是同步函数，以便追踪状态（编写规范如此，非强制）
    // 可以使用 state.obj = { ...state.obj, newProp: 1 } 进行值的替换（新对象替换旧对象，支持增加新属性）
    mutations: {},
    // mutation 里不允许的异步操作，可以放在 action 里面
    actions: {}
})
export default store
```
<br>

## 三、组件内常用操作

### **1、在组件中使用实例调用**
```javascript
// 调用 modulesA
// 1、调用 state
this.$store.state.a.todos
// 2、调用 getter
this.$store.getters.AdoneTodos // namespaced: false
this.$store.getters['a/AdoneTodos'] // namespaced: true
// 3、调用 mutation
this.$store.commit('AmutationChangeState__increment') // namespaced: false
this.$store.commit('a/AmutationChangeState__increment') // namespaced: true
// 4、调用 action
this.$store.dispatch('AmutationChangeState__increment') // namespaced: false
this.$store.dispatch('a/AmutationChangeState__increment') // namespaced: true


// state
this.$store.state['属性key']
// getters
this.$store.getters['属性key']
// 调用 mutation
this.$store.commit('mutationChangeState__increment', payload)
this.$store.commit({ // 对象风格调用，整个对象都将作为 payload 传递给 mutation 方法
    type: 'mutationChangeState__increment',
    payload1: '',
    payload2: ''
})
// 调用 action
this.$store.dispatch('actionCallMutation__mutationChangeState__increment')
this.$store.dispatch({ // 对象风格调用，整个对象都将作为 payload 传递给 dispatch 方法
    type: 'actionCallMutation__mutationChangeState__increment',
    payload1: '',
    payload2: ''
})
```
<br>

### **2、在组件中使用辅助函数调用**
```javascript
// 辅助函数
import { mapState, mapGetters, mapMutations, mapActions, createNamespacedHelpers  } from 'vuex'
// or
const { mapState, mapGetters, mapMutations, mapActions, createNamespacedHelpers } = createNamespacedHelpers('子命名空间')
export default {
    created() {
        // 使用
        // state
        console.log(this.stateName1, this.stateName2)
        // getter
        console.log(this.getterName1, this.getterName2)

        // mutation
        this.mutationName1()
        this.mutationName2()
        // action
        this.actionName1()
        this.actionName2()
    },
    computed: {
        // mapState
        // 数组用法
        ...mapState(['stateName1', 'stateName2']),
        // 对象用法
        ...mapState({
            stateName3: state => state.stateName3,
            stateName4_alias: state => state.stateName4,
            stateName5_alias: 'stateName5', // 此处 'stateName5'(注意是字符串) 等于 state => state.stateName5
            stateName6_resolve: state => { // 获取经过处理的值
                return state.count + this.stateName6
            },
        }),
        // 导出子模块数据
        // 导出用法随 `数组/对象` 用法，只不过加入命名空间前缀
        ...mapState(['stateName1', 'stateName2', 'module/stateName2']),
        ...mapState({
            stateName7_module: state => state.module.stateName7,
            stateName8_module: state => state.module.stateName8
        })
        ...mapState('some/nested/module', {
            stateName9: state => state.stateName9,
            stateName10: state => state.stateName10
        })
        ...mapState('some/nested/module', [
            'stateName9',
            'stateName10'
        ])

        // mapGetters
        // 数组用法
        ...mapGetters(['getterName1', 'getterName2'])
        // 对象用法
        ...mapGetters({
            getterName3_alias: 'getterName3'
        })
    },
    methods: {
        // mapMutations
        // 数组用法
        ...mapMutations(['mutationName1', 'mutationName2']),
        // 对象用法
        ...mapMutations({
            mutationName3_alias: 'mutationName3'
        }),

        // mapActions
        // 数组用法
        ...mapActions(['actionName1', 'actionName2'),
        // 对象用法
        ...mapActions({
            actionName3_alias: 'actionName3'
        }),
        // 导出子模块数据
        // 导出用法随 `数组/对象` 用法，只不过加入命名空间前缀
        ...mapActions(['module/actionName3']),
        ...mapActions('some/nested/module', [
            'actionName3'
        ]),
        ...mapActions('some/nested/module', {
            actionName3_alias: 'actionName3'
        }),
    }
}
```
