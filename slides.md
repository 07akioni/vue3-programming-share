---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
---

# Vue 3 新特性在组件库中的应用

[07akioni · 张乐聪](https://github.com/07akioni)

---

# Vue 3 新特性回顾

- Composition API
- Fragment
- Teleport
- Emits Component Option
- `v-model` arguments
- ...

---

# 目录

- **Composition API**
  - 单一组件层面的应用
  - 跨多组件层面的应用

<div class="opacity-40">

- Fragment
- Teleport
- Emits Component Option
- `v-model` arguments
- ...

</div>

---

# Composition API

组合式 API

组合式 API 可以将同一个逻辑关注点相关代码收集在一起

对于组件库，相比于 Vue 2 的 Option API，Composition API 从很大程度上提升了逻辑密度，降低了维护成本

### 应用

- 单一组件层面

  - 逻辑的拆分与聚合
  - 更好的类型支持

- 跨组件层面
  - 跨组件逻辑复用

<div class="mt-4" v-click>

> 大多数情况，Composition API 的功能都可以使用 Options API 进行实现，但是更加复杂、可维护性低

</div>

---

# Composition API · 逻辑的拆分与聚合

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
    const inputInstRef = (ref < InputInst) | (null > null)
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

# Composition API · 跨组件逻辑复用 1

复用 Form Item 逻辑

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

# Composition API · 跨组件逻辑复用 1

复用 Form Item 逻辑

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
  - Debug 困难

</div>

</div>

---

# Composition API · 跨组件逻辑复用 2

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

# Composition API · 跨组件逻辑复用 2

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

# Composition API · 跨组件逻辑复用 2

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

# Composition API · 跨组件逻辑复用 2

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

# Composition API · 跨组件逻辑复用 2

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

# Composition API · 跨组件逻辑复用 2

优化 Checkbox Group 性能

<img src="/assets/memo-checkbox.png" style="margin: auto; height: 50vh;" />

---

# Composition API · 跨组件逻辑复用 2

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

# Composition API · 跨组件逻辑复用 2

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

# 目录

<div class="opacity-40">

- Composition API

</div>

- **Fragment**

<div class="opacity-40">

- Teleport
- Emits Component Option
- `v-model` arguments
- ...

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

# 目录

<div class="opacity-40">

- Composition API
- Fragment

</div>

- **Teleport**

<div class="opacity-40">

- Emits Component Option
- `v-model` arguments
- ...

</div>

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

# 目录

<div class="opacity-40">

- Composition API
- Fragment
- Teleport

</div>

- Emits Component Option

<div class="opacity-40">

- `v-model` arguments
- ...

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

# 目录

<div class="opacity-40">

- Composition API
- Fragment
- Teleport
- Emits Component Option

</div>

- **`v-model` arguments**

<div class="opacity-40">

- ...

</div>

---

# `v-model` arguments

双向绑定的语法糖

`v-model` 的 API 提供了一种统一的“输入值、改变值”的范式

`v-model:value`

- 输入值 `:value="xxx"`
- 改变值 `@update:value="e => { xxx = e }"`

在 API 设计时我们可以遵守这一范式
