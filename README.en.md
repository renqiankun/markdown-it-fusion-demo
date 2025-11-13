[English](README.en.md) | [中文](README.md)


# markdown-it-vue-component

> A plugin based on Markdown-it, which allows rendering components inside Markdown. Supports streaming rendering and custom placeholders.


#### [github](https://github.com/renqiankun/markdown-it-vue-component-demo)
#### [demo](https://renqiankun.github.io/markdown-it-vue-component-demo/dist/)
---


## Features

* **Powerful Componentization**
  * **Inject Any Component**: Seamlessly render any custom Vue or React component inside Markdown — breaking free from static text limitations.  
  * **HTML-like Syntax**: Use intuitive `<tag>...</tag>` syntax with full support for attributes, self-closing (`<tag />`), and implicit self-closing tags (`<img>`, `<input>`). Extremely low learning curve.  
  * **Flexible Data Passing**: Supports passing tag content as plain text or JSON strings (auto-parsed via option), enabling true data-driven rendering.

* **Streaming-First Design**
  * **Graceful Loading Placeholders**: During streaming, you can show loading states or pre-render components to avoid flicker and layout shifts — delivering a smooth user experience.  
  * **Intelligent State Awareness**: Components are aware of whether the data stream has finished, enabling post-stream actions like syntax highlighting or chart rendering.  
  * **Performance Locking**: Once streaming is complete, component props are intelligently frozen to prevent redundant re-renders, ensuring performance and stability.

* **Structured Output**
  * **`getSegments` API**: Splits the render result into **HTML fragments** and **component mount points**, compatible with both Vue and React renderers.

* **High Compatibility**
  * **Based on markdown-it**: Integrates cleanly without breaking core markdown-it behavior or other plugins.  
  * **Dual-Mode Support**: Works consistently whether the markdown-it `html` option is enabled or disabled.  
  * **Easily Extensible**: Clear and modular architecture — can be adapted to other markdown-it-based component systems with minimal effort.


---

## Installation

```bash
npm install markdown-it-vue-component
# or using yarn
yarn add markdown-it-vue-component
```

---

## Usage

```ts
import { ref, watch, onMounted } from 'vue'
import MarkdownIt from 'markdown-it'
import customComponentPlugin, { type MDVueComponentOptions, type SegmentsResultItem } from 'markdown-it-vue-component'
import MyComponent from './my-component.vue'

const md = MarkdownIt()

// Full configuration
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

// Minimal configuration
md.use(customComponentPlugin, {
  components: {
    'my-component': { component: MyComponent } 
  }
})

// Render Markdown
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

## Rendering in Vue Template

```vue
<template>
  <div>
    <template v-for="item in segments" :key="item.id">
      <!-- Regular HTML segment -->
      <div v-if="item.type === 'html'" v-html="item.content"></div>
      <!-- Vue component segment -->
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

## Plugin Options

### MDVueComponentPluginOptions

| Option           | Type                                  | Default               | Description                          |
| ---------------- | ------------------------------------- | -------------------- | ------------------------------------ |
| components       | Record<string, MDVueComponentOptions> | Required             | List of registered components        |
| placeholderClass | string                                | 'custom-placeholder' | CSS class for placeholders           |
| propsKey         | string                                | '_data'              | The prop name used to pass data      |
| debug            | boolean                               | false                | Enable debug mode                    |

### MDVueComponentOptions

| Option             | Type    | Default               | Description                                         |
| ------------------ | ------- | -------------------- | --------------------------------------------------- |
| component          | any     | Required             | Vue 3 component                                     |
| renderIntermediate | boolean | false                | Render the component before data is fully received |
| loadingText        | string  | 'Loading...'         | Placeholder text when component is loading         |
| propsUseJson       | boolean | false                | Parse data as JSON if true                          |
| multipleProps      | boolean | false                | Destructure JSON and pass multiple props           |
| propsKey           | string  | '_data'              | Prop name for component data                        |
| placeholderClass   | string  | 'custom-placeholder' | CSS class for placeholder                           |

---

## Get Segments

```ts
const segments = md.getSegments(html)
```

Each segment object has the following structure:

```ts
{
  type: 'html' | 'component', // HTML segment or Vue component segment
  id: string | number,
  content?: string,            // Content for HTML segment
  component?: any,             // Vue component
  props?: {
  [key: string]: any,
   _attrs?: Record<string, any>,
  _isComplete?: boolean
  }
}
```

---

## Notes

- This plugin depends on Markdown-it  
- If `propsUseJson` is true and JSON parsing fails, data will be passed as a string  
- In streaming rendering, `renderIntermediate` controls the placeholder behavior before the data is complete  
- Each display area uses a separate Markdown-it instance

---

## License

MIT
