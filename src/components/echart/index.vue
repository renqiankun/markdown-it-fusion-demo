<template>
  <div ref="containerRef" class="chart-custom-wrap" v-bind="$attrs">
    <div v-if="_isComplete && visible" ref="lineRef" class="chart-custom-inner"></div>
    <div v-if="!_isComplete" class="chart-custom-text">暂无数据</div>
  </div>
</template>

<script setup lang="ts">
import { ref, onBeforeUnmount, nextTick, watch, shallowRef, onMounted, computed } from 'vue'
import * as echarts from 'echarts'

let props = withDefaults(
  defineProps<{
    data: any
    colors?: any[]
    _isComplete?: boolean
  }>(),
  {
    data: '',
    colors: () => ['#00AAFF', '#71F0A2', '#FFCA00']
  }
)
let myChart: any = shallowRef()
let lineRef: any = ref('')
let visible = ref(false)

const resetOption = async () => {
  const option: any = {
    color: props?.colors ?? [],
    tooltip: {
      trigger: 'axis'
    },
    legend: {
      x: 'center',
      y: '-4px',
      itemWidth: 18,
      itemHeight: 9,
      itemGap: 20,
      textStyle: { color: '#666C7C', fontSize: 12, lineHeight: 20, fontWeight: 500 },
      pageIconColor: '#fff',
      pageIconInactiveColor: '#999',
      pageIconSize: 10
    },
    grid: {
      right: 0,
      bottom: 10,
      left: 0,
      top: '40px',
      containLabel: true
    },
    backgroundColor: 'transparent',
    xAxis: {
      type: 'category',
      data: ['2022-01', '2022-02', '2022-03', '2022-04', '2022-05', '2022-06', '2022-07'],
      boundaryGap: false, // false线条贴边
      nameTextStyle: { color: '#666c7c' },
      axisLine: { show: false, lineStyle: { color: '#777b89' } }, // 横坐标
      axisTick: { show: false, alignWithLabel: true },
      splitLine: { show: true, lineStyle: { color: 'rgba(255, 255, 255, 0.1)' } }, // 竖向
      // 横坐标文字
      axisLabel: {
        show: true,
        color: '#666C7C',
        fontSize: 12,
        fontWeight: 500,
        padding: [10, 0, 0, 0]
      }
    },
    yAxis: [
      {
        type: 'value',
        name: '元',
        nameTextStyle: { color: '#86909C', padding: [0, 0, 0, -40] },
        position: 'left',
        splitLine: { lineStyle: { color: '#E5E8EF' } },
        // 纵坐标文字
        axisLabel: { color: '#86909C', fontSize: 12 }
      }
    ],
    series: [
      // {
      //   name: 'Line 1',
      //   type: 'line',
      //   data: [220, 302, 181, 234, 210, 290, 150]
      // }
    ]
  }
  option.series = props.data?.map?.((item: any, index: number) => {
    return {
      ...item,
      smooth: true,
      symbolSize: 6,
      lineStyle: lineStyle(index),
      areaStyle: areaStyle(index)
    }
  })
  // 默认先销毁
  visible.value = false
  await nextTick()
  visible.value = true
  await nextTick()
  myChart.value = echarts.init(lineRef.value)

  myChart.value?.setOption(option)
}
const hexToRgba = (hex:any, opacity:any) => {
  let rgbaColor = ''
  const reg = /^#[\da-f]{6}$/i
  if (reg.test(hex)) {
    rgbaColor = `rgba(${parseInt('0x' + hex.slice(1, 3))},${parseInt('0x' + hex.slice(3, 5))},${parseInt('0x' + hex.slice(5, 7))},${opacity})`
  }
  return rgbaColor
}
const lineStyle = (index: number) => {
  const color = props.colors
  return {
    normal: {
      color: color[index],
      shadowColor: hexToRgba(color[index], 0.5),
      shadowBlur: 3,
      shadowOffsetY: 8
    }
  }
}

const areaStyle = (index: number) => {
  const color = props.colors
  return {
    normal: {
      color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
        {
          offset: 0,
          color: hexToRgba(color[index], 0.3)
        },
        {
          offset: 1,
          color: hexToRgba(color[index], 0.1)
        }
      ]),
      shadowColor: hexToRgba(color[index], 0.1),
      shadowBlur: 10
    }
  }
}

const destroyHand = () => {
  myChart.value?.dispose?.()
  myChart.value = null
}

let containerRef = ref()

onBeforeUnmount(() => {
  destroyHand()
  lineRef.value = null
})

watch(
  () => [props.data],
  () => {
    if (props._isComplete) {
      resetOption()
    }
  },
  {
    flush: 'post',
    immediate: true
    // deep: true
  }
)
</script>

<style lang="scss" scoped>
$minHeight: 280px;
.chart-custom-wrap {
  height: 100%;
  height: $minHeight;
  width: 100%;
  text-align: center;
  font-size: 14px;
  color: #909399;
  .chart-custom-inner {
    min-height: $minHeight;
    height: 100%;
    .r-echarts-tooltip {
      background-color: #909399;
    }
  }
  .chart-custom-text {
    display: flex;
    height: 100%;
    align-items: center;
    justify-content: center;
  }
}
</style>

<style lang="scss">
.chart-custom-wrap {
}
</style>
