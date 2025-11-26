<template>
  <!-- 用户头像及描述 -->
  <div class="user-info-wrap">
    <div class="user-head">A</div>
    <div class="user-question">
      <template v-for="item in renderSegments">
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
import 'markdown-it-fusion/style.css'
import customComponentPlugin, {
  useSegments,
  type MDComponentOptions,
  type SegmentsResultItem
} from 'markdown-it-fusion'
import { ref, shallowRef, watch } from 'vue'
import myChart from '@/components/my-chart.vue'
import myImg from '@/components/my-img.vue'
import myCard from '@/components/my-card.vue'
import myTable from '@/components/my-table.vue'
const props = withDefaults(
  defineProps<{
    data: string
  }>(),
  {}
)

const md = markdownIt({
  // html: false,     
})
// 完整配置
md.use(customComponentPlugin, {
  components: {
    'my-card': {
      component: shallowRef(myCard),
      propsUseJson: true,
      multipleProps: true,
    } as MDComponentOptions,
    'my-chart': {
      component: shallowRef(myChart),
      propsUseJson: true,
      propsKey: 'data'
    },
    'my-table': {
      component: shallowRef(myTable),
      propsUseJson: true,
      propsKey: 'data'
    },
    img: {
      component: shallowRef(myImg)
    }
  }
})

const renderSegments = ref<Array<SegmentsResultItem>>([])
const renderHtmlHand = (newVal: string) => {
  // 获取普通html
  let html = md.render(newVal)
  // 构造html与vue组件数据的list,用于后续遍历渲染
 let {id,segments} = useSegments(html)
 renderSegments.value = segments
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
