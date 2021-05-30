# 1. 讲一讲MVVM
1. MVVM 是 model-View-ViewModel缩写，就是将 MVC 中的Controller 演变成 ViewModel。
2. Model 层代表数据模型，View 代表 UI 组件，ViewModel 是 View 和 Model 的桥梁，数据会绑定到 ViewModel 层并自动将数据渲染到页面中，视图的变化的时候也会通知 ViewModel 层更新数据

# 2. Vue2.x 响应式数据原理
    Vue 在初始化数据时，会使用 Object.defineProperty 重新定义data 中的所有属性，当页面使用对应属性时，首先会进行依赖收集（收集当前组件的watcher）
    如果属性发生变化会通知相关依赖进行更新操作（发布订阅）

# 3. Vue3.x 响应式数据原理
    Vue3.x 采用 Proxy 替代 Object.defineProperty。因为 Proxy 可以直接监听对象和数组的变化，并且有13中拦截方法。并且作为新标准受到浏览器厂商重点持续的性能优化

## 1. Proxy 只会代理对象的第一层，那么Vue3又是如何处理这个问题的呢？
    判断当前 Reflect.get 的返回值是否为 Object，如果是则再通过 reactive 方法做代理，这样就实现了深度观测

## 2. 检测数组的时候可能触发多次get/set，那么如何防止触发多次呢？
    我们可以判断 key 是否为当前被代理对象 target 自身属性，也可以判断旧值与新值是否相等，只有满足以上两个条件之一，才有可能执行 target

# 4. Vue2.x 中如何检测数组变化
    使用了函数劫持的方式，重写了数组的方法，Vue将data中的数组进行了原型链的重写，指向自己定义的数组原型方法。
    这样当调用数组API时，可以通知依赖更新。如果数组中包含着引用类型，会对数组中的引用类型再次进行递归遍历进行监控，这样就实现了检测数组变化

# 5. nextTick 实现原理是什么？
    在下次 DOM 更新循环结束之后执行延迟回调。nextTick 主要使用了宏任务和微任务，根据执行环境分别尝试采用
        Promise
        MutationObserver
        setImmediate
        如果以上方法都不行则采用setTimeout

    定义了一个异步方法，多次调用 nextTick 会将方法存入队列中，通过这个异步方法清空当前队列

# 6. Vue 的生命周期
1. beforeCreate()
    * 是 new Vue() 之后触发的第一个钩子，在当前阶段 data、methods、computed 以及 watch 上的数据和方法都不能被访问

2. Create()
    * 在实例创建完成之后发生，在当前阶段已经完成了数据观测，也就是可以使用数据，更改数据，在这里更改数据不会触发 updated 函数。可以做一些数据的获取，在当前阶段无法与 DOM 进行交互，如果要，则可以通过 vm.$nextTick 来访问DOM

3. beforeMounted
    * 发生在挂载之前，在这之前 template 模板已导入渲染函数编译，而当前阶段虚拟 Dom 创建完成，即将开始渲染，在此时可以更改数据，不会触发 Updated

4. Mounted
    * 在挂载完成后发生，在当前阶段，真实的 Dom 挂载完毕，数据完成双向绑定，可以访问 Dom 节点，使用 $refs 属性对 Dom 进行操作

5. beforeUpdate
    * 发生在更新之前，也就是响应式数据发生更新，虚拟 Dom 重新渲染之前被触发，可以在当前阶段更改数据，不会造成重污染

6. Updated
    * 发生在更新完成之后，当前组件 Dom 已经更新完成。注意在此阶段避免更改数据，因为这可能导致无限循环的更新

7. beforeDestroy
    * 发生在销毁实例之前，在当前阶段实例完全可以被使用，我们可以在此阶段进行收尾工作，例如清除定时器

8. Destroyed
    * 发生在实例销毁之后，这个时候只剩下了dom空壳，组件已经被拆解，数据绑定被卸载，监听被移除，子实例也统统被销毁

# 7. 接口请求一般放在哪个生命周期？
* 请求接口一般放在 mounted 中，但需要注意的是服务端渲染时不支持 mounted，需要放到 created 中

# 8. Computed 和 Watch
1. Computed 本质是一个具备缓存的 watcher，依赖的属性发生变化就会更新视图
    * 用来声明式的描述一个值依赖了其它的值，当所依赖的值或者变量改变时，计算属性也会跟着改变
    * 适用于计算比较消耗性能的计算场景。当表达式过于复杂时，在模板中放入过多逻辑会让模板难以维护，可以将复杂的逻辑放入计算属性中处理
2. Watch 没有缓存性，更多的是观察的作用，可以监听某些数据执行回调。当我们需要深度监听对象中的属性时，可以打开 deep：true 选项，这样便会对对象中的每一项进行监听
    * 监听的是已经在 data 中定义的变量，当该变量变化时，会触发 watch 中的方法 
    * 这样会带来性能问题，优化的话可以使用字符串形式监听，如果没有写到组件中，则要手动注销

# 9. v-if和v-show
    当条件不成立，v-if不会渲染DOM元素，v-show操作的样式(display)，切换当前DOM的显示和隐藏

# 10 组件中data为什么是一个函数？
1. 一个组件被复用多次的话，也就会创建多个实例。
2. 本质上，这些实例用的都是同一个构造函数。
3. 如果data是对象时，对象属于引用类型，会影响到所有实例
4. 所以为了保证组件不同的实例之间data不冲突，data必须是一个函数

# 11 v-model 的原理
    v-model 本质是一个语法糖，可以看做是 value + input 方法的语法糖。可以通过model属性的prop和event属性来进行自定义。原生的v-model，会根据标签的不同生成不同的事件和属性

# 12 Vue事件绑定原理
    原生事件绑定是通过 addEventListener 绑定给真实元素，组件绑定是通过Vue自定义的 $on 实现的

# 13 Vue 模板编译原理
1. Vue 中的模板 template 无法被浏览器解析和渲染，因为这不属于浏览器的标准，不是正确的HTML语法
    所以需要将 template 转换为一个JavaScript函数，这样浏览器就可以执行这个函数渲染出对应的HTML元素，就可以让视图跑起来，这样一个转换过程，就成为模板编译

2. Vue 的编译过程就是将 template 转换为 render 函数的过程
3. 模板编译又分为三个阶段：解析parse，优化optimize，生成generate，最终生成可执行函数render
    1. parse 阶段：使用大量的正则表达式对 template 字符串进行解析，将标签、指令、属性等转换为抽象语法树AST
    2. optimize 阶段：深度遍历AST，找到其中一些静态节点并进行标记，方便在页面渲染的时候进行diff比较，直接跳过这一些静态节点，优化runtime的性能
    3. generate阶段：将最终的AST转换为render函数字符串

# 14 Vue2.x 和 Vue3.x 渲染器的diff算法
1. diff 算法有以下过程
    1. 首先，对比节点本身，判断是否为同一节点；如果不为相同节点，则删除该节点重新创建节点进行替换
    2. 如果为相同阶段，进行parthVnode，判断如何对该节点的子节点进行处理，先判断一方有子节点，一方没有子节点的情况（如果新的children没有子节点，将旧的节点移除）
    3. 比较如果都有子节点，则进行updateChildren，判断如果对这些新老节点进行操作（diff核心）
    4. 匹配时，找到相同的节点，递归比较子节点

*   在diff中，只对同层的子节点进行比较，放弃跨级的节点比较，使得时间复杂度O(n^3)降低值O(n)，
    也就是说，只有当新旧chilren都为多个子节点时才需要用核心的Diff算法进行同层级比较

2. Vue2的核心diff算法采用了 双端比较 的算法，同时从新旧children的两端开始比较，借助key值找到可复用的节点，再进行相关的操作。

3. Vue3 借鉴了ivi算法和inferno算法
    在创建VNode时就确定其类型，以及在 mount/patch 的过程中采用 位运算 来判断VNode 的类型，在这个基础之上在配合核心的diff算法，使得性能提升
    
    该算法中还运用了动态规划的思想求最长递归子序列

# 15. 虚拟DOM以及key属性的作用
## 1. 虚拟Dom
1. Virtual Dom 是 Dom 节点在 JavaScript 中的一种抽象数据结构，之所以需要虚拟 Dom，是因为浏览器中操作 Dom 的代价比较昂贵，频繁操作dom会产生性能问题
2. 虚拟 Dom 的作用是在每一次响应式数据发生变化引发重渲染时，Vue 对比更新前后的虚拟 Dom，匹配找出更可能少的需要更新的 Dom，从而提升性能

## 2. key属性
* key的作用就是更可能的复用dom元素
* 在对节点进行diff的过程中，判断是否为相同节点的一个很重要的条件就是key是否相等
    * 如果是相同节点，则会尽可能的复用原有的DOM节点
    * 所以key属性是提供给框架在diff的时候使用

# 16. keep-alive
1. keep-alive 可以实现组件缓存，当组件切换时不会对当前组件进行卸载
2. 常用的两个属性 include/exclude，允许组件有条件的进行缓存
3. 两个生命周期 activated/deactivated，用来得知当前组件是否处于活跃状态
4. keep-alive 中还运用了LRU（leadst Recently Used）算法

# 17. Vue中组件生命周期调用顺序说一下
* 组件调用的顺序都是 先父后子，渲染完成的顺序是 先子后父
* 组件的销毁操作是 先父后子，销毁完成的顺序是 先子后父

1. 加载渲染过程
父beforeCreate->父created->父beforeMount->子beforeCreate->子created->子beforeMount- >子mounted->父mounted

2. 子组件更新过程
父beforeUpdate->子beforeUpdate->子updated->父updated

3. 父组件更新过程
父 beforeUpdate -> 父 updated

4. 销毁过程
父 beforeDestroy -> 子beforeDestroy -> 子destroyed -> 父destroyed

# 18. Vue2.x 组件通信方式有那些
1. 父子组件通信
    父 -> 子 props
    子 -> 父 $on、$emit
    获取父子组件实例 $parent、 $children

    Ref 获取实例的方式调用组件的属性或者方法
2. 兄弟组件通信
    Event Bus 实现跨组件通信 Vue.prototype.$bus = new Vue
    Vuex
3. 跨级组件通信
    Vuex
    $attrs、$listeners
    Provide、inject

# 19. SSR
    SSR 服务端渲染，就是将Vue在客户端把标签渲染成HTML的工作放在服务端完成，然后再把html直接返回给客户端

    SSR游着更好的SEO、并且首屏加载速度更快等优点。不过它也有一些缺点，比如开发条件会受到限制，服务器渲染只支持beforeCreate和created两个钩子，当我们需要一些外部扩展时需要特殊处理，服务端渲染应用程序也需要处于node.js的运行环境，。服务器会有更大的负载

# 20. Vue性能优化
## 1. 编码阶段
1. 尽量减少data中的数据，data中的数据都会增加getter和setter，会收集对应的watcher
2. v-if和v-for不能连用
3. 如果需要使用v-for给每项元素绑定事件时使用事件代理
4. SPA页面采用keep-alive缓存组件
5. 在更多的情况下，使用v-if替代v-show
6. key保证唯一
7. 使用路由懒加载、异步组件
8. 节流、防抖
9. 第三方模块按需导入
10. 长列表滚动到可视区域动态加载
11. 图片懒加载

## 2. SEO优化
1. 预渲染
2. 服务端渲染SSR

## 3. 打包优化
1. 压缩代码
2. Tree Shaking/Scope Hoisting
3. 使用cdn加载第三方模块
4. 多线程打包happypack
5. splitChunks抽离公共文件
6. sourceMap优化

# 21. hash理由和history路由原理
1. location.hash 的值实际就是 URL 中 # 后面的东西
2. history 实际采用了HTML5中提供的API来实现，主要有history.pushState()和history.replaceState()

# 22. 删除数组用 delete 和 Vue.delete 有什么区别？
* delete：只是被删除数组成员变为 empty/undefined，其他元素键值不变
* Vue.delete：直接删了数组成员，并改变了数组的键值（对象是响应式的，确保删除能触发更新视图，这个方法主要用于避开 Vue 不能检测到属性被删除的限制）

# 23. 双向绑定的原理
* View 的变化能实时让 Model 发生变化，而 Model 的变化也能实时更新到 View
* Vue 采用数据劫持&发布-订阅模式方法，通过 ES5 提供的 Object.defineProperty() 方法来劫持（监控）各属性的 getter、setter，并在数据（对象）发生变动时通知订阅者，触发相应的监听回调。由于是在不同的数据上触发回调同步，可以精确的将变更发送给绑定的视图，而不是对所有的数据都执行一次检测
* 要实现 Vue 中的双向数据绑定，大致可以划分为三个模块：Observer、Compile、Watcher
    1. Observer 数据监听器
        * 负责对数据对象的所有属性进行监听（数据劫持），监听到数据变化后通知订阅者
    2. Compile 指令解析器
        * 扫描模板，并对指令进行解析，然后绑定指定事件
    3. Watcher 订阅者
        * 关联 Observer 和 Compile 能够订阅并收到属性变化的通知，执行指令绑定的响应操作，更新视图。Updata() 是它自身的一个方法，用于在 Compile 中绑定回调，更新视图

## 24. v-model 是什么？有什么用呢？
* 一则语法糖，相当于 v-bind:value = "xx" 和 @input，意思是绑定了一个 value 属性的值，子组件可对 value 属性监听，通过 $emit('input', xxx) 的方式给父组件通讯

## 25. axios 是什么？怎么使用它？怎么解决跨域的问题？
* axios 的是一种异步请求，用法和 ajax 类似，请求中包括 get、post、put、patch、delete 等五种请求方式，解决跨域可以在请求头中添加 Access-Control-Allow-Origin，也可以在 index.js 文件中更改 proxyTable 配置等解决跨域问题