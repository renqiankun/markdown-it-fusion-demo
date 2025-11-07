<template>
  <div class="btn-send-wrap" :class="{ disabled: disabled || loading }" @click="clickHand">
    <el-icon :class="{ 'is-loading': loading }" size="18">
      <component :is="iconComponent"></component
    ></el-icon>
  </div>
</template>

<script setup lang="ts">
import { computed, type Component } from 'vue'
import { Top, Loading } from '@element-plus/icons-vue'
const props = withDefaults(
  defineProps<{
    disabled?: boolean
    loading?: boolean
    icon?: Component
  }>(),
  {
    icon: Top
  }
)

let iconComponent = computed(() => {
  if (props.loading) return Loading
  return props.icon
})

const clickHand = () => {
  if (props.disabled || props.loading) return
  emits('click')
}
const emits = defineEmits(['click'])
</script>

<style lang="scss" scoped>
.btn-send-wrap {
  width: 32px;
  height: 32px;
  cursor: pointer;
  border-radius: 50%;
  background-color: var(--el-color-primary);
  display: flex;
  justify-content: center;
  align-items: center;
  color: #fff;
  &.disabled {
    background-color: rgb(0 168 255 / 90%);;
    cursor: not-allowed;
    // opacity: 0.5;
    --color:rgb(255 255 255 / 60%);
  }
}
</style>
