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

# Vue 3 新特性在组件库中的应用（上）

张乐聪 · 字节跳动 · 前端工程师

[07akioni · https://github.com/07akioni](https://github.com/07akioni)

<img style="position: absolute; left: 4%; top: 8%; height: 6%;" src="/assets/qcon.png" />

<img style="position: absolute; left: 0; right: 0; top: 0; bottom: 0; z-index: -1;" src="/assets/background.png"/>

---

# Vue 3 新特性回顾

- Composition API
- Fragment
- Teleport
- Emits Component Option
- `v-model` arguments
- `createApp`
- 静态编译优化
- 更好的 TypeScript 支持
- `inheritAttrs` Component Option
- 单独使用的 `h` 函数
- ...

---

# 目录

对应于新特性的应用或造成的影响，本次分享内容包括以下部分

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

- Composition API
  - 逻辑复用
  - 性能优化
  - 类型支持

<div class="op5">

- Fragment
- Teleport

</div>

---

# Composition API

组合式 API

组合式 API 可以将同一个逻辑关注点相关代码收集在一起

对于组件库，相比于 Vue 2 的 Option API，Composition API 从很大程度上提升了逻辑密度，降低了维护成本，提供了更好的类型支持

### 应用

- 逻辑复用
- 性能优化
- 类型层面
  - ref, provide, inject, tsx

<div class="mt-4" v-click>

> 大多数情况，Composition API 的功能都可以使用 Options API 进行实现，但是更加复杂、可维护性低、类型支持差

</div>

---

# 目录

- Composition API
  - <u>逻辑复用</u>
  - 性能优化
  - 类型支持

<div class="op5">

- Fragment
- Teleport

</div>

---

# Composition API · 逻辑复用 1

Table，逻辑的拆分聚合

<div class="grid grid-cols-2 gap-x-4 mb-4">

<div>

### Table.vue (Vue 2)

```js
export default {
  data() {
    return {
      checkedKeys: [],
      expandedKeys: []
    }
  },
  methods: {
    updateCheckedKeys() {},
    updateExpandedKeys() {}
  }
}
```

<v-click>

逻辑分散于各种 Options 中，随着代码量增大会出现大量跳跃

</v-click>

</div>

<div>

### Table.vue (Vue 3)

```js
function useCheck() {
  const expandedKeysRef = ref([])
  return { expandedKeysRef, updateCheckedKeys() {} }
}

function useExpand() {
  const checkedKeysRef = ref([])
  return { checkedKeysRef, updateExpandedKeys() {} }
}

export default defineComponent({
  setup() {
    const { expandedKeysRef, updateCheckedKeys } = useCheck()
    const { checkedKeysRef, updateExpandedKeys } = useExpand()
    return {
      ...
    }
  }
})
```

</div>

</div>

---

# Composition API · 逻辑复用 2

Form Item，代替 Mixin

<div class="grid grid-cols-2 gap-x-4 mb-4">

<div>

```html
<script>
  const formItemMixin = {
    methods: {
      triggerFormItemUpdate () {
        ...
      }
    }
  }

  export default {
    mixins: [
      formItemMixin
    ],
    methods: {
      doUpdateValue () {
        this.triggerFormItemUpdate()
      }
    }
  }
</script>
```

</div>

<div>

Vue 2 跨组件逻辑复用的方式 `mixin`

- 动态合并
  - 类型不友好
  - 合并策略 implicit
  - Debug 困难
  - 间接暴露接口，无法直接了解做了什么

</div>

</div>

---

# Composition API · 逻辑复用 2

Form Item，代替 Mixin

<div class="grid grid-cols-2 gap-x-4 mb-4">

<div>

```html
<script>
  function useFormItem () {
    return {
      triggerFormItemUpdate () {
        ...
      }
    }
  }

  export default defineComponent({
    setup () {
      const { triggerFormItemUpdate } = useFormItem()
      return {
        doUpdateValue () {
          triggerFormItemUpdate()
        }
      }
    }
  })
</script>
```

</div>

<div>

Composition API

- 直接使用
  - 类型友好
  - Explicit 使用
  - Debug 友好

</div>

</div>

---

# Composition API · 逻辑复用

总结

- 逻辑的拆分聚合
- 代替 `mixin`
  - 请不要使用 `mixin`

---

# 目录

- Composition API
  - 逻辑复用
  - <u>性能优化</u>
  - 类型支持

<div class="op5">

- Fragment
- Teleport

</div>

---

# Composition API · 性能优化

&nbsp;

先讲一下性能优化的两个思路：

- 减少 DOM 操作
  - 最需要注意的是 Reflow
  - 不过这个和 Composition API 并没什么直接关联
- 减少渲染
  - 渲染不一定会导致 DOM 操作
  - 这个是重点

---

# Composition API · 性能优化 1

使用 `watchEffect` 减少渲染次数

假设我们有一个 500 行的 Table。

在滚动开始的时候会向 Table 应用一个类。

```js
export default defineComponent({
  setup() {
    const scrolledRef = ref(false)
    return {
      scrolled: scrolledRef,
      onScroll(e) {
        if (e.target.scrollTop === 0) scrolledRef.value = false
        else scrolledRef.value = true
      }
    }
  },
  render() {
    return (
      <div class={this.scrolled ? 'scrolled' : ''} onScroll={this.onScroll} />
    )
  }
})
```

---

# Composition API · 性能优化 1

使用 `watchEffect` 减少渲染次数

每次开始滚动时都要渲染一次（调用一次 render 函数）。

那么怎么避免这次渲染呢？

<v-click>

当然是直接操作 DOM 了 😅

</v-click>

<v-click>

**这不是常规性能优化，请在你确定瓶颈后进行专门的优化。**

具体的场景是某些小的改动会触发一个耗时的 render。

例如只改动一个小的类名要重新创造 500 个 VNode。

</v-click>

---

# Composition API · 性能优化 1

使用 `watchEffect` 减少渲染次数

```js
export default defineComponent({
  setup() {
    const selfElRef = ref(null)
    const scrolledRef = ref(false)
    watchEffect(() => {
      // 在这里修改 DOM
      selfElRef.value.className = scrolledRef.value ? 'scrolled' : ''
    })
    return {
      selfElRef,
      onScroll(e) {
        if (e.target.scrollTop === 0) scrolledRef.value = false
        else scrolledRef.value = true
      }
    }
  },
  render() {
    return <div ref="selfElRef" onScroll={this.onScroll} />
  }
})
```

---

# Composition API · 性能优化 1

使用 `watchEffect` 减少渲染次数

这究竟是个什么思路呢？

- 本质上是你代替 Vue 控制视图
  - 局部 Svelte 化
- 为什么要这么做
  - 某些改动不值得完整的渲染

<v-click>

为什么使用 `watchEffect`？

- `watch` 不行吗？
  - 可以，但是很难处理更复杂的场景
- `watchEffect` 自动收集依赖
  - `watch` 需要手动指明依赖
- 我们看一个真实 Case

</v-click>

---

# Composition API · 性能优化 1

使用 `watchEffect` 减少渲染次数

```js
watchEffect(() => {
  const { value: leftActiveFixedColKey } = leftActiveFixedColKeyRef
  const { value: rightActiveFixedColKey } = rightActiveFixedColKeyRef
  if (
    !fixedStyleMounted &&
    leftActiveFixedColKey === null &&
    rightActiveFixedColKey === null
  ) {
    return
  }
  // 挂载相关样式
  fixedStyleMounted = true
})
```

涉及多个依赖使用 `watchEffect` 效果好于 `watch`。

---

# Composition API · 性能优化 2

优化 Checkbox Group 性能

### CheckboxGroup API

```html
<template>
  <CheckboxGroup :value="['a', 'b']">
    <Checkbox value="a" />
    <Checkbox value="b" />
    <Checkbox value="c" />
  </CheckboxGroup>
</template>
```

---

# Composition API · 性能优化 2

优化 Checkbox Group 性能

### CheckboxGroup 实现

<div class="grid grid-cols-2 gap-x-4">

<div>

```ts
defineComponent({
  name: 'CheckboxGroup',
  props: ['value'],
  setup(props) {
    provide('checkbox-group', {
      valueRef: toRef(props, 'value')
    })
  }
})
```

</div>

<div>

```ts
defineComponent({
  name: 'Checkbox',
  props: ['value'],
  setup(props) {
    const { valueRef } = inject('checkbox-group')
    return {
      checked: computed(() => {
        return valueRef.value.includes(props.value)
      })
    }
  }
})
```

</div>

</div>

<div v-click class="mt-4">

`computed` 有一个特点，只要其依赖的值更改，即使计算结果不更改，也会触发重新渲染

</div>

---

# Composition API · 性能优化 2

优化 Checkbox Group 性能

### `computed` 导致多余渲染

<div class="grid grid-cols-2 gap-x-4 mb-4">

<div>

`value="['a', 'b']"`

```html
<template>
  <CheckboxGroup :value="['a', 'b']">
    <Checkbox value="a" />
    <Checkbox value="b" />
    <Checkbox value="c" />
  </CheckboxGroup>
</template>
```

</div>

<div>

`value="['a']"`

```html
<template>
  <CheckboxGroup :value="['a']">
    <Checkbox value="a" />
    <!--  RERENDER   -->
    <Checkbox value="b" />
    <!-- DOM CHANGED -->
    <Checkbox value="c" />
    <!--  RERENDER   -->
  </CheckboxGroup>
</template>
```

然而 `'a'`、`'c'` 本可以不 rerender 的

</div>

</div>

---

# Composition API · 性能优化 2

优化 Checkbox Group 性能

解决方案

结合 `ref`、`computed`、`watchEffect`

### ref 不会导致重新渲染

```ts
const valueRef = ref(0)
valueRef.value = 1
valueRef.value = 1 // 不会导致重新渲染
valueRef.value = 1 // 不会导致重新渲染
```

### useMemo

```ts
function useMemo(getter) {
  const computedValueRef = computed(getter)
  const valueRef = ref()
  watchEffect(() => {
    valueRef.value = computedValueRef.value
  })
  return valueRef
}
```

---

# Composition API · 性能优化 2

优化 Checkbox Group 性能

### 使用 UseMemo 实现 Checkbox

<div class="grid grid-cols-2 gap-x-4">

<div>

```ts{7-9}
defineComponent({
  name: 'Checkbox',
  props: ['value'],
  setup (props) {
    const { valueRef } = inject('checkbox-group')
    return {
      checked: computed(() => {
        return valueRef.value.includes(props.value)
      })
    }
  }
})
```

</div>

<div>

```ts{7-9}
defineComponent({
  name: 'Checkbox',
  props: ['value'],
  setup (props) {
    const { valueRef } = inject('checkbox-group')
    return {
      checked: useMemo(() => {
        return valueRef.value.includes(props.value)
      })
    }
  }
})
```

</div>

</div>

---

# Composition API · 性能优化 2

优化 Checkbox Group 性能

<img src="/assets/memo-checkbox.png" style="margin: auto; height: 60%;" />

---

# Composition API · 性能优化 2

优化 Checkbox Group 性能

### 1000 Checkbox

为了放大性能差异，每个 `render` 中都带一个 `console.log`

<div class="grid grid-cols-2 gap-x-4 mb-4">

<div>

### Computed

<img src="/assets/computed-perf.png" />

</div>

<div>

### useMemo

<img src="/assets/memo-perf.png" />

</div>

</div>

[https://codesandbox.io/s/memo-large-data-98jni](https://codesandbox.io/s/memo-large-data-98jni)

---

# Composition API · 性能优化 2

优化 Checkbox Group 性能

使用 mixin？🤮

```js
function createMemoMixin(fieldName, getter) {
  return {
    computed: {
      [`computed-${fieldName}`]: getter
    },
    watch: {
      [`computed-${fieldName}`]: function (value) {
        this[fieldName] = value
      }
    },
    created() {
      this[fieldName] = this[`computed-${fieldName}`]
    },
    data() {
      return { fieldName: null } // ?
    }
  }
}
```

---

# Composition API · 性能优化 3

再提 `useMemo`

我们回到第一个例子，我当时特地挑了一个性能好的写法。

```js
export default defineComponent({
  setup() {
    const scrolledRef = ref(false)
    return {
      scrolled: scrolledRef,
      onScroll(e) {
        if (e.target.scrollTop === 0) scrolledRef.value = false
        else scrolledRef.value = true
      }
    }
  },
  render() {
    return (
      <div class={this.scrolled ? 'scrolled' : ''} onScroll={this.onScroll} />
    )
  }
})
```

---

# Composition API · 性能优化 3

再提 `useMemo`

让我们换一个性能差的。

```js
export default defineComponent({
  setup() {
    const scrolledTopRef = ref(0)
    return {
      scrollTop: scrolledTopRef,
      onScroll(e) {
        scrolledTopRef.value = e.target.scrollTop
      }
    }
  },
  render() {
    return (
      <div
        class={this.scrollTop !== 0 ? 'scrolled' : ''}
        onScroll={this.onScroll}
      />
    ) // 每一次滚动都要重新渲染
  }
})
```

---

# Composition API · 性能优化 3

再提 `useMemo`

换成 `computed`，一样差。

```js
export default defineComponent({
  setup() {
    const scrolledTopRef = ref(0)
    return {
      scrolled: computed(() => scrollTopRef.value !== 0),
      onScroll(e) {
        scrolledTopRef.value = e.target.scrollTop
      }
    }
  },
  render() {
    return (
      <div class={this.scrolled ? 'scrolled' : ''} onScroll={this.onScroll} />
    ) // 每一次滚动都要重新渲染
  }
})
```

---

# Composition API · 性能优化 3

再提 `useMemo`

换成 `useMemo`，只在 `scrolled` 改变时渲染。

https://github.com/07akioni/vooks/blob/master/src/use-memo.ts

```js
export default defineComponent({
  setup() {
    const scrolledTopRef = ref(0)
    return {
      scrolled: useMemo(() => scrollTopRef.value !== 0),
      onScroll(e) {
        scrolledTopRef.value = e.target.scrollTop
      }
    }
  },
  render() {
    return (
      <div class={this.scrolled ? 'scrolled' : ''} onScroll={this.onScroll} />
    ) // 每一次滚动都要重新渲染
  }
})
```

---

# Composition API · 性能优化

总结

- 使用 `watchEffect` 手动调整 DOM
  - 减少不必要渲染
  - 非常规优化，谨慎
- 使用 `useMemo` 避免 `computed` 的不足之处
  - `computed` 不管值变不变都会导致重新渲染
  - `useMemo` 不是 React 里面那个，是我们包装的一个新的 `computed`
    - 只有值变了才重新渲染

---

# 目录

- Composition API
  - 逻辑复用
  - 性能优化
  - <u>类型支持</u>

<div class="op5">

- Fragment
- Teleport

</div>

---

# Composition API · 更好的类型支持

Component 暴露的 Ref 类型

Input 组件具有 focus 方法

```html
<template>
  <input ref="inputInstRef" />
</template>

<script>
  interface InputInst {
    focus: () => void;
  }

  export default defineComponent(() => {
    const inputInstRef = ref<InputInst\|null>( null)
    return {
      inputInstRef
    }
  })
</script>
```

<v-click>

如何在类型层面保证 Input 组件在实现时 focus 方法类型正确？🤔️

</v-click>

---

# Composition API · 更好的类型支持

Component 暴露的 Ref 类型

在 setup 中增加类型约束

```ts
interface InputInst {
  focus: () => void
}

defineComponent({
  name: 'Input',
  setup() {
    const exposedMethods: InputInst = {
      focus: ...
    }
    return {
      ...exposedMethods
    }
  }
})
```

<v-click>

保证用户使用的接口类型安全

</v-click>

---

# Composition API · 更好的类型支持

Component 暴露的 Ref 类型

你会发现之前的方式（在 Runtime 层面）有两点不足：

1. 暴露了不该暴露的属性
2. 在 setup 返回函数时没用

不过我们有了新的 RFC，已经通过

https://github.com/vuejs/rfcs/blob/master/active-rfcs/0042-expose-api.md

```js
export default defineComponent({
  setup(props, { expose }) {
    // public ref props
    expose({})
  }
})
```

---

# Composition API · 更好的类型支持

`provide` & `inject`

使 `provide` 和 `inject` 类型安全。

首先，你需要有一个类型。

比如

```ts
type CoolPerson = '07akioni'
```

```ts
provide<CoolPerson>('coolPerson', '07akioni') // ✅
provide<CoolPerson>('coolPerson', '08akioni') // ❌
```

```ts
const coolPerson = inject<CoolPerson>('coolPerson')
// typeof coolPerson = '07akioni'
```

虽然类型要自己加上去，看起来好像安全点了。

可是还有个问题，这个 `'coolPerson'` 完全是手写的，万一写错了。

---

# Composition API · 更好的类型支持

`provide` & `inject`

使用 `InjectionKey` 类型，`InjectionKey` 拓展了 `Symbol`。

```ts
type CoolPerson = '07akioni'

const coolPersonInjectionKey: InjectionKey<CoolPerson> = Symbol('coolPerson')
```

```ts
provide(coolPersonInjectionKey, '07akioni') // ✅
provide(coolPersonInjectionKey, '08akioni') // ❌
```

```ts
const coolPerson = inject(coolPersonInjectionKey)
// typeof coolPerson = '07akioni'
```

`coolPersonInjectionKey` 所携带的类型会被 `provide` 和 `inject` 读取，不需要重复书写。

同时这是 `Symbol` 保证不会用错。

---

# Composition API · 更好的类型支持

总结

- ref 安全的类型
  - 编写技巧
  - `expose` RFC
- `provide`, `inject`
  - 使用 `InjectionKey` 类型

---

# 目录

<div class="op5">

- Composition API
  - 逻辑复用
  - 性能优化
  - 类型支持

</div>

- Fragment

<div class="op5">

- Teleport

</div>

---

# Fragment

在根部渲染多个节点

<div class="grid grid-cols-2 gap-x-4 mb-4">

<div>

### Segment.vue（根部多个元素）

```html
<template>
  <li>Kirby</li>
  <li>07akioni</li>
  <li>Zhang Lecong</li>
</template>
```

</div>

<div>

### 根部多个 Slot

```html
<template>
  <slot name="a" />
  <slot name="b" />
</template>
```

</div>

</div>

- 不再需要唯一的根结点
- 单一或多个根 Slot
- 和 Teleport 结合可以制作更复杂的组件

---

# Fragment

总结

使用多个节点作为根节点。

---

# 目录

<div class="op5">

- Composition API
  - 逻辑复用
  - 性能优化
  - 类型支持
- Fragment

</div>

- Teleport

---

# Teleport

将子节点渲染到存在于父组件以外的 DOM 节点

实现 Modal

<div class="grid grid-cols-2 gap-x-4">

<div>

### Modal.vue（不使用 Teleport）

```html
<template>
  <div
    class="modal"
    style="position: fixed; left: 100px; right: 100px; top: 100px; bottom: 100px;"
  >
    Modal
  </div>
</template>
```

</div>

<div>

### 优点

- 简单
  - 写特定页面完全可以使用

### 缺点

- `overflow: hidden` 的祖先节点

</div>

</div>

---

# Teleport

Vue 2 中的实现

<div class="grid grid-cols-2 gap-x-4">

<div>

### Modal.vue (Vue 2)

```html
<template>
  <div>
    <div ref="bodyEl" ...>Modal</div>
  </div>
</template>

<script>
  export default {
    mounted() {
      const { bodyEl } = this.$refs
      bodyEl.parentNode.removeChild(bodyEl)
      document.body.appendChild(bodyEl)
    },
    beforeDestroy() {
      document.body.removeChild(bodyEl)
    }
  }
</script>
```

</div>

<div>

### 优点

- 适应性好

### 缺点

- 难以维护
- 难以定义卸载位置
- 需要占位节点
  - 可以利用 functional 组件 hack，难以维护

</div>

</div>

---

# Teleport

Vue 3 中的实现

<div class="grid grid-cols-2 gap-x-4">

<div>

### Modal.vue (Vue 3)

```html
<template>
  <Teleport to="body">
    <div ...>Modal</div>
  </Teleport>
</template>
```

</div>

<div>

### 优点

- 适应性好
- 简单可维护

</div>

</div>

---

# Teleport

总结

实现 Modal 的常用套路。

---

# Fragment + Teleport

结合 Fragment 和 Teleport 可以做更多事情

实现 Popover 组件

<div class="mb-4">

### Popover API

```html
<Popover>
  <template #trigger>
    <button>Trigger</button>
  </template>
  Content
</Popover>
```

</div>

- 两个 Slot
  - Trigger Slot
  - Default Slot

---

# Fragment + Teleport

基本实现

<div class="grid grid-cols-2 gap-x-4">

<div>

### Popover.vue (不使用 Fragment 和 Teleport)

```html
<template>
  <div class="popover-trigger" style="position: relative;">
    <slot name="trigger" />
    <div class="popover" style="position: absolute; ...;">
      <slot />
    </div>
  </div>
</template>
```

</div>

<div>

### 优点

- 简单

### 缺点

- 定位依赖于 trigger 的样式，不稳定
- `overflow: hidden` 的祖先节点

</div>

</div>

---

# Fragment + Teleport

Vue 2 中的实现

<div class="grid grid-cols-2 gap-x-4">

<div>

### Popover.vue (Vue2)

```html{all|2-7}
<template>
  <div class="popover-trigger">
    <slot name="trigger" />
    <div class="popover" ref="popoverEl">
      <slot />
    </div>
  </div>
</template>

<script>
export default {
  mounted () {
    // detach popoverEl
  },
  beforeDestroy () {
    // unmount popoverEl
  }
}
</script>
```

</div>

<div>

### 优点

- 适应性好

### 缺点

- 触发节点需要 wrapper，影响样式
  - 可以利用 functional 组件 hack，难以维护

</div>

</div>

---

# Fragment + Teleport

Vue 3 中的实现

<div class="grid grid-cols-2 gap-x-4">

<div>

### Popover.vue (Vue3)

```ts
defineComponent(() => {
  render () {
    const triggerNode = this.$slots.trigger()[0]
    const contentNode = this.$slots.default()
    appendEvents(triggerNode) // click or hover
    return [
      triggerNode,
      h(Teleport, {
        to: 'body'
      }, [
        h('div', { class: 'popover' }, contentNode)
      ])
    ]
  }
})
```

</div>

<div>

### 优点

- 适应性好
- 触发节点不需要 wrapper
- 可维护性好

### 缺点

- 需要渲染函数
  - 都写底层组件了，你还介意这个吗？

</div>

</div>

---

# Fragment + Teleport

总结

Popover 和其他弹出组件的标准套路。

---

# 总结

- Composition
  - 逻辑复用
  - 性能优化
  - 类型支持
- Fragment
- Teleport

All in 新特性

---

# 案例复盘

- https://www.naiveui.com/
- 所有的错误实践都在这个项目中出现过

<img src="/assets/naive-ui.png" style="height: 80%;" />


---

# Take Away 的一句话

&nbsp;

正确的限制 = 更大的自由

- 寻找正确的方法
- 使用 Good Parts
- 清晰架构
- 正确的限制能让工具人早点下班

---

<img src="/assets/thanks.png" style="position: absolute; left: 0; right: 0; top: 0; bottom: 0;" />
