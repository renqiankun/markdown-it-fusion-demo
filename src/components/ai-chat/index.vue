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
## 功能特点
- 支持在 Markdown 中插入 任意自定义Vue 组件
- 支持组件数据 JSON 化传递
- 支持组件占位符和加载状态
- 支持获取渲染后的 HTML 与组件段落列表 (getSegments)
- 基于 markdown-it 的一个插件，不影响 markdown-it 的其他功能及插件
- 基于 基于markdown-it的其他第三方组件库，都可经简单修改后使用
- 数据处理为文本与组件格式，天然支持 Vue 的 diff 加载 DOM
- 在markdown-it中启用和关闭html同时适用
- 组件缓存，组件在数据接收完成后，props不会再更新，极大提升了性能及组件处理数据的稳定性

-

\`\`\`markdown
下面数据格式：
 <my-component> { "type": "Hello" } </my-component>
\`\`\`

<my-component> { "type": "Hello" } </my-component>

或数据单独为 markdown 的块级结构，上下换行区分：

\`\`\`markdown
下面数据格式：
<my-component>
  { "type": "word" } 
</my-component>
\`\`\`

<my-component>
  { "type": "word" } 
</my-component>


### echarts组件
<echart>
[
  {"name":"Line 1","type":"line","data":[220,302,181,234,210,290,150]},
  {"name":"Line 2","type":"line","data":[120,202,281,271,230,220,130]}
]
</echart>

-
### element-plus table组件
<my-table>
[
  {"date":"2025-10-01","name":"小刀","address":"天津"},
  {"date":"2025-10-02","name":"小刀2","address":"合肥"}
]
</my-table>

结尾
  `

  console.log(list.length)
  //  messages.value[0].content += list
  for (var i of list.split('')) {
    await sleepHand(1)
    messages.value[0].content += i
  }
}

defineExpose({
  init
})
</script>

<style lang="scss" scoped></style>
