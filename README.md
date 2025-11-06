[English](README.en.md) | [中文](README.md)

# markdown-it-vue-component

> 一个基于 Markdown-it 的 Vue3 插件，用于在 Markdown 中渲染 Vue 组件，支持流式渲染和自定义占位符。

#### [github](https://github.com/renqiankun/markdown-it-vue-component-demo)
#### [demo](https://renqiankun.github.io/markdown-it-vue-component-demo/dist/)
---

## 功能特点

- 支持在 Markdown 中插入 Vue 组件
- 支持组件数据 JSON 化传递
- 支持组件占位符和加载状态
- 支持获取渲染后的 HTML 与组件段落列表 (`getSegments`)
- 基于markdown-it的插件

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
import { ref, watch } from 'vue'
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
      component: MyComponent,
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
    'my-component': { component: MyComponent } 
  }
})

// 渲染 Markdown
const segments = ref<Array<SegmentsResultItem>>([])

const renderHtml = (markdownText: string) => {
  const html = md.render(markdownText)
  segments.value = md.getSegments(html)
}

onMounted(() => {
  renderHtml('# Hello Markdown')
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
  props?: any                  // 组件的 props
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
