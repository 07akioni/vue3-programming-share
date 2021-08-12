---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
---

# Vue 3 新特性在组件库中的应用（下）

张乐聪 · 字节跳动 · 前端工程师

[07akioni · https://github.com/07akioni](https://github.com/07akioni)

<img style="position: absolute; left: 4%; top: 8%; height: 6%;" src="/assets/qcon.png" />

<img style="position: absolute; left: 0; right: 0; top: 0; bottom: 0; z-index: -1;" src="/assets/background.png"/>

---

# 目录

<div class="grid grid-cols-2 gap-x-4 mb-4">

<div>

### 上

- Composition API
- Fragment
- Teleport

</div>

<div>

### 下

- Emits Component Option
- `v-model`
- 静态优化带来的影响
- `inheritAttrs`
- TypeScript
- `createApp`
- 渲染函数 `h`
- 尚未解决的痛点

</div>

</div>

---

# 目录

- Emits Component Option

<div class="op5">

- `v-model`
- 静态优化带来的影响
- `inheritAttrs`
- TypeScript
- `createApp`
- 渲染函数 `h`
- 尚未解决的痛点

</div>

---

# Emits Component Option

<div class="grid grid-cols-2 gap-x-4 mb-4">

<div>

```ts
app.component({
  name: 'a-component',
  emits: ['foo', 'bar']
})
```

<v-click>

### 没必要用它

- `xxx` 几乎等同于 `onXxx`
- 缺乏类型

</v-click>

</div>

<div>

<v-click>

```ts
app.component({
  name: 'a-component',
  props: {
    onFoo: Function as PropType<() => void>,
    onBar: Function as PropType<() => void>
  }
})
```

### Props 够用了

- 类型完整

</v-click>

</div>

</div>

<v-click>

> Explicit is better than implicit.

</v-click>

---

# Emits Component Option

总结

建议别用

---

# 目录

<div class="op5">

- Emits Component Option

</div>

- `v-model`

<div class="op5">

- 静态优化带来的影响
- `inheritAttrs`
- TypeScript
- `createApp`
- 渲染函数 `h`
- 尚未解决的痛点

</div>

---

# `v-model`

### Vue 2

`v-model` => `value` + `@input`

在组件的使用 `model` 选项可以修改 prop key 和对应的事件。

```js
export default {
  model: {
    prop: 'foo',
    event: 'fooChange'
  }
}
```

`v-model` 只对这个属性生效。

对于其他属性只能使用 `v-bind:prop.sync` 和 `@update:prop`。

---

# `v-model`

### Vue 3

Vue 3 统一了 `v-model` 和 `.sync`。

`v-model:value`

- 输入值 `:value="xxx"`
- 改变值 `@update:value="e => { xxx = e }"`

在老版本中 `v-model` 和 `.sync` 的表现是割裂的。

现在由 `v-model` 提供了统一的范式。

---

# `v-model`

### 一些建议 1

`v-model` 的 API 提供了一种统一的“输入值、改变值”的范式

- 在 API 设计时遵守这一范式
  - 纯 React Style：`foo` & `onFooChange`
    - 一致性好，失去 `v-model` 的方便
  - 纯 Vue Style：`foo` & `onUpdate:foo`
    - 一致性好，`v-model` 灵活
    - 需要对 JSX 的使用场景优化， `onUpdate:foo` 不能作为 prop
  - 一半 React Style 一半 Vue Style
    - 为什么这样 ❓ 是因为天天都是交付 DDL 导致没空注意 API 的一致性 ❓

---

# `v-model`

### 一些建议 2

不要使用 `v-model` 的默认属性，即 `v-model:modelValue` 和 `@update:modelValue`。

使用 `v-model:value`。

<v-click>

> 显式比隐式更好。

或许你的用户很懒，但是你需要告诉他们正确的事情。

</v-click>

<v-click>

如果你这么用，那么当需要显式绑定值或者监听变化的时候，API 会很丑。

```html
<xxx :modelValue="x" /> <xxx @update:modelValue="x" />
```

什么是 modelValue？在不同的组件中并不一样。最好明确的让用户来使用。

</v-click>

---

# `v-model`

### JSX 的处理

针对 `onUpdate:foo` 不能作为 prop，个人建议写两个 prop。

```js
export default defineComponent({
  props: {
    'onUpdate:value': Function,
    onUpdateValue: Function
  }
})
```

---

# `v-model`

### 边界情况

回调的 key 重复需要专门处理。[vue-template-explorer](https://vue-next-template-explorer.netlify.app/#%7B%22src%22%3A%22%3Cxxx%20v-model%3Avalue%3D%5C%22x%5C%22%20%40update%3Avalue%3D%5C%22y%5C%22%3E%22%2C%22options%22%3A%7B%22mode%22%3A%22module%22%2C%22filename%22%3A%22Foo.vue%22%2C%22prefixIdentifiers%22%3Afalse%2C%22hoistStatic%22%3Afalse%2C%22cacheHandlers%22%3Afalse%2C%22scopeId%22%3Anull%2C%22inline%22%3Afalse%2C%22ssrCssVars%22%3A%22%7B%20color%20%7D%22%2C%22compatConfig%22%3A%7B%22MODE%22%3A3%7D%2C%22whitespace%22%3A%22condense%22%2C%22bindingMetadata%22%3A%7B%22TestComponent%22%3A%22setup%22%2C%22foo%22%3A%22setup%22%2C%22bar%22%3A%22props%22%7D%2C%22optimizeImports%22%3Afalse%7D%7D)

```html
<xxx v-model:value="x" @update:value="y"></xxx>
```

```js
props = {
  'onUpdate:value': [($event) => (_ctx.x = $event), _ctx.y]
}
```

所以请正确的定义 prop 类型。

```js
export default defineComponent({
  props: {
    'onUpdate:value': [Function, Array],
    onUpdateValue: [Function, Array]
  }
})
```

---

# `v-model`

### 总结

- 使用统一的 API 范式
- 避免使用默认的 `v-model`
- 正确的处理它在 JSX 中的使用
- 正确的处理边界情况

---

# 目录

<div class="op5">

- Emits Component Option
- `v-model`

</div>

- 静态优化带来的影响

<div class="op5">

- `inheritAttrs`
- TypeScript
- `createApp`
- 渲染函数 `h`
- 尚未解决的痛点

</div>

---

# 静态优化带来的影响

区块树

区块树是 Vue 3 引入的新概念，是在静态结构中的一些内容，用于加速渲染，例如跳过一些 Diff。

这是个底层概念，但是会影响到组件的编写。

看两个例子：

```html
<my-component>
  <div v-for="i in [0, 1, 2]" :key="i" />
  <!-- 一个区块 -->
</my-component>
```

```tsx
<MyComponent>
  {{ default: () => [0, 1, 2].map((v) => <div key={i} />) }}
</MyComponent>
```

---

# 静态优化带来的影响

区块树

在第一个例子中，`MyComponent` 收到的 `default` 插槽内容是一个 `Fragment`。

```html
<my-component>
  <div v-for="i in [0, 1, 2]" :key="i" />
  <!-- 一个区块 -->
</my-component>
```

```js
_createElementBlock(_Fragment, null, _renderList([0, 1, 2], (i) => {
  return _createElementVNode("div", { key: i })
})
```

在第二个例子中，`MyComponent` 收到的 `default` 插槽内容是一个 `VNode` 的数组。

```tsx
<MyComponent>
  {{ default: () => [0, 1, 2].map((v) => <div key={i} />) }}
</MyComponent>
```

---

# 静态优化带来的影响

区块树

如果你想要直接读取 `default` 插槽的信息，需要对这两种数据结构一起进行处理。

例如 Tabs：

```jsx
function render() {
  const {
    $slots: { default: defaultSlot }
  } = this
  const children = flatten(defaultSlot()) // 同时兼容两种数据结构
  return <div>
    <div class="tabs">
      {children.map(...)}
    </div>
    <div class="pane">
      {children.find(...)}
    </div>
  </div>
}
```

---

# 静态优化带来的影响

静态提升

Vue 3 的模板编译器会将静态节点提升到渲染函数外。你的组件会接受同一个 `VNode` 作为输入。

```html
<div class="item" v-for="i in 5" :key="i">
  <n-dropdown trigger="click" :options="options">
    <div>click</div>
  </n-dropdown>
</div>
```

```js
const _hoisted_1 = /*#__PURE__*/ _createElementVNode(
  'div',
  null,
  'click！',
  -1 /* HOISTED */
)

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  const _component_n_dropdown = _resolveComponent('n-dropdown')
  // ...
}
```

---

# 静态优化带来的影响

静态提升

避免去修改 `VNode`。

### Drropdown

```js
import { defineComponent } from 'vue'

defineComponent({
  name: 'Dropdown',
  render() {
    const child = this.$slots.default()[0]
    appendEvents(child) // ❌
    return (
      <>
        {child}
        <Teleport>
          <DropdownMenu />
        </Teleport>
      </>
    )
  }
})
```

---

# 静态优化带来的影响

静态提升

使用 `cloneVNode`。

### Drropdown

```js
import { defineComponent, cloneVNode } from 'vue'

defineComponent({
  name: 'Dropdown',
  render() {
    const child = cloneVNode(this.$slots.default()[0]) // ✅
    appendEvents(child)
    return (
      <>
        {child}
        <Teleport>
          <DropdownMenu />
        </Teleport>
      </>
    )
  }
})
```

---

# 静态优化带来的影响

总结

- 注意兼容区块树的产物
- 注意兼容静态提升的产物
  - `VNode` 应该被视作不可变的

---

# 目录

<div class="op5">

- Emits Component Option
- `v-model`
- 静态优化带来的影响

</div>

- `inheritAttrs`

<div class="op5">

- TypeScript
- `createApp`
- 渲染函数 `h`
- 尚未解决的痛点

</div>

---

# `inheritAttrs` 属性

&nbsp;

`inheritAttrs` 控制没有声明在 props 中的属性是否向组件传递。

- 在 React 中，对于自定义组件，除了 `key` 和 `ref`，任何属性不会被区别对待，包括 `style` 和 `className`。
- 在 Vue 2 中，`inheritAttrs` 不对 `style` 和 `class` 生效
  - 这限制了组件的编写的灵活性
- 在 Vue 3 中，`inheritAttrs` 对 `style` 也 `class` 生效

---

# `inheritAttrs` 属性

应用场景

### 多个根节点的组件

多个根节点的组件无法自动继承属性，需要手动绑定。

```html
<template>
  <input v-bind="$attrs" class="auto-complete-input" />
  <popmenu />
</template>

<script>
  export default defineComponent({
    name: 'AutoComplete',
    inheritAttrs: false
  })
</script>
```

---

# `inheritAttrs` 属性

应用场景

### JSX

在 JSX 中 `v-bind` 会转化为 `mergeProps` 函数。

```jsx
import { defineComponent, mergeProps, Fragment } from 'vue'

defineComponent({
  name: 'AutoComplete',
  inheritAttrs: false,
  render() {
    return (
      <>
        <input {...mergeProps(this.$attrs, { class: 'auto-complete' })} />
        <Popmenu />
      </>
    )
  }
})
```

---

# `inheritAttrs` 属性

总结

- 在多个根节点的组件有很大作用
- 在 JSX 中可以使用 `mergeProps` 达到 `v-bind` 的效果

---

# 目录

<div class="op5">

- Emits Component Option
- `v-model`
- 静态优化带来的影响
- `inheritAttrs`

</div>

- TypeScript

<div class="op5">

- `createApp`
- 渲染函数 `h`
- 尚未解决的痛点

</div>

---

# TypeScript

&nbsp;

为什么需要 TypeScript：

- 开发更安全
  - 自动提示更多
  -  重构更放心
  - 修改能发现错误
  - ...
- 用户使用更安全
  - 自动提示更多
  -  重构更放心
  - 使用能发现错误
  - ...

开发 Vue 3 项目不用 TS 的话，浪费了 Vue 3 和 TS 提供的能力。

---

# TypeScript

`ExtractPropTypes`

`ExtractPropTypes` 类型的使用值得专门讲一下。

它可以将一个 prop 对象的类型转化为 `setup` 函数中传入的 `props` 的类型。

每一个组件都需要导出一个属性类型供用户使用。

```ts
const buttonProps = {
  // ...
} as const

export type ButtonProps = ExtractPropTypes<typeof buttonProps>

export default Button = defineComponent({
  props: buttonProps,
  setup(props) {
    // <- props 的类型即为 ButtonProps
  }
})
```

---

# TypeScript

`ExtractPropTypes`

但是现在的写法还存在一些问题。在下面的例子中，`props.foo` 一定存在。

```ts
const buttonProps = {
  foo: {
    type: String,
    defualt: ''
  }
} as const

export type ButtonProps = ExtractPropTypes<typeof buttonProps> // 👀 props.foo 一定存在

export default Button = defineComponent({
  props: buttonProps,
  setup(props) {
    // <- props 的类型即为 ButtonProps
    props.foo // 👀 props.foo 一定存在
  }
})
```

但是，用户应该可以不填写 `foo` 的。

---

# TypeScript

`ExtractPropTypes`

但是现在的写法还存在一些问题。在下面的例子中，`props.foo` 一定存在。

```ts
const buttonProps = {
  foo: {
    type: String,
    defualt: ''
  }
} as const

export type ButtonProps = Partial<ExtractPropTypes<typeof buttonProps>> // 属性变得不是必填

export default Button = defineComponent({
  props: buttonProps,
  setup(props) {
    // <- props 的类型即为 ButtonProps
    props.foo // 👀 props.foo 一定存在
  }
})
```

修改后，用户可以不填写 `foo`。

---

# TypeScript

`ExtractPropTypes`

一个相关的建议：

- 任何**公开**组件都不应该有必填属性。
  - 组件可以直接渲染会减少用户的使用成本。

<v-click>

一个和相关建议相关的建议：

- 任何**公开组件**都需要有非受控模式。

支持非受控模式：

```html
<my-input default-value=""></my-input>
```

不支持非受控模式的话：

```html
<my-input v-model:value="xxx" />
<!-- 用户必须要在组件中定义 xxx，要写 v-model 组件才能正常交互 -->
```

</v-click>

---

# TypeScript

一个 TSX 的 hack

假设你想让 Button 组件在 TSX 中接受全部 `button` 标签接受的属性：

- 不可能将所有属性一一在 props 中声明
- `defineComponent` 的产物只接受声明的 props

```tsx
// ❌，没有在 MyButton 的 props 写明的属性都会报错
<MyButton onDragStart={onDragStart} />
```

### 利用 `ExtractPropTypes` 进行 hack

```ts
const ButtonProps = ExtractPropTypes(Button) // 获取 Button 的属性

type NativeButtonProps = Omit<ButtonHTMLAttributes, keyof ButtonProps> // 获取原生属性
type MergedProps = Partial<ButtonProps & NativeButtonProps> // 合并两种属性

export const XButton: new () => { $props: MergedProps } = Button as any // 构建 TSX 专用对象
```

```tsx
<XButton onDragStart={onDragStart} /> // ✅
```

---

# TypeScript

总结

- TypeScript 很有用
- `ExtractPropTypes` 很有用
  - 辅助生成组件的 props 类型
    - 建议所有公开组件不存在必填 prop
    - 建议所有空开组件支持非受控模式
  - Hack TSX

---

# 目录

<div class="op5">

- Emits Component Option
- `v-model`
- 静态优化带来的影响
- `inheritAttrs`
- TypeScript

</div>

- `createApp`

<div class="op5">

- 渲染函数 `h`
- 尚未解决的痛点

</div>

---

# `createApp`

### Vue 2 => Vue 3

任何全局改变 Vue 行为的 API 都会移动到应用实例上

```js
const app = createApp({})
```

### 两个常用点

- `Vue.component` => `app.component`
- `Vue.prototype` => `app.config.globalProperties`

---

# `createApp`

&nbsp;

Vue 如何寻找全局注册的组件？

### Vue 2

直接在 Vue 实例中寻找。

```js
h('my-component', {}, [])
```

### Vue 3

使用 `resolveComponent` 寻找，只寻找 App 中安装的内容。

```js
const myComponent = resolveComponent('my-component')

h(myComponent, {}, {})
```

---

# `createApp` 引申出的一个问题

&nbsp;

`$message.error()` 这类 API 是否足够好

### 在 Vue 2 中使用起来一切正常

```js
$message.error({
  content(h) {
    return h('my-component', {}, ['cool'])
  }
})
```

### 在 Vue 3 中呢？

```js
$message.error({
  content() {
    const myComponent = resolveComponent('my-component') // ❌ 找不到
    return h('my-component', {}, ['cool'])
  }
})
```

---

# `createApp` 引申出的一个问题

&nbsp;

`import { $message } from 'xxx'` 或者 `createApp().use($message)` 意味着什么？

<v-click>

- 单例
  - 每个修改都会影响它
  - 不同的应用使用引发冲突
- 脱离应用环境
  - 难以感知被调用时的环境
  - 定制难（暗色主题）

</v-click>

<v-click>

为什么 message 要作为 global、Vue 的一部分？🤔️

message 本应作为 App 的一部分。

常见的 API 不一定是正确的设计，常见的 API 设计和它被提出时的环境有关联，但是时间、环境会改变。

</v-click>

---

# `createApp` 引申出的一个问题

&nbsp;

将 message 组件化。

### `MessageProvider`

```html
<message-provider>
  <app />
</message-provider>
```

### 如何使其接受配置？

```html
<config-provider theme="dark">
  <message-provider :duration="1000">
    <app-1 />
  </message-provider>
  <message-provider>
    <app-2 />
  </message-provider>
</config-provider>
```

---

# `createApp` 引申出的一个问题

### 如何在 App 中使用 message

```js
import { useMessage } from 'xxx'

export default defineComponent({
  setup() {
    const { error } = useMessage()
  }
})
```

### `MessageProvider` 的实现

```html
<template>
  <teleport>
    <div><div v-for="message in messages" /></div>
  </teleport>
  <slot />
</template>

<script>
  provide('message', {
    // 省略 setup 的代码
    error: (content) => messages.push(content)
  })
</script>
```

---

# `createApp` 引申出的一个问题

&nbsp;

新的 API 具有的问题：

学习成本高，早期会有很多用户来问你怎么用，说原来多方便。

<v-click>

但是相比于用户方便：

1. 项目更好维护更加重要（人力有限，把时间花在更有价值的事情上）；
1. 用户正确的引导更加重要（基础的问题是无休止的）。

给用户正确的东西，而不是迁就。

</v-click>

<v-click>

也不能说没有例外：

除非他们给你付足够多的钱 💰💰💰💰💰💰💰

</v-click>

---

# `createApp`

总结

- 任何全局改变 Vue 行为的 API 都会移动到应用实例上
- Vue 3 使用 `resolveComponent` 寻找组件
- 使用 Provider 的范式将单例 API 组件化
  - 全局的单例 options 式的 API 会逐渐不能满足新的需求

---

# 目录

<div class="op5">

- Emits Component Option
- `v-model`
- 静态优化带来的影响
- `inheritAttrs`
- TypeScript
- `createApp`

</div>

- 渲染函数 `h`

<div class="op5">

- 尚未解决的痛点

</div>

---

# 渲染函数 `h`

### Vue 2

在 Vue 2 中，`h` 通过渲染函数暴露出来。

```js
render (h) {
  return h('div')
}
```

这导致很多的组件 API 中的渲染函数需要暴露同样的 `h`。

### Vue 3

```js
import { h } from 'vue'

render () {
  h('div')
}
```

在新的组件 API 不要再暴露 `h`。

---

# 渲染函数 `h`

总结

- `h` 可以从全局引入
- 在新的组件 API 不要再暴露 `h`

---

# 目录

<div class="op5">

- Emits Component Option
- `v-model`
- 静态优化带来的影响
- `inheritAttrs`
- TypeScript
- `createApp`
- 渲染函数 `h`

</div>

- 尚未解决的痛点

---

# 尚未解决的痛点

&nbsp;

当前 Vue 3 组件开发中的痛点：

- 组件 props 不支持泛型
- 组件 slots 缺乏类型支持

---

# 泛型

&nbsp;

为什么组件开发需要泛型？

```ts
function Select(props: {
  options: Array<{ label: string, value: string | number }>
  value: string | number
  onChange: (value: string | number, option: { label: string, value: string | number }) => void
}) {
  return ...
}
```

这样够吗？如果用户知道传入的选项全都是 `string` 作为 `value`，`onChange` 回调中的 `value` 还是 `string | number`。

`Table` 组件也类似，许多回调会传回对应的行数据。

---

# 泛型

&nbsp;

针对于回调参数，有一个临时解决方案，把 `|` 换成 `&`，但这不是没有代价的。

```ts
function Select(props: {
  options: Array<{ label: string, value: string | number }>
  value: string | number
  onChange: (value: string & number, option: { label: string, value: string & number }) => void
}) {
  return ...
}
```

`onChange` 可以接受更多的类型，例如：

- `(value: string) => void`
- `(value: number) => void`
- `(value: number | number) => void`

但是和 `options` 进行对应的任务交给了用户。

---

# 泛型

&nbsp;

这才是我们期望的类型：

```ts
function Select<Option>(props: {
  options: Option[]
  value: Option['value']
  onChange: (value: Option['value'], option: Option) => void
}) {
  return ...
}
```

然而目前 Vue 提供的 API 无法做到这点。

相关 RFC：https://github.com/vuejs/rfcs/pull/310

非 defineComponent，还在讨论中。

---

# 带类型的 Slots

### 目前的 Slot 类型

```ts
type Slot = (...args: any[]) => VNode[]
```

这个类型太过宽泛了。

假如我们希望对一个 AutoComplete 组件自定义输入框，并且希望对 input 事件做类型检查。

```html
<auto-complete #="{ onInput }">
  <my-input @input="onInput" />
  <auto-complete></auto-complete
></auto-complete>
```

`onInput` 是没有类型的。

---

# 尚未解决的痛点

总结

- Props 不支持泛型
- Slots 不支持类型
  - 自然也不支持泛型

这两项对于组件使用者的开发体验非常重要。

---

# 总结

- Emits Component Option
- `v-model`
- 静态优化带来的影响
- `inheritAttrs`
- TypeScript
- `createApp`
- 渲染函数 `h`
- 尚未解决的痛点

- 为什么用？怎么用？尽力选择最佳实践

---

# 案例复盘

- https://www.naiveui.com/
- 所有的错误实践都在这个项目中出现过

<img src="/assets/naive-ui.png" style="height: 80%;" />

---

# Take Away 的一句话

&nbsp;

把东西做好远远比把事情做对难

一百件对的事不等于一个好的产品

理想还是存在的，只不过追求理想，可能会很痛苦

---

<img src="/assets/thanks.png" style="position: absolute; left: 0; right: 0; top: 0; bottom: 0;" />
