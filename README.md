[English](README.en.md) | [中文](README.md)

# markdown-it-vue-component

> 一个基于 Markdown-it 的 Vue3 插件，用于在 Markdown 中渲染 Vue 组件，支持流式渲染和自定义占位符。

#### [github](https://github.com/renqiankun/markdown-it-vue-component-demo)
#### [demo](https://renqiankun.github.io/markdown-it-vue-component-demo/dist/)
---

## 功能特点

*   **强大的组件化能力**
    *   **任意组件注入**：在 Markdown 中无缝渲染任何自定义  vue、react 组件，打破静态文本的限制。
    *   **标准类 HTML 语法**：采用直观的 `<tag>...</tag>` 风格，完整支持标签属性、自闭合 (`<tag />`) 及隐式自闭合 (`<img>`，`<input>`)，学习成本极低。
    *   **灵活的数据传递**：支持将标签内容作为普通文本或 JSON字符串（参数可控制转为json）传递给组件，轻松实现数据驱动。
*   **为流式渲染而生 (Streaming-First)**
    *   **优雅的加载占位**：在数据流未完成时可控制显示加载状态或直接渲染组件，无组件闪烁和布局抖动，极大提升用户体验。
    *   **智能状态感知**：组件能明确获知数据流是否结束，从而执行数据加载完毕后的特定逻辑（如代码高亮、图表渲染等）。
    *   **性能缓存机制**：组件数据流完成后，组件 Props 将被智能锁定，杜绝组件不必要的重复渲染，确保高性能和组件状态稳定性。
*   **结构化输出**
    *   **结构化输出 (getSegments)**：将渲染结果拆分为 **HTML 片段**和**待挂载的 组件**列表，不限制vue、react渲染
*   **高兼容性**
    *   **基于 markdown-it 插件**：不影响 markdown-it 核心功能或其他插件的正常运作。
    *   **双模式兼容**：无论 markdown-it 的 html 选项开启或关闭，插件均能提供稳定一致的解析能力。
    *   **易于扩展**：架构清晰，可经简单修改后与其他基于 markdown-it 的组件库协同工作。


---

## 安装

```bash
npm install markdown-it-vue-component
# 或者使用 yarn
yarn add markdown-it-vue-component
```

---

## 使用方法

```ts
import { ref, watch ，shallowRef} from 'vue'
import MarkdownIt from 'markdown-it'
import customComponentPlugin , { type MDVueComponentOptions ,type SegmentsResultItem} from 'markdown-it-vue-component'

import MyComponent from './my-component.vue'

const md = MarkdownIt()

//完整配置
md.use(customComponentPlugin, {
  debug: false,
  propsKey: '_data',
  placeholderClass: 'custom-placeholder',
  components: {
    'my-component': {
      component: shallowRef(MyComponent),
      renderIntermediate: false,
      propsUseJson: true,
      multipleProps: true,
      propsKey: '_data',
      placeholderClass: 'custom-placeholder'
    } as MDVueComponentOptions
  }
})
// 极简配置
md.use(customComponentPlugin, {
  components: {
    'my-component': { component: shallowRef(MyComponent) } 
  }
})

// 渲染 Markdown
const segments = ref<Array<SegmentsResultItem>>([])

const renderHtml = (markdownText: string) => {
  const html = md.render(markdownText)
  segments.value = md.getSegments(html)
}

onMounted(() => {
   renderHtml('# Hello Markdown <my-component> { "type": "Hello" } </my-component> ')
})

```

---

## 渲染到 Vue 模板

```vue
<template>
  <div>
    <template v-for="item in segments" :key="item.id">
      <!-- 普通 HTML 段 -->
      <div v-if="item.type === 'html'" v-html="item.content"></div>
      <!-- Vue 组件段 -->
      <component
        v-if="item.type === 'component'"
        :is="item.component"
        v-bind="item.props"
      ></component>
    </template>
  </div>
</template>
```

---

## 插件选项说明

### MDVueComponentPluginOptions

| 参数             | 类型                                  | 默认值               | 说明                     |
| ---------------- | ------------------------------------- | -------------------- | ------------------------ |
| components       | Record<string, MDVueComponentOptions> | 必填                 | 注册的组件列表           |
| placeholderClass | string                                | 'custom-placeholder' | 占位符的 class           |
| propsKey         | string                                | '\_data'             | 组件接收数据的 prop 名称 |
| debug            | boolean                               | false                | 是否开启调试模式         |

### MDVueComponentOptions

| 参数               | 类型    | 默认值               | 说明                                  |
| ------------------ | ------- | -------------------- | ------------------------------------- |
| component          | any     | 必填                 | Vue3 组件                             |
| renderIntermediate | boolean | false                | 数据未完全接收前是否渲染组件          |
| loadingText        | string  | '加载中...'          | 占位符显示文本                        |
| propsUseJson       | boolean | false                | 数据是否尝试解析为 JSON               |
| multipleProps      | boolean | false                | JSON 解析成功后是否解构传入多个 props |
| propsKey           | string  | '\_data'             | 组件接收数据的 prop 名称              |
| placeholderClass   | string  | 'custom-placeholder' | 占位符 class                          |

---

## 获取段落列表

```ts
const segments = md.getSegments(html)
```

返回的每个段落对象格式如下：

```ts
{
  type: 'html' | 'component', // html 段或组件段
  id: string | number,
  content?: string,            // html 段的内容
  component?: any,             // Vue 组件
  props?: {
    [key: string]: any,
    /**标签上属性 */
    _attrs?: Record<string, any>,
    /**数据是否接收完成 */
    _isComplete?: boolean
  }
}
```

---

## 注意事项

- 插件依赖 Vue 3 与 Markdown-it
- propsUseJson: true 时，如果 JSON 解析失败，会将数据作为字符串传入
- 流式渲染场景下，可通过 renderIntermediate 控制组件未完成数据接收时的占位行为
- 每一个内容展示区域为一个markdown-it的实例

---

## 许可证

MIT
