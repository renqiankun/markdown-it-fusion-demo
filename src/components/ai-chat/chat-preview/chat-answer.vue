<template>
  <!-- 用户头像及描述 -->
  <div class="user-info-wrap">
    <div class="user-head">A</div>
    <div class="user-question">
      <template v-for="item in segments">
        <div v-if="item.type == 'html'" :key="item.id" v-html="item.content"></div>
        <component
          v-if="item.type == 'component'"
          :is="item.component"
          :key="item.id"
          v-bind="item.props"
        ></component>
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
import markdownIt from 'markdown-it'
import 'markdown-it-vue-component/style.css'
import customComponentPlugin, {
  type MDVueComponentOptions,
  type SegmentsResultItem
} from 'markdown-it-vue-component'
import myComponent from '@/components/my-component.vue'
import echartCom from '@/components/echart/index.vue'
import myTable from '@/components/ele-table/index.vue'
import { ref, shallowRef, watch } from 'vue'
const props = withDefaults(
  defineProps<{
    data: string
  }>(),
  {}
)

const md = markdownIt()
// 完整配置
md.use(customComponentPlugin, {
  debug: false,
  propsKey: '_data',
  placeholderClass: 'custom-placeholder',
  components: {
    'my-component': {
      // component参数在内部无任何处理，生成segments原样返回，可传字符串，自己匹配组件。
      component: shallowRef(myComponent),
      renderIntermediate: false,
      propsUseJson: true,
      multipleProps: true,
      propsKey: '_data',
      placeholderClass: 'custom-placeholder'
    } as MDVueComponentOptions,
    // echarts
    echart: {
      component: shallowRef(echartCom),
      propsUseJson: true,
      propsKey: 'data',
    },
    // 表格
    'my-table': {
      component: shallowRef(myTable),
      propsUseJson: true,
      propsKey: 'data',
    },
    
  }
})

const segments = ref<Array<SegmentsResultItem>>([])
const renderHtmlHand = (newVal: string) => {
  // 获取普通html
  let html = md.render(newVal)
  // 构造html与vue组件数据的list,用于后续遍历渲染
  segments.value = (md as any).getSegments(html)
}

watch(
  () => props.data,
  (newVal) => {
    if (newVal) {
      renderHtmlHand(newVal)
    }
  },
  {
    immediate: true
  }
)
</script>

<style lang="scss" scoped>
.user-info-wrap {
  display: flex;
  align-items: start;
  column-gap: 20px;
  box-sizing: border-box;
  padding-right: 2px;
  margin-bottom: 32px;
  color: var(--user-question-text-color);
}
.user-head {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background-color: var(--user-question-bg-color);
  display: flex;
  align-items: center;
  justify-content: center;
  color: #000;
  margin-top: 6px;
}
.user-question {
  // padding-top: 3px;
  flex: 1;
  overflow: hidden;
  box-sizing: border-box;
  padding: 0 5px;
}
</style>
