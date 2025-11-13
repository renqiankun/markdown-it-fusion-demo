<template>
  <aiChat v-model="messages"></aiChat>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'
import aiChat from './Chat.vue'
import { sleepHand } from '@/utils'
let dialogVisible = ref(false)

let messages = ref<Array<any>>([
  {
    role: 'assistant',
    useReasoner: false,
    content: ``,
    reasoning_content: ''
  }
])

onMounted(() => {
  init()
})

const init = async () => {
  dialogVisible.value = true

  let list = `
### 以下为部分示例：
-

\`\`\`markdown
 my-card组件已设置为数据转json（propsUseJson:true），
 且解构后传入组件（multipleProps：true）
 <my-card tag='a' isBlock > { "type": "Hello" } </my-card>
\`\`\`

 <my-card tag='a' isBlock > { "type": "Hello" } </my-card>

\`\`\`markdown 
<my-card tag='b'/> 
\`\`\`
 <my-card tag='b'/> 


\`\`\`markdown
占多行数据：
<my-card tag='b'>
  { "type": "word" } 
</my-card>
\`\`\`
<my-card tag='b'>
  { "type": "word" } 
</my-card>

### echarts组件
\`\`\`markdown
<echart>
[
  {"name":"Line 1","type":"line","data":[220,302,181,234,210,290,150]},
  {"name":"Line 2","type":"line","data":[120,202,281,271,230,220,130]}
]
</echart>
\`\`\`
<echart>
[
  {"name":"Line 1","type":"line","data":[220,302,181,234,210,290,150]},
  {"name":"Line 2","type":"line","data":[120,202,281,271,230,220,130]}
]
</echart>

-
### element-plus table组件
\`\`\`markdown
<my-table>
[
  {"date":"2025-10-01","name":"小刀","address":"天津"},
  {"date":"2025-10-02","name":"小刀2","address":"合肥"}
]
</my-table>
\`\`\`
<my-table>
[
  {"date":"2025-10-01","name":"小刀","address":"天津"},
  {"date":"2025-10-02","name":"小刀2","address":"合肥"}
]
</my-table>


-
### 重写img，img替换为Element-plus组件
\`\`\`markdown
<img src='https://fuss10.elemecdn.com/a/3f/3302e58f9a181d2509f3dc0fa68b0jpeg.jpeg'>
\`\`\`
-
<img src='https://fuss10.elemecdn.com/a/3f/3302e58f9a181d2509f3dc0fa68b0jpeg.jpeg'>

-
### 核心优势
-
*   **强大的组件化能力**
    *   **任意组件注入**：在 Markdown 中无缝渲染任何自定义  vue、react 组件，打破静态文本的限制。
    *   **标准类 HTML 语法**：采用直观的 <tag>...</tag> 风格，完整支持标签属性、自闭合 (<tag />) 及隐式自闭合 (\`<img>\`，\`<input>\`)，学习成本极低。
    *   **灵活的数据传递**：支持将标签内容作为普通文本或 JSON字符串（参数可控制转为json）传递给组件，轻松实现数据驱动。
-
*   **为流式渲染而生 (Streaming-First)**
    *   **优雅的加载占位**：在数据流未完成时可控制显示加载状态或直接渲染组件，无组件闪烁和布局抖动，极大提升用户体验。
    *   **智能状态感知**：组件能明确获知数据流是否结束，从而执行数据加载完毕后的特定逻辑（如代码高亮、图表渲染等）。
    *   **性能缓存机制**：组件数据流完成后，组件 Props 将被智能锁定，杜绝组件不必要的重复渲染，确保高性能和组件状态稳定性。
-
*   **结构化输出**
    *   **结构化输出 (getSegments)**：将渲染结果拆分为 **HTML 片段**和**待挂载的 组件**列表，不限制vue、react渲染
-
*   **高兼容性**
    *   **基于 markdown-it 插件**：不影响 markdown-it 核心功能或其他插件的正常运作。
    *   **双模式兼容**：无论 markdown-it 的 html 选项开启或关闭，插件均能提供稳定一致的解析能力。
    *   **易于扩展**：架构清晰，可经简单修改后与其他基于 markdown-it 的组件库协同工作。

-
类HTML语法，支持属性、自闭合、隐式自闭合，
自定义标签默认成对闭合（可参数配置，内部也会智能判断）
原生html中img、input等默认自闭合

-

### 以上涉及到组件的配置：

\`\`\`markdown
md.use(customComponentPlugin, {
  components: {
    'my-card': {
      // component参数在内部无任何处理，生成segments原样返回，可传字符串，自己匹配组件。
      component: shallowRef(myCard),
      renderIntermediate: false,
      propsUseJson: true,
      multipleProps: true,
      propsKey: '_data',
      placeholderClass: 'custom-placeholder'
    },
    echart: {
      component: shallowRef(echartCom),
      propsUseJson: true,
      propsKey: 'data',
    },
    'my-table': {
      component: shallowRef(myTable),
      propsUseJson: true,
      propsKey: 'data',
    },
    'img':{
      component: shallowRef(myImg),
    }
    
  }
})
\`\`\`

结尾
`

  console.log(list.length)
  //  messages.value[0].content += list
  for (var i of list.split('')) {
    await sleepHand(30)
    messages.value[0].content += i
  }
}

defineExpose({
  init
})
</script>

<style lang="scss" scoped></style>
