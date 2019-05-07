---
title: Vue生命周期详解，Vue父子组件生命周期执行顺序。
date: 2019-05-04 13:37:58
tags: [Vue]
categories: [Vue]
description: 总结一下Vue生命周期，还有Vue是怎么渲染父子组件的，那些生命周期父组件先执行，那些生命周期在子组件先执行。
---
## Vue生命周期
一说到Vue的生命周期，大家都能说上来一点.我自己会随着自己对Vue的深入了解，不断的完善自己对Vue的整体认识。
本文Vue的版本 <font color="blue">Vue 2.x</font> 官方生命周期图解：
![vue-hook](../images/vue/vue-hook.png)

|      生命周期钩子   |      详细     |
|:-----------------:|:----------------:|
| beforeCreate | 在实例初始化之后，数据观测(data observer) 和 event/watcher事件配置之前被调用。 |
| created | 在实例创建完成后被立即调用。在这一步,实例已经完成以下配置：数据观测（data observer），属性和方法的运算，watch/event事件回调。然而，挂载阶段还没有开始，<font color="blue">$el</font>属性目前不可见。 |
| beforeMount | 在挂载开始之前被调用：相关的 <font color="blue">render</font> 函数首次被调用。**该钩子在服务器端渲染期间不被调用。** |
| mounted | <font color="blue">el</font>被新创建的<font color="blue">vm.$el</font>替换，并挂载到实例上去之后调用该钩子。如果root实例挂载了一个文档内元素，当 mounted 被调用时 vm.$el 也在文档内。 <br/> 注意 mounted **不会** 承诺所有的子组件也一起被挂载。如果你希望等到整个视图都渲染完毕，可以用 vm.$nextTick 替换掉 mounted。 **该钩子在服务器端渲染期间不被调用。** |
| beforeUpdate | 数据更新时调用，发生在虚拟DOM打补丁之前。这里适合在更新之前访问现在的DOM，比如手动移除已添加的事件监听器。**该钩子在服务器渲染期间不被调用，因为只有初次渲染会在服务器端运行** |
| updated | 由于数据更改导致的虚拟dom重新渲染和打补丁，在这之后会调用该钩子。<br/> 当这个钩子被调用时，组件DOM已经更新，所以你现在可以执行依赖于DOM的操作。然后在大多数情况下，你应该避免在此期间更改状态。如果要相应的状态改变，通常最好使用 [<font color="blue">计算属性</font>](https://cn.vuejs.org/v2/api/#computed)或[<font color="blue">watcher</font>](https://cn.vuejs.org/v2/api/#watch)取而代之。 <br/> 注意 <font color="blue">updated</font> 不会承诺所有的子组件也都一起被重绘。如果你希望等到整个视图都重绘完毕，可以用 vm.$nextTick 替换掉 <font color="blue">updated</font>。 |
| beforeDestory | 实例销毁之前调用。在这一步，实例仍然完全可用。 **该钩子在服务器端渲染期间不被调用。** |
| destoryed | Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。 **该钩子在服务器端渲染期间不被调用。** |

