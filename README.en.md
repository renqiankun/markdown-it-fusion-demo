[English](README.en.md) | [中文](README.md)


# markdown-it-vue-component

> A Vue 3 plugin based on Markdown-it, which allows rendering Vue components inside Markdown. Supports streaming rendering and custom placeholders.


#### [github](https://github.com/renqiankun/markdown-it-vue-component-demo)
#### [demo](https://renqiankun.github.io/markdown-it-vue-component-demo/dist/)
---


## Features

- **Embed Anything**: Seamlessly insert any custom Vue component directly into your Markdown.
- **JSON-Powered Props**: Easily pass data to your components using clean JSON.
- **Smooth Loading States**: Display elegant placeholders and loading indicators while data is streaming.
- **Segmented Output**: Get a structured list of HTML and component segments for advanced rendering control.
- **Plays Well with Others**: A pure `markdown-it` plugin, ensuring full compatibility with the existing ecosystem.
- **Easily Adaptable**: Simple to modify for use with other `markdown-it` based component libraries.
- **Vue-Native Performance**: Leverages Vue's native diffing for highly efficient DOM updates.
- **Flexible HTML Mode**: Works flawlessly with or without `markdown-it`'s HTML mode enabled.
- **Smart Caching**: Boosts performance and stability by caching components and preventing unnecessary prop updates after data is fully loaded.


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
  props?: any                  // Props for the component
}
```

---

## Notes

- This plugin depends on Vue 3 and Markdown-it  
- If `propsUseJson` is true and JSON parsing fails, data will be passed as a string  
- In streaming rendering, `renderIntermediate` controls the placeholder behavior before the data is complete  
- Each display area uses a separate Markdown-it instance

---

## License

MIT
