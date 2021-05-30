## 声明式渲染
* Vue.js 的核心是一个允许采用简洁的模板语法来声明式地将数据渲染进 DOM 的系统
```html
    <div id="app">
        {{ message }}
    </div>
```
```javascript
    var app = new Vue({
        el:'#app',
        data: {
            message: 'Hello World'
        }
    })
```
* 现在数据和 DOM 已经被建立了关联，所有东西都是响应式
* 现在不再和 HTML 直接交互了。一个 Vue 应用会将其挂载到一个 DOM 元素上（#app）然后对其完全控制。这个 HTML 是一个入口，其余都会发生在新创建的 Vue 实例内部

```html
    <div id="app-2">
        <span v-bind:title="message">
            鼠标悬停几秒钟查看此处动态绑定的提示信息！
        </span>
    </div>
```
```javascript
    var app2 = new Vue({
        el: '#app-2',
        data: {
            message: '页面加载于' + new Date().toLocaleString()
        }
    })
```
* 这里的 v-bind attribute 被称为指令。指令前面带有前缀 v- ，以表示它们使 Vue 提供的特殊 attribute。
* 该指令的意思是“将这个元素节点的 title attribute 和 Vue 实例的 message property 保持一致”

## 组件化应用构建
* 组件系统是一种抽象，允许我们使用小型、独立和通常可复用的组件构建大型应用。
* 几乎任意类型的应用界面都可以抽象为一个组件树
* 在 Vue 里，一个组件本质上是一个拥有预定义选项的一个 Vue 实例。
```javascript
    // 定义名为 todo-item 的新组件
    Vue.component('todo-item', {
        template: '<li>这是个待办项</li>'
    })
    var app = new Vue(...)
```
```html
    <!-- 构建另一个组件模板 -->
    <ol>
        <!-- 创建一个 todo-item 组件的实例 -->
        <todo-item></todo-item>
    </ol>
```

## 数据和方法
* 当一个 Vue 实例被创建时，它将 data 对象中的所有的 property 加入到 Vue 的响应式系统中。当这些 property 的值发生变化时，视图将会产生“响应”，即匹配更新为新的值
```javascript
    // 数据对象
    var data = {a: 1}
    // 该对象被加入到一个 Vue 实例中
    var vm = new Vue({
        data: data
    })
    // 获取这个实例上的 property
    // 返回源数据中对应的字段
    vm.a == data.a  // true

    // 设置 property 也会影响到原始数据
    vm.a = 2
    data.a   // 2

    // 反之亦然
    data.a = 3
    vm.a  // 3
```
* 当这些数据改变时，视图会进行重渲染。
    * 注意：只有当实例被创建时就已经存在于 data 中的 property 才是响应式的。也就是说添加一个新的 property
    ```javascript
        vm.b = 'hello'
    ```
    * 那么 b 的改动将不会触发任何视图的更新。
* 所以需要在开始的时候就设置一些初始值
```javascript
    data: {
        newTodoText: '',
        visitCount: 0,
        hideCompletedTodos: false,
        todos: [],
        error: null
    }
```
* 若使用 Object.freeze()，这会阻止修改现有的 property，这意味着响应式系统无法再追踪变化
```javascript
    var obj = {
        foo: 'bar'
    }
    Object.freeze(obj)
    new Vue({
        el: '#app',
        data: obj
    })
```
* Object.freeze(obj) 方法可以冻结一个对象。一个被冻结的对象再也不能被修改；冻结了一个对象则不能向这个对象添加新的属性，不能删除已有属性，不能修改该对象已有属性的可枚举性、可配置性、可写性，以及不能修改已有属性的值。此外，冻结了一个对象后该对象的原型也不能被修改。freeze() 返回和传入的参数相同对象