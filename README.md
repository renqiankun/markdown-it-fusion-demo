[English](README.en.md) | [中文](README.md)

# markdown-it-vue-component

> 一个基于 Markdown-it 的插件，用于在 Markdown 中渲染 Vue 组件，支持流式渲染和自定义占位符。

#### [github](https://github.com/renqiankun/markdown-it-vue-component-demo)
#### [demo](https://renqiankun.github.io/markdown-it-vue-component-demo/dist/)
---

## 功能特点

- **高度可扩展**：打破 Markdown 限制，自由嵌入任意 Vue 组件，释放无限可能。
- **数据驱动**：组件数据由 JSON 、 String 、HTMLString定义，清晰直观，轻松实现动态内容。
- **优雅加载**：流式渲染不再突兀，通过占位符平滑过渡，体验如丝般顺滑。
- **精准控制** `getSegments` 接口，将渲染结果切分为 HTML 与组件，赋予你前所未有的控制力。
- **生态友好**： `markdown-it` 插件化设计，与现有生态简单集成，即插即用。
- **开放兼容**：轻松适配其他 `markdown-it` 组件库，告别重复造轮子，插件本身不依赖其他库，仅生成segments.
- **极致灵活**：完美兼容 `html` 开启/关闭双重模式，适应任何严苛的渲染环境。
- **智能缓存**：独有的 props 锁定机制，杜绝数据流末端的冗余刷新，确保高性能与高稳定性。



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
import { ref, watch ,shallowRef} from 'vue'
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
      // 此字段内部无任何处理，原样返回，可使用字符串，渲染时自己匹配
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
      <component v-if="item.type === 'component'" :is="item.component" v-bind="item.props" ></component>
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
  props?: { [key: string]: any ,_isComplete?: boolean}// 组件的 props
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
