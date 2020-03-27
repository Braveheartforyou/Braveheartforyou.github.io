---
title: 深入Vue系列 ———— Vue中v-model、sync、$attre/$lisrener、provied/inject修饰符解析
date: 2019-08-29 20:22:18
tags: [Vue]
categories: [Vue]
description: Vue中v-model解析、sync修饰符解析、$attr、$listeners
---

*上善若水，水善利萬物而不爭。——《道德經》*

## 简介

---

在平时开发是经常用到一些父子组件通信，经常用到`props`、`vuex`等等，这里面记录另外的三种方式`v-model`、`sync`是怎么使用，再说是怎么实现，其实`v-model`、`sync`都是语法糖。还有`$attr`、`$listener`实现父子组件通信。

## 使用方式

---

## v-model

> 2.2.0+ 新增

`v-mode1`其实就是一个语法糖，默认会利用名为`value`的`props`和名为`input`的事件，但是像单选框、复选框等类型的输入龙剑可能会讲`value`特性用于不同的目的。

`v-model`的使用场景：当**子组件**需要改变父组件通过`props`传入的值

**_父组件_**

- 父组件通过`v-model`绑定值
- 如需根据`v-model`传入的值改变，而触发其他更新请通过`watch`传入的值

**_子组件_**

- 声明`model`对象 设置事件`event`和`prop`字段
- 通过`porps`接受父组件传送值
- 修改是通过`this.$emit`广播事件

代码示例：

**_父组件代码_**

```javascript
<template>
  <children v-model="message"></children>
</template>
<script>
import children from "./children.vue";
export default {
  components: {
    children
  },
  data() {
    return {
      message: "parent"
    };
  },
  watch: {
    // 监听message变化
    message(newV, oldV) {
      console.log(newV, oldV);
    }
  }
};
</script>

```

**_子组件代码_**

```javascript
<template>
  <h1>{{ message }}</h1>
</template>
<script>
export default {
  model: {
    prop: "message", //这个字段，是指父组件设置 v-model 时，将变量值传给子组件的 msg
    event: "input" //这个字段，是指父组件监听 parent-event 事件
  },
  props: {
    message: String //此处必须定义和model的prop相同的props，因为v-model会传值给子组件
  },
  mounted() {
    //这里模拟异步将msg传到父组件v-model，实现双向控制
    setTimeout(_ => {
      this.$emit("input", "children");
      //将这个值通过 emit 触发parent-event，将some传递给父组件的v-model绑定的变量
    }, 1500);
  }
};
</script>

```

上面这个示例是通过`v-model`实现的，下面不通过`v-model`实现同样效果。

### 不使用 v-model 实现

代码示例如下：

**_父组件代码修改_**

```javascript
<template>
  <Children :message="message" @input="(event) => { message = event }"/>
</template>
<script>
// 不变
</script>

```

**_子组件代码修改_**

```javascript
<template>
  // 不变
</template>
<script>
export default {
  props: {
    message: String
  },
  mounted() {
    setTimeout(() => {
      this.$emit("input", "children");
    }, 1500);
  }
};
</script>

```

只是把`v-model`拆分为`props`和`@input`事件，子组件不需要配置`model`,只需要接受`props`和通过`this.$emit`广播事件就可以。
当然这个相对于`v-model`方法比较简便，但是灵活度查很多，选择使用那种看个人喜好。
在线地址：

<iframe src="https://codesandbox.io/embed/vue-template-zcvn3?fontsize=14" title="Vue Template" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

## sync

> 2.3.0+ 新增

在有些情况下，我们可能需要对一个 `prop` 进行**_“双向绑定”_**。不幸的是，真正的**双向绑定**会带来维护上的问题，因为子组件可以修改父组件，且在父组件和子组件都没有明显的改动来源。

这也是为什么我们推荐以 `update:myPropName` 的模式触发事件取而代之。同时也可以通过`sync`修饰符来实现。

在上面代码的基础上大致修改如下：

***父组件***

- 通过修改触发事件`input`为`update:myPropName`实现相同效果

***子组件***

- 通过修改`this.$emit(update:myPropName)`

代码如下：

***父组件代码修改***

```javascript
  // 修改如下
  <Children :message="message" @update:input="(event) => { message = event }"/>
```

***子组件代码修改***

```javascript
  // 其他不变
  this.$emit("update:input", "children");
```

### sync实现

上面的代码可以通过`sync`简写为下面代码：

***父组件代码修改***

```javascript
  // 修改如下
  <Children :messag.sync="message"/>
```

***子组件代码修改***

```javascript
  // 其他不变
  this.$emit("update:messag", "children");
```

同时`sync`也支持对象，要配合`v-bind`实现可以简写为`:`，但是要注意这个对象如下两条：

> 注意带有 `.sync` 修饰符的 `v-bind` 不能和表达式一起使用 (例如 `v-bind:title.sync=”doc.title + ‘!’”` 是无效的)。取而代之的是，你只能提供你想要绑定的属性名，类似 `v-model`。
> 将 `v-bind.sync` 用在一个字面量的对象上，例如 `v-bind.sync=”{ title: doc.title }”`，是无法正常工作的，因为在解析一个像这样的复杂表达式的时候，有很多边缘情况需要考虑。

## $attrs、$listeners

### $attrs

> 2.4.0 新增

- 类型：`{ [key: string]: string }`
- 只读
- 详细：
包含了父作用域中不作为 `prop` 被识别 (且获取) 的特性绑定 (`class` 和 `style` 除外)。当一个组件没有声明任何 `prop` 时，这里会包含所有父作用域的绑定 (`class` 和 `style` 除外)，并且可以通过 `v-bind="$attrs"` 传入内部组件——在创建高级别的组件时非常有用。

### $listeners

> 2.4.0 新增

- 类型：`{ [key: string]: Function | Array<Function> }`
- 只读
- 详细：
包含了父作用域中的 (不含 `.native` 修饰器的) `v-on` 事件监听器。它可以通过 `v-on="$listeners"` 传入内部组件——在创建更高层次的组件时非常有用。

### 实现通信

实现父子组件通信

***父组件代码***

```javascript
<template>
  <div class="parent">
    <Children
      :message="message"
      @upDate="upDate"
      type="del"
      @input="(event) => { message = event }"
    />
  </div>
</template>

<script>
import Children from "./Children";
export default {
  components: {
    Children
  },
  data() {
    return {
      message: "parent",
      type: "del"
    };
  },
  methods: {
    upDate (event) {
      console.log(event);
      this.type = event;
    }
  },
  watch: {
    message: function() {
      console.log("更新message值为" + this.message);
    }
  }
};
</script>
```

***子组件代码***

```javascript
<template>
  <div v-bind="$attrs" v-on="$listeners" class="children">{{message}} <span @click="$listeners.upDate('data')">{{$attrs.type}}</span></div>
</template>

<script>
export default {
  props: {
    message: String
  },
  mounted() {
    // console.log(this.$attrs);
    // console.log(this.$listeners);
    setTimeout(() => {
      this.$emit("input", "children");
      this.$emit('upDate', 'add')
    }, 1500);
  }
};
</script>
```

同时`$attrs`、`$listeners`都是可以跨域父子组件，可以父子子子组件传递，类似于`react`中的`context`，只是一部分设计理念相同。

## 总结

其实就是检测到`.sync`修饰符，在`complier`阶段会编译生成多个`prop`，生成多个`事件`。其实像这个**指令**、**修饰符**、**自定义指令**都是在`vue`编译是解析成为**v8**能执行的代码。

无论是`vue`、`babel`、`react`的`complier`编译阶段大致分为三个阶段：

- 通过词法解析`parse`生成抽象`AST`或`抽象代码树`
- 优化`AST`,比如`vue`标记静态节点，`babal`中抽取静态代码，这个阶段被称为`optimize`或者`优化AST树`
- 在`AST`代码的阶段上，生成可执行代码，这个过程可以叫做`codegen`

`v-model`、`sync`都可以实现父子组件通信，并且可以在子组件中修改父组件传入的值。在平常看法的时候进场可以用到这两种方式，具体选择那种方式看个人喜好。在`element-ui`这个`input`组件也用到相关的属性。

## 参考

> [$attrs](https://cn.vuejs.org/v2/api/#vm-attrs)
> [自定义组件的-v-model](https://cn.vuejs.org/v2/guide/components-custom-events.html#自定义组件的-v-model)
> [sync-修饰符](https://cn.vuejs.org/v2/guide/components-custom-events.html#sync-修饰符)
