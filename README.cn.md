[English](README.md) | [中文](README.cn.md)

# 💠 markdown-it-fusion

> 一款强大的 `markdown-it` 插件，用于在 Markdown 中无缝渲染 Vue/React 组件或替换原生 HTML 标签。专为流式渲染（Streaming）和组件化设计，支持自定义占位符和灵活的数据传递。

| | |
| :--- | :--- |
| **GitHub 仓库** | [🔗 renqiankun/markdown-it-fusion-demo](https://github.com/renqiankun/markdown-it-fusion-demo) |
| **在线演示** | [🚀 Live Demo](https://renqiankun.github.io/markdown-it-fusion-demo/dist/) |

---

## ✨ 功能特性

-   **🚀 强大的组件化能力**
    -   **任意组件注入**：在 Markdown 中无缝渲染任何自定义 Vue 或 React 组件，打破静态文本的限制。
    -   **标准类 HTML 语法**：采用直观的 `<tag>...</tag>` 风格，完整支持标签属性、自闭合 (`<tag />`) 及隐式自闭合 (`<img>`)，学习成本极低。
    -   **灵活的数据传递**：支持将标签内容作为普通文本或 JSON 字符串传递给组件，轻松实现数据驱动。

-   **🌊 为流式渲染而生 (Streaming-First)**
    -   **优雅的加载占位**：在数据流未完成时，可选择显示加载状态或提前渲染组件，避免组件闪烁和布局抖动，极大提升用户体验。
    -   **智能状态感知**：组件能明确获知数据流是否结束，从而执行数据加载完毕后的特定逻辑（如代码高亮、图表渲染等）。
    -   **高性能缓存**：数据流完成后，组件 Props 将被智能锁定，杜绝不必要的重复渲染，确保高性能和组件状态稳定。

-   **🧩 结构化输出**
    -   **分段式结果 (`useSegments`)**：将渲染结果拆分为 **HTML 片段**和**待挂载的组件**列表，让你可以自由地在任何框架（Vue, React, Svelte 等）中进行渲染。

-   **🔧 高兼容性与扩展性**
    -   **原生插件**：基于 `markdown-it` 插件体系，不影响其核心功能或其他插件的运作。
    -   **双模式兼容**：无论 `markdown-it` 的 `html` 选项开启或关闭，插件均能提供稳定一致的解析能力。
    -   **易于扩展**：架构清晰，可轻松修改以与其他基于 `markdown-it` 的库协同工作。

---

## 📦 安装

```bash
# 使用 npm
npm install markdown-it-fusion

# 或者使用 yarn
yarn add markdown-it-fusion
```

---

## 🚀 使用方法

### 1. 初始化插件

首先，在你的 `markdown-it` 实例中引入并使用插件。

```ts
import MarkdownIt from 'markdown-it'
import mdFusion, {
  useSegments,
  useInstanceId,
  destroy,
  type MDComponentOptions,
  type SegmentsResultItem
} from 'markdown-it-fusion'
import 'markdown-it-fusion/style.css'
// 你的 Vue 或 React 组件
const MyComponent = { /* ... 组件定义 ... */ }

const md = MarkdownIt()

// 完整配置示例
md.use(mdFusion, {
  debug: false,
  propsKey: '_data',
  placeholderClass: 'custom-placeholder',
  components: {
    'my-component': {
      // vue下推荐使用shallowRef(MyComponent)
      component: MyComponent,
      renderIntermediate: false,
      propsUseJson: true,
      multipleProps: true,
      propsKey: '_data',
      placeholderClass: 'custom-placeholder'
    } as MDComponentOptions
  }
})

// 极简配置示例
md.use(mdFusion, {
  components: {
    'my-component': { component: MyComponent }
  }
})
```

### 2. 在 Vue 中渲染

使用 `useSegments` 辅助函数来获取 HTML 片段和组件列表，然后在模板中动态渲染。

```vue
<template>
  <div class="content">
    <template v-for="item in segments" :key="item.id">
      <div v-if="item.type === 'html'" v-html="item.content"></div>
      <component v-else-if="item.type === 'component'" :is="item.component" v-bind="item.props" />
    </template>
  </div>
</template>

<script setup>
import { ref, computed, onBeforeUnmount ,shallowRef} from 'vue'
import MarkdownIt from 'markdown-it' // 引入 MarkdownIt
import { useSegments, destroy } from 'markdown-it-fusion'

// 示例组件定义 (假设 MyComponent 已定义)
import MyComponent from './MyComponent.vue'

// 实例化 markdown-it 并使用插件 (为了示例完整性)
const md = MarkdownIt().use(useSegments, {
  components: {
    'my-component': { component: shallowRef(MyComponent) }
  }
})

const markdownText = ref('在 Markdown 中使用 <my-component>组件</my-component>')

const renderedResult = computed(() => {
  const html = md.render(markdownText.value)
  return useSegments(html)
})

const segments = computed(() => renderedResult.value.segments)
const instanceId = computed(() => renderedResult.value.id)

// 在组件销毁时，清理 markdown-it-fusion 实例缓存
onBeforeUnmount(() => {
  if (instanceId.value) {
    destroy(instanceId.value)
  }
})
</script>
```

### 3. 在 React 中渲染

逻辑与 Vue 类似，使用 `useMemo` 来处理渲染结果。
#### 3.1 定义你的自定义组件 (例如 MyComponent.jsx)

```jsx
import React from 'react';

// 这是一个普通的 React 函数式组件
function MyComponent({ type, _isComplete, _attrs }) {
  console.log('正在渲染 MyComponent，类型为:', type);

  return (
    <div style={{ border: '1px solid blue', padding: '10px', margin: '10px 0' }}>
      <h4>这是一个自定义 React 组件</h4>
      <p>类型 (type): {type}</p>
      <p>数据流是否完成 (_isComplete): {_isComplete ? '是' : '否'}</p>
      <p>从标签解析的属性 (_attrs): {JSON.stringify(_attrs)}</p>
    </div>
  );
}

// [性能优化关键] 使用 React.memo 包裹你的组件。
// 这可以防止在 props 没有实际变化时发生不必要的重新渲染。
export default React.memo(MyComponent);
```
#### 3.2 创建 Markdown 渲染器组件 (MarkdownRenderer.jsx)
```jsx
import React, { useEffect } from 'react';
import MarkdownIt from 'markdown-it';
import mdFusion, { useSegments, destroy } from 'markdown-it-fusion';
import MyComponent from './MyComponent'; // 导入上面用 memo 包装的组件

// --- 初始化 markdown-it ---
// 这部分代码通常在组件外部或使用 useMemo 创建，以避免重复初始化
const md = new MarkdownIt();
md.use(mdFusion, {
  components: {
    'my-component': { component: MyComponent, propsUseJson: true }
  }
});
// --------------------------

function MarkdownRenderer({ markdownText }) {
  // 1. 在每次渲染时，计算出 segments
  // 注意：为了性能，推荐将这部分逻辑包裹在 useMemo 中，如下面的注释所示。
  const { id: instanceId, segments } = useSegments(md.render(markdownText));
  
  /*
   * [性能提示]
   * 为了避免在每次父组件更新时都重新执行 md.render 和 useSegments，
   * 考虑上面的计算过程包裹在 React.useMemo 中：
   *
   * const { id: instanceId, segments } = React.useMemo(() => {
   *   const html = md.render(markdownText);
   *   return useSegments(html);
   * }, [markdownText]);
   * 
   */

  // 2. 使用 useEffect 来处理组件卸载时的缓存清理
  useEffect(() => {
    // 返回一个清理函数
    return () => {
      if (instanceId) {
        destroy(instanceId);
      }
    };
  }, [instanceId]); // 仅在 instanceId 变化时重新设置清理函数

  // 3. 将 segments 渲染为 React 元素
  return (
    <div className="content">
      {segments.map((item) => {
        if (item.type === 'html') {
          return (
            <div key={item.id} dangerouslySetInnerHTML={{ __html: item.content }} />
          );
        }
        if (item.type === 'component') {
          const Component = item.component;
          return <Component key={item.id} {...item.props} />;
        }
        return null;
      })}
    </div>
  );
}

export default MarkdownRenderer;

```

---

## ⚙️ API 和配置选项

### 插件顶级选项 (`MDComponentPluginOptions`)

| 参数 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| **`components`** | `Record<string, MDComponentOptions>` | **必填** | 注册的组件映射表。 |
| `placeholderClass` | `string` | `'custom-placeholder'` | 全局占位符的 CSS 类名。 |
| `propsKey` | `string` | `'_data'` | 全局组件接收内容的 Prop 名称。 |
| `debug` | `boolean` | `false` | 是否开启调试模式。 |

### 单个组件选项 (`MDComponentOptions`)

| 参数 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| **`component`** | `any` | **必填** | 要渲染的组件对象 (Vue/React)。 |
| `renderIntermediate` | `boolean` | `false` | 数据流未完成时是否提前渲染组件。 |
| `loadingText` | `string` | `'加载中...'` | `renderIntermediate: false` 时占位符显示的文本。 |
| `propsUseJson` | `boolean` | `false` | 是否尝试将标签内容解析为 JSON。 |
| `multipleProps` | `boolean` | `false` | 若 `propsUseJson` 成功，是否将 JSON 对象解构为多个 Props。 |
| `propsKey` | `string` | `'_data'` | 覆盖全局设置，当前组件接收内容的 Prop 名称。 |
| `placeholderClass` | `string` | `'custom-placeholder'` | 覆盖全局设置，当前组件占位符的 CSS 类名。 |

### 辅助函数

#### 1. `useSegments(html)`

解析 `md.render()` 返回的 HTML 字符串，并返回结构化的段落列表。

-   **返回值**: `{ id: string, segments: SegmentsResultItem[] }`

**`SegmentsResultItem` 对象结构**:

```ts
{
  type: 'html' | 'component', // 段落类型
  id: string,                 // 唯一 ID
  content?: string,           // HTML 内容 (仅 type: 'html')
  component?: any,            // 组件对象 (仅 type: 'component')
  props?: {
    [key: string]: any,
    _attrs?: Record<string, any>, // 标签上的所有 HTML 属性
    _isComplete?: boolean         // 数据流是否已完成
  }
}
```

#### 3. `destroy(instanceId?)`

销毁并清理缓存。这在单页应用 (SPA) 中尤为重要，可防止内存泄漏。

-   **`instanceId`** (可选): 从 `useSegments` 或 `useInstanceId` 获取的实例 ID。如果提供，则只销毁该实例；如果留空，则销毁所有实例的缓存。

```ts
import { useInstanceId, destroy } from 'markdown-it-fusion'

// useSegments 或 useInstanceId 返回的实例 ID
// const { id } = useSegments(html)
const id = useInstanceId(html)
// 销毁指定实例
destroy(id)
// 销毁所有实例
destroy()
```

---

## ⚠️ 注意事项

-   **依赖**: 插件本身依赖 `markdown-it`。
-   **JSON 解析**: 当 `propsUseJson: true` 时，如果 JSON 解析失败，内容将作为普通字符串回退。
-   **流式渲染**: 在流式场景下，通过 `renderIntermediate` 选项来控制组件在数据接收完成前的占位行为。
-   **内存管理**: 在 Vue/React 等 SPA 环境中，请务必在组件销毁时调用 `destroy()` 方法以清理缓存。

---

## 📜 许可证

[MIT](https://github.com/renqiankun/markdown-it-vue-component-demo/blob/main/LICENSE)
