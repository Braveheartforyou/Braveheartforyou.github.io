---
title: 深入element-ui组件 input组件
date: 2019-08-21 15:27:13
tags: [Vue]
categories: [Vue]
description:  element-ui中的input怎么实现值的双向绑定、自适应高度的
---

***察见渊鱼者不详，智料隐匿者有殃。——《列子·说符》***

## 简介

---

在日常开发`PC`管理端界面的时候，会用到**element-ui**或者**iview**ui框架，比较常用的`input`组件是怎么封装的呢，以`element-ui`框架的`input`组件为例，分析一下它是怎么封装实现的，也可以为以后自己封装框架提供思路。

因为代码还是挺多的，主要分析几个点：

- 支持**前置内容**、**后置内容**、**后置元素**
- 支持所有原声的`type`，并且可以切换`password`模式
- 支持`readonly`、`disabled`、`autocomplete`、`maxlength`、`minlength`等等
- 支持常规`input`、`focus`、`blur`、`change`，支持新事件`compositionstart`、`compositionupdate`、`compositionend`
- `element-ui`是怎么做到类似与`vue`的双向绑定的
- `type`为`textare`模式时，做到`calcTextareaHeight`

下面就一步一步分下，主要的点会着重分析，比较简单或者比较不重要的点会快速带过。

源码参考[element-ui input](https://github.com/ElemeFE/element/blob/dev/packages/input/src/input.vue)

## 前置、后置

```html
  <template v-if="type !== 'textarea'">
    <!-- 前置元素 -->
    <div class="el-input-group__prepend" v-if="$slots.prepend">
      <slot name="prepend"></slot>
    </div>
    <!-- 前置内容 -->
    <span class="el-input__prefix" v-if="$slots.prefix || prefixIcon">
      <slot name="prefix"></slot>
      <i class="el-input__icon"
          v-if="prefixIcon"
          :class="prefixIcon">
      </i>
    </span>

    // 主题代码 省略
    <input/>
    <!-- 后置内容 -->
    <span
      class="el-input__suffix"
      v-if="getSuffixVisible()">
      // 省略内容
    </span>
    <!-- 后置元素 -->
    <div class="el-input-group__append" v-if="$slots.append">
      <slot name="append"></slot>
    </div>
  </template>
  <textarea v-else>
    // 省略内容
  </textarea/>
```

上面代码可以首先通过`type`区分开`textarea`单独处理，**前后置**又分为两类：

- 一类直接通过`prop`传入`prefix-icon`、`suffix-icon`传入前后置`icon-class`显示不同的内容
- 另一类是通过`vue`中的内容分发机制`slot`显示不同的内容，更灵活

在后置内容中也会处理`clearable`、`PwdVisible`、`show-word-limit`显示不同的内容。

## 原声type、其它原声属性

以`input`为例，代码如下：

```html
  // 其它地方省略
  <input
    :tabindex="tabindex"
    v-if="type !== 'textarea'"
    class="el-input__inner"
    v-bind="$attrs"
    :type="showPassword ? (passwordVisible ? 'text': 'password') : type"
    :disabled="inputDisabled"
    :readonly="readonly"
    :autocomplete="autoComplete || autocomplete"
    ref="input"
    @compositionstart="handleCompositionStart"
    @compositionupdate="handleCompositionUpdate"
    @compositionend="handleCompositionEnd"
    @input="handleInput"
    @focus="handleFocus"
    @blur="handleBlur"
    @change="handleChange"
    :aria-label="label"
  >
  // 其它地方省略
  <script>
    export default {
      props: {
        // 其它地方省略
        disabled: Boolean,
        readonly: Boolean,
        autocomplete: {
          type: String,
          default: 'off'
        },
        /** @Deprecated in next major version */
        autoComplete: {
          type: String,
          validator(val) {
            process.env.NODE_ENV !== 'production' &&
              console.warn('[Element Warn][Input]\'auto-complete\' property will be deprecated in next major version. please use \'autocomplete\' instead.');
            return true;
          }
        }
        // 其它地方省略
      }
    }
  </script>
```

这里先不关注事件，只关注属性的设置，`element-ui`支持所有`input`原声的属性，其实就是通过`prop`传入`type`传入，如果是非`password`就直接赋值给`input`标签的`type`属性。
如果是`password`类型判断是否传入`show-password`字段，根据传入判断给`input`的`type`复制为`password`或者`text`。

## 其它原声属性

其它属性如`disabled`、`readonly`、`autocomplete`都是通过显示的`prop`传入，哪像一些没有显示接受的`prop`怎么获取到的呢，比如说`maxlength`、`minlength`这种没有显示接收的怎么获取到的呢。
是通过`v-bind="$attrs"`获取到的，使用的时候直接通过`this.$attrs.XXXX`就可以使用对应的属性。`element-ui`中在检测输入长度是否超过设置的长度是有使用到如下：

```javascript
  // 忽略部分代码
  isWordLimitVisible() {
    return this.showWordLimit &&
      this.$attrs.maxlength &&
      (this.type === 'text' || this.type === 'textarea') &&
      !this.inputDisabled &&
      !this.readonly &&
      !this.showPassword;
  },
  upperLimit() {
    return this.$attrs.maxlength;
  },
  textLength() {
    if (typeof this.value === 'number') {
      return String(this.value).length;
    }
    return (this.value || '').length;
  },
  inputExceed() {
    // show exceed style if length of initial value greater then maxlength
    return this.isWordLimitVisible &&
      (this.textLength > this.upperLimit);
  }
  // 忽略部分代码
```

如果不太了解`v-bind="$attrs"`的使用可以看我另一篇文章[Vue中v-model解析、sync修饰符解析](/blog/vue/vue-vModel-sync.html)

## 原声事件、双向绑定

支持常规`input`、`focus`、`blur`、`change`，支持新事件`compositionstart`、`compositionupdate`、`compositionend`。

常用的时间就不用多做介绍了，就是比较新事件`compositionstart`、`compositionupdate`、`compositionend`是用来出里一段文字输入的事件。详细信息请看：
[compositionstart](https://developer.mozilla.org/zh-CN/docs/Web/Events/compositionstart)
[compositionupdate](https://developer.mozilla.org/zh-CN/docs/Web/Events/compositionupdate)
[compositionend](https://developer.mozilla.org/zh-CN/docs/Web/Events/compositionend)

在外层就可以监听的到`input`标签的`input`、`focus`、`blur`、`change`事件，是因为组件内部在每一个方法中都有一个`this.$emit('input/focus/blur/change', evnet.target.value/event)`。

那是怎么做到修改外层通过`v-model`绑定的值的呢？

要想了解他是怎么实现双向绑定的就要了解`v-model`是什么，`v-model`其实就是一个语法糖，在`vue`编译阶段就会解析为`:value="绑定的值"`和默认的`@input=(value) => {绑定的值 = value}`。然后再是在`input`标签上绑定的`handleInput`方法如下：

```javascript
    handleInput(event) {
      // should not emit input during composition
      // see: https://github.com/ElemeFE/element/issues/10516
      if (this.isComposing) return;
      // hack for https://github.com/ElemeFE/element/issues/8548
      // should remove the following line when we don't support IE
      if (event.target.value === this.nativeInputValue) return;
      this.$emit('input', event.target.value);
      // ensure native input value is controlled
      // see: https://github.com/ElemeFE/element/issues/12850
      this.$nextTick(this.setNativeInputValue);
    }
```

到这里`input`的组件的属性基本上就解释完成，`textare`基本上都是一样的，但是`textarea`有一点特殊的操作。

## textarea高度自适应

它是怎么实现高度自适应的，具体的实现代码是在[calcTextareaHeight](https://github.com/ElemeFE/element/blob/dev/packages/input/src/calcTextareaHeight.js)，大致分为几部：

- 创建一个临时隐藏的`textarea`元素
- 通过隐藏的`textarea`计算显示的`textarea`元素的高度
- 并且判断`minRows`、`maxRows`属性
- 最后删除`textarea`元素，并且清空`dom`引用

在`input`组件中通过创建`watch`监听`value`值的变化，每次`value`变化从新计算`textarea`元素的高度。

**创建一个临时隐藏的`textarea`元素**

```javascript
let hiddenTextarea;
// 省略部分代码
// 保证只执行一次
if (!hiddenTextarea) {
  // 创建textarea
  hiddenTextarea = document.createElement('textarea');
  // 添加到页面中
  document.body.appendChild(hiddenTextarea);
}
// 省略部分代码
```

**通过隐藏的`textarea`计算显示的`textarea`元素的高度**

```javascript
const HIDDEN_STYLE = `
  height:0 !important;
  visibility:hidden !important;
  overflow:hidden !important;
  position:absolute !important;
  z-index:-1000 !important;
  top:0 !important;
  right:0 !important
`;

const CONTEXT_STYLE = [
  'letter-spacing',
  'line-height',
  'padding-top',
  'padding-bottom',
  'font-family',
  'font-weight',
  'font-size',
  'text-rendering',
  'text-transform',
  'width',
  'text-indent',
  'padding-left',
  'padding-right',
  'border-width',
  'box-sizing'
];
function calculateNodeStyling(targetElement) {
  const style = window.getComputedStyle(targetElement);

  const boxSizing = style.getPropertyValue('box-sizing');

  const paddingSize = (
    parseFloat(style.getPropertyValue('padding-bottom')) +
    parseFloat(style.getPropertyValue('padding-top'))
  );

  const borderSize = (
    parseFloat(style.getPropertyValue('border-bottom-width')) +
    parseFloat(style.getPropertyValue('border-top-width'))
  );

  const contextStyle = CONTEXT_STYLE
    .map(name => `${name}:${style.getPropertyValue(name)}`)
    .join(';');

  return { contextStyle, paddingSize, borderSize, boxSizing };
}
// 省略部分代码
let {
    paddingSize,
    borderSize,
    boxSizing,
    contextStyle
} = calculateNodeStyling(targetElement);
// 设置属性
hiddenTextarea.setAttribute('style', `${contextStyle};${HIDDEN_STYLE}`);
// 设置value
hiddenTextarea.value = targetElement.value || targetElement.placeholder || '';
// 获取隐藏textarea滚动整体高度
let height = hiddenTextarea.scrollHeight;
const result = {};
// 判断盒模型
if (boxSizing === 'border-box') {
  // 如果为border-box本身的高度加上borderSize
  height = height + borderSize;
} else if (boxSizing === 'content-box') {
  // 如果为content-box本身的高度加上paddingSize
  height = height - paddingSize;
}
// 把隐藏textarea的vlaue设置为空
hiddenTextarea.value = '';
// 获取每行高度： row的高度为滚动高度-padding
let singleRowHeight = hiddenTextarea.scrollHeight - paddingSize;
// 省略部分代码
```

**并且判断`minRows`、`maxRows`属性**

```javascript
// 省略部分代码
  // 判断是否有设置minRows
  if (minRows !== null) {
    // 最小高度 = 每行高度 * 最小行数
    let minHeight = singleRowHeight * minRows;
    if (boxSizing === 'border-box') {
      // 如果boxSizing为border-box时： 最小高度 = 原最小高度 + padding + border;
      minHeight = minHeight + paddingSize + borderSize;
    }
    // 取当前计算的最小高度和上面计算出的scrollHeight度中的最大值
    height = Math.max(minHeight, height);
    // 赋值给result.minHeight
    result.minHeight = `${ minHeight }px`;
  }
  // 判断是否有设置maxRows
  if (maxRows !== null) {
    // 最大高度 = 每行高度 * 最大行数
    let maxHeight = singleRowHeight * maxRows;
    if (boxSizing === 'border-box') {
      // 如果boxSizing为border-box时： 最小高度 = 原最小高度 + padding + border;
      maxHeight = maxHeight + paddingSize + borderSize;
    }
    // 取当前计算的最大高度和上面计算出的scrollHeight度中的最小值
    height = Math.min(maxHeight, height);
  }
  // 赋值给result.minHeight
  result.height = `${ height }px`;
  // 省略部分代码
```

**最后删除`textarea`元素，并且清空`dom`引用**

```javascript
  // 如果存在parentNode 就清除parentNode包含的hiddenTextarea 元素
  hiddenTextarea.parentNode && hiddenTextarea.parentNode.removeChild(hiddenTextarea);
  // 清空dom引用释放变量
  hiddenTextarea = null;
  // 返回最小高度和最大高度
  return result;
```

到此`textarea`结束，可能还有很多细节没有记录到，如果有什么意见请评论。

## 总结

整篇文章都是根据几点分析的，如下：

- 支持**前置内容**、**后置内容**、**后置元素**
- 支持所有原声的`type`，并且可以切换`password`模式
- 支持`readonly`、`disabled`、`autocomplete`、`maxlength`、`minlength`等等
- 支持常规`input`、`focus`、`blur`、`change`，支持新事件`compositionstart`、`compositionupdate`、`compositionend`
- `element-ui`是怎么做到类似与`vue`的双向绑定的
- `type`为`textare`模式时，做到`calcTextareaHeight`

当然还有很多的点没有记录到比如说`clear`、`emitter`、`Migrating`等等，这个会在后面的文章中着重介绍。

## 参考

[element-ui input](https://github.com/ElemeFE/element/blob/dev/packages/input/src/input.vue)
[element-ui calcTextareaHeight](https://github.com/ElemeFE/element/blob/dev/packages/input/src/calcTextareaHeight.js)
