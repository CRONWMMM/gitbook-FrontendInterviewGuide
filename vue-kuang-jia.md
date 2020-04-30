---
description: Vue 框架的相关面试内容
---

# Vue框架

## **生命周期**

1. 在 beforeCreate 钩子函数调用的时候，是获取不到 props 或者 data 中的数据的，因为这些数据的初始化都在 initState 中。
2. 然后会执行 created 钩子函数，在这一步的时候已经可以访问到之前不能访问到的数据，但是这时候组件还没被挂载，所以是看不到的。
3. 接下来会先执行 beforeMount 钩子函数，开始创建 VDOM，最后执行 mounted 钩子，并将 VDOM 渲染为真实 DOM 并且渲染数据。组件中如果有子组件的话，会递归挂载子组件，只有当所有子组件全部挂载完毕，才会执行根组件的挂载钩子。
4. 接下来是数据更新时会调用的钩子函数 beforeUpdate 和 updated，这两个钩子函数没什么好说的，就是分别在数据更新前和更新后会调用。
5. 另外还有 keep-alive 独有的生命周期，分别为 activated 和 deactivated 。用 keep-alive 包裹的组件在切换时不会进行销毁，而是缓存到内存中并执行 deactivated 钩子函数，命中缓存渲染后会执行 actived 钩子函数。
6. 最后就是销毁组件的钩子函数 beforeDestroy 和 destroyed。前者适合移除事件、定时器等等，否则可能会引起内存泄露的问题。然后进行一系列的销毁操作，如果有子组件的话，也会递归销毁子组件，所有子组件都销毁完毕后才会执行根组件的 destroyed 钩子函数。

## **组件通信**

**父子通信：**

1. 子 $emit 父 / 父 props 子
2. v-model 语法糖，本质上是一个名为 value 的 Prop 传递到子组件，子组件发送 input 时间通知父组件
3. 可以通过 $parent / $children 来访问父 / 子组件中的数据和方法
4. $listeners 属性会将父组件中的 \(不含 .native 修饰器的\) v-on 事件监听器传递给子组件，子组件可以通过访问 $listeners 来自定义监听器。类似于 react 的子组件通知父组件的方式，父组件的回调作为 props 传入子组件
5. .sync 属性是个语法糖，可以实现简单的父子组件通信：

   ```text
   <!--父组件中-->
   <input :value.sync="value" />
   <!--以上写法等同于-->
   <input :value="value" @update:value="v => value = v"></comp>
   <!--子组件中-->
   <script>
   this.$emit('update:value', 1)
   </script>
   ```

#### **兄弟组件通信**

对于这种情况可以通过查找父组件中的子组件实现，也就是 this.$parent.$children，在 $children 中可以通过组件 name 查询到需要的组件实例，然后进行通信。

#### **跨层级组件通信**

使用 provide 和 inject （vue2.2+），这对选项需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。如果你熟悉 React，这与 React 的上下文特性很相似。

**任意组件间的通信**

可以使用 Event bus 或者 vuex 通信，所谓 Event bus 就是再新床架一个空的 Vue 实例，然后通过这个实例的 $emit / $on 方法在任意出发送和接受事件并传递参数

## **extend 是用来做什么的**

extend 通常用于挂载全局使用的组件而不需要在每一个需要用到的地方单独 import

```javascript
// 创建组件构造器
let Component = Vue.extend({
  template: '<div>test</div>'
})
// 挂载到 #app 上
new Component().$mount('#app')
// 除了上面的方式，还可以用来扩展已有的组件
let SuperComponent = Vue.extend(Component)
new SuperComponent({
    created() {
        console.log(1)
    }
})
new SuperComponent().$mount('#app')
```

## **mixin 和 mixins**

1. mixin 全局注入，mixins 局部注入
2. 另外需要注意的是 mixins 混入的钩子函数会先于组件内的钩子函数执行，并且在遇到同名选项的时候也会有选择性的进行合并，如果 mixins 和组件内属性同名，以组件内部为准

## **computed 和 watch**

1. computed 是计算属性，依赖其他属性计算值，并且 computed 的值有缓存，只有当计算值变化才会返回内容。擅长处理的场景：一个数据受多个数据影响
2. watch 监听到值的变化就会执行回调，在回调中可以进行一些逻辑操作。擅长处理的场景：一个数据印象多个数据
3. 所以一般来说需要依赖别的属性来动态获得值的时候可以使用 computed，对于监听到值的变化需要做一些复杂业务逻辑的情况可以使用 watch。

## **keep-alive 的作用**

1. 如果你需要在组件切换的时候，保存一些组件的状态防止多次渲染，就可以使用 keep-alive 组件包裹需要保存的组件。
2. 对于 keep-alive 组件来说，它拥有两个独有的生命周期钩子函数，分别为 activated 和 deactivated 。用 keep-alive 包裹的组件在切换时不会进行销毁，而是缓存到内存中并执行 deactivated 钩子函数，命中缓存渲染后会执行 actived 钩子函数。

## **v-show 与 v-if 区别**

## **组件中的 data 为什么不能使用对象**

使用对象的话，该组件的所有实例都会共享一个 data 了。

## **vue-element-admin 中权限控制**

* `router` 中将路由分为了 `constantRoutes` （常量路由）和 `asyncRoutes` （动态路由）两个列表
* `main.js` 中引入了路由守卫拦截文件 `permission.js`
* 路由守卫中判断路由：
  * 如果 `store` 中有存储过的权限列表，放行
  * 否则去后端获取权限列表数据 `List<String>`，获取到后根据后端的数据及 `asyncRoutes` 调用 `router.addRoute` 方法添加新的路由后，放行。这时，如果 `to.path` 在路由里，就跳转到对应页面，否则就跳转 404
* 关键：根据后端数据使用 `router.addRoute` 动态生成 `vue` 路由

## 进阶

Vue 深入原理可以查看[Vue.js 技术揭秘](https://ustbhuangyi.github.io/vue-analysis/)

