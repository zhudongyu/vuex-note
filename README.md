# vuex-note


```JavaScript
1.搞清楚什么是 vuex 是很关键的 ！
  -- Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。
     它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。
  -- vuex  --> store (核心) 
  -- store --> 全局状态管理 --> 共享状态抽离
  -- 共享状态抽离 --> 组件的共享状态抽取出来，形成一个全局单例模式管理，这样我们的组件树构成了一个巨大的“视图”，
                     不管在树的哪个位置，任何组件都能获取状态或者触发行为！
  -- 局部状态 --> 如果有些状态严格属于单个组件，最好还是作为组件的局部状态。

  **注意** ： 
      1. Vuex 的状态存储是响应式的。
         当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。
      2. 状态追踪 --> 不能直接改变 store 中的状态
         改变 store 中的状态的唯一途径就是显式地提交 (commit) mutation。这样使得我们可以方便地跟踪每一个状态的变化。

***Vuex的使用场景***    一般是在项目的组件之间共享状态较多时
***Vuex解决的问题***    1.解决了代码解耦  2.数据组件传递

```
```JavaScript
2.在使用vuex之前先了解几个概念

  -- Vue Components：Vue组件。HTML页面上，负责接收用户操作等交互行为，执行dispatch方法触发对应action进行回应。(Vuex)

  -- dispatch：操作行为触发方法，是唯一能执行action的方法。

  -- actions：操作行为处理模块。负责处理Vue Components接收到的所有交互行为。包含同步/异步操作，支持多个同名方法，
              按照注册的顺序依次触发。向后台API请求的操作就在这个模块中进行，包括触发其他action以及提交mutation的操作。
              该模块提供了Promise的封装，以支持action的链式触发。

  -- commit：状态改变提交操作方法。对mutation进行提交，是唯一能执行mutation的方法。

  -- mutations：状态改变操作方法。是Vuex修改state的唯一推荐方法，其他修改方式在严格模式下将会报错。
                该方法只能进行同步操作，且方法名只能全局唯一。操作之中会有一些hook暴露出来，以进行state的监控等。

  -- state：页面状态管理容器对象。集中存储Vue components中data对象的零散数据，全局唯一，以进行统一的状态管理。
            页面显示所需的数据从该对象中进行读取，利用Vue的细粒度数据响应机制来进行高效的状态更新。

  -- getters：state对象读取方法。官方图中没有单独列出该模块，应该被包含在了render中，Vue Components通过该方法读取全局state对象。
              Vue组件接收交互行为，调用dispatch方法触发action相关处理，若页面状态需要改变，则调用commit方法提交mutation修改state，
              通过getters获取到state新值，重新渲染Vue Components，界面随之更新。

```
```JavaScript
2. vuex 严格模式
   1). 作用：在严格模式下，无论何时发生了状态变更且不是由 mutation 函数引起的，将会抛出错误。
             这能保证所有的状态变更都能被调试工具跟踪到。

   2). 开发模式下启用 生产环境下关闭
       const store = new Vuex.Store({
          strict: process.env.NODE_ENV !== 'production'
          ...
       })

       **注意** ： 
                 严格模式下会一直处于深度监测状态树来检测不合规的状态变更
                 所以必须在发布环境下关闭严格模式，以避免性能损失。


```
```JavaScript
2.初始化store并注入vue实例
store.js
  import Vue from "vue"
  import Vuex from "vuex"
  Vue.use(Vuex) //

  const store = new Vuex.Store({
    ...
  })



main.js
  import store from "./store.js"
  const app = new Vue({
    el: '#app',
    // 把 store 对象提供给 “store” 选项，这可以把 store 的实例注入所有的子组件,且子组件能通过 this.$store 访问到。
    store,
    components: { Counter },
    template: `
      <div class="app">
        <counter></counter>
      </div>
    `
  })

```

```JavaScript
3.state
  1). 作用 ： 

  2).访问state
      @1. this.$store.state.xxxx
      @2.在组件中
         import { mapState } from 'vuex'
         computed : {
            localComputed : function(){
               ....
            },
            // 1.字符串数组 ： 当映射的计算属性的名称与 state 的子节点名称相同时
            ...mapState ([
                "stateA", "stateB","stateC", "stateD"
            ])
            // 2.
            ...mapState ({
                count: state => state.count,               // 箭头函数可使代码更简练
                countAlias: 'count',                       // 传字符串参数 'count' 等同于 `state => state.count`
                countPlusLocalState : function(state) {    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
                  return state.count + this.localCount
                }
            })
         }
  3). 在属于 Vuex 的 state 上使用 v-model  --> 双向绑定的计算属性

            <input v-model="message">
          
            computed: {
               message: {
                  get () {
                     return this.$store.state.obj.message
                  },
                  set (value) {
                     this.$store.commit('updateMessage', value)
                  }
               }
             }

```
```JavaScript
4.getter
  1). 作用 ： 
        @1.访问state中的数据
        @2.从state中派生出一些状态
  2). 特性：
        @1.就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。
        @2.Getter 接受 state 作为其第一个参数：
        @3.Getter 也可以接受其他 getter 作为第二个参数
        @4.通过让 getter 返回一个函数，来实现给 getter 传参。在对 store 里的数组进行查询时非常有用。
            getters: {
              // ...
              getTodoById: (state) => (id) => {
                  return state.todos.find(todo => todo.id === id)
              }
            }
            this.$store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
  3). 访问
        @1.在组件中
           第一种 ：this.$store.getters.xxxx
           第二种 ：使用辅助函数mapGetters 将 store 中的 getter 映射到局部计算属性
                   import { mapGetters} from 'vuex'
                   computed: {
                      // 1.字符串数组
                      ...mapGetters([
                         'doneTodosCount','anotherGetter',
                      ])
                      // 2.
                      ...mapGetters({
                         'doneCount' : 'doneTodosCount'
                      })
                    }
```
```JavaScript
5.mutation
  1). 作用：作为更改 Vuex 的 store 中的状态的唯一方法
  2). 特性：
        @1. 每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)
            回调函数（handler）接收两个参数，第一个是 state,第二个是 提交载荷（payload）
        @2 .需要以相应的 type 调用 store.commit 方法来触发 mutation handler
            commit 接收两个参数，第一个是 事件类型（type） 第二个是 提交载荷（payload）
        @3. Mutation 必须是同步函数，否则无法追踪状态
        @4. Mutation 需遵守 Vue 的响应规则
                -- 提前在 store 中初始化好所有所需属性
                -- 当需要在对象上添加新属性时 -> 使用 Vue.set(obj, 'newProp', 123) / state.obj = { ...state.obj, newProp: 123 }

  3). 定义
          mutations: {
            ...
            increment (state,Payload) {
              state.count += Payload.a
            }
          }

  4). 提交方式
        @1.在组件中
           第一种 ： 直接触发
                     var Payload = {a : 1,b: 2}
                     this.$store.commit('increment', Payload)
           第二种 ： 使用 mapMutations 辅助函数将组件中的 methods 映射为 store.commit 调用
                     import { mapMutations } from 'vuex'         
                     methods: {
                        ...mapMutations([
                          'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`

                          // `mapMutations` 也支持载荷：
                          'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
                        ]),
                        ...mapMutations({
                            add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
                        })
                      }
        @2. 在Actions中触发提交


```
```JavaScript
6.actions
  1). 作用：提交mutation
  2). 特性：
        @1. 可以包含任意异步操作
        @2. Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象 --> 一般会进行参数解构 {state,commit,...}
            context.commit 提交一个 mutation
            context.state 和 context.getters 来获取 state 和 getters
        @3. Action 通过 store.dispatch 方法触发
            dispatch接收两个参数，第一个是 分发的事件类型（type） 第二个是分发的载荷（payloader）

  3). 定义 ： 购物车示例 --> 调用异步 API 和分发多重 mutation
          actions: {
             checkout ({ commit, state }, products) {
                const savedCartItems = [...state.cart.added] // 把当前购物车的物品备份起来
                commit(types.CHECKOUT_REQUEST)  // 发出结账请求，然后乐观地清空购物车
                // 购物 API 接受一个成功回调和一个失败回调
                shop.buyProducts(
                    products,
                    () => commit(types.CHECKOUT_SUCCESS),                      // 成功操作
                    () => commit(types.CHECKOUT_FAILURE, savedCartItems)       // 失败操作
                )
             }
          }
  4). 分发方式
        @1. 在组件中
            第一种 ： 直接触发
                      var Payload = {a : 1,b: 2}
                      this.$store.dispatch('checkout',Payload)
            第二种 ：使用 mapActions 辅助函数将组件的 methods 映射为 store.dispatch 调用
                     import { mapActions } from 'vuex'                  
                     methods: {
                       ...mapActions([
                          'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`

                           // `mapActions` 也支持载荷：
                          'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
                       ]),
                       ...mapActions({
                           add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
                       })
                     }
  5). 组合多个 action，以处理更加复杂的异步流程
        @1. store.dispatch 可以处理被触发的 action 的处理函数返回的 Promise，并且 store.dispatch 仍旧返回 Promise





```
```JavaScript
vuex 源码解读：

通过Vue.use(vuex)时的操作：
先调用install，安装vuex，然后在安装之后调用初始化方法
这个初始化方法包含：在Vue的钩子函数beforeCreate中插入Vuex初始化代码
注入 $store对象


读源码：
读懂人家的代码
读懂人家的技巧
学会灵活运用人家的技巧
```





























