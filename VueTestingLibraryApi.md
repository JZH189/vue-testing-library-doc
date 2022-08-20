# 简介

Vue Testing Library 建立在 DOM Testing Library 之上，通过添加用于使用 Vue components 的 API。它建立在@vue/test-utils之上，这是Vue的官方测试库。

简而言之，Vue 测试库做了三件事：
- 从 DOM 测试库中重新导出查询实用程序和帮助程序。
- 隐藏与测试库指导原则冲突@vue/测试使用方法。
- 调整两个来源的一些方法。

#### 安装

Vue2

```js
npm install --save-dev @testing-library/vue@5

```

Vue3

```js
npm install --save-dev @testing-library/vue

```

现在，您可以使用 DOM 测试库的所有 getBy、getAllBy、queryBy 和 queryAllBy 命令。请参阅此处的[完整查询列表](https://testing-library.com/docs/queries/about#types-of-queries)。

您可能还有兴趣安装@testing库/jest-dom，以便您可以为DOM使用[自定义Jest匹配器](https://github.com/testing-library/jest-dom#readme)。

#### 功能

Vue 测试库是一个非常轻量级的解决方案，用于测试 Vue 组件。它在@vue/test-utils之上提供了轻量级的实用程序功能，鼓励更好的测试实践。

其主要指导原则是：
>您的测试与软件的使用方式越相似，它们能给您的信心就越大。

因此，您的测试将处理实际的 DOM 节点，而不是处理渲染的 Vue 组件的实例。

此库提供的实用程序有助于以与用户相同的方式查询 DOM。它们允许您通过标签文本查找元素，从其文本中查找链接和按钮，并断言您的应用程序是可访问的。

它还公开了一种通过 data-testid 查找元素的推荐方法，作为文本内容和标签没有意义或不实用的元素的“转义舱口”。

#### 例子

```html
<template>
  <div>
    <p>Times clicked: {{ count }}</p>
    <button @click="increment">increment</button>
  </div>
</template>

<script>
  export default {
    data: () => ({
      count: 0,
    }),

    methods: {
      increment() {
        this.count++
      },
    },
  }
</script>
```

```js
import {render, fireEvent} from '@testing-library/vue'
import Component from './Component.vue'

test('increments value on click', async () => {
  // The render method returns a collection of utilities to query your component.
  // render 方法返回用于查询组件的实用工具的集合。
  const {getByText} = render(Component)

  // getByText returns the first matching node for the provided text, and throws an error if no elements match or if more than one match is found.
  //getByText 返回所提供文本的第一个匹配节点，如果没有匹配的元素或找到多个匹配项，则会引发错误。
  getByText('Times clicked: 0')

  const button = getByText('increment')

  // Dispatch a native click event to our button element.
  // 分发原生点击事件到我们的按钮元素。
  await fireEvent.click(button)
  await fireEvent.click(button)

  getByText('Times clicked: 2')
})
```


#### 一个v-model的示例

```html
<template>
  <div>
    <p>Hi, my name is {{ user }}</p>

    <label for="username">Username:</label>
    <input v-model="user" id="username" name="username" />
  </div>
</template>

<script>
  export default {
    data: () => ({
      user: 'Alice',
    }),
  }
</script>
```

```js
import {render, fireEvent} from '@testing-library/vue'
import Component from './Component.vue'

test('properly handles v-model', async () => {
  const {getByLabelText, getByText} = render(Component)

  // Asserts initial state.
  //断言初始状态。
  getByText('Hi, my name is Alice')

  // Get the input DOM node by querying the associated label.
  //通过查询关联的标签来获取输入 DOM 节点。
  const usernameInput = getByLabelText(/username/i)

  // Updates the <input> value and triggers an `input` event.
  // fireEvent.input() would make the test fail.

  //更新该<input>值并触发“输入”事件。fireEvent.input（） 会使测试失败。
  await fireEvent.update(usernameInput, 'Bob')

  getByText('Hi, my name is Bob')
})
```

#### 更多的例子

您将在[测试目录](https://github.com/testing-library/vue-testing-library/tree/main/src/__tests__)中找到使用不同库进行测试的示例。

- [vuex](https://github.com/testing-library/vue-testing-library/blob/main/src/__tests__/vuex.js)
- [vue-router](https://github.com/testing-library/vue-testing-library/blob/main/src/__tests__/vue-router.js)
- [vue-validate](https://github.com/testing-library/vue-testing-library/blob/main/src/__tests__/validate-plugin.js)
- [vue-i18n](https://github.com/testing-library/vue-testing-library/blob/main/src/__tests__/translations-vue-i18n.js)
- [uetify](https://github.com/testing-library/vue-testing-library/blob/main/src/__tests__/vuetify.js)

#### API

Vue Testing Library 从 DOM Testing Library 中重新导出所有内容。

它还公开了以下方法：

- [render(Component, options)](#render)
  - [Parameters](#Parameters)
    - [Component](#Component)
    - [Options](#Options)
  - [render result](#renderResult)
    - [...queries](#queries)
    - [container](#container)
    - [baseElement](#baseElement)
    - [debug](#debug)
    - [unmount](#unmount)
    - [html](#html)
    - [emitted](#emitted)
    - [rerender](#rerender)
- [fireEvent](#fireEvent)
  - [touch(elem)](#touch)
  - [update(elem, value)](#update)
- [cleanup](#cleanup)


##### <a id="render">render(Component, options)</a>

render 是在 Vue 测试库中渲染组件的唯一方法。

它最多需要 2 个参数，并使用一些帮助器方法返回一个对象。

```js
function render(Component, options, callbackFunction) {
  return {
    ...DOMTestingLibraryQueries,
    container,
    baseElement,
    debug(element),
    unmount,
    html,
    emitted,
    rerender(props),
  }
}
```
##### <a id="Parameters">Parameters</a>

- <a id="Component">Component</a>

要测试的有效 Vue 组件。

- <a id="Options">Options</a>

包含要传递到@vue/test-utils mount 的附加信息的对象。

此外，还可以提供以下选项：

- store (Object | Store)

Vuex 存储的对象定义。如果传递了 store，Vue Testing Library 将导入并配置 Vuex 存储。

- routes (Array | VueRouter)

Vue Router 的一组路由。如果提供了 routes，库将导入并配置 Vue Router。如果实例化的 routes 被传递，它将被使用。

- props (Object)

它将与propsData合并。

- container (HTMLElement)

默认情况下，Vue Testing Library 将创建一个 div 并将其附加到 baseElement。这是组件的渲染位置。如果您通过此选项提供自己的 HTMLElement 容器，它不会自动追加到 baseElement。

例如：如果要对 tablebody 元素进行单元测试，则它不能是 div 元素的子元素。在这种情况下，可以指定一个 table 元素作为渲染容器。

```js
const table = document.createElement('table')

const {container} = render(TableBody, {
  props,
  container: document.body.appendChild(table),
})
```

- baseElement (HTMLElement)

如果指定了 container 参数，则容器默认为 baseElement，否则默认为 document.body。baseElement 用作查询的基本元素，以及使用 debug（） 时打印的内容。

##### <a id="renderResult">render result</a>

render 方法返回一个具有几个属性的对象：

- <a id="queries">...queries</a>

render 最重要的功能是，来自 [DOM Testing Library](https://testing-library.com/docs/queries/about) 的查询将自动返回，其第一个参数绑定到 baseElement，默认为 document.body。

有关[Queries](https://testing-library.com/docs/queries/about)，请参阅查询。

```js
const {getByLabelText, queryAllByTestId} = render(Component)
```

- <a id="container">container</a>

渲染的 Vue 组件的容器 DOM 节点。默认情况下，它是一个 div。这是一个常规的 DOM 节点，因此您可以调用 container.querySelector 等来检查子节点。

>提示： 要获取所呈现元素的根元素，请使用 container.firstChild。

>如果您发现自己使用 container 来查询呈现的元素，那么您应该重新考虑！其他查询旨在提高对将要对正在测试的组件所做的更改的弹性。避免使用容器来查询元素！

- <a id="baseElement">baseElement</a>

在容器中渲染 Vue 组件的容器节点。如果未在呈现选项中指定 baseElement，则默认为 document.body。

当您要测试的组件在容器 div 外部呈现某些内容时。例如，当您想要快照测试门户组件时，这非常有用。该组件直接在正文中渲染其 HTML。

>注意：渲染返回的查询会查看 baseElement，因此您可以使用查询来测试门户组件，而无需 baseElement。

- <a id="debug">debug(element)</a>

此方法是 console.log（prettyDOM（element））的快捷方式。

```js
import {render} from '@testing-library/vue'

const HelloWorldComponent = {
  template: `<h1>Hello World</h1>`,
}

const {debug} = render(HelloWorldComponent)
debug()
// <div>
//   <h1>Hello World</h1>
// </div>
```

这是一个关于prettyDOM的简单包装器，它被公开并来自 [DOM Testing Library](https://testing-library.com/docs/dom-testing-library/api-debugging/#prettydom).

- <a id="unmount">unmount()</a>

@vue/test-utils [destroy](https://v1.test-utils.vuejs.org/api/wrapper/#destroy) 的别名。

- <a id="html">html()</a>

@vue/test-utils [html](https://v1.test-utils.vuejs.org/api/wrapper/#html) 的别名。

- <a id="emitted">emitted()</a>

@vue/test-utils [emitted](https://v1.test-utils.vuejs.org/api/wrapper/#emitted) 的别名。

- <a id="rerender">rerender(props)</a>

@vue/test-utils [setProps](https://test-utils.vuejs.org/api/#setprops) 的别名。

返回一个 Promise, 因此你可以使用 await rerender(...).

##### <a id="fireEvent">fireEvent</a>

由于 Vue 在重新渲染期间异步应用 DOM 更新，因此 fireEvent 工具将作为异步函数重新导出。若要确保正确更新 DOM 以响应测试中的事件，建议始终等待 fireEvent。

```js
await fireEvent.click(getByText('Click me'))

```

此外，Vue 测试库公开了两种有用的方法：

- <a id="touch">touch(elem)</a>

它同时触发 ocus() 和 blur（）事件。


```js
await fireEvent.touch(getByLabelText('username'))

// Same as:
await fireEvent.focus(getByLabelText('username'))
await fireEvent.blur(getByLabelText('username'))
```

- <a id="update">update(elem, value)</a>

正确处理由 v-model 控制的输入。它更新 input/select/textarea 内部值，同时发出相应的原生事件。

请参阅 [v-model示例测试](https://testing-library.com/docs/vue-testing-library/examples#example-using-v-model) 中的更新工作示例。


##### <a id="cleanup">cleanup</a>

卸载使用 render 挂载的 Vue 树。

>如果您使用的是支持 afterEach hook 的环境（如在 Jest 中），则无需手动调用清理。Vue Testing Library会为你处理它。

在调用 render 时未能调用 cleanup 可能会导致内存泄漏和测试不等效（这可能导致测试中难以调试错误）。