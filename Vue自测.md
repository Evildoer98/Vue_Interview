# 对 SPA 单页面的理解以及优缺点？
* SPA（single-page application）仅在 Web 页面初始化加载相应的 HTML、JavaScript 和 CSS。一旦页面加载完毕，SPA 不会因为用户的操作而进行页面的重新加载或跳转；取而代之的利用路由机制实现 HTML 内容的交换，UI 与用户的交互，避免页面的重新加载。
* 优点：
    1. 用户体验好、快，内容的改变不需要重新加载整个页面，避免了不必要的跳转的重复渲染
    2. SPA 相对服务器压力小
    3. 前后端职责分离，架构清晰，前端进行交互逻辑，后端负责数据处理
* 缺点：
    1. 初次加载耗时多：为实现单页 Web 应用功能及显示效果，需要在加载页面的时候将 JavaScript、CSS 统一加载，部分页面按需加载
    2. 前进后退路由管理：由于单页应用在同一个页面中显示所有的内容，所以不能使用浏览器的前进后退功能，所有页面的切换需要自己建立堆栈管理
    3. SEO 难度较大：由于所有的内容都在一个页面中动态替换显示

# v-show 与 v-if 有什么区别？
1. v-if 是真正的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当的被销毁和重建；也是惰性的；如果在初始渲染时条件为假，则什么也不做 ———— 直到条件第一次变为真时，才会开始渲染条件块
2. v-show ———— 不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 的"display:none" 属性进行切换
* v-if 适用于在运行时很少改变条件，不需要频繁切换的场景
* v-show 则适用于需要非常频繁切换条件的场景

# Class 与 Style 如何动态绑定？
* class 可以通过对象语法与数组语法进行动态绑定
    1. 对象语法
        ```javascript
            <div v-bind:class="{active: isActive, 'text-danger': hasError}"></div>
            data: {
                isActive: true,
                hasError: flase
            }
        ```
    2. 数组语法
        ```javascript
            <div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
            data: {
                activeClass: 'active',
                errorClass: 'text-danger'
            }
        ```
* style 也可以通过对象语法和数组语法进行动态绑定
    1. 对象语法
        ```javascript
            <div v-bind:style="{color: activeColor, fontSize: fontsize + 'px'}"></div>
            data: {
                activeColor: 'red',
                fontSize: 30
            }
        ```
    2. 数组语法
        ```javascript
            <div v-bind:style="[styleColor, styleSize]"></div>
            data: {
                styleColor: {
                    color: 'red'
                },
                styleSize: {
                    fontSize: '30px'
                }
            }
        ```

# Vue 的单向数据流
* 所有的 prop 都使得其父子 prop 之间形成了一个单向下行绑定：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。这样防止从子组件意外改变父级组件的状态，从而导致应用数据流向难以理解。
* 每次父级组件发生更新时，子组件中所有的 prop 都会刷新为最新的值。
* 子组件要想修改，只能通过 $emit 派发一个自定义事件，父组件接收到后，由父组件修改
* 两种常见的修改 prop 情形
    1. 这个 prop 用来传递一个初始值，这个子组件接下来希望将其作为一个本地的 prop 数据来使用。在这种情况下，最好定义一个本地的 data 属性并将这个 prop 用作其初始值
        ```javascript
            prop: ['initialCounter'],
            data: function () {
                return {
                    counter: this.initialCounter
                }
            }
        ```
    2. 这个 prop 以一种原始的值传入且需要进行叫唤，这种情况下，最好使用 prop 的值来定义一个计算属性
        ```javascript
            prop: ['size'],
            computed: {
                normalizedSize: function () {
                    return this.size.trim().toLowerCase()
                }
            }
        ```

# Vue 组件间通信有哪几种方式
1. props / $emit 适用于 父子组件通信
2. ref 与 $parent / $children 适用 父子组件通信
    * ref：如果在普通函数的 DOM 元素上使用，引用指向的就是 DOM 元素；如果用在子组件上，引用就指向组件实例
    * $parent / $children：访问父/子实例
3. EventBus（$emit/ $on）适用于 父子、隔代、兄弟组件通信
    * 通过一个空的 Vue 实例作为中央事件总线（事件中心），用它来触发事件和监听事件，从而实现任何组件间的通信，包括父子、隔代、兄弟组件
4. $attrs / $listeners 适用于隔代组件通信

# computed 和 watch 的区别和运用的场景？
* computed: 是计算属性，依赖其它属性值，并且 computed 的值有缓存，只有它依赖的属性值发生改变，下一次获取 computed 的值时才会重新计算 computed 的值
* watch：更多的是观察的作用，类似于某些数据的监听回调，每当监听的数据变化时都会执行回调进行后续操作
* 运用场景：
    1. 当需要进行数值计算，并且依赖于其它数据时，应该使用 computed，因为可以利用 computed 的缓存特性，避免每次获取值时，都要重新计算
    2. 当需要在数据变化时执行异步或开销较大的操作时，应该使用 watch，使用 watch 选项允许执行异步操作（访问一个 API），限制执行该操作的频率，并在得到最终结果前，设置中间状态。

# 直接给一个数组项赋值，Vue 能检测到变化吗？
* 由于 JavaScript 的限制，Vue 不能检测到以下数组的变动：
    1. 当利用索引直接设置一个数组项时
        eg：vm.items[indexOfItem] = newValue
    2. 当修改数组的长度时
        eg：vm.items.length = newLength
* 解决第一个问题，Vue 提供了一下操作方法：
    ```javascript
        // Vue.set
        Vue.set(vm.items, indexOfItem, newValue)
        // vm.$set，Vue.set 的一个别名
        vm.$set(vm.items, indexOfItem, newValue)
        // Array.prototype.splice
        vm.items.splice(indexOfItem, 1, newValue)
    ```
* 解决第二个问题，Vue 提供了一下操作方法
    ```javascript
        // Array.prototype.splice
        vm.items.splice(newLength)
    ```

#